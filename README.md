# Kleindeutschland: Telegram-to-Airtable German Vocabulary Automation

A streamlined automation pipeline that captures free-form German language notes, expressions, and vocabulary sent via a custom Telegram bot (**Kleindeutschland**), parses and normalizes them using AI, and automatically logs or updates them in an Airtable vocabulary database.

---

## 1. Technical Architecture

The workflow receives unstructured user messages, transforms the content using an LLM, handles deduplication, and routes the data to Airtable.

```
+------------------+     +---------------+     +-----------------+
| Telegram Trigger | --> |  Edit Fields  | --> | OpenAI (Model)  |
+------------------+     +---------------+     +-----------------+
                                                        │
                                                        ▼
+------------------+     +---------------+     +-----------------+
| Telegram Reply   | <-- | Airtable Node | <-- | If / Search     |
| (Confirmation)   |     | (Create/Upd)  |     | (Deduplication) |
+------------------+     +---------------+     +-----------------+
```

### Node Sequence & Execution Flow
1. **Telegram Trigger**: Listens for incoming chat messages sent to the bot.
2. **Edit Fields**: Extracts raw message content and normalizes the message timestamp (`ISO Date`).
3. **Message a model (OpenAI `gpt-4o-mini`)**: Processes free-form German text using structured system instructions to extract:
   - Canonical dictionary form (`word`)
   - Short English translation (`meaning`)
   - Formatted usage examples (`example`: Original, Formal, Informal)
   - Source reference (`source`)
4. **Edit Fields1 (JSON Parsing)**: Parses the model's output and combines fields into a standardized structure.
5. **Search records (Airtable)**: Queries Airtable to check whether the word/expression already exists (`LOWER({Word})`).
6. **If (Conditional Router)**:
   - **True (Record Exists)**: Appends new examples to the existing record's `Expression` field via an **Update record** node.
   - **False (New Word)**: Inserts a new entry using a **Create a record** node.
7. **Send a text message**: Sends a confirmation response back to the user on Telegram.

---

## 2. Service Configuration

Create the service resources and credentials below before importing or running the n8n workflow. Keep tokens in n8n credentials; never add them to this repository or workflow screenshots.

### Create the base and the table
1. Sign in to [Airtable](https://airtable.com).
2. Select **Create** → **Start from scratch**.
3. Name the base `Kleindeutschland`.
4. Rename the default table to `Vocabulary`.
5. Configure the table fields to match the field names and types expected by the workflow.

### Create the Airtable token (in Airtable)
1. Go to [airtable.com/create/tokens](https://airtable.com/create/tokens) and select **Create new token**.
2. Give the token a descriptive name, such as `n8n-kleindeutschland`.
3. Add these scopes:

   | Scope | Purpose |
   |---|---|
   | `data.records:read` | Search existing vocabulary records |
   | `data.records:write` | Create and update vocabulary records |
   | `schema.bases:read` | Let n8n list the base, table, and fields |

4. Under **Access**, select **Add a base** and grant access to the `Kleindeutschland` base.
5. Create the token and copy it immediately; Airtable displays it only once.

### Create the bot (in Telegram)
1. In Telegram, search for **@BotFather**, open it, and select **Start**.
2. Send `/newbot`.
3. Choose a display name, such as `Kleindeutschland`.
4. Choose a unique username ending in `bot`, such as `Kleindeutschland_bot` (if available).
5. Copy the bot link and token from BotFather. The token grants control of the bot, so do not share or commit it.

### Create the Telegram credential (in n8n)
1. In n8n, select **+** → **Credential** and search for **Telegram API**.
2. Paste the BotFather token into **Access Token**.
3. Name the credential, for example `Telegram – Kleindeutschland`, and save it.
4. If your browser asks whether to save the token as a password, select **No**.

---

## 3. Repository Files

- `Kleindeutschland.json`: The complete n8n workflow export file containing node configurations, expressions, and AI system prompts.
- `lab_proof.md`: Detailed test run proof containing trigger inputs, transformed payloads, destination Airtable record schemas, and field mapping tables.
