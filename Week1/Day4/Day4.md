# My RISC-V Project Journal: Day 4 – GLS, Blocking vs. Non-Blocking, and Simulation-Synthesis Mismatches 💡

Today was a huge step forward\! I moved past simple RTL coding and simulation to focus on a crucial validation step: **Gate Level Simulation (GLS)**. This is where you find out if your code, once turned into actual hardware gates by the synthesis tool, still works the way you intended. Along the way, I had to deeply explore why code behavior in a simulator might not match the final chip—a concept called **Simulation-Synthesis Mismatch**, often caused by the subtle difference between **blocking (`=`)** and **non-blocking (`<=`)** assignments in Verilog.

-----

## Today’s Focus: Bridging the RTL-to-Gate Gap 🌉

On Day 4, I moved beyond just writing and simulating RTL code. The spotlight was on Gate Level Simulation (GLS) and understanding tricky scenarios where the behavior seen in simulation does not match what happens after synthesis. Along the way, I also explored how blocking vs. non-blocking assignments in Verilog can make a big difference.

### Gate Level Simulation (GLS)

  * At the **RTL level**, the design is simulated directly as written in Verilog.
  * At the **GLS level**, the RTL is first synthesized into a gate-level netlist (using real standard cells), and this netlist is simulated.

This is important because:

  * RTL and netlist are expected to be logically equivalent, but mismatches sometimes happen.
  * GLS ensures that after synthesis, the design still behaves correctly.

GLS can be run in two modes:

  * **Functional mode (zero-delay)**: Only logical correctness is checked.
  * **Full timing mode**: If timing info is present, it also validates setup/hold and delay effects.

#### GLS flow using Icarus Verilog

To run GLS with `iverilog`, I provide:

  * the synthesized gate-level netlist,
  * the testbench, and
  * the standard cell Verilog models.

**Syntax:**

```bash
iverilog <path-to-gate-level-verilog-model(s)> <netlist_file.v> <tb_top.v>
```

-----

## Why Synthesis–Simulation Mismatch Happens 😱

Sometimes the behavior in RTL simulation does not match GLS. The main culprits include:

  * Incomplete sensitivity list in an `always` block.
  * **Blocking (`=`) vs. non-blocking (`<=`) assignments**:
      * **Blocking** executes sequentially inside the `always` block.
      * **Non-blocking** updates all signals together at the clock edge.
      * Synthesis generally assumes non-blocking-like hardware, so careless use of blocking can cause mismatches.
  * Non-standard Verilog coding styles.

I tested these cases using MUX designs and a custom “blocking caveat” example.

-----

## Synthesis and GLS Experiments: The Proof is in the Netlist\! 🔬

### 1\. Ternary Operator MUX

#### RTL Simulation

```bash
iverilog ternary_operator_mux.v tb_ternary_operator_mux.v
./a.out
gtkwave tb_ternary_operator_mux.vcd
```

#### Synthesis with Yosys

```bash
yosys
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog ternary_operator_mux.v
synth -top ternary_operator_mux
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
write_verilog -noattr ternary_operator_mux_net.v
```

#### GLS

```bash
iverilog ../my_lib/verilog_model/primitives.v ../my_lib/verilog_model/sky130_fd_sc_hd.v ternary_operator_mux_net.v tb_ternary_operator_mux.v
./a.out
gtkwave tb_ternary_operator_mux.vcd
```

### 2\. Bad MUX Design (incomplete sensitivity list)

Here I wrote a MUX with only `sel` in the sensitivity list. This created a mismatch: in simulation it behaved like a latch, but after synthesis it mapped to a proper combinational MUX.

#### RTL Simulation

```bash
iverilog bad_mux.v tb_bad_mux.v
./a.out
gtkwave tb_bad_mux.vcd
```

#### Synthesis

```bash
yosys
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog bad_mux.v
synth -top bad_mux
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
write_verilog -noattr bad_mux_net.v
```

#### GLS

```bash
iverilog ../my_lib/verilog_model/primitives.v ../my_lib/verilog_model/sky130_fd_sc_hd.v bad_mux_net.v tb_bad_mux.v
./a.out
gtkwave tb_bad_mux.vcd
```

This showed a clear simulation-synthesis mismatch.

### 3\. Blocking Caveat Design

This design demonstrated how using blocking assignments in sequential logic can cause unexpected behavior when compared to synthesized hardware.

#### RTL Simulation

```bash
iverilog blocking_caveat.v tb_blocking_caveat.v
./a.out
gtkwave tb_blocking_caveat.vcd
```

#### Synthesis

```bash
yosys
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog blocking_caveat.v
synth -top blocking_caveat
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
write_verilog -noattr blocking_caveat_net.v
```

#### GLS

```bash
iverilog ../my_lib/verilog_model/primitives.v ../my_lib/verilog_model/sky130_fd_sc_hd.v blocking_caveat_net.v tb_blocking_caveat.v
./a.out
gtkwave tb_blocking_caveat.vcd
```

Here again, pre-synthesis and post-synthesis results did not perfectly align—teaching me the importance of careful coding style.

-----

## 🔑 Key Learnings from Day 4

  * **GLS** validates designs after synthesis and reveals mismatches that plain RTL simulation might hide.
  * **Incomplete sensitivity lists** can create latch-like behavior in simulation but synthesize as pure combinational logic.
  * **Blocking assignments** can cause simulation vs. hardware mismatches if misused.
  * Writing clean, standard Verilog (**`always @(*)`** and **non-blocking** for sequential logic) avoids many pitfalls.

👉 **End of Day 4.**