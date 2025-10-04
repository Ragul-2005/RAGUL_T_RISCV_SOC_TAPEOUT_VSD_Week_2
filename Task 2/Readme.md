<div align="center"> <h1>🛠️ Task 2 – Practical BabySoC Functional Modelling </h1></div>

## 📌 Overview  
This task focuses on the **practical side of BabySoC design** through functional modelling.  
Using **Icarus Verilog (iverilog)** and **GTKWave**, I compiled and simulated BabySoC modules to analyze their behavior.  
The goal was to validate **reset, clocking, and dataflow** operations before moving into RTL and physical design stages.  

---

## 🧩 Step 1 – Repository Setup

Start by cloning the official BabySoC repository and navigating into it:

```bash
git clone https://github.com/manili/VSDBabySoC.git
cd VSDBabySoC/
ls
```
You’ll see directories such as `src/`, `output/`, and a `Makefile`.

## 🧱 Step 2 – Explore Source Modules

List the available modules:

```bash
ls src/
ls src/module/
```

These are the key RTL modules of the BabySoC:

<div align="center">

| File           | Description                                     |
| -------------- | ----------------------------------------------- |
| `rvmyth.tlv`   | TL-Verilog RISC-V core (converted to Verilog).  |
| `avsddac.v`    | 10-bit DAC converting digital output to analog. |
| `avsdpll.v`    | PLL generating stable internal clock.           |
| `clk_gate.v`   | Clock-gating for low-power operation.           |
| `vsdbabysoc.v` | Top-level integration of CPU + PLL + DAC.       |
| `testbench.v`  | Stimulus generator and monitor for simulation.  |
</div>

---

## 🧠 Step 3 – Set Up Python Environment & SandPiper SaaS

The RISC-V core (`rvmyth.tlv`) needs to be converted from TL-Verilog to standard Verilog.

```bash
python3 -m venv sp_env
source sp_env/bin/activate
pip install pyyaml click sandpiper-saas
sandpiper-saas -i ./src/module/rvmyth.tlv -o rvmyth.v --bestsv --noline -p verilog --outdir ./src/module/
```
<p align="center">
  <img src="https://github.com/Ragul-2005/RAGUL_T_RISCV_SOC_TAPEOUT_VSD_Week_2/blob/main/Task%202/Images/Screenshot%20from%202025-10-04%2019-13-56.png?raw=true" width="600"/>
</p>

This produces `rvmyth.v` inside the `src/module/` directory.

---

## ▶️ Step 4 – Pre-Synthesis Simulation

### 🔹Pre-Synthesis Simulation

```bash
mkdir -p output/pre_synth_sim

iverilog -o output/pre_synth_sim/pre_synth_sim.out \
  -DPRE_SYNTH_SIM \
  -I src/include -I src/module \
  src/module/testbench.v

cd output/pre_synth_sim
./pre_synth_sim.out
```
- 📊 View in GTKWave:

```bash
gtkwave pre_synth_sim.vcd
```

<p align="center">
  <img src="https://github.com/Ragul-2005/RAGUL_T_RISCV_SOC_TAPEOUT_VSD_Week_2/blob/main/Task%202/Images/Screenshot%20from%202025-10-04%2021-00-46.png?raw=true" width="600"/>
</p>

---
## ▶️ Step 5 – Pre-Synthesis Waveform

<p align="center">
  <img src="https://github.com/Ragul-2005/RAGUL_T_RISCV_SOC_TAPEOUT_VSD_Week_2/blob/main/Task%202/Images/GTKWave_Output.png?raw=true" width="600"/>
</p>

## 📘 Overview
This document presents an analysis of the **PLL (Phase-Locked Loop)** waveform obtained from the pre-synthesis simulation (`pre_synth_sim.vcd`).  
The captured waveform was viewed using **GTKWave** and focuses on reference (`REF`), VCO control, enable logic, and real-valued analog feedback variables.

## 🔹 Key Observations

<div align="center">
| Signal | Description | Observed Behavior |
|--------|--------------|------------------|
| **CLK** | Primary simulation clock | Stable toggling, consistent period. |
| **EN** | PLL enable control | Logic `1` → PLL active during capture window. |
| **ENb_CP**, **ENb_VCO** | Charge pump & VCO enable lines | Asserted (`ENb_VCO=1`), indicating VCO activation; `ENb_CP=x` likely an initialization artifact. |
| **REF** | Reference input signal | Logic `0` in snapshot — reference phase steady. |
| **VCO_IN** | Internal VCO control signal | Regular toggling, used for frequency feedback. |
| **OUT[9:0]** | Analog output representation | Gradual stabilization from ~0.18 → 0.09, representing analog convergence. |
| **Dext[10:0]** | External DAC output codes | Digital equivalents showing decreasing ramp (0x9E → 0x9B → 0x99 → 0x88 → ...). |
</div>
---

## ⏱️ Measured Timing Parameters

<div align="center">
| Parameter | Meaning | Observed Value | Notes |
|------------|----------|----------------|-------|
| **lastedge** | Timestamp of last VCO edge | `42302.84 ns` | Consistent edge capture. |
| **period** | VCO output period | `≈ 35.416 ns` | Equivalent to ~28.3 MHz oscillation frequency. |
| **refpd** | Reference period difference | `≈ 283.33 ns` | Matches reference clock’s expected phase period. |
</div>
---


