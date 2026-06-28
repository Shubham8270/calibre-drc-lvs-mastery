# Calibre DRC/LVS Master Cheat Sheet

> Quick revision notes for Chapter 01 (Calibre Basic Concepts) and Chapter 02 (Calibre nmDRC Basics)

---

# 1. Calibre Overview

## What is Calibre?

Calibre is Siemens EDA's physical verification platform used before chip fabrication.

### Main Tasks

| Check | Purpose                 |
| ----- | ----------------------- |
| DRC   | Design Rule Checking    |
| LVS   | Layout Versus Schematic |

### Remember

```text
DRC = Is layout manufacturable?
LVS = Does layout match schematic?
```

---

# 2. Calibre Tool Suite

| Tool                | Purpose                      |
| ------------------- | ---------------------------- |
| Calibre nmDRC       | Design Rule Checking         |
| Calibre nmLVS       | Layout vs Schematic          |
| Calibre RVE         | Results Viewer               |
| Calibre Interactive | GUI Frontend                 |
| Calibre DESIGNrev   | Layout Viewer                |
| Calibre PEX         | Parasitic Extraction         |
| Calibre DFM         | Design For Manufacturability |

---

# 3. IC Design Flow

```text
Schematic
    ↓
Netlist
    ↓
Simulation
    ↓
Layout (GDS)
    ↓
DRC
    ↓
LVS
    ↓
Tapeout
```

---

# 4. Calibre Verification Flow

```text
Layout (.gds)
      +
Rule File (.rules)
      ↓
Calibre nmDRC
      ↓
Results Database
      ↓
RVE
      ↓
Fix Errors
      ↓
Re-run
```

---

# 5. Important Terminology

| Term       | Meaning                           |
| ---------- | --------------------------------- |
| SVRF       | Standard Verification Rule Format |
| RDB        | Results Database                  |
| SVDB       | Standard Verification Database    |
| RVE        | Results Viewing Environment       |
| Rule Check | Individual DRC rule execution     |

---

# 6. SVRF File Types

## Process Rule File

* Foundry supplied
* Technology rules
* Usually never modified

## Job SVRF

* User supplied
* Input/output settings

## User SVRF

* Optional
* Rule customization

---

# 7. Essential SVRF Commands

## Layout Input

```svrf
LAYOUT PRIMARY TOP
LAYOUT SYSTEM GDSII
LAYOUT PATH "layout.gds"
```

## Results Output

```svrf
DRC RESULTS DATABASE results.db ASCII
DRC SUMMARY REPORT summary.rpt
```

## Rule Files

```svrf
INCLUDE "drc.rules"
INCLUDE "golden_rules"
```

## DRC Options

```svrf
DRC MAXIMUM RESULTS 1000
```

---

# 8. Basic DRC Runset Template

```svrf
LAYOUT PRIMARY lab1
LAYOUT SYSTEM GDSII
LAYOUT PATH "lab1.gds"

DRC RESULTS DATABASE lab1.drc.results ASCII
DRC SUMMARY REPORT lab1.drc.summary

DRC MAXIMUM RESULTS 1000

INCLUDE "drc.rules"
INCLUDE "golden_rules"
```

---

# 9. Frequently Used Commands

## Launch Calibre GUI

```bash
calibre -gui
```

## Launch DRC GUI

```bash
calibre -gui -drc
```

## Launch DRC GUI with Runset

```bash
calibre -gui -drc demo_runset
```

## Run DRC

```bash
calibre -drc -hier my_job.rules
```

## Open Layout

```bash
calibredrv layout.gds
```

## Open RVE

```bash
calibre -rve demo_drc.db
```

## Open Documentation

```bash
mgcdocs
```

---

# 10. Calibre Interactive Pages

## Rules

* Rule file selection
* Load rules

## Inputs

* Layout format
* Layout file
* Top cell

## Outputs

* Results database
* Summary report

## Options

* Maximum results
* Additional rule files

## Run Control

* CPU count
* Local/Remote host

## Transcript

* Live execution log

---

# 11. GUI ↔ SVRF Mapping

| GUI Setting      | SVRF Command         |
| ---------------- | -------------------- |
| Rules File       | INCLUDE              |
| Top Cell         | LAYOUT PRIMARY       |
| Layout Format    | LAYOUT SYSTEM        |
| Layout Path      | LAYOUT PATH          |
| Results Database | DRC RESULTS DATABASE |
| Summary Report   | DRC SUMMARY REPORT   |
| Maximum Results  | DRC MAXIMUM RESULTS  |

---

# 12. Transcript Keywords

## Rule Completion

```text
DRC RuleCheck active_CO COMPLETED
```

## Cell Expansion

```text
EXPANDING UNIQUE CELL PLACEMENTS
```

## Run Summary

```text
TOTAL RULECHECKS EXECUTED
TOTAL RESULTS GENERATED
DRC RESULTS DATABASE FILE
```

---

# 13. DRC Summary Report Sections

## Job Information

Contains:

* Date
* Version
* Rule File
* Layout Path
* Top Cell

## Runtime Warnings

Contains:

* Memory warnings
* Performance warnings

## Layer Statistics

Example:

```text
LAYER POLY
LAYER METAL1
LAYER VIA1
```

## Rule Statistics

Example:

```text
RULECHECK active_CO
RULECHECK poly_CO
```

---

# 14. RVE Panels

| Panel             | Function             |
| ----------------- | -------------------- |
| Check / Cell Tree | Violation categories |
| Results List      | Individual errors    |
| Results Details   | Coordinates          |
| Rule Comments     | Rule description     |

---

# 15. Cross-Probing Workflow

```text
Select Rule
      ↓
Select Error
      ↓
Double Click
      ↓
Zoom To Error
      ↓
Fix Layout
```

Recommended Mode:

```text
Zoom to Highlights
```

---

# 16. Cell Space vs Top Cell Space

## Cell Space (Recommended)

```text
Local cell coordinates
Easy debugging
```

## Top Cell Space

```text
Top-level coordinates
Harder debugging
```

Enable:

```text
Output Cell Errors In Cell Space
```

---

# 17. Cell Expansion

Sometimes Calibre internally flattens cells.

Look for:

```text
EXPANDING UNIQUE CELL PLACEMENTS
EXPANDING TRIVIAL CELL PLACEMENTS
```

Result:

```text
Errors may appear at parent level
instead of inside the original cell.
```

---

# 18. DRC Result Status

| Status  | Meaning   |
| ------- | --------- |
| Red X   | Not Fixed |
| Green ✓ | Fixed     |
| Gray    | Waived    |

---

# 19. Waivers

## When to Waive

* Approved foundry exception
* Vendor IP issue
* Approved design exception

## Waiver File

```text
results.db.waived
```

## Generate Waiver Report

```text
Tools → Create Waiver Report
```

Always document:

```text
Who approved?
Why approved?
Date approved?
```

---

# 20. Interview Questions

### What is DRC?

Checks layout against manufacturing rules.

### What is LVS?

Checks layout connectivity against schematic.

### What is SVRF?

Calibre verification rule language.

### What is RVE?

Results Viewing Environment.

### What is a Runset?

Saved Calibre GUI configuration.

### What is Cell Expansion?

Temporary flattening of cells during DRC.

### Difference Between RDB and SVDB?

| RDB         | SVDB         |
| ----------- | ------------ |
| DRC Results | LVS Database |

---

# One-Minute Revision
```text
DRC = Manufacturability

LVS = Layout vs Schematic

SVRF = Calibre Language

DESIGNrev = Layout Viewer

RVE = Results Viewer

INCLUDE = Load Rule File

LAYOUT PRIMARY = Top Cell

LAYOUT PATH = GDS File

DRC RESULTS DATABASE = Results File

DRC SUMMARY REPORT = Summary File

DRC MAXIMUM RESULTS = Violation Limit

Flow:

Layout + Rules
      ↓
    nmDRC
      ↓
     RVE
      ↓
     Fix
      ↓
   Re-run
```

---

# Formula for Remembering Calibre

```text
GDS + SVRF
     ↓
   nmDRC
     ↓
    RDB
     ↓
    RVE
     ↓
 Fix & Repeat
```
