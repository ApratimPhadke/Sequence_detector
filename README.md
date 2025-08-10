````markdown
# Verilog "1011" Sequence Detector (Mealy & Moore FSM)

## 📖 Overview
This repository provides a deep dive into digital logic design through a classic project: a **"1011" sequence detector**.  
It contains synthesizable Verilog code for two distinct implementations using **Finite State Machines (FSMs)**:  
- **Mealy machine**
- **Moore machine**

This project serves as a practical and educational resource for students, hobbyists, and engineers who are learning or working with digital logic, FSM design, and hardware description languages (HDLs). It demonstrates how the same problem can be solved with different design philosophies, each with its own trade-offs in **timing** and **complexity**.

---

## 💡 Project Motivation
This project was developed as a **hands-on exercise** to solidify my understanding of **fundamental digital design concepts**, specifically the theory and application of **Finite State Machines (FSMs)**.

While a sequence detector is a classic academic problem, implementing it from scratch in Verilog provides invaluable experience in:

- **State-Based Logic:** Translating a state diagram into synthesizable hardware description code.  
- **Design Trade-offs:** Directly comparing the architectural differences between Mealy and Moore FSMs.  
- **Verification and Simulation:** Writing comprehensive testbenches to validate the design against various edge cases (including overlapping sequences).  

By building and testing both FSM types, this repository serves not only as a **portfolio piece** but also as a **practical demonstration** of core digital engineering principles.

---

## ✨ Key Features
- **Dual FSM Implementations:** Mealy and Moore state machines for comprehensive learning.
- **Overlapping Sequence Detection:** Correctly detects overlapping sequences (e.g., `1011011`).
- **Fully Synthesizable Code:** Ready for FPGA or ASIC implementation.
- **Self-Contained Testbenches:** Verify functionality through simulation.
- **In-Depth Documentation:** Covers theory, installation, simulation, and explanation.

---

## 📊 Simulation Waveform
Below is an example simulation waveform from the **Mealy FSM** testbench.  
Notice how the `detected` signal goes high **as soon as** the final `1` of the `1011` sequence arrives, showcasing the immediate output behavior of a Mealy machine.

---

## 🧠 Verilog Source Code & FSM Theory

### 1️⃣ Mealy Machine Implementation
In a **Mealy machine**, the output is a function of both the **current state** and the **current input**.  
This allows immediate reaction to input changes, often resulting in faster detection but with potentially asynchronous output timing relative to the clock.

<details>
<summary><strong>Click to expand Mealy FSM Verilog Code (sequence_detector_mealy.v)</strong></summary>

```verilog
//-----------------------------------------------------------------------------
// Design: 1011 Sequence Detector (Mealy Machine)
//-----------------------------------------------------------------------------
module sequence_detector_mealy (
    input  wire       clk,
    input  wire       reset,
    input  wire       data_in,
    output reg        detected
);

    // State declaration
    parameter S_IDLE = 2'b00;
    parameter S_1    = 2'b01;
    parameter S_10   = 2'b10;
    parameter S_101  = 2'b11;

    reg [1:0] current_state, next_state;

    // State Transitions
    always @(current_state or data_in) begin
        case (current_state)
            S_IDLE: next_state = data_in ? S_1 : S_IDLE;
            S_1:    next_state = data_in ? S_1 : S_10;
            S_10:   next_state = data_in ? S_101 : S_IDLE;
            S_101:  next_state = data_in ? S_1 : S_10;
            default: next_state = S_IDLE;
        endcase
    end

    // Output Logic (Mealy)
    always @(current_state or data_in) begin
        detected = ((current_state == S_101) && (data_in == 1'b1));
    end

    // State Register Update
    always @(posedge clk or posedge reset) begin
        if (reset)
            current_state <= S_IDLE;
        else
            current_state <= next_state;
    end
endmodule

//-----------------------------------------------------------------------------
// Testbench
//-----------------------------------------------------------------------------
module tb_sequence_detector_mealy;
    reg clk, reset, data_in;
    wire detected;

    sequence_detector_mealy uut (
        .clk(clk), .reset(reset), .data_in(data_in), .detected(detected)
    );

    // Clock
    initial begin
        clk = 0;
        forever #5 clk = ~clk;
    end

    // Test
    initial begin
        reset = 1; data_in = 0;
        #15 reset = 0;
        $display("Time\tCLK\tRESET\tDATA_IN\tDETECTED");
        $monitor("%3d\t%b\t%b\t%b\t%b", $time, clk, reset, data_in, detected);

        // 1011
        #10 data_in = 1; #10 data_in = 0; #10 data_in = 1; #10 data_in = 1;
        #10 data_in = 0;

        // 1011011
        #20 data_in = 1; #10 data_in = 0; #10 data_in = 1; #10 data_in = 1;
        #10 data_in = 0; #10 data_in = 1; #10 data_in = 1;
        #10 data_in = 0;

        #20 $finish;
    end
endmodule
````

</details>

---

### 2️⃣ Moore Machine Implementation

In a **Moore machine**, the output is based **only** on the **current state**.
This results in an output that is perfectly synchronized with the clock but introduces a **one-cycle delay**.

<details>
<summary><strong>Click to expand Moore FSM Verilog Code (sequence_detector_moore.v)</strong></summary>

```verilog
//-----------------------------------------------------------------------------
// Design: 1011 Sequence Detector (Moore Machine)
//-----------------------------------------------------------------------------
module sequence_detector_moore (
    input  wire       clk,
    input  wire       reset,
    input  wire       data_in,
    output wire       detected
);

    // State declaration
    parameter S_IDLE  = 3'b000;
    parameter S_1     = 3'b001;
    parameter S_10    = 3'b010;
    parameter S_101   = 3'b011;
    parameter S_1011  = 3'b100;

    reg [2:0] current_state, next_state;

    // State Transitions
    always @(current_state or data_in) begin
        case (current_state)
            S_IDLE:  next_state = data_in ? S_1 : S_IDLE;
            S_1:     next_state = data_in ? S_1 : S_10;
            S_10:    next_state = data_in ? S_101 : S_IDLE;
            S_101:   next_state = data_in ? S_1011 : S_10;
            S_1011:  next_state = data_in ? S_1 : S_10;
            default: next_state = S_IDLE;
        endcase
    end

    // State Register Update
    always @(posedge clk or posedge reset) begin
        if (reset)
            current_state <= S_IDLE;
        else
            current_state <= next_state;
    end

    // Output Logic (Moore)
    assign detected = (current_state == S_1011);

endmodule

//-----------------------------------------------------------------------------
// Testbench
//-----------------------------------------------------------------------------
module tb_sequence_detector_moore;
    reg clk, reset, data_in;
    wire detected;

    sequence_detector_moore uut (
        .clk(clk), .reset(reset), .data_in(data_in), .detected(detected)
    );

    initial begin
        clk = 0;
        forever #5 clk = ~clk;
    end

    initial begin
        reset = 1; data_in = 0;
        #15 reset = 0;
        $display("Time\tCLK\tRESET\tDATA_IN\tDETECTED");
        $monitor("%3d\t%b\t%b\t%b\t%b", $time, clk, reset, data_in, detected);

        // 1011
        #10 data_in = 1; #10 data_in = 0; #10 data_in = 1; #10 data_in = 1;
        #10 data_in = 0;

        // 1011011
        #20 data_in = 1; #10 data_in = 0; #10 data_in = 1; #10 data_in = 1;
        #10 data_in = 0; #10 data_in = 1; #10 data_in = 1;
        #10 data_in = 0;

        #20 $finish;
    end
endmodule
```

</details>

---

## 🛠️ Setup and Simulation Guide

### **Prerequisites**

* Windows or Linux PC
* Xilinx Vivado ML Edition (**Free version works**) — Tested with Vivado 2020.1

### **Steps**

1. **Install Vivado** from AMD/Xilinx official website.
2. **Create New Project** → RTL Project → No constraints → Device-independent.
3. **Add Source Files** → Choose `sequence_detector_mealy.v` or `sequence_detector_moore.v`.
4. **Set as Top Module**.
5. **Run Simulation** → Behavioral Simulation.
6. **View Waveform** and verify `data_in`, `current_state`, and `detected`.

---

## 📜 License

This project is open-source under the **MIT License**.

