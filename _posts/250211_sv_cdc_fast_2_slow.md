# CDC from a fast clock domain to a slow clock domain



When transferring signals from a **fast clock domain to a slow clock domain** in **Verilog**, you must ensure reliable synchronization to prevent metastability and data loss. Here’s how to design a **Clock Domain Crossing (CDC) module** from a **fast clock to a slow clock** using **a dual flip-flop synchronizer and a handshake mechanism**.

---

##  **Key Techniques for Fast-to-Slow Clock Synchronization**
1. **Avoid metastability**  
   - Use a **two-stage synchronizer** in the slow clock domain.
2. **Ensure correct data transfer**  
   - If transferring a **single-bit control signal** (e.g., a pulse or flag), use **a pulse synchronizer**.
   - If transferring **multi-bit data**, use **a handshake mechanism** to avoid data loss.

---

## **Method 1: Single-Bit Pulse Synchronization**
For synchronizing a **single-bit signal** (e.g., an event trigger), use a **toggle flip-flop** in the fast clock and a **dual flip-flop synchronizer** in the slow clock.

- Toggler in fast clock domain
- 2 F/F synchronizer in slow clock domain

### **Verilog Code for Single-Bit Synchronization**
```verilog
module cdc_fast_to_slow (
    input  wire fast_clk,    // Fast clock domain
    input  wire slow_clk,    // Slow clock domain
    input  wire rst_n,       // Active-low reset
    input  wire fast_pulse,  // Single-bit event in fast clock
    output reg  slow_pulse   // Synchronized pulse in slow clock
);

    // Toggle flip-flop in fast clock domain
    reg fast_toggle;
    always @(posedge fast_clk or negedge rst_n) begin
        if (!rst_n)
            fast_toggle <= 1'b0;
        else if (fast_pulse)
            fast_toggle <= ~fast_toggle;
    end

    // Dual F/F Synchronizers in slow clock domain
    reg sync1, sync2;
    always @(posedge slow_clk or negedge rst_n) begin
        if (!rst_n) begin
            sync1 <= 1'b0;
            sync2 <= 1'b0;
            slow_pulse <= 1'b0;
        end else begin
            sync1 <= fast_toggle; // First-stage synchronizer
            sync2 <= sync1;       // Second-stage synchronizer
            slow_pulse <= sync1 ^ sync2; // Detect toggle change
        end
    end

endmodule
```

### **Explanation**
1. A **toggle flip-flop** in the fast clock domain changes state (`0 → 1` or `1 → 0`) on every `fast_pulse` event.
2. The toggle signal is **double-synchronized** into the slow clock domain.
3. The slow clock detects a **toggle edge** (`sync1 ^ sync2`) and generates a **one clock cycle pulse** in the slow domain.





---

## 🔹 **Method 2: Multi-Bit Data Transfer Using Handshake**
For transferring **multi-bit data**, use a **handshake mechanism** to ensure that the slow clock reads the data correctly before the next fast clock update.

### **Verilog Code for Multi-Bit Synchronization (Handshake)**
```verilog
module cdc_data_fast_to_slow (
    input  wire fast_clk,      // Fast clock domain
    input  wire slow_clk,      // Slow clock domain
    input  wire rst_n,         // Active-low reset
    input  wire [7:0] fast_data, // 8-bit data from fast clock
    input  wire fast_valid,    // Data valid signal from fast clock
    output reg  slow_ready,    // Ready signal in slow clock
    output reg  [7:0] slow_data // Synchronized data in slow clock
);

    reg [7:0] data_reg;
    reg fast_valid_d, handshake_req, handshake_ack;

    // Capture data in fast clock domain and generate handshake request
    always @(posedge fast_clk or negedge rst_n) begin
        if (!rst_n) begin
            fast_valid_d <= 0;
            handshake_req <= 0;
        end else begin
            fast_valid_d <= fast_valid;
            if (fast_valid && !fast_valid_d) begin
                data_reg <= fast_data;
                handshake_req <= 1'b1;  // Assert handshake request
            end else if (handshake_ack) begin
                handshake_req <= 1'b0;  // Clear handshake request after acknowledgment
            end
        end
    end

    // Synchronize handshake request to slow clock domain
    reg handshake_req_sync1, handshake_req_sync2;
    always @(posedge slow_clk or negedge rst_n) begin
        if (!rst_n) begin
            handshake_req_sync1 <= 0;
            handshake_req_sync2 <= 0;
            slow_ready <= 0;
            slow_data  <= 0;
        end else begin
            handshake_req_sync1 <= handshake_req;
            handshake_req_sync2 <= handshake_req_sync1;
            if (handshake_req_sync1 && !handshake_req_sync2) begin
                slow_data <= data_reg;  // Capture the synchronized data
                slow_ready <= 1'b1;     // Indicate data is ready
            end else begin
                slow_ready <= 1'b0;
            end
        end
    end

    // Synchronize handshake acknowledgment back to fast clock domain
    always @(posedge fast_clk or negedge rst_n) begin
        if (!rst_n)
            handshake_ack <= 0;
        else
            handshake_ack <= slow_ready;  // Acknowledge when slow domain captures data
    end

endmodule
```

### **Explanation**
1. **Fast clock domain**
   - Captures `fast_data` when `fast_valid` is asserted.
   - Generates a `handshake_req` signal.
2. **Slow clock domain**
   - Detects the `handshake_req` using a **dual flip-flop synchronizer**.
   - Captures `fast_data` into `slow_data` when the handshake request is detected.
   - Asserts `slow_ready` to indicate successful data transfer.
3. **Synchronization back to fast clock**
   - `slow_ready` is synchronized back to **fast_clk** as `handshake_ack`.
   - The fast clock clears `handshake_req`, allowing the next data transfer.

---

## 🏆 **Which Method to Use?**
| Use Case | Best Method |
|----------|------------|
| Single-bit control signal (e.g., flag, event trigger) | **Toggle flip-flop & dual flip-flop synchronizer** |
| Multi-bit data transfer (e.g., bus signals) | **Handshake mechanism** |
| Continuous high-speed data transfer | **Asynchronous FIFO** |

Would you like an **asynchronous FIFO implementation** for even faster multi-bit data transfer? 🚀