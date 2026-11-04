# Lab Proof: Telegram screams, Airtable remembers
**Workflow:** Unioptt enquiries: Telegram → Airtable

```
Telegram Trigger  →  Edit Fields (Set)  →  Airtable (Create a record)
    (source)          (translator)            (destination)
```

## 1. Scenario
**Unioptt Enquiry Bot.** Unioptt helps students find a university abroad. Students and parents often ask questions informally instead of filling in a form, so Unioptt shares a Telegram bot link on Instagram and flyers where they can simply write, e.g. "I want to study psychology in Germany, what are my options?". Without automation these messages stay in a chat with no overview, and enquiries get lost or answered late. The workflow logs every message as one structured row in an Airtable enquiry tracker, so each request can be sorted, followed up and later fed into the matching process.

**Technical behaviour:** When a student sends a message to the Unioptt bot, log their first name, username, chat ID, message and timestamp as a new record in the Airtable table `Enquiries` (base `Unioptt Enquiries`).

## 2. Workflow export
File: `unioptt_telegram_airtable.json` (n8n → ⋯ → Download).
Credentials are stored only in n8n Credentials; the export contains only credential **references** (name/ID), no tokens. Credential IDs and the webhook ID were removed before sharing.

| Node | Type | Key settings |
|---|---|---|
| Telegram Trigger | `telegramTrigger` | Trigger On: message · credential `Telegram – UniopttBot` |
| Edit Fields | `set` (Manual Mapping) | 5 String fields, see mapping table |
| Create a record | `airtable` | Resource: Record · Operation: Create · Base: Unioptt Enquiries · Table: Enquiries · Mapping: Map Automatically · credential `Airtable – n8n-lab` (scopes: `data.records:read`, `data.records:write`, `schema.bases:read`; access: this base only) |

## 3. Source payload (Telegram Trigger output)
```json
[
  {
    "update_id": 287122355,
    "message": {
      "message_id": 2,
      "from": {
        "id": 1488716269,
        "is_bot": false,
        "first_name": "Elif",
        "last_name": "Yurdakul",
        "language_code": "en"
      },
      "chat": {
        "id": 1488716269,
        "first_name": "Elif",
        "last_name": "Yurdakul",
        "type": "private"
      },
      "date": 1793804662,
      "text": "I want to study psychology in Germany. What are my options?"
    }
  }
]
```
Note: the sender has **no `username`** field. Telegram leaves it out when a user has not set an @username.

## 4. Transform (Edit Fields output)
```json
{
  "Name": "Elif",
  "Username": "",
  "Message": "I want to study psychology in Germany. What are my options?",
  "Chat ID": "1488716269",
  "Timestamp": "2026-11-04T16:04:22.000+01:00"
}
```

## 5. Destination record (Airtable node output)
```json
[
  {
    "id": "recNUTw1aqz5v9H09",
    "createdTime": "2026-11-04T15:27:21.000Z",
    "fields": {
      "Timestamp": "2026-11-04T15:04:22.000Z",
      "Chat ID": "1488716269",
      "Message": "I want to study psychology in Germany. What are my options?",
      "Name": "Elif"
    }
  }
]
```
Airtable view: the row shows `Elif | (empty) | I want to study psychology… | 1488716269 | 11/4/2026 4:04pm`.
`Username` is missing from the output because Airtable does not return empty fields.

## 6. Execution trace (live run)
After unpinning and publishing, a new real message was sent to the bot:
"Hi, I'm interested in a master's in data science in the Netherlands. What do I need?"

- Executions: **Telegram Trigger → Edit Fields → Create a record**, all successful, **726 ms**
- Source `date: 1793806195` → Airtable row 3: `Elif | | Hi, I'm interested in a mas… | 1488716269 | 11/4/2026 4:29pm`
- Screenshots: `execution_trace.png`, `airtable_records.png`

## 7. Mapping table
| Source (Telegram) | Destination (Airtable) | Transformation | Why |
|---|---|---|---|
| `message.from.first_name` | Name | none | Shows who wrote; the person's display name |
| `message.from.username` | Username | `\|\| ''` (empty text if missing) | Many users have no @username; without the fallback the value is `undefined` |
| `message.text` | Message | none | The enquiry itself; Long text column for longer messages |
| `message.chat.id` | Chat ID | `String(...)` number → text | The Airtable column is text; the ID is the address to reply to this student later (in a private chat `chat.id` = `from.id`) |
| `message.date` | Timestamp | `DateTime.fromSeconds(...).toISO()` Unix seconds → ISO date | Airtable needs a real date; we store when the enquiry **arrived**, not when the row was written |

Field names in Edit Fields match the Airtable columns exactly (case-sensitive), so **Map Automatically** works. Edit Fields is the translator: nested Telegram JSON in, flat, correctly typed fields out.

**Timestamp check (same moment in three formats):**
| Step | Value |
|---|---|
| Telegram | `1793804662` (Unix seconds) |
| Edit Fields | `2026-11-04T16:04:22.000+01:00` (Berlin) |
| Airtable | `2026-11-04T15:04:22.000Z` (UTC), displayed as 4:04pm local |

`createdTime` (15:27Z) ≠ `Timestamp` (15:04Z): during building I used **pinned data**, so the row was written 23 minutes after the message was sent. This is why the message time is stored as its own field.

## 8. Explain: which field breaks first, and how would I detect it?
**`message.text`** is the most fragile field. It only exists for plain text messages:
- photos, voice messages, stickers or documents have no `text` (a photo has `caption` instead)
- an **edited** message arrives as `edited_message`, not `message`, so every path (`message.from…`, `message.text`) becomes empty

The row would still be created, but with an empty Message or empty Name. The workflow shows "success", so nobody notices.

**Detection and prevention:**
- an **If** node after the trigger: continue only when `{{ $json.message.text }}` is not empty; send other cases to a separate branch
- a daily check (or Airtable view filter) for rows where Message is empty
- an **Error Workflow** (Settings → Error Workflow) that alerts me on Telegram when a run fails

The second risk is `message.date`: if it is missing, `DateTime.fromSeconds` fails and the Airtable write stops.

## 9. Operational and consultant notes
- **Secrets:** Telegram and Airtable tokens are only in n8n Credentials, never in nodes, screenshots or this file. During the lab, a webhook secret was accidentally shared in class chat; it was deleted from n8n, replaced with a new one, and the old key was no longer accepted.
- **Least privilege:** the Airtable token has 3 scopes and access to one base only.
- **Personal data (GDPR):** name, chat ID and message text are personal data. Airtable stores data in the US. For real Unioptt users (possibly minors) this needs a privacy notice, a data processing agreement, or an EU-hosted destination. In this lab only my own test messages were stored.
- **One bot = one workflow:** Telegram keeps one webhook URL per bot.
- **Duplicates:** if the same message were processed twice, Airtable would get two rows. A fix would be **Create or Update** matched on `message_id` + `Chat ID`.
