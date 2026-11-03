# Lab proof: Telegram Inbox for Notion

## Purpose

I use a Telegram bot to capture project ideas and learning resources without opening Notion and copying information manually. I can send text or a voice message. n8n normalizes the message, classifies it with AI, and creates a record in the appropriate existing Notion database. Project ideas are viewed as a Kanban board; Content is viewed as a table. Telegram confirms which database received the entry.

Technical behavior: one message from the configured private Telegram chat is normalized and routed to Notion Projektideen, Notion Content, or a Telegram clarification reply.

## Artifacts

- [Workflow export](<labwork (1).json>) — nodes, mappings, and connections.
- [Telegram source and confirmation](<Screenshot 2026-11-03 184134.png>) — a submitted YouTube link and the reply “In Content gespeichert”.

The export is the reference for the implementation below. The screenshot shows the source and response, but does not independently prove the destination record contents.

## Implemented workflow

```text
Telegram Trigger
  -> If von mir: check configured chat ID
     -> false: Telegram rejection reply
     -> true: If Sprachnachricht
        -> voice: Get a file -> Transcribe a recording -> Edit Fields
        -> text: Edit Fields
           -> Text Classifier + OpenAI Chat Model
              -> Projektideen: Notion Create -> Telegram confirmation
              -> Content: Notion Create -> Telegram confirmation
              -> Sonstiges: Telegram clarification
```

Both message branches feed one shared Edit Fields node. The classifier uses gpt-4o-mini, with the Responses API disabled in this export. It classifies natural language without requiring command prefixes.

- **Projektideen:** personal ideas and plans to develop or implement.
- **Content:** articles, videos, books, podcasts, and learning resources to save.
- **Sonstiges:** messages fitting neither category. Telegram replies: “Bitte sende mir nur Content oder Projektideen.”

## Input and transformation

Telegram text arrives in `message.text`. Voice messages provide `message.voice.file_id`; the file is retrieved and transcribed into `text`.

Edit Fields creates three string fields and excludes other input fields:

```javascript
inputText: {{ $json.text ?? $json.message?.text ?? '' }}
receivedAt: {{ $now.toISO() }}
chatId: {{ String($('Telegram Trigger').item.json.message.chat.id) }}
```

The classifier reads `{{ $json.inputText }}` for either message type. `receivedAt` is the normalization time, not the original Telegram send time.

The initial guard compares `message.chat.id` with the configured private chat ID. It restricts the chat; it is not a separate sender-ID check for a shared group.

## Destination mapping in the export

Both Notion nodes use Database Page Create to add a record to an existing database.

### Projektideen

- Title: `{{ $('Edit Fields').item.json.inputText }}` — stores the full normalized text.
- Telegram confirms “unter Projektideen gespeichert” after the create node succeeds.

### Content

- Link (URL): `{{ $json.inputText }}`.
- Eingangsdatum (date): `{{ $now }}` at the destination node.
- Telegram confirms “In Content gespeichert” after the create node succeeds.

**Current limitation:** Content expects the entire input to be a valid URL. The export does not yet extract a URL from mixed text, populate a Content title or original-text field, or assign a subtype such as Video or Book. General text or spoken recommendations need an improved mapping before they can be reliably stored in this branch. No automatic rewriting step is present.

Both confirmation nodes retrieve the chat ID from the Telegram Trigger because Notion returns the created record rather than the original Telegram payload. The exported confirmations name the database but do not include the created record URL.

## Evidence and verification

The attached screenshot shows a submitted YouTube URL and a Content confirmation. It also contains an earlier generic reply, so it does not establish exactly-once processing.

The following evidence still needs to be added and linked here before submission:

- **Source payload:** one real Telegram Trigger JSON item, with personal identifiers redacted consistently.
- **Execution trace:** a successful run showing the trigger, Edit Fields output, classifier route, Notion result, and Telegram response.
- **Destination record:** a Notion screenshot or sanitized record JSON matching the same source event.
- **Voice-path test:** one recording-to-transcript-to-Notion execution, since the workflow includes that branch.

Use a distinctive test message and compare its text or URL across the source, execution, and destination. The export proves configuration, not successful runtime execution. Runtime tests have not been independently executed from this repository.

## Risks and detection

- **Most brittle mapping:** Content maps all of `inputText` to a URL property. A sentence containing a link or a spoken recommendation is not a standalone URL. Test a bare link and a text recommendation, inspect the Notion result, and compare destination values. Store original text separately and extract an optional URL to support both cases.
- **AI classification:** ambiguous messages can be routed incorrectly. Compare the selected route with the input and inspect the destination database. Sonstiges provides a clarification path but cannot guarantee correct classification.
- **Duplicates:** rerunning an execution may create another record; the current export has no message-ID deduplication.
- **Empty text:** unsupported Telegram message types can normalize to an empty string. An explicit non-empty-input guard would improve handling.

Telegram, transcription, chat-model, and Notion credentials must be configured in n8n. Both databases must be shared with the Notion integration. Review exports and evidence for secrets and personal data before committing.

## Assignment scope

`instructions.md` allows a chosen source and destination, so this workflow uses Telegram and Notion. `rubric.md` still refers to Discord and Google Sheets; mention this discrepancy in the submission PR.
