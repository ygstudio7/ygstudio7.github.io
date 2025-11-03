# SPI Slave System RTL Specification

## 1. Overview

This document describes a multi-domain SPI slave system capable of communicating over the SPI protocol (with CPOL/CPHA support), and interacting with multiple system clock domains through asynchronous FIFOs.

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

## 4. Functional Description

### 4.1 SPI FSM (sclk domain)

- Handles CPOL/CPHA SPI protocol.
- Receives MOSI stream, assembles 8-bit word.
- Sends MISO stream using preloaded `tx_data`.
- Updates RX FIFO on full word.
- Reads from TX FIFO for every new byte transfer.

### 4.2 Domain Routing

- First byte received (`0xF0` ~ `0xF4`) selects domain `0~4`.
- Subsequent MOSI data is routed to selected domain's RX FIFO.
- TX data is sourced from the selected domain's TX FIFO.

## 5. Asynchronous FIFO (Per Domain)

### 5.1 Architecture

- Dual clock (write/read)
- Gray-coded pointer CDC
- `wr_clk` = SPI clock or system domain clock
- `rd_clk` = system domain clock or SPI clock

### 5.2 FIFO Parameters

| Parameter  | Value                                  |
| ---------- | -------------------------------------- |
| Data Width | 8                                      |
| Depth      | 8 (can be configured via `ADDR_WIDTH`) |

## 6. Timing Diagrams

### 6.1 CPOL=0, CPHA=0 (Mode 0)

```
SCLK:  ____/\____/\____/\
MOSI:  ----<b7>---<b6>---<b5>---
Sample:  ^      ^      ^
```

## 7. Simulation Testbench

### Scenarios

- SPI Master sends `0xF0` to select domain 0
- Sends `0x55` → delivered to domain 0's RX FIFO
- Domain 0 preloaded `0xA5` → returned via MISO

## 8. Extension Options

| Feature              | Description                  |
| -------------------- | ---------------------------- |
| LSB-first mode       | Optional serial direction    |
| CRC support          | Add data integrity check     |
| SPI command decoding | Define richer protocol       |
| Broadcast            | Send one data to all domains |

## 9. Synthesis Notes

- SPI FSM uses `sclk` as edge detector, not a real clock
- async_fifo is fully CDC-safe and synthesisable
- Use `-name SDC_TIMING_GROUP` for cross-domain clocking constraints