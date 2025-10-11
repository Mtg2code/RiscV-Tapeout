# VSDBabySoC OpenSTA Static Timing Analysis

## Overview

This repository contains an implementation and environment for performing static timing analysis on digital circuits using OpenSTA (Open Static Timing Analyzer). The VSDBabySoC project demonstrates timing analysis on a small System-on-Chip that integrates a RISC-V core with custom analog IPs (DAC and PLL).

The project includes all necessary scripts, configurations, and libraries to perform comprehensive timing analysis across different process corners.

## Installation of OpenSTA

To install OpenSTA on your system:

1. **Prerequisites:**  
   - GCC, Make, and Tcl/Tk development libraries

2. **Clone and Build:**
   ```bash
   git clone https://github.com/The-OpenROAD-Project/OpenSTA.git
   cd OpenSTA
   mkdir build && cd build
   cmake ..
   make
   sudo make install
   ```

3. **Path Setup:**  
   - Add the OpenSTA binary directory to your PATH:
   ```bash
   export PATH=$PATH:/path/to/OpenSTA/build
   ```

## Running Static Timing Analysis

### Basic OpenSTA Commands

```tcl
# Start OpenSTA
sta

# Load design
read_liberty -min src/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_liberty -max src/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog src/module/vsdbabysoc.synth.v
link_design vsdbabysoc
read_sdc src/sdc/vsdbabysoc_synthesis.sdc

# Run timing analysis
report_checks
```

### VSDBabySoC Analysis

For analyzing the VSDBabySoC design:

```tcl
# Load libraries
read_liberty -min src/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_liberty -max src/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_liberty -min src/lib/avsdpll.lib
read_liberty -max src/lib/avsdpll.lib
read_liberty -min src/lib/avsddac.lib
read_liberty -max src/lib/avsddac.lib

# Load design
read_verilog src/module/vsdbabysoc.synth.v
link_design vsdbabysoc

# Load constraints
read_sdc src/sdc/vsdbabysoc_synthesis.sdc

# Run analysis
report_checks
```

## PVT Corner Analysis

The project supports multi-corner analysis across different Process, Voltage, and Temperature (PVT) conditions:

### Key Corners
- **Setup-critical (max path, slowest):** ss_LowTemp_LowVolt, ss_HighTemp_LowVolt
- **Hold-critical (min path, fastest):** ff_LowTemp_HighVolt, ff_HighTemp_HighVolt

### Current Results

| PVT_CORNER      | Worst Setup Slack | Status |
|-----------------|-------------------|--------|
| tt_025C_1v80    | 1.11              | MET    |

**Note:** Currently only the typical corner (tt_025C_1v80) is available in the setup. Additional corner results will be added when more libraries are available.

## Viewing Timing Reports

### Generated Report Files

```bash
# Navigate to reports directory
cd timing_reports

# View detailed timing report
cat min_max_sky130_fd_sc_hd__tt_025C_1v80.lib.txt

# View slack summary
cat sta_worst_max_slack.txt
cat sta_worst_min_slack.txt
```

### Useful OpenSTA Commands

```tcl
# Setup timing (max delay)
report_checks -path_delay max

# Hold timing (min delay)  
report_checks -path_delay min

# Summary metrics
report_worst_slack -max
report_worst_slack -min
report_tns
report_wns

# Detailed path report
report_checks -path_delay min_max -fields {nets cap slew input_pins fanout} -digits 4
```

## Troubleshooting

### Common Issues

1. **Library Syntax Errors**
   ```
   Error: src/lib/avsdpll.lib line 50, syntax error
   ```
   **Solution:** Comment out problematic lines or fix syntax issues in the library files.

2. **Missing Corner Libraries**
   Download additional Sky130 libraries from:
   https://github.com/efabless/skywater-pdk-libs-sky130_fd_sc_hd/tree/master/timing

3. **OpenSTA Session Management**
   OpenSTA doesn't support the `clear` command. For clean analysis:
   - Exit OpenSTA (`exit`) and restart for each corner
   - Or use separate TCL script files for each corner

## Directory Structure

- `images/`: Design and flow diagrams
- `OpenSTA/`: OpenSTA source, build files, and examples
- `src/`:
  - `gds/`: GDS files for IP blocks
  - `include/`: Include files for design
  - `layout_conf/`: Layout configuration
  - `lef/`: LEF files for IP blocks
  - `lib/`: Liberty timing libraries
  - `module/`: Verilog source files
  - `script/`: Analysis scripts
  - `sdc/`: Timing constraints
- `timing_reports/`: Analysis output files

## References

- [OpenSTA GitHub Repository](https://github.com/The-OpenROAD-Project/OpenSTA)
- [Sky130 Timing Libraries](https://github.com/efabless/skywater-pdk-libs-sky130_fd_sc_hd/tree/master/timing)