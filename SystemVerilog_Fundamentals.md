# SystemVerilog / UVM Interview Quick Revision Sheet

## Table of Contents

**Language Basics**
1. [wire vs logic vs reg](#1-wire-vs-logic-vs-reg)
2. [Blocking vs Non-blocking](#2-blocking-vs-non-blocking)
3. [always_comb / always_ff / always_latch](#3-always_comb--always_ff--always_latch)
4. [Race Condition](#4-race-condition)
5. [Clocking Block](#5-clocking-block)
6. [Driver and Monitor](#6-driver-and-monitor)

**OOP Concepts**
7. [Inheritance](#7-inheritance)
8. [Virtual Function](#8-virtual-function)
9. [Polymorphism](#9-polymorphism)
10. [Static Variable](#10-static-variable)
11. [Instance Variable](#11-instance-variable)
12. [Parameterized Class](#12-parameterized-class)
13. [new() Constructor](#13-new-constructor)
14. [new() vs type_id::create()](#14-new-vs-type_idcreate)

**Randomization**
15. [rand and randomize](#15-rand-and-randomize)
16. [randomize() with Constraint](#16-randomize-with-constraint)
17. [inside](#17-inside)
18. [rand vs randc](#18-rand-vs-randc)

**Process Control**
19. [fork / join](#19-fork--join)
20. [disable fork](#20-disable-fork)

**UVM Utilities**
21. [$cast()](#21-cast)
22. [post_randomize()](#22-post_randomize)
23. [XOR Parity (^data)](#23-xor-parity-data)
24. [Transaction Object](#24-transaction-object)
25. [copy vs clone](#25-copy-vs-clone)
26. [Shallow vs Deep Copy](#26-shallow-vs-deep-copy)
27. [TLM FIFO](#27-tlm-fifo)
28. [UVM Driver Concept](#28-uvm-driver-concept)

[Final Memory Table](#final-memory-table)

---

## 1. wire vs logic vs reg

| Type | Category | Usage | Stores value? |
|---|---|---|---|
| `wire` | Net type | Continuous assignment | No |
| `reg` | Old Verilog variable | Procedural blocks | Not always a real register |
| `logic` | SystemVerilog type | Replaces most `reg` usage | Depends on coding style |

**wire**
```verilog
wire y;
assign y = a & b;
```

**reg**
```verilog
reg q;
always @(posedge clk)
    q <= d;
```

**logic** — hardware inferred depends on coding style, not the keyword itself.

> **Interview answer:** "`logic` removes confusion caused by `reg`. It is used for most SystemVerilog signals."

---

## 2. Blocking vs Non-blocking

| | Blocking (`=`) | Non-blocking (`<=`) |
|---|---|---|
| Used for | Combinational logic, `always_comb` | Sequential logic, `always_ff`, flip-flops |
| Executes | Immediately, line by line | Updates at clock edge |

**Blocking**
```verilog
always_comb begin
    y = a & b;
end
```

**Non-blocking**
```verilog
always_ff @(posedge clk)
    q <= d;
```

**Rule of thumb:**
```
No clock  → =
Clock     → <=
```

> **Interview answer:** "Blocking updates immediately and is used for combinational logic. Non-blocking updates at the clock edge and is used for sequential logic."

---

## 3. always_comb / always_ff / always_latch

| Block | Creates | Notes |
|---|---|---|
| `always_comb` | Combinational logic | No memory |
| `always_ff` | Flip-flop | Requires clock |
| `always_latch` | Latch | Intentional latch behavior |

**Unintended latch — problem:**
```verilog
always_comb begin
    if (enable)
        y = a;
    // no else → y holds previous value → latch inferred
end
```

**Correct (fully specified):**
```verilog
always_comb begin
    if (enable)
        y = a;
    else
        y = 0;
end
```

> **Interview answer:** "`always_comb` describes combinational logic, `always_ff` describes sequential logic, and `always_latch` describes intentional latch behavior."

---

## 4. Race Condition

**Problem:** Two processes access the same signal at the same simulation time — the result depends on execution order.

**Prevention:**
- Non-blocking assignments
- Clocking blocks
- UVM synchronization

> **Interview answer:** "A race condition occurs when multiple processes access the same signal at the same time and the result depends on scheduling order."

---

## 5. Clocking Block

**Purpose:** Controls when the driver drives signals and when the monitor samples signals.

**Flow:**
```
Clock edge → DUT updates → Monitor samples → Driver drives next input
```

> **Interview answer:** "Clocking blocks define timing between the DUT and testbench to avoid race conditions."

---

## 6. Driver and Monitor

| Component | Role | Flow |
|---|---|---|
| Driver | Sends inputs to DUT | Driver → DUT |
| Monitor | Reads DUT outputs | DUT → Monitor |

Both use the **same clock timing** to avoid races.

---

## 7. Inheritance

Child class gets the parent's properties and functions.

```verilog
class child extends parent;
endclass

class axi_packet extends packet;
endclass
```

**Flow:** `packet → axi_packet`

---

## 8. Virtual Function

Used for polymorphism — lets a child override a parent's function.

**Parent:**
```verilog
virtual function void display();
endfunction
```

**Child:**
```verilog
function void display();
endfunction
```

---

## 9. Polymorphism

The same function call behaves differently depending on the object type.

```verilog
packet p;
p = new axi_packet();
p.display();
```

**Requires:** the function must be declared `virtual`.

**Rule:**
```
Parent handle + Child object + virtual  =  Child function runs
```

---

## 10. Static Variable

One shared copy across **all** objects.

```verilog
static int count;
```

**Used for:** counting objects, shared information.

```
packet1 created → count = 1
packet2 created → count = 2
```

---

## 11. Instance Variable

Each object has its **own** copy.

```verilog
int id;
```

```
Object1: id = 10
Object2: id = 20
```

---

## 12. Parameterized Class

One class definition, different sizes via parameters.

```verilog
class packet #(int WIDTH = 8);
    bit [WIDTH-1:0] data;
endclass
```

**Usage:**
```verilog
packet #(8)  p8;
packet #(16) p16;
```

`#()` passes class parameters.

---

## 13. new() Constructor

`new()` creates an object.

```verilog
packet p;
p = new();
```

Handle `p` points to the `packet` object.

---

## 14. new() vs type_id::create()

| | `new()` | `type_id::create()` |
|---|---|---|
| Creation | Direct | UVM factory-based |
| Class created | Exact class specified | Can be overridden |
| Override support | No | Yes |

**new()**
```verilog
packet p;
p = new();
```

**type_id::create()**
```verilog
driver = driver_type::type_id::create("driver");
```

The factory can substitute `normal_driver` with `stress_driver` without changing the test code.

> **Interview answer:** "`new` creates a fixed object. `type_id::create` uses the UVM factory and allows replacement using overrides."

---

## 15. rand and randomize

```verilog
rand bit [7:0] data;
```

Generate random values:
```verilog
pkt.randomize();
```

---

## 16. randomize() with Constraint

```verilog
object.randomize() with {
    constraint;
};
```

**Example:**
```verilog
pkt.randomize() with {
    data > 50;
};
```

Generates a random value that satisfies the condition.

---

## 17. inside

Used in constraints to restrict a value to a set/range.

```verilog
address inside {[100:200]};   // range
opcode  inside {1, 3, 5};     // discrete set
```

---

## 18. rand vs randc

| | `rand` | `randc` |
|---|---|---|
| Behavior | Random values | Random **cyclic** |
| Repetition | Allowed | No repeat until all values used |

**rand example sequence:** `5, 8, 5, 2, ...`

---

## 19. fork / join

| Keyword | Waits for |
|---|---|
| `join` | ALL processes |
| `join_any` | ONE (first) process |
| `join_none` | NONE (doesn't wait) |

---

## 20. disable fork

Stops remaining fork processes.

```verilog
fork
    process1();
    process2();
join_any
disable fork;
```

**Use case:** once one process finishes, stop the others.

---

## 21. $cast()

Used for downcasting (and can check upcasting-safe assignments).

```verilog
$cast(destination, source);
```

**Upcasting** (child handle → parent handle) — implicit, no cast needed:
```verilog
parent = child;
```

**Downcasting** (parent handle → child handle) — needs `$cast`:
```verilog
$cast(child, parent);
```

---

## 22. post_randomize()

Runs automatically right after `randomize()` completes.

```verilog
function void post_randomize();
    parity = ^data;
endfunction
```

**Used for:** calculating derived values after randomization.

---

## 23. XOR Parity (^data)

```verilog
data = 8'b10101010;
parity = ^data;   // XOR of all bits
```

Used for parity checking.

---

## 24. Transaction Object

A class object representing one operation/data transfer.

```verilog
class packet;
    rand bit [7:0] data;
endclass

packet pkt;
```

**Flows through:** Sequence → Driver → DUT

---

## 25. copy vs clone

| Method | Behavior |
|---|---|
| `copy()` | Copies data into an **existing** object |
| `clone()` | Creates a **new** object and copies data into it |

```verilog
dest.copy(src);          // copy
new_obj = old_obj.clone(); // clone
```

---

## 26. Shallow vs Deep Copy

| | Shallow copy | Deep copy |
|---|---|---|
| Outer object | New | New |
| Internal (nested) objects | Shared | Also copied |

UVM uses `clone()` to avoid shared transaction modification.

---

## 27. TLM FIFO

Transaction communication mechanism in UVM.

```verilog
uvm_tlm_fifo #(packet) fifo;
fifo = new("fifo");
```

**Producer:**
```verilog
fifo.put(pkt);
```

**Consumer:**
```verilog
fifo.get(pkt);
```

**Flow:** `Monitor → TLM FIFO → Scoreboard`

---

## 28. UVM Driver Concept

UVM provides a base `uvm_driver`, which is extended for custom drivers:

```verilog
class alu_driver extends uvm_driver;
```

The custom driver drives DUT signals; the environment holds the driver handle.

**Flow:** `Test → Environment → Driver → DUT`

---

## Final Memory Table

### Procedural / RTL
| Symbol/Keyword | Meaning |
|---|---|
| `=` | Combinational |
| `<=` | Sequential |
| `always_comb` | Combinational logic |
| `always_ff` | Flip-flop |
| `always_latch` | Latch |

### OOP
| Keyword | Meaning |
|---|---|
| `extends` | Inheritance |
| `virtual` | Polymorphism |
| `$cast` | Downcasting |
| `static` | Shared variable |

### Randomization
| Keyword | Meaning |
|---|---|
| `rand` | Random |
| `randc` | Unique/cyclic random |

### Object Creation
| Method | Meaning |
|---|---|
| `new` | Direct object creation |
| `type_id::create` | Factory creation |
| `copy` | Copy into existing object |
| `clone` | Copy into new object |

### fork/join
| Keyword | Meaning |
|---|---|
| `join` | Wait for all |
| `join_any` | Wait for one |
| `join_none` | Don't wait |

### TLM
| Method | Meaning |
|---|---|
| `put()` | Send transaction |
| `get()` | Receive transaction |
