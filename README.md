Verilog "1011" Sequence Detector (Mealy & Moore FSM)📖 OverviewThis repository provides a deep dive into digital logic design through a classic project: a "1011" sequence detector. It contains synthesizable Verilog code for two distinct implementations using Finite State Machines (FSMs): a Mealy machine and a Moore machine.This project serves as a practical and educational resource for students, hobbyists, and engineers who are learning or working with digital logic, FSM design, and hardware description languages (HDLs). It demonstrates how the same problem can be solved with different design philosophies, each with its own trade-offs in timing and complexity.💡 Project MotivationThis project was developed as a hands-on exercise to solidify my understanding of fundamental digital design concepts, specifically the theory and application of Finite State Machines (FSMs). While a sequence detector is a classic academic problem, implementing it from scratch in Verilog provides invaluable experience in:State-Based Logic: Translating a state diagram into synthesizable hardware description code.Design Trade-offs: Directly comparing the architectural differences between Mealy and Moore FSMs, observing their unique impacts on output timing and resource complexity.Verification and Simulation: Writing comprehensive testbenches to rigorously validate a design against various edge cases, including overlapping sequences.By building and testing both FSM types, this repository serves not only as a portfolio piece but also as a clear, practical demonstration of core digital engineering principles.✨ Key FeaturesDual FSM Implementations: Includes both Mealy and Moore state machine designs for comprehensive learning.Overlapping Sequence Detection: Both designs correctly identify overlapping sequences (e.g., detecting "1011" twice in the input stream 1011011).Fully Synthesizable Code: The Verilog modules are written in a synthesizable subset of the language, ready for implementation on FPGAs or ASICs.Self-Contained Testbenches: Each design is paired with a thorough testbench to verify its functionality through simulation.In-Depth Documentation: A complete guide covering theory, installation, simulation, and code explanation.📊 Simulation WaveformBelow is an example simulation waveform from the Mealy FSM testbench. Notice how the detected signal goes high as soon as the final '1' of the "1011" sequence arrives, showcasing the immediate output behavior of a Mealy machine.🧠 Verilog Source Code & FSM TheoryThis section contains the complete, commented Verilog code for both FSM implementations.1. Mealy Machine ImplementationIn a Mealy machine, the output is a function of both the current state and the current input. This allows the machine to react immediately to input changes, often resulting in faster detection at the cost of potentially asynchronous output timing relative to the clock.<details><summary><strong>Click to expand Mealy FSM Verilog Code (sequence_detector_mealy.v)</strong></summary>//-----------------------------------------------------------------------------
// Design: 1011 Sequence Detector (Mealy Machine)
//
// Description:
// This module implements a Mealy FSM to detect the overlapping sequence "1011".
// The output 'detected' goes high as soon as the final '1' of the sequence
// is received on the input.
//-----------------------------------------------------------------------------
module sequence_detector_mealy (
    input  wire       clk,
    input  wire       reset,
    input  wire       data_in,
    output reg        detected
);

    // State declaration using parameters for readability
    parameter S_IDLE = 2'b00;
    parameter S_1    = 2'b01;
    parameter S_10   = 2'b10;
    parameter S_101  = 2'b11;

    // State registers
    reg [1:0] current_state, next_state;

    // Combinational Logic for State Transitions
    always @(current_state or data_in) begin
        case (current_state)
            S_IDLE: next_state = data_in ? S_1 : S_IDLE;
            S_1:    next_state = data_in ? S_1 : S_10;
            S_10:   next_state = data_in ? S_101 : S_IDLE;
            S_101:  next_state = data_in ? S_1 : S_10; // Handles overlapping sequences
            default: next_state = S_IDLE;
        endcase
    end

    // Combinational Logic for Output (Mealy Characteristic)
    // Output depends on current state AND current input
    always @(current_state or data_in) begin
        if ((current_state == S_101) && (data_in == 1'b1)) begin
            detected = 1'b1;
        end else begin
            detected = 1'b0;
        end
    end

    // Sequential Logic for State Register Update
    always @(posedge clk or posedge reset) begin
        if (reset)
            current_state <= S_IDLE;
        else
            current_state <= next_state;
    end
endmodule

//-----------------------------------------------------------------------------
// Testbench for Mealy Sequence Detector
//-----------------------------------------------------------------------------
module tb_sequence_detector_mealy;
    reg clk, reset, data_in;
    wire detected;

    // Instantiate the Unit Under Test (UUT)
    sequence_detector_mealy uut (
        .clk(clk), .reset(reset), .data_in(data_in), .detected(detected)
    );

    // Clock generation
    initial begin
        clk = 0;
        forever #5 clk = ~clk; // 100MHz clock
    end

    // Test sequence execution
    initial begin
        reset = 1; data_in = 0;
        #15;
        reset = 0;
        $display("Time\tCLK\tRESET\tDATA_IN\tDETECTED");
        $monitor("%3d\t%b\t%b\t%b\t%b", $time, clk, reset, data_in, detected);

        // Test case 1: Simple 1011
        #10 data_in = 1; #10 data_in = 0; #10 data_in = 1; #10 data_in = 1; // Detect
        #10 data_in = 0;

        // Test case 2: Overlapping sequence 1011011
        #20 data_in = 1; #10 data_in = 0; #10 data_in = 1; #10 data_in = 1; // Detect 1
        #10 data_in = 0; #10 data_in = 1; #10 data_in = 1; // Detect 2
        #10 data_in = 0;
        #20 $finish;
    end
endmodule
</details>2. Moore Machine ImplementationIn a Moore machine, the output is a function only of the current state. This results in an output that is stable for the entire clock cycle and is perfectly synchronized with the clock, though it may introduce a one-cycle latency compared to a Mealy machine.<details><summary><strong>Click to expand Moore FSM Verilog Code (sequence_detector_moore.v)</strong></summary>//-----------------------------------------------------------------------------
// Design: 1011 Sequence Detector (Moore Machine)
//
// Description:
// This module implements a Moore FSM to detect the overlapping sequence "1011".
// The output 'detected' goes high for one full clock cycle when the FSM is in
// the final state.
//-----------------------------------------------------------------------------
module sequence_detector_moore (
    input  wire       clk,
    input  wire       reset,
    input  wire       data_in,
    output wire       detected
);

    // State declaration, including a dedicated final state for the output
    parameter S_IDLE = 3'b000;
    parameter S_1    = 3'b001;
    parameter S_10   = 3'b010;
    parameter S_101  = 3'b011;
    parameter S_1011 = 3'b100; // State where output is high

    // State registers
    reg [2:0] current_state, next_state;

    // Combinational Logic for State Transitions
    always @(current_state or data_in) begin
        case (current_state)
            S_IDLE: next_state = data_in ? S_1 : S_IDLE;
            S_1:    next_state = data_in ? S_1 : S_10;
            S_10:   next_state = data_in ? S_101 : S_IDLE;
            S_101:  next_state = data_in ? S_1011 : S_10;
            S_1011: next_state = data_in ? S_1 : S_10; // Handles overlapping sequences
            default: next_state = S_IDLE;
        endcase
    end

    // Sequential Logic for State Register Update
    always @(posedge clk or posedge reset) begin
        if (reset)
            current_state <= S_IDLE;
        else
            current_state <= next_state;
    end

    // Output Logic (Moore Characteristic)
    // Output depends only on the current state
    assign detected = (current_state == S_1011);

endmodule

//-----------------------------------------------------------------------------
// Testbench for Moore Sequence Detector
//-----------------------------------------------------------------------------
module tb_sequence_detector_moore;
    reg clk, reset, data_in;
    wire detected;

    // Instantiate the Unit Under Test (UUT)
    sequence_detector_moore uut (
        .clk(clk), .reset(reset), .data_in(data_in), .detected(detected)
    );

    // Clock generation
    initial begin
        clk = 0;
        forever #5 clk = ~clk; // 100MHz clock
    end

    // Test sequence execution
    initial begin
        reset = 1; data_in = 0;
        #15;
        reset = 0;
        $display("Time\tCLK\tRESET\tDATA_IN\tDETECTED");
        $monitor("%3d\t%b\t%b\t%b\t%b", $time, clk, reset, data_in, detected);

        // Test case 1: Simple 1011
        #10 data_in = 1; #10 data_in = 0; #10 data_in = 1; #10 data_in = 1; // State becomes S_1011
        #10 data_in = 0; // Output is high during this cycle

        // Test case 2: Overlapping sequence 1011011
        #20 data_in = 1; #10 data_in = 0; #10 data_in = 1; #10 data_in = 1; // State becomes S_1011
        #10 data_in = 0; // Detect 1 here
        #10 data_in = 1;
        #10 data_in = 1; // State becomes S_1011
        #10 data_in = 0; // Detect 2 here
        #20 $finish;
    end
endmodule
</details>🛠️ Setup and Simulation GuidePrerequisitesA computer running Windows or Linux.Xilinx Vivado ML Edition (the free version is sufficient). This guide uses version 2020.1, but newer versions will work.Step 1: Install Xilinx VivadoDownload: Navigate to the Vivado ML Edition downloads page.Create Account: You will need a free AMD/Xilinx account to download.Run Installer:Launch the web installer and sign in.Select Vivado ML Standard when prompted for the edition.In the customization screen, you can deselect device families you don't need to save disk space.Agree to the terms and proceed with the installation.Step 2: Run the Simulation in VivadoCreate a New Project:Open Vivado and click Create Project.Name your project (e.g., sequence_detector) and choose a location.Select RTL Project and check "Do not specify sources at this time".Skip adding constraints and choosing a default part. The simulation is device-independent.Finish the project creation wizard.Add Source Files:In the "Sources" panel on the left, right-click Design Sources and select Add Sources.Choose "Add or create design sources".Click Add Files, navigate to this repository's folder, and select either sequence_detector_mealy.v or sequence_detector_moore.v.Important: Check the box "Copy sources into project".Set as Top Module:Expand the Design Sources tree. Vivado will see both the design and testbench modules.Right-click on the design module (sequence_detector_mealy or sequence_detector_moore) and select Set as Top.Run the Simulation:In the left-hand Flow Navigator, click Run Simulation -> Run Behavioral Simulation.Vivado will launch its simulator, compile the files, and run the testbench.Analyze the Waveform:A waveform window will appear. Click the "Zoom Fit" icon (a magnifying glass with a box) to see the entire timeline.Examine the data_in, current_state, and detected signals to verify the FSM's behavior matches the theory.📜 LicenseThis project is open-source and licensed under the MIT License. You
