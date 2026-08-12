# Silicon Fundamentals — Interview Cheat Sheet

## Table of Contents

**Foundational**
1. [Combinational vs Sequential Circuits and Metastability](#1-combinational-vs-sequential-circuits-and-metastability)
2. [Single-bit CDC: 2-Flop Synchronizer](#2-single-bit-cdc-2-flop-synchronizer)
3. [Multi-bit CDC: Handshake and Asynchronous FIFO](#3-multi-bit-cdc-handshake-and-asynchronous-fifo)
4. [Synchronous vs Asynchronous Reset](#4-synchronous-vs-asynchronous-reset)
5. [Power-Aware Basics: Clock Gating and Power Gating](#5-power-aware-basics-clock-gating-and-power-gating)

**Commonly Asked**
6. [SRAM vs DRAM](#6-sram-vs-dram)
7. [Memory Hierarchy](#7-memory-hierarchy)
8. [FIFO Concepts: Full/Empty Detection, Read/Write Pointers, Gray Code](#8-fifo-concepts-fullempty-detection-readwrite-pointers-gray-code)
9. [FIFO Depth Calculation](#9-fifo-depth-calculation)
10. [AMBA Interfaces: AXI and APB](#10-amba-interfaces-axi-and-apb)
11. [Setup/Hold Violation and Maximum Frequency Calculation](#11-setuphold-violation-and-maximum-frequency-calculation)
12. [TPU-Relevant Architecture Basics](#12-tpu-relevant-architecture-basics)
13. [PCIe Basics](#13-pcie-basics)

**Previously Asked**
14. [FSM Design — Moore vs Mealy](#14-fsm-design--moore-vs-mealy)
15. [STA Problem-Solving Round: Hold, Max Frequency, FIFO Depth, Metastability Combined](#15-sta-problem-solving-round-combined)

---

# 1. Combinational vs Sequential Circuits and Metastability

## Interview Question

> "What is the difference between combinational and sequential circuits? What is metastability?"

## Concept

**Combinational circuits** — output depends only on current inputs, no memory, no clock.

**Sequential circuits** — output depends on current inputs AND past state (memory), clocked.

**Metastability** — when a flip-flop samples data too close to its clock edge, violating setup or hold time, the output becomes unpredictable (neither 0 nor 1) for an unknown period.

## Problem / Motivation

Understanding this difference is fundamental to all digital design — every timing, CDC, and synchronization problem in silicon traces back to it. Metastability is the root cause of data corruption in clock-domain-crossing scenarios.

## Flow / Architecture

### Combinational Circuit
```
Input A ─┐
          ├─ Logic Gate ─── Output Y
Input B ─┘

Y = f(A, B)   [no clock, no state]
```

### Sequential Circuit
```
Input ─── Flip-Flop ─── Output
               ↑
             Clock
             
Output = f(Input, Previous State)
```

### Metastability
```
Data changes near clock edge
          ↓
Setup/Hold time violated
          ↓
Flip-flop cannot resolve to 0 or 1
          ↓
Output stays in undefined state
          ↓
Unpredictable behavior downstream
```

## Example

**Combinational:** Full adder — inputs A, B, Cin directly produce Sum and Cout with no clock.

**Sequential:** D flip-flop — output Q only changes on the clock edge, remembers the last state.

**Metastability:** In a synchronizer, if the async input changes within a nanosecond of the clock edge, the flip-flop may oscillate between 0 and 1 — "metastable state" — before eventually resolving. If it propagates into logic before resolving, it corrupts the entire downstream computation.

## Interview Answer

> "Combinational circuits produce outputs purely from current inputs with no memory. Sequential circuits have memory elements (flip-flops) and produce outputs based on both current inputs and stored state, clocked synchronously. Metastability occurs when a flip-flop samples data that changes too close to the clock edge, violating setup or hold time, causing the output to be temporarily indeterminate — neither a valid 0 nor a valid 1."

## Common Follow-up Questions

### Q1. Can metastability be completely eliminated?
No — it can only be reduced to an acceptably low probability using synchronizers, but never fully eliminated from asynchronous interfaces.

### Q2. What happens if metastability propagates into combinational logic?
The indeterminate voltage level causes all gates connected to that signal to produce unpredictable results — potentially corrupting an entire computation or causing two flip-flops downstream to "disagree" on the same value.

### Q3. Why do sequential circuits need setup and hold times?
The flip-flop's internal latch mechanism needs the data to be stable for a minimum time *before* (setup) and *after* (hold) the clock edge so it can reliably capture the correct value.

## Quick Revision

```
Combinational: Output = f(inputs only)       No clock, no memory
Sequential:    Output = f(inputs + state)    Clocked, has memory
Metastability: Setup/Hold violated → output undefined → use synchronizer
```

---

# 2. Single-bit CDC: 2-Flop Synchronizer

## Interview Question

> "What is a clock domain crossing (CDC) problem? How does a 2-flop synchronizer work, and why 2 flops instead of 1?"

## Concept

A **Clock Domain Crossing (CDC)** occurs when a signal generated in one clock domain (Clock A) is sampled by logic in a different clock domain (Clock B). Since the two clocks are asynchronous, the receiving flip-flop may sample the signal during a transition — causing metastability.

A **2-flop synchronizer** reduces (not eliminates) the probability of metastability propagating into the destination logic.

## Problem / Motivation

Any time a signal must travel between two clock domains, metastability risk exists. A single flip-flop resolves metastability but there's a non-zero probability it won't resolve in time for the next flip-flop's setup window. Adding a second flip-flop gives the signal one full additional clock cycle to resolve, dramatically reducing the probability of failure reaching the rest of the logic.

## Flow / Architecture

```
Clock Domain A          Clock Domain B
─────────────           ──────────────
                        
Async Signal ──► FF1 ──► FF2 ──► Synchronized Signal
               (clk_B)  (clk_B)
```

- **FF1:** samples the async signal; may become metastable.
- **FF2:** samples FF1's output; by the time FF2 samples, FF1 has had a full clock period to resolve, so the probability of FF2 also seeing metastability is vanishingly small.

## Why NOT Just 1 Flip-Flop?

```
With 1 FF:
Async Signal → FF1 → Logic
              (metastable output could reach logic within the same clock cycle)

With 2 FFs:
Async Signal → FF1 → FF2 → Logic
              (FF1 metastability resolves before FF2 samples, with high probability)
```

The MTBF (Mean Time Between Failures) increases exponentially with each additional synchronizer stage.

## Example

A button press in the user-input domain (12 MHz) must signal a processor core running at 400 MHz. A 2-flop synchronizer on the processor's 400 MHz clock domain ensures the button signal is cleanly captured before reaching any 400 MHz logic.

## Interview Answer

> "A CDC problem occurs when a signal crosses between two unrelated clock domains, risking metastability at the receiving flip-flop. A 2-flop synchronizer places two flip-flops in series, both clocked by the destination clock. The first flip-flop may become metastable, but the second flip-flop gives it a full clock period to resolve. This dramatically reduces the probability of metastability propagating into the rest of the logic. We use 2 flops instead of 1 because a single flip-flop doesn't provide enough settling time — metastability could still propagate. The 2-flop synchronizer does NOT eliminate metastability; it only reduces its probability."

## Common Follow-up Questions

### Q1. Does the 2-flop synchronizer work for multi-bit signals?
No — if multiple bits are synchronized independently through separate 2-flop synchronizers, they can arrive at different times (one resolves to the new value, another to the old), creating a corrupted intermediate state. Multi-bit CDC needs a different approach (see topic 3).

### Q2. When would you use 3 flops instead of 2?
At very high clock frequencies where even a full clock period isn't enough settling time — a 3-stage synchronizer provides two full clock periods for metastability resolution.

### Q3. Can you put the 2-flop synchronizer on the source side instead of the destination side?
No — the synchronizer must be on the **destination** clock domain, because it's the destination flip-flop that samples the signal and risks metastability.

## Quick Revision

```
CDC = signal crossing clock domains → metastability risk
2-flop synchronizer = both FFs on destination clock
FF1 may go metastable → FF2 gives FF1 time to resolve
Does NOT eliminate metastability, only reduces probability
Only works for SINGLE-bit signals
```

---

# 3. Multi-bit CDC: Handshake and Asynchronous FIFO

## Interview Question

> "Why can't a 2-flop synchronizer be used for multi-bit CDC? What are the solutions?"

## Concept

When multiple bits must cross a clock domain boundary together, individual 2-flop synchronizers on each bit create a **sampling skew** problem — different bits may resolve at different clock edges, producing a corrupted intermediate value that never existed in the source domain.

## Problem / Motivation

```
Source sends: 0101 1010 (valid data)
FF1 resolves bit[3:2] on cycle 1, bit[1:0] on cycle 2
Destination sees: 0110 ???? (garbage — mix of old and new)
```

Multi-bit CDC requires a mechanism that guarantees **all bits are captured atomically**.

## Solutions

### Solution 1: Handshake Protocol (for slow data or control signals)

```
Source Domain              Destination Domain
─────────────              ──────────────────
Data valid                 
REQ ──────────────────────► 2-flop sync ──► ACK generation
                            ↓
                           Sample Data (only when ACK received)
ACK ◄────────────────────── 2-flop sync ◄── ACK
```

The source sends REQ, destination syncs REQ, samples data, then sends ACK back. The source waits for ACK before changing data. Guarantees data is stable when sampled — but slow (multiple clock cycles of latency per transfer).

### Solution 2: Asynchronous FIFO (for high-throughput data)

```
Write Domain              Read Domain
────────────              ────────────
Write Pointer ──► Gray Code ──► 2-flop sync ──► Compare
                                               ↑
Read Pointer ──► Gray Code ──► 2-flop sync ──►
```

- Data is written and read at independent rates.
- Pointers use **Gray code** so only 1 bit changes at a time — safe to synchronize with a 2-flop synchronizer.
- Full/empty flags are generated by comparing synchronized pointers.

## Why Gray Code for Pointers?

```
Binary:  011 → 100  (3 bits change simultaneously — unsafe for single sync)
Gray:    010 → 110  (only 1 bit changes — safe for single sync)
```

If binary pointers were synchronized, multiple bits changing simultaneously could be sampled in a corrupt intermediate state. Gray code ensures at most 1 bit changes per increment.

## Comparison

| | Handshake | Async FIFO |
|---|---|---|
| Throughput | Low | High |
| Latency | High | Low |
| Complexity | Simple | Higher |
| Use case | Occasional control signals | Streaming data |

## Interview Answer

> "A 2-flop synchronizer cannot be used for multi-bit CDC because each bit synchronizes independently — different bits can resolve at different clock edges, producing a corrupted intermediate value. For slow control signals, a handshake protocol ensures the data is stable before being sampled. For high-throughput data, an asynchronous FIFO is used with Gray-coded read/write pointers, so only one bit changes at a time and is safe to synchronize with a standard 2-flop synchronizer."

## Common Follow-up Questions

### Q1. Why does only 1 bit changing matter for Gray code pointers?
Because even if the synchronizer sees a metastable state on that 1 bit, it resolves to either the old or new pointer value — both of which are valid — unlike binary where multiple simultaneous bit changes can produce an invalid intermediate pointer.

### Q2. What happens to data written to the FIFO during the synchronization latency?
The FIFO buffers it — that's exactly the purpose of the FIFO. It absorbs the rate/latency difference between the two clock domains.

### Q3. When would you choose handshake over async FIFO?
When data only needs to cross occasionally (e.g., a configuration register update) and the extra latency of handshaking is acceptable — async FIFOs add design complexity that isn't worth it for low-frequency transfers.

## Quick Revision

```
Multi-bit CDC problem: bits resolve at different times → corrupt intermediate value
Handshake: slow, simple, for control signals
Async FIFO: fast, for data streams, uses Gray-coded pointers
Gray code: only 1 bit changes per step → safe to 2-flop sync
```

---

# 4. Synchronous vs Asynchronous Reset

## Interview Question

> "What is the difference between synchronous and asynchronous reset? What are the tradeoffs?"

## Concept

**Synchronous reset** — reset is sampled on the clock edge, just like data. The flip-flop only resets when the clock arrives.

**Asynchronous reset** — reset takes effect immediately, regardless of the clock edge.

## Problem / Motivation

Reset strategy affects simulation behavior, timing analysis, FPGA/ASIC tool compatibility, and reliability. Choosing incorrectly can cause glitches, missed resets, or timing violations.

## Flow / Architecture

### Synchronous Reset
```systemverilog
always_ff @(posedge clk) begin
    if (reset)
        q <= 0;
    else
        q <= d;
end
```
Reset is part of the combinational logic feeding the flip-flop's D input. Only activates on the clock edge.

### Asynchronous Reset
```systemverilog
always_ff @(posedge clk or posedge reset) begin
    if (reset)
        q <= 0;
    else
        q <= d;
end
```
Reset directly clears the flip-flop immediately — no clock needed.

## Comparison

| | Synchronous Reset | Asynchronous Reset |
|---|---|---|
| Activation | On clock edge only | Immediately |
| Glitch sensitivity | Immune (clock filters glitches) | Sensitive to reset glitches |
| Reset pulse width | Must be ≥ 1 clock cycle | Can be very short |
| Timing analysis | Included in data path timing | Has separate recovery/removal timing |
| FPGA compatibility | Good | Good (most FPGAs support both) |
| Common in ASIC | Yes | Yes, but reset synchronizer often added |

## Example

**Synchronous reset problem:** if the clock stops (power saving mode), a synchronous reset cannot clear the flip-flop — the reset is never sampled.

**Asynchronous reset problem:** a short glitch on the reset line (e.g., due to ESD or noise) immediately resets the flip-flop, causing spurious behavior even when no real reset was intended.

**Best practice for ASIC:** assert asynchronously, de-assert synchronously (use a reset synchronizer) — this combines the ability to reset without a clock with the glitch-free release of reset.

## Interview Answer

> "Synchronous reset is sampled on the clock edge, so it's immune to glitches and integrates naturally into timing analysis, but requires the clock to be running. Asynchronous reset takes effect immediately regardless of the clock, useful when the clock may not be running, but is sensitive to glitches on the reset line. In ASICs, best practice is to assert reset asynchronously and de-assert it synchronously using a reset synchronizer."

## Common Follow-up Questions

### Q1. What is a reset synchronizer?
Two flip-flops in series on the destination clock domain with their D input tied to 1 and their asynchronous reset input tied to the global reset signal — this ensures the de-assertion of reset is synchronized to the local clock, preventing glitchy behavior.

### Q2. Why does asynchronous de-assertion of reset cause problems?
If reset is released at a random point in the clock cycle, the flip-flop may violate its recovery time (the equivalent of setup time for the reset signal), causing metastability at reset release.

### Q3. Which is more common in industry ASIC flows?
Both are used, but asynchronous assertion with synchronous de-assertion (reset synchronizer) is the most robust and widely accepted approach for large ASICs.

## Quick Revision

```
Synchronous reset:  clock edge dependent, glitch immune, requires active clock
Asynchronous reset: immediate, glitch sensitive, works without clock
Best practice:      assert async, de-assert sync (reset synchronizer)
```

---

# 5. Power-Aware Basics: Clock Gating and Power Gating

## Interview Question

> "What is clock gating and power gating? Why are they used?"

## Concept

Both are power-reduction techniques used in ASICs:

**Clock gating** — stops the clock from reaching flip-flops that don't need to toggle, eliminating dynamic power in those flops.

**Power gating** — shuts off the power supply to an entire block that isn't needed, eliminating both dynamic and static (leakage) power.

## Problem / Motivation

In modern chips (especially AI/ML accelerators like TPUs), power is a primary constraint. Flip-flops consume power every time they toggle — even when computing nothing useful. Entire IP blocks may be idle for long periods. Without power management, the chip would exceed thermal limits and battery/power budgets.

## Flow / Architecture

### Clock Gating
```
Clock ──► Clock Gate (EN) ──► Flip-Flop
              ↑
           Enable signal
           
When EN = 0: clock is stopped → flip-flop doesn't toggle → no dynamic power
When EN = 1: clock passes through → flip-flop operates normally
```

Uses an integrated clock gate (ICG) cell — typically a latch + AND gate — to prevent glitches on the gated clock.

### Power Gating
```
Power Supply
     ↓
Power Switch (PMOS/NMOS)
     ↓
IP Block (completely powered off)
     ↑
Sleep signal
```

When the block is sleeping, the power switch disconnects VDD, cutting off all leakage current.

## Comparison

| | Clock Gating | Power Gating |
|---|---|---|
| Eliminates | Dynamic power (switching) | Dynamic + Static (leakage) |
| State retained | Yes | No (must save/restore) |
| Wake-up latency | Very fast | Slow (power ramp-up) |
| Complexity | Low | High |
| Use case | Idle flip-flops | Long idle IP blocks |

## Interview Answer

> "Clock gating stops the clock from reaching flip-flops that don't need to switch, eliminating their dynamic switching power. Power gating cuts power to an entire IP block that isn't needed, eliminating both dynamic and static leakage power. Clock gating is simpler and retains state, while power gating removes leakage but requires saving and restoring state on wake-up."

## Common Follow-up Questions

### Q1. Why use a latch inside the clock gate cell instead of just an AND gate?
A plain AND gate on the clock could produce a glitch if the enable changes while the clock is high — the latch ensures enable is only sampled on the falling clock edge, guaranteeing a glitch-free gated clock.

### Q2. What's the risk of getting clock gating wrong?
If the enable signal has a glitch, it can produce a spurious clock edge and cause the flip-flop to incorrectly capture data — creating a functional bug that's very hard to debug in silicon.

### Q3. Why is power gating more complex?
The IP block must save its state to retention registers before power is removed and restore it when power returns — and power-up sequencing must be carefully controlled to avoid inrush currents and supply noise.

## Quick Revision

```
Clock Gating:  stops clock → removes dynamic power → state kept → fast
Power Gating:  cuts VDD → removes dynamic + leakage → state lost → slow
```

---

# 6. SRAM vs DRAM

## Interview Question

> "What is the difference between SRAM and DRAM? Where is each used?"

## Concept

**SRAM (Static RAM)** — stores each bit using 6 transistors in a cross-coupled latch. Holds data as long as power is on, no refresh needed.

**DRAM (Dynamic RAM)** — stores each bit using 1 transistor + 1 capacitor. Charge leaks over time, requires periodic refresh to maintain data.

## Problem / Motivation

Different parts of a chip (cache, main memory, accelerator scratchpads) have very different speed, area, and power requirements. SRAM and DRAM trade off these parameters differently, so the right choice depends on the use case.

## Comparison

| | SRAM | DRAM |
|---|---|---|
| Transistors per bit | 6T | 1T + 1C |
| Volatility | Volatile (needs power) | Volatile (needs power + refresh) |
| Refresh required | No | Yes |
| Speed | Fast (sub-nanosecond) | Slow (nanoseconds) |
| Area per bit | Large | Small |
| Cost per bit | High | Low |
| Power | Higher (active) | Lower (idle, due to small cell) |
| Typical use | Cache (L1/L2/L3), register files, scratchpads | Main memory (DDR), off-chip memory |

## Example

**CPU:**
- L1 cache: SRAM (fastest, smallest, closest to CPU core)
- L2/L3 cache: SRAM (larger, slightly slower)
- Main memory: DRAM (DDR4/DDR5 — large capacity, slower, cheap per bit)

**TPU:**
- On-chip scratchpad (HBM tile buffer): SRAM — fast access for matrix operands
- Off-chip high-bandwidth memory (HBM): DRAM — large capacity, high bandwidth

## Interview Answer

> "SRAM uses 6 transistors per bit in a cross-coupled latch — fast, no refresh needed, but large and expensive per bit, used for on-chip caches and scratchpads. DRAM uses 1 transistor and 1 capacitor per bit — smaller and cheaper per bit but requires periodic refresh and is slower, used for off-chip main memory."

## Common Follow-up Questions

### Q1. Why does DRAM need refresh and SRAM doesn't?
DRAM stores charge on a capacitor that leaks over time (~64ms refresh interval in standard DRAM). SRAM uses a cross-coupled inverter pair that actively holds its state as long as power is supplied — no charge leaks.

### Q2. What is HBM (High Bandwidth Memory) and is it SRAM or DRAM?
HBM is DRAM, but stacked in 3D using through-silicon vias (TSVs) and placed close to the processor die — giving much higher bandwidth than conventional DDR at lower power, critical for AI accelerators like TPUs.

### Q3. In a TPU, why not just use SRAM for everything?
SRAM is far too expensive and large per bit at the capacity needed for model weights and activations — DRAM provides the orders-of-magnitude more capacity required at a viable die area and cost.

## Quick Revision

```
SRAM: 6T, no refresh, fast, large/expensive → caches, scratchpads
DRAM: 1T+1C, refresh needed, slow, small/cheap → main memory
```

---

# 7. Memory Hierarchy

## Interview Question

> "Explain the memory hierarchy. Why do tradeoffs exist at each level?"

## Concept

The memory hierarchy is a layered system where each level trades speed, capacity, and cost to maximize overall system performance within practical constraints.

## Problem / Motivation

No single memory technology is simultaneously fast, large, and cheap. The hierarchy exploits **locality of reference** (programs repeatedly access the same small working set of data) — keeping frequently used data in fast, small, expensive memory, and the rest in slow, large, cheap memory.

## Flow / Architecture

```
Register (fastest, smallest)
      ↓
L1 Cache (SRAM, ~32KB, <1ns)
      ↓
L2 Cache (SRAM, ~256KB, ~4ns)
      ↓
L3 Cache (SRAM, ~8–32MB, ~10ns)
      ↓
Main Memory DRAM (GBs, ~60ns)
      ↓
Storage SSD/HDD (TBs, microseconds–milliseconds)
```

## Tradeoffs at Each Level

| Level | Technology | Speed | Capacity | Cost/bit |
|---|---|---|---|---|
| Register | FF array | ~0.1ns | Bytes | Very high |
| L1 Cache | SRAM | <1ns | 32–64KB | High |
| L2 Cache | SRAM | ~4ns | 256KB–1MB | High |
| L3 Cache | SRAM | ~10ns | 8–64MB | Medium |
| Main Memory | DRAM | ~60ns | GBs | Low |
| Storage | Flash/HDD | ms–µs | TBs | Very low |

## Why the Hierarchy Works

Programs exhibit **temporal locality** (reuse the same data recently accessed) and **spatial locality** (access data near recently accessed data). The L1/L2 cache captures this working set, hiding DRAM's 60ns latency for most accesses.

## Interview Answer

> "The memory hierarchy exists because no single memory is simultaneously fast, large, and cheap. Registers are fastest but tiny. SRAM caches are fast but expensive per bit. DRAM is cheap and large but slow. The hierarchy exploits locality of reference — frequently accessed data stays in the fast, small levels while the bulk of data remains in slower, larger levels."

## Common Follow-up Questions

### Q1. What is a cache miss and why does it matter?
When requested data isn't in the cache, the processor must fetch it from a slower level (L2, L3, or DRAM), stalling for many cycles. Minimizing cache misses is critical to processor performance.

### Q2. How does a TPU's memory hierarchy differ from a CPU's?
A TPU doesn't have a general-purpose cache hierarchy — instead it has a large on-chip scratchpad (SRAM) that the compiler/programmer explicitly manages, plus HBM for weights and activations. This trades flexibility for higher sustained memory bandwidth.

### Q3. What is cache coherency and why is it a problem?
In multi-core systems, each core has its own L1 cache — if two cores cache the same memory location and one writes to it, the other's cache line becomes stale. Cache coherency protocols (MESI, MOESI) solve this.

## Quick Revision

```
Register → L1 → L2 → L3 → DRAM → Storage
Faster ← → Larger
Smaller ← → Cheaper
Hierarchy works because of locality of reference
```

---

# 8. FIFO Concepts: Full/Empty Detection, Read/Write Pointers, Gray Code

## Interview Question

> "How does a synchronous FIFO detect full and empty conditions? Why is gray code used for pointers in an asynchronous FIFO?"

## Concept

A FIFO (First-In, First-Out) buffer uses read and write pointers to track where to write next and where to read next. The full and empty conditions are derived by comparing these pointers.

## Problem / Motivation

A FIFO decouples producers and consumers that run at different rates or with bursty traffic. Correct full/empty detection is critical — a missed full causes overflow (data lost/corrupted), and a missed empty causes underflow (reading invalid data). In asynchronous FIFOs (cross-clock-domain), the pointer comparison must be safe across clock domains.

## Flow / Architecture

### Synchronous FIFO
```
Write Pointer (WP) ──► Memory Array ◄── Read Pointer (RP)
                           ↑
                    Data stored here

Full:  (WP + 1) == RP    (next write would overwrite unread data)
Empty: WP == RP           (nothing written that hasn't been read)
```

### Read/Write Pointer Behavior
```
Initial: WP = 0, RP = 0 → EMPTY
Write:   WP advances → WP = 1, 2, 3...
Read:    RP advances → RP = 1, 2, 3...
Full:    WP wraps around to RP position
```

Pointers are typically 1 bit wider than the address space to distinguish full from empty when both pointers are equal modulo the FIFO size.

### Gray Code for Asynchronous FIFO Pointers

```
Binary count:  000 → 001 → 010 → 011 → 100
               (multiple bits change at transitions 001→010, 011→100)

Gray code:     000 → 001 → 011 → 010 → 110
               (only 1 bit changes at every transition)
```

When a binary pointer is synchronized across clock domains, multiple bits changing simultaneously can be sampled in a corrupt intermediate state. Gray code guarantees only 1 bit changes per increment — safe for 2-flop synchronization.

## Interview Answer

> "A synchronous FIFO uses write and write pointer comparison for full and empty detection. Empty is when both pointers are equal. Full is when the write pointer plus one equals the read pointer. In asynchronous FIFOs, pointers are Gray-coded before crossing clock domains because Gray code ensures only one bit changes per increment — this makes it safe to synchronize with a 2-flop synchronizer, avoiding the corrupted intermediate states that binary pointers would produce."

## Common Follow-up Questions

### Q1. Why make pointers 1 bit wider than the address space?
If the FIFO has depth N, pointers wrap at 2N (not N). This allows distinguishing between full (WP and RP differ only in the MSB) and empty (WP == RP exactly), without needing a separate count register.

### Q2. Can you use a counter instead of pointers?
A separate count register works for synchronous FIFOs but adds a register that must be updated atomically. Pointer-based designs are cleaner and extend naturally to async FIFOs.

### Q3. What's the worst case for a FIFO? When does it need to be deepest?
When the write rate exceeds the read rate for the maximum possible burst duration — the FIFO must hold the maximum accumulated data without overflowing.

## Quick Revision

```
Empty: WP == RP
Full:  (WP + 1) == RP   (or MSB differs, rest equal — depends on implementation)
Pointers 1 bit wider than address space → distinguish full from empty
Gray code: 1 bit changes per step → safe for 2-flop sync across clock domains
```

---

# 9. FIFO Depth Calculation

## Interview Question

> "Given a producer writing at 4 words/cycle and a consumer reading at 2 words/cycle with a burst of 10 cycles, what is the minimum FIFO depth?"

## Concept

The FIFO must be deep enough to absorb the maximum data accumulation without overflowing — the difference between what has been written and what has been read at the worst-case moment.

## Problem / Motivation

If the FIFO is too shallow, overflow occurs — data is lost or corrupted. If it's too deep, silicon area is wasted. Sizing it correctly requires calculating the maximum burst accumulation.

## Flow / Architecture

```
Producer (writes faster)
      ↓
FIFO (absorbs the difference)
      ↓
Consumer (reads slower)
```

## Solution

**Given:**
- Write rate = 4 words/cycle
- Read rate = 2 words/cycle
- Burst duration = 10 cycles

**Step 1 — Accumulation rate:**
```
Accumulation = Write rate - Read rate
             = 4 - 2
             = 2 words/cycle
```

**Step 2 — Maximum accumulated data:**
```
Maximum data = Accumulation × Burst duration
             = 2 × 10
             = 20 words
```

**Step 3 — FIFO depth:**
- Minimum: **20 entries**
- Practical: **32 entries** (round up to next power of two)

**Why round up to a power of two?**
- Pointer arithmetic and wraparound logic is simpler with power-of-two depths.
- Provides a safety margin for timing/latency overhead.
- Gray code pointer generation is natural at power-of-two sizes.

## Interview Answer

> "The accumulation rate is write rate minus read rate: 4 minus 2 equals 2 words per cycle. Over a 10-cycle burst, the maximum accumulated data is 2 times 10 equals 20 words. The minimum FIFO depth is 20 entries. In practice, round up to 32 entries for a power-of-two size, which simplifies pointer logic and provides a safety margin."

## Common Follow-up Questions

### Q1. What if the burst duration isn't specified?
You need to analyze the worst-case scenario from the system architecture — typically defined by the upstream producer's maximum burst length or the downstream consumer's maximum stall time.

### Q2. Does the FIFO depth change if the read rate is faster?
If the consumer reads faster than the producer writes, the FIFO never fills — but it can underflow (run empty) if the producer pauses. In that case, you'd need backpressure signaling rather than a deep FIFO.

### Q3. What safety margin is typically added?
Industry practice varies — some add 10–20% above the calculated depth, others simply round up to the next power of two and rely on that margin.

## Quick Revision

```
FIFO Depth = (Write Rate - Read Rate) × Burst Duration
           = (4 - 2) × 10 = 20 → round up to 32
```

---

# 10. AMBA Interfaces: AXI and APB

## Interview Question

> "Explain the AXI valid/ready handshake. What is the difference between AXI and APB? AXI-Lite vs AXI3?"

## Concept

**AMBA (Advanced Microcontroller Bus Architecture)** is ARM's family of on-chip bus protocols. AXI (Advanced eXtensible Interface) and APB (Advanced Peripheral Bus) are the two most commonly tested.

## Problem / Motivation

On-chip communication between IP blocks needs a standardized protocol — otherwise every IP would have a custom interface. AMBA provides this standard, enabling plug-and-play IP integration.

## AXI Valid/Ready Handshake

```
Master          Slave
──────          ──────
VALID ──────────────►  (master: I have valid data/request)
      ◄────────── READY  (slave: I am ready to accept)

Transfer occurs ONLY when VALID = 1 AND READY = 1 simultaneously
```

**Key rule:** Neither side can deassert VALID or READY based on what the other side is doing, until a transfer occurs. Master must hold VALID until READY is seen.

### AXI Channels
AXI has 5 independent channels:
```
Write Address (AW): address for write
Write Data (W):     write data
Write Response (B): write response
Read Address (AR):  address for read
Read Data (R):      read data + response
```

## AXI vs APB

| | AXI | APB |
|---|---|---|
| Performance | High | Low |
| Burst support | Yes | No |
| Outstanding transactions | Multiple | 1 at a time |
| Channels | 5 | 1 |
| Complexity | High | Simple |
| Use case | Memory, DMA, accelerators | UART, GPIO, slow peripherals |

## AXI-Lite vs AXI3/AXI4

| | AXI-Lite | AXI3/AXI4 |
|---|---|---|
| Purpose | Simple register access | High-throughput data transfer |
| Burst | No (1 transfer per transaction) | Yes (up to 256 beats in AXI4) |
| Outstanding | Limited | Multiple |
| Typical use | CSR (control/status registers) | Memory, DMA, accelerators |

## Interview Answer

> "The AXI handshake uses VALID and READY signals — a transfer occurs only when both are asserted simultaneously. AXI supports burst transfers, multiple outstanding transactions, and five independent channels, making it suitable for high-performance memory and DMA. APB is simpler with a single channel, no burst support, and is used for low-speed peripherals like UART and GPIO. AXI-Lite is a simplified AXI for register access with no burst support, while AXI3/AXI4 adds full burst and multiple outstanding transactions for high throughput."

## Common Follow-up Questions

### Q1. Why does AXI have separate address and data channels?
This decoupling allows pipelining — a master can send the next address while data from the previous transaction is still being transferred, increasing throughput.

### Q2. What happens if VALID is asserted but READY never comes?
The transaction is stalled — the master must hold VALID high. This is the backpressure mechanism; it's expected behavior when the slave is busy, not an error.

### Q3. In a TPU, why would AXI be preferred over APB for memory access?
TPU matrix engines need to stream large tensors from on-chip memory at maximum bandwidth — AXI's burst support and multiple outstanding transactions provide the throughput necessary for this; APB's single-transaction model would create a massive bottleneck.

## Quick Revision

```
AXI handshake: VALID + READY both = 1 → transfer occurs
AXI: bursts, multiple outstanding, 5 channels → high performance
APB: simple, single channel, no burst → slow peripherals
AXI-Lite: AXI without bursts → register access
```

---

# 11. Setup/Hold Violation and Maximum Frequency Calculation

## Interview Question

> "Given a timing path with propagation delays and setup/hold times, identify violations and calculate maximum frequency."

## Concept

**Setup time (Tsu)** — data must be stable for at least this long *before* the clock edge.

**Hold time (Th)** — data must remain stable for at least this long *after* the clock edge.

**Setup violation** — data arrives too late (not stable before clock edge).

**Hold violation** — data changes too early (doesn't stay stable after clock edge).

## Problem / Motivation

Timing analysis determines the maximum safe operating frequency of a chip. Missing a setup time means the flip-flop captures invalid data. Missing a hold time also captures invalid data — but hold violations cannot be fixed by slowing down the clock; they require adding delay on the data path.

## Flow / Architecture

```
Launch FF          Combinational Logic          Capture FF
    ↓ Tcq                ↓ Tlogic                    ↓ 
Clock edge → Data valid ──────────────── Data arrives → must be stable Tsu before clock
                                                         must remain stable Th after clock
```

## Setup Violation and Maximum Frequency

**Condition for no setup violation:**
```
Tcq + Tlogic + Tsu ≤ Tclk
```

**Minimum clock period:**
```
Tclk ≥ Tcq + Tlogic + Tsu
```

**Maximum frequency:**
```
Fmax = 1 / (Tcq + Tlogic + Tsu)
```

**Example:**
```
Tcq = 1 ns, Tlogic = 5 ns, Tsu = 1 ns
Tclk ≥ 1 + 5 + 1 = 7 ns
Fmax = 1 / 7ns ≈ 142 MHz
```

## Hold Violation

**Condition for no hold violation:**
```
Tcq + Tlogic(min) ≥ Th
```
(Data must stay stable long enough after the clock edge.)

**Hold violation example:**
```
Tcq = 0.2 ns, Tlogic(min) = 0.1 ns, Th = 0.5 ns
Data arrival = 0.2 + 0.1 = 0.3 ns
Required     = 0.5 ns
0.3 < 0.5 → Hold violation
```

**Fix for hold violation:** add delay buffers in the data path to increase Tlogic(min).

**Note:** hold violations cannot be fixed by changing the clock frequency — only by adding delay.

## Interview Answer

> "For maximum frequency, the minimum clock period must be at least the sum of clock-to-Q delay, combinational logic delay, and setup time. Fmax equals 1 divided by that sum. For hold violations, the data arrival time — clock-to-Q plus minimum logic delay — must be greater than or equal to the hold time. If it isn't, a hold violation exists and must be fixed by adding delay cells, not by slowing the clock."

## Common Follow-up Questions

### Q1. Why can't you fix a hold violation by slowing the clock?
A hold violation means data changes too soon *after* the clock edge. Slowing the clock only increases the period — it doesn't change how quickly data propagates through combinational logic after the launch edge.

### Q2. What is clock skew and how does it affect setup/hold?
Positive clock skew (clock arrives later at the capture FF) helps setup but hurts hold. Negative clock skew helps hold but hurts setup. STA tools account for this.

### Q3. What is the difference between STA and simulation for timing verification?
STA analyzes all paths exhaustively and mathematically, without needing test vectors — much faster and complete. Simulation checks timing only for the specific scenarios simulated — faster setup but not exhaustive.

## Quick Revision

```
Setup: Tclk ≥ Tcq + Tlogic + Tsu    → Fmax = 1/(Tcq + Tlogic + Tsu)
Hold:  Tcq + Tlogic(min) ≥ Th       → fix by adding delay (not clock change)
```

---

# 12. TPU-Relevant Architecture Basics

## Interview Question

> "Describe the high-level data path of a TPU system — from host CPU to the TPU compute engine."

## Concept

You don't need TPU internals — just the data path shape: how the host communicates with the TPU, where memory sits, and what the compute engine does at a conceptual level.

## Problem / Motivation

This role is specifically for the TPU team — showing you understand what a TPU does and where it sits in the system signals genuine interest and contextual awareness, even without knowing micro-architectural details.

## Flow / Architecture

```
Host CPU
    ↓
PCIe (host interface — sends commands, model weights, activations)
    ↓
TPU Chip
    ├── On-chip HBM Interface (High-Bandwidth Memory — model weights, large tensors)
    ├── On-chip SRAM Scratchpad (fast tile buffers — operands for current computation)
    └── Compute Engine (Systolic Array — performs matrix multiply/accumulate operations)
         ↓
    Results returned via PCIe to Host CPU
```

## Key Concepts

**Host CPU ↔ PCIe ↔ TPU:**
- The host CPU sends inference requests, model weights, and input activations to the TPU over PCIe.
- The TPU returns results (output activations) to the host.

**HBM (High Bandwidth Memory):**
- DRAM stacked close to the TPU die via through-silicon vias.
- Stores model weights and large activation tensors.
- Much higher bandwidth than DDR (hundreds of GB/s) — critical since ML workloads are memory-bandwidth-bound.

**On-chip SRAM Scratchpad:**
- Fast on-chip memory (sub-nanosecond access).
- Tiles of weights/activations are loaded here before the systolic array consumes them.
- Explicitly managed by the compiler, not cached automatically.

**Systolic Array (Compute Engine):**
- A grid of MAC (Multiply-Accumulate) units.
- Data flows through the array rhythmically (like a heartbeat — hence "systolic").
- Designed for high-throughput matrix multiplication — the core operation in neural networks (GEMM).
- Achieves high compute utilization by reusing data as it flows through the array, reducing memory bandwidth pressure.

## Interview Answer

> "In a TPU system, the host CPU communicates with the TPU over PCIe, sending model weights, input activations, and inference commands. Inside the TPU, large tensors are stored in HBM — high bandwidth DRAM stacked close to the die. Tiles of data are loaded from HBM into on-chip SRAM scratchpads before being fed to the compute engine, which is a systolic array performing matrix multiply-accumulate operations. Results are returned to the host over PCIe. The systolic array achieves high efficiency by reusing data as it flows through the array, reducing the memory bandwidth requirements for compute."

## Common Follow-up Questions

### Q1. Why is a systolic array better than a general-purpose CPU for matrix multiply?
A CPU must explicitly load/store operands for every multiply-add. A systolic array passes data between adjacent MACs without going back to memory for each operation — dramatically increasing arithmetic intensity (compute per byte of memory traffic).

### Q2. Why does a TPU use an explicit scratchpad instead of a cache?
ML workloads have predictable, regular access patterns that the compiler can manage explicitly — a cache's hardware prediction adds latency and area overhead without benefit, while an explicit scratchpad gives the compiler full control to maximize reuse.

### Q3. What's the role of HBM vs on-chip SRAM in the TPU's memory hierarchy?
HBM provides large capacity at high bandwidth for the full model; on-chip SRAM provides ultra-fast access for the active tile being computed. The compiler streams tiles from HBM to SRAM to overlap compute with memory access (double-buffering).

## Quick Revision

```
Host CPU → PCIe → TPU
TPU: HBM (weights/activations) → SRAM Scratchpad (active tile) → Systolic Array (GEMM)
Systolic array: data flows through MACs rhythmically → high reuse → high throughput
```

---

# 13. PCIe Basics

## Interview Question

> "Explain PCIe at a conceptual level — root complex, endpoint, lanes, and link training."

## Concept

**PCIe (Peripheral Component Interconnect Express)** is the high-speed serial interconnect used to connect the host CPU to peripherals (GPU, TPU, NVMe SSD, NIC).

## Problem / Motivation

Older parallel buses (like PCI) couldn't scale to the bandwidth needed by modern accelerators and storage. PCIe provides a scalable, high-bandwidth, low-latency serial interconnect with a simple point-to-point topology.

## Flow / Architecture

```
Root Complex (CPU side)
      ↕  (PCIe Link)
Endpoint (Device side — GPU, TPU, NVMe SSD)
```

### Root Complex
- Sits on the CPU/SoC side.
- Manages the PCIe fabric, handles memory-mapped I/O, and issues/receives transactions.
- Think of it as the PCIe "host controller" on the CPU.

### Endpoint
- The peripheral device (GPU, TPU, NVMe SSD, NIC) connected to the root complex.
- Responds to read/write requests from the root complex.

### Lanes
- Each PCIe lane = 1 differential pair for TX + 1 differential pair for RX (full duplex).
- Link width: x1, x4, x8, x16 (number of lanes).
- Bandwidth scales linearly: PCIe Gen4 x16 = ~32 GB/s.

### Link Training
- Before any data transfer, PCIe goes through link training — both sides negotiate lane count, speed (Gen1/2/3/4/5), and equalization settings.
- Completely automatic and transparent to software.

```
Power-on
    ↓
Link Training (auto-negotiate speed, lanes, equalization)
    ↓
Link Up (data transfer begins)
```

## Interview Answer

> "PCIe is a high-speed serial interconnect connecting the host CPU (root complex) to devices (endpoints) such as GPUs, TPUs, and NVMe SSDs. Each PCIe lane is a full-duplex differential pair. Link width scales from x1 to x16, with bandwidth scaling proportionally. On power-up, PCIe performs link training automatically — both sides negotiate speed, lane count, and equalization before data transfer begins. In a TPU system, the TPU is a PCIe endpoint connected to the host CPU root complex."

## Common Follow-up Questions

### Q1. What generation of PCIe is commonly used with AI accelerators?
PCIe Gen4 (x16 = ~32 GB/s) and Gen5 (x16 = ~64 GB/s) are now common for GPU/TPU connections — enough bandwidth for model weight streaming from host memory.

### Q2. What is MMIO and how does it relate to PCIe?
Memory-Mapped I/O — the PCIe device's registers and memory are mapped into the host CPU's address space, allowing the CPU to read/write them with ordinary load/store instructions. The PCIe root complex translates these into PCIe TLPs (Transaction Layer Packets).

### Q3. What is PCIe peer-to-peer communication?
Two PCIe endpoints (e.g., two GPUs) communicating directly without going through the CPU/host memory — reduces latency and CPU overhead for GPU-to-GPU or TPU-to-TPU data transfers.

## Quick Revision

```
Root Complex = CPU side (host controller)
Endpoint     = Device side (GPU, TPU, SSD)
Lane         = 1 TX + 1 RX differential pair (full duplex)
Link width   = x1, x4, x8, x16 (bandwidth scales with width)
Link training = auto-negotiate speed + lanes on power-up
```

---

# 14. FSM Design — Moore vs Mealy

## Interview Question

> "Design a Finite State Machine. What is the difference between Moore and Mealy machines?"

## Concept

An **FSM (Finite State Machine)** has a finite number of states, transitions between states based on inputs, and outputs.

**Moore machine** — outputs depend only on the current state.
**Mealy machine** — outputs depend on current state AND current inputs.

## Problem / Motivation

FSMs are the fundamental building block of digital control logic — controllers, protocol sequencers, traffic lights, memory arbiters, and USB/AXI state machines are all FSMs. Interviewers test FSM design to assess digital logic depth.

## Flow / Architecture

### Moore Machine
```
Input → Next State Logic → Current State (register) → Output Logic → Output
                              (clock)
Output = f(state only)
```

### Mealy Machine
```
Input → Next State Logic → Current State (register) → Output Logic → Output
  ↓                           (clock)                    ↑
  └──────────────────────────────────────────────────────┘
Output = f(state, input)
```

## Example: Traffic Light FSM (Moore)

**States:** RED, GREEN, YELLOW
**Output:** light color (based on state only)

```
State Diagram:
RED ──(timer)──► GREEN ──(timer)──► YELLOW ──(timer)──► RED

Outputs:
RED:    red_light = 1, green_light = 0, yellow_light = 0
GREEN:  red_light = 0, green_light = 1, yellow_light = 0
YELLOW: red_light = 0, green_light = 0, yellow_light = 1
```

## Verilog Implementation (Moore)

```systemverilog
module traffic_light_fsm (
    input clk, reset, timer_expired,
    output reg red, green, yellow
);

typedef enum logic [1:0] {
    RED_ST   = 2'b00,
    GREEN_ST = 2'b01,
    YELLOW_ST= 2'b10
} state_t;

state_t current_state, next_state;

// State register
always_ff @(posedge clk or posedge reset) begin
    if (reset)
        current_state <= RED_ST;
    else
        current_state <= next_state;
end

// Next state logic
always_comb begin
    case (current_state)
        RED_ST:    next_state = timer_expired ? GREEN_ST  : RED_ST;
        GREEN_ST:  next_state = timer_expired ? YELLOW_ST : GREEN_ST;
        YELLOW_ST: next_state = timer_expired ? RED_ST    : YELLOW_ST;
        default:   next_state = RED_ST;
    endcase
end

// Output logic (Moore: output from state only)
always_comb begin
    red = 0; green = 0; yellow = 0;
    case (current_state)
        RED_ST:    red    = 1;
        GREEN_ST:  green  = 1;
        YELLOW_ST: yellow = 1;
    endcase
end

endmodule
```

## Moore vs Mealy Comparison

| | Moore | Mealy |
|---|---|---|
| Output depends on | State only | State + Input |
| Output changes | Only on state change (clock) | Can change with input (combinational) |
| Number of states | Usually more states needed | Fewer states needed |
| Output glitches | No (registered) | Possible (combinational) |
| Latency | 1 extra clock cycle vs Mealy | Responds immediately to input |

## Interview Answer

> "A Moore FSM produces outputs based only on the current state — outputs change only on clock edges, so they're glitch-free. A Mealy FSM produces outputs based on both state and current inputs — outputs can change combinationally with inputs, requiring fewer states but potentially producing glitches. For the FSM implementation, I separate it into three always blocks: state register (sequential), next-state logic (combinational), and output logic (combinational for Moore based on state only)."

## Common Follow-up Questions

### Q1. Why separate next-state logic, state register, and output logic into three blocks?
It follows the standard 3-block FSM template that maps cleanly to synthesis tools, avoids unintended latches, and is the most readable/maintainable style for review and hand-off.

### Q2. When would you choose Mealy over Moore?
When minimizing state count or achieving lower latency matters more than glitch-free outputs — e.g., a simple handshake acknowledgment that can respond immediately to an input without waiting for the next clock.

### Q3. What is state encoding and why does it matter?
How state values are represented in binary: binary encoding (compact), one-hot encoding (faster, fewer glitches, good for FPGAs), and Gray code encoding (low power, only 1 bit changes per transition). The choice affects area, power, and speed.

## Quick Revision

```
Moore: output = f(state)         → glitch-free, more states
Mealy: output = f(state, input)  → immediate response, fewer states

3-block template:
1. State register   (always_ff)
2. Next-state logic (always_comb)
3. Output logic     (always_comb)
```

---

# 15. STA Problem-Solving Round: Combined

## Interview Question

> "A peer-set ASIC/DV candidate reported a full STA round covering hold violations, max frequency calculation, FIFO depth, and metastability in one session — be ready for all four in a single answer."

## Combined Interview Answer

> "In an STA round, I first identify whether the issue is setup or hold by checking data arrival time relative to the clock edge. For a hold violation, I verify that clock-to-Q plus minimum logic delay meets the hold time requirement — if it doesn't, I add delay cells to the data path. For maximum frequency, I calculate the minimum clock period as the sum of clock-to-Q, logic delay, and setup time, then take the reciprocal. For FIFO sizing, I subtract the read rate from the write rate to get the accumulation rate, multiply by burst duration, and round up to a power of two. For CDC issues, I identify metastability and use a 2-flop synchronizer for single-bit signals — noting it reduces, not eliminates, the probability of failure."

## Problem 1: Hold Violation

**Given:** Tcq = 0.2 ns, Tlogic(min) = 0.1 ns, Th = 0.5 ns

```
Data arrival = Tcq + Tlogic(min) = 0.2 + 0.1 = 0.3 ns
Required     = Th = 0.5 ns
0.3 < 0.5 → Hold violation exists
Fix: add delay cells to data path
```

## Problem 2: Maximum Frequency

**Given:** Tcq = 1 ns, Tlogic = 5 ns, Tsu = 1 ns

```
Tclk ≥ Tcq + Tlogic + Tsu = 1 + 5 + 1 = 7 ns
Fmax = 1 / 7 ns ≈ 142 MHz
```

## Problem 3: FIFO Depth

**Given:** Write rate = 4, Read rate = 2, Burst = 10 cycles

```
Accumulation = 4 - 2 = 2 words/cycle
Max data     = 2 × 10 = 20 words
FIFO depth   = 32 entries (round up to power of two)
```

## Problem 4: Metastability

```
Signal crosses Clock Domain A → Clock Domain B
Receiving FF samples near clock edge → setup/hold violation → metastable state
Solution: 2-flop synchronizer on destination clock
FF1: may go metastable
FF2: provides one full clock period settling time
Result: dramatically reduced probability of metastability propagating
Note: does NOT eliminate metastability, only reduces probability
```

## Quick Revision Table

| Topic | Formula / Rule |
|---|---|
| Hold Violation | Data arrival < Th → violation → add delay (not clock change) |
| Max Frequency | Fmax = 1 / (Tcq + Tlogic + Tsu) |
| FIFO Depth | (Write Rate - Read Rate) × Burst Duration → power of two |
| Metastability | CDC → 2-flop sync → reduces (not eliminates) probability |
