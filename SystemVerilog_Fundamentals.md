========================================================
SystemVerilog / UVM Interview Quick Revision Sheet
========================================================


========================
1. wire vs logic vs reg
========================

wire:
- Net type.
- Used for continuous assignment.
- Does not store values.

Example:
wire y;
assign y = a & b;


reg:
- Old Verilog variable.
- Used inside procedural blocks.
- Does not always mean hardware register.

Example:

reg q;

always @(posedge clk)
    q <= d;


logic:
- SystemVerilog replacement for most reg usage.
- Hardware depends on coding style, not keyword.


Interview:
"logic removes confusion caused by reg. It is used for most SystemVerilog signals."


========================================================


=============================
2. Blocking vs Non-blocking
=============================

Blocking (=)

Used for:
- Combinational logic
- always_comb

Example:

always_comb begin
    y = a & b;
end

Behavior:
- Executes immediately
- Line by line


Non-blocking (<=)

Used for:
- Sequential logic
- always_ff
- Flip-flops

Example:

always_ff @(posedge clk)
    q <= d;


Rule:

No clock  → =
Clock     → <=


Interview:
"Blocking updates immediately and is used for combinational logic. Non-blocking updates at clock edge and is used for sequential logic."


========================================================


=====================================
3. always_comb / always_ff / always_latch
=====================================

always_comb:

Creates:
- Combinational logic

No memory.


always_ff:

Creates:
- Flip-flop

Requires clock.


always_latch:

Creates:
- Latch


Unintended Latch:

Problem:

always_comb begin

    if(enable)
        y = a;

end


If enable = 0:

No assignment to y.

Hardware stores previous value.

Latch is created.


Correct:

always_comb begin

    if(enable)
        y = a;
    else
        y = 0;

end


Interview:
"always_comb describes combinational logic, always_ff describes sequential logic, and always_latch describes intentional latch behavior."


========================================================


==================
4. Race Condition
==================

Problem:

Two processes access the same signal at the same simulation time.

Result depends on execution order.


Prevention:

- Non-blocking assignment
- Clocking blocks
- UVM synchronization


Interview:
"Race condition occurs when multiple processes access the same signal at the same time and the result depends on scheduling order."


========================================================


=================
5. Clocking Block
=================

Purpose:

Controls:

- When driver drives signals.
- When monitor samples signals.


Flow:

Clock edge

↓

DUT updates

↓

Monitor samples

↓

Driver drives next input


Interview:
"Clocking blocks define timing between DUT and testbench to avoid race conditions."


========================================================


==========================
6. Driver and Monitor
==========================

Driver:

- Sends inputs to DUT.

Flow:

Driver → DUT


Monitor:

- Reads DUT outputs.

Flow:

DUT → Monitor


Both use same clock timing to avoid races.


========================================================


================
7. Inheritance
================

Child gets parent properties and functions.


Syntax:

class child extends parent;

endclass


Example:

class axi_packet extends packet;

endclass


Flow:

packet
  |
  ↓
axi_packet


========================================================


====================
8. Virtual Function
====================

Used for polymorphism.

Parent:

virtual function void display();

endfunction


Child:

function void display();

endfunction


Allows child to override parent function.


========================================================


=================
9. Polymorphism
=================

Same function call behaves differently depending on object.


Example:

packet p;

p = new axi_packet();

p.display();


Requires:

virtual function


Rule:

Parent handle + Child object + virtual
=
Child function runs


========================================================


====================
10. Static Variable
====================

One shared copy for all objects.


Example:

static int count;


Used for:

- Counting objects
- Shared information


Example:

packet1 created

count = 1

packet2 created

count = 2


========================================================


=====================
11. Instance Variable
=====================

Each object has its own copy.


Example:

int id;


Object1:

id = 10


Object2:

id = 20


========================================================


========================
12. Parameterized Class
========================

One class with different sizes.


Example:

class packet #(int WIDTH = 8);

    bit [WIDTH-1:0] data;

endclass


Usage:

packet #(8)

packet #(16)


#() passes class parameters.


========================================================


=====================
13. new() Constructor
=====================

new() creates an object.


Example:

packet p;

p = new();


Handle:

p

points to

packet object


========================================================


===============================
14. new() vs type_id::create()
===============================


new():

- Direct creation.
- Creates exact class.
- Cannot be overridden.


Example:

packet p;

p = new();



type_id::create():

- UVM factory creation.
- Allows override.


Example:

driver = driver_type::type_id::create("driver");


Factory can change:

normal_driver

to

stress_driver


Interview:

"new creates a fixed object. type_id::create uses UVM factory and allows replacement using overrides."


========================================================


====================
15. rand and randomize
====================

rand:

Allows randomization.


Example:

rand bit [7:0] data;


Call:

pkt.randomize();


Generates random values.


========================================================


=============================
16. randomize() with Constraint
=============================


Syntax:

object.randomize() with {

    constraint;

};


Example:

pkt.randomize() with {

    data > 50;

};


Means:

Generate random value but satisfy condition.


========================================================


===========
17. inside
===========


Used in constraints.

Example:

address inside {[100:200]};


Means:

Address must be between 100 and 200.


Other:

opcode inside {1,3,5};


========================================================


==============
18. rand vs randc
==============


rand:

- Random values.
- Repetition allowed.


Example:

5
8
5
2



randc:

- Random cyclic.
- No repeat until all values used.


========================================================


================
19. fork / join
================


fork:

Runs processes parallel.


join:

Wait for all processes.


join_any:

Wait for first process.


join_none:

Do not wait.


Memory:

join       → ALL

join_any   → ONE

join_none  → NONE


========================================================


================
20. disable fork
================


Stops remaining fork processes.


Common:

fork

process1();

process2();

join_any

disable fork;


Use:

After one process finishes, stop others.


========================================================


====================
21. $cast()
====================


Used for downcasting.


Syntax:

$cast(destination, source);


Example:

$cast(child_handle, parent_handle);



Upcasting:

Child handle → Parent handle

Example:

parent = child;


Downcasting:

Parent handle → Child handle

Example:

$cast(child,parent);


========================================================


=========================
22. post_randomize()
=========================


Runs automatically after randomize().


Example:

function void post_randomize();

    parity = ^data;

endfunction


Used for:

Calculating derived values after randomization.


========================================================


========================
23. XOR Parity (^data)
========================


Example:

data = 8'b10101010


parity = ^data;


XOR all bits together.

Result stored in parity.


Used for parity checking.


========================================================


==========================
24. Transaction Object
==========================


Transaction is a class object representing one operation/data transfer.


Example:

class packet;

    rand bit [7:0] data;

endclass


Object:

packet pkt;


Used between:

Sequence → Driver → DUT


========================================================


========================
25. copy vs clone
========================


copy():

Copies data into existing object.


Example:

dest.copy(src);



clone():

Creates new object and copies data.


Example:

new_obj = old_obj.clone();


========================================================


=====================
26. Shallow vs Deep Copy
=====================


Shallow copy:

- New outer object.
- Internal objects shared.


Deep copy:

- New outer object.
- Internal objects also copied.


UVM uses clone to avoid shared transaction modification.


========================================================


==================
27. TLM FIFO
==================


Transaction communication mechanism in UVM.


Syntax:

uvm_tlm_fifo #(packet) fifo;


Means:

FIFO stores packet objects.


Create:

fifo = new("fifo");


Producer:

fifo.put(pkt);


Consumer:

fifo.get(pkt);



Flow:

Monitor

↓

TLM FIFO

↓

Scoreboard


========================================================


========================
28. UVM Driver Concept
========================


UVM provides:

uvm_driver


We extend it:

class alu_driver extends uvm_driver;


Our custom driver drives DUT signals.


Environment contains driver handle.


Flow:

Test

↓

Environment

↓

Driver

↓

DUT


========================================================


FINAL MEMORY TABLE
========================================================


=              → combinational

<=             → sequential

always_comb    → logic

always_ff      → flip-flop

always_latch   → latch


extends       → inheritance

virtual       → polymorphism

$cast         → downcasting

static        → shared variable

rand          → random

randc         → unique random


new           → direct object creation

type_id::create → factory creation


copy          → existing object copy

clone         → new object copy


join          → all

join_any      → one

join_none     → none


put()         → send transaction

get()         → receive transaction
