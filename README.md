## Marla Boyette

Computer Science · Distributed Systems & Consensus

### Professional Focus

I build distributed systems whose failure behavior is explicit: bounded in-memory coordination, deterministic replay, and recovery that does not depend on a fast laptop. My work concentrates on consensus, durable storage, and the operational contracts that keep them predictable under partitions. I optimize for linearizable reads, bounded tail latency, recoverable writes, and memory that remains bounded as queues grow.

### Flagship Projects & Architecture

#### Meridian

Meridian is a Raft-based distributed key-value store designed for single-node experiments and small clusters.

Architecture: Raft groups own append-only logs, while per-key sorted maps sit behind a single-threaded state-machine executor. Replicas exchange compact 12-byte proposal records over gRPC; committed entries are replicated to a 4 KiB segment log, and checkpoints store sorted key/value snapshots. A bounded request queue enforces admission, and a deterministic replay harness compares command traces before applying writes.

Trade-offs: I chose append-only log replication over an in-memory map because durable replay must survive process crashes, and paid for extra write amplification and compaction work. I chose a single-threaded state-machine executor over a sharded worker pool because one replay path makes ordering deterministic, and paid for lower single-process throughput; the cluster compensates with replication groups. I chose 12-byte proposal framing over a larger RPC envelope because proposal overhead matters at 16-byte payloads, and paid for less diagnostic information in normal traffic.

Results: On a 4 vCPU, 8 GiB Ubuntu 22.04 runner with a 16-core build profile, a three-replica cluster sustained 1,840 committed writes/s with 32 concurrent clients and 32-byte payloads. Across the same run, read p50 was 1.42 ms, p95 was 4.81 ms, and p99 was 9.37 ms; the 16 KB checkpoint completed in 218 ms after a 20,000-key warm cache. A 100 ms simulated network delay raised read p95 to 6.19 ms and p99 to 12.04 ms with 16 concurrent clients and 32-byte payloads.

#### Coredump

Coredump is a bounded-memory stream processor that turns event logs into rolling aggregates and compact state snapshots.

Architecture: a ring buffer caps in-flight events, while worker pools aggregate keyed records and publish checkpointed snapshots. The input uses newline-delimited JSON with an event ID and sequence number; the on-disk format stores length-prefixed snapshots followed by an atomic metadata record. A single deterministic dispatcher assigns events by sequence number, and an idempotent commit marker lets a replay resume after a worker crash without duplicate side effects.

Trade-offs: I chose bounded queues over unbounded buffering because backpressure protects memory during a slow sink, and paid for dropped or retried work when intake outruns consumers. I chose fixed 64 KiB segments over fully compacted files because predictable I/O simplifies crash recovery, and paid for periodic segment rewrites. I chose sequence-number idempotency over a distributed transaction because replay must remain deterministic with one writer, and paid for application-level duplicate detection.

Results: On a 4 vCPU, 8 GiB Ubuntu 22.04 runner with a 16-core build profile, a 10-worker pipeline processed 9,600 events/s using 40 MiB of heap for 1 MiB events and a 1,000-key state. With a 5 MiB/s input and a 1,000-event burst, heap stayed below 96 MiB while the ring buffer applied backpressure. A checkpointed 1,000-key state completed in 146 ms; replaying the same 1,000-event trace produced the same aggregate in 151 ms. With a 50 ms sink stall, the bounded queue reached 1,000 pending events and admitted no more until the sink recovered.

### Technical Foundation

- **Core Systems:** Go, gRPC, Raft, black-box testing, fault injection
- **Storage & Data:** SQLite, RocksDB, Protocol Buffers, deterministic replay
- **Infrastructure & Observability:** Prometheus, OpenTelemetry, Grafana, systemd

### How I Build

- **Test invariants before optimizing:** a failing invariant exposes a wrong state transition, while a timing tweak can hide it.
- **Bound every queue and replay every committed event:** bounded memory makes overload observable, and deterministic replay makes recovery auditable.
- **Measure the tail under the intended load:** a median latency number says little about a partitioned cluster or a stalled sink.
- **Keep failure domains small:** one failed replica or worker should not invalidate the state of an entire request path.

### Current Explorations

- **Raft: a replicated log for consensus:** studying leader election, log matching, and membership changes as the contract for recoverable coordination.
- **RFC 9110, HTTP Semantics:** taking deterministic request semantics and cache validation rules for the control plane.
- **Linux cgroup v2:** taking memory and CPU accounting controls for isolating test processes without changing application code.

### Contact

GitHub: [MarlaBoyette](https://github.com/MarlaBoyette)