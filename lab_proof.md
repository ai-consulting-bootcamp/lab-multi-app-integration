# Lab proof: Telegram screams, Sheets remembers

**Author:** Ayhan Demir · **Date:** 3 Nov 2026 · **Track:** Telegram Trigger → Edit Fields → Airtable (+ extensions)

This file is the entry point for the reviewer. Everything else in this folder is referenced from here.

| File | What it is |
|---|---|
| `lab_proof.md` | This file |
| `workflow_order_bot.json` | Main workflow export (IDs and credential IDs removed) |
| `workflow_error_alerts.json` | Error workflow export (IDs removed) |
| `screenshots/airtable_order_tracker.png` | Destination records in Airtable |
| `screenshots/telegram_chat_replies_and_alerts.png` | Customer view: orders, confirmations, "ask for text" reply, error alerts |
| `screenshots/slack_orders_channel.png` | Second destination: AI order summaries in `#orders` |

---

## 1. Scenario

A small bakery has no website order form: customers message the shop's Telegram bot with what they want. Orders pile up as chat messages, and nobody has a clear view of who ordered what and when, so requests get missed. The workflow captures every message automatically, writes one structured row into an Airtable order tracker, posts a one-line AI summary to the team's Slack channel, and confirms receipt to the customer. The team can sort and act on every order without scrolling through Telegram, and the customer knows the order arrived.

**One-sentence technical behavior:**
When a client sends a text message to the order bot, log their name, username, chat ID, message, a unique message ID and a UTC timestamp as a new record in an Airtable order tracker, post an AI summary to Slack, and reply with a confirmation.

---

## 2. Workflow export

Files: `workflow_order_bot.json` and `workflow_error_alerts.json`.
Removed before sharing: workflow IDs, version IDs, instance ID, webhook IDs, credential IDs, Airtable base and table IDs, owner chat ID. No tokens are stored in any node: all secrets live in n8n credentials.

```
Telegram Trigger ──┐
                   ├─► Has Text? ──true──► Edit Fields ─► AI summary (OpenAI) ─┬─► Airtable: create order ─► Telegram: confirm order
Dry run (manual) ─►│      │                                                    └─► Slack: post order
Sample order ──────┘      └─false─► Send a message: ask for order in text

Error alerts (separate workflow):  Error Trigger ─► Telegram: alert me
```

| Node | Role |
|---|---|
| Telegram Trigger | Source: receives every message sent to the bot (webhook, pushed by Telegram) |
| Dry run (manual) + Sample order for dry run | Step 5 dry run: same downstream path without sending a real message |
| Has Text? (Voice/Video rejected) | Step 9 guard: only messages with text continue |
| Send a message: ask for order in text | False branch: tells the customer to send the order as text |
| Edit Fields | The translator: flattens and converts the Telegram payload into Airtable columns |
| AI summary (OpenAI) | Extension: `gpt-5.4-mini` writes a one-line order summary |
| Airtable: create order | Destination: one record per message |
| Telegram: confirm order | Extension: confirmation reply, only after the record is saved |
| Slack: post order | Extension: second destination for the team |
| Error alerts workflow | Extension: any failed production run sends me a Telegram alert with the failing node and a link to the execution |

---

## 3. One source payload (Telegram Trigger output)

Execution: `[execution number of the 16:18 order]` · Message: "one soup 3 salads wto pastas and 1 piyya please".
Chat and user IDs are replaced with placeholders.

```json
{
  "update_id": "[from execution]",
  "message": {
    "message_id": 38,
    "from": {
      "id": "<CHAT_ID>",
      "is_bot": false,
      "first_name": "Ayhn",
      "last_name": "Dmr",
      "username": "ayhndmr7",
      "language_code": "en"
    },
    "chat": {
      "id": "<CHAT_ID>",
      "first_name": "Ayhn",
      "last_name": "Dmr",
      "username": "ayhndmr7",
      "type": "private"
    },
    "date": 1793719112,
    "text": "one soup 3 salads wto pastas and 1 piyya please"
  }
}
```

---

## 4. One destination record (Airtable)

Airtable base `Order Tracker`, table `Messages`, row 18 (see `screenshots/airtable_order_tracker.png`).

```json
{
  "id": "[rec... from Airtable node output]",
  "fields": {
    "Name": "Ayhn",
    "Username": "ayhndmr7",
    "Message": "one soup 3 salads wto pastas and 1 piyya please",
    "Chat ID": "<CHAT_ID>",
    "Message ID": "<CHAT_ID>-38",
    "Timestamp": "2026-11-03T15:18:32.000Z"
  }
}
```

Airtable displays the timestamp in local time (3.11.2026 4:18pm Berlin). The same message reached the other two outputs:

| Output | Evidence |
|---|---|
| Slack `#orders` | "New order from Ayhn (2026-11-03T15:18:32.000Z UTC) · Summary: 1 soup, 3 salads, 2 pastas, 1 pizza" |
| Telegram reply | "Thanks Ayhn, we got your order!" at 16:18 |

The AI summary also corrected the customer's typos ("wto pastas", "piyya") without adding items.

---

## 5. Mapping table

| Source (Telegram) | Destination (Airtable) | Transformation | Why |
|---|---|---|---|
| `message.from.first_name` | Name | none | Who ordered, in a form staff can read |
| `message.from.username` | Username | `\|\| ''` (empty string if missing) | Many Telegram users have no @username; an empty value is normal and must not break the run |
| `message.text` | Message | none | The order itself, stored exactly as written so nothing is lost or reinterpreted |
| `message.chat.id` | Chat ID | `String(...)` | Telegram sends a number; the Airtable column is text. Needed to reply to the right customer |
| `message.date` | Timestamp | `DateTime.fromSeconds(...).toUTC().toISO()`: Unix seconds → ISO 8601 UTC | Airtable's Date field needs a real date; UTC avoids timezone ambiguity, Airtable shows it in local time |
| `message.chat.id` + `message.message_id` | Message ID | concatenated, e.g. `<CHAT_ID>-38` | `message_id` is only unique within one chat, so the pair is a unique key. Lets any record be traced back to one exact Telegram message |
| `message.text` (via OpenAI) | Slack: Summary | `gpt-5.4-mini`, one-line summary | Team gets a scannable order line; Airtable keeps the original text as the source of truth |

All Edit Fields names match the Airtable column names exactly, letter by letter.

---