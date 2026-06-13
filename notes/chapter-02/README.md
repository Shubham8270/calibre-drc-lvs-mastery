# Chapter 02 — Calibre nmDRC Basics

> **Course:** Using Calibre nmDRC/nmLVS (Siemens EDA)  
> **Level:** Beginner-friendly — everything explained from scratch  
> **Topics Covered:** Setting up an nmDRC run · DRC output results · Common DRC RVE tasks

---

## Table of Contents

1. [What is Calibre nmDRC?](#1-what-is-calibre-nmdrc)
2. [Chapter Overview — What You Will Learn](#2-chapter-overview--what-you-will-learn)
3. [Part 1 — Set Up an nmDRC Run](#part-1--set-up-an-nmdrc-run)
   - 3.1 [The Big Picture — nmDRC Layout Verification Process Flow](#31-the-big-picture--nmdrc-layout-verification-process-flow)
   - 3.2 [Executing a Calibre nmDRC Job](#32-executing-a-calibre-nmdrc-job)
   - 3.3 [Step 1 — Specify the Rule File](#33-step-1--specify-the-rule-file)
   - 3.4 [Step 2 — Specify Input Information](#34-step-2--specify-input-information)
   - 3.5 [Step 3 — Specify Output Information](#35-step-3--specify-output-information)
   - 3.6 [Step 4 — Specify nmDRC Options](#36-step-4--specify-nmdrc-options)
   - 3.7 [Step 5 — Include Another SVRF File](#37-step-5--include-another-svrf-file)
   - 3.8 [Step 6 — Execute the nmDRC Job](#38-step-6--execute-the-nmdrc-job)
4. [Part 2 — DRC Output Results](#part-2--drc-output-results)
   - 4.1 [Reviewing nmDRC Job Output — Overview](#41-reviewing-nmdrc-job-output--overview)
   - 4.2 [Review the Transcript](#42-review-the-transcript)
   - 4.3 [Review the DRC Summary Report](#43-review-the-drc-summary-report)
   - 4.4 [Invoke RVE to View DRC Results](#44-invoke-rve-to-view-drc-results)
5. [Part 3 — Common DRC RVE Tasks](#part-3--common-drc-rve-tasks)
   - 5.1 [Modify RVE Check/Cell Tree Display](#51-modify-rve-checkcell-tree-display)
   - 5.2 [Cross-Probe DRC Results to Layout Viewer](#52-cross-probe-drc-results-to-layout-viewer)
   - 5.3 [Displaying Errors in a Hierarchical Design](#53-displaying-errors-in-a-hierarchical-design)
   - 5.4 [Report Errors in Cell Space](#54-report-errors-in-cell-space)
   - 5.5 [What If the Error Is Still Reported at the Top Level?](#55-what-if-the-error-is-still-reported-at-the-top-level)
   - 5.6 [Set DRC Result Display Format](#56-set-drc-result-display-format)
   - 5.7 [Set DRC Result Status](#57-set-drc-result-status)
   - 5.8 [Comment Waived Results](#58-comment-waived-results)
   - 5.9 [View Waived Result Comments](#59-view-waived-result-comments)
6. [Key Terms Glossary](#6-key-terms-glossary)
7. [Quick Reference — The Job Settings SVRF File](#7-quick-reference--the-job-settings-svrf-file)

---

## 1. What is Calibre nmDRC?

When engineers design a microchip, they draw the physical shapes of transistors, wires, and insulating layers in a tool called a **layout editor**. These shapes must follow very strict manufacturing rules — for example, two metal wires must not be too close together, or they might short-circuit on the silicon wafer.

**Calibre nmDRC** (nanometer Design Rule Check) is a tool made by **Siemens EDA** (formerly Mentor Graphics) that automatically checks your layout against all these manufacturing rules and reports any violations. Think of it as a very smart spell-checker, but instead of grammar it checks physical geometry.

---

## 2. Chapter Overview — What You Will Learn

![Chapter 2 Title Slide](../screenshots/chapter-02/ch02-diagram-01.png)

This chapter is divided into three major sections, each with its own set of learning objectives:

### Part 1 — Set Up an nmDRC Run

![Objectives Part 1](../screenshots/chapter-02/ch02-diagram-02.png)

By the end of Part 1 you will be able to:

- **Specify rule files** — tell Calibre which design rules to check against
- **Specify input and output information** — tell Calibre where your layout is and where to write results
- **Specify nmDRC options** — fine-tune how the check runs

### Part 2 — DRC Output Results

![Objectives Part 2](../screenshots/chapter-02/ch02-diagram-05.png)

By the end of Part 2 you will be able to:

- **Review the run transcript** — read the real-time log Calibre produces while running
- **Review the DRC Summary Report** — read the human-friendly summary of all rule checks
- **View DRC results in RVE** — open the Results Viewing Environment to browse errors interactively

### Part 3 — Common DRC RVE Tasks

![Objectives Part 3](../screenshots/chapter-02/ch02-diagram-08.png)

By the end of Part 3 you will be able to:

- **Modify RVE check/cell tree display** — change how errors are grouped and sorted
- **Cross-probe results to layout viewer** — click an error in RVE and jump straight to that location in your layout
- **Modify result status** — mark errors as Fixed or Waived to track your progress

---

## Part 1 — Set Up an nmDRC Run

---

### 3.1 The Big Picture — nmDRC Layout Verification Process Flow

Before diving into individual steps, it helps to understand the full workflow.

![Calibre nmDRC Layout Verification Process Flow](../screenshots/chapter-02/ch02-diagram-10.png)

The diagram above shows the complete loop:

```
Layout (your design)  ──┐
                        ├──► Calibre nmDRC ──► DRC Results Database(s)
Rule File               ──┘         │                    │
                                    │                    ▼
                                    ▼         Locate errors using
                               DRC Report     Calibre RVE & Layout tool
                                                         │
                                                         ▼
                                              Fix layout errors ──► (back to Layout)
```

**Step-by-step explanation:**

1. **Layout** — Your chip design, stored as a file (commonly in GDSII format). The arrow looping back from "Fix layout errors" shows this is an iterative process — you keep fixing and re-checking until there are zero violations.

2. **Rule File** — A text file written in Calibre's **SVRF** (Standard Verification Rule Format) language. It contains every manufacturing rule your foundry requires. You do not write this yourself — it is provided by the foundry (e.g., TSMC, Samsung).

3. **Calibre nmDRC** — The engine that reads both inputs and runs the checks.

4. **DRC Report** — An optional text summary (shown with a dashed border because it is optional).

5. **DRC Results Database** — A binary or ASCII file containing the exact geometric locations of every violation found.

6. **Locate errors using Calibre RVE & Layout tool** — You open the results database in RVE (Results Viewing Environment) and cross-probe to your layout editor to see exactly where each error is.

7. **Fix layout errors** — You edit the layout to fix the violations, then re-run Calibre. Repeat until clean.

---

### 3.2 Executing a Calibre nmDRC Job

![Executing a Calibre nmDRC Job](../screenshots/chapter-02/ch02-diagram-03.png)

Running an nmDRC job always involves these five tasks, in order:

| # | Task | What it means |
|---|------|---------------|
| 1 | Specify rule file | Tell Calibre which `.rules` file contains the design rules |
| 2 | Specify input information | Tell Calibre where your layout file is and which cell is the top cell |
| 3 | Specify output information | Tell Calibre where to write the results database and summary report |
| 4 | Specify nmDRC job options | Set parameters like the maximum number of results to report |
| 5 | Execute nmDRC job | Actually launch the check |

#### Two Ways to Configure a Run

You have two equivalent methods:

**Method A — Calibre Interactive GUI**
A graphical window where you fill in fields and click buttons. Everything is visual and guided. This is what most of the slides demonstrate.

**Method B — Job Rule File (command line)**
You write all the settings directly into a text file called the **Job_rule file** (also called the **runset file**) using SVRF syntax, then launch Calibre from the terminal. This is useful for scripting and automation.

> 💡 **Both methods do exactly the same thing.** The GUI simply generates the SVRF commands behind the scenes. Throughout this chapter, each GUI step is shown alongside its equivalent SVRF command so you understand both.

#### The Basic SVRF Job Settings File

The code block on the right side of the slide above is the most minimal runset file you can write:

```svrf
// Job and User settings SVRF file
// Most basic setup file for DRC run
// Date: 12/1/2021
LAYOUT PRIMARY lab1    // TOPCELL
LAYOUT SYSTEM GDSII    // SPICE
LAYOUT PATH "/home/student/work/lab1.gds"
DRC RESULTS DATABASE lab1.drc.results ASCII
DRC SUMMARY REPORT lab1.drc.summary

INCLUDE "drc.rules"
INCLUDE "golden_rules"
```

Each line is explained in detail in the sections below.

---

### 3.3 Step 1 — Specify the Rule File

![Specify Rule File](../screenshots/chapter-02/ch02-diagram-11.png)

The **Rules pane** in Calibre Interactive is where you tell the tool which rule file to load.

#### How to do it in the GUI

1. Click **Rules** in the left navigation panel to open the Rules pane.
2. In the **Rules File** field, either:
   - Type the file name directly (e.g., `drc.rules`), **or**
   - Click the folder icon to browse to the file.
3. Click **Load** — this reads the rule file and populates other GUI fields (like layer names) from the rules.
4. Optionally click **View** to open the rule file in a Notepad window and inspect its contents.

The **Check Selection** area lets you choose a "Recipe" — a pre-defined subset of checks to run. By default it runs all checks selected inside the rule file itself.

#### Equivalent SVRF command

```svrf
INCLUDE "drc.rules"
```

The `INCLUDE` statement tells Calibre to read and execute the contents of the named file. This is how the rule file is connected to the job.

> 📌 **Key insight:** The GUI "Rules File" field is simply a friendly wrapper around the `INCLUDE` statement. When you type `drc.rules` in the GUI and click Run, Calibre internally generates `INCLUDE "drc.rules"`.

---

### 3.4 Step 2 — Specify Input Information

![Specify Input Information](../screenshots/chapter-02/ch02-diagram-12.png)

The **Inputs pane** is where you tell Calibre what layout to check.

#### How to do it in the GUI

1. Click **Inputs** in the left navigation panel.
2. Set **Layout Format** — almost always `GDSII` for standard IC layouts. Other options include OASIS.
3. Set **Layout File** — the path to your `.gds` file (e.g., `lab1.gds`).
4. Set **Top Cell** — the name of the top-level cell in your design hierarchy (e.g., `lab1`). This is the cell Calibre will start from when traversing your design.
5. The **Export from layout viewer** checkbox, when enabled, instructs the layout viewer (e.g., Virtuoso or Calibre DESIGNrev) to automatically write a fresh copy of the layout to disk before the DRC run begins. This ensures Calibre sees your latest edits.

#### What is a "Top Cell"?

IC layouts are hierarchical — a top-level cell contains subcells, which contain more subcells, like a tree. Calibre needs to know where to start. The **Top Cell** is the root of that tree — it represents your entire design.

#### Equivalent SVRF commands

```svrf
LAYOUT PRIMARY lab1    // TOPCELL  — the top cell name
LAYOUT SYSTEM GDSII    // the layout file format
LAYOUT PATH "/home/student/work/lab1.gds"  // the file path
```

- `LAYOUT PRIMARY` sets the top cell name.
- `LAYOUT SYSTEM` sets the format.
- `LAYOUT PATH` sets the file location.

---

### 3.5 Step 3 — Specify Output Information

![Specify Output Information](../screenshots/chapter-02/ch02-diagram-13.png)

The **Outputs pane** controls where Calibre writes its results.

#### Two output files are configured here

**A) DRC Results Database** — the main output, containing all violation geometries.

- **Format** — choose `ASCII` (human-readable text) or binary. ASCII is easier to inspect manually but larger.
- **File** — the output file name, e.g., `lab1.drc.results`.

**B) DRC Summary Report** — an optional but highly recommended text summary.

- Enable by checking **DRC Summary Report**.
- Set the file name, e.g., `lab1.drc.summary`.
- **Previous contents** set to `REPLACE` means each new run overwrites the old report.
- Checking **View DRC Summary Report after run finishes** automatically opens the report when the run completes.

#### Other notable options

- **Output cell errors in cell space** — controls where hierarchical errors are reported (explained in detail in Section 5.3).
- **Side RDBs** — create additional results databases for specific checks.
- **Create HTML Report** — generates a web-browser-friendly version of results.

#### Equivalent SVRF commands

```svrf
DRC RESULTS DATABASE lab1.drc.results ASCII
DRC SUMMARY REPORT lab1.drc.summary
```

---

### 3.6 Step 4 — Specify nmDRC Options

![Specify nmDRC Options](../screenshots/chapter-02/ch02-diagram-14.png)

The **Options pane** provides fine-grained control over how the DRC run behaves. You access it by:

1. Going to **Settings → Show Pages → Options** to make the Options button visible in the left panel (if it is not already showing).
2. Clicking **Options** in the left navigation panel.
3. Selecting the desired tab within the Options pane.

#### Example — DRC Maximum Results

A very common option to configure is **DRC Maximum Results**. This limits how many violations Calibre reports per rule check. The default is typically 1000.

**Why would you limit results?**
If a design has 500,000 violations of the same rule (e.g., a systematic spacing error across an entire array), writing all 500,000 to disk is wasteful. Setting a maximum of 1000 lets you see representative examples and fix the root cause without generating a huge file.

In the GUI: enable **DRC Maximum Results**, set **Upper Limit** to `Count`, and enter `1000` in **Maximum Result Count**.

#### Equivalent SVRF command

```svrf
DRC MAXIMUM RESULTS 1000
```

> ⚠️ **Important note:** Not every option available in SVRF can be set from the GUI. If you need a specific option that is not visible in the Options pane, consult the **SVRF Reference Manual** and add the command directly to your runset file.

---

### 3.7 Step 5 — Include Another SVRF File

![Include Another SVRF File](../screenshots/chapter-02/ch02-diagram-06.png)

Sometimes you need to include more than one rule file. For example, your foundry may provide a main `drc.rules` file plus a separate `golden_rules` file for additional checks.

#### How to do it in the GUI

1. Open the **Options** pane (left navigation panel).
2. Scroll down to find **Include Rules Files** and check the checkbox to enable it.
3. In the **Additional Rules Files** text box, type the name of the additional file (e.g., `golden_rules`), or browse to it.

#### Equivalent SVRF command

```svrf
INCLUDE "golden_rules"
```

You can have as many `INCLUDE` statements as you need. They are processed in order, top to bottom.

---

### 3.8 Step 6 — Execute the nmDRC Job

![Execute nmDRC Job](../screenshots/chapter-02/ch02-diagram-07.png)

Once all settings are configured, running the job is a single click.

#### How to do it in the GUI

Click the **Run DRC** button at the bottom of the left navigation panel.

After clicking:

- The **Transcript pane** opens automatically and shows the live log of the run as it progresses.
- When the run finishes, a summary line like `0 Errors, 1 Warning, 1 Info` appears below the transcript.
- Any warnings or informational messages appear in a table at the bottom (e.g., "Please increase descriptors limit for best performance").

#### What the transcript summary tells you at a glance

The end of the transcript shows key statistics:

```
--- TOTAL RULECHECKS EXECUTED = 11
--- TOTAL RESULTS GENERATED = 57 (230)
--- DRC RESULTS DATABASE FILE = lab1.drc.results (ASCII)
```

- `TOTAL RULECHECKS EXECUTED` — how many individual rules were checked.
- `TOTAL RESULTS GENERATED = 57 (230)` — 57 results are stored in the database; the number in parentheses (230) is the actual count before any maximum limit was applied.
- `DRC RESULTS DATABASE FILE` — confirms where the output was written.

---

## Part 2 — DRC Output Results

---

### 4.1 Reviewing nmDRC Job Output — Overview

![DRC Output Results Title](../screenshots/chapter-02/ch02-diagram-04.png)

![Reviewing nmDRC Job Output](../screenshots/chapter-02/ch02-diagram-06.png)

After the job finishes, there are three output review tasks:

| Task | Tool used | What you learn |
|------|-----------|----------------|
| Review the Transcript | Terminal or GUI Transcript pane | Real-time execution details, warnings, timing |
| Review the DRC Summary Report | Any text editor or GUI Files pane | Which rules were checked, how many violations each has |
| Invoke RVE to view DRC results | Calibre RVE | Interactive, graphical error browser with layout cross-probing |

---

### 4.2 Review the Transcript

![Review the Transcript](../screenshots/chapter-02/ch02-diagram-08.png)

The **transcript** is a detailed log of everything Calibre did during the run. It is your first place to look if something goes wrong.

#### Two places to read the transcript

**When run from command line** — the transcript prints directly to the terminal window.

**When run from GUI** — the transcript appears in the **Transcript pane** inside Calibre Interactive, scrollable and searchable.

Both show exactly the same content.

#### What to look for in the transcript

- **Layer processing messages** — shows each layer being read, processed, and deleted from memory.
  ```
  Layer active_CO::<1> DELETED -- LVHEAP = 1/3/4
  DRC RuleCheck active_CO COMPLETED. Number of Results = 21
  ```
- **RuleCheck completion lines** — one per rule, showing how many results were found.
- **EXPANDING messages** — indicates Calibre expanded (flattened) a cell for processing (important for understanding hierarchical error reporting — see Section 5.5).
- **Summary at the end:**
  ```
  --- TOTAL RULECHECKS EXECUTED = 2
  --- TOTAL RESULTS GENERATED = 23 (23)
  --- DRC RESULTS DATABASE FILE = demo_drc.db (ASCII)
  --- CALIBRE::DRC-H COMPLETED -- Wed Jan 19 13:04:20 2022
  --- PROCESSOR COUNT = 1
  ```
- **Warnings and Errors** — shown in the bottom table in the GUI (e.g., suggestions to increase memory limits).

> 💡 **Tip:** Search the transcript for `ERROR` or `WARNING` to quickly find important messages. In the GUI, use the **Search** pane.

---

### 4.3 Review the DRC Summary Report

![Review the DRC Summary Report](../screenshots/chapter-02/ch02-diagram-09.png)

The **DRC Summary Report** is an optional but very useful text file that provides a structured overview of the entire run.

#### How to open it

- If you enabled "View DRC Summary Report after run finishes" in the Outputs pane, it opens automatically in the **Files pane** after the run.
- You can also open it manually from the Files pane or in any text editor.

#### Structure of the Summary Report

The report has four main sections, shown in the diagram above:

**1. Job Information Section**
```
=== CALIBRE::DRC-H SUMMARY REPORT
===
Execution Date/Time:    Wed Jan 19 13:10:47 2022
Calibre Version:        v2020.4_34.17
Rule File Pathname:     _demo_rules_
Layout System:          GDS
Layout Path(s):         demo.gds
Layout Primary Cell:    TOP
Current Directory:      /home/student/rule_writing
User Name:              student
Maximum Results/RuleCheck: 1000
Maximum Result Vertices:   4096
DRC Results Database:   demo_drc.db (ASCII)
Layout Depth:           ALL
Text Depth:             PRIMARY
Summary Report File:    demo.drc.summary (REPLACE)
```
This section tells you exactly how the job was configured — useful for auditing and reproducing results later.

**2. Runtime Warnings**
Any warnings generated during the run (e.g., memory recommendations).

**3. Layers Processed**
```
--- ORIGINAL LAYER STATISTICS
LAYER AA ..... TOTAL Original Geometry Count = 4  (4)
LAYER CON .... TOTAL Original Geometry Count = 23 (23)
LAYER POLY ... TOTAL Original Geometry Count = 2  (2)
```
Shows each input layer and how many geometric shapes were on it.

**4. RuleCheck Information and Statistics by Cell**
```
--- RULECHECK RESULTS STATISTICS
RULECHECK active_CO ..... TOTAL Result Count = 21 (21)
RULECHECK poly_CO ....... TOTAL Result Count = 2  (2)

--- RULECHECK RESULTS STATISTICS (BY CELL)
CELL TOP ... TOTAL Result Count = 23 (23)
    RULECHECK active_CO ..... TOTAL Result Count = 23 (23)
    RULECHECK poly_CO ....... TOTAL Result Count = 2  (2)
```
The most important part — shows exactly how many violations each rule check found, and breaks it down by cell. This is where you see which rules are the worst offenders.

> 📌 **Note:** The DRC Summary Report is **optional** — it only exists if you specified it in the Outputs pane or with the `DRC SUMMARY REPORT` command. If you did not specify it, the file will not be created.

---

### 4.4 Invoke RVE to View DRC Results

![Invoke RVE to View DRC Results](../screenshots/chapter-02/ch02-diagram-10.png)

**RVE (Results Viewing Environment)** is Calibre's dedicated graphical tool for browsing DRC results. It is much more powerful than reading the text report because it connects directly to your layout viewer.

#### Two ways to open RVE

**Method A — Command line:**
```bash
# First, find the results database name in your rule file:
grep "DRC RESULTS" demo.rules
# Output: DRC RESULTS DATABASE demo_drc.db

# Then launch RVE with that file:
calibre -rve demo_drc.db
```

**Method B — From Calibre Interactive GUI:**
In the **Run Control** pane, under **RVE Options**, enable:
- **Show Results in RVE**
- **Start RVE as soon as results are available** (most convenient — RVE opens automatically when the run finishes)

Other RVE options in the GUI include:
- **Start RVE after batch run** — for runs submitted to a cluster
- **Run RVE on local host**
- **Close RVE before running Calibre DRC** — prevents stale results from showing
- **Close RVE when Calibre Interactive closes**

#### The RVE Interface — Key Areas

The RVE window has several important panels:

| Panel | Location | What it shows |
|-------|----------|---------------|
| **Check / Cell Summary Tree** | Left | A hierarchical tree listing every rule check and every cell that has violations |
| **Results List** | Top right | Numbered list of individual violations for the selected check |
| **Results Details** | Bottom right | Geometric coordinates of the selected violation |
| **Rule Check Comments** | Bottom bar | The comment text written by the rule author to explain the check |

Example from the diagram:
```
Filter: Show All    TOP, 23 Results (in 2 of 2 Checks)

Check / Cell       Results
✗ Check active_CO    21
✗ Check poly_CO       2
```

The status bar at the top tells you: you are viewing cell `TOP`, there are 23 total results, spread across 2 of the 2 checks in the database.

---

## Part 3 — Common DRC RVE Tasks

---

### 5.1 Modify RVE Check/Cell Tree Display

![Modify RVE Check/Cell Tree Display](../screenshots/chapter-02/ch02-diagram-11.png)

By default, RVE groups results with checks at the top level and cells underneath (**Check / Cell** view). You can change this grouping to suit your workflow.

#### How to change the grouping

Go to **View → Tree Options → Group By** and choose from:

| Option | What it shows | Best used when... |
|--------|---------------|-------------------|
| **Check / Cell** *(default)* | Top level = rule checks; expand to see which cells have violations | You want to tackle one rule at a time |
| **Cell / Check** | Top level = cells; expand to see which checks failed in each cell | You want to fix all problems in one cell at a time |
| **Check** | Only check names, no cell breakdown | Quick count of violations per rule |
| **Cell** | Only cell names | Quick count of violations per cell |

#### The Cell / Check view example (top half of diagram)

```
Cell / Check          Results
✗ Cell lab2 ┐          4
   ✗ Check min_spacing_metal1   3
   ✗ Check min_spacing_metal2   1
✗ Cell a2311           3
✗ Cell a1230           1
```

Here you can immediately see that `lab2` has the most errors, with 3 in `min_spacing_metal1` and 1 in `min_spacing_metal2`.

#### Sorting

Click on any **column heading** (e.g., "Results") to sort the list by that column. The small arrow in the heading indicates ascending or descending order. Sorting by Results descending immediately shows you the most problematic rule or cell at the top.

#### Other View options

- **Show Empty Checks** — show rule checks that ran but found zero violations (hidden by default)
- **Show Branch Prefix** — show the full hierarchical path prefix for each cell name

---

### 5.2 Cross-Probe DRC Results to Layout Viewer

![Steps to Cross-Probe DRC Results in Layout Viewer](../screenshots/chapter-02/ch02-diagram-12.png)

**Cross-probing** is the killer feature of RVE — it lets you click on a violation in RVE and instantly see that violation highlighted in your layout viewer (e.g., Virtuoso, Calibre DESIGNrev).

#### Step-by-step cross-probing

1. **Select a rule check** from the Check / Cell tree that has errors (e.g., `Check active_CO`).
2. **Choose a highlighting option** by clicking the highlight icon in the RVE toolbar (the magnifying glass with a star). Options include:
   - **No View Change** — highlight but do not move the layout view
   - **Pan to Highlights** — pan the layout to show the highlighted error
   - **Zoom to Highlights** — zoom in to the highlighted error *(most useful)*
   - **Clear Existing Highlights** — remove all current highlights
3. **Double-click on an error number** in the Results List (e.g., click `10`). The layout viewer immediately zooms to that location and highlights the violating geometry in red.
4. **Read the rule check comments** in the text pane at the bottom of RVE. The rule author typically explains what the rule checks and sometimes gives hints on how to fix it.

> 💡 **Workflow tip:** Use "Zoom to Highlights" as your default. After fixing an error in the layout editor, come back to RVE, right-click the error, and mark it as **Fixed** to track your progress.

---

### 5.3 Displaying Errors in a Hierarchical Design

![Displaying Errors in a Hierarchical Design](../screenshots/chapter-02/ch02-diagram-13.png)

This is an important concept for anyone working with hierarchical (non-flat) layouts.

#### The default behavior — errors reported in top cell space

In a hierarchical design, cells are instantiated (placed) multiple times. By default, Calibre reports errors using **top cell space coordinates** — specifically the coordinates of the **left-most, lowest instance** of the cell containing the error.

**What this means visually:** If you double-click an error in RVE and the highlighted marker appears somewhere in the top-level view, you may need to zoom in to find the actual violation within a subcell.

#### The alternative — errors reported in cell space

If you check **Output cell errors in cell space** in the Outputs pane, Calibre uses the **local coordinate system of the cell** where the error actually lives.

**Benefit:** When you double-click the error in RVE, it opens and zooms into **the cell's own view**, making the violation much easier to find and fix directly.

**How to enable it in the GUI:**
In the **Outputs** pane → check **Output cell errors in cell space**.

> 📌 **Default setting:** "Output cell errors in cell space" is **checked by default** in the Calibre Interactive GUI. It is the recommended setting for hierarchical designs.

---

### 5.4 Report Errors in Cell Space

![Report Errors in Cell Space](../screenshots/chapter-02/ch02-diagram-14.png)

This slide reinforces the cell space setting and shows its SVRF equivalent.

#### GUI setting

In the **Outputs** pane:
- ✅ **Output cell errors in cell space** → errors reported in local cell coordinates (default, recommended)
- ☐ Unchecked → errors reported in top cell space coordinates

#### Equivalent SVRF effect

When this option is active, it is reflected in how the results are written to the database — no separate SVRF command is needed; it is controlled by the GUI option or by corresponding SVRF output statements.

The complete runset at this point looks like:

```svrf
// Job and User settings SVRF file
// Most basic setup file for DRC and LVS run
// Date: 12/1/2021
LAYOUT PRIMARY lab1    // TOPCELL
LAYOUT SYSTEM GDSII    // SPICE
LAYOUT PATH "/home/student/work/myLayout.gds"
DRC RESULTS DATABASE lab1.drc.results ASCII
DRC SUMMARY REPORT lab1.drc.summary

INCLUDE "drc.rules"
INCLUDE "golden_rules"
DRC MAXIMUM RESULTS 1000
```

---

### 5.5 What If the Error Is Still Reported at the Top Level?

![What If the Error Is Still Reported at the Top Level](../screenshots/chapter-02/ch02-diagram-15.png)

Even with "Output cell errors in cell space" enabled, you may sometimes see an error reported at the top level instead of inside its cell. This is expected behavior in specific situations.

#### Why this happens — Cell Expansion

Calibre's hierarchical engine is very efficient, but sometimes it needs to **expand (flatten) a cell** to properly process it. When a cell is expanded, Calibre treats its geometry as if it were directly in the parent cell, so any violations are reported at the parent level.

Reasons Calibre might expand a cell include:
- The cell is very small (trivial)
- The cell is used only once (unique placement)
- The cell is "light-weight" (has no internal hierarchy)

#### How to check if your cell was expanded

Search the **Transcript** for the word `EXPANDING`:

```
297  EXPANDING UNIQUE TRANSPARENT CELL PLACEMENTS
298  PRE-EXPAND COMPLETE CPU TIME = 0  REAL TIME = 0
299  EXPANDING UNIQUE LIGHT-WEIGHT CELL PLACEMENTS
300  PRE-EXPAND COMPLETE CPU TIME = 0  REAL TIME = 0
301  a1720 in lab1 at (308.25, 355)     ← this cell was expanded!
302  EXPAND COMPLETE. CPU TIME = 0  REAL TIME = 0
303  EXPANDING TRIVIAL CELL PLACEMENTS
```

In this example, `a1720` was expanded. If you have an error that you expect to be inside `a1720` but it shows up in the top-level cell, this is why.

> 💡 There is no way to prevent expansion in all cases — it is an internal optimization. Simply be aware of it when tracing errors.

---

### 5.6 Set DRC Result Display Format

![Set DRC Result Display Format](../screenshots/chapter-02/ch02-diagram-16.png)

RVE offers three ways to display individual results in the Results List pane. You switch between them using the icon in the **upper right corner of the Results List**.

#### Three display modes

**1. List view** (default)
```
1   2   3
```
Simple numbered list. Compact, good for designs with many results.

**2. Icons view**
```
✗1  ✗2  ✗3
```
Each result number is accompanied by a status icon (red X = not fixed, green check = fixed, dimmed X = waived). Good for visually tracking which results you have already reviewed.

**3. Details view**
```
| ID | Flat | #EC      | #EL      | #EW       | Vertices |
|----|------|----------|----------|-----------|----------|
|  1 |  1   | 4.00000  | 4.98000  | 0.200000  |    4     |
|  2 |  1   | 4.00000  | 4.60000  | 0.800000  |    4     |
|  3 |  1   | 4.00000  | 4.95400  | 0.300000  |    4     |
```

Tabular view showing geometric measurements for each result:
- **#EC** — x-coordinate of the error centroid
- **#EL** — y-coordinate of the error centroid
- **#EW** — width of the error marker
- **Vertices** — number of vertices in the error polygon
- **Flat** — whether the result is in flat (top-level) coordinates

> 💡 The icon in the toolbar changes to reflect whichever mode is currently selected.

---

### 5.7 Set DRC Result Status

![Set DRC Result Status](../screenshots/chapter-02/ch02-diagram-17.png)

As you review DRC violations, you can mark each one with a status to track your progress. There are three statuses:

| Status | Visual indicator | Meaning |
|--------|-----------------|---------|
| **Not Fixed** | Red X or red text | Default; violation has not been addressed |
| **Fixed** | Green check or green text | You have edited the layout to fix this violation |
| **Waived** | Gray/dimmed X or gray text | You have deliberately decided this violation is acceptable and does not need to be fixed |

#### When to use "Waived"

Waived is used when a violation is **intentional or approved**. For example, a foundry may allow certain rule exceptions in specific situations (like seal rings), or an IP block from a vendor may have pre-approved rule waivers. You waive the error to indicate you have reviewed it and it is OK.

#### How to set the status

1. In the Results List, right-click on one or more results.
2. From the context menu, select:
   - **Fix** → marks as Fixed
   - **Waive → Waive** → marks as Waived
   - To return to Not Fixed: disable both Fixed and Waived

#### Important behaviors

- **Waived results are saved** in a file named `<result_file_name>.waived` (e.g., `lab1.drc.results.waived`). This file persists between runs.
- **Fixed results are reset** each time you re-run DRC. This is correct behavior — after editing the layout, you need to re-run to confirm the fix actually worked.

---

### 5.8 Comment Waived Results

![Comment Waived Results](../screenshots/chapter-02/ch02-diagram-18.png)

When waiving a result, best practice is to add a comment explaining **why** it is being waived. This is essential for design reviews and audits — someone looking at the results six months later needs to understand why an error was waived.

#### How to enable waiver comments

1. In RVE, click the **Setup (gear icon)** button in the toolbar to open the RVE Options dialog.
2. Go to the **Waivers** category.
3. Check **"Show comment dialog while reviewing (PERC only) or waiving results"**.
4. Click **Apply**.

#### What happens when you waive a result with comments enabled

A dialog box pops up:

```
┌─────────────────────────────────────────────────────┐
│  Waived Result Comment                              │
│  Waived by student on Thu Jun 01 08:24:17 PDT 2017  │
│  ┌───────────────────────────────────────────────┐  │
│  │ This error waived per Ted Smith.              │  │
│  └───────────────────────────────────────────────┘  │
│                                  [OK]    [Cancel]   │
└─────────────────────────────────────────────────────┘
```

The timestamp and username are added automatically. You type your justification in the text box. Click **OK** to save.

---

### 5.9 View Waived Result Comments

![View Waived Result Comments](../screenshots/chapter-02/ch02-diagram-19.png)
![View Waived Result Comments Continued](../screenshots/chapter-02/ch02-diagram-20.png)

Once you have added waiver comments, there are three ways to view them.

#### Method 1 — Hover tooltip in RVE

In the Results List, hover your mouse over a waived result (shown with a dimmed X). A tooltip popup appears showing:

```
Waived by calibre on Mon Jul 25 11:18:18 PDT 2011
This error waived per Ted Smith.
```

Quick and convenient for spot-checking individual waivers.

#### Method 2 — Open the `.waived` file in RVE

1. In RVE, go to **File → Open Text File...**
2. Navigate to your working directory.
3. Select the file `<results_name>.waived` (e.g., `lab2.drc.results.waived`).
4. The file opens in a new tab inside RVE.

The `.waived` file format:
```
Lab2 1000 1
min_spacing_metal1
2 2 6 Jul 25 11:22:46 2011
WEO This error waived per Ted Smith.
WEI 1swallow/1311b18135
e 1 2
221900 451000 226500 451000
222500 451800 227100 451000
e 2 2
...
```

Lines starting with `WEO` contain the waiver comment. Lines starting with `e` contain the geometric coordinates of the waived error.

#### Method 3 — Create an ASCII Waiver Report

For formal design reviews, you can export a clean summary of all waivers:

1. In RVE, go to **Tools → Create Waiver Report...**
2. Choose a file name (e.g., `lab2.drc.results.report.txt`).
3. Click **Save**.

The report looks like:

```
##################################################################
# Waived Error Report for /home/calibre/calibre_drc/lab2/lab2.drc.results
# Created by user calibre on Mon Jul 20 10:59:54 PDT 2009
##################################################################

Cell: lab2
    Check: min_spacing_metal1
        Error 2:  Mon Jul 20 08:12:03 PDT 2009, calibre
                  This error waived per Ted Smith.
        Error 3:  Mon Jul 20 07:42:27 PDT 2009, calibre

End of Report
```

This report is easy to attach to a design review package or tapeout checklist.

---

## 6. Key Terms Glossary

| Term | Definition |
|------|------------|
| **DRC** | Design Rule Check — automated verification that a layout conforms to manufacturing rules |
| **nmDRC** | Calibre's DRC engine, optimized for nanometer-scale designs |
| **SVRF** | Standard Verification Rule Format — the scripting language used to write Calibre rule files and job settings |
| **Runset file** | A SVRF file containing the job configuration (layout path, output paths, options) — distinct from the rule file which contains the actual checks |
| **Rule file** | A SVRF file containing the actual design rules (minimum width, minimum spacing, etc.) — provided by the foundry |
| **INCLUDE** | SVRF statement that reads and executes another file |
| **LAYOUT PRIMARY** | SVRF statement that specifies the top cell name |
| **LAYOUT SYSTEM** | SVRF statement that specifies the layout file format (e.g., GDSII) |
| **LAYOUT PATH** | SVRF statement that specifies the layout file path |
| **DRC RESULTS DATABASE** | SVRF statement that specifies the output results file name and format |
| **DRC SUMMARY REPORT** | SVRF statement that specifies the output summary report file name |
| **DRC MAXIMUM RESULTS** | SVRF statement that limits how many violations are reported per rule check |
| **Top Cell** | The root cell in a hierarchical layout — the cell that contains all other cells |
| **GDSII** | A common binary file format for IC layouts (Graphic Design System II) |
| **RVE** | Results Viewing Environment — Calibre's graphical tool for browsing and navigating DRC results |
| **Cross-probe** | The action of clicking an error in RVE and having the layout viewer zoom to that exact location |
| **Transcript** | The real-time text log generated by Calibre during a run |
| **Hierarchical run** | A DRC run that preserves the cell hierarchy of the layout (as opposed to flattening everything) |
| **Cell expansion** | When Calibre internally flattens a subcell for processing, causing its errors to be reported at the parent level |
| **Cell space coordinates** | Error coordinates expressed in the local coordinate system of the cell where the error lives |
| **Top cell space coordinates** | Error coordinates expressed in the global coordinate system of the top cell |
| **Waived** | A result status meaning the violation has been reviewed and intentionally accepted |
| **Fixed** | A result status meaning the layout has been edited to fix the violation |
| **`.waived` file** | A file automatically created by RVE to store the list of waived results persistently |

---

## 7. Quick Reference — The Job Settings SVRF File

Below is the complete, annotated runset file that was built up step by step throughout this chapter.

```svrf
// ============================================================
// Job and User settings SVRF file
// Most basic setup file for DRC and LVS run
// Date: 12/1/2021
// ============================================================

// --- INPUT SETTINGS ---
LAYOUT PRIMARY lab1         // Top cell name in the layout
LAYOUT SYSTEM GDSII         // Layout file format
LAYOUT PATH "/home/student/work/lab1.gds"  // Path to the layout file

// --- OUTPUT SETTINGS ---
DRC RESULTS DATABASE lab1.drc.results ASCII  // Where to write violation data
DRC SUMMARY REPORT lab1.drc.summary          // Where to write the text summary

// --- OPTIONS ---
DRC MAXIMUM RESULTS 1000    // Report at most 1000 results per rule check

// --- RULE FILES ---
INCLUDE "drc.rules"         // Main foundry rule file
INCLUDE "golden_rules"      // Additional rule file
```

> 💡 **Save this as a template.** Most DRC jobs share this same basic structure. You only need to change the layout path, top cell name, and output file names for each new project.

---

*End of Chapter 02 — Calibre nmDRC Basics*

*Next: Chapter 03 — Calibre nmDRC Job Customization*
