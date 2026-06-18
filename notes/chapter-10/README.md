# Chapter 10 — Electrical Rule Checks & Antenna Rule Basics

> **Learning Objectives:** By the end of this chapter, you will be able to:
> - Explain what Electrical Rule Checks (ERC) are and why they are needed
> - Perform ERC during an LVS run using Calibre
> - Configure ERC output, select/deselect specific checks, and view results in RVE
> - Explain the antenna effect and why it matters during IC fabrication
> - Interpret the Net Area Ratio (NAR) and use `NET AREA RATIO` in an SVRF rule
> - Generate geometric output files (GDSII/OASIS) from a Calibre nmDRC run

---

## Table of Contents

1. [What Are Electrical Rule Checks (ERC)?](#1-what-are-electrical-rule-checks-erc)
2. [Calibre ERC Operations](#2-calibre-erc-operations)
3. [ERC Workflow — Step by Step](#3-erc-workflow--step-by-step)
4. [Task: Specify ERC Output Information](#4-task-specify-erc-output-information)
5. [Task: Enable Selected ERC Rule Checks](#5-task-enable-selected-erc-rule-checks)
6. [Task: Disable Selected ERC Rule Checks](#6-task-disable-selected-erc-rule-checks)
7. [Task: Limit ERC Result Count](#7-task-limit-erc-result-count)
8. [Task: Execute the ERC Job](#8-task-execute-the-erc-job)
9. [Task: View ERC Results](#9-task-view-erc-results)
10. [Generating Geometric Output From Calibre](#10-generating-geometric-output-from-calibre)
11. [What Is an Antenna?](#11-what-is-an-antenna)
12. [Antenna Rule Basics and the NAR](#12-antenna-rule-basics-and-the-nar)
13. [Antenna Rule Example](#13-antenna-rule-example)
14. [Additional Antenna Considerations](#14-additional-antenna-considerations)
15. [Chapter Summary](#chapter-summary)
16. [Quick Reference — SVRF Commands](#quick-reference--svrf-commands)

---

## 1. What Are Electrical Rule Checks (ERC)?

Standard **DRC** (Design Rule Checking) and **LVS** (Layout vs. Schematic) are the most common Calibre tasks — but sometimes you need to ask more specific electrical questions about your layout. For example:

- Which nets have a path to ground (including paths through devices like transistors)?
- Which nets do NOT have a path to either power or ground?
- Which PMOS devices are connected to VCC through the PSD layer?

These questions cannot be answered by geometric DRC alone because they require understanding how the layout is **electrically connected**. Checks that answer these kinds of questions are called **Electrical Rule Checks (ERC)**.

### Why ERC Requires LVS

ERC has two special requirements that set it apart from standard DRC:

1. **It requires extraction of layout connectivity** — Calibre must first figure out which shapes in the layout are connected to form nets.
2. **It must be able to trace connectivity through devices** — wires may connect to each other through a transistor or a resistor, not just by touching shapes.

Because of these requirements, **ERC checks must be executed as part of an LVS job** (not a standalone DRC job). The LVS engine already does connectivity extraction, so ERC piggybacks on that process.

![Electrical Rule Checks — ERC](../../resources/screenshots/chapter-10/ch10-diagram-06.png)

> 💡 **Think of it this way:** DRC asks "Do my shapes follow the rules?" — ERC asks "Is my circuit connected correctly?" They are complementary checks.

---

## 2. Calibre ERC Operations

Calibre provides **two ERC-specific operations** in SVRF:

| SVRF Operation | What It Does |
|----------------|-------------|
| `PATHCHK` | Reports nets in the layout that do (or do not) have a path to power, ground, or labeled nets — or any combination of these conditions |
| `DEVICE LAYER` | Derives a layer containing all device seed shapes from all device instances of a named type (e.g., all NMOS gate shapes) |

ERC operations produce **output layer data** — just like DRC results — which can then be highlighted in the layout viewer. ERC can also produce **text files** listing nets that satisfy a specified condition.

### Key SVRF Specification Statements for ERC

These three statements are the most commonly used when setting up an ERC job:

```
ERC RESULTS DATABASE  result_file
ERC SELECT CHECK      rule_check [rule_check ...]
ERC SUMMARY REPORT    report_file
```

| Statement | Purpose |
|-----------|---------|
| `ERC RESULTS DATABASE` | Specifies the file where ERC results will be written |
| `ERC SELECT CHECK` | Selects which specific ERC rule checks to execute |
| `ERC SUMMARY REPORT` | Specifies a file to receive a summary of the ERC run (optional) |

> 💡 **Note:** ERC operations produce output layer data similar to DRC — you can highlight ERC results in the layout viewer using RVE, exactly as you would for DRC errors.

---

## 3. ERC Workflow — Step by Step

To execute a complete ERC job, you typically perform these tasks **in order**:

1. **Specify output information** — Where should results be written?
2. **Enable selected ERC rule checks** — Which checks should run?
3. **Disable selected ERC rule checks** — Are there any checks to skip?
4. **Limit ERC result count** — How many results per check before moving on?
5. **Execute the ERC job** — Run Calibre LVS with ERC enabled
6. **View ERC results** — Inspect findings in RVE

Each of these steps is covered in the sections below.

---

## 4. Task: Specify ERC Output Information

Before running ERC, you must tell Calibre where to write the results. This is done with three SVRF statements:

```
ERC RESULTS DATABASE  result_file
ERC SUMMARY REPORT    report_file
LVS EXECUTE ERC      {YES | NO}
```

### Example:

```
ERC RESULTS DATABASE  lab5a.erc.results
ERC SUMMARY REPORT    lab5a.erc.summary
LVS EXECUTE ERC       YES
```

### In the Calibre Interactive GUI:

Navigate to the **ERC** panel in the left sidebar. You will find:

- **LVS Execute ERC** — set to `Yes`
- **ERC Results Database → File** — enter your results filename (e.g., `lab5a.erc.results`)
- **ERC Summary Report → File** — enter your summary filename (e.g., `lab5a.erc.summary`)

![Task: Specify ERC Output Information](../../resources/screenshots/chapter-10/ch10-diagram-01.png)

> 💡 **Tip:** `ERC SUMMARY REPORT` is optional but highly recommended. It gives you a human-readable run log with statistics about what was checked.

> ⚠️ **Important:** Refer to the *SVRF Manual* for the full list of available options for each of these statements — there are additional modifiers and keywords beyond what is shown here.

---

## 5. Task: Enable Selected ERC Rule Checks

By default in an LVS run, **no ERC rule checks are executed**. You must explicitly select which checks to run using `ERC SELECT CHECK`.

### SVRF Syntax:

```
ERC SELECT CHECK  rule_check [rule_check ...]
```

### Example:

```
ERC SELECT CHECK  erc_rule_metal1_no_power
```

This restricts ERC rule check execution to only the named rule(s).

### In the Calibre Interactive GUI:

1. In the **ERC** panel, find **ERC Check Selection**
2. The **Recipe** dropdown shows `Checks selected in the rules file`
3. Click the **Edit** button — this opens the **Recipe Editor**
4. In the Recipe Editor, you can browse all available checks and place checkmarks next to the ones you want to include

![Task: Enable Selected ERC Rule Checks](../../resources/screenshots/chapter-10/ch10-diagram-02.png)

> 📌 **Key Point:** Rule checks do NOT need to contain ERC operations (like `PATHCHK`) to be executed. Any rule check can be selected for ERC execution — including standard DRC rule checks. `ERC SELECT CHECK` simply builds a list of checks to run.

---

## 6. Task: Disable Selected ERC Rule Checks

Sometimes you want to start from a broad selection and then **remove specific checks**. This is done with `ERC UNSELECT CHECK`.

### SVRF Syntax:

```
ERC UNSELECT CHECK  rule_check [rule_check ...]
```

### Example:

```
ERC UNSELECT CHECK  erc_rule_metal2_no_power
```

### How SELECT and UNSELECT work together:

The workflow is:
1. `ERC SELECT CHECK` first **builds a list** of checks to execute
2. `ERC UNSELECT CHECK` then **removes specific checks** from that list

So if you select 10 checks and then unselect 2, you end up running 8.

### In the Calibre Interactive GUI:

The Recipe Editor (same as above) shows checks with green checkmarks (selected) and red X marks (unselected). You can toggle individual checks on or off.

![Task: Disable Selected ERC Rule Checks](../../resources/screenshots/chapter-10/ch10-diagram-03.png)

> 💡 **Tip:** The `rule_check` name can refer to either a **specific rule check name** or a **rule group name**. Using groups lets you enable/disable whole sets of related checks at once.

---

## 7. Task: Limit ERC Result Count

When a rule check finds many violations, Calibre can spend a lot of time and disk space recording them all. `ERC MAXIMUM RESULTS` lets you cap how many results are written per check.

### SVRF Syntax:

```
ERC MAXIMUM RESULTS  {count | ALL}
```

### Example:

```
ERC MAXIMUM RESULTS  50
```

### How it works:

- Once the **current rule check** has generated `count` results, Calibre stops collecting output for that check and moves to the next one
- Use `ALL` if you want no limit (collect every result)
- The **default result limit is 1000**

### In the Calibre Interactive GUI:

In the **ERC** panel, check the **ERC Maximum Results** checkbox, then:
- Set **Upper Limit** to `Count`
- Enter the number (e.g., `50`) in the **Count** field

![Task: Limit ERC Result Count](../../resources/screenshots/chapter-10/ch10-diagram-04.png)

> ⚠️ **Warning:** Setting the limit too low (e.g., 1) means you will miss most errors. Set a value that lets you understand the scope of the problem without overwhelming the results file.

---

## 8. Task: Execute the ERC Job

ERC is executed as part of an LVS run. There are two ways to invoke it:

### Command Line Syntax:

```bash
# Run LVS with SPICE extraction (which triggers ERC)
calibre -spice ... rule_file

# OR equivalently:
calibre -spice ... -lvs ... rule_file

# Example:
calibre -spice "alu.sp" -hier my_rules
```

### SVRF Statement:

```
LVS EXECUTE ERC  {YES | NO}
```

### Example:

```
LVS EXECUTE ERC  YES
```

> 📌 **Key Point:** ERC operations are executed **only during the extraction phase of LVS**. The `LVS EXECUTE ERC YES` statement is not mandatory — ERC checks will be executed by default once you have selected them. However, including it explicitly makes your intent clear.

### In the Calibre Interactive GUI:

Click **Run LVS** in the ERC panel (with `LVS Execute ERC` set to `Yes`).

![Task: Execute the ERC Job](../../resources/screenshots/chapter-10/ch10-diagram-05.png)

> 💡 **Tip:** Refer to the *Calibre Verification User's Manual* for the full list of command line options and additional run configurations.

---

## 9. Task: View ERC Results

After the ERC job completes, results are stored in the file you specified with `ERC RESULTS DATABASE`. Use **Calibre RVE** to inspect them.

### How to Access ERC Results in RVE:

1. In the RVE **Navigator** tab, look for the **ERC** section
2. Click **ERC Results** to open the results tab — this shows which checks fired and how many results each has
3. Click **ERC Summary** to open the summary text report

### What You See:

- **ERC Results tab** — lists checks with result counts (e.g., `Check erc_rule_metal1_no_power: 37 results`)
- **ERC Summary tab** — full text log with run statistics, layer data, parameter settings
- **Layout view** — clicking a result highlights the corresponding shapes in the layout, just like DRC results

![Task: View ERC Results](../../resources/screenshots/chapter-10/ch10-diagram-06.png)

> 💡 **Tip:** ERC results can be highlighted in the layout viewer exactly like any DRC results — the workflow is identical once you have the results database open in RVE.

---

## 10. Generating Geometric Output From Calibre

In addition to the standard ASCII results database, you can instruct Calibre to write DRC results to a specific file in **GDSII** or **OASIS** format. This is useful when you want to overlay Calibre results back onto a layout in a mask-data or post-processing workflow.

### SVRF Syntax:

```
DRC CHECK MAP  rule_check [GDSII|OASIS]  layer  datatype  filename
```

### Important Rules:

- Provide a `DRC CHECK MAP` statement **for every rule** if the `DRC RESULTS DATABASE` statement specifies GDSII or OASIS as the format for all DRC output
- If you only want individual checks written to separate files, you can use `DRC CHECK MAP` on a per-check basis
- Additional options are available — consult the *SVRF Manual*

![Generating Geometric Output From Calibre](../../resources/screenshots/chapter-10/ch10-diagram-07.png)

### Examples:

**Example 1: "Grow all metal1 by 0.1u and save the result as GDS data."**

```
Grow_Metal1 {@ Grow metal1 by 0.1u
    SIZE metal1 BY 0.1
}
DRC CHECK MAP  Grow_Metal1  GDSII  21  0  "./mask/grown_m1.gds"
```

**Example 2: "Output all metal shapes in net 'RESET' for a GDS plot."**

```
LAYER m_123  metal1 metal2 metal3   //Define layer set "m_123"
Plot_RESET {@ Output all metal shapes belonging to RESET net
    NET m_123 RESET
}
DRC CHECK MAP  Plot_RESET  GDSII  1  0  "./plots/RESET.gds"
```

> 📌 **Key Point:** The `DRC CHECK MAP` statement maps the output of a named rule check to a specific GDS layer number and datatype, then writes it to a named file. This is how you create custom geometry outputs from Calibre.

---

## 11. What Is an Antenna?

The word "antenna" in IC design has nothing to do with radio antennas. It refers to a **fabrication-time charging problem** that can permanently damage transistor gates.

### How the Antenna Effect Happens:

During the IC fabrication process, layers of metal and polysilicon interconnect are deposited and etched one at a time. During these plasma etching steps:

1. **Metal and poly interconnect paths build up electrical charge** — they act like antennas collecting charge from the plasma
2. If enough charge accumulates, it needs somewhere to go
3. If the net connects to a **transistor gate**, the charge may arc from the poly layer, **through the thin gate oxide**, to the well below
4. This arcing **destroys or damages the gate** — the transistor may no longer function

> ⚠️ **Why it matters:** A damaged gate cannot be repaired after fabrication. The chip is simply broken. Antenna violations that go undetected cause **yield loss** — chips that come out of the fab dead.

![What Is an Antenna?](../../resources/screenshots/chapter-10/ch10-diagram-08.png)

---

## 12. Antenna Rule Basics and the NAR

### The Net Area Ratio (NAR)

To predict antenna damage risk, engineers use the **Net Area Ratio (NAR)** — the ratio of the antenna area (total interconnect area on the net) to the gate area (area of the transistor gate connected to the net):

```
NAR = Antenna Area / Gate Area
```

A **large NAR** means there is a lot of interconnect area relative to the small gate — more charge can accumulate, more risk of damage.

**Antenna rules flag nets whose NAR exceeds a specific threshold value** set by the foundry.

![Antenna Rule Basics](../../resources/screenshots/chapter-10/ch10-diagram-09.png)

### SVRF Syntax for Antenna Rules:

```
NET AREA RATIO  antenna_layers  gate_layer  >  ratio_limit
```

- `antenna_layers` — the interconnect layers to sum up as antenna area (e.g., metal1, metal2, poly)
- `gate_layer` — the layer that defines the gate area
- `ratio_limit` — the maximum allowed ratio (foundry-specified)

> 💡 **Reading the diagram:** The dashed box = antenna area; the gray box = gate area. The rule says: if (antenna area / gate area) exceeds the antenna ratio, flag it as a violation.

---

## 13. Antenna Rule Example

Here is a simple antenna rule that considers shapes on **three layers of interconnect** (poly, metal1, and metal2):

```
NET AREA RATIO  metal1  metal2  poly  gate  >  20
    RDB "antenna.rdb"  metal1  metal2  poly
```

### Breaking this down:

| Part | Meaning |
|------|---------|
| `NET AREA RATIO` | The SVRF operator that computes NAR |
| `metal1 metal2 poly` | The interconnect layers summed as antenna area |
| `gate` | The layer defining gate area |
| `> 20` | Flag if NAR exceeds 20 |
| `RDB "antenna.rdb"` | Write results to this file |
| `metal1 metal2 poly` (after RDB) | Output shapes from all three layers to the RDB |

> 📌 **Key Point about the `RDB` keyword:** If the `RDB` keyword is NOT used, only shapes from the **first listed layer** (metal1) are copied to the output. Specifying all three layers after `RDB` ensures you can see the full antenna path when you debug in RVE.

### Setting Up in Calibre Interactive:

When the rule file is loaded into Calibre Interactive, the side RDB (antenna.rdb) appears in the **Outputs panel → Side RDBs** section. You can configure RVE to automatically open this side RDB when the run completes.

![Antenna Rule Example](../../resources/screenshots/chapter-10/ch10-diagram-10.png)

### Viewing Antenna Results in RVE:

![Antenna Rule Example — RVE Results](../../resources/screenshots/chapter-10/ch10-diagram-11.png)

When you open the antenna RDB in RVE:

- **Two antennas were found**, each represented by a **cluster of results**
- The cluster contains multiple result shapes (the interconnect segments forming the antenna)
- **Selecting the cluster name** shows cluster details in the bottom panel — including the computed NAR value
- Clicking the **highlight button** (☀ icon) highlights the **entire cluster** (the full antenna path) in the layout
- The **computed Net Area Ratio value** is displayed in the details panel

---

## 14. Additional Antenna Considerations

Antenna analysis is not static — as more metal layers are added during fabrication, the antenna situation changes:

![Additional Antenna Considerations](../../resources/screenshots/chapter-10/ch10-diagram-12.png)

### How Antennas Change as Layers Are Added:

| Stage | Situation | Antenna Count |
|-------|-----------|--------------|
| **Add metal1 only** | Three separate metal1 segments (MET1a, MET1b, MET1c), each connected to only 2 gates | 3 antennas, 2 gates each |
| **Add metal2** | Metal2 connects some of the metal1 segments together, merging nets | 2 antennas: one with 2 gates, one with 4 gates |
| **Add metal3** | Metal3 connects everything into a single net | 1 antenna with 6 gates |

### What This Tells Us:

- **Adding metal layers can merge separate antennas into one** — reducing the number of antenna violations
- **The gate count changes** as more devices get connected through higher metal layers
- **The NAR changes** at each step — a net that violated the antenna rule after metal1 may pass after metal2, or vice versa
- Foundries specify antenna rules **per layer** to account for this layer-by-layer fabrication sequence

> ⚠️ **Key Insight:** Antenna checking must account for the incremental nature of fabrication. As each layer is added, the antenna area and gate count both evolve. Calibre's antenna checks are designed to evaluate this correctly using the `NET AREA RATIO` operator.

---

## Chapter Summary

| Concept | Key Takeaway |
|---------|-------------|
| **ERC** | Electrical Rule Checks — answer connectivity questions requiring trace-through-device analysis |
| **ERC requires LVS** | Must run as part of an LVS job because it needs connectivity extraction |
| `PATHCHK` | ERC operation: reports nets with/without a path to power or ground |
| `DEVICE LAYER` | ERC operation: derives a layer of device seed shapes by type |
| `ERC RESULTS DATABASE` | Specifies the output file for ERC results |
| `ERC SELECT CHECK` | Enables specific rule checks for ERC execution |
| `ERC UNSELECT CHECK` | Removes specific checks from the ERC execution list |
| `ERC MAXIMUM RESULTS` | Caps the number of results per check (default: 1000) |
| `LVS EXECUTE ERC YES` | Triggers ERC during the LVS extraction phase |
| **Antenna effect** | Charge buildup on interconnect during fabrication can destroy transistor gates |
| **NAR** | Net Area Ratio = antenna area / gate area — the key metric for antenna rules |
| `NET AREA RATIO` | SVRF operator that computes NAR and flags violations above a threshold |
| `DRC CHECK MAP` | Writes DRC/rule check results to GDSII or OASIS geometry files |

---

## Quick Reference — SVRF Commands

```
// ── ERC Output Setup ──────────────────────────────────────────────
ERC RESULTS DATABASE  result_file
ERC SUMMARY REPORT    report_file        // Optional
LVS EXECUTE ERC       {YES | NO}

// ── ERC Check Selection ──────────────────────────────────────────
ERC SELECT CHECK    rule_check [rule_check ...]
ERC UNSELECT CHECK  rule_check [rule_check ...]

// ── ERC Result Limiting ──────────────────────────────────────────
ERC MAXIMUM RESULTS  {count | ALL}      // Default = 1000

// ── Running ERC from Command Line ───────────────────────────────
calibre -spice "netlist.sp" -hier rule_file
calibre -spice ... -lvs ... rule_file

// ── Antenna Rule ─────────────────────────────────────────────────
NET AREA RATIO  metal1 metal2 poly  gate  >  20
    RDB "antenna.rdb"  metal1 metal2 poly

// ── Geometric Output (GDSII/OASIS from DRC check) ────────────────
DRC CHECK MAP  rule_check  [GDSII|OASIS]  layer  datatype  filename

// Example — save grown metal1 as GDS:
Grow_Metal1 {@ Grow metal1 by 0.1u
    SIZE metal1 BY 0.1
}
DRC CHECK MAP  Grow_Metal1  GDSII  21  0  "./mask/grown_m1.gds"
```

---

*Next Chapter: [Chapter 11 — Advanced DRC Topics](../chapter-11/)*
