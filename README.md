# Orin (v0.1)

Orin is a Docker-first local web app with:

- FastAPI backend
- file-based thread storage (`/Chats/threads/*.json`)
- append-only message logs (`/Chats/messages/*.jsonl`)
- hybrid learning review output queue (`/Chats/learning/learning_queue.jsonl`)
- LLM-based training need synthesis output (`/Chats/training/*.json`)

Local-first file storage with optional LLM-backed responses.

## Run

```bash
cd orin
make up
```

Open: http://localhost:8000

## Chat Storage

Local folders (mounted into the container at `/Chats`):

- `Chats/threads`
- `Chats/messages`
- `Chats/learning`
- `Chats/training`

Thread file example (`Chats/threads/<thread_id>.json`):

```json
{
  "id": "thread_123abc",
  "title": "Organize my files on mac",
  "created_at": "2026-04-27T10:00:00Z",
  "updated_at": "2026-04-27T10:00:00Z",
  "status": "active"
}
```

Message log line example (`Chats/messages/<thread_id>.jsonl`):

```json
{"id":"msg_123abc","thread_id":"thread_123abc","role":"user","content":"Hello","created_at":"2026-04-27T10:00:00Z","model":null,"metadata":{}}
```

## Learning Review CLI

Run hybrid learning extraction:

```bash
docker compose exec web python -m app.learning.review
```

This reads `Chats/messages/*.jsonl` and appends moments to:

- `Chats/learning/learning_queue.jsonl`

Review behavior:

- always runs deterministic detection
- if `REVIEW_MODEL` is set, also runs LLM-based learning extraction
- if `REVIEW_MODEL` is missing, runs deterministic-only mode
- each moment includes evidence message IDs and source thread IDs

## Training Synthesis CLI

Run LLM-based training synthesis:

```bash
docker compose exec web python -m app.learning.synthesize
```

This reads `Chats/learning/learning_queue.jsonl` (and source messages from `Chats/messages/*.jsonl`) and writes one file per training need:

- `Chats/training/<training_need_id>.json`

Synthesis uses `SYNTHESIS_MODEL` and fails clearly if not configured.
Duplicate prevention skips creating a new training need when an existing `pending` or `approved` one has the same normalized title.

## Environment

Copy env template:

```bash
cp .env.example .env
```

Default:

- `CHAT_ROOT=/Chats`
- `OPENAI_API_KEY=` (required for chat replies, review, and synthesis)
- `CHAT_MODEL=` (required for assistant chat replies, for example `gpt-4o-mini`)
- `REVIEW_MODEL=` (optional; if unset, review runs deterministic-only)
- `SYNTHESIS_MODEL=` (required for training synthesis, use a stronger model)
- `LLM_MODEL=` (legacy fallback for chat model, optional)

Assistant replies use `CHAT_MODEL`.
Learning review uses `REVIEW_MODEL` when configured.
Training need synthesis uses `SYNTHESIS_MODEL`.

Flow:

- `Chats/messages/*.jsonl` -> `python -m app.learning.review` -> `Chats/learning/learning_queue.jsonl`
- `Chats/learning/learning_queue.jsonl` -> `python -m app.learning.synthesize` -> `Chats/training/*.json`

## Useful Commands

- `make up` - build and start the app
- `make down` - stop and remove containers
- `make logs` - tail app logs
- `make shell` - open a shell in the running app container
- `make reset-db` - clear local thread/message/learning/training files under `Chats/`
