# CHORUS Architecture ▷⦾◁

---

## 1. Physical Layer — *Chronome*
**Purpose:** provide rhythm, discovery, and packet exchange between nodes.

- Transport: UDP discovery, TCP/QUIC data channels.
- Message format: TLV (type-length-value) envelopes with Ed25519 signatures.
- Ports: 6000–6010 reserved for Chorus LAN experiments.
- Log format: JSONL, human-readable, tail-friendly.

Success = deterministic packet round-trips with <25 ms mean latency and <0.1 % loss.

---

## 2. Semantic Layer — *Alembic*
**Purpose:** transform, compress, and preserve curvature in embedding space.

- Built on Rust + Candle + CUDA (Gemma embeddings).  
- Evaluates intra-cluster vs cross-cluster similarity.  
- Exposes Matryoshka truncation (768 → 384 → 256 d).  
- Produces vectors with ‖v‖ = 1, Δ < 1e-6 determinism.

Success = curvature restored (Δ ≥ 0.15, intra ≥ 0.6, cross ≤ 0.35).

---

## 3. Perceptual Layer — *Sensorium*
**Purpose:** act as the I/O and environment interface.

- Captures user input, sensory streams, external APIs.  
- Converts stimuli into structured packets for Alembic.  
- Later stages may integrate gamepad, camera, or network sensors.

---

## 4. Memory Plane (in progress - mnematode)
Shared, dimensionality-aware store for embeddings, code, and sensor data.

- Append-only, content-addressed (BLAKE3).  
- Linked manifests define artefact lineage.  
- Acts as Chorus’s collective long-term memory.

---

## 5. Identity and Provenance

Each node maintains:

```json
{
  "node_id": "hash(pubkey)",
  "device": "jetson-orin",
  "dtype": "fp16",
  "commit": "abc123",
  "uptime": "2025-10-04T10:00Z"
}
