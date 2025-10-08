# 🜂 CHORUS Reflex Architecture — Tiered Nervous System

**Date:** 2025-10-08  
**Author:** Mentat / Ulli  
**Purpose:** Define the multi-tier reflex ecosystem uniting Cortex, Ganglia, and Daemons.

## 1. System Overview
CHORUS models an artificial nervous system composed of three tiers:

| Tier | Example Hardware | Role | Typical Workload |
|------|------------------|------|------------------|
| **Cortex** | Workstation w/ RTX or similar | High-context reasoning & orchestration | planning, code gen, retrieval, simulation |
| **Ganglia** | Jetson / Coral / NUC | Specialized local intelligence (SLM / Gemma) | embedding, semantic search, RAG |
| **Daemons** | Pi Zero / ESP32 / small SBC | Deterministic reflex execution | sampling, PID tuning, compression, batching |

Data flows **down** as intent, **up** as telemetry.

[CORTEX] ←→ [GANGLIA] ←→ [DAEMONS]

## 2. Transport & Messaging
| Direction | Protocol | Purpose |
|------------|-----------|----------|
| Downstream | **Chronome** | control, model updates, coordination |
| Upstream | **Alembic** | telemetry, metrics, compressed data |
| Lateral | **Mesh-Bus** (UDP multicast / QUIC) | peer-to-peer coordination |

## 3. Training & Deployment
| Stage | Model | Environment | Duration |
|--------|--------|--------------|-----------|
| Design / heavy training | Foundation | Cortex | hours |
| Fine-tuning / specialization | SLM (Gemma-class) | Ganglion | minutes |
| Field deployment | `.reflex` binary | Daemon | seconds |

Ganglia periodically retrain Reflexes offline and push verified updates outward.

## 4. Advantages
- **Energy locality:** most decisions near the data source.  
- **Graceful degradation:** higher-tier loss ≠ system failure.  
- **Composable:** new nodes plug in, no retraining required.  
- **Heterogeneous compute:** each node contributes proportionally.  
- **Human legibility:** clear functional boundaries.

## 5. Deployment Topology Example
CORTEX-PC (RTX)
├─ ganglion-01 (Jetson Orin NX)
│ ├─ Provides: embedding-gemma (RAG)
│ └─ Controls:
│ • daemon-01 (Pi Zero – Chronome batch reflex)
│ • daemon-02 (Pi 3 – PID auto-tuner)
│ • daemon-03 (ESP32 – sensor sampler)
│
└─ ganglion-02 (NUC)
└─ Provides: speech recognition, local cache

## 6. Next Steps
1. Formalize **Chronome-Lite** protocol.  
2. Integrate **Gemma / Candle** inference on Ganglia.  
3. Build **`chorusctl`** for deployment & discovery.  
4. Benchmark reflex propagation latency and stability.  

**Guiding Principle:**  
> *Every node, from RTX to Pi Zero, contributes to collective intelligence in proportion to its power.*
