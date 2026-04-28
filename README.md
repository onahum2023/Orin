# Orin (v0.1)

Orin is a Docker-first local web app with:

- FastAPI backend
- file-based thread storage (`/Chats/threads/*.json`)
- append-only message logs (`/Chats/messages/*.jsonl`)
- hybrid learning review output queue (`/Chats/learning/learning_queue.jsonl`)
- LLM-based training need synthesis output (`/Chats/training/*.json`)
