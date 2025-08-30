---
layout: post
title: ITCH Parser Design in SystemVerilog
description: A low-latency market data feed handler designed in SystemVerilog for FPGA implementation. This project demonstrates finite state machine design and high-frequency trading protocols by parsing ITCH messages containing stock order information including message type, stock ID, order details, and pricing data.
skills: 
  - SystemVerilog
  - FPGA design
  - Finite State Machine (FSM)
  - Digital signal processing
  - High-frequency trading protocols
  - RTL design
  - Testbench development
  - Simulation and verification

main-image: /itch-parser.jpg
---

## Project Overview

I recently found out that trading firms value FPGA engineers because they are attracted by the high speed and parallel computing power provided by FPGAs. By leveraging this technology, trades can be made very fast, and if you ever heard of the phrase "time is money," that most certainly is the case when it comes to trading. One of the communication protocols used by trading firms is ITCH, which allows a full detailed order message to be transmitted at low latency. I decided to give the protocol a go by designing a low-latency market data feed handler in SystemVerilog.

## Message Structure Design

While an ITCH message can carry a lot of information about the type of trade that needs to be made, I decided to keep my implementation simple by using 16 byte messages that carry information about the type of message (add, delete, update stock), stock id, order id, order type (bid or ask), price and quantity. I store the byte-wise translated information in an output struct:

```systemverilog
typedef struct packed { //packed means no padding between fields
    msg_type_t msg_type; // 1 byte
    logic [7:0] stock_id; // 1 byte
    logic [31:0] order_id; // 4 bytes
    order_side_t order_side; // 1 byte
    logic [31:0] price; // 4 bytes
    logic [31:0] quantity; // 4 bytes
    logic [7:0] padding; // 1 byte
} parsed_msg_t;
```

### Message Fields Breakdown

- **MESSAGE TYPE** (ADD, DELETE, UPDATE)
- **STOCK ID** (2^8 = 255 different stocks)
- **ORDER ID** (2^32 different order numbers)
- **ORDER SIDE** (BID or SELL)
- **PRICE** (up to $2^32)
- **QUANTITY** (up to 2^32)
- **PADDING**

## Finite State Machine Design

After deciding on what the incoming message looks like, we need a way to parse this information with the help of a finite state machine that decodes the message byte by byte:

```systemverilog
typedef enum logic [3:0] {
    IDLE,
    MSG_TYPE,
    STOCK_ID,
    ORDER_ID,
    ORDER_SIDE,
    PRICE,
    QUANTITY,
    PADDING,
    DONE
} parser_state_t;
```

So the FSM goes from an IDLE state to decoding the message for different values based on whether certain conditions are met.

## Message Type and Order Side Definitions

We also need a way to know what the message type and order side correspond to, which we can do with the help of two enumerations:

```systemverilog
typedef enum logic[7:0] {
    MSG_ADD = 8'h41, // 'A'
    MSG_DELETE = 8'h44, // 'D'
    MSG_UPDATE = 8'h55, // 'U'
    MSG_NULL = 8'hFF // for unrecognized order types
} msg_type_t;

// Order sides
typedef enum logic[7:0] {
    ORDER_SIDE_BID = 8'h42, //'B'
    ORDER_SIDE_ASK = 8'h41, //'A'
    ORDER_SIDE_UNKNOWN = 8'hFF // for unrecognized order sides
} order_side_t;
```

## Example Message Parsing

Now that we have these definitions set, we can expect a 16 byte message such as `0x41 01 00 00 00 01 42 00 00 01 F4 00 00 00 0A 00` to translate into:

> "Add stock to order book. Stock has an id of 1. The order number is 1. We want to bid the stock. Bid at $500. Bid 10 stocks."

## Parser Module Implementation

In order to actually carry out the parsing, we can use a module named `parser_fsm`, which is declared as:

```systemverilog
module parser_fsm (
    input logic clk,
    input logic reset,
    input logic byte_valid,
    input logic [7:0] byte_in,
    output parsed_msg_t parsed_msg, // struct to hold the parsed message
    output logic done
);
```

The full code can be found in my GitHub repository for the project, but what it does is that it changes state depending on the number of bytes read, and within each state it shifts data into a shift register and decodes the message the bytes imply. For example, after 1 byte is read as `MSG_ADD` in the `MSG_TYPE` state, move to `STOCK_ID` state. Similarly, since `order_id` is 4 bytes long, stay in state `ORDER_ID` until 4 bytes have been read per the declared `order_id` length.

## Simulation and Testing

Now with our RTL design complete, we simulate the design with my custom testbench. This leads to the following message in the tcl console and corresponding waveform in either Modelsim or Vivado:

### Test Vectors
```
Test_vectors.hex:
41 01 00 00 00 01 42 00 00 01 F4 00 00 00 0A 00
44 02 00 00 00 02 FF FF FF FF FF FF FF FF FF FF FF
55 03 00 00 00 03 41 00 00 03 E8 00 00 00 64 CD
```

{% include image-gallery.html images="itch-simulation-waveform.jpg" height="400" %}

Upon inspection in a simulation tool, we can conclude that the waveform shows expected behavior to our parser design!

## Reflection

ITCH is a fun communication protocol to implement. With my experience in embedded systems, I've explored UART, SPI, I2C (and even written blogs about them), but ITCH is in an entirely different field. It's fun to see how learning FPGA programming can lead Electrical Engineers to work with Finance Bros, so if you want to go into Finance but are reading this blog as an engineer, there is still a door open for you!

---

*This project bridges the gap between electrical engineering and financial technology, demonstrating how FPGA skills can be applied in high-frequency trading environments.*
