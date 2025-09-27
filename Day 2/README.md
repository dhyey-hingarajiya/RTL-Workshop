# Timing Libs, Hierarchical vs Flat Synthesis and Efficient Flop Coding Styles

## Lab 4: The Synthesis Toolbox - Standard Cell Libraries (.lib)

Before we can build anything, we need to know what parts are available. The `.lib` file is the detailed catalog of every standard cell "Lego brick" we can use for synthesis.

#### Decoding PVT Corners

The filename sky130_fd_sc_hd__tt_025C_1v80.lib tells us the exact Process, Voltage, and Temperature (PVT) corner this library is characterized for:
* `tt`: A Typical Typical process.
* `025C`: A Temperature of 25° Celsius.
* `1v80`: A Voltage of 1.80 Volts.

Our designs must be robust enough to work across all PVT corners, ensuring they function reliably everywhere from a hot car engine to a cold satellite.

#### Cell "Flavors" and the PPA Trade-Off

For any given logic gate, the library offers multiple "flavors" with different drive strengths (e.g., `and2_0`, `and2_2`, `and2_4`). This is where we encounter the fundamental Power-Performance-Area (PPA) trade-off.

The screenshot below clearly illustrates this. As we move from a weaker cell (`and2_0`) to a stronger one (`and2_4`), we see the consequences:

![DIffrence in cell flavours](./assets/cell%20flavours.png)


### Inside the Library: The Anatomy of a `.lib` File

Opening a `.lib` file is like popping the hood to see how the engine works. It contains all the nitty-gritty details the synthesis tool needs.

#### The Header: The Rulebook
The top of the file sets the global rules:
* **`Units`**: Defines the playground's physics—time is in nanoseconds (`ns`), power is in nanowatts (`nW`), capacitance is in picofarads (`pF`), etc.
* **`Operating Conditions`**: Confirms the PVT corner we decoded from the filename.

#### Cell Definitions: The Building Blocks
The rest of the file is a massive list of every available cell. Each `cell` block is a datasheet containing:
* **`area`**: How much silicon real estate the cell takes up.
* **`leakage_power`**: How much power the cell "leaks" when it's just sitting there.
* **Pin Characteristics**: Details about each input and output pin, like its `capacitance`.
* **Timing and Power Tables**: The most important part! These are lookup tables that predict the cell's **delay (performance)** and **dynamic power** based on how fast the input signal is changing and how much load it has to drive.

## The Main Event: Cell "Flavors" & the PPA Trade-Off

Here's the coolest part. For a single logic function, like a 2-input AND gate, the library doesn't give us just one option. It gives us several "flavors" with different **drive strengths** (e.g., `and2_0`, `and2_2`, `and2_4`).

Why? Because every design decision is a trade-off between **Power, Performance (Speed), and Area (Cost)**. This is the **PPA** triangle, the golden rule of chip design.

Our screenshot provides the perfect evidence. Let's analyze it:

This image compares our three AND gate flavors. The number at the end (`_0`, `_2`, `_4`) tells us its strength. A stronger cell uses wider transistors, leading to a classic engineering trade-off:

* **Area (Cost)**: As drive strength increases, the cell gets bigger. We can see the `area` value climbing from **6.256** to **7.507** to **8.758**. More performance costs more space.

* **Power**: Bigger transistors leak more power. For the same input state (`!A&B`), the `leakage_power` also climbs from **~0.0019 nW** to **~0.0039 nW** to **~0.0045 nW**.

* **Performance (Speed)**: So why pay the price in area and power? For speed! The `and2_4` gate is the **fastest** of the three. It can push signals through to the next stage of logic much more quickly, which is critical for meeting timing requirements in a high-speed chip.

The synthesis tool reads this entire library(`.lib` file) and picks the perfect cell flavor for every single part of our design to get the best possible PPA.

---

## Lab 5 - Hierarchical vs. Flat Synthesis**

This lab explores different logic synthesis methodologies using the **Yosys Open SYnthesis Suite**. The primary goal is to understand the distinction between hierarchical and flat synthesis, observe their effects on the final netlist, and learn how to control the synthesis process. We will also cover the "bottom-up" approach of synthesizing individual sub-modules and discuss its practical applications in managing design complexity and improving efficiency.

### **Design Overview**

The experiments in this lab use a simple Verilog design file, `multiple_modules.v`, which is structured with a clear hierarchy.

* **`sub_module1`**: A basic 2-input AND gate.
* **`sub_module2`**: A basic 2-input OR gate.
* **`multiple_modules` (Top Module)**: This module instantiates `sub_module1` (as `u1`) and `sub_module2` (as `u2`). The output of instance `u1` is connected to one of the inputs of instance `u2`.

### **Hierarchical Synthesis**

Hierarchical synthesis is the default approach where the synthesis tool preserves the boundaries and structure of the original HDL code. It synthesizes each module as a distinct entity.

The tool processes the design from the top down, maintaining the integrity of each sub-module. This is useful for managing large designs, as it keeps the structure organized and understandable.

#### **Procedure & Observations**
1.  **Synthesis**: The design is synthesized using the top module as the target.
    ```bash
    yosys> synth -top multiple_modules
    ```
2.  **Visualization (`show`)**: When visualized, the netlist appears as a block diagram. We see blocks for `u1` and `u2` connected, but the internal logic (the primitive AND/OR gates) is encapsulated within those blocks. The hierarchy is clearly visible.

    ![Hierarchical synthesis view](./assets/hier%20synth.png)

3.  **Output Netlist (`write_verilog ..._hier.v`)**: The generated Verilog netlist file contains separate `module` definitions for `multiple_modules`, `sub_module1`, and `sub_module2`. The top module instantiates the sub-modules, perfectly mirroring the original design structure.

### **Flat Synthesis**

Flat synthesis dissolves the hierarchical boundaries, merging all logic into a single, top-level module. This gives the tool a global view of the entire design, which can sometimes lead to better cross-boundary optimizations.

The `flatten` command instructs Yosys to remove all sub-module instantiations and pull their internal logic up into the top-level module.

#### **Procedure & Observations**
1.  **Flatten Command**: After the initial synthesis, the `flatten` command is executed.
    ```bash
    yosys> flatten
    ```
2.  **Visualization (`show`)**: After flattening, the visualization no longer shows the `u1` and `u2` blocks. Instead, it displays a single, unified netlist of primitive standard cells (inverters, AND gates, NAND gates, etc.) that are directly interconnected.

    ![Flat synthesis view](./assets/flat%20synth.png)

3.  **Output Netlist (`write_verilog ..._flat.v`)**: The generated netlist contains only one `module` definition: `multiple_modules`. The logic from the original `sub_module1` and `sub_module2` is instantiated directly within this single module. All traces of the original hierarchy are gone.

### **Sub-Module Synthesis (Bottom-Up Approach)**

This technique involves synthesizing a specific sub-module as if it were the top-level design. This is a powerful feature for managing complex projects.

#### **Concept**
By changing the target of the `synth -top` command, we can direct the tool to focus its synthesis and optimization efforts on a single, isolated part of the design hierarchy.

#### **Procedure & Visualization**
To synthesize only `sub_module1`, the following command is used. The resulting visualization shows only the netlist for that specific module.

```bash
yosys> synth -top sub_module1
```
![Submodule1 synthesis view](./assets/sub%20module%201.png)
#### Rationale & Use Cases

This bottom-up approach is highly valuable in two primary scenarios:

1. **Design Reuse & Efficiency:** If a design contains many instances of the same complex module (e.g., a CPU core, a DSP block, a multiplier), you can synthesize that module just once. The resulting optimized netlist can then be instantiated multiple times in the top-level design. This significantly reduces the total synthesis time and effort.

2. **Divide and Conquer for Massive Designs:** For extremely large and complex SoCs, a flat synthesis can be computationally prohibitive and may yield suboptimal results. By breaking the design into smaller, manageable sub-modules and synthesizing each one individually, designers can achieve better optimization for each block. These pre-synthesized blocks are then integrated at the top level to form the complete design.

---

## Flip-Flop Coding Styles
Flip-flops are fundamental sequential elements in digital design, used to store binary data. Below are efficient coding styles for different reset/set behaviors.
#### Asynchronous Reset D Flip-Flop
```verilog
module dff_asyncres (input clk, input async_reset, input d, output reg q);
  always @ (posedge clk, posedge async_reset)
    if (async_reset)
      q <= 1'b0;
    else
      q <= d;
endmodule
```

* **Asynchronous reset**: Overrides clock, setting q to 0 immediately.
* **Edge-triggered**: Captures d on rising clock edge if reset is low.

#### Asynchronous Set D Flip-Flop
```verilog
module dff_async_set (input clk, input async_set, input d, output reg q);
  always @ (posedge clk, posedge async_set)
    if (async_set)
      q <= 1'b1;
    else
      q <= d;
endmodule
```
* **Asynchronous set**: Overrides clock, setting q to 1 immediately.

#### Synchronous Reset D Flip-Flop
```verilog
module dff_syncres (input clk, input async_reset, input sync_reset, input d, output reg q);
  always @ (posedge clk)
    if (sync_reset)
      q <= 1'b0;
    else
      q <= d;
endmodule
```
* **Synchronous reset**: Takes effect only on the clock edge.

### Simulation and Synthesis Workflow
#### Icarus Verilog Simulation
```bash
iverilog dff_asyncres.v tb_dff_asyncres.v
./a.out
gtkwave tb_dff_asyncres.vcd
```
![Waveform for AsyncRes](./assets/Waveform%20AsyncRes.png)

#### Yosys Synthesis

##### Asynchronous Reset D Flip-Flop
```bash
yosys
yosys> read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
yosys> read_verilog dff_asyncres.v
yosys> synth -top dff_asyncres
yosys> dfflibmap -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
yosys> abc -liberty ../lib/sky130/file/sky130_fd_sc_hd__tt_025C_1v80.lib
yosys> show
```
![AsyncRes Synthesis](./assets/flop%20asyncres.png)

##### Asynchronous Set D Flip-Flop
```bash
yosys
yosys> read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
yosys> read_verilog dff_async_set.v
yosys> synth -top dff_async_set
yosys> dfflibmap -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
yosys> abc -liberty ../lib/sky130/file/sky130_fd_sc_hd__tt_025C_1v80.lib
yosys> show
```
![AsyncSet Synthesis](./assets/flop%20asyncset.png)

##### Synchronous Reset D Flip-Flop
```bash
yosys
yosys> read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
yosys> read_verilog dff_syncres.v
yosys> synth -top dff_syncres
yosys> dfflibmap -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
yosys> abc -liberty ../lib/sky130/file/sky130_fd_sc_hd__tt_025C_1v80.lib
yosys> show
```
![AsyncSet Synthesis](./assets/flop%20syncres.png)

