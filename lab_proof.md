# Bakery order intake

The bakery takes orders through a Telegram bot because customers already use it to contact the shop. The counter team uses the Notion database **Bakery Order Tracker** to review incoming orders in one place. With each order in its own row, staff can see who sent it and when it arrived without copying messages out of Telegram.

When a customer sends a text message to the bot, n8n maps the sender, chat ID, message, and send time into one new Notion database page.

## Workflow and live execution

[Workflow export](./bakery-orders-workflow.json)

The export has no credential bindings or pinned input and uses a placeholder for the Notion database ID. Its node IDs were regenerated for sharing.

The published workflow received the order below and completed successfully in 687 ms. The execution trace showed one item from Telegram Trigger to Edit Fields and one item from Edit Fields to Create a database page. The Notion node returned one created page, and I verified the matching row in Bakery Order Tracker.

The excerpts below come from that same execution. Personal values and the chat ID are replaced consistently, and unrelated metadata and private links are omitted.

## Telegram Trigger output

```json
[
  {
    "message": {
      "from": { "first_name": "Sample", "username": "sample_customer" },
      "chat": { "id": 123456789, "type": "private" },
      "date": 1793837206,
      "text": "One rye loaf and three cinnamon rolls for Sunday, please."
    }
  }
]
```

## Notion output

```json
[
  {
    "property_name": "Sample",
    "property_username": "sample_customer",
    "property_message": "One rye loaf and three cinnamon rolls for Sunday, please.",
    "property_chat_id": "123456789",
    "property_timestamp": {
      "start": "2026-11-05T01:06:00.000+01:00",
      "end": null,
      "time_zone": null
    }
  }
]
```

## Field mapping

| Telegram field | Notion property | Transformation and reason |
| --- | --- | --- |
| `message.from.first_name` | Name (Title) | Copy the first name so the team can identify the sender at a glance. |
| `message.from.username` | Username (Text) | Use an empty string when the username is absent. Telegram usernames are optional. |
| `message.text` | Message (Text) | Copy the order verbatim so quantities and pickup details stay intact. |
| `message.chat.id` | Chat ID (Text) | Convert the number to a string because the chat ID is an identifier. |
| `message.date` | Timestamp (Date) | Convert Unix seconds to ISO 8601 UTC so the row reflects the message send time. |

The trigger timestamp converts to `2026-11-05T00:06:46.000Z` in Edit Fields. The Notion output returned the same local minute with a `+01:00` offset and seconds set to zero.

## Most fragile field

`message.text` is the most likely field to be missing because Telegram message updates can carry a photo or sticker without a text body. I would test with a photo without a caption and compare the trigger payload with the Message field in Edit Fields. A missing or blank value means the event has no usable order text. A filter before the Notion node could skip those events and prevent empty orders.
