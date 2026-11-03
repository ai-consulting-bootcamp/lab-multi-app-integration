# Lab proof — Telegram screams, Sheets remembers

**Student:** Nicolò
**Workflow:** `Order bot: Telegram -> Airtable` (n8n)
**Track:** Telegram Trigger → If (guard) → Edit Fields (Set) → Airtable (Create a record) → Telegram (confirmation)
**Traced execution:** #1233 — 2026-11-03 15:11 (Berlin)

---

## 1. Scenario

**Order Intake Bot**
A small bakery has no website order form: customers message the shop's Telegram bot with what they want. Orders pile up as chat messages, and nobody has a clear view of who ordered what and when, so requests get missed. The workflow captures every incoming message automatically, writes one structured record into an Airtable order tracker and sends the customer an automatic confirmation. The team can sort and act on every order in Airtable without scrolling through Telegram.

**One-sentence technical behavior:**
When a customer sends a message to the order bot, log their name, username, chat ID, message and timestamp as a new record in the Airtable `Order Tracker` → `Messages` table, then reply to the customer on Telegram.

---

## 2. Workflow

```
Telegram Trigger ──► If (text ≠ "ciao") ──true──► Edit Fields ──► Create a record (Airtable) ──► Send a text message ("Thanks for reaching out…")
                                         └─false─► Send a text message1 ("Message is empty. Resend")

When clicking 'Execute workflow' (Manual Trigger, pinned sample payload) — used for dry runs, currently disconnected
```

| Node | Type | Role |
|---|---|---|
| Telegram Trigger | `telegramTrigger` (updates: message) | Source — receives the customer message |
| If | `if` | Guard — routes the message to Airtable or to an error reply |
| Edit Fields | `set` (manual mapping) | Translator — flattens and formats the payload |
| Create a record | `airtable` (Record → Create) | Destination — base `Order Tracker`, table `Messages` |
| Send a text message | `telegram` (sendMessage) | Confirmation to the customer |
| Send a text message1 | `telegram` (sendMessage) | Reply on the false branch of the guard |
| When clicking 'Execute workflow' | `manualTrigger` | Dry runs with a pinned sample payload |

### Workflow export (sanitized)

Credential IDs, base/table IDs, webhook IDs and the workflow ID are replaced with `<REDACTED>`. No tokens are stored in the workflow: they live only in the n8n credentials.

```json
{
  "name": "Order bot: Telegram -> Airtable",
  "nodes": [
    {
      "parameters": { "updates": ["message"], "additionalFields": {} },
      "type": "n8n-nodes-base.telegramTrigger",
      "typeVersion": 1.2,
      "position": [0, 0],
      "name": "Telegram Trigger",
      "webhookId": "<REDACTED>",
      "credentials": { "telegramApi": { "id": "<REDACTED>", "name": "Telegram - AC Lab Bot" } }
    },
    {
      "parameters": {
        "conditions": {
          "options": { "caseSensitive": true, "leftValue": "", "typeValidation": "strict", "version": 3 },
          "conditions": [
            {
              "leftValue": "={{ $json.message.text }}",
              "rightValue": "ciao",
              "operator": { "type": "string", "operation": "notEquals" }
            }
          ],
          "combinator": "and"
        },
        "looseTypeValidation": false,
        "options": {}
      },
      "type": "n8n-nodes-base.if",
      "typeVersion": 2.3,
      "position": [176, 0],
      "name": "If"
    },
    {
      "parameters": {
        "mode": "manual",
        "duplicateItem": false,
        "assignments": {
          "assignments": [
            { "name": "Name", "value": "={{ $json.message.from.first_name }}", "type": "string" },
            { "name": "username", "value": "={{ $json.message.from.username || '' }}", "type": "string" },
            { "name": "message", "value": "={{ $json.message.text }}", "type": "string" },
            { "name": "Chat ID", "value": "={{ String($json.message.chat.id) }}", "type": "string" },
            { "name": "Timestamp", "value": "={{ DateTime.fromSeconds($json.message.date).toISO() }}", "type": "string" }
          ]
        },
        "includeOtherFields": false,
        "options": {}
      },
      "type": "n8n-nodes-base.set",
      "typeVersion": 3.4,
      "position": [416, 0],
      "name": "Edit Fields"
    },
    {
      "parameters": {},
      "type": "n8n-nodes-base.manualTrigger",
      "typeVersion": 1,
      "position": [16, -144],
      "name": "When clicking 'Execute workflow'"
    },
    {
      "parameters": {
        "authentication": "airtableTokenApi",
        "resource": "record",
        "operation": "create",
        "base": { "__rl": true, "value": "<REDACTED>", "mode": "list", "cachedResultName": "Order Tracker" },
        "table": { "__rl": true, "value": "<REDACTED>", "mode": "list", "cachedResultName": "Messages" },
        "columns": {
          "mappingMode": "defineBelow",
          "value": {
            "Name": "={{ $json.Name }}",
            "Username": "={{ $json.username }}",
            "Message": "={{ $json.message }}",
            "Chat ID": "={{ $json['Chat ID'] }}",
            "Timestamp": "={{ $json.Timestamp }}"
          },
          "matchingColumns": [],
          "attemptToConvertTypes": false,
          "convertFieldsToString": false
        },
        "options": {}
      },
      "type": "n8n-nodes-base.airtable",
      "typeVersion": 2.2,
      "position": [624, 0],
      "name": "Create a record",
      "credentials": { "airtableTokenApi": { "id": "<REDACTED>", "name": "Airtable - n8n-lab" } }
    },
    {
      "parameters": {
        "resource": "message",
        "operation": "sendMessage",
        "chatId": "={{ $json.fields['Chat ID'] }}",
        "text": "Thanks for reaching out! We will get back to you shortly",
        "replyMarkup": "none",
        "additionalFields": {}
      },
      "type": "n8n-nodes-base.telegram",
      "typeVersion": 1.2,
      "position": [832, 0],
      "name": "Send a text message",
      "webhookId": "<REDACTED>",
      "credentials": { "telegramApi": { "id": "<REDACTED>", "name": "Telegram - AC Lab Bot" } }
    },
    {
      "parameters": {
        "resource": "message",
        "operation": "sendMessage",
        "chatId": "={{ $('Telegram Trigger').item.json.message.chat.id }}",
        "text": "Message is empty. Resend",
        "replyMarkup": "none",
        "additionalFields": {}
      },
      "type": "n8n-nodes-base.telegram",
      "typeVersion": 1.2,
      "position": [432, 208],
      "name": "Send a text message1",
      "webhookId": "<REDACTED>",
      "credentials": { "telegramApi": { "id": "<REDACTED>", "name": "Telegram - AC Lab Bot" } }
    }
  ],
  "connections": {
    "Telegram Trigger": { "main": [[{ "node": "If", "type": "main", "index": 0 }]] },
    "If": {
      "main": [
        [{ "node": "Edit Fields", "type": "main", "index": 0 }],
        [{ "node": "Send a text message1", "type": "main", "index": 0 }]
      ]
    },
    "Edit Fields": { "main": [[{ "node": "Create a record", "type": "main", "index": 0 }]] },
    "Create a record": { "main": [[{ "node": "Send a text message", "type": "main", "index": 0 }]] }
  },
  "settings": { "executionOrder": "v1" }
}
```

---

## 3. Source payload (Telegram Trigger output, execution #1233)

```json
{
  "update_id": 613927269,
  "message": {
    "message_id": 6,
    "from": {
      "id": 8942534498,
      "is_bot": false,
      "first_name": "Nicolo",
      "language_code": "en"
    },
    "chat": {
      "id": 8942534498,
      "first_name": "Nicolo",
      "type": "private"
    },
    "date": 1793715060,
    "text": "😁"
  }
}
```

Note: the sender has no Telegram @username, so `message.from.username` is absent from the payload.

### Intermediate step (Edit Fields output)

```json
{
  "Name": "Nicolo",
  "username": null,
  "message": "😁",
  "Chat ID": "8942534498",
  "Timestamp": "\t2026-11-03T15:11:00.000+01:00"
}
```

---

## 4. Destination record (Airtable node output, execution #1233)

```json
{
  "id": "recVoFEyYAzOFLAJg",
  "createdTime": "2026-11-03T14:11:02.000Z",
  "fields": {
    "Chat ID": "8942534498",
    "Message": "😁",
    "Name": "Nicolo",
    "Timestamp": "2026-11-03T14:11:00.000Z"
  }
}
```

Second record from an earlier run (execution #1222, before the If guard was added):
`reckrnA8av28TVkzU` → Name `Nicolo`, Message `Hi! Id like to order some bread`, Chat ID `8942534498`, Timestamp `2026-11-03T14:04:28.000Z`.

**Trace check:** `message.date` 1793715060 = 2026-11-03 15:11:00 Berlin = 14:11:00 UTC → matches the Airtable Timestamp. Chat ID and Name match the payload. The customer received the confirmation "Thanks for reaching out! We will get back to you shortly" (Telegram message_id 7).

---

## 5. Mapping table

| Source (Telegram) | Edit Fields field | Destination (Airtable) | Transformation | Why |
|---|---|---|---|---|
| `message.from.first_name` | `Name` | Name (Single line text) | none | Who placed the order; always present in a private chat |
| `message.from.username` | `username` | Username (Single line text) | fallback to `''` if missing | Many Telegram users have no @username; the field must not break the run |
| `message.text` | `message` | Message (Long text) | none | The order itself |
| `message.chat.id` | `Chat ID` | Chat ID (Single line text) | number → string via `String()` | Telegram sends a number, the Airtable column is text; also the key to reply to the customer |
| `message.date` | `Timestamp` | Timestamp (Date, include time) | Unix seconds → ISO date via `DateTime.fromSeconds().toISO()` | Airtable needs a real date, not a raw Unix number |

The Airtable column names (`Username`, `Message`) differ in case from the Edit Fields names (`username`, `message`). This works because the Airtable node uses **Map Each Column Manually**; **Map Automatically** would leave those two columns empty.

---

## 6. Which field breaks first, and how to detect it

**Most likely to break: `message.text`.**
It only exists for plain text messages. A photo, sticker, voice note or location arrives without `text` (a photo carries `caption` instead), and an edited message arrives as `edited_message` instead of `message`. In those cases the Message column is written empty, or the If node fails on strict type validation because it compares `undefined` to a string.

`message.from.username` is the second weak point: it is missing for any user without a @username, so it relies on the fallback.

**How to detect it:**
- **Guard:** an If node that checks `{{ $json.message.text }}` **is not empty** and routes everything else to a reply asking for a text message
- **Executions panel:** failed or unexpected runs show which node broke and with what input
- **Destination check:** an Airtable view filtered on `Message is empty` surfaces records that slipped through
- **Error workflow:** connect an error-alert workflow (Settings → Error Workflow) so a failed run sends a notification instead of failing silently

---

## Security and data notes

- Telegram and Airtable tokens are stored only in n8n credentials, never in nodes or in this file
- The Airtable token is limited to `data.records:read`, `data.records:write`, `schema.bases:read` and the `Order Tracker` base only
- Name, username and chat ID are personal data and Airtable stores data in the US: real customer data would need a GDPR check with the client first
