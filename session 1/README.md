# From RTL to Gates: Blocking Assignments, Yosys Synthesis, and a RISC-V SoC Case Study
<p>
  <img src="https://img.shields.io/badge/Tool-Icarus%20Verilog-blue" alt="Icarus Verilog">
  <img src="https://img.shields.io/badge/Tool-GTKWave-orange" alt="GTKWave">
  <img src="https://img.shields.io/badge/Tool-Yosys-green" alt="Yosys">
  <img src="https://img.shields.io/badge/PDK-SKY130-red" alt="SKY130">
  <img src="https://img.shields.io/badge/Language-Verilog-9cf" alt="Verilog">
</p>

Digital design bugs rarely announce themselves. A module can simulate "almost right," pass a casual glance, and still be functionally wrong in a way that only shows up once you either look closely at a waveform or hand the design to a synthesis tool. This piece works through one such bug in a multiplexer, then uses the corrected design as a vehicle for understanding what Yosys actually does when it turns RTL into gates — including the sometimes-overlooked question of what happens when a design has multiple sub-modules. It closes with a look at how these same principles play out in a real (if small) RISC-V SoC.

## 1. Blocking vs. Non-Blocking Assignments: Why the Symbol Choice Matters

Verilog gives you two assignment operators inside procedural blocks — `=` (blocking) and `<=` (non-blocking) — and the language lets you use either one almost anywhere. That flexibility is exactly what makes this such a common source of bugs.

**Blocking assignment (`=`)** executes and completes before the next statement in the same block runs. If you write:

```verilog
a = b;
c = a;
```

`c` gets the *new* value of `a`, because the first line finished before the second began. This is ordinary, immediate, top-to-bottom execution — the same mental model as a line of C code.

**Non-blocking assignment (`<=`)** schedules the update to happen at the end of the current simulation time step, after every non-blocking assignment in that step has had its right-hand side evaluated. If you write:

```verilog
a <= b;
c <= a;
```

`c` gets the *old* value of `a` — the value `a` held before this time step began — because all the right-hand sides are evaluated using the pre-update values, and the actual writes happen simultaneously afterward.

That second behavior is precisely what you want for flip-flops: on a clock edge, every register should sample the *previous* state of every other register, not a value that another always block just updated in the same edge. That's how a shift register or a swap (`a <= b; b <= a;`) works correctly. But that same "wait and update later" behavior is wrong for combinational logic, where the output is supposed to be a pure, immediate function of the current inputs — no delay, no ordering dependency.

**The bug pattern.** A combinational block written with `<=` instead of `=` — like a multiplexer that uses non-blocking assignment inside an `always @(*)` block — will usually *simulate* something close to correct, because in an isolated testbench the delta-cycle delay is often invisible. The danger is twofold:

1. **Simulation-synthesis mismatch.** A synthesis tool builds a MUX from combinational gates regardless of which operator you used — gates have no concept of "the end of a time step." So the synthesized hardware behaves like true combinational logic, while your RTL simulation (if the delta-cycle timing happens to matter for a downstream comparison, race condition, or multi-driver scenario) may not match it exactly. You end up debugging a "gate-level netlist doesn't match RTL" mystery days or weeks later, far from the actual root cause.
2. **It signals sloppy intent.** Even when it happens not to cause an observable bug in a given testbench, using `<=` in combinational logic misrepresents what the block is doing, which makes the code harder to review and easier to break later when someone adds a register nearby.

**The industry-standard rule of thumb**, and the one nearly every lint tool and style guide enforces:
- Combinational logic (`always @(*)`) → blocking assignments (`=`)
- Sequential logic (`always @(posedge clk)`) → non-blocking assignments (`<=`)
- Never mix both operator types within the same always block

The fix for a bad combinational MUX is mechanical once you know the rule — swap every `<=` for `=` inside the affected block — but *finding* the bug is the harder skill. The tell is almost always a waveform where a signal appears to be responding one delta cycle (or, if unlucky, one full time step) late, even though the code "looks" instantaneous.
<img width="1148" height="1280" alt="image" src="https://github.com/user-attachments/assets/cb75110e-544c-4004-9a26-72f810e9303f" />


## 2. What Yosys Actually Does During Synthesis

`yosys` is an open-source logic synthesis framework, and the classic command sequence —

```
read_verilog design.v
synth -top design
abc -liberty <cell_library>.lib
show
write_verilog netlist.v
```

— is doing more than it looks like on the surface. Each step is worth unpacking:

- **`read_verilog`** parses the RTL into Yosys's internal representation (RTLIL). At this stage the design still looks conceptually like your source code — registers, always blocks, arithmetic operators.
- **`synth -top <module>`** runs Yosys's default synthesis script: it elaborates the hierarchy, infers memories and flip-flops from your always blocks, performs coarse-grain optimizations (constant folding, dead-code elimination, expression simplification), and eventually converts high-level RTL constructs into a **generic gate-level netlist** built from technology-independent primitives (`$_AND_`, `$_OR_`, `$_DFF_P_`, and similar internal cell types). At this point the design is structurally gates-and-flip-flops, but not yet tied to any real standard-cell library.
- **`abc -liberty <file>.lib`** is the technology-mapping step. ABC (A System for Sequential Synthesis and Verification) takes the generic netlist and maps it onto the actual cells available in the target `.lib` file — for the SKY130 open PDK, that means choosing from real standard cells like `sky130_fd_sc_hd__nand2_1`, `sky130_fd_sc_hd__dfxtp_1`, and so on, each with its own real timing and area characteristics. This is also where basic technology-aware optimization happens — picking the smallest or fastest equivalent cell for a given logic function.
- **`show`** renders a schematic view of the current netlist so you can visually confirm what got built — useful for sanity-checking that, say, a 2-to-1 MUX really synthesized to a MUX-shaped structure and not something pathological.
- **`write_verilog`** dumps the final gate-level netlist back out as Verilog, which is what a downstream tool (or a post-synthesis simulation) will consume.

**Why simulate at both stages.** Running the same testbench against the pre-synthesis RTL and again against the post-synthesis, `.lib`-mapped netlist (with SKY130 primitive models included) is a standard sign-off practice: it's the most direct way to catch synthesis-introduced discrepancies — including exactly the kind of blocking/non-blocking mismatch discussed above — before the design goes any further in the flow.
<img width="720" height="1280" alt="image" src="https://github.com/user-attachments/assets/90484147-8ec9-4e9c-9370-354a107ee082" />


## 3. Sequential Logic: How a Counter Actually Gets Built

A simple free-running counter is a useful second example because, unlike a MUX, it forces Yosys to infer state elements. Each bit of an n-bit counter typically becomes:

- **One flip-flop per bit** — for an active-high, active-high-reset design, this usually maps to something like a `$_DFF_PP0_` primitive at the generic stage, and then a real sequential cell (e.g., `sky130_fd_sc_hd__dfxtp_1` or a reset-capable variant) after technology mapping.
- **Independent next-state logic per bit** — rather than one monolithic "increment by one" adder, small counters frequently synthesize into a scattered mesh of simple gates (NOR/NAND-based logic implementing each bit's XOR-with-carry relationship), because for a 2-bit counter the optimizer finds it cheaper to hand-roll the next-state equations than to instantiate a generic adder.

The broader lesson: **synthesis output rarely looks like a literal translation of your RTL structure.** You wrote "increment `cnt` by 1 every clock," but what comes out the other side is whatever minimal set of gates satisfies that truth table for the specific bit-width and library you gave it. Reading the actual generated netlist — not just trusting that "the RTL said increment, so there must be an adder in there" — is the only reliable way to know what you actually got.
<img width="1280" height="655" alt="image" src="https://github.com/user-attachments/assets/bf190757-7062-4340-82b2-53f46f2837d6" />
<img width="555" height="911" alt="image" src="https://github.com/user-attachments/assets/39cf32c5-8f60-45f1-8089-efbb0ab4fdbf" />

## 4. Hierarchical vs. Flattened Synthesis

When a design is built from multiple sub-modules, Yosys gives you a choice that matters more than it might first appear:

**Hierarchical synthesis** (the default after a plain `synth -top`) keeps each sub-module as a distinct, separately-visible block in the netlist and schematic. If `top_module` instantiates `sub1` and `sub2`, those instance boundaries — and their internal signal names, like `u1.internal_signal` — remain intact and traceable.

**Flattening** (via the `flatten` command) dissolves every sub-module instance into the parent's flat list of gates. The logical function doesn't change — two AND-OR sub-modules still synthesize to the same AND and OR gates whether hierarchy is kept or removed — but the *bookkeeping* does. After flattening, all that's left of the original module structure is `$scopeinfo` metadata; the gates themselves are addressed directly, with hierarchical instance names folded into flat signal names like `u1.a` sitting right next to the primitive that uses them.

**Why you'd choose one over the other:**

| Consideration | Hierarchical | Flattened |
|---|---|---|
| Debug traceability | Easy — you can find "the OR gate inside `sub_module2`" | Harder — everything is one flat gate soup |
| Global optimization | Limited — ABC can't optimize *across* module boundaries as aggressively | Better — the optimizer can merge/simplify logic that spans what used to be separate modules |
| Runtime on large designs | Faster to synthesize incrementally, and enables hierarchical timing closure | Can be slower for very large designs, since the whole netlist becomes one flat optimization problem |
| Typical real-world use | Preferred for block-level sign-off, reuse, and readability | Often used just before final technology mapping/optimization, or when cross-module optimization opportunities matter more than traceability |

In production ASIC flows, it's common to synthesize hierarchically for most of the design (to keep IP blocks reusable and debuggable) and flatten selectively only where cross-boundary optimization is worth the loss of structure.
<img width="1280" height="720" alt="image" src="https://github.com/user-attachments/assets/29edebc7-daae-4467-bfed-f48f7b6b2d73" />

## 5. Case Study: VSDBabySoC and the RVMYTH Core

VSDBabySoC is an open-source, small-scale educational SoC built around three main pieces:

- **RVMYTH** — a minimal RISC-V CPU core, notable for being authored in **TL-Verilog**, a higher-level HDL extension designed around "transaction-level" pipelining constructs, and then compiled down into ordinary synthesizable Verilog (`rvmyth_gen.v`) for the rest of the flow to consume.
- **An analog PLL model (`avsdpll.v`)** — generates the clock the digital core runs on, standing in for what would be a real analog PLL block on silicon.
- **An analog DAC model (`avsddac.v`)** — takes a digital output word from the core and produces an analog-equivalent signal, illustrating the digital/analog boundary that a real mixed-signal chip has to cross.

This kind of design is a good teaching vehicle precisely because it's small enough to read end-to-end in an afternoon, yet it touches on real SoC concerns: clock generation and gating, a real (if tiny) instruction-executing CPU core, and a digital-to-analog interface — the same categories of block you'd find, just far more complex, in a production chip.

**RVMYTH's instruction pattern.** Small RISC-V teaching cores like this one are typically hand-built (or TL-Verilog-generated) around a short, deliberately simple instruction loop — arithmetic instructions like `ADDI`/`ADD`/`SUB` updating a register, and a branch instruction (`BNE`/`BEQ`) closing the loop — enough to demonstrate a real fetch-decode-execute cycle without the complexity of a full pipeline or instruction set.

**Why the pre/post-synthesis testbench split matters here too.** The same `PRE_SYNTH_SIM` / `POST_SYNTH_SIM` macro-switch pattern used for VSDBabySoC is the SoC-scale version of the RTL-vs-netlist comparison discussed above for the MUX and counter: define one macro and the testbench pulls in the original RTL sources (RVMYTH, PLL model, DAC model, exactly as written); define the other and it instead pulls in the synthesized gate-level netlist plus SKY130 primitive models. Running both and comparing waveforms is how you catch the exact class of bug this piece opened with — a synthesis/simulation mismatch — at full-chip scale rather than in a two-line MUX.
<img width="1280" height="720" alt="image" src="https://github.com/user-attachments/assets/83fde278-2444-4101-b161-4f2e2f11821c" />
<img width="1280" height="720" alt="image" src="https://github.com/user-attachments/assets/651436b3-8864-45bf-aa12-bcbcb6143a79" />

## 6. Practical Takeaways

- **Treat `<=` inside `always @(*)` as a bug until proven otherwise.** It's one of the single most common and most consequential Verilog style violations, and most lint tools flag it by default for exactly this reason.
- **Don't assume synthesis output mirrors your RTL's structure.** A counter doesn't necessarily contain "an adder"; a MUX doesn't necessarily contain "a MUX cell." Read the actual netlist, or use `show`, to check.
- **Understand what technology mapping (`abc -liberty ...`) is actually doing** — it's not a formality, it's the step where your generic gates become real, characterized silicon-ready cells with real area and timing.
- **Choose hierarchical vs. flattened synthesis deliberately, not by default.** Flattening trades away structural traceability for a chance at more aggressive cross-module optimization — worth it in some contexts, wasteful in others.
- **Simulate at both RTL and gate level whenever the flow supports it.** The pre-/post-synthesis macro-switch pattern used in VSDBabySoC is a lightweight, reusable way to get this comparison essentially for free once the testbench is written to support it.
- **Small educational SoCs like VSDBabySoC are worth building end-to-end** precisely because they compress the same categories of problem — clocking, core logic, analog interfacing, synthesis verification — that show up in much larger designs, into something you can fully understand in one sitting.
- #Author#: J.KAVYA, BTECH ECE
