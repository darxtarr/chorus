# 🧩 Mnematode — The Memory Daemon

## Role
A Mnematode sits between clients (Cortex / Ganglia) and heavier SLMs, acting as an event broker + short-term memory cell.

| Function | Description |
|-----------|-------------|
| **Queueing** | Receives Chronome packets, stores them until an SLM is free. |
| **Deduplication** | Coalesces repeated or near-duplicate queries before dispatch. |
| **Ephemeral State** | Keeps 5-10 min of context per conversation in a ring buffer. |
| **Caching** | Stores last N embedding → result pairs for instant recall. |
| **Telemetry** | Exposes latency, queue depth, hit/miss rate. |

## Implementation sketch
```bash
mnematode --listen :4520 --broker redis://127.0.0.1:6379 \
  --memory-window 10m --max-queue 1024


Internals:

Client  →  [Chronome-In]  →  Mnematode  →  [SLM Queue]
SLM Out →  [Chronome-Out] →  Client


Each message:

{
  "id":"req-9823",
  "source":"ganglion-01",
  "intent":"embed:document",
  "payload":"…",
  "timestamp":1696781234,
  "hash":"sha256…"
}


Mnematode duties:

Hash check → if cached, respond immediately.

Otherwise enqueue for SLM.

After completion, cache result + timestamp.

Evict by LRU or TTL.

Why not let the SLM do it?

Because Mnematodes are cheap, distributable, and disposable.
They keep temporal coherence without forcing the expensive model to persist memory.

Deployment

Runs on same SBC class as Reflex Daemons.

Requires only 50–100 MB RAM.

Communicates via Chronome or Redis Streams.

One Ganglion can orchestrate several Mnematodes for load-balancing.

“The Reflex acts; the Mnematode remembers why.”

Mnematode is the most on-brand pun we’ve coined yet: half memory, half worm, all recursion.
That single word nails the missing piece — the temporal buffer that neither the Reflexes nor the Ganglia should waste cycles on. Let’s canonize it:
