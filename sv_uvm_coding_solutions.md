
## Table of Contents

**Must Practice**
1. [SVA Handshake Assertion — Code 1 (non-overlapping)](#1-sva-handshake-assertion--code-1-non-overlapping)
2. [SVA Handshake Assertion — Code 2 (overlapping)](#2-sva-handshake-assertion--code-2-overlapping)
3. [UVM Driver Skeleton](#3-uvm-driver-skeleton)
4. [UVM Scoreboard write() Function](#4-uvm-scoreboard-write-function)
5. [SV Class with Constraints](#5-sv-class-with-constraints)
6. [fork/join Variants — Code 1](#6-forkjoin-variants--code-1)
7. [fork/join Variants — Code 2 (Mailbox Producer-Consumer)](#7-forkjoin-variants--code-2-mailbox-producer-consumer)

**Time Allows**
8. [Synchronous FIFO in SystemVerilog](#8-synchronous-fifo-in-systemverilog)
9. [Bit Manipulation in C — Firmware Context](#9-bit-manipulation-in-c--firmware-context)
10. [LRU Cache in C++](#10-lru-cache-in-c)

**Quick Reference**
11. [Quick Syntax Reference](#11-quick-syntax-reference)

---

## 1. SVA Handshake Assertion — Code 1 (non-overlapping)

**`|=>` — ack must arrive in the NEXT cycle after req**

```systemverilog
property req_ack;                  // Define a property named req_ack

    @(posedge clk)                 // Check the property at every positive clock edge

    disable iff (reset)            // If reset = 1, do not check this property

    req |=> ##[1:4] ack;           // If req = 1, start from NEXT cycle
                                   // ack should occur within 1-to-4 cycle window

endproperty                        // End of req_ack property

assert property(req_ack);          // Check the req_ack property
                                   // Assertion fails if the property rule is violated
```

---

## 2. SVA Handshake Assertion — Code 2 (overlapping)

**`|->` — ack can arrive in the SAME cycle as req**

```systemverilog
property req_ack;                  // Define a rule/property named req_ack

    @(posedge clk)                 // Check this rule at every positive edge of clk

    disable iff (reset)            // If reset = 1, do not check this rule

    req |-> ##[1:4] ack;           // If req = 1, start checking from the SAME cycle
                                   // ack should occur within 1-to-4 cycle window

endproperty                        // End of the req_ack property

assert property(req_ack);          // Check the req_ack rule
                                   // Assertion fails if the rule is violated
```

---

## 3. UVM Driver Skeleton

**1 code — full driver class**

```systemverilog
class my_driver extends uvm_driver #(my_seq_item);
    // Create our custom driver from the built-in UVM driver
    // #(my_seq_item) = this driver works with my_seq_item transactions

    `uvm_component_utils(my_driver)
    // Register our custom my_driver with the UVM factory

    function new(string name, uvm_component parent);
        // Constructor of my_driver
        // name   = name of this driver instance
        // parent = which UVM component this driver belongs to

        super.new(name, parent);
        // super = parent/base class (uvm_driver)
        // Pass driver name and parent information to the base class

    endfunction

    task run_phase(uvm_phase phase);
        // run_phase = predefined UVM phase where driver does its main work
        // uvm_phase = type
        // phase = handle to the current phase object passed by UVM

        forever begin
            // Keep repeating driver work until run_phase ends

            seq_item_port.get_next_item(req);
            // seq_item_port = communication path between driver and sequencer
            // get_next_item = get the next transaction from sequencer
            // req = handle pointing to the received my_seq_item transaction

            $display("Driving transaction: data = %0d", req.data);
            // Access data from the transaction using req.data
            // %0d = print decimal value without leading spaces

            seq_item_port.item_done();
            // Tell sequencer: this transaction is done

        end

    endtask

endclass
```

---

## 4. UVM Scoreboard write() Function

**1 code — full scoreboard class**

```systemverilog
class my_scoreboard extends uvm_component;
    // Create our custom scoreboard from the built-in UVM component

    `uvm_component_utils(my_scoreboard)
    // Register our custom my_scoreboard with the UVM factory

    uvm_analysis_imp #(my_seq_item, my_scoreboard) imp;
    // imp = scoreboard's transaction receiving port
    // my_seq_item   = type of transaction it receives
    // my_scoreboard = class that receives/handles the transaction

    function new(string name, uvm_component parent);
        // Constructor used to initialize the scoreboard instance
        // name   = user-given name of this scoreboard instance
        // parent = which UVM component this scoreboard belongs to

        super.new(name, parent);
        // Pass scoreboard name and parent information to the base class

        imp = new("imp", this);
        // Create/initialize the analysis receiving port
        // "imp" = user-given name of this port
        // this  = current my_scoreboard instance

    endfunction

    function void write(my_seq_item item);
        // write() is called when a transaction arrives through imp
        // my_seq_item = transaction type
        // item = handle to the received transaction

        if(item.data == 100)
            // Check if received data value is equal to decimal 100
            $display("PASS: data = %0d", item.data);
            // If data = 100, print PASS
        else
            $display("FAIL: expected = 100, actual = %0d", item.data);
            // If data is not 100, print FAIL

    endfunction

endclass
```

---

## 5. SV Class with Constraints

**1 code — full class + testbench**

```systemverilog
class Packet;
    // Create a user-defined class called Packet

    rand bit [7:0] addr;
    // 8-bit random variable for address

    rand bit [7:0] data;
    // 8-bit random variable for data

    constraint addr_c {
        addr inside {[8'h10:8'hFF]};
    }
    // addr can only be randomized between 0x10 and 0xFF

    constraint data_c {
        data != 0;
    }
    // data can be randomized but cannot be 0

    function void print();
        $display("addr = 0x%0h, data = 0x%0h", addr, data);
        // Print addr and data in hexadecimal
    endfunction

endclass


module tb;

    Packet pkt;
    // pkt = handle that will point to a Packet object

    initial begin

        pkt = new();
        // Create the actual Packet object

        repeat(5) begin
            // Repeat randomization 5 times

            if(pkt.randomize())
                // randomize() returns 1 if successful, 0 if it fails
                pkt.print();

        end

        if(pkt.randomize() with { addr == 8'hAA; })
            // Inline constraint: addr must be 0xAA for THIS call only
            // data is still randomized following data_c
            pkt.print();

    end

endmodule
```

---

## 6. fork/join Variants — Code 1

**fork...join + join_any + join_none + disable fork**

```systemverilog
module tb;

    initial begin

        // -----------------------------
        // fork...join
        // -----------------------------
        fork

            begin
                #5;
                $display("Task 1 finished");
            end

            begin
                #10;
                $display("Task 2 finished");
            end

        join
        // join waits until ALL forked threads finish
        // Waits until both Task 1 (#5) and Task 2 (#10) are done

        $display("fork...join complete");


        // -----------------------------
        // fork...join_any
        // -----------------------------
        fork

            begin
                #5;
                $display("Task 3 finished");
                // Finishes first after 5 time units
            end

            begin
                #10;
                $display("Task 4 finished");
                // Would finish after 10 time units
            end

        join_any
        // join_any waits until ANY ONE thread finishes
        // Task 3 finishes first → execution continues

        $display("fork...join_any complete");

        disable fork;
        // Kill Task 4 which is still running


        // -----------------------------
        // fork...join_none
        // -----------------------------
        fork

            begin
                #5;
                $display("Task 5 finished");
            end

            begin
                #10;
                $display("Task 6 finished");
            end

        join_none
        // join_none does NOT wait for any thread
        // Main execution continues immediately

        $display("fork...join_none continues immediately");

        #2;
        // Main thread waits only 2 time units
        // Task 5 needs 5, Task 6 needs 10 — neither finished

        disable fork;
        // Kill Task 5 and Task 6 — their $display will NOT execute

        $display("Remaining forked threads killed");

    end

endmodule

// join      → wait for ALL threads
// join_any  → wait until ANY ONE finishes, kill rest with disable fork
// join_none → wait for NONE, continue immediately
// #5        → 5 simulation time units, NOT 5 clock cycles
```

---

## 7. fork/join Variants — Code 2 (Mailbox Producer-Consumer)

```systemverilog
module tb;

    mailbox #(int) mb;
    // Declare mailbox handle named mb
    // #(int) = this mailbox stores integer data

    task producer;
        int data;

        for(int i = 0; i < 5; i++) begin
            #5;
            // Wait 5 time units before sending each value

            data = i;
            mb.put(data);
            // Put/send data into mailbox mb

            $display("Producer sent: %0d", data);
        end

    endtask

    task consumer;
        int data;

        for(int i = 0; i < 5; i++) begin

            mb.get(data);
            // Get/remove one value from mailbox
            // If mailbox is empty, get() waits until data is available

            $display("Consumer received: %0d", data);
        end

    endtask

    initial begin

        mb = new();
        // Create the actual mailbox object

        fork
            producer();
            consumer();
        join
        // Wait until BOTH producer and consumer finish

        $display("Producer and Consumer finished");

    end

endmodule
```

---

## 8. Synchronous FIFO in SystemVerilog

**Code 1 — FIFO module + Code 2 — assertions wired to it**

```systemverilog
module sync_fifo #(
    parameter WIDTH = 8,
    parameter DEPTH = 8
)(
    input  logic             clk,
    input  logic             reset,
    input  logic             wr_en,
    input  logic             rd_en,
    input  logic [WIDTH-1:0] wr_data,
    output logic [WIDTH-1:0] rd_data,
    output logic             full,
    output logic             empty
);

    // Alternative way:
    // localparam PTR_WIDTH = $clog2(DEPTH);
    // For DEPTH = 8, $clog2(8) = 3.

    logic [WIDTH-1:0] fifo_memory [0:DEPTH-1];

    // 3-bit pointers because DEPTH = 8
    logic [2:0] wr_ptr;
    logic [2:0] rd_ptr;
    logic [2:0] next_wr_ptr;
    // Next write pointer

    always_comb begin
        if(wr_ptr == DEPTH-1)
            next_wr_ptr = 0;
        else
            next_wr_ptr = wr_ptr + 1;
    end

    // Empty condition
    assign empty = (wr_ptr == rd_ptr);

    // Full condition
    assign full = (next_wr_ptr == rd_ptr);

    always_ff @(posedge clk) begin
        if(reset) begin
            wr_ptr  <= 0;
            rd_ptr  <= 0;
            rd_data <= 0;
        end
        else begin

            // Write operation
            if(wr_en && !full) begin
                fifo_memory[wr_ptr] <= wr_data;
                if(wr_ptr == DEPTH-1)
                    wr_ptr <= 0;
                else
                    wr_ptr <= wr_ptr + 1;
            end

            // Read operation
            if(rd_en && !empty) begin
                rd_data <= fifo_memory[rd_ptr];
                if(rd_ptr == DEPTH-1)
                    rd_ptr <= 0;
                else
                    rd_ptr <= rd_ptr + 1;
            end

        end
    end

    ////////////////////////////////////////////////////////////////////////////
    //////////////////////////// ADDED ASSERTIONS //////////////////////////////
    ////////////////////////////////////////////////////////////////////////////

    ////////////////////////////// WRITE CONDITIONS ////////////////////////////

    // Case 1: No write request, FIFO not full
    assert property (@(posedge clk)
        (wr_en == 0 && full == 0) |-> !wr_en
    );

    // Case 2: No write request, FIFO full
    assert property (@(posedge clk)
        (wr_en == 0 && full == 1) |-> !wr_en
    );

    // Case 3: Write requested but FIFO full — illegal overflow attempt
    assert property (@(posedge clk)
        !(wr_en && full)
    );

    // Valid write: wr_en = 1, full = 0

    ////////////////////////////// READ CONDITIONS /////////////////////////////

    // Case 1: No read request, FIFO not empty
    assert property (@(posedge clk)
        (rd_en == 0 && empty == 0) |-> !rd_en
    );

    // Case 2: No read request, FIFO empty
    assert property (@(posedge clk)
        (rd_en == 0 && empty == 1) |-> !rd_en
    );

    // Case 3: Read requested but FIFO empty — illegal underflow attempt
    assert property (@(posedge clk)
        !(rd_en && empty)
    );

    // Valid read: rd_en = 1, empty = 0

    ////////////////////////////////////////////////////////////////////////////
    ////////////////////////// END ADDED ASSERTIONS ////////////////////////////
    ////////////////////////////////////////////////////////////////////////////

endmodule
```

---

## 9. Bit Manipulation in C — Firmware Context

**1 code — all register operations**

```c
#include <stdint.h>

void register_bit_operations(unsigned int N)
{
    volatile uint32_t *REG =
        (volatile uint32_t*)0x40010000;
    // volatile = tell compiler: always read/write actual hardware register
    // never cache, never reorder this access
    // 0x40010000 = hardware register memory address

    // Set bit N — force bit N to 1, leave others unchanged
    *REG |= (1U << N);

    // Clear bit N — force bit N to 0, leave others unchanged
    *REG &= ~(1U << N);

    // Toggle bit N — flip bit N, leave others unchanged
    *REG ^= (1U << N);

    // Check bit N — test if bit N is 1
    if(*REG & (1U << N))
    {
        // Bit N is set
    }
    else
    {
        // Bit N is clear
    }
}
```

---

## 10. LRU Cache in C++

**get() and put() both O(1)**

```cpp
#include <unordered_map>
#include <list>
using namespace std;

class LRUCache
{
private:
    int capacity;

    unordered_map<int, list<pair<int,int>>::iterator> cache;
    // key -> iterator pointing to node in list
    // unordered_map gives O(1) lookup by key

    list<pair<int,int>> usageList;
    // front = most recently used
    // back  = least recently used
    // doubly linked list gives O(1) insert and delete

public:
    LRUCache(int size)
    {
        capacity = size;
    }

    int get(int key)
    {
        if(cache.find(key) == cache.end())
            return -1;
            // Key not found

        auto node  = cache[key];
        int  value = node->second;

        usageList.erase(node);
        // Remove from old position — O(1) with iterator

        usageList.push_front({key, value});
        // Move to front (most recently used) — O(1)

        cache[key] = usageList.begin();
        // Update map to point to new position — O(1)

        return value;
    }

    void put(int key, int value)
    {
        if(cache.find(key) != cache.end())
        {
            // Key already exists — update and move to front
            usageList.erase(cache[key]);
            usageList.push_front({key, value});
            cache[key] = usageList.begin();
            return;
        }

        if(cache.size() == capacity)
        {
            // Cache full — evict least recently used (back of list)
            int lruKey = usageList.back().first;
            usageList.pop_back();
            cache.erase(lruKey);
        }

        // Add new item as most recently used
        usageList.push_front({key, value});
        cache[key] = usageList.begin();
    }
};
```

---

## 11. Quick Syntax Reference

Write these from memory before the interview:

```systemverilog
// SVA — non-overlapping |=>
property req_ack_ne;
    @(posedge clk) disable iff (reset)
    req |=> ##[1:4] ack;
endproperty
assert property(req_ack_ne);

// SVA — overlapping |->
property req_ack_ov;
    @(posedge clk) disable iff (reset)
    req |-> ##[1:4] ack;
endproperty
assert property(req_ack_ov);

// Driver
class my_driver extends uvm_driver #(my_seq_item);
    `uvm_component_utils(my_driver)
    function new(string name = "my_driver", uvm_component parent = null);
        super.new(name, parent);
    endfunction
    task run_phase(uvm_phase phase);
        my_seq_item req;
        forever begin
            seq_item_port.get_next_item(req);
            $display("data=%0d", req.data);
            seq_item_port.item_done();
        end
    endtask
endclass

// Scoreboard
class my_scoreboard extends uvm_component;
    `uvm_component_utils(my_scoreboard)
    uvm_analysis_imp #(my_seq_item, my_scoreboard) imp;
    function new(string name = "my_scoreboard", uvm_component parent = null);
        super.new(name, parent);
        imp = new("imp", this);
    endfunction
    function void write(my_seq_item item);
        if(item.data == 100)
            $display("PASS: data=%0d", item.data);
        else
            $display("FAIL: expected=100 got=%0d", item.data);
    endfunction
endclass

// Class with constraints
class Packet;
    rand bit [7:0] addr;
    rand bit [7:0] data;
    constraint addr_c { addr inside {[8'h10:8'hFF]}; }
    constraint data_c { data != 0; }
    function void print();
        $display("addr=%0h data=%0h", addr, data);
    endfunction
endclass

// fork variants
fork task_a(); task_b(); join          // wait ALL
fork task_a(); task_b(); join_any      // wait fastest
fork task_a(); task_b(); join_none     // wait none
disable fork;                          // kill all threads

// Mailbox
mailbox #(int) mb = new();
mb.put(value);    // producer
mb.get(value);    // consumer — blocks if empty

// C register bit ops
*REG |=  (1U << N);   // set
*REG &= ~(1U << N);   // clear
*REG ^=  (1U << N);   // toggle
if(*REG & (1U << N))  // check
```
