# The Operating Systems Primer for the GenAI Engineer Targeting Top-Tier Companies

*A dense, industry-aligned walkthrough — from college fundamentals to the depth top-tier interviewers actually probe in 2024–2026.*

## TL;DR

- **The OS layer is the lowest-level mental model every other system implicitly assumes — a distributed system is processes + IPC + scheduling, a database is page tables + journaling + caches, an LLM inference server is virtual memory (PagedAttention) + scheduling (continuous batching) + DMA (pinned memory) + IPC (NCCL), and an HFT system is the OS with everything turned off and bypassed.** Master ~22 core topics with named implementations and real numbers and you'll thread interviews from Anthropic to Citadel.
- **The single highest-leverage update over textbook knowledge: Linux replaced CFS with EEVDF in kernel 6.6 (Oct 2023) and removed CFS entirely in 6.12 (Nov 2024). Naming EEVDF, vLLM's PagedAttention, GPUDirect RDMA's `nvidia-peermem`, io_uring, jemalloc vs tcmalloc, and DPDK kernel-bypass instantly separates you from candidates reciting 2015 textbooks.**
- **By vertical**: AI labs (Anthropic/OpenAI/DeepMind) probe inference batching + KV cache + multi-GPU collectives; data engineering (Databricks/Snowflake) tests Spark OS tuning + concurrency rounds + Delta Lake internals; HFT (Citadel/Jane Street/HRT) demands mechanical sympathy, kernel bypass, lock-free, NUMA, cache-line awareness; FAANG SWE covers classical fundamentals + concurrency + systems design.

---

## Key Findings

1. **Linux's scheduler IS now EEVDF, not CFS.** Per Linux kernel docs: *"The Linux kernel began transitioning to EEVDF in version 6.6 (as a new option in 2024), moving away from the earlier Completely Fair Scheduler (CFS)."* Per Virginia Tech course slides by Huaicheng Li: *"Phase 2 (Linux 6.12, Nov 2024): CFS code removed; EEVDF becomes the sole fair scheduler."*

2. **Anthropic's most reported system-design question is GPU inference batching.** Per Exponent's Anthropic guide, built from multiple recent candidates: *"Design an inference batching system. You have a single GPU that can process up to 100 inputs per batch… The inference batching question is the most commonly reported Anthropic system design prompt."*

3. **Databricks has a dedicated concurrency round.** Per interviewing.io's Databricks guide: *"Coding: Concurrency/Multithreading (1 hour). This round focuses on implementing programs that leverage multithreading for efficiency."*

4. **HFT firms drill mechanical sympathy.** Per techinterview.org's Citadel Securities guide: *"What works: explicit latency budget reasoning (microsecond level for critical paths), kernel bypass and CPU-pinning awareness, lock-free or wait-free data structures where contention matters, hardware-aware design (NUMA, cache hierarchies)."*

5. **eBPF is now production-critical at Meta/Google/Netflix.** Per the eBPF Foundation: *"Meta uses eBPF to process and load balance every packet coming into their data centers"* (Katran). *"Netflix uses eBPF for fleet-wide network observability."*

6. **GPUDirect RDMA failure mode is a top AI-infra interview signal.** Per Pingdo's GPUDirect deep dive: *"If [the `nvidia-peermem`] module is missing, NCCL will silently fallback to System Memory Relays, resulting in a 3-5x performance degradation in All-Reduce operations."*

---

## Details

### 0. Recommended Study Order / Roadmap

A 6–8 week plan, mapped to interview difficulty:

| Week | Focus | Primary Resource | Deliverable |
|---|---|---|---|
| 1 | Processes, threads, scheduling, syscalls | OSTEP Part I (Virtualization, Ch. 1–10) | Build a tiny shell in C |
| 2 | Synchronization, deadlock, IPC | OSTEP Part II (Concurrency, Ch. 25–34) | Producer-consumer with mutex+condvar, then lock-free SPSC ring |
| 3 | Virtual memory, paging, TLB, allocators | OSTEP Ch. 13–24; jemalloc paper | Write a slab/free-list allocator |
| 4 | File systems, journaling, COW, async I/O | OSTEP Part III; `io_uring` docs | Implement an `epoll`/`io_uring` echo server |
| 5 | Linux internals: cgroups, namespaces, eBPF, `/proc` | *Linux Kernel Development* (Love); Brendan Gregg's *BPF Performance Tools* | Build a mini-container in 200 lines of Go/Rust |
| 6 | CPU caches, MESI, NUMA, atomics, memory barriers | Drepper, *What Every Programmer Should Know About Memory*; *C++ Concurrency in Action* (Williams) | Microbench false sharing; show MESI in `perf c2c` |
| 7 | ML/AI infra OS layer: CUDA streams, pinned memory, NCCL, GPUDirect RDMA, PagedAttention | vLLM docs, NCCL Developer Guide, Meta RoCE paper, OpenAI Kubernetes blog | Profile a PyTorch DataLoader with `pin_memory` on/off; trace an NCCL all-reduce |
| 8 | HFT / low-latency: kernel bypass, DPDK, CPU pinning, busy polling | DPDK Programmer's Guide; Solarflare OpenOnload docs | Build a UDP echo server pinned with `taskset`, measure jitter with `cyclictest` |

**Books that matter (ranked):**

1. **OSTEP (Arpaci-Dusseaus, *Operating Systems: Three Easy Pieces*)** — free, the gold standard. Covers virtualization / concurrency / persistence.
2. **Robert Love, *Linux Kernel Development*** — kernel-level depth without reading source.
3. **Tanenbaum, *Modern Operating Systems*** — the classic "dinosaur" book; comprehensive but heavier.
4. **Kerrisk, *The Linux Programming Interface*** — every syscall you'll ever need; the reference manual.
5. **Brendan Gregg, *Systems Performance*** and *BPF Performance Tools* — for SRE / production debugging depth.
6. **Williams, *C++ Concurrency in Action*** — atomics, memory ordering, lock-free.
7. **Drepper, *What Every Programmer Should Know About Memory*** — free PDF; cache/NUMA bible.
8. **Bovet & Cesati, *Understanding the Linux Kernel*** — older but the source-walking standard.

**Beyond books — primary sources to internalize:**
- Linux kernel docs (`docs.kernel.org`) on EEVDF, cgroups v2, io_uring.
- Meta's *"RoCE networks for distributed AI training at scale"* (engineering.fb.com, Aug 2024).
- OpenAI's *"Scaling Kubernetes to 7,500 nodes"*.
- vLLM's PagedAttention design docs.
- NVIDIA NCCL Developer Guide.

---

### 1. Processes vs Threads vs Coroutines vs Fibers

**Core concept.** A *process* is an isolated address space with its own page tables, file descriptors, and PCB. A *thread* (kernel thread) shares its process's address space but has its own stack, registers, and PC; it's scheduled by the kernel. A *coroutine* is a user-space cooperative construct — a function that can suspend and resume, scheduled by a runtime (Go scheduler, Python `asyncio`, Kotlin). A *fiber* is a user-space thread with its own stack, scheduled cooperatively — think Boost.Fiber, Project Loom virtual threads.

**Why it matters.** Process vs thread = isolation vs throughput. Coroutines win for **I/O-bound concurrency at massive fan-out** (one OS thread can drive 100k+ coroutines) because they avoid kernel context-switch cost (~1–5 µs) and reduce per-task stack memory dramatically. Goroutines start at **2 KB** (the Go runtime's `_StackMin = 2048` since Go 1.4, after the switch from segmented to contiguous stacks) and grow on demand, whereas Linux NPTL pthreads default to the shell `ulimit -s` (typically **8 MB** — per `pthread_create(3)`: *"the stack size defaults to the value given by the 'stack size' resource limit: $ ulimit -s 8192 # The stack size limit is 8 MB"*). They lose for **CPU-bound** work because they don't add parallelism.

**Interview depth.**
- *"What does `fork()` actually do?"* → COW page tables, file descriptor inheritance, signal handler reset on `exec`, parent/child PIDs, zombie problem if parent doesn't `wait()`.
- *"Why are Go goroutines cheap but Java threads expensive?"* → 2 KB growable stack vs 8 MB default per pthread; M:N scheduling vs 1:1.
- *"When do you reach for a thread instead of an async task?"* → blocking C library, GIL-bound code, true parallelism on a CPU kernel.
- Anthropic/Databricks follow-up: *"Implement a thread-safe LRU cache"* or *"Make a hash map thread-safe; minimize contention."*

**Cross-links.** Networking (event loops); databases (connection pool = threads vs async); distributed systems (gRPC's server is thread-pool or coroutine).

**Verticals.**
- **AI/ML infra (Anthropic, OpenAI, Meta GenAI):** Python's GIL → `asyncio` + multiprocessing dominate for inference servers. PyTorch DDP launches one process per GPU — a hard constraint enforced by FSDP (per arXiv:2304.11277, *"FSDP enforces a single CUDA device per rank"*).
- **HFT (Jane Street, Citadel):** Avoid threads on the hot path. One pinned thread per critical role (feed handler, order gateway), busy-polling.
- **Backend SWE:** Threads + thread pools (Tomcat, Netty) or coroutines (Go, Kotlin).

---

### 2. Process Lifecycle, PCB, and Context Switching

States: **new → ready → running → waiting/blocked → terminated**. The **PCB** is a kernel struct (`task_struct` in Linux) holding PID, register snapshot, page-table pointer (`mm_struct`), open FDs, scheduling info, signal handlers. A **context switch** saves the running task's registers and restores another's — typically 1–5 µs on Linux; cache pollution adds another 50–500 µs of "hidden" cost from TLB/L1 misses on the new working set.

**Interview depth.** *"Walk me through what happens when the timer interrupt fires on a CFS scheduler."* Expected: interrupt handler enters kernel mode → updates `current`'s `vruntime` → checks if leftmost rbtree node has smaller `vruntime` plus wakeup granularity → if yes, calls `schedule()` → saves current registers into `task_struct`, loads new one, switches `cr3` (page table base), flushes part of the TLB unless PCID is in use, returns to user mode.

**Verticals.**
- **HFT:** `taskset -c 3 ./trader` + `isolcpus=3` kernel boot flag to keep the scheduler off that core entirely.
- **Cloud SRE (Netflix):** eBPF `runqlat.bt` measures `runqueue latency` — the single best signal of noisy neighbors.

---

### 3. Scheduling Algorithms (EEVDF, not CFS)

Classical interview table:

| Algorithm | Preemptive | Fairness | Use |
|---|---|---|---|
| FCFS | No | Poor (convoy effect) | Batch only |
| SJF / SRTF | SRTF yes | Best avg waiting time | Theoretical; needs oracle |
| Round Robin | Yes | Equal share | Time-sharing baseline |
| Priority + aging | Usually | Configurable | Real-time, interactive |
| MLFQ | Yes | Adaptive | Solaris, old Windows, OSTEP |
| **CFS** (Linux 2.6.23 → 6.5) | Yes | `vruntime` rbtree | Linux's default until 2024 |
| **EEVDF** (Linux ≥ 6.6; sole fair scheduler from 6.12, Nov 2024) | Yes | Lag + virtual deadlines | Linux today |

**EEVDF — the modern correct answer.** *Earliest Eligible Virtual Deadline First*, originally a 1995 paper by Stoica and Abdel-Wahab. Per the Linux kernel docs: *"EEVDF aims to distribute CPU time equally among all runnable tasks with the same priority. To do so, it assigns a virtual run time to each task, creating a 'lag' value… a task with a positive lag is owed CPU time, while a negative lag means the task has exceeded its portion. EEVDF picks tasks with lag greater or equal to zero and calculates a virtual deadline (VD) for each, selecting the task with the earliest VD to execute next."*

**The killer line:** *"CFS was replaced by EEVDF in 6.6 as an option and became the sole fair scheduler in 6.12 (Nov 2024); EEVDF cleans up CFS's ad-hoc latency heuristics by using a formal eligibility + virtual-deadline model."*

**Interview depth.**
- *"How does `nice` work?"* → scales the weight controlling lag accrual.
- *"How does CFS group scheduling work?"* → cgroup CPU controller assigns a group weight, then divides within the group.
- *"Real-time scheduling?"* → `SCHED_FIFO`, `SCHED_RR`, `SCHED_DEADLINE` (EDF-based, since 3.14) preempt all `SCHED_NORMAL`.

**Verticals.**
- **AI infra (OpenAI):** Per OpenAI's *"Scaling Kubernetes to 7,500 nodes"*: *"Kubernetes by default won't necessarily prioritize fulfilling all requests from one StatefulSet over another. For example if two experiments each requested 100% of the cluster's capacity… Kubernetes might schedule only half of each experiment's pods, leading to a deadlock."* Hence gang scheduling.
- **HFT:** `SCHED_FIFO` priority 99 + `chrt -f 99` for order thread, with isolated cores.

---

### 4. Inter-Process Communication (IPC)

Mechanisms: **pipes** (anonymous & named/FIFO), **UNIX domain sockets**, **TCP/UDP sockets**, **POSIX/SysV shared memory**, **POSIX/SysV message queues**, **signals**, **memory-mapped files** (`mmap MAP_SHARED`).

Speed ranking (rough): shared memory (zero-copy, ~10 ns) > UNIX domain sockets (~1–5 µs) > pipes ≈ → local TCP (~10–20 µs) → message queues.

**Interview depth.**
- *"How does Chrome do IPC between renderer and browser?"* → UNIX domain sockets on Linux, mojo IPC, shared memory for big buffers.
- *"What's a futex?"* → fast user-space mutex; uncontended case is a single atomic CAS with no syscall; contended case enters kernel. Foundation of `pthread_mutex` since NPTL.
- Apple-style: *"Write a circular buffer in C usable across two processes via `mmap`."*

**Verticals.**
- **AI infra:** PyTorch DataLoader workers use shared memory (`/dev/shm`); per PyTorch Forums: *"Device pointers to the data need to be passed back and forth between processes using CUDA IPC memory handles."*
- **HFT:** Lock-free SPSC ring buffers in shared memory between feed-handler and strategy processes. Aeron is canonical.

---

### 5. Synchronization: Mutex, Semaphore, Spinlock, RW Locks, Condition Variables

- **Mutex**: binary, sleeps on contention.
- **Spinlock**: busy-waits; very short critical sections only.
- **Semaphore**: counted permits.
- **RW lock**: many readers OR one writer.
- **Condition variable**: predicate wait; always with a mutex; idiom `while (!pred) cv.wait(lock);`.
- **Atomic operations** (`compare_exchange_strong`, `fetch_add`): lock-free building blocks.

**Databricks dedicates an entire round.** Per interviewing.io's Databricks guide: *"Coding: Concurrency/Multithreading (1 hour). This round focuses on implementing programs that leverage multithreading for efficiency."* A Blind candidate report: *"Basically just wrap the hash map in a class and add a semaphore to make sure you don't operate a bucket that's locked."*

**Anthropic concurrency frequency.** Per Exponent's Anthropic guide: *"Concurrency is a recurring theme across rounds, not isolated to one question."*

**Interview drills.**
- Implement bounded blocking queue with mutex + 2 condvars.
- Build `Once`.
- Implement RW lock; writer starvation; `pthread_rwlock_t` policy.
- Why `pthread_cond_wait` releases the mutex atomically with sleeping; spurious wakeups → `while`, not `if`.

**Verticals.**
- **HFT:** Lock-free queues (Disruptor), `std::atomic` with explicit `memory_order_acquire`/`release`, never `seq_cst` on hot paths.
- **Data engineering:** Java's `LongAdder` for contended counters.

---

### 6. Deadlocks

**Coffman's four conditions**: mutual exclusion, hold-and-wait, no preemption, circular wait.

- **Prevention**: global lock ordering (most common in production), try-lock with timeout/backoff.
- **Avoidance**: **Banker's algorithm** — almost never used in production; classic interview question.
- **Detection**: wait-for graph cycle detection. PostgreSQL runs this after `deadlock_timeout` (default 1 s) and aborts the youngest cycle member.

**Interview depth.**
- *"Explain dining philosophers; give two solutions."* → Asymmetric (philosopher n picks left then right; n−1 right then left) breaks circular wait. Or a waiter semaphore allowing only n−1 to eat.
- Banker's algorithm step-by-step on a small matrix — common at Indian-campus interviews and Amazon SDE-I screens.

---

### 7. Memory Management: Paging, Segmentation, Virtual Memory, TLB

Each process sees a virtual address space (typically 48-bit canonical on x86-64, 256 TB). MMU translates via 4-level page tables (5 with `LA57`). **TLB** caches recent translations.

**Critical numbers (memorize):**
- TLB hit ~1 cycle; TLB miss = page walk = up to 4 memory accesses = 40–400 cycles.
- Per abhik.ai: *"with standard 4 KB pages, even a generous TLB with 1,536 entries only covers about 6 MB of memory."*
- 2 MB huge page + same TLB covers ~3 GB — 512× more TLB reach.

**Interview depth.**
- *"What happens on a page fault?"* → CPU traps to kernel → kernel finds VMA → if valid, allocates a frame (possibly reading from disk/swap), updates PTE, flushes TLB for that page, retries instruction. **Major** (disk) vs **minor** (no I/O) faults.
- *"What is COW in `fork()`?"* → Parent and child share all pages read-only after `fork`; first write traps, kernel copies the page.
- *"Measure TLB pressure?"* → `perf stat -e dTLB-load-misses,iTLB-load-misses`.
- *"Explain `mmap` vs `read()`."* → `mmap` maps file into address space; reads are demand-paged memory accesses. Used heavily by Kafka, Lucene, Spark shuffle.

**Verticals.**
- **Data engineering:** Disable Transparent Huge Pages for JVM/Hadoop (per Cloudera, Splunk).
- **AI training:** Pinned memory is mandatory for fast H2D (see §17).
- **HFT:** Explicit huge pages via `hugetlbfs`; THP off to avoid `khugepaged` jitter.

---

### 8. Page Replacement Algorithms

- **OPT** — optimal but unachievable; baseline.
- **FIFO** — Belady's anomaly.
- **LRU** — approximated, not exact.
- **Clock / Second Chance** — the LRU approximation real kernels use.
- **LFU** with aging.
- **2Q / ARC / LIRS** — modern (PostgreSQL, ZFS).

Per Redis docs, Redis approximates LRU by sampling N keys (`maxmemory-samples`, default 5) and evicting the oldest in the sample — pure LRU is too expensive at scale.

**Interview drill.** LeetCode 146 — LRU in O(1) with hashmap + doubly linked list.

---

### 9. Memory Allocators: malloc Internals, Slab, jemalloc, tcmalloc

Variants:
- **ptmalloc2** (glibc default) — per-thread arenas; fragments badly under multithreaded long-running workloads.
- **jemalloc** (Meta-maintained; Redis, Firefox, FreeBSD default).
- **tcmalloc** (Google) — thread-local caches.
- **mimalloc** (Microsoft).
- **Slab allocator** (Linux kernel) — fixed-size object caches.

Per DEV.to benchmark by Kunal Ganglani: *"With glibc, I've seen RSS grow 30-40% above the actual working set due to fragmentation. jemalloc typically stays within 10-15%. tcmalloc lands somewhere in between."*

**Industry validation with specific numbers.** Per Arcesium engineering blog by Sanket Mehta (VP, Senior Principal Engineer), March 2025, *"From Malloc to Jemalloc: Slashing Java Container Memory Usage by 10%"*: *"this optimization lowered our overall memory consumption from 2.2 TB to 1.98 TB for our application, resulting in an approximate 10% cost reduction."* Per Piotr Sarna, Software Engineer at ScyllaDB (as cited by Kunal Ganglani): *"The ScyllaDB team documented a 40% performance boost after switching to jemalloc… attributed the improvement to reduced memory fragmentation and improved scalability on multi-core systems."*

**Interview depth.**
- *"Walk me through `malloc(24)`."* → Size class 32 B → thread cache → per-CPU/arena cache → central free list → `mmap`/`sbrk` if needed.
- *"What's `MALLOC_CONF`?"* → jemalloc tunables (`narenas`, `dirty_decay_ms`, `prof:true`).

**Verticals.**
- **AI infra (PyTorch):** PyTorch's caching allocator is a tcmalloc-shaped layer over `cudaMalloc`; `cudaMallocAsync` (CUDA 11.2+) is the stream-ordered native equivalent.
- **HFT:** No `malloc` on hot path — pre-allocate, object pools.

---

### 10. File Systems: inodes, Journaling, COW

- **Journaling** (ext3/4, XFS, NTFS) — log metadata (and optionally data) before applying; crash replay.
- **Copy-on-write** (ZFS, Btrfs, APFS) — never overwrite in place; new blocks + pointer swap. Per Klara Systems: *"Both ZFS and Btrfs employ copy-on-write (CoW). Instead of overwriting existing blocks, new blocks are written and metadata is updated to point to them."*
- ZFS combines filesystem + volume manager + RAID; CDDL license keeps it a kernel module on Linux.
- Btrfs RAID 5/6 still flagged unsafe for production.
- APFS (Apple, 2017+) is COW with cloning, snapshots, encryption — replaced HFS+.

**Interview depth.**
- *"Walk through `open(\"/etc/passwd\", O_RDONLY)`."* → path lookup in dcache → inode read → perms check → allocate FD in `task_struct->files`.
- *"Why is `rename()` atomic on ext4?"* → single journaled metadata transaction.

**Verticals.**
- **Data engineering:** Delta Lake / Iceberg / Hudi = COW + journaling layered over S3. Per Datavidhya's Databricks guide: *"Delta Lake is an open-source storage layer that brings reliability to data lakes by providing ACID transactions, scalable metadata handling, and unified streaming and batch data processing… built on top of the Parquet file format but adds a JSON-based transaction log (the 'Delta Log')."* Databricks interviewers grill on optimistic concurrency and vacuum semantics.

---

### 11. I/O: Blocking, Non-blocking, async, io_uring, epoll, kqueue

- **Blocking I/O** — one thread per connection; caps at ~10k.
- **Readiness notification** — `select` (1024 FD cap, O(n)) → `poll` → **`epoll`** (Linux, O(1) ready set) / **`kqueue`** (BSD/macOS).
- **Asynchronous** — kernel does the work; you get the result. **`io_uring`** (Linux 5.1+, 2019, Jens Axboe) uses two mmap'd ring buffers shared between user and kernel.

Per gocodeo: *"io_uring uses memory-mapped queues to eliminate redundant syscalls, allowing applications to perform thousands of I/O operations with a single io_uring_enter() call, or none at all in polling mode."*

**Real-world nuance — io_uring does NOT always beat epoll.** Per GitHub `axboe/liburing#189` (alexhultman): *"In all tests epoll is performing better by a measurable amount. In no single test have I seen io_uring beat epoll in this kind of test"* (simple TCP echo, 40–400 clients). Wins concentrate in mixed file+network I/O and very high QPS.

**Interview depth.**
- *"Compare `select`, `poll`, `epoll`."* → FD limits, O(n) vs registered + delta, ET vs LT.
- *"What's `eventfd`?"* → kernel counter usable as an FD for `epoll`.
- *"Why is async file I/O historically hard on Linux?"* → Regular files always show ready in `epoll`; `libaio` only worked for O_DIRECT; `io_uring` finally fixed it.

**Verticals.**
- **Backend at scale:** `epoll` universal (Nginx, Redis, Envoy, Node.js, Tokio). Green-field kernel-5.10+ projects default to `io_uring`.
- **HFT:** Neither — bypass the kernel (DPDK).
- **AI infra:** GPU↔HBM dominates; `io_uring` is gaining traction in DataLoader replacements.

---

### 12. System Calls: User vs Kernel Mode

On x86-64 the instruction is `syscall`; args in registers per SysV ABI; syscall number in `rax`; result in `rax`. Cost: ~100–300 ns for a no-op like `getpid()`.

**KPTI impact is workload-dependent, not a flat doubling.** Per Brendan Gregg's 2018 analysis (brendangregg.com): *"The KPTI patches to mitigate Meltdown can incur massive overhead, anything from 1% to over 800%."* For Netflix's highest-rate workloads (~50k syscalls/sec/CPU on databases), Gregg estimated *"between 0.1% and 6% overhead with KPTI"* — highly workload-dependent.

**Interview depth.**
- *"Trace `read(fd, buf, 4096)`."* → User registers → `syscall` → `entry_SYSCALL_64` → kernel stack → syscall table → `sys_read` → `file->f_op->read_iter` → FS/socket layer → return → `sysretq`.
- *"How does `vDSO` make `gettimeofday()` ~10× faster?"* → Kernel exposes clock data via shared RO mapping; no syscall needed.

---

### 13. Interrupts, Traps, Exceptions

- **Interrupt** — async hardware (NIC, timer, disk).
- **Trap** — synchronous intentional (syscall).
- **Exception** — synchronous unintentional (page fault, segfault, #DE divide-by-zero).

Linux splits handlers into **top half** (interrupt context, can't sleep) + **bottom half** (softirq/tasklet/workqueue). 100 GbE saturates a core with interrupts → **NAPI** (interrupt+polling hybrid) → **DPDK** (pure polling, IRQs disabled on dataplane core).

---

### 14. Boot → Kernel Init

UEFI → ESP → bootloader (GRUB/systemd-boot) → loads `vmlinuz` + `initramfs` → kernel decompresses, sets up paging, mounts root → `pid 1` (systemd) → user space.

Apple embedded interviews drill this. Per Dataford's Apple embedded guide: *"How would you determine if a machine's stack grows up or down?"*

---

### 15. Linux-Specific: cgroups, Namespaces, eBPF, /proc

- **cgroups v2**: CPU/memory/I/O/PID limits via `/sys/fs/cgroup`.
- **Namespaces**: PID, NET, MNT, IPC, UTS, USER, CGROUP, TIME.
- **eBPF**: sandboxed bytecode running in kernel, verified for safety; attached to kprobes, tracepoints, XDP, sockets, LSM.

**Production eBPF (eBPF Foundation case studies):**
- *"Meta uses eBPF to process and load balance every packet coming into their data centers"* (Katran).
- *"Google uses eBPF in GKE, developed and uses BPF LSM to replace audit and it uses eBPF for networking."*
- *"Netflix uses eBPF for fleet-wide network observability and performance diagnosis."* Per causely.ai on the 2024 eBPF Summit, Shweta Saraf at Netflix said *"Netflix uses eBPF to measure how long processes spend in the CPU scheduled state. When processes are taking too long, it usually indicates a performance bottleneck on CPU resources — like CPU throttling or over-allocation."* Netflix released `bpftop`.
- Cloudflare uses XDP eBPF for DDoS mitigation.

**Interview depth.**
- *"How does Docker isolate?"* → namespaces (PID, NET, MNT, IPC, UTS, USER) + cgroups (CPU, MEM) + overlay FS + seccomp + AppArmor/SELinux + capabilities drop.
- *"What is XDP?"* → eBPF at NIC driver / hardware offload point, before kernel network stack.
- *"How does Cilium replace `kube-proxy`?"* → eBPF in-kernel service load balancing, no iptables hairpinning.

---

### 16. Containers and VMs

Containers = namespaces + cgroups + overlay FS. Type 1 hypervisors (Xen, KVM, ESXi, Hyper-V) run on hardware; Type 2 (VirtualBox, VMware Workstation) run on a host OS. AWS Firecracker microVMs (used by Lambda) blend the two for ~125 ms cold starts.

Per OpenAI's *"Scaling Kubernetes to 7,500 nodes"*: *"A large machine learning job spans many nodes and runs most efficiently when it has access to all of the hardware resources on each node. This allows GPUs to cross-communicate directly using NVLink, or GPUs to directly communicate with the NIC using GPUDirect. So for many of our workloads, a single pod occupies the entire node. Any NUMA, CPU, or PCIE resource contention aren't factors for scheduling."*

---

### 17. OS in ML/AI (the deep section)

This is where GenAI engineers earn their salary. Top AI labs probe these in depth.

#### 17.1 Pinned (page-locked) memory + CUDA streams

Per PyTorch official tutorial: *"a pinned (or page-locked or non-pageable) memory is a type of memory that cannot be swapped out to disk… If the memory is page-locked, the device can access the memory directly in the main memory… If the memory is pageable, all the pages will have to be brought to the main memory before being sent to the GPU."* Per MoldStud citing NVIDIA's developer blog, *"allocating pinned host memory (cudaHostAlloc) typically accelerates data movement by up to 30% compared to pageable RAM."*

**CUDA streams pattern (canonical overlap):**
```cuda
cudaMemcpyAsync(d_buf, h_pinned, size, H2D, stream1);
myKernel<<<grid, block, 0, stream1>>>(d_buf);
// On stream2 in parallel: next batch's H2D
```
Overlap requires pinned memory — pageable copy is synchronous regardless.

#### 17.2 PagedAttention and KV cache (the canonical AI-OS topic)

**vLLM's PagedAttention (Kwon et al., SOSP '23)** directly applies OS virtual-memory ideas: break KV cache into fixed-size **blocks** (typically 16 tokens × layers × heads); per-request **block table** maps logical → physical (analogous to page table); global free block pool. Per Hamza Elshafie's first-principles writeup of the vLLM paper: *"Previous systems wasted 60%–80% of the KV cache memory, whereas vLLM achieves near-optimal memory usage with less than 4% waste."* Per Anyscale: *"By leveraging vLLM, users can achieve 23x LLM inference throughput while reducing p50 latency."*

**This is the question Anthropic asks.** Per Exponent's Anthropic system-design guide built from multiple recent candidates:

> *"Design an inference batching system. You have a single GPU that can process up to 100 inputs per batch. Users submit requests synchronously and wait for results. Design the system that receives inputs, batches them, processes them on the GPU, and returns responses to the correct users."*

Exponent flags it as *"the most commonly reported Anthropic system design prompt."*

A 2025 Anthropic candidate report (Linkjob.ai): *"The question I got was to design an inference API for serving large language models… handling variable-length requests, managing GPU memory under concurrent requests, implementing a priority-based request queue, and supporting streaming responses… how to dynamically group requests with similar lengths to maximize GPU utilization… KV cache management and auto-scaling strategies. I proposed using queue depth weighted by the estimated token count as the signal for scaling, instead of just relying on raw GPU utilization."*

**KV cache memory math** (per digitalapplied.com): *"On a Llama 70B baseline at 1M context, KV cache hits ~135 GB at FP16 — more than the 140 GB of FP16 model weights."* Above 128K context, KV cache dominates parameter memory.

#### 17.3 Continuous batching (Orca, OSDI '22)

Per the USENIX Orca paper: *"iteration-level scheduling, a new scheduling mechanism that schedules execution at the granularity of iteration (instead of request) where the scheduler invokes the execution engine to run only a single iteration."* Per Anyscale: *"The ORCA paper… demonstrated 36.9x throughput improvement over FasterTransformer at equivalent latency targets."*

#### 17.4 NCCL, NVLink, GPUDirect RDMA

Per the NVIDIA NCCL Developer Guide: *"NCCL implements multi-GPU and multi-node communication primitives… all-gather, all-reduce, broadcast, reduce, reduce-scatter as well as point-to-point send and receive that are optimized to achieve high bandwidth and low latency over PCIe and NVLink high-speed interconnects within a node and over NVIDIA Mellanox Network across nodes."*

**Numbers to memorize**, per NVIDIA's developer blog *"Scaling Deep Learning Training with NCCL"*: *"If some of the GPUs are connected to different CPU sockets without NVLink, GPUs will have to communicate through shared memory which will limit the bandwidth to 5GB/s. With all GPUs connected directly through PCI, NCCL can use GPU Direct P2P… to achieve 12GB/s… Pascal GPUs can communicate at up to 62GB/s using four NVLink connections, while Volta GPUs can reach 132GB/s using six second generation NVLinks."*

**GPUDirect RDMA failure mode (top AI-infra interview signal).** Per Pingdo: *"In modern Linux distributions, the `nvidia-peermem` module acts as a translator between the NVIDIA GPU Driver and the Mellanox/Broadcom IB Core… If this module is missing, NCCL will silently fallback to System Memory Relays, resulting in a 3-5x performance degradation in All-Reduce operations."* And: *"GPUDirect RDMA is only successful if the NIC and GPU share the same Root Complex."*

#### 17.5 PyTorch FSDP/DDP at the OS layer

Per the Meta FSDP paper (arXiv:2304.11277): *"FSDP enforces a single CUDA device per rank and uses a single process group for both AllGather and ReduceScatter, which means that its collectives run sequentially in the process group's internal NCCL stream. In the backward pass, FSDP issues the ReduceScatter for the current FlatParameter and then the AllGather for the next FlatParameter."*

#### 17.6 Production engineering blogs (quote these)

- **Meta** (engineering.fb.com, *"RoCE networks for distributed AI training at scale"*, Aug 2024): *"We opted for RDMA Over Converged Ethernet version 2 (RoCEv2) as the inter-node communication transport for the majority of our AI capacity. We have successfully expanded our RoCE networks, evolving from prototypes to the deployment of numerous clusters, each accommodating thousands of GPUs."* And: *"a typical generative AI (GenAI) job may necessitate tight coordination of tens of thousands of GPUs over the course of several weeks."*
- **OpenAI** (*"Scaling Kubernetes to 7,500 nodes"*): *"we often use MPI to coordinate between optimizer members, and MPI is sensitive to group membership changes."*

#### 17.7 OpenAI-reported interview questions

Per IGotAnOffer's OpenAI Coding Guide: *"Many OpenAI candidates also report getting 'Implement a GPU credit calculator' during the phone screen."*

A 2025 Medium candidate report by Anqi Silvia: *"Infra System Design: Design a distributed training platform for foundation models. I needed to consider sharded training, logging, fault tolerance, and model and data versioning. Interviewer Follow-up: If you had to scale to 2000 GPUs, how would you partition the parameters? If a task fails, how would you automatically recover?"*

#### 17.8 What separates a good answer from a great one (Anthropic-grade)

**Good:** *"I'd batch on the server, use a queue, manage GPU memory."*

**Great:**
> *"I'd put a token-aware admission queue in front. Each request has an estimated token count. I'd implement continuous batching (Orca iteration-level scheduling) so that finished sequences free their slots immediately. KV cache uses PagedAttention with 16-token blocks; eviction by LRU on idle requests. Autoscaler scales on queue depth weighted by estimated total tokens, not raw GPU util, because raw util misleads under variable sequence length. For multi-GPU, I'd shard with tensor parallelism inside a node over NVLink (132 GB/s on V100), pipeline parallelism across nodes over RoCE/IB. I'd ensure `nvidia-peermem` is loaded so all-reduce uses GPUDirect RDMA rather than bouncing through host memory — that's a 3–5× difference. Training/inference processes pinned to the NUMA node hosting their NIC and GPUs to avoid cross-socket UPI traffic."*

That answer threads OS fundamentals, GPU systems, distributed systems, and named numbers. It's the answer Anthropic and OpenAI want.

---

### 18. OS in Distributed Systems

- **Network stack tail-latency knobs**: `TCP_NODELAY` (disable Nagle), `SO_REUSEPORT` (kernel-level load balance), `SO_BUSY_POLL`, `TFO` (TCP Fast Open).
- **TIME_WAIT exhaustion** on busy connection-pool clients — classic gotcha.
- **CAP and the OS**: P is partly the kernel's network stack — a microburst causes a 20 ms TCP retransmit that looks like a partition.
- **Consensus and `fsync`**: Raft/Paxos commit ≈ `fsync` cost — ~100 µs on NVMe, ~1–10 ms on rotational.
- **Clock**: NTP (ms), PTP (µs, hardware-assisted), Spanner's TrueTime (atomic clocks + GPS).

**Kafka** is famously a thin coat of paint over OS primitives: append + `fsync`, `mmap` of segments for zero-copy reads (`sendfile`), `epoll` for client sockets, page cache as the cache layer.

---

### 19. OS for Data Engineering (Spark, Huge Pages)

**The Spark/Hadoop Linux-tuning checklist:**

- **Disable Transparent Huge Pages.** Per Cloudera: *"Most Linux platforms supported by CDH 5 include a feature called transparent hugepages, which interacts poorly with Hadoop workloads and can seriously degrade performance."* Symptom: `top` showing 30%+ system CPU.
- Per Splunk: *"Splunk has observed a minimum of a 30% degradation in indexing and search performance on Linux systems where THP is active, with a similar percentage increase in latency."*
- **`vm.swappiness=1`.** Per Cloudera: *"between 1 and 10, preferably 1, for minimum swapping on systems where the RHEL kernel is 2.6.32-642.el6 or higher."*
- **Memory-mapped shuffle**: Spark uses `mmap` to read shuffle data without page-cache→user-buffer copying.
- **JVM + jemalloc**: `LD_PRELOAD=/usr/lib/x86_64-linux-gnu/libjemalloc.so.2`. Arcesium reported ~10% cost reduction (2.2 TB → 1.98 TB).

**THP for ML/columnar is different.** Per abhik.ai: *"In-memory analytics engines like Spark and ClickHouse process billions of rows in columnar data structures. The combination of large allocations and sequential scans makes them ideal THP candidates, with measured query speedups of 20-40%."* The decision rule: **THP off by default for JVM-Spark, explicit `hugetlbfs` for predictable workloads.**

**Databricks Photon.** Per Datavidhya's Databricks guide: *"Photon is a native execution engine developed by Databricks, rewritten from the ground up in C++. Unlike the standard Spark engine that runs on the JVM, Photon uses vectorized execution to process data in batches rather than row-by-row."* Vectorized = SIMD-friendly memory layout = cache-line awareness (§20).

---

### 20. CPU Caches, MESI, False Sharing, Memory Barriers, Atomics

| Level | Size | Latency | Per |
|---|---|---|---|
| L1 D/I | 32–64 KB | ~1 ns / 4 cycles | core |
| L2 | 256 KB – 1 MB | ~3 ns / 12 cycles | core |
| L3 | 8–64 MB | ~10 ns / 40 cycles | socket (shared) |
| DRAM | 64+ GB | ~100 ns | system |

**Cache line = 64 B on x86.**

**MESI protocol**: each line is **M**odified / **E**xclusive / **S**hared / **I**nvalid. Writes to S require invalidations → S elsewhere → I.

**False sharing**: two threads write *different* variables on the *same* cache line. Ping-pong → 100× slowdown. Fix: `alignas(64)` in C++, `@Contended` in Java.

**`std::memory_order`**:
- `relaxed` — atomicity only.
- `acquire` (loads) / `release` (stores) — standard lock-free producer-consumer.
- `seq_cst` — expensive default; avoid on hot paths.

**HFT interview depth.**
- *"Show me a case where `volatile` in C++ is NOT sufficient for thread safety."* → `volatile` only prevents compiler reordering, not CPU reordering. Use `std::atomic`. (Apple favorite.)
- *"Implement SPSC ring buffer with no locks."* → `head`/`tail` are `std::atomic<size_t>`; producer reads `tail` with `acquire`, writes `head` with `release`; consumer dual.
- *"Explain the Disruptor pattern."* → LMAX's lock-free SPMC ring; padding to avoid false sharing; sequence barriers.

Per VXRL on HFT: *"firms like Jane Street, Optiver, and Citadel care obsessively about latency. We aren't just talking about milliseconds; we are talking about nanoseconds and CPU clock cycles. To get that level of performance, you need 'mechanical sympathy' – a deep understanding of how the CPU actually executes your code."*

---

### 21. NUMA Architecture

Each socket has its own memory controller and "local" DRAM. **The NUMA factor is typically 1.5–3×.** Per systemoverflow.com on NUMA: *"Accessing local memory takes 80 to 100 nanoseconds. Accessing remote memory on another socket takes 150 to 300 nanoseconds. The ratio is called the NUMA factor, typically 1.5x to 3x."* PCIe devices (NICs, GPUs) are also socket-attached — cross-socket access kills latency.

**Tools.** `numactl --hardware`, `numactl --cpunodebind=0 --membind=0 ./app`, `lstopo`.

**Verticals.**
- **HFT:** Pin to socket connected to the NIC.
- **AI infra:** Pin training processes to socket co-located with NIC and GPUs (see §17.4 — GPUDirect RDMA requires same root complex).
- **Databases:** Postgres / MySQL configured per socket.

---

### 22. OS for HFT (Kernel Bypass, DPDK, Busy Polling, CPU Pinning)

The complete HFT OS-tuning checklist — table stakes at Citadel Securities / Jane Street / Jump / HRT:

1. **Kernel bypass for networking**: **DPDK** (Intel, Linux Foundation), **Solarflare OpenOnload**, **PF_RING ZC**, **RDMA/InfiniBand**, **AF_XDP**. Per systemdr Substack: *"Traditional operating systems add 20-50 microseconds of latency just moving packets through the kernel. For context, light travels only 6 kilometers in 20 microseconds. This is why elite HFT firms bypass the kernel entirely."*
2. **Polling, not interrupts.** DPDK uses Poll Mode Drivers; CPU shows 100% busy regardless of load. Per Intel: *"in data plane applications… the DPDK is supposed to poll a certain port for incoming packets in an infinite loop, pinned to a certain logical core."*
3. **CPU pinning + isolation.** `isolcpus=` boot param + `taskset` / `pthread_setaffinity_np`. Per QuantVPS: *"Pin workloads to specific CPUs to limit context switching… Disable hyper-threading to allocate dedicated CPU resources for essential tasks. Enable CPU isolation to keep non-essential processes from interfering with critical operations."*
4. **No SMT/hyper-threading** on hot cores (sibling logical cores share L1/L2 → jitter).
5. **CPU frequency governor `performance`.** Disable C-states (C6 wake can be 100+ µs).
6. **NUMA-aware placement.**
7. **Explicit huge pages**; THP off.
8. **Real-time scheduling.** `SCHED_FIFO` priority 99.
9. **Lock-free everything** on hot path. Pre-allocated object pools. No `malloc`, no syscalls, no synchronous-disk logging.
10. **FPGA offload** for the lowest latencies. Per appinventiv citing NVG Associates: *"FPGA-based high-frequency trading (HFT) systems using the Solarflare Application Onload Engine consistently achieve latencies between 750–800 nanoseconds for market events to order messages."*

**Citadel Securities interview signal.** Per techinterview.org: *"What works: explicit latency budget reasoning (microsecond level for critical paths), kernel bypass and CPU-pinning awareness, lock-free or wait-free data structures where contention matters, hardware-aware design (NUMA, cache hierarchies). What doesn't: generic cloud-architecture answers ignoring the latency-critical reality."*

**Branchless / constant-time programming** comes up too. Per VXRL: *"In cryptography, branched code is a massive vulnerability… To defeat this, cryptographers must use branchless programming (often called 'Constant-Time Programming')."*

---

### 23. Top Interview Questions, With Ideal Answer Frameworks

**Tier 1 — every company:**

1. **Process vs thread vs coroutine.** Framework: isolation/scheduling/cost; COW `fork()`; M:N for goroutines.
2. **Mutex vs semaphore vs spinlock.** Framework: binary vs counted vs busy-wait; when each applies.
3. **Walk through a page fault.** Framework: trap → VMA check → frame allocation → PTE update → TLB flush → retry. Major vs minor.
4. **`fork()` + `exec()` — why two syscalls?** Framework: shell redirection between them; child sets up FDs before exec.
5. **Thread-safe LRU (LeetCode 146 + concurrency).** Hashmap + DLL O(1). Sharded lock OR `shared_mutex`.
6. **`epoll` vs `select`.** RB-tree interest set vs full scan; ET vs LT.
7. **Deadlock — Coffman + detection + prevention.** Lock ordering; wait-for graph; Banker's algorithm.

**Tier 2 — senior:**

8. **MESI + false sharing.** Padding to 64 B.
9. **`volatile` is NOT thread-safe in C++.** Apple favorite.
10. **CUDA stream + pinned memory overlap.** 3-stream pipeline.
11. **Design LLM inference batching.** *Use the §17.8 paragraph.* (Anthropic.)
12. **THP — when on, when off?** DBs/Redis/JVM-Spark off; ML/columnar on.
13. **Mutex+condvar producer-consumer, then lock-free SPSC.**
14. **Docker isolation.** Namespaces + cgroups + overlay + seccomp + caps.
15. **Syscall trace.** Mention KPTI.

**Tier 3 — vertical:**

16. **Sub-µs market data path.** DPDK + isolated CPU + lock-free SPSC + FPGA.
17. **TLB pressure on 100 GB Postgres pool.** Explicit huge pages.
18. **NCCL all-reduce 3–5× regression?** Missing `nvidia-peermem`; or NIC/GPU not on same PCIe root complex.

---

### 24. "Expert Signal" Tips — What Separates Good from Great

1. **Name the specific implementation.** Not "the Linux scheduler" — say **EEVDF** (replaced CFS in 6.6, sole scheduler since 6.12 Nov 2024). Not "allocator" — say **jemalloc** (Meta-maintained) or **tcmalloc** (Google). Not "namespaces" — say **PID, NET, MNT, IPC, UTS, USER, CGROUP, TIME**.
2. **Quote real numbers.** *4 KB page, 64 B cache line, 1 ns L1, 100 ns DRAM, ~1 µs context switch, 10 µs local TCP RTT, ~100 µs NVMe `fsync`, 132 GB/s NVLink2, 12 GB/s PCIe P2P, 5–400 Gb/s RoCE/IB, NUMA factor 1.5–3×.*
3. **Always discuss the workload.** Reads vs writes, short vs long critical sections, throughput vs latency, p50 vs p99.
4. **Tie OS to higher-level systems.** PagedAttention = virtual memory applied to KV cache. Kafka = `mmap` + page cache + `sendfile`. RDBMS lock manager = OS deadlock detector with wait-for graph.
5. **Reference primary sources.** *"Per Meta's SIGCOMM '24 paper on RoCE…"*; *"Per the Orca OSDI '22 paper…"*; *"Per OpenAI's Kubernetes-to-7500 blog…"*.
6. **Show awareness of where classical theory breaks.** Banker's algorithm needs max-resource oracle. OPT page replacement unachievable; Clock is what ships. Pure LRU too expensive; Redis samples.
7. **Talk failure modes, not just happy path.** What happens on a NIC failure mid-all-reduce? On `fsync` returning success but disk lying? On a deadlock at p99 only?
8. **AI labs: speak GPU systems fluently.** NCCL collectives, NVLink topology, GPUDirect RDMA, `nvidia-peermem`, PagedAttention, continuous batching, KV cache eviction, pinned memory.
9. **HFT: speak mechanical sympathy.** Cache lines, branchless, atomics with explicit ordering, lock-free queues, NUMA, kernel bypass.
10. **Structure your answer.** Top-tier interviewers explicitly grade clarity of explanation as much as technical correctness — every Anthropic/Google/OpenAI guide cited above says this.

---

### 25. Cross-Reference Tag Index

- **#processes** → §1, §2
- **#scheduling** → §3, §15, §17 (gang scheduling)
- **#ipc** → §4, §11, §17 (CUDA IPC)
- **#synchronization** → §5, §6, §20
- **#memory** → §7, §8, §9, §17, §19, §21
- **#filesystem** → §10, §11, §19
- **#io** → §11, §18, §22
- **#kernel-internals** → §12, §13, §14, §15
- **#containers** → §15, §16, §17
- **#caches-and-coherence** → §20, §21
- **#ml-systems** → §17 (the deep section), §3, §16, §21
- **#distributed-systems** → §17, §18, §15, §16
- **#data-engineering** → §19, §11, §9
- **#hft-low-latency** → §22, §20, §21, §13

---

### 26. Final Mental Model

The OS layer is the lowest-level mental model every other layer implicitly assumes:

- A **distributed system** is processes + IPC + scheduling.
- A **database** is page tables + journaling + caches.
- An **LLM inference server** is virtual memory (PagedAttention) + scheduling (continuous batching) + DMA (pinned memory) + IPC (NCCL).
- An **HFT system** is the OS with everything turned off and bypassed.

Three things to internalize:

1. **The right mental models** (process, address space, page, cache line, scheduler, fs, syscall).
2. **What Linux actually uses today**: EEVDF (not CFS), cgroups v2, io_uring beside epoll, jemalloc/tcmalloc on real services, PagedAttention as the AI-infra VM analogue.
3. **Connect the dots upward** — every higher-level system is a re-instantiation of an OS concept.

Do those three and you'll do well at any company named in the brief.

---

## Recommendations

**Staged 8-week plan, with thresholds for staying vs pivoting:**

1. **Weeks 1–2 (Foundations).** Read OSTEP Parts I + II. Write a tiny shell and a producer-consumer with mutex+condvar in C. **Threshold:** can you, without notes, explain a page fault, a context switch, and why `pthread_cond_wait` releases the mutex atomically? If yes → proceed. If no → re-read, don't accelerate.

2. **Weeks 3–4 (Memory + I/O).** OSTEP Part III. Implement an `epoll` echo server, then port it to `io_uring`. Measure with `wrk`. **Threshold:** can you name 5 differences between `epoll` and `io_uring` and identify one workload where `epoll` still wins? If yes → proceed.

3. **Weeks 5–6 (Linux internals + caches).** Read Love's *Linux Kernel Development*, Drepper's memory paper. Write a microbench for false sharing; show MESI transitions in `perf c2c`. **Threshold:** can you explain what `nvidia-peermem` does and what breaks without it?

4. **Week 7 (AI infra OS, the differentiator).** Read vLLM PagedAttention docs, NCCL Developer Guide, Meta SIGCOMM '24 paper, OpenAI K8s blog. Practice the Anthropic batching question out loud, twice — once at 5 minutes and once at 30 minutes. **Threshold:** can you deliver the §17.8 "great answer" paragraph from memory in 90 seconds?

5. **Week 8 (Specialization).**
   - Targeting **Anthropic/OpenAI/DeepMind**: deep-dive Orca, FlashAttention, FSDP paper. Practice GPU credit calculator coding problem.
   - Targeting **HFT (Jane Street/Citadel)**: implement an SPSC ring buffer, learn DPDK basics, study Disruptor pattern.
   - Targeting **Databricks/Snowflake**: brush up on Delta Lake, Photon, Spark concurrency. Solve LeetCode concurrency tag end-to-end.
   - Targeting **Meta/Google/Netflix**: study eBPF and `bpftop`, read Katran paper.

**Decision rules:**
- **If you can recite §17 (the AI-OS deep section) from memory cold**, you're ready for Anthropic infrastructure interviews.
- **If you can derive `vruntime` and explain EEVDF's lag mechanism on a whiteboard**, you're ready for Google L5+ systems interviews.
- **If you can write a lock-free SPSC ring with correct `memory_order`** without referencing notes, you're ready for HFT phone screens.
- **If you score below 80% on the §23 Tier 1 questions**, do NOT yet take on-site interviews — backfill foundations first.

**Highest-leverage single action: read OpenAI's "Scaling Kubernetes to 7,500 nodes" blog and Meta's "RoCE networks for distributed AI training at scale" (SIGCOMM '24) end-to-end this week.** These two pieces alone supply most of the AI-infra OS vocabulary top labs expect.

---

## Caveats

1. **Some candidate-reported questions cited above (Linkjob.ai, Jobright.ai, Medium first-person posts) are single-source and unverifiable.** They should be treated as plausible signal, not authoritative interview banks. Higher-confidence sources are official engineering blogs (Meta, OpenAI), primary papers (Orca OSDI '22, FSDP arXiv), and Exponent / IGotAnOffer / interviewing.io / techinterview.org guides which aggregate across multiple verified candidates.

2. **The "23× throughput" (vLLM) and "36.9× throughput" (Orca) numbers are paper-reported, on specific benchmarks (FasterTransformer baseline; LLM serving with variable-length requests).** Real-world gains vary 5–25× depending on workload variance. Don't cite as if they're production guarantees.

3. **The Anthropic, OpenAI, DeepMind interview questions cited are from 2024–2025 candidate reports and aggregator guides; processes and questions change.** The pattern (GPU inference batching, KV cache, distributed training fault tolerance) is stable; the exact prompt is not.

4. **Some sources lean predictive or speculative.** glennklockwood.com on Anthropic/OpenAI prompt-cache tier locations explicitly says *"likely implemented in GPU HBM"* — inferred, not company-confirmed. Treat these as hypotheses to demonstrate reasoning, not facts.

5. **The 132 GB/s NVLink and 12 GB/s PCIe P2P figures from NVIDIA's developer blog are Volta-generation.** H100/Blackwell have higher numbers (NVLink 4: ~900 GB/s aggregate per GPU; H100 PCIe Gen5: 128 GB/s). Adapt the numbers to the latest generation you're claiming familiarity with.

6. **EEVDF is still being actively tuned in 2025–2026.** Per Phoronix, *"EEVDF Scheduler On The Verge Of Being 'Complete'"* (July 2024) noted continued patch work. The scheduler in 6.16 may behave differently from 6.6's first cut. The conceptual model (lag + virtual deadline + eligibility) is stable; specific tunables drift.

7. **`io_uring` is not universally faster than `epoll` in benchmarks.** The arXiv DBMS paper (2512.04859, 2025) carefully argues *"when and how to use it"* — caveat hype. Use it for mixed file+network I/O at very high QPS; stick with `epoll` for mature TCP-only code.

8. **THP guidance is workload-specific and historically contentious.** Cloudera/Splunk/Redis recommend off; ML/columnar workloads benefit. Always benchmark before changing in production.

9. **HFT numbers (750 ns FPGA latency, 20–50 µs kernel overhead) come from vendor and industry blogs**; they're representative but not your specific hardware. Treat as orders of magnitude.

10. **This primer assumes Linux x86-64.** Apple Silicon (ARM64) has different cache topology and a different scheduler story; Windows uses a different scheduler entirely. Adjust answers to the platform of the team you're interviewing with.