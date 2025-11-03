# SPI Slave System RTL Specification

## 1. Overview

This document describes a multi-domain SPI slave system capable of communicating over the SPI protocol (with CPOL/CPHA support), and interacting with multiple system clock domains through asynchronous FIFOs. One of the system domains supports Wishbone bus interface with SPI-triggered burst read/write operations.

## 2. Top-Level Block Diagram

(Refer to the generated SPI system architecture diagram.)

## 3. Interfaces

### 3.1 SPI Slave I/O

| Signal | Direction | Description               |
| ------ | --------- | ------------------------- |
| `sclk` | Input     | SPI Clock from Master     |
| `mosi` | Input     | Master Out, Slave In      |
| `miso` | Output    | Master In, Slave Out      |
| `ss_n` | Input     | Slave Select (Active Low) |
| `cpol` | Input     | SPI Clock Polarity        |
| `cpha` | Input     | SPI Clock Phase           |

### 3.2 System Domain I/F (per domain)

| Signal            | Direction | Description                   |
| ----------------- | --------- | ----------------------------- |
| `sys_clk[i]`      | Input     | Clock of domain `i`           |
| `sys_rst_n[i]`    | Input     | Reset for domain `i`          |
| `sys_tx_data[i]`  | Input     | Data to send to SPI master    |
| `sys_tx_wr_en[i]` | Input     | Write enable for TX FIFO      |
| `sys_tx_full[i]`  | Output    | TX FIFO full flag             |
| `sys_rx_data[i]`  | Output    | Data received from SPI master |
| `sys_rx_rd_en[i]` | Input     | Read enable for RX FIFO       |
| `sys_rx_empty[i]` | Output    | RX FIFO empty flag            |

### 3.3 Wishbone Interface (domain `i = WB`)

| Signal     | Direction | Description                        |
| ---------- | --------- | ---------------------------------- |
| `wb_clk`   | Input     | Wishbone Clock Domain              |
| `wb_rst_n` | Input     | Active-low Reset                   |
| `wb_cyc`   | Output    | Bus cycle asserted during burst    |
| `wb_stb`   | Output    | Strobe signal for data valid cycle |
| `wb_we`    | Output    | Write enable                       |
| `wb_addr`  | Output    | Address for access                 |
| `wb_data`  | Output    | Data to write                      |
| `wb_ack`   | Input     | Acknowledge from bus               |

## 4. Functional Description

### 4.1 SPI FSM (sclk domain)

- Handles CPOL/CPHA SPI protocol.
- Receives MOSI stream, assembles 8-bit word.
- Sends MISO stream using preloaded `tx_data`.
- Updates RX FIFO on full word.
- Reads from TX FIFO for every new byte transfer.
- Detects special command (`0xF8`) to initiate burst mode.

### 4.2 Domain Routing

- First byte received (`0xF0` ~ `0xF4`) selects domain `0~4`.
- `0xF8` triggers Wishbone burst mode.
- SPI burst sequence: CMD, LEN, ADDR_H, ADDR_L, DATA*
- TX data is sourced from the selected domain's TX FIFO.

### 4.3 Wishbone Burst Handling

- Burst commands are parsed in SPI FSM and pushed into a burst FIFO (sclk → wb_clk).
- Burst packet includes:
  - `wr`: 1-bit (write=1/read=0)
  - `addr`: 16-bit base address
  - `len`: 8-bit burst length
  - `data`: up to 16-bytes (128-bit max)
- Wishbone FSM reads from burst FIFO and performs sequential transactions.

## 5. Asynchronous FIFO (Per Domain)

### 5.1 Architecture

- Dual clock (write/read)
- Gray-coded pointer CDC
- `wr_clk` = SPI clock or system domain clock
- `rd_clk` = system domain clock or SPI clock

### 5.2 FIFO Parameters

| Parameter  | Value                             |
| ---------- | --------------------------------- |
| Data Width | 8 or 137 (for burst FIFO)         |
| Depth      | 8 (configurable via `ADDR_WIDTH`) |

## 6. Timing Diagrams

### 6.1 CPOL=0, CPHA=0 (Mode 0)

```
SCLK:  ____/\____/\____/\
MOSI:  ----<b7>---<b6>---<b5>---
Sample:  ^      ^      ^
```

### 6.2 Burst SPI Example

```
Master sends: 0xF8, 0x80, 0x03, 0x00, 0x10, 0xAA, 0xBB, 0xCC
Meaning: Write Burst, 3 bytes to addr 0x0010
```

## 7. Simulation Testbench

### Scenarios

- SPI Master sends `0xF0` to select domain 0
- Sends `0x55` → delivered to domain 0's RX FIFO
- Domain 0 preloaded `0xA5` → returned via MISO

### Burst Scenario

- SPI Master sends: `0xF8 0x80 0x03 0x00 0x10 0xAA 0xBB 0xCC`
- Wishbone domain writes 3 bytes to `0x0010` ~ `0x0012`

## 8. Extension Options

| Feature              | Description                                       |
| -------------------- | ------------------------------------------------- |
| LSB-first mode       | Optional serial direction                         |
| CRC support          | Add data integrity check                          |
| SPI command decoding | Define richer protocol                            |
| Broadcast            | Send one data to all domains                      |
| Burst Read           | Return data to SPI master over MISO from Wishbone |

## 9. Synthesis Notes

- SPI FSM uses `sclk` as edge detector, not a real clock
- async_fifo is fully CDC-safe and synthesisable
- Use `-name SDC_TIMING_GROUP` for cross-domain clocking constraints
- Burst packet width is 137 bits, ensure routing and LUT usage is optimized