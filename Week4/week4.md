# CMOS Circuit Design and SPICE Simulation using SKY130 Technology

## Table of Contents

- [CMOS Circuit Design and SPICE Simulation using SKY130 Technology](#cmos-circuit-design-and-spice-simulation-using-sky130-technology)
  - [Table of Contents](#table-of-contents)
  - [Introduction](#introduction)
  - [Workshop Collaterals](#workshop-collaterals)
  - [MOSFET Theory and Operation](#mosfet-theory-and-operation)
    - [Basic Concepts](#basic-concepts)
    - [Regions of Operation](#regions-of-operation)
    - [MOSFET Formulas](#mosfet-formulas)
    - [Short-Channel Effects](#short-channel-effects)
  - [Process Corners and Variation](#process-corners-and-variation)
  - [Lab 1: MOSFET ID vs. VDS Characteristics](#lab-1-mosfet-id-vs-vds-characteristics)
    - [Purpose](#purpose)
    - [SPICE Netlist](#spice-netlist)
    - [What to Measure](#what-to-measure)
    - [Expected Observations](#expected-observations)
  - [Lab 2: Threshold Voltage Extraction](#lab-2-threshold-voltage-extraction)
    - [Purpose](#purpose-1)
    - [SPICE Netlist](#spice-netlist-1)
    - [Extraction Methods](#extraction-methods)
    - [Expected Observations](#expected-observations-1)
  - [Lab 3: CMOS Inverter VTC](#lab-3-cmos-inverter-vtc)
    - [Purpose](#purpose-2)
    - [SPICE Netlist](#spice-netlist-2)
    - [What to Measure](#what-to-measure-1)
    - [Expected Observations](#expected-observations-2)
  - [Lab 4: Transient Behavior - Rise/Fall Delays](#lab-4-transient-behavior---risefall-delays)
    - [Purpose](#purpose-3)
    - [SPICE Netlist](#spice-netlist-3)
    - [Delay Definitions](#delay-definitions)
    - [Measurement Commands](#measurement-commands)
    - [Expected Observations](#expected-observations-3)
  - [Lab 5: Noise Margin Analysis](#lab-5-noise-margin-analysis)
    - [Purpose](#purpose-4)
    - [SPICE Netlist](#spice-netlist-4)
    - [Noise Margin Definitions](#noise-margin-definitions)
    - [Expected Observations](#expected-observations-4)
  - [Lab 6: Power Supply and Device Variation](#lab-6-power-supply-and-device-variation)
    - [Purpose](#purpose-5)
    - [SPICE Netlist for Power Supply Variation](#spice-netlist-for-power-supply-variation)
    - [SPICE Netlist for Device Variation](#spice-netlist-for-device-variation)
    - [Expected Observations](#expected-observations-5)
  - [Results Analysis](#results-analysis)
    - [Summary Table Format](#summary-table-format)
    - [Key Observations to Document](#key-observations-to-document)
  - [Conclusion](#conclusion)
  - [References](#references)

## Introduction

This document covers CMOS circuit design and SPICE simulation using the open-source SKY130 technology node. The focus is on understanding transistor-level circuit behavior that drives the timing characteristics analyzed in Static Timing Analysis (STA). By working through practical SPICE simulations, we gain insights into:

- How transistor physics affects circuit performance
- The relationship between device parameters and timing delays
- Circuit robustness against noise and process variations
- The physical foundations of STA models

These concepts form the bridge between device physics and higher-level circuit behavior that design engineers must understand to create reliable integrated circuits.

## Workshop Collaterals

All simulations are based on the SKY130 Process Design Kit (PDK):
- Workshop materials: https://github.com/kunalg123/sky130CircuitDesignWorkshop/
- SKY130 PDK models path: `../sky130_fd_pr/models/sky130.lib.spice`

## MOSFET Theory and Operation

### Basic Concepts

**MOSFET Structure and Terminals**
- **Gate (G)**: Controls the channel formation
- **Drain (D)**: Terminal where current exits (for NMOS)
- **Source (S)**: Terminal where current enters (for NMOS)
- **Bulk/Body (B)**: Substrate terminal, affects threshold via body effect

**Key Parameters**
- **W**: Channel width (µm)
- **L**: Channel length (µm)
- **VGS**: Gate-source voltage
- **VDS**: Drain-source voltage
- **VSB**: Source-body voltage
- **Vth**: Threshold voltage (voltage required to form channel)
- **µ**: Carrier mobility (different for electrons and holes)
- **Cox**: Gate oxide capacitance per unit area

### Regions of Operation

**1. Cutoff Region**
- Condition: VGS < Vth
- Behavior: No channel formed, negligible current flows
- Equation: ID ≈ 0 (neglecting subthreshold leakage)

**2. Linear/Triode Region**
- Condition: VGS > Vth and VDS < (VGS - Vth)
- Behavior: Channel formed, acts like voltage-controlled resistor
- Equation: ID = μn·Cox·(W/L)·[(VGS-Vth)·VDS - VDS²/2]·(1+λVDS)

**3. Saturation Region**
- Condition: VGS > Vth and VDS ≥ (VGS - Vth)
- Behavior: Channel "pinches off" near drain, current relatively independent of VDS
- Equation: ID = (μn·Cox/2)·(W/L)·(VGS-Vth)²·(1+λVDS)

**Channel Length Modulation**
- Parameter λ accounts for effective channel length reduction as VDS increases
- Causes slight increase in ID with VDS in saturation
- Measured by slope of ID vs VDS in saturation region: λ ≈ (1/ID0)·(∂ID/∂VDS)

### MOSFET Formulas

**Overdrive Voltage**
- VOV = VGS - Vth (measure of how strongly transistor is turned on)

**Body Effect**
- Vth = Vth0 + γ·(√(2ΦF+VSB) - √(2ΦF))
- Where: γ is body effect coefficient, ΦF is Fermi potential

**Transconductance**
- gm = ∂ID/∂VGS = μn·Cox·(W/L)·(VGS-Vth) [in saturation]
- gm = μn·Cox·(W/L)·VDS [in linear region]

**Output Conductance**
- gds = ∂ID/∂VDS = λ·ID [in saturation]
- gds = μn·Cox·(W/L)·[(VGS-Vth) - VDS] [in linear region]

### Short-Channel Effects

As transistor dimensions shrink (L < 1µm), several "short-channel effects" become significant:

**Velocity Saturation**
- At high electric fields, carriers reach maximum velocity (vsat)
- Current becomes limited by: ID ≈ W·Cox·vsat·(VGS-Vth)
- Changes ID-VGS relationship from quadratic to linear at high VGS

**Drain-Induced Barrier Lowering (DIBL)**
- High VDS lowers effective threshold voltage
- Vth decreases as VDS increases: Vth = Vth0 - η·VDS
- Where η is the DIBL coefficient

**Channel Length Modulation**
- More pronounced in short-channel devices
- λ increases as L decreases

**Subthreshold Leakage**
- Current flow when VGS < Vth, follows exponential relationship
- ID,sub ≈ ID0·exp((VGS-Vth)/(n·VT))
- Where n is subthreshold swing factor, VT is thermal voltage (≈26mV at room temperature)

## Process Corners and Variation

**Process Corners** are predefined sets of model parameters that represent manufacturing variations:

**TT (Typical-Typical)**
- Nominal/typical process parameters for both NMOS and PMOS
- Used for standard characterization
- Referenced as "tt" in SPICE model selection

**FF (Fast-Fast)**
- Both NMOS and PMOS have higher mobility and lower threshold voltage
- Results in faster switching and lower delays
- Higher leakage current

**SS (Slow-Slow)**
- Both NMOS and PMOS have lower mobility and higher threshold voltage
- Results in slower switching and higher delays
- Lower leakage current

**SF/FS (Skewed Corners)**
- SF: Slow NMOS, Fast PMOS
- FS: Fast NMOS, Slow PMOS
- Used to check worst-case scenarios for timing paths

**Other Variations**
- Temperature corners: Typically -40°C, 25°C, 125°C
- Voltage corners: ±10% of nominal supply voltage

Process corners are critical for STA as they define the timing extremes that must be accounted for in the design to ensure functionality across all manufacturing variations.

## Lab 1: MOSFET ID vs. VDS Characteristics

### Purpose
To observe MOSFET operation in linear and saturation regions by measuring drain current (ID) for different gate and drain voltages.

### SPICE Netlist
```spice
* ID vs VDS characteristics of NMOS transistor
.param temp=27

*Include model file
.lib "../sky130_fd_pr/models/sky130.lib.spice" tt

*Netlist Description
XM1 Vdd n1 0 0 sky130_fd_pr__nfet_01v8 w=5 l=2
R1 n1 in 55
Vdd vdd 0 1.8V
Vin in 0 1.8V

*Simulation commands
.op
.dc Vdd 0 1.8 0.1 Vin 0 1.8 0.2

*Interactive interpreter commands
.control
run
display
setplot dc1
plot -Vdd#branch
.endc
.end
```

### What to Measure
1. ID vs VDS curves for different VGS values
2. Transition point from linear to saturation region
3. Channel length modulation effect (slope in saturation region)

### Expected Observations
- At low VDS: Linear relationship between ID and VDS
- At higher VDS: Current "saturates" and becomes relatively independent of VDS
- Higher VGS values result in higher current levels
- Slight upward slope in saturation region due to channel length modulation

## Lab 2: Threshold Voltage Extraction

### Purpose
To determine the threshold voltage (Vth) of a MOSFET and observe velocity saturation effects in short-channel devices.

### SPICE Netlist
```spice
*Threshold voltage extraction for NMOS
.param temp=27

*Include model file
.lib "../sky130_fd_pr/models/sky130.lib.spice" tt

*Netlist Description
XM1 Vdd n1 0 0 sky130_fd_pr__nfet_01v8 w=5 l=2
R1 n1 in 55
Vdd vdd 0 1.8V
Vin in 0 1.8V

*Simulation commands
.op
.dc Vin 0 1.8 0.01 Vdd 0 1.8 1.8

*Interactive interpreter commands
.control
run
display
setplot dc1
plot -Vdd#branch
.endc
.end
```

### Extraction Methods

**1. Square-Root Method (for long-channel devices)**
- Plot √ID vs VGS
- Find linear region and extrapolate to x-axis
- Intersection point gives Vth

**2. Constant Current Method**
- Define a specific current: ID = (W/L) × 100nA
- Find VGS where this current flows
- This VGS is the threshold voltage

**3. Transconductance Method**
- Plot transconductance (gm = ∂ID/∂VGS) vs VGS
- Extrapolate the linear region to zero
- The x-intercept gives Vth

### Expected Observations
- For long-channel devices: Clear square-law behavior (ID ∝ (VGS-Vth)²)
- For short-channel devices: Sub-quadratic relationship due to velocity saturation
- SKY130 NMOS Vth typically around 0.8V (but verify with your specific model)

## Lab 3: CMOS Inverter VTC

### Purpose
To analyze how a CMOS inverter transitions between logic states and to identify the switching threshold.

### SPICE Netlist
```spice
*VTC of CMOS inverter
.param temp = 27

*Netlist description
XM1 out in Vdd Vdd sky130_fd_pr__pfet_01v8 W=0.84 L=0.15
XM2 out in 0 0 sky130_fd_pr__nfet_01v8 W=0.36 L=0.15 
cload out 0 50fF
Vdd Vdd 0 1.8V
Vin in 0 1.8V

*Include model file
.lib "../sky130_fd_pr/models/sky130.lib.spice" tt

*Simulation commands
.op
.dc Vin 0 1.8V 0.01

*Interactive interpreter commands
.control
run
display
setplot dc1
plot in out
.endc
.end
```

### What to Measure
1. Switching threshold (Vm): point where Vin = Vout
2. Gain in transition region: maximum value of |dVout/dVin|

### Expected Observations
- S-shaped transfer curve
- Switching threshold around 0.9V for balanced design (Wp/Lp ≈ 2.5×Wn/Ln)
- High gain in transition region (steep slope)
- Relationship between transistor sizing and Vm position

## Lab 4: Transient Behavior - Rise/Fall Delays

### Purpose
To measure propagation delays in CMOS inverters, which determine circuit speed.

### SPICE Netlist
```spice
*Transient analysis of CMOS inverter
.param temp = 27

*Netlist description
XM1 out in Vdd Vdd sky130_fd_pr__pfet_01v8 W=0.84 L=0.15
XM2 out in 0 0 sky130_fd_pr__nfet_01v8 W=0.36 L=0.15 
cload out 0 50fF
Vdd Vdd 0 1.8V
Vin in 0 pulse 0 1.8V 0 10ps 10ps 2ns 4ns

*Include model file
.lib "../sky130_fd_pr/models/sky130.lib.spice" tt

*Simulation commands
.op
.tran 10ps 10ns

*Interactive interpreter commands
.control
run
display
setplot tran1
plot in out
.endc
.end
```

### Delay Definitions
- **Rise delay (tpLH)**: Time for output to rise from 10% to 90% when input falls
- **Fall delay (tpHL)**: Time for output to fall from 90% to 10% when input rises
- **Propagation delay**: Time between 50% input and 50% output transitions

### Measurement Commands
```spice
.control
  let vdd_half = 1.8/2
  meas tran tphl TRIG v(in) VAL={vdd_half} RISE=1 TARG v(out) VAL={vdd_half} FALL=1
  meas tran tplh TRIG v(in) VAL={vdd_half} FALL=1 TARG v(out) VAL={vdd_half} RISE=1
  print tphl tplh
.endc
```

### Expected Observations
- For balanced inverter (Wp/Lp ≈ 2.5×Wn/Ln), rise and fall delays are approximately equal
- For different sizes, observe how delay ratio changes
- Typical delays in the picosecond range for SKY130 technology

## Lab 5: Noise Margin Analysis

### Purpose
To determine a CMOS inverter's noise margins, which indicate its ability to tolerate noise without producing erroneous outputs.

### SPICE Netlist
```spice
*Noise margin analysis of CMOS inverter
.param temp = 27

*Netlist description
XM1 out in Vdd Vdd sky130_fd_pr__pfet_01v8 W=1 L=0.15
XM2 out in 0 0 sky130_fd_pr__nfet_01v8 W=0.36 L=0.15
cload out 0 50fF
Vdd Vdd 0 1.8V
Vin in 0 1.8V

*Include model file
.lib "../sky130_fd_pr/models/sky130.lib.spice" tt

*Simulation commands
.op
.dc Vin 0 1.8V 0.01

*Interactive interpreter commands
.control
run
display
setplot dc1
plot out vs in
.endc
.end
```

### Noise Margin Definitions
- **VOH**: Maximum output voltage for logic '1' (typically ≈ VDD)
- **VOL**: Minimum output voltage for logic '0' (typically ≈ 0V)
- **VIH**: Input voltage above which output is recognized as logic '0'
- **VIL**: Input voltage below which output is recognized as logic '1'
- **NMH** (Noise Margin High): VOH - VIH
- **NML** (Noise Margin Low): VIL - VOL

VIH and VIL are typically defined at points where the slope of the VTC equals -1.

### Expected Observations
- For a balanced SKY130 inverter:
  - VOH ≈ 1.8V (VDD)
  - VOL ≈ 0.1V
  - VIH ≈ 0.95V
  - VIL ≈ 0.8V
  - NMH ≈ 0.85V
  - NML ≈ 0.7V

## Lab 6: Power Supply and Device Variation

### Purpose
To analyze how variations in supply voltage and transistor sizing affect CMOS inverter performance.

### SPICE Netlist for Power Supply Variation
```spice
*Power supply variation analysis
.param temp = 27

*Netlist description
XM1 out in Vdd Vdd sky130_fd_pr__pfet_01v8 w=1 l=0.15
XM2 out in 0 0 sky130_fd_pr__nfet_01v8 w=0.36 l=0.15
cload out 0 50fF
Vin in 0 1.8V
Vdd Vdd 0 1.8V

*Include model file
.lib "../sky130_fd_pr/models/sky130.lib.spice" tt

*Control commands
.control
let powersupply = 1.8
alter Vdd = powersupply
     let supplyvoltagevariation = 0
     dowhile supplyvoltagevariation < 6
     dc Vin 0 1.8 0.01
     let powersupply = powersupply - 0.2
     alter Vdd = powersupply
     let supplyvoltagevariation = supplyvoltagevariation + 1
    end

plot dc1.out vs in dc2.out vs in dc3.out vs in dc4.out vs in dc5.out vs in xlabel "input voltage [V]" 
ylabel "output voltage [V]" title "Inverter dc characteristics as a function of supply voltage" 
.endc
.end
```

### SPICE Netlist for Device Variation
```spice
*Device variation analysis
.param temp=27

*Include model file
.lib "../sky130_fd_pr/models/sky130.lib.spice" tt

*Netlist Description
XM1 out in vdd vdd sky130_fd_pr__pfet_01v8 w=7 l=0.15
XM2 out in 0 0 sky130_fd_pr__nfet_01v8 w=0.42 l=0.15
Cload out 0 50fF
Vdd vdd 0 1.8V
Vin in 0 1.8V

*Simulation commands
.op
.dc Vin 0 1.8 0.01

*Interactive interpreter command
.control
run
setplot dc1
display
plot out vs in
.endc
.end
```

### Expected Observations

**Power Supply Variation**
- Lower VDD reduces noise margins
- Switching threshold scales approximately linearly with VDD
- Propagation delays increase as VDD decreases

**Device Variation**
- Increasing PMOS width shifts switching threshold higher
- Wp/Lp ≈ 2.5×Wn/Ln provides balanced rise and fall delays
- Larger transistors reduce delays but increase power consumption

## Results Analysis

### Summary Table Format

Create a table to summarize your key measurements across different conditions:

| Parameter | Sizing (Wp:Wn) | TT Corner | SS Corner | FF Corner |
|-----------|----------------|-----------|-----------|-----------|
| Vth (V)   | -              | value     | value     | value     |
| Vm (V)    | 1:1            | value     | value     | value     |
| Vm (V)    | 2.5:1          | value     | value     | value     |
| tpLH (ps) | 1:1            | value     | value     | value     |
| tpHL (ps) | 1:1            | value     | value     | value     |
| tpLH (ps) | 2.5:1          | value     | value     | value     |
| tpHL (ps) | 2.5:1          | value     | value     | value     |
| NMH (V)   | 2.5:1          | value     | value     | value     |
| NML (V)   | 2.5:1          | value     | value     | value     |

### Key Observations to Document

1. **MOSFET Characteristics**
   - Linear to saturation transition points
   - Effect of velocity saturation on short-channel devices
   - Channel length modulation effects

2. **Threshold Voltage**
   - Extracted value comparison with PDK specification
   - Process corner effects on threshold voltage

3. **CMOS VTC**
   - Relationship between transistor sizing and switching threshold
   - Gain in transition region

4. **Delay Analysis**
   - Relationship between transistor sizing and delay balance
   - Process corner impact on delays
   - Supply voltage impact on delays

5. **Noise Margins**
   - How sizing affects noise margins
   - Process and voltage effects on noise immunity

## Conclusion

This workshop bridges device physics and circuit performance through hands-on SPICE simulation. Key takeaways include:

1. **Physical Fundamentals**: Understanding how device-level parameters (mobility, threshold, geometry) translate to circuit behavior.

2. **Sizing Strategy**: The trade-offs involved in transistor sizing:
   - Wp/Lp ≈ 2.5×Wn/Ln provides balanced rise/fall delays
   - Different applications may require different optimization targets

3. **Variability Management**: How process, voltage, and temperature variations affect circuit performance and timing constraints.

4. **STA Foundations**: The physical basis for delay models, corners, and margin requirements used in Static Timing Analysis.

By connecting device physics to circuit timing, we build intuition for designing robust circuits that can be accurately characterized for timing verification.

## References

1. Kunalg123, "SKY130 Circuit Design Workshop," GitHub repository, https://github.com/kunalg123/sky130CircuitDesignWorkshop/

2. Google/SkyWater, "Open Source PDK for the 130nm Process Node," https://github.com/google/skywater-pdk

3. Rabaey, J. M., Chandrakasan, A., & Nikolić, B. (2003). Digital integrated circuits: A design perspective. Pearson Education.

4. Baker, R. J. (2010). CMOS: Circuit Design, Layout, and Simulation (3rd ed.). Wiley-IEEE Press.

5. Kang, S. M., & Leblebici, Y. (2003). CMOS Digital Integrated Circuits Analysis & Design. McGraw-Hill.

6. Sedra, A. S., & Smith, K. C. (2014). Microelectronic Circuits (7th ed.). Oxford University Press.

7. NgSpice Documentation, http://ngspice.sourceforge.net/docs.html