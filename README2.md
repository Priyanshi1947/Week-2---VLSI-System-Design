## RVMYTH Core Mixed-Signal Simulation Verification & Known Issue

This document summarizes the successful verification of the **RVMYTH core's digital functionality** and addresses the persistent issue with the **static analog output** from the Digital-to-Analog Converter (DAC) in the current simulation environment.

<hr>

## 1. Digital Core Verification Status

The digital core's functionality has been successfully verified using the following key signals:

### Reset Operation

The successful initialization of the digital core is confirmed by observing the correct response of the **PC** (Program Counter) and the **reset** signal.
* **Verification:** The `reset` signal correctly forces the `PC` to its initial state, demonstrating the core's proper power-on or reset sequence.

### Clocking and Synchronous Behavior

Synchronous operation is verified by examining the relationship between the clock and any registered digital signal.
* **Verification:** Signals from any digital register are observed to change state only on the active edge of the **CLK** signal, confirming **synchronous behavior**.

### Dataflow and Instruction Execution

The core's successful execution of instructions and data processing is confirmed by monitoring the digital input to the DAC.
* **Verification:** The **RV\_TO\_DAC[9:0]** signal (the digital input to the DAC) shows **dynamic changes** over time, indicating the RVMYTH core is actively executing instructions and generating new output data values.

![Digital core signals showing successful initialization and data flow](image1)
***Figure 1: Digital Core Signals (PC, CLK, reset, RV\_TO\_DAC[9:0]) showing successful verification.***

<hr>

## 2. Known Issue: Static DAC Analog Output

While the digital core functions correctly, the DAC's analog output remains static, which is a known limitation in the current compilation setup.

### Problem Description

The analog output signal **dac.OUT** (or **real OUT**) remains static (e.g., at a fixed voltage) throughout the entire simulation, despite the verified, dynamic changes on the digital input **RV\_TO\_DAC[9:0]**.

![Waveform showing dynamic digital input but static analog output](image2)
***Figure 2: Waveform showing dynamic RV\_TO\_DAC[9:0] but static dac.OUT.***

### Root Cause (Reasoning)

This behavior is specifically observed when compiling **mixed-signal designs** using **standard Icarus Verilog (iverilog)** *outside* of the original, specialized make environment.

The DAC utilizes an internal Verilog model that requires the **`real`** datatype to model analog values. The failure is attributed to one of the following:

1.  **Missing External Library:** The `iverilog` compilation process is likely missing a required external library or file necessary to elaborate the specialized analog modeling constructs.
2.  **Unsupported Constructs:** The DAC model contains constructs utilizing the `real` datatype or other mixed-signal features that the specific version of `iverilog` used for compilation cannot fully elaborate or interpret.

**Result:** The failure to elaborate the specialized DAC model causes it to default to a static, unreactive state, while the standard digital logic (the RVMYTH core) elaborates and simulates correctly.
