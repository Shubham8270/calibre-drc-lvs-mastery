# Chapter 05 — Calibre nmLVS Process Flow

> **Learning Objectives:** By the end of this chapter, you will be able to:
> - Describe the Calibre nmLVS process flow
> - Execute a Calibre nmLVS job using the GUI and command line
> - Specify rule files, layout input, source information, and output settings
> - Review and interpret LVS reports using Calibre RVE
> - Understand hierarchical cell (H-Cell) matching and LVS BOX concepts

---

![Calibre nmLVS Process Flow — Chapter Title](ch-05-diagram-01.png)

---

## 1. What is Layout versus Schematic (LVS)?

**LVS (Layout vs. Schematic)** is one of the two core physical verification tasks in Calibre. Its job is to confirm that your physical layout on silicon *exactly matches* the circuit you originally designed.

Think of LVS as a **consistency check** between two representations of the same circuit:

| Representation | Tool Used | Description |
|---|---|---|
| **Layout (Physical)** | Calibre DESIGNrev / Virtuoso | The actual geometric shapes drawn on silicon layers |
| **Schematic (Logical)** | Design Architect / Composer | The circuit diagram showing components and their connections |

> **Key Principle:** If you drew a transistor in your schematic but missed it in the layout — or if you connected two nets incorrectly in the layout — LVS will catch it.

![What Is Layout versus Schematic?](ch-05-diagram-02.png)

### How LVS Works — Two Phases

**Phase 1: Extraction**
- Calibre reads the layout (GDS file) and performs **Connectivity Extraction** and **Device Extraction**
- It generates a **Layout Netlist** — a text description of what components and connections actually exist in your layout

**Phase 2: Comparison**
- Calibre takes the **Source Netlist** (from your schematic) and compares it against the Layout Netlist
- Mismatches → **LVS Errors**; perfect match → **LVS Clean**

---

## 2. Calibre nmLVS Layout Verification Process Flow

The full nmLVS process flow connects several components together:

![Calibre nmLVS Layout Verification Process Flow](ch-05-diagram-03.png)

### Inputs
- **Rule Files** — LVS rules provided by the foundry (PDK), telling Calibre how to recognize devices and connectivity
- **Layout** — Your GDS file containing the physical design

### What Calibre Does
- Runs **Layout Netlist Extraction** to pull out connectivity and device info from the layout
- Runs **Netlist Comparison** between the extracted layout netlist and the source netlist

### Outputs
- **Extraction Report** — Details of what Calibre found during extraction
- **Layout Netlist** — The extracted netlist in SPICE format
- **LVS Report** — Pass/Fail comparison results
- **svdb** (Standard Verification Database) — All files created during the LVS run, used by Calibre RVE

### Command Line Options

```bash
# Extract layout netlist only
calibre -spice layout_netlist.spi my_lvs_rules

# Run LVS comparison only
calibre -lvs -hier my_lvs_rules

# Extract AND compare in one step
calibre -spice layout_netlist.spi -lvs my_lvs_rules
```

> ⚠️ **Important:** If LVS finds errors, fix the layout and re-run until you achieve zero errors.

---

## 3. Executing a Calibre nmLVS Job

LVS is a **two-step process** that can be run separately or combined:

1. **Extract the layout netlist**
2. **Compare the extracted netlist with the source netlist**

![Executing a Calibre nmLVS Job](ch-05-diagram-05.png)

### Combined Extract/Compare Job — Key Tasks

When running a combined nmLVS job, you must:

1. Specify the **rule file**
2. Specify **input information** for layout and source
3. Specify **output information**
4. Specify the **name of the extracted netlist file**
5. Specify **nmLVS job options**
6. **Execute** the nmLVS job

### Example Job SVRF File (Most Basic Setup)

```
// Job and User settings SVRF file
// Most basic setup file for LVS run
// Date: 12/1/2021
LAYOUT PRIMARY lab1        // TOPCELL
LAYOUT SYSTEM GDSII        // SPICE
LAYOUT PATH "/home/student/work/lab1.gds"

SOURCE PRIMARY lab1
SOURCE SYSTEM SPICE
SOURCE PATH "/home/student/work/lab1.spi"

MASK SVDB DIRECTORY svdb QUERY
LVS REPORT lvs.report
LVS SUMMARY REPORT lvs.summary

LVS RECOGNIZE GATES ALL

INCLUDE "lvs.rules"
```

> 💡 **Tip:** This module walks through how to accomplish all the above tasks both via the **Calibre Interactive GUI** and directly in the **rulefile**.

---

## 4. Task: Specify the Rule File

The first step in setting up an LVS run is specifying the rule file.

### Launching Calibre Interactive for LVS

```bash
# From command line
calibre -gui -lvs

# Or from Calibre DESIGNrev Verification menu
```

![Task: Specify Rule File](ch-05-content-03.png)

### In the GUI

1. Open the **Rules** pane
2. Enter the rule file name (e.g., `lvs.rules`) or browse to it
3. Click **View** to open the rule file in a Notepad window for inspection
4. Click **Load** to load GUI fields from settings already defined in the rule file

> 💡 **GUI Equivalent:** Setting the Rules File in the GUI is equivalent to the `INCLUDE "lvs.rules"` statement in your SVRF file.

---

## 5. Task: Specify Layout Input Information

After setting the rule file, you configure where Calibre should find your layout.

![Task: Specify Layout Input Information](ch-05-content-04.png)

### In the GUI — Inputs Pane

1. Open the **Inputs** pane
2. Set the **Layout File** (e.g., `lab1.gds`)
3. Set the **Top Cell** name (e.g., `lab1`)
4. Set the **Layout Netlist** file name — this is where the extracted netlist will be saved (e.g., `lab1_layout.spi`)

### SVRF Equivalent

```
LAYOUT PRIMARY lab1
LAYOUT SYSTEM GDSII
LAYOUT PATH "/home/student/work/lab1.gds"
```

---

## 6. Task: Specify Source Information

The **source netlist** is the schematic-derived netlist that Calibre will compare against your layout.

![Task: Specify Source Information](ch-05-content-05.png)

### In the GUI — Inputs Pane (Source Path Section)

1. Open the **Inputs** pane → scroll to **Source Path**
2. Select **Source Format** (typically `SPICE`)
3. Enter the **SPICE File** name (e.g., `lab1_source.spi`)
4. Set the **Top Cell** name (must match the layout top cell)

> 💡 **Note:** Checking **"Export from source viewer"** instructs the schematic viewer to generate the source netlist automatically from the named schematic file — useful in Virtuoso flows.

### SVRF Equivalent

```
SOURCE PRIMARY lab1
SOURCE SYSTEM SPICE
SOURCE PATH "/home/student/work/lab1.spi"
```

---

## 7. Task: Specify Output Information

You need to tell Calibre where to save results so you can review them later.

![Task: Specify Output Information](ch-05-content-07.png)

### In the GUI — Outputs Pane

| Setting | What It Does |
|---|---|
| **Mask SVDB** checkbox | Creates the Standard Verification Database directory |
| **SVDB Directory** | Name of the directory (e.g., `svdb`) |
| **Generate cross-reference data for RVE** | Enables viewing results in Calibre RVE |
| **LVS Report** | Name of the LVS comparison report file (e.g., `lvs.report`) |
| **LVS Summary Report** | Name of the summary report file (e.g., `lvs.summary`) |
| **View LVS Report after run finishes** | Auto-opens the report when the job completes |

### SVRF Equivalent

```
MASK SVDB DIRECTORY svdb QUERY
LVS REPORT lvs.report
LVS SUMMARY REPORT lvs.summary
```

---

## 8. Task: Specify nmLVS Job Options

### Key LVS Options

```
LVS RECOGNIZE GATES ALL       # Recognize all standard gate types
LVS IGNORE PORTS YES          # Ignore port mismatches
LVS ISOLATE SHORTS NO         # Treat shorts normally
LVS INJECT LOGIC NO           # Do not inject logic optimization
```

> 💡 **Note:** `LVS RECOGNIZE GATES ALL` is commonly used. It allows Calibre to recognize standard gate-level logic cells (AND, OR, NAND, NOR, etc.) as hierarchical blocks instead of flattening them to transistor level during comparison.

---

## 9. Running Calibre nmLVS From Command Line

For automation or batch flows, you can run LVS entirely from the command line without the GUI.

![Running Calibre LVS From Command Line](ch-05-content-10.png)

### Steps

1. Create or edit your `lvs.rules` file (contains both SVRF settings and the INCLUDE for the foundry rules)
2. Run Calibre nmLVS:

```bash
calibre -spice layout.netlist -lvs -hier lvs.rules
```

3. Watch the **Terminal Window Transcript** — Calibre prints progress and final status here
4. When done, the transcript shows:
   - `LVS completed. INCORRECT.` (errors found) OR
   - `LVS completed. CORRECT.` (clean)

> 💡 **Tip:** The `LVS REPORT` statement produces both `lvs.rep` (comparison results) and `lvs.rep.ext` (extraction report).

---

## 10. Running Calibre nmLVS From the GUI

You can also run LVS directly from Calibre Interactive GUI.

![Running Calibre nmLVS From the GUI](ch-05-content-11.png)

### Steps

1. After filling in all GUI fields (Rules, Inputs, Outputs, Options)
2. Click **Run LVS** button
3. The **Transcript** panel at the bottom shows live output as Calibre runs
4. When complete, click **Show RVE** to open results

---

## 11. Task: Review the LVS Report

After running LVS, always check the report files to understand what happened.

![Task: Review the LVS Report](ch-05-content-15.png)

### Viewing the Report in the GUI

- In the **Outputs** pane, click the **View** button next to **LVS Report**
- The report opens in a new tab in the **Files** page of Calibre Interactive

### What the LVS Report Contains

The report shows the **OVERALL COMPARISON RESULTS** section, which indicates:

```
INCORRECT  ← LVS found mismatches

Error: Different numbers of nets (see below).
Error: Different numbers of instances (see below).

INITIAL NUMBERS OF OBJECTS
               Layout    Source    Component Type
Ports:              0         0
Nets:             334        11    *
Instances:          0         7    * MN (3 pins)
                    0         7    * MP (3 pins)
Total Inst:         0        14
```

> ⚠️ **Common Errors:** Different nets, different instances, or mismatched port connections are the most frequent LVS errors. Each must be investigated and fixed in the layout.

---

## 12. Common LVS RVE Tasks

After an LVS run, use **Calibre RVE** to visually locate and understand errors.

![Common LVS RVE Tasks](ch-05-content-20.png)

Common tasks you can do in LVS RVE:

- **Modify result viewing area content** — customize what is shown in the results panel
- **Cross-probe results** to layout viewer, schematic viewer, and netlists — clicking an error highlights it in your layout tool
- **Query layout and source entity information** — inspect individual nets, devices, and ports
- **Customize layer highlighting** — change colors to make errors more visible

---

## 13. Hierarchical LVS — H-Cells

### Why Hierarchical LVS?

When your design has repeated sub-blocks (like standard cells, macros, or IP blocks), it is more efficient to verify each block **once** and then reuse the result — instead of flattening everything to transistor level.

**Calibre nmLVS-H** (Hierarchical) supports this through **H-Cells** (Hierarchical Cells).

![Hierarchical LVS — Objectives](ch-05-content-25.png)

### Objectives for H-Cells

- **Automatic hierarchical cell matching by name** — Calibre can automatically match layout cells to source cells when names match
- **User-specified hierarchical cell names** — you provide an H-Cell file listing which cells to treat hierarchically
- **Specification of boxed cells** — using `LVS BOX` to treat certain cells as black boxes

---

## 14. Specifying H-Cell Names

### Automatic Name Matching

By default, Calibre checks if layout cell names and source cell names match. If **"Match cells by name"** is enabled in the H-Cells GUI pane, Calibre automatically creates the H-Cell list.

### User-Specified H-Cell File

You can also provide your own H-Cell file mapping layout cell names to source cell names:

```
// H-CELL FILE
// LAYOUT CELL    SOURCE CELL

a1220             NAND2_1
a1230             MUX2
a1240             NOR2_1
a1310             a1310
a1620             a1620
a1720             a1720
a2311             a2311
a33*              a3310
```

> 💡 **Note:** Wildcards (`*`) are supported in layout cell names. Calibre treats cell names as **case-insensitive** and will issue a warning for any cell names not found in the layout or source.

![Specifying H-Cell Names](ch-05-content-30.png)

### In the GUI — H-Cells Pane

1. Open the **H-Cells** pane
2. Check **H-Cells File** checkbox
3. Enter the H-Cells file name (e.g., `cell_file`)
4. Enable **"Use H-Cells file"** option

---

## 15. LVS BOX — Treating Cells as Black Boxes

### What is LVS BOX?

`LVS BOX` tells Calibre to treat a specific cell as a **black box** — Calibre does not look inside the cell to verify its content. Instead, it only checks that the external pins connect correctly.

### When to Use LVS BOX

- For **pre-verified IP blocks** that you don't have source netlists for
- For **analog blocks** where transistor-level LVS is not meaningful
- For cells where layout-to-source matching should happen only at the interface level

### LVS BOX Syntax

```
LVS BOX [BLACK | GRAY [DEVICES]] [LAYOUT [ONLY]] [[TRANSFORM] SOURCE]
         cellname ['('pin_swap_list')' …] …
```

![LVS BOX in Calibre Interactive GUI](ch-05-content-34.png)

### In the GUI — Options Pane

1. Open the **Options** pane → LVS Options
2. Find the **LVS Box** section
3. Add entries — specify `SOURCE` or `LAYOUT` or both for each cell:
   - `LVS Box  SOURCE s1220`
   - `LVS Box  LAYOUT a1220`

> 💡 **Tip:** The GUI provides a template to add or override `LVS BOX` statements from the rulefile.

---

## 16. Diagrams Reference

The following diagrams from this chapter illustrate key concepts:

| Diagram | Description |
|---|---|
| `ch-05-diagram-01.png` | Chapter Title — Calibre nmLVS Process Flow |
| `ch-05-diagram-02.png` | What Is Layout versus Schematic? |
| `ch-05-diagram-03.png` | Calibre nmLVS Layout Verification Process Flow |
| `ch-05-diagram-04.png` | LVS Interactive GUI Overview |
| `ch-05-diagram-05.png` | Executing a Calibre nmLVS Job |
| `ch-05-diagram-06.png` | LVS RVE Results Overview |
| `ch-05-diagram-07.png` | H-Cells Hierarchical Matching |
| `ch-05-diagram-08.png` | LVS BOX Black Box Concept |
| `ch-05-diagram-09.png` | LVS Summary Flow |

---

## Chapter Summary

| Concept | Key Takeaway |
|---|---|
| **LVS** | Compares physical layout against schematic to ensure they match electrically |
| **Two LVS Phases** | (1) Extract layout netlist, (2) Compare with source netlist |
| **SVRF Job File** | User-written file specifying layout path, source path, outputs, and options |
| **Calibre Interactive** | GUI for setting up and running LVS jobs without writing SVRF manually |
| **LVS Report** | Text file showing CORRECT or INCORRECT along with error details |
| **svdb** | Standard Verification Database — all output files from a Calibre nmLVS run |
| **Calibre RVE** | Tool to view LVS results and cross-probe to layout/schematic |
| **H-Cells** | Hierarchical cells — verified once and reused, improving run efficiency |
| **LVS BOX** | Treats a cell as a black box — only external pin connections are verified |
| **LVS RECOGNIZE GATES** | Lets Calibre recognize gate-level logic as hierarchical blocks |

---

*Previous Chapter: [Chapter 04 — Calibre nmDRC Advanced](../chapter-04/)*  
*Next Chapter: [Chapter 06 — Calibre nmLVS Advanced Topics](../chapter-06/)*
