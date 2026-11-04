# Lab proof — Telegram screams, Sheets remembers

**Student:** Martha Olukayode
**Integration:** Telegram → n8n Edit Fields (Set) → Airtable  
**Workflow:** `Bakery Order Tracker : Telegram -> Airtable` (n8n)
**Destination:** Airtable base `Bakery Order Tracker`, table `Orders`
**Track:** Telegram Trigger → Edit Fields (Set) → Airtable (Create a record) → Telegram (confirmation)

---

## 1. Scenario

**Order Intake Bot**
A small bakery receives customer orders through Telegram. The owner needs a central record of incoming requests rather than manually copying messages into a tracker. The workflow captures each incoming bot message, converts it into five consistent fields, and creates an Airtable record. This reduces manual entry and makes requests easier to inspect and follow up.

**Technical behavior:**
When a customer sends a message to the order bot, log their name, username, chat ID, message and timestamp as a new record in the Airtable `Orders` → `Messages` table, then reply to the customer on Telegram.

---

## 2. Workflow export and execution evidence

**Node path:** `Telegram Trigger (On message)` → `Edit Fields (Set)` → `Airtable (Record: Create)`.

**Workflow export:** `Bakery_Order_Tracker.json` — exported from n8n separately. Include the downloaded JSON file alongside this Markdown document in the submission repository. The export is not embedded here because its contents were not supplied for verification.

**Observed checks:**
- Telegram Trigger received a real `/start` event and then a customer order event.
- Edit Fields produced one item with the five mapped fields (verified in its output JSON).
- Airtable Create returned a new record ID and populated fields.
- The created record was visibly confirmed in the Airtable `Orders` table.
- A subsequent live end-to-end test was reported successful; its execution trace and destination-row screenshot should be attached as evidence if available.

**Evidence to include with submission:** Screenshot of the three connected nodes and successful execution; Telegram Trigger JSON; Edit Fields output; Airtable destination row. Use screenshots from the actual n8n/Airtable run, not simulated evidence.

## 3. Source event — Telegram Trigger JSON

The following is the actual customer-order payload observed during the lab:

```json
{
  "update_id": 943759239,
  "message": {
    "message_id": 2,
    "from": {
      "id": 8822322293,
      "is_bot": false,
      "first_name": "Martty",
      "last_name": "Olu",
      "language_code": "en"
    },
    "chat": {
      "id": 8822322293,
      "first_name": "Martty",
      "last_name": "Olu",
      "type": "private"
    },
    "date": 1793810612,
    "text": "Hello, I would like to order 5 chocolate cupcakes for tomorrow."
  }
}
```

## 4. Mapping and transformation

The Set node uses **Manual Mapping**, with these exact destination field names:

| Airtable field | n8n Set value | Transformation |
|---|---|---|
| `Sender` | `{{ $json.message.from.first_name }}` | Extract sender's first name |
| `Message text` | `{{ $json.message.text }}` | Extract message body |
| `Chat` | `{{ String($json.message.chat.id) }}` | Convert numeric chat ID to text |
| `Timestamp` | `{{ $now.toISO() }}` | Record processing time as an ISO timestamp |
| `Status` | `New` | Assign a fixed initial status |

The Set node's **Include Other Input Fields** option was off, so the result contained only the five destination fields. `Timestamp` represents the time n8n processed the item, not the Telegram message's original `date` field.

**Actual Set output from a separate mapping test (`hi`):**

```json
[
  {
    "Sender": "Martty",
    "Message text": "hi",
    "Chat": "8822322293",
    "Timestamp": "2026-11-04T18:33:59.419+01:00",
    "Status": "New"
  }
]
```

This Set output is from the `hi` test, not from the cupcake-order payload above.

## 5. Destination record — Airtable

The Airtable node used **Resource: Record**, **Operation: Create**, **Base: Bakery Order Tracker**, **Table: Orders**, and manual field mapping from the Set output.

The following values were verified in the Airtable Create response and in the Airtable `Orders` grid for the `hi` test:

| Airtable field | Observed value |
|---|---|
| `Sender` | `Martty` |
| `Message text` | `hi` |
| `Chat` | `8822322293` |
| `Status` | `New` |
| `Timestamp` | `2026-11-04T17:33:59.419Z` (API response; displayed in Airtable in local time) |

**Airtable record ID:** `rechQu9U8AFme45hj`  
**Record creation time (API):** `2026-11-04T18:40:49.000Z`

The Airtable response also contained an `Attachment Summary` field with an empty computed value; it was not part of the five mapped fields.

**Final live test:** The user confirmed that a new Telegram message successfully passed through the workflow and reached Airtable. The exact final order-row values were not provided here, so they are not invented in this proof.

## 6. Verification and operational risk

**Verification:** Compare the Telegram Trigger's `message.text` and sender with the Set output and the corresponding Airtable row. A successful Airtable `Create` response with a record ID, together with the matching row visible in `Orders`, proves that a destination record was written. Check the n8n execution trace to confirm that all three nodes succeeded in sequence.

**Operational risk:** The trigger may also receive commands such as `/start`, empty messages, or non-order messages. Without a guard, these could create irrelevant rows. A later improvement would be an IF condition to accept only nonempty order messages. Telegram credentials and Airtable tokens must remain in n8n's credential store, not in exported workflow text or screenshots.

---

**Submission reminder:** Place this `lab_proof.md` and the separately exported workflow JSON in the GitHub submission. Add real screenshots or links to the recorded execution and destination row if required by the course portal.

