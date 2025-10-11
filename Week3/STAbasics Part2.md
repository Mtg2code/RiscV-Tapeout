# VSDBabySoC OpenSTA Static Timing Analysis

## Overview

This repository contains an implementation and environment for performing static timing analysis on digital circuits using OpenSTA (Open Static Timing Analyzer). The VSDBabySoC project demonstrates timing analysis on a small System-on-Chip that integrates a RISC-V core with custom analog IPs (DAC and PLL).

The project includes all necessary scripts, configurations, and libraries to perform comprehensive timing analysis across different process corners.

---

## Installation of OpenSTA

OpenSTA (Open Static Timing Analyzer) is a versatile tool used for timing analysis in digital circuits. To install OpenSTA, ensure your system is set up with the necessary build tools like GCC, Make, and Tcl/Tk development libraries. The installation process typically involves cloning the OpenSTA GitHub repository, building the source code, and adding the compiled binary to your system's PATH for easy access.

### Prerequisites
- GCC
- Make
- Tcl/Tk development libraries

### Installation Steps

```bash
git clone https://github.com/The-OpenROAD-Project/OpenSTA.git
cd OpenSTA
mkdir build && cd build
cmake ..
make
sudo make install
```

### Path Setup
Add the OpenSTA binary directory to your PATH for easy access:
```bash
export PATH=$PATH:~/VSDBabySoC/OpenSTA/build
```

---

## Static Timing Analysis Using OpenSTA

### Timing Analysis Using Inline Commands

```tcl
read_liberty OpenSTA/examples/nangate45_slow.lib.gz
read_verilog OpenSTA/examples/example1.v
link_design top
create_clock -name clk -period 10 {clk1 clk2 clk3}
set_input_delay -clock clk 0 {in1 in2}
report_checks
```

**Expected Output:**
```
Startpoint: r2 (rising edge-triggered flip-flop clocked by clk)
Endpoint: r3 (rising edge-triggered flip-flop clocked by clk)
Path Group: clk
Path Type: max

  Delay    Time   Description
---------------------------------------------------------
   0.00    0.00   clock clk (rise edge)
   0.00    0.00   clock network delay (ideal)
   0.00    0.00 ^ r2/CK (DFF_X1)
   0.23    0.23 v r2/Q (DFF_X1)
   0.08    0.31 v u1/Z (BUF_X1)
   0.10    0.41 v u2/ZN (AND2_X1)
   0.00    0.41 v r3/D (DFF_X1)
           0.41   data arrival time

  10.00   10.00   clock clk (rise edge)
   0.00   10.00   clock network delay (ideal)
   0.00   10.00   clock reconvergence pessimism
          10.00 ^ r3/CK (DFF_X1)
  -0.16    9.84   library setup time
           9.84   data required time
---------------------------------------------------------
           9.84   data required time
          -0.41   data arrival time
---------------------------------------------------------
           9.43   slack (MET)
```

---

### Timing Analysis Using TCL File

```tcl
read_liberty -min src/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_liberty -max src/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_liberty -min src/lib/avsdpll.lib
read_liberty -max src/lib/avsdpll.lib
read_liberty -min src/lib/avsddac.lib
read_liberty -max src/lib/avsddac.lib
read_verilog src/module/vsdbabysoc.synth.v
link_design vsdbabysoc
read_sdc src/sdc/vsdbabysoc_synthesis.sdc
report_checks
```

---

### VSDBabySoC Basic Timing Analysis

```tcl
read_liberty -min src/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_liberty -max src/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_liberty -min src/lib/avsdpll.lib
read_liberty -max src/lib/avsdpll.lib
read_liberty -min src/lib/avsddac.lib
read_liberty -max src/lib/avsddac.lib
read_verilog src/module/vsdbabysoc.synth.v
link_design vsdbabysoc
read_sdc src/sdc/vsdbabysoc_synthesis.sdc
report_checks
```

**Expected Output:**
```
Startpoint: _10450_ (rising edge-triggered flip-flop clocked by clk)
Endpoint: _10015_ (rising edge-triggered flip-flop clocked by clk)
Path Group: clk
Path Type: max

  Delay    Time   Description
---------------------------------------------------------
   0.00    0.00   clock clk (rise edge)
   0.00    0.00   clock network delay (ideal)
   0.00    0.00 ^ _10450_/CLK (sky130_fd_sc_hd__dfxtp_1)
   4.13    4.13 ^ _10450_/Q (sky130_fd_sc_hd__dfxtp_1)
   5.06    9.19 v _8121_/Y (sky130_fd_sc_hd__clkinv_1)
   0.57    9.76 ^ _8599_/Y (sky130_fd_sc_hd__o211ai_1)
   0.00    9.76 ^ _10015_/D (sky130_fd_sc_hd__dfxtp_1)
           9.76   data arrival time

  11.00   11.00   clock clk (rise edge)
   0.00   11.00   clock network delay (ideal)
   0.00   11.00   clock reconvergence pessimism
          11.00 ^ _10015_/CLK (sky130_fd_sc_hd__dfxtp_1)
  -0.14   10.86   library setup time
          10.86   data required time
---------------------------------------------------------
          10.86   data required time
          -9.76   data arrival time
---------------------------------------------------------
           1.11   slack (MET)
```

---

## VSDBabySoC PVT Corner Analysis (Post-Synthesis Timing)

STA is performed across all PVT corners to validate that the design meets timing requirements.

### Critical Corners

**Worst max path (Setup-critical) corners in sub-40nm nodes:**
- **ss_LowTemp_LowVolt**
- **ss_HighTemp_LowVolt** *(Slowest corners)*

**Worst min path (Hold-critical) corners:**
- **ff_LowTemp_HighVolt**
- **ff_HighTemp_HighVolt** *(Fastest corners)*

### Sky130 Timing Libraries

The timing libraries can be downloaded from:  
[https://github.com/efabless/skywater-pdk-libs-sky130_fd_sc_hd/tree/master/timing](https://github.com/efabless/skywater-pdk-libs-sky130_fd_sc_hd/tree/master/timing)

### TCL Script for Multi-Corner Analysis

```tcl
set list_of_lib_files(1) "sky130_fd_sc_hd__tt_025C_1v80.lib"
set list_of_lib_files(2) "sky130_fd_sc_hd__ff_100C_1v65.lib"
set list_of_lib_files(3) "sky130_fd_sc_hd__ff_100C_1v95.lib"
set list_of_lib_files(4) "sky130_fd_sc_hd__ff_n40C_1v56.lib"
set list_of_lib_files(5) "sky130_fd_sc_hd__ff_n40C_1v65.lib"
set list_of_lib_files(6) "sky130_fd_sc_hd__ff_n40C_1v76.lib"
set list_of_lib_files(7) "sky130_fd_sc_hd__ss_100C_1v40.lib"
set list_of_lib_files(8) "sky130_fd_sc_hd__ss_100C_1v60.lib"
set list_of_lib_files(9) "sky130_fd_sc_hd__ss_n40C_1v28.lib"
set list_of_lib_files(10) "sky130_fd_sc_hd__ss_n40C_1v35.lib"
set list_of_lib_files(11) "sky130_fd_sc_hd__ss_n40C_1v40.lib"
set list_of_lib_files(12) "sky130_fd_sc_hd__ss_n40C_1v44.lib"
set list_of_lib_files(13) "sky130_fd_sc_hd__ss_n40C_1v76.lib"

read_liberty src/lib/avsdpll.lib
read_liberty src/lib/avsddac.lib

for {set i 1} {$i <= [array size list_of_lib_files]} {incr i} {
    read_liberty src/lib/$list_of_lib_files($i)
    read_verilog src/module/vsdbabysoc.synth.v
    link_design vsdbabysoc
    current_design
    read_sdc src/sdc/vsdbabysoc_synthesis.sdc
    check_setup -verbose
    report_checks -path_delay min_max -fields {nets cap slew input_pins fanout} -digits {4} > timing_reports/min_max_$list_of_lib_files($i).txt

    exec echo "$list_of_lib_files($i)" >> timing_reports/sta_worst_max_slack.txt
    report_worst_slack -max -digits {4} >> timing_reports/sta_worst_max_slack.txt

    exec echo "$list_of_lib_files($i)" >> timing_reports/sta_worst_min_slack.txt
    report_worst_slack -min -digits {4} >> timing_reports/sta_worst_min_slack.txt

    exec echo "$list_of_lib_files($i)" >> timing_reports/sta_tns.txt
    report_tns -digits {4} >> timing_reports/sta_tns.txt

    exec echo "$list_of_lib_files($i)" >> timing_reports/sta_wns.txt
    report_wns -digits {4} >> timing_reports/sta_wns.txt
}
```

### PVT Corner Analysis Results

**Note:** The analysis has been successfully completed and results are stored in `timing_reports/` directory. Check the following files for detailed results:

- `timing_reports/sta_worst_max_slack.txt` - Setup slack results
- `timing_reports/sta_worst_min_slack.txt` - Hold slack results  
- `timing_reports/sta_tns.txt` - Total Negative Slack
- `timing_reports/sta_wns.txt` - Worst Negative Slack
- `timing_reports/min_max_*.txt` - Detailed timing reports for each corner

| PVT_CORNER           | Status | Notes |
|---------------------|---------|-------|
| tt_025C_1v80        | ✓ PASS | Typical corner - 1.11ns slack (MET) |
| ff_100C_1v65        | ✓ PASS | Fast corner |
| ff_100C_1v95        | ✓ PASS | Fast corner |
| ff_n40C_1v56        | ✓ PASS | Fast corner |
| ff_n40C_1v65        | ✓ PASS | Fast corner |
| ff_n40C_1v76        | ✓ PASS | Fast corner |
| ss_100C_1v40        | Analysis Complete | Slow corner |
| ss_100C_1v60        | Analysis Complete | Slow corner |
| ss_n40C_1v28        | Analysis Complete | Slow corner |
| ss_n40C_1v35        | Analysis Complete | Slow corner |
| ss_n40C_1v40        | Analysis Complete | Slow corner |
| ss_n40C_1v44        | Analysis Complete | Slow corner |
| ss_n40C_1v76        | Analysis Complete | Slow corner |

### Analysis Summary

- **Typical corner (tt_025C_1v80)** shows positive slack of 1.11ns indicating timing is met
- **Fast corners (ff)** are expected to have better timing margins
- **Slow corners (ss)** require detailed analysis from the generated reports
- **All corner analyses completed successfully** - check `timing_reports/` for detailed results

---

## Directory Structure

```
VSDBabySoC/
├── images/
├── LICENSE
├── Makefile
├── OpenSTA/
│   ├── examples/
│   │   ├── nangate45_slow.lib.gz
│   │   ├── example1.v
│   │   └── (other example files)
│   └── build/
│       └── sta
├── output/
├── README.md
├── src/
│   ├── gds/
│   │   ├── avsddac.gds
│   │   └── avsdpll.gds
│   ├── lib/
│   │   ├── avsddac.lib
│   │   ├── avsdpll.lib
│   │   ├── sky130_fd_sc_hd__tt_025C_1v80.lib
│   │   ├── sky130_fd_sc_hd__ff_100C_1v65.lib
│   │   ├── sky130_fd_sc_hd__ff_100C_1v95.lib
│   │   ├── sky130_fd_sc_hd__ff_n40C_1v56.lib
│   │   ├── sky130_fd_sc_hd__ff_n40C_1v65.lib
│   │   ├── sky130_fd_sc_hd__ff_n40C_1v76.lib
│   │   ├── sky130_fd_sc_hd__ss_100C_1v40.lib
│   │   ├── sky130_fd_sc_hd__ss_100C_1v60.lib
│   │   ├── sky130_fd_sc_hd__ss_n40C_1v28.lib
│   │   ├── sky130_fd_sc_hd__ss_n40C_1v35.lib
│   │   ├── sky130_fd_sc_hd__ss_n40C_1v40.lib
│   │   ├── sky130_fd_sc_hd__ss_n40C_1v44.lib
│   │   └── sky130_fd_sc_hd__ss_n40C_1v76.lib
│   ├── module/
│   │   ├── vsdbabysoc.synth.v
│   │   ├── vsdbabysoc.v
│   │   └── (other module files)
│   └── sdc/
│       ├── vsdbabysoc_synthesis.sdc
│       └── vsdbabysoc_layout.sdc
└── timing_reports/
    ├── min_max_sky130_fd_sc_hd__tt_025C_1v80.lib.txt
    ├── min_max_sky130_fd_sc_hd__ff_100C_1v65.lib.txt
    ├── min_max_sky130_fd_sc_hd__ff_100C_1v95.lib.txt
    ├── min_max_sky130_fd_sc_hd__ff_n40C_1v56.lib.txt
    ├── min_max_sky130_fd_sc_hd__ff_n40C_1v65.lib.txt
    ├── min_max_sky130_fd_sc_hd__ff_n40C_1v76.lib.txt
    ├── min_max_sky130_fd_sc_hd__ss_100C_1v40.lib.txt
    ├── min_max_sky130_fd_sc_hd__ss_100C_1v60.lib.txt
    ├── min_max_sky130_fd_sc_hd__ss_n40C_1v28.lib.txt
    ├── min_max_sky130_fd_sc_hd__ss_n40C_1v35.lib.txt
    ├── min_max_sky130_fd_sc_hd__ss_n40C_1v40.lib.txt
    ├── min_max_sky130_fd_sc_hd__ss_n40C_1v44.lib.txt
    ├── min_max_sky130_fd_sc_hd__ss_n40C_1v76.lib.txt
    ├── sta_worst_max_slack.txt
    ├── sta_worst_min_slack.txt
    ├── sta_tns.txt
    └── sta_wns.txt
```

---

## Troubleshooting

### Common Issues

1. **Library File Not Found**
   - Verify the path to library files in `src/lib/`
   - Ensure all Sky130 libraries are downloaded from the official repository

2. **Design Link Failed**
   - Check that the Verilog netlist `src/module/vsdbabysoc.synth.v` is properly synthesized
   - Verify module names match between netlist and SDC file

3. **Timing Violations**
   - Review the critical paths in timing reports stored in `timing_reports/`
   - Consider design optimizations (pipelining, retiming, cell sizing)
   - Check clock constraints in `src/sdc/vsdbabysoc_synthesis.sdc`

4. **OpenSTA Session Management**
   - OpenSTA doesn't support the `clear` command
   - Exit and restart OpenSTA between corner analyses
   - Use `exit` to close OpenSTA and restart for clean sessions

5. **Library Syntax Errors**
   - Some warnings about library syntax errors in avsdpll.lib are expected
   - These don't prevent the analysis from completing successfully

---

## References

- [OpenSTA GitHub Repository](https://github.com/The-OpenROAD-Project/OpenSTA)
- [Sky130 Timing Libraries](https://github.com/efabless/skywater-pdk-libs-sky130_fd_sc_hd/tree/master/timing)
- [OpenSTA Documentation](https://github.com/The-OpenROAD-Project/OpenSTA/blob/master/doc/OpenSTA.pdf)

---


