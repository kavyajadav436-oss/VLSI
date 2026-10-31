# RTL Design Workshop — Expanded Edition

A five-day walkthrough of the RTL-to-gates flow: writing Verilog, simulating it, synthesizing it against a real standard-cell library, optimizing it, and verifying that the gate-level netlist still behaves like the RTL it came from.

Every day below keeps the same topic list as the original workshop log, but each topic is written out in full — the underlying concept, why it matters, and what a lab run against it actually shows — rather than left as a bullet pointing at a separate file.

---

## 📚 Workshop Progress

| Day | Focus | Core Question Answered |
|---|---|---|
| Day 1 | Verilog RTL, Icarus Verilog, GTKWave, Yosys | How does a design go from text to a gate-level schematic? |
| Day 2 | Timing libraries, synthesis strategies, flip-flop coding | Why does the *same* cell come in a dozen variants, and how does reset style change hardware? |
| Day 3 | Logic optimization, constant propagation | What can a synthesis tool simplify away without changing behavior? |
| Day 4 | Gate-level simulation, coding-style pitfalls | Does the netlist actually behave like the RTL simulation predicted? |
| Day 5 | IF/CASE coding, latch inference, loops | How does a single missing branch turn combinational logic into unwanted storage? |

---

## 🔄 Overall RTL Design Flow

```
RTL Code (Verilog)
      ↓
RTL Simulation (Icarus Verilog + testbench)
      ↓
Waveform Analysis (GTKWave)
      ↓
Synthesis (Yosys, reading the RTL + a .lib file)
      ↓
Logic Optimization (constant propagation, dead-logic removal, resource sharing)
      ↓
Technology Mapping (abc, mapping generic gates → real standard cells)
      ↓
Gate-Level Netlist
      ↓
Gate-Level Simulation (same testbench, netlist instead of RTL)
      ↓
Verification (RTL behavior == netlist behavior?)
```

Nothing in this flow is optional if the goal is silicon: RTL simulation alone only proves the *description* is self-consistent, not that the *hardware* Yosys eventually builds will behave the same way. Day 4 exists specifically because those two things can quietly diverge.

---

# 🟢 Day 1 — RTL Design, Simulation & Synthesis

## Simulator, Design, and Testbench

Three pieces are needed before anything can be verified:

- **Design** — the Verilog module describing intended hardware behavior.
- **Testbench** — a non-synthesizable wrapper module that instantiates the design, drives its inputs over time, and (usually) dumps signal activity to a waveform file. A testbench is never itself meant to become hardware.
- **Simulator** — the tool that executes both, advancing simulated time and evaluating every `always`/`assign` statement whenever its inputs change.

The simulator's job is event-driven: it doesn't run continuously — it wakes up a process only when something that process depends on changes value, evaluates the new outputs, and goes back to sleep. Understanding this event-driven model is what makes waveform debugging make sense later.

## Icarus Verilog Simulation Flow

Icarus Verilog (`iverilog`) is a two-step tool: it first compiles the design and testbench into an intermediate executable, then that executable is run separately to actually produce the waveform dump.

```bash
iverilog -o sim_out design.v testbench.v
./sim_out
```

Running the compiled output executes every `$dumpfile`/`$dumpvars` call inside the testbench, which is what produces the `.vcd` (Value Change Dump) file GTKWave later reads.

## 2:1 Multiplexer — First Design

```verilog
module mux_2to1 (
    input  a,
    input  b,
    input  sel,
    output y
);
assign y = sel ? b : a;
endmodule
```

This is deliberately the simplest possible combinational circuit: one continuous assignment, no clock, no state. It is the first design in the workshop precisely because any simulation-synthesis mismatch here can only come from a tooling or coding-style issue — there's no sequential timing to confuse the picture.

## GTKWave Waveform Analysis

GTKWave loads the `.vcd` file and renders every dumped signal as a timing diagram. The core skill being built here is reading a waveform back into a mental model of the RTL: does `y` track `b` exactly while `sel` is high, and `a` exactly while `sel` is low, with no glitches at the transition edges? A clean mux waveform shows `y` changing only at the same simulation timestep as whichever input is currently selected, or as `sel` itself.

## RTL Design and Synthesis

**Synthesis** is the process of translating an RTL description into a network of logic gates drawn from a specific technology library — it's not a generic "compile," because the output is constrained to the exact cells that exist in silicon for the target process. Yosys performs this translation in stages: parsing, generic technology-independent logic building, optimization passes, and finally technology mapping to real cells.

## Introduction to Yosys

Yosys is an open-source synthesis framework driven by a scripting interface — every synthesis run in this workshop is a short sequence of Yosys commands rather than a GUI flow. The three commands used constantly are:

- `read_verilog` — parse RTL into Yosys's internal representation
- `synth -top <module>` — run the built-in generic synthesis script against a chosen top module
- `show` — render the current internal netlist as a schematic

## Understanding `.lib` Files

A `.lib` (Liberty format) file is the technology library — it lists every standard cell available in a given process, along with its logical function, timing arcs (delay as a function of input slew and output load), area, and power characteristics. Synthesis without a `.lib` file produces only generic, technology-independent gates; mapping to real, fabricable hardware requires this file.

## Faster and Slower Cell Flavors

Most standard-cell libraries ship the *same* logical cell (say, a 2-input NAND) in multiple drive-strength variants — a small, low-drive version and a large, high-drive version of identical function. The small version is cheaper in area and power but drives its output more slowly (worse for heavily loaded or timing-critical nets); the large version is the opposite trade. This is why a `.lib` file contains dozens of entries for what looks, functionally, like the same handful of gates.

## Cell Selection Based on Design Requirements

Technology mapping (the `abc` step) doesn't pick cells at random — it chooses among the available flavors to meet timing constraints while minimizing area and power where slack allows. A net on a critical timing path is more likely to get mapped to a higher-drive cell; a net with generous slack is mapped to the cheapest cell that still meets its requirements.

## Yosys Synthesis Flow (End to End)

```bash
yosys
read_liberty -lib sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog mux_2to1.v
synth -top mux_2to1
abc -liberty sky130_fd_sc_hd__tt_025C_1v80.lib
show
```

Each command has a distinct job: `read_liberty` loads the target cell library, `read_verilog` loads the design, `synth` performs generic optimization and elaboration, and `abc` performs the actual technology mapping against the loaded library.

## Synthesis Statistics

After synthesis, Yosys's `stat` command reports cell counts by type — how many of each standard cell the design used, plus a total area estimate if the `.lib` file includes area data. This is the first quantitative signal of whether a design is efficient: two RTL descriptions of the same function can produce very different cell counts depending on coding style.

## Gate-Level Representation and Netlist

The final output is a **netlist** — a textual (or graphical, via `show`) description of the design purely in terms of instantiated standard cells and the wires connecting them, with all Verilog-level abstraction (behavioral `always` blocks, `assign` statements) gone. `write_verilog` dumps this netlist back out as structural Verilog, which is what actually gets carried forward to gate-level simulation and physical design.

---

# 🟢 Day 2 — Timing Libraries, Synthesis Methods & Flip-Flop RTL

## The SKY130 Technology Library

SKY130 is an open-source 130nm process design kit, and its standard-cell `.lib` files are what every synthesis run in this workshop targets. Using a real, public PDK (rather than a generic/abstract cell set) means the area, power, and timing numbers Yosys reports correspond to an actual fabricable process rather than illustrative placeholders.

## `.lib` Timing Libraries in Detail

Beyond just listing available cells, a `.lib` file encodes **timing arcs** as lookup tables — propagation delay and output transition time as functions of input slew and output capacitive load. This is what lets a static timing analyzer (downstream of synthesis) compute whether a design meets its clock frequency target, without ever running a transistor-level simulation.

## Process, Voltage, and Temperature (PVT) Conditions

The same physical cell behaves differently depending on manufacturing process variation, supply voltage, and operating temperature — collectively "PVT." A `.lib` filename like `sky130_fd_sc_hd__tt_025C_1v80.lib` encodes a specific corner: `tt` (typical process), `025C` (25°C), `1v80` (1.8V supply). Real designs are synthesized and checked across multiple corners (worst-case slow, best-case fast) because a design that meets timing at one corner may fail at another.

## Hierarchical Synthesis

Hierarchical synthesis preserves a design's module boundaries in the output netlist — each Verilog submodule remains a distinct instantiated block. This keeps the netlist's structure readable and mirrors the RTL's organization, but it can block certain cross-module optimizations, since Yosys won't restructure logic across a hierarchy boundary by default.

## Flattened Synthesis

`flatten` collapses all module hierarchy into one single top-level netlist before optimization. This exposes the entire design to Yosys's optimization passes simultaneously, sometimes producing a smaller or faster netlist than hierarchical synthesis — at the cost of a netlist that no longer maps cleanly back to the original module structure, which complicates debugging and incremental re-synthesis.

## Hierarchical vs. Flattened — the Trade-off

| | Hierarchical | Flattened |
|---|---|---|
| Netlist readability | Preserves module boundaries | Single flat block |
| Cross-module optimization | Limited | Full |
| Debug traceability | Easy to map back to RTL | Harder |
| Typical use | Large designs, incremental flows | Small blocks, maximum optimization |

## Asynchronous Reset D Flip-Flop

```verilog
module dff_async_reset (
    input  clk,
    input  reset,
    input  d,
    output reg q
);
always @(posedge clk, posedge reset) begin
    if (reset)
        q <= 1'b0;
    else
        q <= d;
end
endmodule
```

Because `reset` sits in the sensitivity list alongside `clk`, the flip-flop responds to reset **immediately**, independent of any clock edge — the moment `reset` rises, `q` is forced to 0 without waiting for `clk`.

## Asynchronous Set D Flip-Flop

```verilog
module dff_async_set (
    input  clk,
    input  set,
    input  d,
    output reg q
);
always @(posedge clk, posedge set) begin
    if (set)
        q <= 1'b1;
    else
        q <= d;
end
endmodule
```

Structurally identical to the async-reset case, but forcing `q` to `1` instead of `0`. Both async variants map to distinct SKY130 flip-flop cells (`dfrtp`/`dfstp`-style primitives) that have a dedicated set/reset pin — this is a different physical cell, not the same cell with extra logic in front of it.

## Synchronous Reset D Flip-Flop

```verilog
module dff_sync_reset (
    input  clk,
    input  reset,
    input  d,
    output reg q
);
always @(posedge clk) begin
    if (reset)
        q <= 1'b0;
    else
        q <= d;
end
endmodule
```

Here `reset` only appears *inside* the clocked block, not the sensitivity list — the flip-flop only samples reset at a clock edge, so a reset pulse shorter than one clock period can be missed entirely. This is the key behavioral difference from the async versions, and it shows up as a different cell choice at synthesis (a plain D flip-flop with reset logic built from an input-side mux, rather than a dedicated async-reset cell).

## Icarus Verilog Simulation and GTKWave for Sequential Logic

The same `iverilog`/`gtkwave` flow from Day 1 applies, but reading the waveform now requires tracking clock edges explicitly: `q` should only ever change value immediately after a rising `clk` edge (synchronous case) or immediately upon `reset`/`set` toggling (asynchronous case) — any other transition timing signals a coding error.

## Yosys Synthesis for Sequential Designs

```bash
yosys
read_liberty -lib sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog dff_async_reset.v
synth -top dff_async_reset
dfflibmap -liberty sky130_fd_sc_hd__tt_025C_1v80.lib
abc -liberty sky130_fd_sc_hd__tt_025C_1v80.lib
show
```

## `dfflibmap`

`dfflibmap` is a Yosys command specific to sequential synthesis — it maps Yosys's generic internal flip-flop cells to the *actual* flip-flop primitives available in the loaded `.lib` file (async reset, async set, sync reset, no reset, etc., each a genuinely different cell). Without this step, `abc` (which is primarily a combinational-logic mapper) has no mechanism to select among sequential cell variants.

## Technology Mapping via `abc`

`abc` performs the final combinational technology mapping — converting Yosys's generic AND/OR/NOT-level logic into the specific standard cells from the `.lib` file, selecting drive strengths to balance area, power, and timing as described in Day 1.

---

# 🟢 Day 3 — RTL & Logic Optimization

## RTL Optimization vs. Logic Optimization

**RTL optimization** happens at the level of the Verilog description — rewriting expressions, removing redundant assignments, restructuring `always` blocks before elaboration. **Logic optimization** happens after elaboration, at the level of the gate-level Boolean network — simplifying expressions, propagating constants, and removing dead logic once everything is already expressed as gates. Yosys performs both, but the logic-optimization passes are where most of the visible netlist shrinkage in this workshop actually happens.

## AND Logic

```verilog
module and_gate (input a, input b, output y);
assign y = a & b;
endmodule
```

Synthesizes directly to a single 2-input AND standard cell — the simplest possible target, useful as a baseline before looking at anything Yosys has to actually *work* to simplify.

## OR Logic

```verilog
module or_gate (input a, input b, output y);
assign y = a | b;
endmodule
```

Maps identically to a single 2-input OR cell — included specifically to confirm Yosys handles both basic gate types with equal directness, no unexpected extra logic on either.

## Three-Input AND Logic

```verilog
module and3_gate (input a, input b, input c, output y);
assign y = a & b & c;
endmodule
```

The interesting result here isn't the function — it's that Yosys maps this to a *single* 3-input AND cell rather than chaining two 2-input AND cells together. If a 3-input AND standard cell exists in the library, using it directly is strictly better (less delay, less area) than building the same function from smaller gates, and Yosys's technology mapping recognizes that.

## Constant Propagation

When a signal's value is known at compile/synthesis time — because it's tied to a constant, or because control logic forces it to a fixed value under all reachable conditions — Yosys substitutes that known value everywhere the signal is used and then deletes whatever logic becomes redundant as a result. This is one of the highest-value optimization passes because it can cascade: eliminating one signal often makes a second signal constant too, and so on.

## D Flip-Flop Optimization — Three Variants

**DFF Constant 1:**
```verilog
module dff_const1 (input clk, input reset, output reg q);
always @(posedge clk, posedge reset) begin
    if (reset) q <= 1'b0;
    else       q <= 1'b1;
end
endmodule
```
`q` genuinely depends on `reset` — it settles to a different constant depending on which branch executes — so synthesis has to preserve reset-handling logic even though both branches are "just" constants.

**DFF Constant 2:**
```verilog
module dff_const2 (input clk, input reset, output reg q);
always @(posedge clk, posedge reset) begin
    if (reset) q <= 1'b1;
    else       q <= 1'b1;
end
endmodule
```
Both branches drive `q` to the *same* constant — `reset` no longer has any influence on the output value at all. Constant propagation lets Yosys drop the reset-dependent logic entirely here, producing a visibly smaller netlist than DFF Constant 1 despite near-identical-looking RTL.

**DFF Constant 3:**
```verilog
module dff_const3 (input clk, input reset, output reg q);
always @(posedge clk) begin
    if (reset) q <= 1'b0;
    else       q <= 1'b1;
end
endmodule
```
`reset` is synchronous here (absent from the sensitivity list) — a structurally different design from the first two, included specifically to contrast how sync vs. async reset placement changes both simulated behavior and the resulting cell selection.

## Counter Optimization

```verilog
module counter_opt (input clk, input reset, output q);
reg [2:0] count;
assign q = count[0];
always @(posedge clk, posedge reset) begin
    if (reset) count <= 3'b000;
    else       count <= count + 1;
end
endmodule
```

Only `count[0]` is ever wired to an output — `count[1]` and `count[2]` influence nothing observable outside the module. Yosys's unused-logic elimination pass recognizes this and builds only **one** flip-flop instead of three, even though the RTL explicitly declares a 3-bit register. This is the clearest demonstration in the workshop that RTL bit-width and synthesized flip-flop count are not the same thing.

## Importance of Optimization

Unoptimized synthesis (a literal, unsimplified translation of RTL to gates) would carry forward every declared register and every written expression regardless of whether it affects any output — burning area, static power, and potentially timing margin on hardware that computes nothing observable. Optimization passes are what keep synthesized area/power proportional to what the design *actually does*, not to how it happens to be written.

## Comparing Optimized vs. Unoptimized Logic

Running `synth` with and without the later optimization sub-passes (or comparing `stat` output before/after `opt`) on the same design makes the effect directly measurable: the counter example above should report 3 flip-flops pre-optimization and 1 post-optimization if compared this way.

## Key Observations

- Structurally similar RTL (DFF Constant 1 vs. 2) can produce meaningfully different netlists depending on whether a value is *truly* constant across all paths.
- Declared register width is an upper bound on flip-flop count, not a guarantee — unused bits get eliminated.
- Synchronous vs. asynchronous reset placement changes both behavior and cell selection, not just style.

---

# 🟢 Day 4 — RTL to Gate-Level Simulation

## The RTL-to-Gate-Level Simulation Flow

Gate-level simulation (GLS) re-runs the **same testbench** against the synthesized netlist instead of the original RTL. The point is verification, not new functional testing: if RTL simulation and GLS produce different waveforms for identical stimulus, either the design has a coding-style bug that only manifests post-synthesis, or the synthesis/optimization flow introduced an unintended behavioral change.

```bash
iverilog -o gls_sim design_netlist.v sky130_primitives.v testbench.v
./gls_sim
gtkwave gls_dump.vcd
```

Note the extra file: gate-level simulation needs Verilog behavioral models of the standard cells themselves (a primitives library), since the netlist now instantiates SKY130 cells rather than behavioral Verilog.

## Ternary Operator MUX

```verilog
module mux_ternary (input a, input b, input sel, output y);
assign y = sel ? b : a;
endmodule
```

The continuous-assignment ternary form is the *recommended* way to write a simple mux precisely because it can't accidentally omit a case — every value of `sel` is covered by construction, with no possibility of the incomplete-branch problem explored in Day 5.

## MUX Working Principle

A multiplexer selects exactly one of several data inputs to route to a single output, based on a separate select signal — the fundamental building block for resource sharing (routing one shared datapath to multiple possible sources or destinations) throughout digital design.

## RTL Simulation (Baseline)

Before touching gate-level netlists, the RTL itself is simulated and its waveform recorded as the reference/expected behavior that GLS will later be checked against.

## Synthesis (Producing the Comparison Target)

Standard `read_verilog` → `synth` → `abc` → `write_verilog` flow, producing a structural netlist file that GLS will simulate in place of the RTL.

## Gate-Level Simulation (The Comparison Itself)

Running the identical testbench against the netlist and overlaying (or side-by-side comparing) the resulting waveform against the RTL simulation's waveform is the actual verification step — any divergence needs to be root-caused before the design can be trusted.

## Incomplete Sensitivity Lists — the "Bad MUX"

```verilog
module bad_mux (input a, input b, input sel, output reg y);
always @(sel) begin
    if (sel) y = b;
    else     y = a;
end
endmodule
```

The sensitivity list only includes `sel` — a change in `a` or `b` while `sel` stays constant produces **no re-evaluation** in RTL simulation, so `y` appears to freeze even though the "hardware intent" was a simple combinational mux. This is a simulation-only artifact: real combinational hardware responds to any input change immediately, so a synthesized netlist of this same code will *not* reproduce the freeze — meaning RTL simulation and gate-level simulation diverge, and the RTL sim was the misleading one.

## Correct Coding Using `always @(*)`

```verilog
module good_mux (input a, input b, input sel, output reg y);
always @(*) begin
    if (sel) y = b;
    else     y = a;
end
endmodule
```

`@(*)` auto-infers the full sensitivity list from every signal read inside the block, eliminating the entire class of bug above. This is why `always @(*)` is the standard recommendation for combinational blocks over manually listing signals.

## Blocking Assignment Caveat

Blocking assignments (`=`) execute sequentially and immediately within a procedural block — each statement's effect is visible to the *next* statement in the same block right away. In a combinational block this is usually fine and intended; in a *sequential* (clocked) block, using blocking assignments for values that should represent "next state after this clock edge" can produce simulation results that depend on statement order in ways real flip-flop hardware never would.

## Blocking Assignments

`=` — evaluates and assigns immediately, in program order, within the current time step. Appropriate for combinational logic where each line genuinely depends on the previous line's already-updated value.

## Non-Blocking Assignments

`<=` — evaluates the right-hand side using values from *before* the current time step, and schedules the assignment to happen only after all non-blocking assignments in the current time step have been evaluated. This models real flip-flop behavior correctly: every flip-flop in a clocked block samples its input at the same instant, regardless of the order statements happen to appear in the Verilog.

## Blocking vs. Non-Blocking — Practical Rule

| Block Type | Correct Assignment | Why |
|---|---|---|
| Combinational (`always @(*)`) | Blocking (`=`) | Order-dependent evaluation matches intended immediate logic |
| Sequential (`always @(posedge clk)`) | Non-blocking (`<=`) | Models simultaneous flip-flop sampling correctly |

Mixing these up in a sequential block is one of the most common sources of a genuine simulation-synthesis mismatch, because a simulator will execute blocking assignments in the written order (producing a result that depends on that order) while synthesized flip-flops have no such ordering dependency at all.

## Simulation-Synthesis Mismatch

A mismatch is any case where RTL simulation and gate-level simulation disagree on a design's behavior for the same stimulus. Root causes fall into two categories: (1) an RTL coding issue that simulates one way but synthesizes to different real hardware (incomplete sensitivity lists, misused blocking assignments), or (2) a genuine synthesis tool bug — rare, but the reason GLS exists as a mandatory check rather than an optional one.

## RTL Simulation vs. Gate-Level Simulation — Summary

| | RTL Simulation | Gate-Level Simulation |
|---|---|---|
| What's simulated | Behavioral Verilog | Structural netlist of real cells |
| Speed | Fast | Slower (more granular model) |
| Purpose | Functional verification | Confirms synthesis preserved behavior |
| Needs cell models? | No | Yes (primitives library) |

## Key Observations and Learning Outcomes

- `always @(*)` should be the default for combinational blocks — manual sensitivity lists are a recurring, avoidable bug source.
- Blocking vs. non-blocking assignment choice is not a style preference — using the wrong one in a sequential block changes simulated behavior.
- Gate-level simulation is the actual proof that synthesis didn't change design behavior; RTL simulation alone never proves this.

---

# 🟢 Day 5 — RTL Coding Styles & Looping Constructs

## IF-ELSE and CASE Statement Coding Styles

Both constructs describe conditional logic, but they carry different risks: an `if` chain implies priority (the first matching condition wins, which can synthesize to a chain of muxes), while a `case` statement implies parallel, non-prioritized selection among the listed values — closer to a direct decoder in hardware. Neither construct is inherently safer than the other with respect to the central risk of this day: **incomplete coverage**.

## Inferred Latches — the Core Mechanism

A synthesis tool cannot leave a signal's value undefined for some input combination — Verilog semantics require it to hold its *previous* value in that case, in both simulation and hardware. The only way to build "hold the previous value" in physical logic is a **latch** (level-sensitive storage). So any combinational block where some reachable condition leaves an output unassigned will synthesize a latch for that output — this is not a synthesis tool bug, it's the only behavior consistent with the RTL as written.

## Labs 1–2: Incomplete IF Statements

```verilog
module incomplete_if (input a, input b, input sel, output reg y);
always @(*) begin
    if (sel)
        y = a;
    // no else — y is unassigned when sel == 0
end
endmodule
```

With `sel` low, `y` has no assignment in this pass through the block — Verilog holds its previous value, and Yosys synthesizes a D-latch (transparent while `sel` is high, holding while `sel` is low) to implement exactly that. The waveform signature to recognize: `y` tracks `a` while `sel` is asserted, then visibly *freezes* rather than changing when `sel` drops — even if `a` continues to change.

## Labs 3–5: CASE Statements

```verilog
module incomplete_case (input i0, input i1, input [1:0] sel, output reg y);
always @(*) begin
    case (sel)
        2'b00: y = i0;
        2'b01: y = i1;
        // 2'b10 and 2'b11 unhandled
    endcase
end
endmodule
```

Identical failure mode to the IF case, different syntax — two of the four possible `sel` values leave `y` unassigned, so a latch is inferred. The fix is a `default` branch covering every otherwise-unlisted value:

```verilog
module complete_case (input i0, input i1, input i2, input [1:0] sel, output reg y);
always @(*) begin
    case (sel)
        2'b00:   y = i0;
        2'b01:   y = i1;
        default: y = i2;
    endcase
end
endmodule
```

One `default` line closes every remaining gap at once, and the synthesized result is pure combinational logic with no latch primitive anywhere in the netlist.

## Lab 6: Overlapping CASE Statements

```verilog
module overlap_case (input a, input b, input [1:0] sel, output reg y);
always @(*) begin
    case (sel)
        2'b00: y = a;
        2'b0?: y = b;   // overlaps with 2'b00
        default: y = 0;
    endcase
end
endmodule
```

When two `case` items can match the same actual value (here, `2'b0?` overlaps `2'b00` under `casez`-style wildcard matching), simulators resolve the ambiguity by picking the **first** matching branch in source order — but this priority behavior isn't guaranteed to be what a reader expects from a construct that's supposed to imply parallel selection, making overlapping case items a readability and maintainability hazard even when they don't cause an outright latch.

## Redundancy Optimization During Synthesis

Beyond latch inference, Yosys's optimization passes also collapse `case`/`if` structures that describe genuinely redundant logic — for example, two branches that happen to compute the same expression get merged rather than synthesized as separate parallel paths, shrinking the netlist without any change in RTL required.

## Looping Constructs in Verilog

`generate`/`for` loops let a fixed, compile-time-known structure be described once and replicated, rather than writing out every instance by hand. Critically, these loops are fully unrolled at elaboration time — the loop itself never exists in synthesized hardware, only its expanded result does.

## Loop-Based MUX

```verilog
module mux_loop #(parameter N = 4) (
    input  [N-1:0] data_in,
    input  [1:0]   sel,
    output reg     y
);
integer i;
always @(*) begin
    y = 1'b0;
    for (i = 0; i < N; i = i + 1)
        if (sel == i) y = data_in[i];
end
endmodule
```

The initial `y = 1'b0` before the loop is what keeps this block fully specified — every reachable path through the loop either matches one `sel` value and overrides `y`, or falls through to the default assigned before the loop even started, so no latch is inferred despite the loop never containing an explicit `else`.

## Loop-Based DEMUX

```verilog
module demux_loop #(parameter N = 4) (
    input            data_in,
    input  [1:0]     sel,
    output reg [N-1:0] y
);
integer i;
always @(*) begin
    y = {N{1'b0}};
    for (i = 0; i < N; i = i + 1)
        if (sel == i) y[i] = data_in;
end
endmodule
```

Same defensive pattern as the mux above — a default assignment to the *entire* output vector before the loop runs, so every bit of `y` is guaranteed a value on every pass regardless of `sel`.

## Ripple Carry Adder (RCA)

```verilog
module full_adder (input a, input b, input cin, output sum, output cout);
assign sum  = a ^ b ^ cin;
assign cout = (a & b) | (b & cin) | (a & cin);
endmodule

module rca_8bit (input [7:0] a, input [7:0] b, input cin,
                  output [7:0] sum, output cout);
wire [7:0] carry;
genvar i;
generate
    for (i = 0; i < 8; i = i + 1) begin : stage
        if (i == 0)
            full_adder fa (.a(a[i]), .b(b[i]), .cin(cin), .sum(sum[i]), .cout(carry[i]));
        else
            full_adder fa (.a(a[i]), .b(b[i]), .cin(carry[i-1]), .sum(sum[i]), .cout(carry[i]));
    end
endgenerate
assign cout = carry[7];
endmodule
```

Eight `full_adder` instances are generated once via the loop rather than written out individually, with each stage's carry-out feeding the next stage's carry-in — the defining structural trait of a ripple-carry design, where total addition latency scales with bit-width because the final sum bit can't be correct until the carry has rippled all the way from bit 0.

## Overall Summary — Day 5

| Symptom | Cause | Fix |
|---|---|---|
| Output freezes for some input combination | Missing `else` / missing `case` branch | Cover every reachable condition explicitly |
| Latch persists despite a `default` branch | One specific *signal* left unassigned in one branch, even though the branch itself exists | Assign every output in every branch, not just every branch in general |
| Ambiguous priority on matching conditions | Overlapping `case` items | Ensure case items are mutually exclusive, or make priority intentional and documented |
| Loop-described logic still latch-free | Default assignment placed before the loop body | Keep using this pattern for any loop-based combinational block |

---

## 🛠 Tools & Technologies Used

- Verilog HDL
- Icarus Verilog
- GTKWave
- Yosys
- SKY130 Standard Cell Library
- Linux / Ubuntu
- Git & GitHub

---

## 🎯 Overall Learning Outcomes

- Writing and simulating Verilog RTL against self-built testbenches
- Reading waveforms as a direct check against intended RTL behavior
- Running full Yosys synthesis flows against a real, open-source PDK
- Distinguishing hierarchical from flattened synthesis and choosing between them deliberately
- Recognizing how reset style (sync vs. async) changes both behavior and synthesized cell choice
- Predicting where constant propagation and dead-logic elimination will shrink a netlist
- Performing gate-level simulation as the actual proof that synthesis preserved RTL behavior
- Diagnosing simulation-synthesis mismatches from incomplete sensitivity lists or blocking-assignment misuse
- Recognizing the exact RTL patterns that cause unwanted latch inference, and the general fix for each
- Using loop-based generate constructs to describe repetitive structural hardware (MUX, DEMUX, ripple-carry adders) correctly

---

## 👤 Author

**J KAVYA**
