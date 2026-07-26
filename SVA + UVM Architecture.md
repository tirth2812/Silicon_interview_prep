# UVM Architecture & SVA — Interview Quick Revision Sheet

## Table of Contents

**UVM Architecture**
1. [What is UVM?](#1-what-is-uvm)
2. [UVM Hierarchy](#2-uvm-hierarchy)
3. [UVM Transaction Flow](#3-uvm-transaction-flow)
4. [uvm_object vs uvm_component](#4-uvm_object-vs-uvm_component)
5. [UVM Phases](#5-uvm-phases)

**SystemVerilog Assertions (SVA)**
6. [SVA Implication (Antecedent / Consequent)](#6-sva-implication-antecedent--consequent)
7. [assert vs assume vs cover](#7-assert-vs-assume-vs-cover)
8. [SVA Coding Question — req/ack within N cycles](#8-sva-coding-question--reqack-within-n-cycles)

[Quick Revision Tricks](#quick-revision-tricks)

---

## 1. What is UVM?

**UVM (Universal Verification Methodology)** is a SystemVerilog-based verification framework used to build reusable testbenches.

**Basic idea:** instead of one big Verilog `initial` block, UVM splits the testbench into reusable, hierarchical components.

---

## 2. UVM Hierarchy

```
uvm_test
    ↓
uvm_env
    ↓
uvm_agent
    ↓
 ┌──────────┬──────────┬──────────┐
 ↓          ↓          ↓
Sequencer  Driver    Monitor
                         ↓
                    Scoreboard
```

### uvm_test
Top-level test. **Purpose:** starts the test, creates the environment, configures test settings.
```verilog
class my_test extends uvm_test;
endclass
```

### uvm_env
Environment container. **Purpose:** holds agents and organizes the testbench.
```verilog
class my_env extends uvm_env;
endclass
```

### uvm_agent
Groups verification components for one interface.
```
Agent
 ├── Sequencer
 ├── Driver
 └── Monitor
```

### uvm_sequencer
Controls transaction flow — receives transactions from a sequence and sends them to the driver.

### uvm_driver
Converts transactions into DUT signals.
```
Transaction → Driver → DUT signals
```
**Example:**
```
Transaction:  Address = 100, Data = 50
Driver output: addr_bus = 100, data_bus = 50
```

### uvm_monitor
Observes DUT signals — the reverse of the driver.
```
DUT signals → Monitor → Transaction
```

### Scoreboard
Checks correctness.
```
Expected result = 30
Actual result   = 30
→ PASS
```

---

## 3. UVM Transaction Flow

**Example — ALU ADD operation:** `A = 10`, `B = 20`, `Operation = ADD`

| Step | Sender | Receiver | What happens |
|---|---|---|---|
| 1 | Test | Sequence | Starts stimulus |
| 2 | Sequence | Transaction | Creates transaction object |
| 3 | Sequence | Sequencer | Sends transaction |
| 4 | Sequencer | Driver | Gives transaction |
| 5 | Driver | DUT | Converts transaction to signals |
| 6 | DUT | Monitor | Produces output signals |
| 7 | Monitor | Scoreboard | Converts signals back to transaction |
| 8 | Scoreboard | Compare | Expected vs Actual |

**Flow diagram:**
```
Sequence → Transaction Object → Sequencer → Driver → DUT → Monitor → Scoreboard
```

---

## 4. uvm_object vs uvm_component

| | `uvm_object` | `uvm_component` |
|---|---|---|
| Role | Data holder | Testbench structural block |
| Hierarchy | No | Yes |
| Phases | No | Yes |
| Lifetime | Transient (created/used as needed) | Lives throughout simulation |
| Creation | `new()` | Typically `type_id::create()` |
| Examples | Sequence, Transaction | Driver, Monitor, Agent, Environment |

**uvm_object example:**
```verilog
class packet extends uvm_object;
endclass
```

**uvm_component example:**
```verilog
class my_driver extends uvm_driver;
endclass
```

---

## 5. UVM Phases

Phases control the lifetime of the testbench.

```
build_phase → connect_phase → run_phase → check_phase → report_phase
```

| Phase | Purpose | Direction | Why |
|---|---|---|---|
| `build_phase` | Create components | Top → Down | Parent creates child |
| `connect_phase` | Connect already-created components | Bottom → Up | Everything must exist before connecting |
| `run_phase` | Actual simulation activity | — | Contains delays, uses `task` |
| `check_phase` | Check results | — | Compares expected vs actual |
| `report_phase` | Final report | — | Prints pass/fail summary |

### build_phase
**Example:**
```verilog
env_handle = my_env::type_id::create("env", this);
```
**Direction:** Top → Down
```
test → env → agent → driver
```

### connect_phase
**Example:**
```
Sequencer -------- Driver
Monitor   -------- Scoreboard
```
**Direction:** Bottom → Up

### run_phase
**Example:** driver drives DUT; monitor observes DUT.
Uses a `task` (not a `function`) because it contains delays:
```verilog
task run_phase(uvm_phase phase);
```

### check_phase
**Example:**
```
Expected = 10
Actual   = 10
→ PASS
```

### report_phase
**Example:**
```
TEST PASSED
ERRORS = 0
```

### Full UVM Phase Example
```verilog
class my_component extends uvm_component;

    function new(string name = "my_component",
                 uvm_component parent = null);
        super.new(name, parent);
    endfunction

    function void build_phase(uvm_phase phase);
        super.build_phase(phase);
        $display("BUILD PHASE");
    endfunction

    function void connect_phase(uvm_phase phase);
        super.connect_phase(phase);
        $display("CONNECT PHASE");
    endfunction

    task run_phase(uvm_phase phase);
        phase.raise_objection(this);
        $display("RUN PHASE");
        #10;
        phase.drop_objection(this);
    endtask

    function void check_phase(uvm_phase phase);
        super.check_phase(phase);
        $display("CHECK PHASE");
    endfunction

    function void report_phase(uvm_phase phase);
        super.report_phase(phase);
        $display("REPORT PHASE");
    endfunction

endclass
```

---

## 6. SVA Implication (Antecedent / Consequent)

**Example:**
```systemverilog
req |-> ack
```
```
req = Antecedent
ack = Consequent
```
**Meaning:** IF `req` happens, THEN `ack` should happen.

### Overlapping Implication (`|->`)
Same clock cycle.
```systemverilog
req |-> ack
```
```
Cycle: 1 2 3 4 5
req:            1
ack:            1
```
If `req = 1` at cycle 5, `ack` must be `1` at cycle 5.

### Non-overlapping Implication (`|=>`)
Next clock cycle.
```systemverilog
req |=> ack
```
If `req = 1` at cycle 5, `ack` must be `1` at cycle 6.

**Memory:**
```
|->  = Same cycle
|=>  = Next cycle
```

---

## 7. assert vs assume vs cover

| Keyword | Meaning |
|---|---|
| `assert` | Check DUT correctness |
| `assume` | Restrict input/environment |
| `cover` | Check that a scenario happened |

### assert
```systemverilog
assert property(req |-> ack);
```
**Meaning:** if `req` happens, `ack` must happen. **Failure → DUT error.**

### assume
```systemverilog
assume property(reset |-> !req);
```
**Meaning:** during reset, assume `req` will not happen. Used to restrict inputs (typically for formal verification).

### cover
```systemverilog
cover property(write ##1 read);
```
**Meaning:** did `write` happen, followed by `read` one cycle later? No failure — only tracks coverage.

---

## 8. SVA Coding Question — req/ack within N cycles

**Interview question:** "Write an assertion where `ack` should come within 3 cycles of `req`."

```systemverilog
assert property(
    @(posedge clk)
    req |-> ##[1:3] ack
);
```

**Meaning:** if `req` happens, `ack` must happen 1 to 3 cycles later.

```
req
 ↓
cycle +1
cycle +2
cycle +3   ← ack must appear somewhere in this window
```

---

## Quick Revision Tricks

### UVM
```
Test
 ↓
Env
 ↓
Agent
 ↓
Sequencer → Driver → DUT
              ↑
           Monitor
              ↓
         Scoreboard
```
**Remember:**
```
Sequence creates
Sequencer sends
Driver drives
Monitor observes
Scoreboard checks
```

### UVM Phases
```
Build   → Create
Connect → Connect
Run     → Work
Check   → Verify
Report  → Print
```
**Direction:**
```
Build:   Top → Down
Connect: Bottom → Up
```

### SVA
```
assert  → Check
assume  → Restrict
cover   → Occurred?

|->  → Same cycle
|=>  → Next cycle

A |-> B
A = IF condition
B = Expected result
```
