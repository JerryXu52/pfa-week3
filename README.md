# Procedural Cable & Hose Rig Tool

**Assessment 2 · Route B** · Autodesk Maya · `CableHoseRigTool.py`

A Python tool for Autodesk Maya that procedurally generates realistic hanging cable and hose bundles between selected 3D anchor points. Select two or more locators, press one button, and the tool builds a gravity-correct, collision-free bundle — complete with material and a non-destructive cleanup system.

## Key Features

- **Procedural catenary curves** — every span hangs on a true catenary, the curve a real chain makes under gravity, with sag expressed as a fraction of span length so it behaves consistently at any scale.
- **Sunflower bundle packing** — strands are distributed on a Fermat spiral at the golden angle (≈137.5°), the arrangement that spaces points most evenly for a given radius.
- **Collision relaxation** — the tool measures true segment-to-segment distance on the built geometry and iteratively widens the bundle or eases the slack until every strand clears its neighbours. Verified at 0 intersections across 240 extreme and 148 realistic configurations.
- **Spiral twist, length stagger and slack variation** — rotate the bundle along its length, trim strands to different lengths for a layered look, or give each strand its own sag.
- **Custom material manager** — a colour picker drives one shared Lambert (`cableMat_custom`), updated in place rather than duplicated on each run.
- **Dual-mode execution** — runs as a Maya UI, or launches itself into a running Maya session from any system terminal over TCP.
- **Single-chunk undo and safe cleanup** — an entire bundle undoes with one Ctrl+Z, and cleanup removes only the tool's own prefixed nodes, never the artist's work.

## Prerequisites & Requirements

| Requirement | Details |
|---|---|
| **Software** | Autodesk Maya 2022 or later (Python 3). Developed and tested on Maya 2026. |
| **Terminal launcher** | Python 3 on your system path. |
| **Dependencies** | None. Built entirely on the Python standard library (`math`, `random`, `colorsys`, `socket`, `base64`, `argparse`, `os`, `sys`) and `maya.cmds`. |

> **Note on Python 2.7:** the terminal launcher uses Python 3-only features (`open(..., encoding=)` and `ConnectionRefusedError`), so it will not run under Python 2. Maya 2022 and later ship with Python 3, which covers every currently supported release.

## How to Run

### Method 1 — Direct execution inside Maya

1. Open the **Script Editor** (`Windows ▸ General Editors ▸ Script Editor`) and switch to a **Python** tab.
2. Paste the contents of `CableHoseRigTool.py` and press **Execute**. The tool window opens automatically.

To load it as a module instead:

```python
import sys
sys.path.append(r"/path/to/folder/containing/the/script")

import CableHoseRigTool
CableHoseRigTool.show_ui()
```

### Method 2 — Remote socket launcher

**Step 1.** Inside Maya, open Command Port 7002 once (Python tab):

```python
import maya.cmds as cmds

if not cmds.commandPort("127.0.0.1:7002", query=True):
    cmds.commandPort(name="127.0.0.1:7002", sourceType="mel")
```

**Step 2.** From any system terminal, in the script's folder:

```bash
python CableHoseRigTool.py
```

Use `python3` instead if `python` on your system still points to Python 2. The script Base64-encodes its own source, wraps it in a single MEL `python()` call — which eliminates every quoting and newline hazard — and posts it to Maya over TCP. The UI then appears inside Maya.

Optional flags: `--host`, `--port`, `--timeout`. Run with `--help` for details.

### Using the tool

Select **two or more** transforms or locators in anchor order — cables are built between consecutive pairs, so selecting A, B, C produces two spans. Adjust the sliders and press **Generate Cable Rig**. **Clear Rig** removes everything the tool created and nothing else.

## 10 Lines Code Explanation

Here is an explanation of 10 fundamental lines from `CableHoseRigTool.py` in plain English:

1. `import math`
   - **Explanation:** Imports Python's built-in math module so the script can use mathematical functions like square roots and sine/cosine to calculate cable curves and spiral twists.

2. `MAYA_HOST = "127.0.0.1"`
   - **Explanation:** Stores the local IP address so the script knows to connect to the Maya application running on the same computer.

3. `WINDOW_NAME = "cableHoseRigToolWin"`
   - **Explanation:** Sets a unique text ID for the window so the tool can check for and close existing copies before opening a new one.

4. `positions = []`
   - **Explanation:** Creates an empty list to store the 3D coordinates of all the selected anchor points in the Maya scene.

5. `def vector_sub(a, b):`
   - **Explanation:** Defines a helper function that takes two 3D points and subtracts one from the other.

6. `return (a[0] - b[0], a[1] - b[1], a[2] - b[2])`
   - **Explanation:** Subtracts the X, Y, and Z numbers of the two points individually and sends the resulting 3D vector back.

7. `count = 0`
   - **Explanation:** Sets a counter variable to zero so we can start counting how many invalid or overlapping anchors exist.

8. `count += 1`
   - **Explanation:** Increases the counter number by 1 every time the script finds a pair of anchors sitting in the exact same spot.

9. `if len(positions) < 2:`
   - **Explanation:** Checks if fewer than two anchors were collected so the tool can safely stop before trying to build a cable without enough points.

10. `print("[CableHoseRigTool] " + text)`
    - **Explanation:** Outputs a status message with a clear tag into the Maya Script Editor so users can see what the tool is doing.
