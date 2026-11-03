## 1. Scenario
Scenario: Salon Booking Bot
A small hair salon gets appointment requests through many channels, and staff often miss messages while working with clients. Customers can now send their booking request (for example “Haircut on Friday at 3 pm”) to the salon’s Telegram bot. The workflow saves every request automatically as a new row in an Airtable booking tracker. The receptionist sees all requests in one table, sorted by time, and can confirm them without scrolling through chats. This saves time and makes sure no booking request gets lost.
One-sentence technical behavior:
When a customer sends a message to the salon bot, log their name, username, chat ID, message and timestamp as a new record in an Airtable booking tracker.

## 3. Source payload
[
  {
    "update_id": 380445067,
    "message": {
      "message_id": 2,
      "from": {
        "id": 5861331058,
        "is_bot": false,
        "first_name": "Sweet Hustle",
        "username": "xyaadb",
        "language_code": "en"
      },
      "chat": {
        "id": 5861331058,
        "first_name": "Sweet Hustle",
        "username": "xyaadb",
        "type": "private"
      },
      "date": 1793719158,
      "text": "Haircut on Friday at 3 pm"
    }
  }
]

## 4. Destination record

[
  
{
  
  
"id": 
"recn4cpODO3hYSFkg",
  
  
"createdTime": 
"2026-11-03T15:47:14.000Z",
  
  
"fields": 
{
  
  
  
"Message": 
"Haircut on Friday at 3 pm",
  
  
  
"Timestamp": 
"2026-11-03T15:19:18.000Z",
  
  
  
"Name": 
"Sweet Hustle",
  
  
  
"Username": 
"xyaadb",
  
  
  
"Chat ID": 
"5861331058"
  
  
}
  
}
]


# Lab Proof – Telegram → Airtable (Salon Booking Bot)

## 1. Scenario
A small hair salon gets appointment requests through many channels, and
staff often miss messages while working with clients. Customers can now
send their booking request (for example "Haircut on Friday at 3 pm") to
the salon's Telegram bot. The workflow saves every request automatically
as a new row in an Airtable booking tracker. The receptionist sees all
requests in one table, sorted by time, and can confirm them without
scrolling through chats. This saves time and makes sure no booking
request gets lost.

**Technical behavior:** When a customer sends a message to the salon bot,
log their name, username, chat ID, message and timestamp as a new record
in an Airtable booking tracker.

## 2. Workflow
`Telegram Trigger → Edit Fields (Set) → Airtable (Create a record)`

- Export: [salon_bot_workflow.json](salon_bot_workflow.json)
  (credential IDs removed; tokens are never included in n8n exports)
- Airtable base: Salon Bookings, table: Messages

## 3. Source payload (Telegram Trigger output)
```json

[
  
{
  
  
"update_id": 
380445068,
  
  
"message": 
{
  
  
  
"message_id": 
3,
  
  
  
"from": 
{
  
  
  
  
"id": 
5861331058,
  
  
  
  
"is_bot": 
false,
  
  
  
  
"first_name": 
"Sweet Hustle",
  
  
  
  
"username": 
"xyaadb",
  
  
  
  
"language_code": 
"en"
  
  
  
},
  
  
  
"chat": 
{
  
  
  
  
"id": 
5861331058,
  
  
  
  
"first_name": 
"Sweet Hustle",
  
  
  
  
"username": 
"xyaadb",
  
  
  
  
"type": 
"private"
  
  
  
},
  
  
  
"date": 
1793721243,
  
  
  
"text": 
"Color and cut on Monday at 10 am"
  
  
}
  
}
]
```

## 4. Destination record (Airtable node output)
```json

[
  
{
  
  
"id": 
"recHyDL6sy0dXtDJt",
  
  
"createdTime": 
"2026-11-03T15:54:04.000Z",
  
  
"fields": 
{
  
  
  
"Message": 
"Color and cut on Monday at 10 am",
  
  
  
"Timestamp": 
"2026-11-03T15:54:03.000Z",
  
  
  
"Name": 
"Sweet Hustle",
  
  
  
"Username": 
"xyaadb",
  
  
  
"Chat ID": 
"5861331058"
  
  
}
  
}
]
```
![Airtable rows](images/airtable_end_to_end.png)

## 5. Mapping table
| Source (Telegram) | Destination (Airtable) | Transformation | Why |
|---|---|---|---|
| `message.from.first_name` | Name | none | Shows who sent the booking |
| `message.from.username` | Username | fallback `\|\| ''` | Many users have no @username; avoids an undefined value |
| `message.text` | Message | none | The booking request itself |
| `message.chat.id` | Chat ID | `String(...)`, number → text | The column is text; the ID identifies the customer's chat |
| `message.date` | Timestamp | `DateTime.fromSeconds(...).toISO()`, Unix seconds → ISO date | Airtable's Date field needs a real date, not a raw number |

Note: in the Airtable node, `Chat ID` is referenced as `$json['Chat ID']`
because the field name contains a space.

## 6. Execution trace
Live test: a new message "Color and cut on Monday at 10 am" ran through
all 3 nodes (Succeeded in 732 ms) and created row 2 in Airtable.

![Execution trace](images/execution_trace.png)

Observation: the Executions view showed credential warnings on the
Telegram and Airtable nodes, but the run succeeded, the record was
created, and the Editor shows no issues. It is a display issue only.

## 7. Which field is most likely to break?
**`message.text`.** It only exists for text messages. If a customer
sends a photo, sticker or voice message, there is no `text`, so the
Message column stays empty. `message.from.username` is a similar risk,
but it is already handled with the `|| ''` fallback.

**How to detect it:** check Airtable for rows with an empty Message,
and inspect the Telegram Trigger output in the execution trace to see
which field was missing. **How to prevent it:** add an If node after the
trigger that only continues when `message.text` is not empty.

## 8. Security and data notes
- Tokens are stored only in n8n credentials, never in nodes,
  screenshots or this file.
- The Airtable token has least privilege: 3 scopes, 1 base.
- Name, username and chat ID are personal data; Airtable stores data
  in the US, so a real salon should check GDPR before using it.