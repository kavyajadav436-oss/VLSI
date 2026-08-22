<div align="center">

# 🧬 RTL Optimization and Synthesis

### Day 3 of the RTL Workshop

This lab looks at how a synthesis tool takes an RTL description and turns it into an efficient gate-level implementation — covering basic logic optimization, constant propagation in D flip-flops, sequential optimization, and a counter example, all synthesized and examined in **Yosys**.

<p>
  <img src="https://img.shields.io/badge/Tool-Icarus%20Verilog-blue?style=for-the-badge" alt="Icarus Verilog">
  <img src="https://img.shields.io/badge/Tool-GTKWave-orange?style=for-the-badge" alt="GTKWave">
  <img src="https://img.shields.io/badge/Tool-Yosys-2ea44f?style=for-the-badge" alt="Yosys">
  <img src="https://img.shields.io/badge/PDK-SKY130-red?style=for-the-badge" alt="SKY130">
  <img src="https://img.shields.io/badge/Language-Verilog-9cf?style=for-the-badge" alt="Verilog">
</p>

`Part of the` [**RTL Workshop**](https://github.com/ArpithaGarrepalli/RTL_Workshop) `series`

<br>

[Objective](#-objective) · [RTL Optimization](#-what-rtl-optimization-means) · [Logic Optimization](#-logic-optimization) · [Constant Propagation](#-constant-propagation) · [DFF Optimization](#-d-flip-flop-optimization) · [Counter](#-counter-optimization) · [Why It Matters](#-why-optimization-matters) · [Observations](#-key-observations) · [Conclusion](#-conclusion) · [Credits](#-credits)

</div>

---

## 🎯 Objective

<details open>
<summary><b>What this session sets out to cover</b></summary>
<br>

- Understand why RTL gets optimized during synthesis, and how.
- See how Boolean expressions get mapped onto hardware cells.
- Study constant propagation in both combinational and sequential circuits.
- Understand how redundant or unnecessary hardware gets eliminated.
- Examine synthesized gate-level output directly.
- Look at sequential optimization on a concrete example — a counter.
- Cross-check sequential behavior against simulation waveforms.

</details>
<img width="1920" height="1012" alt="image" src="https://github.com/user-attachments/assets/365a297b-a45c-4862-aa1b-9b3faa2d84fd" />

## 🏗️ What RTL Optimization Means

<details open>
<summary><b>Same function, better hardware</b></summary>
<br>

RTL optimization improves how an RTL design gets implemented in hardware — without changing what it's supposed to do. An RTL description only specifies *behavior*; many different hardware structures can satisfy the same behavior, and the synthesis tool picks among them based on the target technology and cell library.

**What the tool can do during optimization:**
- Boolean simplification
- Constant propagation
- Redundant-logic removal
- Unused-signal removal
- Logic restructuring
- Technology mapping
- Sequential optimization

The goal is a functionally correct implementation that uses hardware resources efficiently — and the choices made here ripple directly into:

- Area
- Power consumption
- Timing
- Standard-cell count
- Switching activity

Which is why RTL optimization sits as a critical step between RTL design and physical implementation.

</details>

---

## 🔣 Logic Optimization

<details open>
<summary><b>Simplifying combinational logic without changing behavior</b></summary>
<br>

Combinational logic can often be simplified using nothing more than basic Boolean identities:

```text
A AND 1 = A
A AND 0 = 0
A OR  0 = A
A OR  1 = 1
```

Synthesis tools apply identities like these to strip constants and redundant terms out of an expression before mapping what's left onto cells from the target library. The three experiments below walk through this on progressively larger AND/OR expressions.

</details>

<details open>
<summary><b>AND logic</b></summary>
<br>

A two-input AND gate: `Y = A · B` — output is HIGH only when both inputs are HIGH.

| A | B | Y |
|:-:|:-:|:-:|
| 0 | 0 | 0 |
| 0 | 1 | 0 |
| 1 | 0 | 0 |
| 1 | 1 | 1 |

The RTL maps directly onto the matching logic cell in the target library.

![AND logic optimization](images/opt_check.png)

The synthesized result shows the standard cell the RTL Boolean expression resolves to.

</details>

<details>
<summary><b>OR logic</b></summary>
<br>

A two-input OR gate: `Y = A + B` — output is HIGH when at least one input is HIGH.

| A | B | Y |
|:-:|:-:|:-:|
| 0 | 0 | 0 |
| 0 | 1 | 1 |
| 1 | 0 | 1 |
| 1 | 1 | 1 |

![OR logic optimization](images/opt_check2.png)

Same idea as the AND case — the Boolean function maps directly to its corresponding cell.

</details>

<details>
<summary><b>Three-input AND logic</b></summary>
<br>

Extending to three inputs: `Y = A · B · C` — output is HIGH only when all three are HIGH.

| A | B | C | Y |
|:-:|:-:|:-:|:-:|
| 0 | 0 | 0 | 0 |
| 0 | 0 | 1 | 0 |
| 0 | 1 | 0 | 0 |
| 0 | 1 | 1 | 0 |
| 1 | 0 | 0 | 0 |
| 1 | 0 | 1 | 0 |
| 1 | 1 | 0 | 0 |
| 1 | 1 | 1 | 1 |

![Three-input AND optimization](images/opt_check3.png)

Shows how a multi-input Boolean function gets mapped once more inputs are involved.

</details>

---

## 🔢 Constant Propagation

<details open>
<summary><b>Letting known values simplify the logic around them</b></summary>
<br>

If a signal is known to always be `0` or `1`, the synthesis tool can fold that fact directly into whatever logic depends on it:

```text
A AND 0 = 0
A AND 1 = A
A OR  0 = A
A OR  1 = 1
```

Practically: an AND gate with one input permanently tied to `0` never needs to actually compute anything — its output is always `0`. Same logic applies in reverse to an OR gate with an input tied to `1`.

This isn't limited to combinational logic — whenever the tool can prove a *stored* value is fixed, the same propagation applies to sequential circuits too, cutting down the hardware needed for the final implementation.

</details>

---
<img width="1920" height="1012" alt="image" src="https://github.com/user-attachments/assets/6662afb5-788a-43fd-b049-6ea7e69ba27c" />


## 🔁 D Flip-Flop Optimization

<details open>
<summary><b>What a constant D input means for synthesis</b></summary>
<br>

A D flip-flop is a basic storage element in synchronous logic — `D` is the data to store, and the clock decides when it gets latched to the output. For a simple positive-edge-triggered flip-flop:

```text
Q(next) = D   — at the active clock edge
```

If `D` is tied to a constant, the tool already knows the flip-flop's eventual behavior:

- `D = 0` → the stored value settles to `0`
- `D = 1` → the stored value settles to `1`

The three labs below walk through this with increasingly clear synthesis results.

</details>

<details open>
<summary><b>DFF Constant 1</b></summary>
<br>

A D flip-flop with a constant tied to its data input.

**Synthesized circuit**

![DFF Constant 1 synthesized](images/dff_const1_diag.png)

**Simulation waveform**

![DFF Constant 1 waveform](images/dff_const1.png)

Shows what it looks like to wire a constant directly into a storage element's data input.

</details>

<details>
<summary><b>DFF Constant 2</b></summary>
<br>

Pushes the same idea further — once the tool confirms a signal never changes, it propagates that constant through the surrounding logic too.

**Synthesized circuit**

![DFF Constant 2 synthesized](images/dff_const2_diag.png)

**Simulation waveform**

![DFF Constant 2 waveform](images/dff_const2.png)

The waveform is the functional cross-check against clock and output behavior.

</details>

<details>
<summary><b>DFF Constant 3</b></summary>
<br>

Continues the constant-propagation study — by this point, the effect on the synthesized structure is more pronounced.

**Synthesized circuit**

![DFF Constant 3 synthesized](images/dff_const3_diag.png)

**Simulation waveform**

![DFF Constant 3 waveform](images/dff_const3.png)

Confirms the optimized circuit still behaves as expected — checking structure *and* simulation matters here.

</details>

---

## 🔄 Counter Optimization

<details open>
<summary><b>Optimizing a sequential circuit with real state</b></summary>
<br>

A counter steps through a fixed sequence of states using flip-flops plus combinational next-state logic. For an N-bit counter, the number of reachable states is `2^N`.

A 3-bit counter, for example:

```text
000 → 001 → 010 → 011 → 100 → 101 → 110 → 111 → (back to 000)
```

Counters are a good test case for sequential optimization since they combine storage *and* next-state logic in one design.

**Original counter**

![Counter optimization](images/counter_opt.png)

**Modified counter**

![Modified counter optimization](images/counter_opt_modified.png)

Comparing the two shows how even a small RTL change can shift the resulting hardware structure.

</details>

---

## ⚖️ Why Optimization Matters

<details open>
<summary><b>Area, power, timing, and hardware efficiency</b></summary>
<br>

**Area** — cutting unnecessary logic means fewer standard cells, which means less silicon.

**Power** — every switching node burns dynamic power; less unnecessary logic and switching activity means lower consumption.

**Timing** — the number and type of cells along a path set the propagation delay; optimization can shorten the critical path.

**Hardware efficiency** — the same function ends up built from fewer or better-suited resources.

Taken together, these are what make optimization a real, practical part of getting to a usable VLSI implementation.

</details>

---
<img width="1920" height="1012" alt="image" src="https://github.com/user-attachments/assets/cdbc9852-6f69-4dc6-a73e-34f4347c6d2b" />

## 🔑 Key Observations

<details open>
<summary><b>What Day 3 showed, concretely</b></summary>
<br>

1. RTL Boolean expressions map directly onto matching hardware cells during synthesis.
2. Different Boolean functions produce visibly different synthesized structures.
3. Boolean identities are the mechanism behind combinational simplification.
4. Constant values propagate through sequential logic just as they do through combinational logic.
5. Synthesis actively removes portions of a design that turn out to be unnecessary.
6. The synthesized circuit's structure can differ from the RTL's while preserving identical behavior.
7. Sequential circuits need extra care — behavior depends on stored state and clock events, not just current inputs.
8. Counters combine storage elements with next-state logic, making them a good sequential test case.
9. Waveforms remain essential for verifying optimized sequential designs actually behave correctly.
10. Optimization choices ripple into area, power, timing, and overall hardware efficiency.

</details>

---

## ✅ Conclusion

Day 3 was hands-on practice with RTL optimization and synthesis. The combinational logic labs showed how basic Boolean operations map onto hardware; the constant-propagation labs showed how known values simplify logic; and the D flip-flop labs applied the same ideas to sequential circuits. The counter example extended this to a larger sequential design with real storage and next-state logic.

Across all of it, the same point kept coming back: synthesis isn't a literal RTL-to-gates translation. The tool actively analyzes the design, applies optimization techniques, and produces an efficient hardware representation — all while preserving the behavior the RTL described.

---

<div align="center">

## filled by
J.kavya
btech-ece

**Kavya**

[![GitHub](https://img.shields.io/badge/GitHub-RTL__Workshop-181717?style=for-the-badge&logo=github)](https://github.com/ArpithaGarrepalli/RTL_Workshop)

</div>
