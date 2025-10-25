# OpenROAD Flow Setup and Floorplan + Placement Guide

## Detailed Installation Commands and Execution Steps

### Prerequisites Installation
```bash
# Update system packages
sudo apt-get update
sudo apt-get upgrade

# Install required dependencies (if not already installed)
sudo apt-get install -y build-essential git cmake python3 python3-pip
```

### 1. OpenROAD Installation Steps

```bash
# Clone the OpenROAD repository
git clone --recursive https://github.com/The-OpenROAD-Project/OpenROAD-flow-scripts
cd OpenROAD-flow-scripts

# Run the setup script
sudo ./setup.sh

# Build OpenROAD with local installation
./build_openroad.sh --local

# Set up the environment
source ./env.sh

# Verify installation
yosys -help
openroad -help
```

### 2. Running Floorplan and Placement

```bash
# Navigate to flow directory
cd flow

# Set up design configuration
# Edit config.mk to specify your design and only run up to placement
# Example configuration settings:
DESIGN_CONFIG=./designs/src/gcd/config.mk
FLOW_VARIANT=base

# Run only up to placement stage
make STEPS="synthesis floorplan placement" DESIGN_CONFIG=$DESIGN_CONFIG FLOW_VARIANT=$FLOW_VARIANT
```

### 3. Verification Commands

```bash
# To view floorplan results
make gui_floorplan

# To view placement results
make gui_placement

# To check logs
cat logs/floorplan.log
cat logs/placement.log
```

### 4. Important Directory Structure
```
OpenROAD-flow-scripts/
├── flow/
│   ├── design/          # Design files
│   ├── logs/           # Generated log files
│   ├── reports/        # Analysis reports
│   └── results/        # Output files and layouts
```

### 5. Key Files to Monitor
- `flow/reports/floorplan.rpt` - Floorplan report
- `flow/reports/placement.rpt` - Placement report
- `flow/logs/floorplan.log` - Detailed floorplan logs
- `flow/logs/placement.log` - Detailed placement logs

### 6. Expected Outputs
After successful execution, you should see:
- Core area and die dimensions in floorplan reports
- Standard cell placement information in placement reports
- GUI showing placed cells when using `make gui_placement`

### 7. Documentation Collection
```bash
# Capture terminal output
script installation_log.txt
# (perform installation steps)
exit

# Take screenshots
# Use your system's screenshot tool or:
scrot -u 'floorplan_%Y%m%d_%H%M%S.png'
scrot -u 'placement_%Y%m%d_%H%M%S.png'
```

### Troubleshooting Tips
- If environment variables are not set: `source ./env.sh`
- If tools are not found: Check PATH in `env.sh`
- For GUI issues: Ensure X11 forwarding is enabled if running remotely