# Week 3 Task – STA Fundamentals & Timing Graphs with OpenSTA

## Objective

Gain basic STA knowledge and generate timing graphs using OpenSTA.  
Interpret timing paths, setup/hold analysis, and report slack.

---

## Part 3 – Generate Timing Graphs with OpenSTA

> Use OpenSTA for timing analysis of your synthesized BabySoC design.

### Reference Materials

- [OpenSTA GitHub](https://github.com/The-OpenROAD-Project/OpenSTA)
- [Example Script – Day 19](https://github.com/arunkpv/vsd-hdp/blob/main/docs/Day_19.md)
- [OpenSTA Documentation (PDF)](https://github.com/The-OpenROAD-Project/OpenSTA/blob/master/doc/OpenSTA.pdf)

---

## Step-by-Step OpenSTA Flow

### 1. Prepare the OpenSTA Input Script

Adapted from [Reference Day 19 Script](https://github.com/arunkpv/vsd-hdp/blob/main/docs/Day_19.md):

```tcl
# opensta_input.tcl

# Load Liberty cell library
read_liberty /home/VSDBabySoC/src/lib/sky130_fd_sc_hd__tt_025C_1v80.lib

# Load synthesized gate-level netlist
read_verilog /home/VSDBabySoC/output/post_synth_sim/vsdbabysoc.synth.v

# Load SDC constraints file (edit filename if needed)
read_sdc /home/VSDBabySoC/constraints/babysoc_constraints.sdc

# Set up clock, input/output delays, etc. (Example)
# create_clock -name clk -period 10.0 [get_ports clk]
# set_input_delay 2.0 [get_ports reset]
# set_output_delay 2.0 [get_ports done]

# Run timing analysis
report_tns
report_wns
report_checks -path_delay min
report_checks -path_delay max
report_timing -from [get_ports clk] -to [get_ports reset]

# To view/graph timing paths interactively (see OpenSTA documentation)
```

---

### 2. Run OpenSTA

```bash
sta opensta_input.tcl
```

---

### 3. Capture Timing Reports & Graphs

- Save the terminal output or use `redirect` in the TCL script to save reports:
  ```tcl
  redirect /home/VSDBabySoC/output/sta/report_timing.txt { report_timing }
  ```

- Take screenshots of timing graphs/reports with your userid and timestamp visible.

---

## Deliverables

- OpenSTA input scripts (`opensta_input.tcl`)
- Timing reports and graph screenshots (with userid and timestamp)
- **Observations:**  
  - What is the critical path?
  - What does the reported slack indicate about timing margin?

---

## Example Observations

- **Critical Path:** Path with the least slack; determines max clock speed.
- **Slack:** Positive slack means timing is met; negative slack means violation.

---

## References

- [OpenSTA GitHub](https://github.com/The-OpenROAD-Project/OpenSTA)
- [Reference Script – Day 19](https://github.com/arunkpv/vsd-hdp/blob/main/docs/Day_19.md)
- [OpenSTA Documentation (PDF)](https://github.com/The-OpenROAD-Project/OpenSTA/blob/master/doc/OpenSTA.pdf)

---

**For questions or improvements, please open an issue or pull request.**