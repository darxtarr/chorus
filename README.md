# CHORUS ▷⦾◁  
*Perceive → Distill → Resonate*

---

## Overview

**CHORUS** is an open, symbiotic intelligence stack — a living network of specialised model-nodes ("ganglia") that communicate through a lightweight, inspectable protocol rather than opaque APIs.

It is **not** a monolithic AI framework, but a distributed organism designed around three principles:

1. **Perception (Sensorium ▷)** — ingest signals from the world.  
2. **Transformation (Alembic ⦾)** — distil, compress, and recombine meaning.  
3. **Resonance (Chronome ◁)** — synchronise and maintain temporal coherence across nodes.

Together, they form a *chorus*: many minds, one rhythm.

---

## Why CHORUS Exists

Modern AI systems suffer from three systemic pain points:

| Pain | Description |
|------|--------------|
| **Context scarcity** | LLMs forget; embeddings flatten meaning. |
| **Bandwidth mismatch** | Text protocols (JSON, HTTP) are bottlenecks for model-to-model communication. |
| **Identity amnesia** | Models lack persistent memory, provenance, and continuity. |

CHORUS attacks these problems at the architectural level:  
by defining **inspectable, deterministic, semantically-aware** channels for AI-AI and AI-human communication.

---

## Guiding Philosophy

> *Boutique ≠ zero dependencies; boutique means you understand every line you depend on.*

- Prefer **small, well-shaped components** over frameworks.  
- Write what you can; include what has proven to endure.  
- Every binary should be composable, CLI-first, and human-inspectable (`tail -f`, `jq`, `xxd`).  
- Determinism beats cleverness.  
- Each node should be able to explain itself.

---

## Structure

| Layer | Repo | Role |
|-------|------|------|
| **Sensorium** | [`sefi`](https://github.com/darxtarr/sefi) | Input / perception layer |
| **Alembic** | [`vlc`](https://github.com/darxtarr/vlc) | Semantic distillation & compression |
| **Chronome** | *(new)* | Synchrony, transport, timing |

All three together compose **CHORUS**.

---

## Current Phase: Resonance Test

We are validating the **Chronome** transport layer by connecting two physical nodes (WSL or Jetson).  
Goals: discovery, signed packet exchange, stable latency, and deterministic replay.

Once the rhythm is stable, higher layers (Sensorium + Alembic) can begin to sing atop it.
