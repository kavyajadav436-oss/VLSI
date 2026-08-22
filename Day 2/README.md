<div align="center">

# ⏱️ Timing Libraries, Synthesis & Flip-Flop RTL

### Day 2 of the RTL Workshop

This lab digs into **technology libraries and timing data**, compares **hierarchical vs. flattened synthesis**, and works through several ways of describing **reset/set behavior in flip-flops** using Verilog RTL.

<p>
  <img src="https://img.shields.io/badge/Tool-Icarus%20Verilog-blue?style=for-the-badge" alt="Icarus Verilog">
  <img src="https://img.shields.io/badge/Tool-GTKWave-orange?style=for-the-badge" alt="GTKWave">
  <img src="https://img.shields.io/badge/Tool-Yosys-2ea44f?style=for-the-badge" alt="Yosys">
  <img src="https://img.shields.io/badge/PDK-SKY130-red?style=for-the-badge" alt="SKY130">
  <img src="https://img.shields.io/badge/Language-Verilog-9cf?style=for-the-badge" alt="Verilog">
</p>

`Part of the` [**RTL Workshop**](https://github.com/ArpithaGarrepalli/RTL_Workshop) `series`

<br>

[Tech Libraries](#-technology-libraries) · [Synthesis Styles](#-hierarchical-vs-flattened-synthesis) · [Flip-Flop RTL](#-flip-flop-rtl-coding) · [Sim & Synth](#-simulation--synthesis) · [Conclusion](#-conclusion) · [Credits](#-credits)

</div>

---

## 📚 Technology Libraries

<details open>
<summary><b>Reading a <code>.lib</code> file and what its name tells you</b></summary>
<br>

A technology library describes every standard cell available for synthesis — functionality, timing, power, and the operating conditions it was characterized under.

The SKY130 library used here:

```text
sky130_fd_sc_hd__tt_025C_1v80.lib
```

The filename itself encodes the operating point:

| Segment | Meaning |
|---|---|
| `tt` | Typical process corner |
| `025C` | 25°C temperature |
| `1v80` | 1.8 V supply |

Open it directly to inspect cells and timing data:

```bash
gedit sky130_fd_sc_hd__tt_025C_1v80.lib
```

<img width="1920" height="1012" alt="SKY130 .lib file contents" src="https://github.com/user-attachments/assets/7bf75f62-4888-4244-90fb-07d948804610" />

</details>

---

## 🌳 Hierarchical vs. Flattened Synthesis

<details open>
<summary><b>Keeping structure vs. collapsing it</b></summary>
<br>

Synthesis can either preserve a design's module boundaries or merge everything into one flat representation — the choice trades off debuggability against how much the tool can optimize across boundaries.

**Hierarchical synthesis** keeps the relationships between modules intact:

```text
        Top Module
        /        \
   Module A    Module B
```

Good for keeping a design organized and individual blocks easy to trace.

**Flattened synthesis** removes those boundaries entirely:

```text
Module A ──┐
           ├──► Flat Design
Module B ──┘
```

This gives the tool room to optimize logic that spans what used to be separate modules.

| Feature | Hierarchical | Flattened |
|---|---|---|
| Module hierarchy | Preserved | Removed |
| Optimization scope | Localized per module | Across the whole design |
| Debugging | Easier | Harder |
| Structure | Modular | Unified |

<img width="1920" height="1012" alt="Hierarchical synthesis netlist" src="https://github.com/user-attachments/assets/46b3f459-8efa-4149-8ac2-36d6915c6fe4" />

<img width="1920" height="1012" alt="Flattened synthesis netlist" src="https://github.com/user-attachments/assets/83b31d8b-c046-4b55-8d50-74a3b9e85dab" />

</details>

---

## 🔁 Flip-Flop RTL Coding

<details open>
<summary><b>Asynchronous reset</b></summary>
<br>

An asynchronous reset can force the flip-flop's output to change without waiting for a clock edge.

```verilog
module dff_asyncres (
    input clk,
    input async_reset,
    input d,
    output reg q
);

always @(posedge clk, posedge async_reset)
begin
    if (async_reset)
        q <= 1'b0;
    else
        q <= d;
end

endmodule
```

As soon as `async_reset` goes high, `q` drops to `0` — no clock edge required.

</details>

<details>
<summary><b>Asynchronous set</b></summary>
<br>

Same idea, opposite direction — forces the output to `1` independent of the clock.

```verilog
module dff_async_set (
    input clk,
    input async_set,
    input d,
    output reg q
);

always @(posedge clk, posedge async_set)
begin
    if (async_set)
        q <= 1'b1;
    else
        q <= d;
end

endmodule
```

</details>

<details>
<summary><b>Synchronous reset</b></summary>
<br>

A synchronous reset is only evaluated on the active clock edge, not immediately.

```verilog
module dff_syncres (
    input clk,
    input sync_reset,
    input d,
    output reg q
);

always @(posedge clk)
begin
    if (sync_reset)
        q <= 1'b0;
    else
        q <= d;
end

endmodule
```

```text
Asynchronous reset:  Reset ─────────► Output changes immediately

Synchronous reset:   Reset ──► Clock edge ──► Output changes
```

</details>

---

## 🧪 Simulation & Synthesis

<details open>
<summary><b>Simulating <code>dff_asyncres</code> in Icarus Verilog</b></summary>
<br>

Compile RTL + testbench:

```bash
iverilog dff_asyncres.v tb_dff_asyncres.v
```

Run it:

```bash
./a.out
```

Open the waveform:

```bash
gtkwave tb_dff_asyncres.vcd
```

<img width="1920" height="1012" alt="dff_asyncres simulation waveform" src="https://github.com/user-attachments/assets/e971737d-85bb-4b78-983a-27dd0c5a2af4" />

</details>

<details open>
<summary><b>Synthesizing it in Yosys</b></summary>
<br>

```bash
yosys
```

Load the library:

```bash
read_liberty -lib /path/to/sky130_fd_sc_hd__tt_025C_1v80.lib
```

Read the RTL:

```bash
read_verilog /path/to/dff_asyncres.v
```

Run synthesis:

```bash
synth -top dff_asyncres
```

Map the flip-flop onto a library cell:

```bash
dfflibmap -liberty /path/to/sky130_fd_sc_hd__tt_025C_1v80.lib
```

Technology-map the rest of the logic:

```bash
abc -liberty /path/to/sky130_fd_sc_hd__tt_025C_1v80.lib
```

View the result:

```bash
show
```

| Command | Purpose |
|---|---|
| `read_liberty` | Load the standard-cell library |
| `read_verilog` | Read the RTL design |
| `synth -top` | Run synthesis on the top module |
| `dfflibmap` | Map flip-flops onto library cells |
| `abc` | Technology-map remaining logic |
| `show` | Display the synthesized circuit |

<img width="1920" height="1012" alt="dff_asyncres gate-level schematic" src="https://github.com/user-attachments/assets/b6aefb1f-9c47-4dcb-8233-5bf461664823" />

</details>

---

## ✅ Conclusion

Day 2 covered how **technology libraries and timing data** feed into synthesis, walked through the trade-offs between **hierarchical and flattened synthesis**, and worked through several flip-flop reset/set styles in Verilog. Each design was verified in **Icarus Verilog** before being synthesized in **Yosys**, tying the RTL coding style directly to the hardware it produces.

---

<div align="center">

## 👤 Credits

**Kavya**

[![GitHub](https://img.shields.io/badge/GitHub-RTL__Workshop-181717?style=for-the-badge&logo=github)](https://github.com/ArpithaGarrepalli/RTL_Workshop)

</div>
