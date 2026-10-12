# QxQ Fixture Tools — User Guide

QxQ Fixture Tools prepares the files needed to build a test fixture from a customer's board package.
It turns the customer's test point list into the fixture Excel file and the drill files. It also finds
the components that are too tall for the fixture and lays out the milling pockets that clear them.
Every file it writes is read back and checked before you use it.

**[Download the latest version](https://github.com/Chris4lyfie/FixtureTools-Releases/releases/latest)**:
`QxQ_Fixture_Tools.exe`, a single Windows program. Nothing needs to be installed.

---

## Contents

1. [At a glance](#1-at-a-glance)
2. [Getting started](#2-getting-started)
3. [Test Points tab](#3-test-points-tab)
4. [Height & Milling tab](#4-height--milling-tab)
5. [Board view](#5-board-view)
6. [A typical job, start to finish](#6-a-typical-job-start-to-finish)
7. [Updates, versions and settings](#7-updates-versions-and-settings)
8. [Troubleshooting](#8-troubleshooting)
9. [Things to keep in mind](#9-things-to-keep-in-mind)

---

## 1. At a glance

The program has two tabs. Each takes files from the customer package and writes its results next to them.

| | **Test Points** | **Height & Milling** |
|---|---|---|
| **Purpose** | Customer test point list → fixture Excel + drill files | Parts too tall for the fixture → milling pockets |
| **You give it** | The customer's test point file (`.csv`, `.xls`, `.xlsx`) | The package folder: ODB++ plus a STEP model and/or an Altium `.PcbDoc` |
| **It writes** | `TP=<name>.xls` and a `TP=<name>_DRL` folder of drill files | A `Height_Report` folder and a `Milling_Report` folder (HTML, CSV, DXF) |
| **It checks** | Every written file is read back and compared with the customer's list | Every test point hole is checked against every milling pocket |

The **board view** on the Height & Milling tab draws the milling pockets and the test point holes
together, from the files on disk. You see exactly what will be milled and drilled, and any hole that
breaks into a pocket is marked in red.

---

## 2. Getting started

### Requirements

- Windows 10 or 11 (64-bit).
- Nothing else. Python and every library the program needs are built into the `.exe`.
- An internet connection only for automatic updates. Everything else works offline.

### Installing

1. Download `QxQ_Fixture_Tools.exe` from the
   [latest release](https://github.com/Chris4lyfie/FixtureTools-Releases/releases/latest). It's under
   **Assets**. The "Source code" links there are an automatic GitHub snapshot and aren't needed.
2. Put it in a folder **you can write to**, such as your Desktop or a folder in Documents. Don't use
   `Program Files` or a read-only network share: the program replaces itself when it updates.
3. Double-click it. It takes a second or two to start because it unpacks itself first.

**"Windows protected your PC"**: the program isn't code-signed, so Windows SmartScreen may warn the
first time. Click **More info → Run anyway**. You only need to do this once.

### Opening files

- **Drag and drop**: drop test point files or a package folder onto the window, or onto
  `QxQ_Fixture_Tools.exe` itself.
- **Browse buttons**: every tab also has a Browse button.

---

## 3. Test Points tab

Converts the customer's test point list into the files the fixture is built from.

### 3.1 Input: the customer's test point file

**File types:** `.csv`, `.xls`, `.xlsx`. You can drop several files at once; they're loaded one after
another.

**Columns it looks for** (the names don't have to match exactly):

| Column | Also recognised as | Required |
|---|---|---|
| Net | Signal Name, Net Name | Net **or** Name |
| Name | TP Label, TP Name, Test Point, TP# | Net **or** Name |
| X / Y | X Coord, X Coordinate, X Position, Center X; or one combined `X/Y Coordinates` column | Yes |
| Side | Layer, Board Side, Mount Side, Top/Bottom | Recommended |
| Hole Size | Hole, Drill Size, Hole Diameter | No |
| Type | Mount, Mount Type | No |
| Comment | Note, Notes, Remark, or an unnamed text column right after the table | No |

**What it copes with:**

- **Layouts:** Altium test point CSVs. Excel sheets where the table doesn't start on the first row or
  the first sheet. Section label rows (POWER, JTAG, …) between points. Notes below the table.
- **Units:** coordinates in **mm, mil or inches**, from the column header (`X Coord (mil)`) or the value
  itself (`12.5mm`). Everything is converted to mm.
- **Side values:** `Top`, `Bottom`, `Top Layer`, `Bot`, `T`, `B`, `Front`, `Back`, `Component`,
  `Solder`, … are all understood.
- **Missing names:** if the Name column is empty, names are assigned.
- **Missing side:** if the file has no Side column, the program asks which side the points are on.

**Nothing goes missing silently.** Every non-blank row is accounted for. A row the program can't read
with confidence is never dropped quietly. It's shown in a red **"Needs your OK before Generate"** box,
for example:

- a row with a name or net but no readable X/Y
- formulas Excel never calculated (they read as blank). Open the file in Excel, save it, and drop it again.
- a second sheet that also contains a test point table (only one is read)
- two test points on the exact same spot but on **different nets**
- points whose side isn't Top or Bottom (they can't go in a drill file)

Tick **"I checked these – OK to generate"** to go ahead, or use **Pick points / sides…** to fix them.

### 3.2 Options

| Option | What it does |
|---|---|
| **TP names (column E)** | How the names in the fixture Excel are written. **6-char rule** (default): keep the customer's names if they all fit in 6 characters, otherwise assign new ones. **Customer names**: always keep them. **Without pin suffix**: `TP102-1` → `TP102`, if the names stay unique. **Always TP / TTP / BTP**: always assign new names. Assigned names are `TP1, TP2, …` on a one-sided board, or `TTP1…` (Top) and `BTP1…` (Bottom) on a two-sided one. Customer numbers such as `TP12` are kept where possible and never reused. The customer's original name always stays in column G. |
| **T04 diameter (in)** | Diameter written for the probe tool in the raw drill files. Default `0.069`. |
| **Probe sizing** | **DFT spacing table** (recommended): each hole gets the largest probe (100, 75, 50 or 39 mil) the QxQ Design-for-Test spacing minimums allow, checked against every other hole on that side. **Old CENSORT limits**: the older nearest-neighbour rule, for matching jobs made the old way. |
| **Board upside down in fixture** | Swaps the drill files: customer Top points go to `PRB.DRL`, Bottom points to `TOP.DRL`. It applies to this board only and isn't remembered. |
| **Pick points / sides** | After loading, opens a list where each point can be set to **Probe** (wired), **Clear** (clearance hole only) or **Skip** (left out, with the reason recorded). You can also move points between Top and Bottom and add extra holes by hand. |
| **Pocket check** | Checks every hole against the milling pockets of the board in the same package. It uses the limits and offset from the Height & Milling tab. |

### 3.3 Working through a file

1. Drop the test point file. The preview shows every hole at its real drill size on the **TOP** and
   **PRB (bottom)** tabs, and the summary shows the probe count per side.
2. Deal with anything in the red **Needs your OK** box.
3. Click **Generate files**.
4. Read the result in the **Log**. A green **CHECK OK** means every point in the customer's file is
   accounted for in every output file. A red **CHECK FAILED** means don't use the files; the problems
   are listed.
5. Click **Open output folder** to see the files.

**Skip** moves on to the next file without writing anything.

### 3.4 Output

The files are written **next to the customer's test point file**.

**`TP=<name>.xls`**: the fixture Excel file.

| Columns | Content |
|---|---|
| A–D | X and Y, in mm and in inches (inches are formulas from the mm values) |
| E | Fixture test point name (see *TP names*) |
| F–L | The customer's columns: Net, Name, X, Y, Side, Hole Size, Type |
| M | The customer's comments, if any |

- **Order:** Top points first, then Bottom, separated by a blank row. Inside each group the customer's
  order is kept.
- **Labels and notes:** section labels are kept above their points, and notes from below the customer's
  table are copied below the points.

**`TP=<name>_DRL\`**: the drill files, in Excellon format (inches).

| File | Content |
|---|---|
| `TOP.DRL`, `PRB.DRL` | Raw drill files for the top and bottom probe plates (probe holes and clearance holes) |
| `TOP.DRL.MAP`, `PRB.DRL.MAP` | Hole positions with their test point names |
| `SORTTOP`, `SORTPRB` | Holes grouped by probe size |
| `SORTTOP.DRL`, `SORTPRB.DRL` | Final drill files with the real drill sizes |
| `SORT*.DRL_DRL.REP` | Hole count per drill tool |
| `SPACING_REPORT.txt` | Every hole, its nearest neighbour, the spacing and the drill chosen |

- **Missing sides:** a side with no points gets no files.
- **Stale files:** files from an earlier run that no longer apply are removed, so an out-of-date file
  can't be used by mistake.

**Test points on the same spot.** Two test points at exactly the same X/Y (e.g. TP9 and TP11) both
stay in the Excel and the MAP, each in its own place in the customer's order. The spot is drilled
**once**. The spacing report and board view name it `TP9+TP11`.

---

## 4. Height & Milling tab

Finds the components that are too tall for the fixture and creates a milling pocket for each one.

### 4.1 Input: the package folder

Drop the customer's package folder, or use **Browse folder…**. The program finds the board files inside
it:

| File | Used for | Required |
|---|---|---|
| **ODB++** (extracted folder, `.tgz` or `.zip`) | Components, footprints, board outline | Yes |
| **STEP** (`.step` / `.stp`) | 3D body of every part: heights and pocket outlines | STEP and/or PcbDoc |
| **Altium PcbDoc** (`.PcbDoc`) | 3D model heights and part descriptions | STEP and/or PcbDoc |
| Test point file | For the pocket check (found automatically, or **Choose TP file…**) | No |

**Settings:**

| Setting | Default | Meaning |
|---|---|---|
| Top limit | 140 mil | Parts on the top side taller than this are flagged |
| Bottom limit | 100 mil | Parts on the bottom side taller than this are flagged |
| Pocket offset per side | 50 mil | Clearance added on **each** side of a part's 3D body |
| DXF units | inch | Units of the milling DXF (inch or mm) |

### 4.2 How it decides

- **Height** = the taller of the STEP body and the PcbDoc 3D model, to be safe. A part is flagged
  when it is **taller than** its side's limit. Where the STEP, PcbDoc and ODB++ heights disagree, the
  report adds a cross-check note.
- **Unplaced bodies:** a STEP body that's over the limit but doesn't belong to any component is
  reported too, and gets a pocket.
- **Pocket** = the part's full 3D body outline (leads and overhangs included), grown by the offset on
  each side, with the same centre. Example: a 300 × 200 mil body with a 50 mil offset gives a
  400 × 300 mil pocket.
- **Parts without a STEP body:** a flagged part with no body in the STEP uses its ODB++ footprint
  outline.

### 4.3 Working through a package

1. Drop the package folder and check the limits and offset.
2. Click **Run height check + milling**. A large STEP file can take a minute.
3. The summary lists the parts that are too tall, their pockets, and any test point hole that breaks
   into a pocket. The **Pockets** tab lists every pocket.
4. Open the reports with **Height report**, **Milling report** or **Open folder**.

### 4.4 Output

Written **inside the package folder**.

**`Height_Report\`**

| File | Content |
|---|---|
| `<board>_height_report.html` | Summary, board maps of both sides, and tables of the flagged parts, to view or print |
| `<board>_height_report.csv` | The flagged parts with heights, positions and descriptions, for Excel or email |

**`Milling_Report\`**

| File | Content |
|---|---|
| `<board>_milling_report.html` | Pocket list, board maps, and any test point holes that touch a pocket |
| `<board>_milling_report.csv` | Each pocket's centre, size and corners, for CAM or Excel |
| `<board>_milling_<side>.dxf` | One per side that has pockets: board outline and pocket rectangles (AutoCAD R12, opens in any CAD/CAM program). Layers: `BOARD_OUTLINE`, `POCKETS`, `LABELS`. |

All coordinates are CAD coordinates seen **from the top of the board**, relative to the board origin.
Bottom-side pockets are not mirrored.

---

## 5. Board view

The lower half of the Height & Milling tab draws each side of the board:

- the board outline
- every milling pocket
- every test point hole, at its real drill size

A hole that touches or overlaps a pocket on the same side is boxed in **red**.

**It shows the files on disk**: the milling DXFs and the final drill files (`SORTTOP.DRL` /
`SORTPRB.DRL` with their `.MAP`). That makes it a check of what will actually be milled and drilled,
not of what the program intended. It refreshes after a run, and after **Generate files** on the Test
Points tab.

| Control | What it does |
|---|---|
| **Board TOP side / Board BOTTOM side** | Switch between the two sides |
| Mouse wheel / drag | Zoom / move |
| **Fit** | Show the whole board |
| **Flip view** | See the side from below |
| Hover | A hole shows its name, net, probe size and drill file; a pocket shows its size |
| **Open job folder…** | View any job's files without running anything, including older jobs and hand-made sets |
| **drill files** list | Choose which drill file set to show when a folder has more than one |

**Board upside down** is worked out from the fixture Excel next to the drill files. Each probe plate
is drawn against the pockets of the board side it faces.

---

## 6. A typical job, start to finish

1. Put the customer's files in one package folder: ODB++, STEP and/or PcbDoc, and the test point file.
2. **Height & Milling tab**: drop the folder, check the limits, click **Run height check + milling**.
   Review the height and milling reports.
3. **Test Points tab**: drop the test point file. Set the options, sort out anything in the red box,
   and click **Generate files**. Make sure the Log says **CHECK OK**.
4. Back on **Height & Milling**, look at the board view. If any hole is boxed in red, decide what to
   do (move the probe, clearance hole, skip, or change the pocket) and generate again.
5. Use `TP=<name>.xls`, the `SORT*.DRL` drill files and the milling DXF to build the fixture.

---

## 7. Updates, versions and settings

**Automatic updates.**
- **When:** each time the program starts, it checks for a newer version.
- **If there is one:** a window shows what's new, with **Update now** or **Later**.
- **Update now:** downloads it, verifies it, and restarts the program in a few seconds.
- **Later:** keeps your current version; you're asked again at the next start.
- **No connection:** the program simply starts as normal.

**Version numbers** follow `MAJOR.MINOR.PATCH`:

| Change | Example | Meaning |
|---|---|---|
| PATCH | 1.0.0 → 1.0.1 | Bug fixes only |
| MINOR | 1.0.1 → 1.1.0 | New features; files come out as before |
| MAJOR | 1.1.0 → 2.0.0 | The output files or the way you use the program changed. Check the release notes and your first job carefully. |

Every release's notes are on the [releases page](https://github.com/Chris4lyfie/FixtureTools-Releases/releases).
The window title shows the version you're running.

**Settings** (options, limits, folders) are remembered between runs in
`%APPDATA%\FixtureTools\settings.json`. To stop the update check on one PC, set
`"check_updates": false` in that file.

---

## 8. Troubleshooting

| Problem | What to do |
|---|---|
| "Windows protected your PC" | **More info → Run anyway** (only the first time). |
| "Couldn't find a header row with X, Y and Net or Name" | The file has no recognisable column headers. Rename the headers (e.g. `Net`, `X`, `Y`, `Side`) and drop it again. |
| Points listed as "formulas Excel never calculated" | Open the file in Excel, save it, and drop it again. |
| **CHECK FAILED** after Generate | Don't use the files. The Log lists the problems. Fix the cause and generate again. |
| "No ODB++ found next to the test point file" | The pocket check needs the package. Click **Change folder…** and pick the package folder. |
| Height check finds no STEP | The check runs on PcbDoc heights only. Add the STEP file to the package for pocket outlines that include leads and overhangs. |
| Update says the folder can't be written to | Move `QxQ_Fixture_Tools.exe` to your Desktop or Documents, or download the new version by hand from the releases page. |
| Drag and drop doesn't work | Use the Browse buttons. |

---

## 9. Things to keep in mind

- **Same origin.** Test point X/Y must use the same CAD origin as the ODB++. Otherwise the pocket
  check and board view compare things that don't line up.
- **Units.** Check the warnings in the Log when a file's coordinates were in mil or inches. They were
  converted, and the Log says so.
- **The red box is there for a reason.** Ticking "I checked these" means the files are written exactly
  as listed. Read the items first.
- **Check the files on disk.** The board view and the read-back check both look at the written files.
  Use them before the files go to the machine.
