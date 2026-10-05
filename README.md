# 9-bit SAR ADC Design in 180nm CMOS

A transistor-level mixed-signal design and simulation of a **9-bit Successive Approximation Register (SAR) ADC)** implemented in **Cadence Virtuoso using a 180nm CMOS technology process**.

The project focuses on understanding the complete signal path of a SAR ADC, from analog input sampling and capacitive DAC conversion to comparator-based decision making and SAR control logic.

---

## Project Overview

Successive Approximation Register (SAR) ADCs are widely used in low-power data-conversion systems because they provide a useful balance between **resolution, conversion speed, power consumption, and circuit complexity**.

In this project, a **9-bit SAR ADC** was designed and simulated as a mixed-signal system. The architecture combines transistor-level analog blocks with behavioral digital/control modeling to implement the successive-approximation conversion process.

The ADC consists of four primary blocks:

```text
                  Analog Input
                       │
                       ▼
                ┌─────────────┐
                │ Sample &    │
                │ Hold        │
                └──────┬──────┘
                       │
                       ▼
                ┌─────────────┐
                │ StrongARM   │◄──────────────┐
                │ Comparator  │               │
                └──────┬──────┘               │
                       │                      │
                 Comparator Decision          │
                       │                      │
                       ▼                      │
                ┌─────────────┐               │
                │ SAR Control │───────────────┤
                │ Logic       │               │
                └──────┬──────┘               │
                       │                      │
                 Digital Code                │
                       │                      │
                       ▼                      │
                ┌─────────────┐               │
                │ Capacitive  │───────────────┘
                │ DAC         │
                └─────────────┘
```

The SAR controller performs the conversion using a **9-step binary-search process**, determining the output code from the MSB to the LSB.

---

# ADC Specifications

| Parameter           | Specification                 |
| ------------------- | ----------------------------- |
| Resolution          | 9-bit                         |
| Sampling Rate       | 20 kS/s                       |
| Input Range         | 0 – 0.8 V                     |
| Analog Supply       | 1.8 V                         |
| Digital Supply      | 1.2 V                         |
| DAC Architecture    | Binary-weighted Capacitor DAC |
| Input Configuration | Single-ended                  |
| Technology          | 180nm CMOS                    |
| Design Environment  | Cadence Virtuoso              |
| Simulator           | Spectre                       |

---

# Architecture

The ADC follows the conventional SAR conversion principle.

For each conversion:

1. The analog input is sampled and held.
2. The SAR controller initially sets the **MSB**.
3. The capacitor DAC generates the corresponding trial voltage.
4. The StrongARM comparator compares the input with the DAC output.
5. Depending on the comparator decision, the trial bit is retained or cleared.
6. The process proceeds toward the LSB.
7. After the final comparison, the 9-bit digital output represents the sampled input.

This binary-search approach significantly reduces the number of comparisons required compared with architectures that test every possible ADC code.

---

# 1. Sample-and-Hold

The Sample-and-Hold stage provides a stable representation of the analog input during the SAR conversion cycle.

During the sampling phase, the input signal is captured. Once the circuit enters the hold phase, the sampled voltage is maintained while the remaining ADC circuitry performs the successive comparisons.

### Design Considerations

The implementation was studied with attention to:

* Charge injection
* Clock feedthrough
* Sampling accuracy
* Voltage droop during the conversion interval
* Stable operation during the hold phase

The S/H stage is particularly important in a SAR ADC because the comparator must evaluate the same input voltage throughout the complete conversion.

---

# 2. Capacitive DAC

The DAC is responsible for converting the temporary SAR digital code into an analog trial voltage.

A **binary-weighted capacitor array** was used as the DAC structure. Instead of continuously generating an analog voltage, the DAC uses charge redistribution to produce the required comparison voltage for each SAR decision.

### DAC Operation

For every bit trial:

```text
SAR Trial Code
      │
      ▼
Capacitor Switching
      │
      ▼
Charge Redistribution
      │
      ▼
DAC Trial Voltage
      │
      ▼
Comparator
```

The capacitor array is updated after every comparator decision, allowing the SAR controller to progressively converge toward the sampled input voltage.

### Why a Capacitive DAC?

A capacitor-based DAC is attractive for SAR ADCs because it performs conversion through **charge redistribution**, avoiding the need for an active amplifier in the DAC path and making it suitable for low-power mixed-signal implementations.

---

# 3. StrongARM Latch Comparator

The comparator forms the decision-making element of the ADC.

A **StrongARM latch comparator** was used to determine the relative magnitude of the sampled input and the DAC-generated trial voltage.

The comparator operates dynamically and regenerates a small input difference into a digital decision.

### Key Characteristics

* Dynamic operation
* Regenerative amplification
* High-speed decision making
* Ideally negligible static power consumption
* Compatibility with clocked SAR operation

During each conversion step, the comparator produces the decision required by the SAR controller to determine whether the current trial bit should remain set or be cleared.

---

# 4. SAR Control Logic

The SAR controller coordinates the entire conversion sequence.

Rather than converting the input in a single operation, the controller performs a **binary search from MSB to LSB**.

For a 9-bit conversion, the controller performs nine successive decisions.

### Decision Process

```text
Set MSB
   │
   ▼
DAC generates trial voltage
   │
   ▼
Comparator decision
   │
   ├── Input > DAC → Keep bit
   │
   └── Input < DAC → Clear bit
   │
   ▼
Move to next bit
   │
   ▼
Repeat until LSB
```

The SAR logic was implemented using **Verilog-A behavioral modeling** and integrated with the transistor-level analog blocks in the Cadence simulation environment.

This allowed the digital control sequence and analog circuitry to be evaluated together as a mixed-signal system.

---

# Mixed-Signal Simulation

A major objective of the project was to understand how behavioral models and transistor-level circuits can be combined within a mixed-signal design environment.

The design flow used:

* **Cadence Virtuoso** for schematic-level circuit implementation
* **Spectre** for circuit simulation
* **Verilog-A** for behavioral modeling of the SAR control logic
* **GPDK 180nm** for CMOS device-level implementation
* **Siemens Calibre** for design verification-related workflow

The analog portions of the ADC were designed at circuit level, while the SAR control functionality was modeled behaviorally.

---

# Design Flow

The overall implementation followed the sequence below:

```text
Architecture Definition
        │
        ▼
ADC Specifications
        │
        ▼
Block-Level Design
        │
        ├── Sample & Hold
        ├── Capacitive DAC
        ├── StrongARM Comparator
        └── SAR Logic
        │
        ▼
Individual Block Simulation
        │
        ▼
Mixed-Signal Integration
        │
        ▼
Transient Simulation
        │
        ▼
ADC Conversion Verification
```

Each major block was considered separately before integrating the complete SAR ADC.

---

# Key Design Challenges

The project provided practical exposure to several issues that are often hidden by high-level ADC descriptions.

### Analog–Digital Interaction

The SAR ADC is not simply an analog circuit connected to a digital circuit. Timing between sampling, DAC switching, comparator evaluation, and SAR decisions directly affects the conversion process.

### Capacitor Switching

The DAC depends on accurate charge redistribution. Therefore, the capacitor network and switching sequence directly influence the generated trial voltage.

### Comparator Decisions

The comparator must reliably resolve relatively small voltage differences during each successive approximation step.

### Timing

The conversion sequence requires proper coordination between:

* Sampling clock
* DAC update
* Comparator evaluation
* SAR decision
* Next-bit transition

Understanding these interactions was an important part of the project.

---

# Tools & Technologies

### EDA / Simulation

* Cadence Virtuoso
* Cadence Spectre
* Siemens Calibre

### Modeling

* Verilog-A

### Technology

* GPDK 180nm CMOS

### Circuit Blocks

* Sample-and-Hold
* Capacitive DAC
* StrongARM Latch Comparator
* SAR Control Logic

---

# What I Learned

This project provided practical experience with the design flow of a mixed-signal integrated circuit rather than studying the SAR ADC architecture only at the block-diagram level.

Key areas explored include:

* SAR ADC architecture and binary-search conversion
* Capacitive DAC operation and charge redistribution
* Dynamic comparator operation
* Sample-and-hold behavior
* Mixed-signal circuit integration
* Verilog-A behavioral modeling
* Cadence Virtuoso schematic design
* Spectre-based transient simulation
* Analog and digital signal interaction
* Timing considerations in data-converter design

More importantly, the project helped connect **transistor-level CMOS circuits with higher-level ADC functionality**, providing a better understanding of how individual circuit blocks work together to implement a complete data converter.

---

# Project Highlights

* Designed a **9-bit SAR ADC architecture**
* Implemented a **binary-weighted capacitive DAC**
* Designed a **StrongARM latch comparator**
* Developed SAR control functionality using **Verilog-A**
* Integrated transistor-level analog blocks with behavioral control logic
* Simulated the complete mixed-signal ADC in **Cadence Virtuoso/Spectre**
* Worked with a **180nm CMOS technology design flow**

---

# Author

**Saksham Kapoor**

B.Tech. Electronics and Communication Engineering
**National Institute of Technology Hamirpur**

[GitHub](https://github.com/Saksham2k6)
