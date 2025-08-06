---
layout: post
title: FIFO design in SystemVerilog
description:  short description of the project
skills: 
- SystemVerilog
- Vivado
- Digital design
- Circular queues
- Memory design
- Buffering
main-image: /FIFO.png
---

# FIFO Memory Buffer Implementation for ITCH Protocol in SystemVerilog

This project demonstrates the design and simulation of a **FIFO (First In, First Out) memory buffer** used in a low-latency stock market data feed handler based on the **ITCH protocol**. The FIFO decouples the **parser** from the **order book**, enabling smooth message handling even when downstream logic can't keep up.

📁 [Project GitHub Repository](https://github.com/mmzarif/market_data_parser)

---

## 🧠 Background

**ITCH** is a protocol used by stock exchanges to stream real-time order book updates. Parsing these messages quickly is essential, but storing them temporarily is just as critical to avoid loss or backpressure when the order book is busy.

To handle this, we insert a FIFO buffer between the parser and the order book:

```
ITCH Stream → Parser → FIFO → Order Book
```

---

## 🔧 FIFO Architecture

The FIFO is implemented as a **circular queue**, supporting synchronous read/write operations and parameterized message width and depth.

### Key Features:

- 16-entry buffer (parameterizable)
- Each entry is a custom 16-byte `parsed_msg_t` struct
- Circular indexing using `write_ptr` and `read_ptr`
- Full and Empty flags for safe reads/writes
- Resettable to initial state

---

## 🧱 Module Ports

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

## 🔄 Core Logic

### Internal Memory and Pointers

```systemverilog
parsed_msg_t fifo_mem [0:FIFO_DEPTH-1];
logic [3:0] write_ptr, read_ptr, count;
```

### Circular Write/Read

- `write_ptr` and `read_ptr` wrap back to 0 when they hit the buffer limit
- `count` tracks how many messages are currently stored

---

## 🧪 Simulation

Testbench simulates the FIFO with `FIFO_DEPTH = 4` for clarity.

📂 [Testbench](https://github.com/mmzarif/market_data_parser/blob/main/sim/msg_fifo_tb.sv)

### Results:

- ✅ `msg_out` remains undefined until `write_en` is asserted
- ✅ Messages are correctly read out in order when `read_en` is high
- ✅ FIFO handles overflow conditions and asserts `full`
- ✅ FIFO handles underflow conditions and asserts `empty`

---

## 📌 Key Takeaways

- ✅ FIFO buffers decouple timing-sensitive logic
- ✅ Circular queues are memory-efficient and simple to implement
- ✅ Parameterization allows scaling to match protocol requirements

---

## 🧠 Why It Matters

In high-frequency trading and real-time systems, **latency = money**. Proper FIFO buffering ensures:

- Messages aren't dropped if downstream logic stalls
- Data remains ordered and easy to access
- System remains modular and verifiable

---

**Author:** Mustahsin Zarif  
**Date:** August 2025
