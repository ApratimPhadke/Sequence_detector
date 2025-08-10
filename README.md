Verilog "1011" Sequence Detector (Mealy & Moore FSM)📖 OverviewThis repository provides a deep dive into digital logic design through a classic project: a "1011" sequence detector. It contains synthesizable Verilog code for two distinct implementations using Finite State Machines (FSMs): a Mealy machine and a Moore machine.This project serves as a practical and educational resource for students, hobbyists, and engineers who are learning or working with digital logic, FSM design, and hardware description languages (HDLs). It demonstrates how the same problem can be solved with different design philosophies, each with its own trade-offs in timing and complexity.✨ Key FeaturesDual FSM Implementations: Includes both Mealy and Moore state machine designs for comprehensive learning.Overlapping Sequence Detection: Both designs correctly identify overlapping sequences (e.g., detecting "1011" twice in the input stream 1011011).Fully Synthesizable Code: The Verilog modules are written in a synthesizable subset of the language, ready for implementation on FPGAs or ASICs.Self-Contained Testbenches: Each design is paired with a thorough testbench to verify its functionality through simulation.In-Depth Documentation: A complete guide covering theory, installation, simulation, and code explanation.📊 Simulation WaveformBelow is an example simulation waveform from the Mealy FSM testbench. Notice how the detected signal goes high as soon as the final '1' of the "1011" sequence arrives, showcasing the immediate output behavior of a Mealy machine.📁 Repository Structure.
├── sequence_detector_mealy.v   # Verilog source & testbench for the Mealy FSM
├── sequence_detector_moore.v   # Verilog source & testbench for the Moore FSM
└── README.md                   # This detailed documentation file
🧠 Technical Deep Dive: FSM TheoryA sequence detector is a sequential circuit that outputs a '1' when a specific pattern of bits is detected in a serial input stream. The core of such a circuit is a Finite State Machine (FSM).Mealy MachineIn a Mealy machine, the output is a function of both the current state and the current input. This allows the machine to react immediately to input changes.State Diagram (Mealy):           Input: 1
         +----------+
         |          |
         v          |
      +-------+  Input: 1  +--------+  Input: 1  +---------+
(START)-->| S_IDLE|----------->|  S_1   |----------->|  S_101  |
      +-------+            +--------+            +---------+
         ^  | Input: 0       | Input: 0            | Input: 0
         |  +----------------+                     |
         |                                         |
         |                                         v
         +--------------------------------------+--------+
                                                |  S_10  |
                                                +--------+
                                                   ^
                                                   | Input: 1 / DETECT = 1
                                                   |
                                                (from S_101)
<details><summary><strong>Click to view Mealy FSM Verilog Code (sequence_detector_mealy.v)</strong></summary>// Design: 1011 Sequence Detector (Mealy Machine)
module sequence_detector_mealy (
    input  wire       clk,
    input  wire       reset,
    input  wire       data_in,
    output reg        detected
);

    // State declaration
    parameter S_IDLE = 2'b00, S_1 = 2'b01, S_10 = 2'b10, S_101 = 2'b11;

    // State registers
    reg [1:0] current_state, next_state;

    // State Transition Logic (Combinational)
    always @(current_state or data_in) begin
        case (current_state)
            S_IDLE: next_state = data_in ? S_1 : S_IDLE;
            S_1:    next_state = data_in ? S_1 : S_10;
            S_10:   next_state = data_in ? S_101 : S_IDLE;
            S_101:  next_state = data_in ? S_1 : S_10;
            default: next_state = S_IDLE;
        endcase
    end

    // Output Logic (Combinational - Mealy characteristic)
    always @(current_state or data_in) begin
        if ((current_state == S_101) && (data_in == 1'b1)) begin
            detected = 1'b1;
        end else begin
            detected = 1'b0;
        end
    end

    // State Register Update (Sequential)
    always @(posedge clk or posedge reset) begin
        if (reset)  current_state <= S_IDLE;
        else        current_state <= next_state;
    end
endmodule

// Testbench for Mealy FSM
module tb_sequence_detector_mealy;
    reg clk, reset, data_in;
    wire detected;

    sequence_detector_mealy uut (.clk(clk), .reset(reset), .data_in(data_in), .detected(detected));

    initial begin
        clk = 0;
        forever #5 clk = ~clk;
    end

    initial begin
        reset = 1; data_in = 0;
        #15;
        reset = 0;
        $display("Time\tCLK\tRESET\tDATA_IN\tDETECTED");
        $monitor("%3d\t%b\t%b\t%b\t%b", $time, clk, reset, data_in, detected);

        // Test case: 1011
        #10 data_in = 1; #10 data_in = 0; #10 data_in = 1; #10 data_in = 1;
        #10 data_in = 0;

        // Test case: Overlapping 1011011
        #20 data_in = 1; #10 data_in = 0; #10 data_in = 1; #10 data_in = 1; // Detect 1
        #10 data_in = 0; #10 data_in = 1; #10 data_in = 1; // Detect 2
        #10 data_in = 0;
        #10 $finish;
    end
endmodule
</details>Moore MachineIn a Moore machine, the output is a function only of the current state. The output is stable throughout the entire clock cycle and only changes after a state transition on a clock edge.State Diagram (Moore):           Input: 1
         +----------+
         |          |
         v          |
      +-------+  Input: 1  +--------+  Input: 1  +---------+
(START)-->| S_IDLE|----------->|  S_1   |----------->|  S_101  |
      | (D=0) |            | (D=0)  |            | (D=0)   |
      +-------+            +--------+            +---------+
         ^  | Input: 0       | Input: 0            | Input: 1
         |  +----------------+                     |
         |                                         v
         +--------------------------------------+--------+
                                                | S_1011 |
                                                | (D=1)  |
                                                +--------+
<details><summary><strong>Click to view Moore FSM Verilog Code (sequence_detector_moore.v)</strong></summary>// Design: 1011 Sequence Detector (Moore Machine)
module sequence_detector_moore (
    input  wire       clk,
    input  wire       reset,
    input  wire       data_in,
    output wire       detected
);

    // State declaration
    parameter S_IDLE = 3'b000, S_1 = 3'b001, S_10 = 3'b010, S_101 = 3'b011, S_1011 = 3'b100;

    // State registers
    reg [2:0] current_state, next_state;

    // State Transition Logic (Combinational)
    always @(current_state or data_in) begin
        case (current_state)
            S_IDLE: next_state = data_in ? S_1 : S_IDLE;
            S_1:    next_state = data_in ? S_1 : S_10;
            S_10:   next_state = data_in ? S_101 : S_IDLE;
            S_101:  next_state = data_in ? S_1011 : S_10;
            S_1011: next_state = data_in ? S_1 : S_10;
            default: next_state = S_IDLE;
        endcase
    end

    // State Register Update (Sequential)
    always @(posedge clk or posedge reset) begin
        if (reset)  current_state <= S_IDLE;
        else        current_state <= next_state;
    end

    // Output Logic (Combinational - Moore characteristic)
    assign detected = (current_state == S_1011);

endmodule

// Testbench for Moore FSM
module tb_sequence_detector_moore;
    reg clk, reset, data_in;
    wire detected;

    sequence_detector_moore uut (.clk(clk), .reset(reset), .data_in(data_in), .detected(detected));

    initial begin
        clk = 0;
        forever #5 clk = ~clk;
    end

    initial begin
        reset = 1; data_in = 0;
        #15;
        reset = 0;
        $display("Time\tCLK\tRESET\tDATA_IN\tDETECTED");
        $monitor("%3d\t%b\t%b\t%b\t%b", $time, clk, reset, data_in, detected);

        // Test case: 1011
        #10 data_in = 1; #10 data_in = 0; #10 data_in = 1; #10 data_in = 1;
        #10 data_in = 0; // Output is high during this cycle

        // Test case: Overlapping 1011011
        #20 data_in = 1; #10 data_in = 0; #10 data_in = 1; #10 data_in = 1; // State becomes S_1011
        #10 data_in = 0; // Detect 1 here
        #10 data_in = 1;
        #10 data_in = 1; // State becomes S_1011
        #10 data_in = 0; // Detect 2 here
        #10 $finish;
    end
endmodule
</details>🛠️ Setup and Simulation GuidePrerequisitesA computer running Windows or Linux.Xilinx Vivado ML Edition (the free version is sufficient). This guide uses version 2020.1, but newer versions will work.Step 1: Install Xilinx VivadoDownload: Navigate to the Vivado ML Edition downloads page.Create Account: You will need a free AMD/Xilinx account to download.Run Installer:Launch the web installer and sign in.Select Vivado ML Standard when prompted for the edition.In the customization screen, you can deselect device families you don't need to save disk space.Agree to the terms and proceed with the installation.Step 2: Run the Simulation in VivadoCreate a New Project:Open Vivado and click Create Project.Name your project (e.g., sequence_detector) and choose a location.Select RTL Project and check "Do not specify sources at this time".Skip adding constraints and choosing a default part. The simulation is device-independent.Finish the project creation wizard.Add Source Files:In the "Sources" panel on the left, right-click Design Sources and select Add Sources.Choose "Add or create design sources".Click Add Files, navigate to this repository's folder, and select either sequence_detector_mealy.v or sequence_detector_moore.v.Important: Check the box "Copy sources into project".Set as Top Module:Expand the Design Sources tree. Vivado will see both the design and testbench modules.Right-click on the design module (sequence_detector_mealy or sequence_detector_moore) and select Set as Top.Run the Simulation:In the left-hand Flow Navigator, click Run Simulation -> Run Behavioral Simulation.Vivado will launch its simulator, compile the files, and run the testbench.Analyze the Waveform:A waveform window will appear. Click the "Zoom Fit" icon (a magnifying glass with a box) to see the entire timeline.Examine the data_in, current_state, and detected signals to verify the FSM's behavior matches the theory.📜 LicenseThis project is open-source and licensed under the MIT License. See the LICENSE file for more details. Feel free to use, modify, and distribute the code for
