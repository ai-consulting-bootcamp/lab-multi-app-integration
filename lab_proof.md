# Use Case:
A small restaurant has no online booking system, so guests request a table by messaging the restaurant's Telegram bot. Staff copied these requests into a paper notebook between shifts, and some got lost or double-booked. The workflow writes every request as one entry in a Notion reservations database that the floor manager checks before each service. The chat ID lets staff reply with a confirmation, and the timestamp shows who asked first

When a guest sends a message to the reservations bot, log their name, username, chat ID, request and timestamp as a new entry in the Notion reservations database.


# Telegram Trigger working

[
  {
    "update_id": 833685366,
    "message": {
      "message_id": 1,
      "from": {
        "id": 5344754370,
        "is_bot": false,
        "first_name": "P",
        "last_name": "A",
        "username": "P_D_AAA",
        "language_code": "nl"
      },
      "chat": {
        "id": 5344754370,
        "first_name": "P",
        "last_name": "A",
        "username": "P_D_AAA",
        "type": "private"
      },
      "date": 1793893152,
      "text": "/start",
      "entities": [
        {
          "offset": 0,
          "length": 6,
          "type": "bot_command"
        }
      ]
    }
  }
]

# Succesfull Notion Create Database Run
[
  {
    "id": "40ffd5c2-7ef6-81ff-9851-e07856da675f",
    "name": "P",
    "url": "https://app.notion.com/p/P-40ffd5c27ef681ff9851e07856da675f",
    "property_chat_id": "5344754370",
    "property_timestamp": {
      "start": "2026-11-05T16:39:00.000+01:00",
      "end": null,
      "time_zone": null
    },
    "property_message": "/start",
    "property_telegram_first_name": "",
    "property_username": "P_D_AAA",
    "property_name": "P"
  }
]

# Succesful Run Trigger JSON Output

[
  {
    "update_id": 833685368,
    "message": {
      "message_id": 3,
      "from": {
        "id": 5344754370,
        "is_bot": false,
        "first_name": "P",
        "last_name": "A",
        "username": "P_D_AAA",
        "language_code": "nl"
      },
      "chat": {
        "id": 5344754370,
        "first_name": "P",
        "last_name": "A",
        "username": "P_D_AAA",
        "type": "private"
      },
      "date": 1793894762,
      "text": "I want to make a reservation"
    }
  }
]

# Succesful run Ex ID

Nov 5, 17:06:02
Succeeded in 1.02s | ID#2734

