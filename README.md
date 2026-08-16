# RTL Workshop — Day 1

## Introduction

Day 1 is focused on the basic RTL simulation flow using Verilog. The main goal is to understand how a Verilog design is tested before it is implemented in hardware.

### Topics covered

- RTL and Verilog basics
- Design module and testbench
- Icarus Verilog simulation
- VCD waveform generation
- GTKWave waveform viewing
- Basic Yosys synthesis idea
- 2:1 multiplexer implementation

---

## 1. Important Terms

### RTL Design

RTL (Register Transfer Level) is a way of describing digital hardware using a hardware description language such as Verilog. The code describes how signals and logic behave.

### Simulator

A simulator runs the Verilog design without requiring physical hardware. It allows us to check whether the circuit gives the expected result for different input values.

### Testbench

A testbench is a separate Verilog module used to apply inputs to the design under test. It observes the output and helps verify the design.

### DUT

DUT means **Design Under Test**. In this exercise, the 2:1 multiplexer is the DUT.

---

## 2. Simulation Flow

The basic flow used in this exercise is:

**Verilog Design + Testbench → Icarus Verilog → VCD waveform → GTKWave**

Icarus Verilog compiles and runs the Verilog simulation. The testbench creates a VCD file containing signal changes, and GTKWave displays those changes as waveforms.

---

## 3. 2:1 Multiplexer

A 2:1 multiplexer has:

- Two data inputs: `data_a`, `data_b`
- One select input: `select`
- One output: `result`

The select signal decides which input reaches the output.

| select | result |
|--------|--------|
| 0      | data_a |
| 1      | data_b |

The Boolean expression is:

`result = (~select & data_a) | (select & data_b)`

---

## 4. Files in This Folder

```text
Day_1/
├── README.md
├── mux2x1.v
└── mux2x1_tb.v
```

---

## 5. Design File

The multiplexer is described in `mux2x1.v`.

The design uses a continuous assignment. Since this is combinational logic, the output changes whenever one of its inputs changes.

---

## 6. Testbench

`mux2x1_tb.v` supplies all eight possible combinations of the three one-bit inputs.

The testbench also creates a VCD file named:

`mux2x1_wave.vcd`

The waveform can then be opened in GTKWave.

---

## 7. Running the Simulation

Open a terminal inside this folder.

### Compile

```bash
iverilog -o mux2x1_sim mux2x1.v mux2x1_tb.v
```

### Run

```bash
vvp mux2x1_sim
```

### Open the waveform

```bash
gtkwave mux2x1_wave.vcd
```

Icarus Verilog can compile Verilog source files and `vvp` runs the generated simulation. GTKWave is used to inspect the resulting waveform. 

---

## 8. Expected Observation

When `select = 0`, the output should follow `data_a`.

When `select = 1`, the output should follow `data_b`.

The testbench checks both select conditions with different input combinations, so the complete truth table is covered.

---

## 9. Yosys — Basic Idea

Yosys is an open-source synthesis tool. In a typical RTL flow, synthesis converts the RTL description into a lower-level netlist representation.

For this Day 1 exercise, the main focus is simulation rather than a complete synthesis flow.

---

## 10. What I Learned

- RTL code describes digital hardware behavior.
- A testbench is used to provide test inputs.
- Icarus Verilog can be used for Verilog simulation.
- A VCD file records signal changes during simulation.
- GTKWave helps us inspect those signal changes visually.
- A 2:1 MUX selects one of two inputs using a select signal.
- Simulation is useful for finding logic errors before hardware implementation.

