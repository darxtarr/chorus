# Micro-Optimization Playbook v1

**Status:** Reference document for future optimization sessions
**Author:** Opus (2025-12-06)
**Scope:** Foundation code in Chi, Solidus, Reflex

---

## Philosophy

1. **Prove it works first.** Correctness before speed.
2. **Measure before optimizing.** Intuition lies; profilers don't.
3. **Focus on foundations.** 10% improvement in TLV framing multiplies across every message.
4. **Understand what's actually happening.** Read the assembly. Know the syscalls.

---

## Pre-Optimization Checklist

Before any optimization session:

- [ ] **Identify hot path.** Use `perf record` / `perf report` or `cargo flamegraph`.
- [ ] **Establish baseline.** Measure current performance with realistic workload.
- [ ] **Understand the code.** Read the implementation, not just the interface.
- [ ] **Check compiler output.** Use `cargo asm` or `objdump -d` on release builds.
- [ ] **Know the target.** x86-64? ARM (Orin)? Cache line size? Register count?

---

## Hot Path Candidates

### 1. Chi: TLV Framing (`chi_lib_tlv_framing.rs`)

**Why it matters:** Every bubble crosses this code path.

**Current issues:**

| Function | Issue | Impact |
|----------|-------|--------|
| `encode_frame` | Allocates `Vec` per call | Memory pressure, cache misses |
| `decode_frame` | 3 `read_exact()` calls | 3 syscalls minimum (without BufReader) |
| `write_frame` | Calls `encode_frame` + `write_all` | Double handling, intermediate allocation |
| `write_frame` | `flush()` after every frame | Forces TCP packet, kills Nagle batching |

**Optimization experiments:**

```
Experiment TLV-1: Coalesce header reads
  Before: read_exact(type, 1) + read_exact(len, 4) + read_exact(payload, N)
  After:  read_exact(header, 5) + read_exact(payload, N)
  Measure: syscall count via strace, latency via criterion

Experiment TLV-2: Direct write (no intermediate Vec)
  Before: encode_frame() -> Vec -> write_all(Vec)
  After:  write_all(header, 5) + write_all(payload, N)
  Measure: allocations via DHAT, throughput via criterion

Experiment TLV-3: Caller-controlled flush
  Before: flush() after every frame
  After:  Remove flush(), let caller batch
  Measure: TCP packet count via tcpdump, throughput

Experiment TLV-4: Zero-copy decode
  Before: Allocate Vec<u8> for payload
  After:  Return reference into caller-provided buffer
  Measure: Allocation count, cache behavior

Experiment TLV-5: Vectored I/O
  Before: Two write_all() calls
  After:  writev() with header + payload iovecs
  Measure: syscall count, throughput
```

**Data needed:**
- `perf stat` on encode/decode loops
- `strace -c` for syscall breakdown
- `criterion` benchmarks with varying payload sizes
- `cargo asm chi::chi_lib_tlv_framing::encode_frame`

---

### 2. Solidus: B-Spline Evaluation (not yet implemented)

**Why it matters:** Called O(samples × glyphs × contours) during tessellation.

**De Boor's algorithm considerations:**

```rust
// Naive: Recursion with repeated computation
fn de_boor_naive(k: usize, t: f64, knots: &[f64], points: &[Point2]) -> Point2 {
    // Recursive calls recompute intermediate values
}

// Better: Iterative with workspace
fn de_boor_iterative(t: f64, knots: &[f64], points: &[Point2], workspace: &mut [Point2]) -> Point2 {
    // Reuse workspace to avoid allocations
    // Compute bottom-up, not top-down
}
```

**Optimization experiments:**

```
Experiment BSPL-1: Iterative vs recursive
  Measure: Cycle count per evaluation, cache misses

Experiment BSPL-2: Workspace reuse
  Before: Allocate intermediate arrays per call
  After:  Caller provides workspace, reuse across calls
  Measure: Allocation count, throughput

Experiment BSPL-3: SIMD for point operations
  Before: Scalar Point2 { x: f64, y: f64 }
  After:  __m128d for paired x,y operations
  Measure: Throughput on large glyph sets

Experiment BSPL-4: Precompute basis functions
  For fixed sample counts, basis functions are constant
  Precompute and store, multiply at runtime
  Measure: Memory vs compute tradeoff
```

**Data needed:**
- Representative glyph complexity distribution
- Sample count requirements for visual quality
- Cache behavior on Orin ARM vs x86-64

---

### 3. Reflex: Inference Loop

**Why it matters:** Runs at 100ms-1s intervals per reflex, must be sub-microsecond.

**Current state:** Decision tree traversal, quantized int16 weights.

**Optimization experiments:**

```
Experiment RFX-1: Branch prediction hints
  Decision trees have predictable paths for common cases
  Use likely()/unlikely() or profile-guided optimization
  Measure: Branch misprediction rate via perf

Experiment RFX-2: Tree flattening
  Before: Pointer-chasing through tree nodes
  After:  Flat array with computed indices
  Measure: Cache misses, cycle count

Experiment RFX-3: Feature normalization cost
  Before: Normalize features at inference time
  After:  Pre-normalize at telemetry collection
  Measure: Total path latency

Experiment RFX-4: SIMD feature comparison
  Multiple thresholds can be compared in parallel
  Measure: Throughput on batched inference
```

**Data needed:**
- Typical tree depth and branching factor
- Feature vector size across reflexes
- Inference frequency under load

---

### 4. Cross-Cutting: Time Functions

**Why it matters:** Timestamps everywhere (telemetry, provenance, logging).

**Common mistake:** Using `SystemTime::now()` when `Instant::now()` suffices.

```rust
// Expensive: System clock (may syscall, handles timezone)
let t = std::time::SystemTime::now();

// Cheap: Monotonic clock (often just rdtsc or clock_gettime(MONOTONIC))
let t = std::time::Instant::now();

// Cheapest: Coarse monotonic (lower resolution, fewer syscalls)
// Rust doesn't expose this directly, but libc does:
// clock_gettime(CLOCK_MONOTONIC_COARSE, &ts)
```

**Audit:** Grep for `SystemTime::now()` and evaluate if `Instant::now()` suffices.

---

### 5. Cross-Cutting: Allocations

**Patterns to watch:**

```rust
// Bad: Allocate in hot loop
for msg in messages {
    let frame = encode_frame(msg);  // Vec allocation
    send(frame);
}

// Better: Reuse buffer
let mut buffer = Vec::with_capacity(MAX_FRAME_SIZE);
for msg in messages {
    buffer.clear();
    encode_frame_into(msg, &mut buffer);  // Reuse allocation
    send(&buffer);
}

// Best: Avoid heap entirely for small frames
let mut buffer = [0u8; 256];  // Stack allocation
for msg in messages.filter(|m| m.len() < 256) {
    let n = encode_frame_into(msg, &mut buffer);
    send(&buffer[..n]);
}
```

**Tools:**
- `DHAT` (via valgrind or cargo-dhat) for allocation profiling
- `heaptrack` for allocation over time
- `#[global_allocator]` with counting allocator for coarse metrics

---

### 6. Cross-Cutting: Cache Behavior

**Key numbers (x86-64):**
- L1 cache line: 64 bytes
- L1D size: 32KB per core
- L2 size: 512KB per core
- L3 size: varies (96MB on 7800X3D)

**Struct packing:**

```rust
// Bad: 24 bytes due to padding
struct Message {
    msg_type: u8,      // 1 byte + 7 padding
    length: u64,       // 8 bytes
    flags: u8,         // 1 byte + 7 padding
}

// Better: 10 bytes, fits more per cache line
#[repr(C, packed)]
struct Message {
    msg_type: u8,
    flags: u8,
    length: u64,
}

// Or reorder fields
struct Message {
    length: u64,       // 8 bytes
    msg_type: u8,      // 1 byte
    flags: u8,         // 1 byte + 6 padding
}  // 16 bytes total
```

**Tools:**
- `perf stat -e cache-misses,cache-references`
- `pahole` for struct layout analysis
- `cachegrind` for detailed cache simulation

---

## Optimization Session Template

```markdown
# Optimization Session: [Component]

## Target
- Function: `module::function_name`
- Location: `path/to/file.rs:line`
- Why: [Why this is a hot path]

## Baseline
- Workload: [Description of test workload]
- Current performance: [Metric: value]
- Measurement method: [Tool used]

## Hypothesis
[What we think is slow and why]

## Experiment
[Specific change to try]

## Results
- Before: [Metric: value]
- After: [Metric: value]
- Delta: [Percentage change]

## Analysis
[Why did it work / not work?]

## Decision
[ ] Ship the change
[ ] Need more investigation
[ ] Not worth the complexity
[ ] Blocked on [something]

## Assembly Notes
[Relevant observations from cargo asm / objdump]
```

---

## Reference: Useful Commands

```bash
# Profile CPU usage
perf record -g ./target/release/spirit
perf report

# Count syscalls
strace -c ./target/release/hello-bubble

# Generate flamegraph
cargo install flamegraph
cargo flamegraph --bin spirit

# View assembly for function
cargo install cargo-show-asm
cargo asm chi::chi_lib_tlv_framing::encode_frame

# Allocation profiling
cargo install cargo-dhat
cargo dhat --bin spirit

# Cache behavior
valgrind --tool=cachegrind ./target/release/spirit

# Struct layout
pahole ./target/release/libchi.rlib

# Benchmark (requires criterion in dev-deps)
cargo bench

# Check what compiler optimizes
RUSTFLAGS="-C opt-level=3 -C target-cpu=native" cargo build --release
objdump -d target/release/spirit | less
```

---

## Reference: musl Patterns

When in doubt, see how musl does it:

- **String operations:** https://git.musl-libc.org/cgit/musl/tree/src/string
- **Memory operations:** https://git.musl-libc.org/cgit/musl/tree/src/string/memcpy.c
- **Time functions:** https://git.musl-libc.org/cgit/musl/tree/src/time
- **Math functions:** https://git.musl-libc.org/cgit/musl/tree/src/math

Key insight: musl prioritizes correctness and simplicity over micro-optimization. When they *do* optimize, it's surgical and well-documented.

---

## Version History

- **v1 (2025-12-06):** Initial playbook. TLV, B-spline, Reflex, cross-cutting concerns.
