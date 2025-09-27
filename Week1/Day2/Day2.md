# My RISC-V Project Journal: Day 2 – Deconstructing the Digital DNA

## Today's Focus: From Theory to Gates

After setting up my digital workshop on Day 0, today was all about diving into the fundamental concepts of digital synthesis. My goal was to understand how a high-level hardware description in Verilog gets translated into a real, physical collection of standard cells. I explored the "DNA" of these cells in timing libraries, experimented with different synthesis strategies, and learned about the building blocks of sequential logic: flip-flops.

---

### Understanding the Blueprint: The Liberty Timing Library (`.lib`)

Before I can build anything, I need to understand my materials. In chip design, the "materials" are pre-designed logic gates called **standard cells**, and their master catalog is the **Liberty file (`.lib`)**.

-   **What is it?** It's a text file that acts as a detailed datasheet for every single cell available in a given technology (like SkyWater 130nm). It contains everything the synthesis tool needs to know to make intelligent decisions.
-   **What's inside?**
    -   **Global Info:** Units for time (`ns`), power (`mW`), voltage (`V`), etc.
    -   **Operating Conditions (PVT Corners):** How cells behave under different Process, Voltage, and Temperature conditions (e.g., worst-case slow, typical, best-case fast). The file we're using, `sky130_fd_sc_hd__tt_025C_1v80.lib`, represents the **typical** corner.
    -   **Cell-Specific Info:** For each gate, it defines its logical function, timing delays (how long it takes for a signal to pass through), power consumption, and physical area.

![A snippet from a .lib file showing its structure](images/day2-lib-file-example.png)

---

### A Look at the Standard Cell Catalog

I took a closer look at a few examples to understand the variety of cells available.

-   **Complex Cell (`a2111o_1`):** This single cell implements the logic `X = !((A1 & A2) | B1 | C1 | D1)`. Using complex cells like this allows the synthesizer to build logic more efficiently than using only basic AND/OR/NOT gates.
    ![Logic diagram for the a2111o_1 standard cell](images/day2-a2111o-gate.png)

-   **Drive Strength Variants (`and2_1`, `and2_2`, `and2_4`):** The library contains multiple versions of the same simple 2-input AND gate. The only difference is their **drive strength**.
    -   `_1`: **Low drive strength.** Smallest area and lowest power, but the slowest.
    -   `_4`: **High drive strength.** Largest area and highest power, but the fastest. It can drive signals across longer wires or to more subsequent gates.
    This demonstrates the fundamental **Power-Performance-Area (PPA)** trade-off in chip design. The synthesizer's job is to pick the right variant to meet the timing and power goals.
    ![Comparison of three different drive strength AND gates](images/day2-and2-variants.png)

---

### Synthesis Strategies: Hierarchical vs. Flat

Next, I experimented with how the synthesizer handles a design that is broken into smaller pieces (sub-modules).

#### Hierarchical Synthesis (The Default)

This approach preserves the modular structure of my Verilog code. When I synthesize my top module, `multiple_modules`, Yosys keeps `sub_module1` and `sub_module2` as distinct blocks. This is great for managing large, complex designs.

-   **My Process:**
    ```bash
    yosys
    read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
    read_verilog multiple_modules.v
    synth -top multiple_modules
    abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
    show multiple_modules
    ```

-   **Outputting the Hierarchical Netlist:**
    ```bash
    write_verilog -noattr multiple_modules_hierarchical.v
    ```

#### Flat Synthesis

In this approach, I tell the synthesizer to dissolve all the submodule boundaries and combine everything into one single, massive logic block. This can sometimes allow for better optimization on small designs but makes debugging large ones much harder.

-   **My Process:** (Continuing from the steps above)
    ```bash
    flatten
    show multiple_modules
    ```

-   **Outputting the Flat Netlist:**
    ```bash
    write_verilog -noattr multiple_modules_flat.v
    ```

---

### The Heartbeat of Logic: Flip-Flops

Combinational logic is prone to **glitches**—brief, unwanted signal changes caused by unequal path delays. **Flip-flops (flops)** are the solution. They are memory elements that only sample their input and change their output on a specific clock edge (e.g., the rising edge). This acts as a filter, ensuring that only stable, glitch-free data is passed between stages of a design.

I explored several common flop styles and synthesized each one to see how Yosys implements them using standard cells.

#### DFF with Asynchronous Set

The output `Q` is forced to `1` immediately when `async_set` is high, regardless of the clock.

-   **Synthesis Commands:**
    ```bash
    yosys
    read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
    read_verilog dff_async_set.v
    synth -top dff_async_set
    dfflibmap -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
    abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
    show
    ```

#### DFF with Asynchronous Reset

The output `Q` is forced to `0` immediately when `async_reset` is high. This is the most common type of reset.

-   **Synthesis Commands:**
    ```bash
    yosys
    read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
    read_verilog dff_asyncres.v
    synth -top dff_asyncres
    dfflibmap -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
    abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
    show
    ```

#### DFF with Synchronous Reset

The reset only occurs if `sync_reset` is high *at the moment of a rising clock edge*.

-   **Synthesis Commands:**
    ```bash
    yosys
    read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
    read_verilog dff_syncres.v
    synth -top dff_syncres
    dfflibmap -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
    abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
    show
    ```

#### DFF with Both Asynchronous and Synchronous Resets

This hybrid design includes an immediate asynchronous reset and a clock-dependent synchronous reset. The async reset always has priority.

-   **Synthesis Commands:**
    ```bash
    yosys
    read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
    read_verilog dff_asyncres_syncres.v
    synth -top dff_asyncres_syncres
    dfflibmap -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
    abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
    show
    ```

---

### Clever Optimizations in Synthesis

I finished the day by looking at how smart the synthesizer can be. Instead of implementing arithmetic with complex adder and multiplier circuits, Yosys can often find much simpler solutions.

#### Case 1: Synthesis of `mul2` (Multiply by 2)

My Verilog code had `y = a * 2;`. Instead of building a multiplier, Yosys recognized that multiplying by 2 is the same as a **left bit-shift (`a << 1`)**. In hardware, this requires no logic gates at all—it's just a re-wiring of the connections! The synthesized result was empty, proving the optimization.

-   **My Process:**
    ```bash
    yosys
    read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
    read_verilog mult_2.v
    synth -top mul2
    show
    write_verilog -noattr mult2_net.v
    ```

#### Case 2: Synthesis of `mult8` (Multiply by 8)

Similarly, multiplying by 8 is just a left bit-shift by 3 places (`a << 3`). Again, Yosys optimized this down to simple wiring, resulting in no standard cells being used. This is a powerful demonstration of how synthesis tools reduce area and improve performance automatically.

-   **My Process:**
    ```bash
    yosys
    read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
    read_verilog mult_8.v
    synth -top mult8
    show
    write_verilog -noattr mult8_net.v
    ```

    **👉 End of Day 2.** 