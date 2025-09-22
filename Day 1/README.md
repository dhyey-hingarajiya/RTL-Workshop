# Introduction to Icarus Verilog, Design, and Testbench

### What is a Simulator?
A **simulator** is a software tool used to verify a digital circuit's design. It checks if the Register Transfer Level (RTL) design adheres to its specified requirements by simulating its behavior. In this context, we use **Icarus Verilog (iverilog)** as the simulation tool.

### What is a Design?
A **design** refers to the actual Verilog code, which can be a single file or a collection of files. This code describes the intended functionality of the digital circuit, ensuring it meets the required specifications.

### What is a Testbench?
A **testbench** is a Verilog module created to verify the correctness of the design. It's a setup that applies a specific stimulus (also known as test vectors) to the design's inputs and checks if the outputs match the expected results, thereby confirming its functionality.

### How a Simulator Works?
The simulation process is event-driven and follows a simple but crucial principle:
* The simulator continuously **monitors the input signals** for any changes.
* Whenever a **change is detected** in an input signal, the simulator **evaluates the output**.
* If there is **no change in the input**, the simulator does not re-evaluate the output, which optimizes the simulation process.

---

## Test Bench Architecture
The following diagram illustrates the relationship between the testbench and the design:

![Testbench Architecture](./assets/Test%20Bench%20Architecture.png)

* The **Stimulus Generator** produces input signals (Primary Inputs) for the design.
* The **Design** (or Device Under Test - DUT) processes these inputs.
* The **Stimulus Observer** captures the design's outputs (Primary Outputs) to verify them against expected values.

**Key Points:**
* A **design** can have one or more primary inputs and outputs.
* A **testbench (TB)**, however, does not have any external primary inputs or outputs; it encapsulates the design.

---
# Lab 1: Introduction to Lab Environment Setup

This lab session covers the setup of the necessary tools and files required for the course. The process involves cloning the required Git repositories.

### Cloning the Workshop Repository 📂

The main step is to clone the workshop repository which contains all the necessary lab files, libraries, and Verilog models.

1.  **Navigate to your working directory**: Use the `cd` command to navigate to the directory where you want to download the lab files. For example:
    ```bash
    cd ~/path/to/your/workspace
    ```

2.  **Clone the Repository**: Use the following `git clone` command to download the workshop files. This command is copied directly from Kunal Ghosh's GitHub page in the video.
    ```bash
    git clone [https://github.com/kunalg123/sky130RTLDesignAndSynthesisWorkshop.git](https://github.com/kunalg123/sky130RTLDesignAndSynthesisWorkshop.git)
    ```

After running this command, a new directory named `sky130RTLDesignAndSynthesisWorkshop` will be created in your current location.

### Understanding the Cloned Directory Structure

Inside the `sky130RTLDesignAndSynthesisWorkshop` directory, you'll find several important folders:

* **`my_lib`**: This folder contains the library files required for synthesis.
    * **`lib`**: Contains the Sky130 standard cell library files (`.lib`).
    * **`verilog_model`**: Contains the Verilog behavioral models for the standard cells.
* **`verilog_files`**: This folder contains all the source `.v` files and testbench files that will be used throughout the lab experiments.

---

## Icarus Verilog Simulation Flow
![](./assets/Simulation%20flow.png)
The simulation flow using Icarus Verilog and GTKWave involves the following steps:
1.  **Input Files**: The **Design** file (`.v`) and the **Testbench** file (`.v`) are provided as inputs to the **Icarus Verilog (iverilog)** compiler/simulator.
2.  **Simulation**: `iverilog` compiles and simulates the testbench, which in turn tests the design.
3.  **Output File**: The simulator generates a **Value Change Dump (`.vcd`)** file. This file logs all the signal value changes that occurred during the simulation.
4.  **Waveform Viewing**: The `.vcd` file is then opened using a waveform viewer like **GTKWave**. This tool visualizes the signal changes over time, allowing for detailed analysis and debugging of the design's behavior.

---

# Lab 2: Simulation with Icarus Verilog and GTKWave

This lab demonstrates how to simulate a Verilog design using **Icarus Verilog** and visualize the resulting waveforms with **GTKWave**. We will use a simple multiplexer (`good_mux.v`) as our example design.

### Simulation and Waveform Visualization Flow 🚀

##### 1. Navigate to the Design Directory
First, navigate to the directory containing all the Verilog source files for the labs.
```bash
cd sky130RTLDesignAndSynthesisWorkshop/verilog_files/
```
Inside this folder, you'll notice that for every design file (e.g., `good_mux.v`), there is a corresponding testbench file prefixed with `tb_` (e.g., `tb_good_mux.v`).

#### 2. Run the Icarus Verilog Simulation
To compile and simulate the design, use the `iverilog` command with both the design and its testbench file.
```bash
iverilog good_mux.v tb_good_mux.v
```
This command creates a compiled output file named `a.out`.

#### 3. Execute the Compiled File
Next, execute the `a.out` file to run the simulation and generate the waveform dump file.
```bash
./a.out
```
Running this produces a `tb_good_mux.vcd` file.

#### 4. Launch GTKWave
Finally, open the generated `.vcd` file with GTKWave to view the waveforms.
```bash
gtkwave tb_good_mux.vcd
```
![Observed Waveform](./assets/Lab2%20waveform.png)
### Code Breakdown
* **The Design**(`good_mux.v`): This file defines a simple 2-to-1 multiplexer. Based on the `sel` line, it routes one of the two inputs (`i0` or `i1`) to the output `y`.
* **The Testbench**(`tb_good_mux.v`): This file is responsible for testing the multiplexer. It instantiates the `good_mux` design, generates a **stimulus** by toggling the input signals over time, sets up the VCD file generation for waveform viewing, and controls the total simulation runtime.

---

# Logic Synthesis and Yosys

This section covers the fundamental concepts of logic synthesis and its practical implementation using the Yosys synthesis tool.


### Introduction to Logic Synthesis

Logic synthesis is the process of converting a high-level, abstract description of a circuit, known as a Register Transfer Level (RTL) design, into a physical implementation made of standard logic gates.

#### RTL Design vs. Gate-Level Netlist
* **RTL (Register Transfer Level) Design**: This is a **behavioral** representation of a digital circuit written in a Hardware Description Language (HDL) like Verilog. It describes *what* the circuit should do in terms of data flow between registers.
* **Synthesis**: This is the automated process that translates the RTL design into a **gate-level netlist**. The netlist is a structural description that specifies the exact logic gates (AND, OR, NOT, Flip-Flops) and their interconnections.


#### The Standard Cell Library (`.lib` file)
The key to synthesis is the **standard cell library**, provided in a `.lib` file.
* It's a collection of pre-designed, pre-characterized logic gates (AND, OR, D-Flip-Flop, etc.) for a specific manufacturing technology.
* Crucially, it contains different **"flavors"** of the same gate, such as **slow**, **medium**, and **fast** versions.

#### Why Different Gate Speeds (Flavors)? 🤔
A digital circuit's maximum operating speed (clock frequency) is determined by timing constraints. The synthesizer must select the right cell flavors to meet two critical timing checks: **setup time** and **hold time**.

##### Setup Time (Why we need FAST cells 🚀)
For data to be correctly captured by a flip-flop, it must arrive *before* the clock edge and be stable for a certain period. This means the total delay of the path must be less than the clock period.
* **Formula**: $T_{clk} > T_{CQ} + T_{combi} + T_{setup}$
* **Goal**: To achieve a high clock frequency (small $T_{clk}$), the combinational logic delay ($T_{combi}$) must be minimized.
* **Solution**: The synthesizer uses **fast cells** on critical paths to reduce delay and meet setup time requirements.

##### Hold Time (Why we need SLOW cells 🐢)
To prevent the *new* data from arriving too early and overwriting the *current* data before it's captured, the data path must have a minimum delay.
* **Formula**: $T_{hold} < T_{CQ} + T_{combi}$
* **Goal**: The total delay of the path must be greater than the flip-flop's hold time.
* **Solution**: If a path is too fast, it can cause a hold violation. The synthesizer intentionally inserts **slow cells** or buffers to add delay and fix these violations.

#### The Trade-off: Faster vs. Slower Cells
The choice between fast and slow cells involves a critical trade-off:
* **Faster Cells**:
    * **Pros**: Low delay.
    * **Cons**: Built with wider transistors, leading to **larger area** and **higher power consumption**.
* **Slower Cells**:
    * **Pros**: Built with narrower transistors, resulting in **smaller area** and **lower power consumption**.
    * **Cons**: High delay.

#### Guiding the Synthesizer with Constraints
Because of these trade-offs, we must guide the synthesizer using **constraints**. These constraints define the design goals, such as the target clock frequency, maximum area, and power limits. The synthesizer uses this guidance to select the optimal combination of cells.

---

# Introduction to Synthesis with Yosys

**Yosys** is the open-source synthesis tool used in this course to perform the RTL-to-netlist conversion.

#### The Synthesis Flow
![Synthesis Flow](./assets/Yosys%20flow.png)
The basic flow of synthesis involves taking two primary inputs and producing one output:
1.  **Inputs**:
    * **Design (RTL Verilog)**: The behavioral Verilog code for your circuit.
    * **Standard Cell Library (`.lib`)**: The file defining the available logic gates.
2.  **Process**:
    * **Yosys** reads the design and maps the RTL logic to the standard cells defined in the `.lib` file.
3.  **Output**:
    * **Netlist**: A Verilog file that describes the circuit as a collection of interconnected standard cells.


#### Key Yosys Commands
Inside the Yosys environment, the synthesis process is controlled by a sequence of commands:
* `read_verilog <design_file.v>`: Reads the RTL Verilog design file.
* `read_liberty -lib <library_file.lib>`: Reads the standard cell library.
* `write_verilog <output_netlist_file.v>`: Writes the synthesized gate-level netlist.

#### Verifying the Synthesis ✅
After synthesis, it's crucial to verify that the generated netlist has the same logical functionality as the original RTL design. This is done via **gate-level simulation**.
1.  **Inputs to Simulator**: The synthesized **Netlist** and the **exact same Testbench** used for the original RTL simulation.
2.  **Simulation**: The files are run through a simulator like Icarus Verilog.
3.  **Verification**: The output waveform is viewed in GTKWave. This waveform **must be identical** to the waveform generated from the pre-synthesis RTL simulation to confirm success. 

---

# Lab 3: Synthesis using Yosys and Sky130 PDKs

This lab demonstrates the practical steps of synthesizing a Verilog RTL design (`good_mux.v`) into a gate-level netlist using the Yosys synthesis tool and the Sky130 Process Design Kit (PDK).

### Synthesis Procedure ⚙️

The synthesis process involves launching Yosys and executing a series of commands to read the design, map it to a technology library, and write the final netlist.

#### 1. Launch Yosys
First, navigate to the `verilog_files` directory and start Yosys from the terminal.
```bash
yosys
```
#### 2. Read Library and Design
Once inside the Yosys prompt, load the standard cell library and the Verilog design file.
```tcl
# Read the standard cell library (.lib file)
yosys> read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib

# Read the Verilog RTL design file
yosys> read_verilog good_mux.v
```

#### 3. Synthesize and Map to Gates

Next, run the synthesis for the top-level module and map the generic logic to the specific gates available in the Sky130 library using the `abc` command.
```tcl
# Run synthesis for the 'good_mux' module
yosys> synth -top good_mux

# Map the synthesized design to the technology library
yosys> abc -liberty ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

### Visualizing the Synthesized Netlist 🔬
Yosys can generate a visual representation of the synthesized gate-level circuit.

* **Command**: The `show` command generates a `.dot` file and opens a viewer (like `xdot`) to display the schematic.
```tcl
yosys> show
```
#### Netlist Diagram
![Netlist Diagram](./assets/netlist%20diagram.png)
* **Analysis**: The generated netlist shows that Yosys successfully identified that the RTL code's logic perfectly matches the functionality of a dedicated 2-to-1 multiplexer cell. It therefore used a single, optimized `sky130_fd_sc_hd__mux2_1` standard cell, which is more efficient in terms of area and performance than building it from individual gates.

### Generating the Final Netlist 📜
The final step is to write the gate-level netlist to a Verilog file. After technology mapping, Yosys intelligently selects the most optimal cell from the library that matches the required logic.

* **Command**: The `write_verilog` command creates the output file. The `-noattr` flag provides a cleaner, more readable netlist.
```tcl
yosys> write_verilog -noattr good_mux_netlist.v
```
* **Resulting Netlist**: The generated netlist shows that Yosys successfully identified that the RTL code's logic perfectly matches the functionality of a dedicated 2-to-1 multiplexer cell. It therefore used a single, optimized `sky130_fd_sc_hd__mux2_1` standard cell, which is more efficient in terms of area and performance than building it from individual gates.

```verilog
module good_mux(i0, i1, sel, y);
  input i0;
  wire i0;
  input i1;
  wire i1;
  input sel;
  wire sel;
  output y;
  wire y;
  wire _0_;
  wire _1_;
  wire _2_;
  wire _3_;
  sky130_fd_sc_hd__mux2_1 _4_ (
    .A0(_0_),
    .A1(_1_),
    .S(_2_),
    .X(_3_)
  );
  assign _0_ = i0;
  assign _1_ = i1;
  assign _2_ = sel;
  assign y = _3_;
endmodule
```



