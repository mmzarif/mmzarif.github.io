---
layout: post
title: Parameterized FIFO supporting circular buffering, overflow protection, and simulation-based verification in Vivado
date: 2025-08-06
description:  First In, First Out memory buffer RTL design
skills: 
- SystemVerilog
- Vivado
- Digital design
- Circular queues
- Memory design
- Buffering
main-image: /FIFO.png
---

## Project Overview

When you have two pieces of hardware that run at their own pace and need to hand data between them, you can't just wire the output of one straight into the input of the other — the moment the consumer stalls, you lose data. The standard fix is a **FIFO (First In, First Out) buffer** sitting between them, absorbing bursts and letting each side operate on its own schedule.

I built a parameterized FIFO in SystemVerilog for exactly this situation, in the context of a low-latency stock-market data feed handler based on the **ITCH protocol** (a companion project of mine). There, the FIFO decouples the **parser** from the **order book** — so if the order book is busy, incoming parsed messages queue up safely instead of getting dropped:

```
ITCH Stream → Parser → FIFO → Order Book
```

📁 [Project GitHub Repository](https://github.com/mmzarif/market_data_parser)

---

## Background: why a FIFO here

ITCH is a protocol stock exchanges use to stream real-time order book updates. Parsing those messages quickly matters, but *buffering* them matters just as much — if the parser produces a message while the downstream order book is mid-update, that message needs somewhere to wait. Without a buffer, you either drop it or stall the whole pipeline (backpressure). A FIFO gives the messages a place to sit, in order, until the order book is ready for them.

---

## FIFO Architecture

I implemented the FIFO as a **circular queue** — a fixed block of memory with two pointers chasing each other around it. This is the memory-efficient way to build a FIFO: nothing ever physically shifts, you just move the pointers.

Key features:

- 16-entry buffer, parameterizable via `FIFO_DEPTH`
- Each entry is a custom 16-byte `parsed_msg_t` struct (shared with the ITCH parser)
- Circular indexing with a `write_ptr` and a `read_ptr`
- `full` and `empty` flags to protect against overflow and underflow
- Fully resettable to a clean initial state

![FIFO](/_projects/FIFO design in SystemVerilog/FIFO.png)

---

## Module Ports

```systemverilog
module msg_fifo #(
    parameter FIFO_DEPTH = 16
) (
    input  logic clk,
    input  logic reset,
    input  logic write_en,
    input  logic read_en,
    input  parsed_msg_t msg_in,
    output logic full,
    output logic empty,
    output parsed_msg_t msg_out
);
```

---

## Core Logic

The internal storage is just an array of message structs, indexed by two 4-bit pointers plus a `count` that tracks occupancy:

```systemverilog
parsed_msg_t fifo_mem [0:FIFO_DEPTH-1];
logic [3:0] write_ptr, read_ptr, count;
```

The trick that makes it "circular" is that both pointers wrap back to 0 once they hit the end of the buffer, so the memory is reused endlessly without ever moving data. `count` is what lets me generate the `full` and `empty` flags cleanly — when it hits `FIFO_DEPTH` the buffer is full, and at 0 it's empty. Those flags are what stop a write from clobbering unread data (overflow) or a read from pulling garbage out of an empty buffer (underflow).

---

## Simulation

I ran the testbench with `FIFO_DEPTH = 4` — a smaller buffer just makes the full/empty edge cases easier to hit and read in the waveform.

📂 [Testbench](https://github.com/mmzarif/market_data_parser/blob/main/sim/msg_fifo_tb.sv)

![Waveform](/_projects/FIFO design in SystemVerilog/waveform.png)

The results lined up with what a correct FIFO should do:

- `msg_out` stays undefined until `write_en` is asserted — nothing read before anything's written
- messages come back out in the exact order they went in
- `full` asserts correctly when the buffer overflows, blocking further writes
- `empty` asserts correctly on underflow, blocking reads

---

## Reflection

The thing I took away from this is how much a small, well-verified buffer earns its place in a real system. In something like high-frequency trading the cost of a dropped message is measured in money, and a FIFO is the unglamorous block that quietly makes sure that never happens — it keeps data ordered, keeps timing-sensitive logic decoupled, and stays modular enough to verify on its own. Writing the circular-queue logic by hand also made the pointer arithmetic and the full/empty conditions concrete in a way that reading about them never did.

---

## Future Improvements

- **Asynchronous (dual-clock) FIFO** — the current design is synchronous; a real system often needs the read and write sides on different clock domains, which means clock-domain-crossing logic and Gray-coded pointers.
- **Almost-full / almost-empty flags** to give surrounding logic early warning before the hard limits are hit.
- **Parameterize the data type**, not just the depth, so the same FIFO can buffer things other than `parsed_msg_t`.
