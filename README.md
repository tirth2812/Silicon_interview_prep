
# Round 1 SystemVerilog / DV Quick Revision Cheat Sheet

---

# 📗 Foundational

---

## 1. wire vs logic vs reg

### wire
- Net type.
- Used for continuous assignments.
- Does not store values.
- Supports multiple drivers.

Example:
```systemverilog
wire y;
assign y = a & b;
```

### reg
- Old Verilog variable type.
- Used inside procedural blocks.
- Does NOT always mean hardware register.

Clocked:
```systemverilog
reg q;

always @(posedge clk)
    q <= d;
```
→ Flip-flop

Combinational:
```systemverilog
reg y;

always @(*)
    y = a & b;
```
→ Combinational logic

### logic
- SystemVerilog replacement for most `reg` usage.
- Removes confusion caused by the name `reg`.
- Hardware depends on coding style, not the keyword.

### Interview Answer:
"SystemVerilog introduced logic to remove confusion caused by reg. A reg is only a procedural variable and does not necessarily represent a hardware register. Logic can replace most uses of reg and many uses of wire, but wire is still needed for multiple drivers."

---

# 2. Blocking (=) vs Non-blocking (<=)

## Blocking (=)

Used for:
- Combinational logic
- always_comb

Behavior:
- Immediate update
- Executes line by line

Example:
```systemverilog
always_comb begin
    y = a & b;
end
```

## Non-blocking (<=)

Used for:
- Sequential logic
- always_ff
- Flip-flops

Behavior:
- Updates together at clock edge

Example:
```systemverilog
always_ff @(posedge clk)
    q <= d;
```

### Rule:
No clock → `=`

Clock → `<=`

### Interview Answer:
"Blocking assignments update immediately and are used for combinational logic. Non-blocking assignments update together at clock edges and are used for sequential logic. Mixing them in clocked logic can create simulation mismatches."

---

# 3. always_comb vs always_ff vs always_latch

## always_comb
Creates:
- Combinational logic

No memory.

## always_ff
Creates:
- Flip-flops

Requires clock.

## always_latch
Creates:
- Latches

### Unintended Latch

Problem:

```systemverilog
always_comb begin
    if(enable)
        y = a;
end
```

When enable is false, y has no assignment.

Hardware creates a latch.

### Interview Answer:
"always_comb describes combinational logic, always_ff describes clocked sequential logic, and always_latch describes intentional latch behavior. Missing assignments in combinational blocks can unintentionally infer latches."

---

# 4. Race Conditions

## What is a race condition?

When two processes access the same signal at the same simulation time and result depends on execution order.

### Prevention:
- Non-blocking assignments
- Clocking blocks
- UVM synchronization

### Interview Answer:
"A race condition occurs when multiple processes access the same signal at the same simulation time and the result depends on scheduling order. SystemVerilog reduces this using non-blocking assignments, clocking blocks, and synchronization mechanisms."

---

# 5. Clocking Blocks

Purpose:
- Control when testbench drives signals.
- Control when testbench samples signals.
- Avoid DUT/TB race conditions.

Flow:

Clock edge  
↓  
DUT updates  
↓  
Monitor samples  
↓  
Driver drives next input

### Interview Answer:
"Clocking blocks define timing relationships between testbench and DUT signals, preventing race conditions by controlling when signals are sampled and driven."

---

# 6. Driver and Monitor Synchronization

Driver:
- Writes inputs.

Monitor:
- Reads outputs.

Both use same clock convention to:
- Avoid race conditions.
- Sample correct values.
- Match hardware timing.

### Interview Answer:
"The driver and monitor synchronize with the DUT clock so stimulus is applied and outputs are sampled at predictable times. This avoids races and ensures simulation matches hardware behavior."

---

# 7. OOP Concepts

## Inheritance

Child reuses parent.

Syntax:

```systemverilog
class child extends parent;
```

---

## Virtual Function

Allows polymorphism.

Parent:

```systemverilog
virtual function void display();
endfunction
```

Child overrides.

---

## Polymorphism

Same function call behaves differently depending on object type.

---

## Static Variable

One copy shared by all objects.

Example:

```systemverilog
static int count;
```

Used for:
- Object counting
- Shared information

---

## Instance Variable

Each object has separate copy.

---

## Parameterized Class

One class supports different configurations.

Example:

```systemverilog
class packet #(int WIDTH=8);
```

---

# 8. disable fork

Purpose:
- Stop remaining forked processes.

Common:

```systemverilog
join_any

disable fork;
```

### Interview Answer:
"disable fork terminates all active child processes created by the current fork block. It is commonly used after join_any to stop remaining unnecessary processes."

---

# 9. rand vs randc

## rand
- Random values.
- Duplicates allowed.

Example:
```
5
8
5
2
```

## randc
- Random cyclic.
- No repetition until all values are used.

### Interview Answer:
"rand allows repeated random values, while randc generates random values without repetition until the complete value space is exhausted."

---

# 10. randomize() with Inline Constraint

Example:

```systemverilog
pkt.randomize() with {
    address inside {[100:200]};
};
```

Meaning:
- Temporary constraint.
- Applies only during this randomization.

### Interview Answer:
"randomize() with applies temporary constraints during randomization without modifying the original class constraint."

---

# 11. fork join_any / join_none

## join
Wait for all tasks.

## join_any
Wait for first task.

## join_none
Do not wait.

Memory:

```
join       → ALL
join_any   → ONE
join_none  → NONE
```

---

# 📘 Commonly Asked

---

# 12. new() vs type_id::create()

## new()
- Direct creation.
- Fixed type.

## type_id::create()
- Factory-based creation.
- Supports overrides.

### Interview Answer:
"new() directly creates the specified object, while type_id::create() uses the UVM factory and allows type or instance overrides without modifying the environment."

---

# 13. copy() vs clone()

## copy()

Copies into existing object.

```systemverilog
dest.copy(src);
```

## clone()

Creates new object and copies.

```systemverilog
new_obj = old_obj.clone();
```

### Interview Answer:
"copy() copies data into an existing object, while clone() creates a new object and copies the contents into it."

---

# 14. Mailbox vs TLM FIFO

## Mailbox
- SystemVerilog feature.
- Process communication.

Task → Mailbox → Task

## TLM FIFO
- UVM feature.
- Transaction communication.

Monitor → FIFO → Scoreboard

### Common Answers:

## Queue vs mailbox
"Queue stores data, while mailbox provides synchronized communication between processes."

## Why use TLM communication?
"TLM allows components to exchange complete transactions instead of individual signals, improving modularity and reuse."

## How does scoreboard receive transactions?
"Monitor sends transactions through analysis ports, and scoreboard receives them through analysis implementation or TLM FIFO."

---

# 📕 Previously Asked

## Mixed SV Coding Question

Topics combined:

- fork_join_none
- virtual functions
- $cast
- static variables
- post_randomize parity

Remember:

```
extends        → inheritance

virtual        → polymorphism

$cast          → safe conversion

static         → shared variable

randomize()    → generate values

post_randomize → calculate derived fields

join_none      → background execution
```

---

# Final One-Line Revision

```
wire       → connection
logic      → modern signal type
reg        → old procedural variable

=          → combinational
<=         → sequential

always_comb → gates
always_ff   → flip-flops
always_latch→ latch

virtual    → child behavior
$cast      → safe conversion
static     → shared variable

rand       → random
randc      → unique random

copy       → existing object
clone      → new object

join       → all
join_any   → one
join_none  → none

new        → direct creation
create     → factory creation
```
