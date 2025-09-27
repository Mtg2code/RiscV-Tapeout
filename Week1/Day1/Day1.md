# My RISC-V Project Journal: Day 1 – From Concept to Code

## Today's Focus: RTL Design, Simulation, and First Synthesis

With my EDA environment fully set up, today was my first real dive into the digital design workflow. The mission was to learn the fundamentals: how to describe a piece of hardware using the Verilog language, how to thoroughly test that design to make sure it's logically correct, and finally, how to take that abstract code and synthesize it into a concrete collection of digital gates using Yosys and the Sky130 technology library.

---

### Step 1: Functional Verification with Simulation

Before a chip design can be considered for manufacturing, it must be rigorously **validated** to ensure it behaves exactly as intended. This process is called simulation, and it's where I spent the first part of my day.

-   **My Simulator: Icarus Verilog (`iverilog`)**
    This tool is a compiler that takes my Verilog design and a test file (called a testbench) and simulates their interaction. Its main output is a `.vcd` (Value Change Dump) file, which is a log of how every signal in my design changed over time.

-   **My Visualizer: GTKWave**
    Reading a `.vcd` file directly is nearly impossible. GTKWave reads this file and displays the signals as graphical waveforms, which is essential for debugging and truly understanding the circuit's behavior.

The core idea is simple: I write a **design** that describes what the circuit should do, and a **testbench** that acts like a virtual lab technician, applying inputs and checking the outputs.


---

### My First Design: A 2:1 Multiplexer (MUX)

To put theory into practice, I designed a simple 2:1 multiplexer. It's like a digital switch: based on the value of a selector line (`sel`), it chooses one of two inputs (`i0` or `i1`) to pass to the output (`y`).

-   **The Design (`good_mux.v`):**
     The MUX uses behavioral Verilog inside an `always @(*)` block. This tells the synthesizer to create combinational logic that updates the output `y` whenever any of the inputs change.

-   **The Testbench (`tb_good_mux.v`):**
    This file instantiates the MUX design. Its only job is to generate a sequence of input signals to test all possible conditions of the MUX. I used `$dumpfile` and `$dumpvars` to tell the simulator to record the signal activity for viewing in GTKWave.

-   **Simulation Results:**
    After compiling and running the simulation, I opened the `.vcd` file in GTKWave. The waveforms clearly showed that the output `y` correctly followed `i0` when `sel` was low, and followed `i1` when `sel` was high.

    ![GTKWave screenshot showing the successful MUX simulation waveforms](images/day1-mux-waveform.png)

---

### Step 2: Synthesis with Yosys

With the MUX functionally verified, it was time for **synthesis**. This is the process of converting my abstract, behavioral Verilog code into a **gate-level netlist**—a structural description of the circuit built using only the standard cells available in the Sky130 library.

I used **Yosys**, the open-source synthesis tool I installed yesterday, to perform this translation.

#### My Yosys Synthesis Workflow

I followed a step-by-step command sequence inside the Yosys environment:

1.  **Launch Yosys:**
    ```bash
    yosys
    ```

2.  **Read the Liberty File:** This is the most critical step. I'm telling Yosys about the characteristics (timing, power, function) of all the standard cells it's allowed to use from the Sky130 library.
    ```bash
    read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
    ```

3.  **Read My Verilog Design:**
    ```bash
    read_verilog good_mux.v
    ```

4.  **Synthesize the Design:** The `synth` command performs the main logic synthesis, converting the Verilog into a generic gate representation.
    ```bash
    synth -top good_mux
    ```

5.  **Map to Technology (ABC):** This step takes the generic gates and maps them to the specific, optimized standard cells from the Sky130 Liberty file.
    ```bash
    abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
    ```

6.  **View the Resulting Schematic:** The `show` command generates a visual diagram of the synthesized gate-level netlist. For my MUX, it showed a clear implementation using basic logic gates.
    ```bash
    show
    ```
    ![The synthesized schematic of the MUX as shown by Yosys](images/day1-yosys-schematic.png)

7.  **Write the Final Netlist:** Finally, I saved the result as a new Verilog file. This file no longer contains behavioral code; instead, it's a structural list of standard cell instances and the wires connecting them.
    ```bash
    write_verilog -noattr good_mux_netlist.v
    ```
    ![The final gate-level netlist file for the MUX](images/day1-mux-netlist.png)

---

### 🔑 Key Learnings from Day 1

-   Verilog is used to describe the *behavior* of a digital circuit at a high level.
-   A **testbench** is absolutely essential for verifying that the design's logic is correct before moving forward.
-   The combination of `iverilog` and `gtkwave` provides a powerful, visual environment for simulation and debugging.
-   **Yosys** is the bridge between the abstract RTL design and a concrete gate-level implementation using a specific technology library like Sky130.
-   I successfully completed my first full **RTL → Simulation → Synthesis** flow.

**👉 End of Day 1.** 