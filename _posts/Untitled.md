- 

---

## **📝 Testbench for `cdc_data_fast_to_slow` (Multi-Bit Handshake CDC)**

```verilog
`timescale 1ns/1ps

module tb_cdc_data_fast_to_slow();

    reg fast_clk, slow_clk, rst_n;
    reg [7:0] fast_data;
    reg fast_valid;
    wire slow_ready;
    wire [7:0] slow_data;

    // Instantiate the DUT (Device Under Test)
    cdc_data_fast_to_slow uut (
        .fast_clk(fast_clk),
        .slow_clk(slow_clk),
        .rst_n(rst_n),
        .fast_data(fast_data),
        .fast_valid(fast_valid),
        .slow_ready(slow_ready),
        .slow_data(slow_data)
    );

    // Clock generation
    initial begin
        fast_clk = 0;
        forever #5 fast_clk = ~fast_clk; // 100 MHz clock (10 ns period)
    end

    initial begin
        slow_clk = 0;
        forever #20 slow_clk = ~slow_clk; // 25 MHz clock (40 ns period)
    end

    // Reset sequence
    initial begin
        rst_n = 0;
        fast_valid = 0;
        fast_data = 0;
        #50 rst_n = 1; // Release reset after 50 ns
    end

    // Stimulus
    initial begin
        #100;

        // Send data from fast to slow domain
        #30 fast_data = 8'hA5; fast_valid = 1; #10 fast_valid = 0;
        #100 fast_data = 8'h3C; fast_valid = 1; #10 fast_valid = 0;
        #200 fast_data = 8'h7F; fast_valid = 1; #10 fast_valid = 0;

        #500 $stop; // Stop simulation
    end

    // Monitor signals
    initial begin
        $dumpfile("cdc_data_fast_to_slow.vcd"); // GTKWave dump file
        $dumpvars(0, tb_cdc_data_fast_to_slow);
        $monitor($time, " fast_data=%h, fast_valid=%b, slow_data=%h, slow_ready=%b", fast_data, fast_valid, slow_data, slow_ready);
    end

endmodule
```

### **🔹 Explanation**

- Applies `fast_data` and asserts `fast_valid` for one cycle.
- Monitors `slow_data` and `slow_ready` to confirm successful transfer.
- Dumps simulation data for **GTKWave** waveform analysis.

---

## **🏆 Running the Testbench in ModelSim**

### **1️⃣ Compile the Design**

```tcl
vlog cdc_fast_to_slow.v tb_cdc_fast_to_slow.v
```

or

```tcl
vlog cdc_data_fast_to_slow.v tb_cdc_data_fast_to_slow.v
```

### **2️⃣ Run the Simulation**

```tcl
vsim work.tb_cdc_fast_to_slow
```

or

```tcl
vsim work.tb_cdc_data_fast_to_slow
```

### **3️⃣ Add Waveforms**

```tcl
add wave -position insertpoint sim:/tb_cdc_fast_to_slow/*
run -all
```

or

```tcl
add wave -position insertpoint sim:/tb_cdc_data_fast_to_slow/*
run -all
```

### **4️⃣ View Waveform in GTKWave (Optional)**

```sh
gtkwave cdc_fast_to_slow.vcd
```

or

```sh
gtkwave cdc_data_fast_to_slow.vcd
```

---

## **🔍 Expected Waveform Results**

✅ **For `cdc_fast_to_slow` (Single-Bit Pulse)**:

- `fast_pulse` toggles in the **fast clock domain**.
- `slow_pulse` detects **rising edges** correctly in the **slow clock domain**.

✅ **For `cdc_data_fast_to_slow` (Multi-Bit Handshake)**:

- `fast_data` updates when `fast_valid=1`.
- `slow_data` captures `fast_data` after a few slow clock cycles.
- `slow_ready` indicates successful reception.

---

## **🎯 Summary**

- **Single-bit CDC** → Uses a **toggle synchronizer**.
- **Multi-bit CDC** → Uses a **handshake mechanism**.
- **ModelSim testbench** verifies correct synchronization.

Would you like a **FIFO-based CDC testbench** for continuous data transfer? 🚀