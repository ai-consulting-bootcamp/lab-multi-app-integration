# LAB Proof | Telegram screams, Sheets remembers

## 1. Scenario

**Scenario (Order Intake Bot):**
A small bakery has no website order form, so customers message the shop's Telegram bot (`AC Lab Bot`) directly with what they want to order. As messages pile up in chat, staff lose track of who ordered what and when, causing customer requests to be missed. This n8n workflow automatically captures every incoming Telegram message, normalizes the sender and message details, and logs a structured record into an Airtable `Order Tracker` base. The bakery team can then sort, filter, and fulfill every customer order from a central table without manually copying and pasting from Telegram.

**One-sentence technical behavior:**

> When a client sends a message to the Telegram order bot, the workflow extracts and formats their first name, username, chat ID, message text, and timestamp, and writes them as a new record in the Airtable `Order Tracker` (`Messages` table).

---

## 2. Workflow Export (Sanitized JSON)

All sensitive credential IDs, webhook IDs, Airtable base/table IDs, and instance identifiers have been redacted.

```json
{
  "name": "Telegram screams, sheet remembers - LAB",
  "nodes": [
    {
      "parameters": {
        "updates": [
          "message"
        ],
        "additionalFields": {}
      },
      "type": "n8n-nodes-base.telegramTrigger",
      "typeVersion": 1.2,
      "position": [
        0,
        0
      ],
      "id": "1354677b-3528-4a73-95d6-0990a818240c",
      "name": "Telegram Trigger",
      "webhookId": "<REDACTED_WEBHOOK_ID>",
      "credentials": {
        "telegramApi": {
          "id": "<REDACTED_TELEGRAM_CREDENTIAL_ID>",
          "name": "Telegram – AC Lab Bot – Jedrzej"
        }
      }
    },
    {
      "parameters": {
        "assignments": {
          "assignments": [
            {
              "id": "61e5342e-63c8-4336-9c80-80120dde4bec",
              "name": "Name",
              "value": "={{ $json.message.from.first_name }}",
              "type": "string"
            },
            {
              "id": "1bb59aeb-c7bb-40ec-a97c-a4c4326782b9",
              "name": "Username",
              "value": "={{ $json.message.from.username || '' }}",
              "type": "string"
            },
            {
              "id": "15f3c8d9-a21e-4cc3-95b7-b17a2c672c31",
              "name": "Message",
              "value": "={{ $json.message.text }}",
              "type": "string"
            },
            {
              "id": "82aa23cf-8437-4d97-8321-0c79a4b708ba",
              "name": "Chat ID",
              "value": "={{ String($json.message.chat.id) }}",
              "type": "string"
            },
            {
              "id": "3ab53e4c-1d55-43ce-a613-60a36e010613",
              "name": "Timestamp",
              "value": "={{ DateTime.fromSeconds($json.message.date).toISO() }}",
              "type": "string"
            }
          ]
        },
        "options": {}
      },
      "type": "n8n-nodes-base.set",
      "typeVersion": 3.4,
      "position": [
        208,
        0
      ],
      "id": "b2359d85-305b-47aa-b009-6f9927033947",
      "name": "Edit Fields"
    },
    {
      "parameters": {
        "operation": "create",
        "base": {
          "__rl": true,
          "value": "<REDACTED_BASE_ID>",
          "mode": "list",
          "cachedResultName": "Order Tracker",
          "cachedResultUrl": "https://airtable.com/<REDACTED_BASE_ID>"
        },
        "table": {
          "__rl": true,
          "value": "<REDACTED_TABLE_ID>",
          "mode": "list",
          "cachedResultName": "Messages",
          "cachedResultUrl": "https://airtable.com/<REDACTED_BASE_ID>/<REDACTED_TABLE_ID>"
        },
        "columns": {
          "mappingMode": "defineBelow",
          "value": {
            "Name": "={{ $json.Name }}",
            "Username": "={{ $json.Username }}",
            "Message": "={{ $json.Message }}",
            "Timestamp": "={{ $json.Timestamp }}",
            "Chat ID": "={{ $json['Chat ID'] }}"
          },
          "matchingColumns": [],
          "schema": [
            {
              "id": "Name",
              "displayName": "Name",
              "required": false,
              "defaultMatch": false,
              "canBeUsedToMatch": true,
              "display": true,
              "type": "string",
              "readOnly": false,
              "removed": false
            },
            {
              "id": "Username",
              "displayName": "Username",
              "required": false,
              "defaultMatch": false,
              "canBeUsedToMatch": true,
              "display": true,
              "type": "string",
              "readOnly": false,
              "removed": false
            },
            {
              "id": "Message",
              "displayName": "Message",
              "required": false,
              "defaultMatch": false,
              "canBeUsedToMatch": true,
              "display": true,
              "type": "string",
              "readOnly": false,
              "removed": false
            },
            {
              "id": "Status",
              "displayName": "Status",
              "required": false,
              "defaultMatch": false,
              "canBeUsedToMatch": true,
              "display": true,
              "type": "options",
              "options": [
                {
                  "name": "Todo",
                  "value": "Todo"
                },
                {
                  "name": "In progress",
                  "value": "In progress"
                },
                {
                  "name": "Done",
                  "value": "Done"
                }
              ],
              "readOnly": false,
              "removed": true
            },
            {
              "id": "Chat ID",
              "displayName": "Chat ID",
              "required": false,
              "defaultMatch": false,
              "canBeUsedToMatch": true,
              "display": true,
              "type": "string",
              "readOnly": false,
              "removed": false
            },
            {
              "id": "Timestamp",
              "displayName": "Timestamp",
              "required": false,
              "defaultMatch": false,
              "canBeUsedToMatch": true,
              "display": true,
              "type": "dateTime",
              "readOnly": false,
              "removed": false
            }
          ],
          "attemptToConvertTypes": false,
          "convertFieldsToString": false
        },
        "options": {}
      },
      "type": "n8n-nodes-base.airtable",
      "typeVersion": 2.2,
      "position": [
        416,
        0
      ],
      "id": "103e3c66-82cc-4794-a7fa-5c1c32e322a4",
      "name": "Create a record",
      "credentials": {
        "airtableTokenApi": {
          "id": "<REDACTED_AIRTABLE_CREDENTIAL_ID>",
          "name": "Airtable-n8n_lab-Jedrzej"
        }
      }
    }
  ],
  "pinData": {},
  "connections": {
    "Telegram Trigger": {
      "main": [
        [
          {
            "node": "Edit Fields",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Edit Fields": {
      "main": [
        [
          {
            "node": "Create a record",
            "type": "main",
            "index": 0
          }
        ]
      ]
    }
  },
  "active": true,
  "settings": {
    "executionOrder": "v1",
    "binaryMode": "separate"
  },
  "versionId": "<REDACTED_VERSION_ID>",
  "meta": {
    "templateCredsSetupCompleted": true,
    "instanceId": "<REDACTED_INSTANCE_ID>"
  },
  "id": "<REDACTED_WORKFLOW_ID>",
  "tags": []
}
```

---

## 3. One Source Payload (Telegram Trigger Output)

Captured when sending `"I want to order 7 croissants."` to `AC Lab Bot` on **4 November 2026 at 15:08:46 CET**:

```json
[
  {
    "update_id": 849302116,
    "message": {
      "message_id": 6428,
      "from": {
        "id": 1567536641,
        "is_bot": false,
        "first_name": "Jędrzej",
        "last_name": "Czarnota",
        "username": "jedczar",
        "language_code": "en"
      },
      "chat": {
        "id": 1567536641,
        "first_name": "Jędrzej",
        "last_name": "Czarnota",
        "username": "jedczar",
        "type": "private"
      },
      "date": 1793801326,
      "text": "I want to order 7 croissants."
    }
  }
]
```

---

## 4. One Destination Record (Airtable Output)

### Airtable Node JSON Output

```json
[
  {
    "id": "rec6XkLm2PqR7vWzN",
    "createdTime": "2026-11-04T14:08:47.000Z",
    "fields": {
      "Name": "Jędrzej",
      "Username": "jedczar",
      "Message": "I want to order 7 croissants.",
      "Chat ID": "1567536641",
      "Timestamp": "2026-11-04T14:08:46.000Z"
    }
  }
]
```

### Airtable Grid View (`Order Tracker` → `Messages` Table, Row 6)

| # | Name | Username | Message | Status | Chat ID | Timestamp |
|---|---|---|---|---|---|---|
| 4 | Jędrzej | jedczar | Test message | | 1567536641 | 4/11/2026 14:50 |
| 5 | Jędrzej | jedczar | Test message | | 1567536641 | 4/11/2026 14:50 |
| **6** | **Jędrzej** | **jedczar** | **I want to order 7 croissants.** | | **1567536641** | **4/11/2026 15:08** |

---

## 5. Mapping Table

| Source (Telegram) | Destination (Airtable) | Transformation | Why |
|---|---|---|---|
| `message.from.first_name` | `Name` | None (`{{ $json.message.from.first_name }}` → `"Jędrzej"`) | Extracts the customer's display first name from the nested `from` object into a flat text field so staff know who placed the order. |
| `message.from.username` | `Username` | Fallback empty string (`{{ $json.message.from.username \|\| '' }}` → `"jedczar"`) | Telegram usernames are optional; if a customer has no `@username`, this prevents `undefined`/`null` values and safely passes an empty string to Airtable. |
| `message.text` | `Message` | None (`{{ $json.message.text }}` → `"I want to order 7 croissants."`) | Pulls the customer's raw order text out of the nested `message` payload into Airtable's `Long text` column. |
| `message.chat.id` | `Chat ID` | Number → String (`{{ String($json.message.chat.id) }}` → `"1567536641"`) | Telegram provides the chat identifier as a number (`1567536641`), whereas the Airtable `Chat ID` column is a `Single line text` field and requires a string. |
| `message.date` | `Timestamp` | Unix seconds → ISO-8601 (`{{ DateTime.fromSeconds($json.message.date).toISO() }}`) | Telegram timestamps are sent in Unix epoch seconds (`1793801326`), while Airtable's `Date` (with time) column expects an ISO-8601 date string (rendered in local CET time as `4/11/2026 15:08`). |

---

## 6. Fragility & Detection Analysis

### Which field is most likely to break when the source payload changes?

1. **`message.text` (Non-text messages):**
   * **Why it breaks:** In Telegram, `message.text` only exists when a user sends a standard text message. If a customer sends a photo of a pastry with a caption, a voice note, a sticker, a location, or a document, Telegram omits `message.text` entirely (using fields like `message.caption`, `message.photo`, or `message.voice` instead). In our current workflow, `Message` evaluates to `undefined`, creating a blank order row in Airtable.
   * **Edited messages (`edited_message`):** If a customer edits their order message in Telegram and the bot listens to edits, Telegram wraps the payload in `edited_message` instead of `message`, causing all `{{ $json.message.* }}` expressions (including `DateTime.fromSeconds($json.message.date)`) to fail.

### How would you detect and prevent it?

1. **Input Guard (`If` Node):** Add an **If** node immediately after the `Telegram Trigger` that checks whether `{{ $json.message.text }}` **is not empty** (and exists).
   * **True branch:** Proceed to `Edit Fields` → `Create a record`.
   * **False branch:** Route non-text messages either to a Telegram reply node telling the customer *"Please send your order as a text message"* or to a separate logging path.
2. **Fallback Expression for Captions:** Update the `Message` expression in `Edit Fields` to `{{ $json.message.text || $json.message.caption || '[Non-text message]' }}` so media captions are still captured and non-text events are clearly flagged in Airtable.
3. **Monitoring & Error Alerting:** Configure an n8n **Error Workflow** (under *Workflow Settings → Error Workflow*) to send an instant Slack/email/Telegram alert to the workflow owner whenever an execution throws an unhandled error, and periodically filter the Airtable `Messages` table for empty `Message` cells.