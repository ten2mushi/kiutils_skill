---
name: kiutils
description: |
  KiCad file manipulation with kiutils Python library for offline parsing and modification. Use when:
  - Parsing/modifying .kicad_pcb (PCB boards) without running KiCad
  - Parsing/modifying .kicad_sch (schematics) without running KiCad
  - Creating/editing .kicad_mod (footprints) programmatically
  - Creating/editing .kicad_sym (symbol libraries)
  - Managing sym_lib_table / fp_lib_table (library tables)
  - Modifying .kicad_dru (design rules)
  - Editing .kicad_wks (worksheet templates)
  - Batch processing KiCad files without GUI
  - Scripted design rule checks or modifications
  - Automated BOM generation from schematics
  - Programmatic footprint/symbol library management
  Triggers: "kiutils", "kicad file", "kicad_pcb", "kicad_sch", "kicad_mod", "kicad_sym", "parse kicad", "offline kicad", "kicad scripting"
---

# kiutils - KiCad File Manipulation Library

kiutils is a Python library for parsing, creating, and modifying KiCad design files **without requiring a running KiCad instance**. It works directly with the S-expression file formats used by KiCad 6+.

## Environment Setup

**IMPORTANT:** kiutils is installed in a Poetry virtual environment. Use the venv Python interpreter directly to run scripts from any project directory:

```bash
# Define the kiutils Python interpreter
KIUTILS_PYTHON="/path/to/kiutils"

# Run a script from your current project directory
$KIUTILS_PYTHON my_kicad_script.py

# Run inline Python (works from any directory)
$KIUTILS_PYTHON -c "from kiutils.board import Board; board = Board.from_file('project.kicad_pcb'); print(len(board.footprints))"

# Interactive shell with kiutils available
$KIUTILS_PYTHON
```

The kiutils environment is located at: `/path/to/kiutils/`

**Note:** File paths in your scripts are relative to your current working directory, not the venv location.

## Quick Start

The universal pattern for all kiutils operations:

```python
from kiutils.board import Board

# Load file
board = Board.from_file("project.kicad_pcb")

# Modify (add track, footprint, etc.)
# ...

# Save (overwrites original, or specify new path)
board.to_file()
# or: board.to_file("output.kicad_pcb")
```

This pattern applies to ALL kiutils classes:
- `Board.from_file()` / `board.to_file()`
- `Schematic.from_file()` / `schematic.to_file()`
- `Footprint.from_file()` / `footprint.to_file()`
- `SymbolLib.from_file()` / `symbollib.to_file()`
- `DesignRules.from_file()` / `designrules.to_file()`
- `FootprintTable.from_file()` / `fptable.to_file()`
- `SymbolTable.from_file()` / `symtable.to_file()`
- `WorkSheet.from_file()` / `worksheet.to_file()`

## Critical Concepts

### Units: Millimeters Throughout

kiutils uses **millimeters** for all dimensions. No unit conversion needed:

```python
from kiutils.items.common import Position

# Position is in millimeters, angle in degrees
pos = Position(X=10.0, Y=20.0, angle=45.0)
```

Compare to kipy (KiCad IPC) which uses nanometers:
- kiutils: `x = 10.0` (10mm)
- kipy: `x = 10_000_000` (10mm in nm)

### Net References: Numbers Not Objects

**Critical gotcha**: Track segments, vias, and pads reference nets by NUMBER, not by Net object:

```python
# CORRECT: Find net, use its number
gnd_net = next((n for n in board.nets if n.name == "GND"), None)
segment.net = gnd_net.number  # Use the number!

# WRONG: This won't work
segment.net = gnd_net  # Don't assign the Net object
```

### Layers: String Names

Layers are referenced by their string names:

```python
# Copper layers
"F.Cu", "B.Cu", "In1.Cu", "In2.Cu"

# Technical layers
"F.SilkS", "B.SilkS", "F.Mask", "B.Mask", "F.Paste", "B.Paste"

# Mechanical layers
"Edge.Cuts", "Dwgs.User", "Cmts.User", "F.CrtYd", "B.CrtYd", "F.Fab", "B.Fab"
```

### File Preservation

kiutils attempts to preserve file structure but:
- Whitespace/formatting may change on save
- Comments in the original file are NOT preserved
- Version-specific tokens may be updated

## Supported File Types

| Extension | Class | Import |
|-----------|-------|--------|
| `.kicad_pcb` | Board | `from kiutils.board import Board` |
| `.kicad_sch` | Schematic | `from kiutils.schematic import Schematic` |
| `.kicad_mod` | Footprint | `from kiutils.footprint import Footprint` |
| `.kicad_sym` | SymbolLib | `from kiutils.symbol import SymbolLib` |
| `.kicad_dru` | DesignRules | `from kiutils.dru import DesignRules` |
| `fp-lib-table` | FootprintTable | `from kiutils.libraries import FootprintTable` |
| `sym-lib-table` | SymbolTable | `from kiutils.libraries import SymbolTable` |
| `.kicad_wks` | WorkSheet | `from kiutils.wks import WorkSheet` |

## Board Operations

### Loading and Saving

```python
from kiutils.board import Board

board = Board.from_file("project.kicad_pcb")
# ... modifications ...
board.to_file()  # Save to same file
board.to_file("modified.kicad_pcb")  # Save to new file
```

### Working with Nets

```python
# List all nets
for net in board.nets:
    print(f"Net {net.number}: {net.name}")

# Find a specific net
def get_net_number(board, net_name):
    """Get net number by name (required for assigning to traces)"""
    net = next((n for n in board.nets if n.name == net_name), None)
    return net.number if net else None

vcc_num = get_net_number(board, "VCC")
```

### Adding Track Segments

```python
from kiutils.items.brditems import Segment
from kiutils.items.common import Position

# Create a track segment
segment = Segment(
    start=Position(X=100.0, Y=50.0),
    end=Position(X=120.0, Y=50.0),
    width=0.25,
    layer="F.Cu",
    net=gnd_num  # Net NUMBER, not object
)
board.traceItems.append(segment)
```

### Adding Arc Traces

```python
from kiutils.items.brditems import Arc

arc = Arc(
    start=Position(X=100.0, Y=50.0),
    mid=Position(X=105.0, Y=45.0),  # Midpoint of arc
    end=Position(X=110.0, Y=50.0),
    width=0.25,
    layer="F.Cu",
    net=net_num
)
board.traceItems.append(arc)
```

### Adding Vias

```python
from kiutils.items.brditems import Via

via = Via(
    position=Position(X=110.0, Y=50.0),
    size=0.8,
    drill=0.4,
    layers=["F.Cu", "B.Cu"],  # Through-hole via
    net=net_num
)
board.traceItems.append(via)
```

### Board Graphics

```python
from kiutils.items.gritems import GrLine, GrRect, GrCircle, GrText, GrArc, GrPoly
from kiutils.items.common import Position, Stroke

# Line
line = GrLine(
    start=Position(X=0, Y=0),
    end=Position(X=10, Y=0),
    layer="Edge.Cuts",
    stroke=Stroke(width=0.1)
)
board.graphicItems.append(line)

# Rectangle
rect = GrRect(
    start=Position(X=0, Y=0),
    end=Position(X=50, Y=30),
    layer="Dwgs.User",
    stroke=Stroke(width=0.15)
)
board.graphicItems.append(rect)

# Circle
circle = GrCircle(
    center=Position(X=25, Y=15),
    end=Position(X=30, Y=15),  # Point on circumference
    layer="F.SilkS",
    stroke=Stroke(width=0.12)
)
board.graphicItems.append(circle)

# Text
from kiutils.items.common import Effects, Font

text = GrText(
    text="LABEL",
    position=Position(X=10, Y=5, angle=0),
    layer="F.SilkS",
    effects=Effects(font=Font(size=1.0, thickness=0.15))
)
board.graphicItems.append(text)
```

### Working with Footprints

```python
# Iterate footprints on board
for fp in board.footprints:
    print(f"{fp.entryName} at ({fp.position.X}, {fp.position.Y})")

    # Access pads
    for pad in fp.pads:
        print(f"  Pad {pad.number}: net {pad.net}")

# Move a footprint
fp = board.footprints[0]
fp.position.X += 10.0
fp.position.Y += 5.0

# Rotate a footprint
fp.position.angle = 90.0
```

### Zones (Copper Pours)

```python
from kiutils.items.zones import Zone, ZoneFillSettings, ZoneKeepoutSettings

# Access existing zones
for zone in board.zones:
    print(f"Zone: net={zone.netName}, layer={zone.layer}")

# Zones have polygon outlines
zone = board.zones[0]
for point in zone.polygon.coordinates:
    print(f"  ({point.X}, {point.Y})")
```

## Schematic Operations

### Loading Schematics

```python
from kiutils.schematic import Schematic

sch = Schematic.from_file("project.kicad_sch")
```

### Working with Symbols

```python
# Iterate schematic symbols
for sym in sch.schematicSymbols:
    ref = sym.property("Reference")
    value = sym.property("Value")
    print(f"{ref}: {value}")

# Access symbol position
for sym in sch.schematicSymbols:
    pos = sym.position
    print(f"Symbol at ({pos.X}, {pos.Y}), angle={pos.angle}")
```

### Wires and Junctions

Wires are `Connection` objects stored in `graphicalItems`:

```python
from kiutils.schematic import Schematic, Connection, Junction, Position, Stroke

# Access wires (Connection objects with type='wire')
for item in sch.graphicalItems:
    if isinstance(item, Connection) and item.type == 'wire':
        pts = item.points  # List of Position objects
        print(f"Wire: {[(p.X, p.Y) for p in pts]}")

# Create a wire
wire = Connection(
    type='wire',
    points=[Position(X=50.8, Y=50.8), Position(X=101.6, Y=50.8)],
    stroke=Stroke(width=0, type='default'),
    uuid="unique-id"
)
sch.graphicalItems.append(wire)

# Access junctions
for junction in sch.junctions:
    print(f"Junction at ({junction.position.X}, {junction.position.Y})")

# Create a junction
junction = Junction(
    position=Position(X=76.2, Y=50.8),
    diameter=0
)
sch.junctions.append(junction)
```

### Labels

Labels are `LocalLabel` for local and `GlobalLabel` for global:

```python
from kiutils.schematic import LocalLabel, GlobalLabel, Position, Effects

# Local labels (same sheet)
for label in sch.labels:
    print(f"Label: {label.text}")

# Create local label
label = LocalLabel(
    text="SIGNAL_NAME",
    position=Position(X=88.9, Y=50.8, angle=0),
    effects=Effects()
)
sch.labels.append(label)

# Global labels (cross-sheet)
for glabel in sch.globalLabels:
    print(f"Global: {glabel.text} ({glabel.shape})")

# Create global label (shapes: input, output, bidirectional, tri_state, passive)
glabel = GlobalLabel(
    text="VCC",
    position=Position(X=127.0, Y=50.8, angle=0),
    shape="input",
    effects=Effects()
)
sch.globalLabels.append(glabel)

# Hierarchical labels
for hlabel in sch.hierarchicalLabels:
    print(f"Hierarchical: {hlabel.text}")
```

### No-Connect Markers

```python
from kiutils.schematic import NoConnect, Position

# Add no-connect marker
nc = NoConnect(
    position=Position(X=50.8, Y=50.8)
)
sch.noConnects.append(nc)
```

### Hierarchical Sheets

```python
# Access sheet instances
for sheet in sch.sheets:
    print(f"Sheet: {sheet.sheetName}")
    for pin in sheet.pins:
        print(f"  Pin: {pin.name}")
```

### BOM Generation

```python
def generate_bom(schematic_path):
    """Generate simple BOM from schematic"""
    sch = Schematic.from_file(schematic_path)
    bom = []

    for sym in sch.schematicSymbols:
        ref = sym.property("Reference")
        value = sym.property("Value")
        footprint = sym.property("Footprint")

        # Skip power symbols
        if ref and not ref.startswith("#"):
            bom.append({
                "Reference": ref,
                "Value": value,
                "Footprint": footprint
            })

    return sorted(bom, key=lambda x: x["Reference"])
```

## Footprint Operations

### Loading/Creating Footprints

```python
from kiutils.footprint import Footprint

# Load existing footprint
fp = Footprint.from_file("MyPart.kicad_mod")

# Create new footprint
fp = Footprint()
fp.entryName = "MyNewPart"
fp.layer = "F.Cu"
```

### Adding Pads

```python
from kiutils.items.fpitems import Pad, DrillDefinition
from kiutils.items.common import Position

# SMD pad
smd_pad = Pad(
    number="1",
    type="smd",
    shape="rect",
    position=Position(X=-1.27, Y=0),
    size=Position(X=1.5, Y=0.9),  # Size uses Position class
    layers=["F.Cu", "F.Paste", "F.Mask"]
)
fp.pads.append(smd_pad)

# Through-hole pad
th_pad = Pad(
    number="1",
    type="thru_hole",
    shape="circle",
    position=Position(X=0, Y=0),
    size=Position(X=1.7, Y=1.7),
    drill=DrillDefinition(diameter=1.0),
    layers=["*.Cu", "*.Mask"]
)
fp.pads.append(th_pad)
```

### Pad Types and Shapes

```python
# Pad types
"smd"          # Surface mount
"thru_hole"    # Through-hole
"np_thru_hole" # Non-plated through-hole
"connect"      # Edge connector

# Pad shapes
"circle"       # Round
"rect"         # Rectangle
"oval"         # Oval/oblong
"trapezoid"    # Trapezoid
"roundrect"    # Rounded rectangle
"custom"       # Custom shape
```

### Footprint Graphics

```python
from kiutils.items.fpitems import FpLine, FpRect, FpCircle, FpText, FpArc
from kiutils.items.common import Position, Stroke

# Silkscreen outline
line = FpLine(
    start=Position(X=-2.5, Y=-1.5),
    end=Position(X=2.5, Y=-1.5),
    layer="F.SilkS",
    stroke=Stroke(width=0.12)
)
fp.graphicItems.append(line)

# Courtyard
rect = FpRect(
    start=Position(X=-3.0, Y=-2.0),
    end=Position(X=3.0, Y=2.0),
    layer="F.CrtYd",
    stroke=Stroke(width=0.05)
)
fp.graphicItems.append(rect)

# Reference designator
ref_text = FpText(
    type="reference",
    text="REF**",
    position=Position(X=0, Y=-2.5),
    layer="F.SilkS"
)
fp.graphicItems.append(ref_text)

# Value
val_text = FpText(
    type="value",
    text="MyPart",
    position=Position(X=0, Y=2.5),
    layer="F.Fab"
)
fp.graphicItems.append(val_text)
```

### 3D Model Attachment

```python
from kiutils.items.common import Coordinate

fp.models.append({
    "path": "${KICAD7_3DMODEL_DIR}/Package_SO.3dshapes/SOIC-8.wrl",
    "offset": Coordinate(X=0, Y=0, Z=0),
    "scale": Coordinate(X=1, Y=1, Z=1),
    "rotate": Coordinate(X=0, Y=0, Z=0)
})
```

### Saving Footprints

```python
fp.to_file("MyNewPart.kicad_mod")
```

## Symbol Operations

### Loading Symbol Libraries

```python
from kiutils.symbol import SymbolLib, Symbol

# Load library
lib = SymbolLib.from_file("MySymbols.kicad_sym")

# Access symbols
for sym in lib.symbols:
    print(f"Symbol: {sym.entryName}")
```

### Creating Symbols

```python
from kiutils.symbol import Symbol, SymbolPin
from kiutils.items.syitems import SyRect, SyCircle, SyPolyline, SyText
from kiutils.items.common import Position

# Create new symbol
sym = Symbol(entryName="MySymbol")

# Add graphic body (rectangle)
body = SyRect(
    start=Position(X=-5.08, Y=5.08),
    end=Position(X=5.08, Y=-5.08),
    strokeWidth=0.254
)
sym.graphic.append(body)
```

### Symbol Pins

```python
# Add pins
pin1 = SymbolPin(
    electricType="input",
    graphicStyle="line",
    position=Position(X=-7.62, Y=2.54, angle=0),
    length=2.54,
    name="IN",
    number="1"
)
sym.pins.append(pin1)

pin2 = SymbolPin(
    electricType="output",
    graphicStyle="line",
    position=Position(X=7.62, Y=2.54, angle=180),
    length=2.54,
    name="OUT",
    number="2"
)
sym.pins.append(pin2)
```

### Pin Electrical Types

```python
"input"           # Input pin
"output"          # Output pin
"bidirectional"   # Bidirectional
"tri_state"       # Tri-state
"passive"         # Passive (resistors, caps)
"power_in"        # Power input (VCC, GND)
"power_out"       # Power output
"open_collector"  # Open collector
"open_emitter"    # Open emitter
"unconnected"     # No connect
"free"            # Unspecified
```

### Pin Graphic Styles

```python
"line"            # Standard line
"inverted"        # With bubble (active low)
"clock"           # Clock input
"inverted_clock"  # Inverted clock
"input_low"       # Active low input
"clock_low"       # Active low clock
"output_low"      # Active low output
"edge_clock_high" # Edge-triggered clock
"non_logic"       # Non-logic (power, etc.)
```

### Saving Symbol Libraries

```python
# Add symbol to library
lib.symbols.append(sym)

# Save
lib.to_file("MySymbols.kicad_sym")
```

## Library Table Operations

### Footprint Library Table

```python
from kiutils.libraries import FootprintTable, LibraryEntry

# Load fp-lib-table
fp_table = FootprintTable.from_file("fp-lib-table")

# Add new library
new_lib = LibraryEntry(
    name="MyFootprints",
    type="KiCad",
    uri="${KIPRJMOD}/MyFootprints.pretty",
    options="",
    descr="My custom footprints"
)
fp_table.libs.append(new_lib)

# Save
fp_table.to_file()
```

### Symbol Library Table

```python
from kiutils.libraries import SymbolTable, LibraryEntry

# Load sym-lib-table
sym_table = SymbolTable.from_file("sym-lib-table")

# Add new library
new_lib = LibraryEntry(
    name="MySymbols",
    type="KiCad",
    uri="${KIPRJMOD}/MySymbols.kicad_sym",
    options="",
    descr="My custom symbols"
)
sym_table.libs.append(new_lib)

sym_table.to_file()
```

## Design Rules Operations

```python
from kiutils.dru import DesignRules

# Load design rules
dru = DesignRules.from_file("project.kicad_dru")

# Access rules
for rule in dru.rules:
    print(f"Rule: {rule.name}")
    for constraint in rule.constraints:
        print(f"  {constraint}")

# Modify and save
dru.to_file()
```

## Common Gotchas

### 1. Net Assignment

```python
# WRONG - assigning Net object
segment.net = board.nets[0]

# CORRECT - assigning net number
segment.net = board.nets[0].number
```

### 2. Position Object Required

```python
# WRONG - tuple
segment.start = (10, 20)

# CORRECT - Position object
segment.start = Position(X=10, Y=20)
```

### 3. Size Uses Position Class

```python
# Pad size uses Position(X=width, Y=height)
pad.size = Position(X=1.5, Y=0.9)
```

### 4. Layer Names Are Strings

```python
# WRONG - using layer numbers
segment.layer = 0

# CORRECT - using layer names
segment.layer = "F.Cu"
```

### 5. Circle End Point

```python
# Circle "end" is a point on the circumference, not diameter/radius
circle = GrCircle(
    center=Position(X=10, Y=10),
    end=Position(X=15, Y=10)  # Radius = 5mm
)
```

### 6. Symbol Properties

```python
# Access symbol property values
ref = symbol.property("Reference")  # Returns value or None
val = symbol.property("Value")

# This searches the properties list for matching key
```

### 7. Append to Lists

```python
# Items are stored in lists - append to add
board.traceItems.append(segment)
board.graphicItems.append(line)
fp.pads.append(pad)
```

## Comparison with kipy

| Aspect | kiutils | kipy |
|--------|---------|------|
| Requires KiCad | No | Yes (running instance) |
| Connection | File I/O | IPC socket |
| Units | Millimeters | Nanometers |
| Transactions | None | Commit required |
| File types | All (.pcb, .sch, .mod, .sym) | PCB only |
| Use case | Batch processing, offline | Live manipulation |
| Install | `pip install kiutils` | Included in KiCad |

## API Documentation

Load the appropriate API reference based on the task domain:

| Task Type | File Extension | Read First |
|-----------|----------------|------------|
| **Schematics** | .kicad_sch | `api/schematics_apis.md` |
| **Board/PCB** | .kicad_pcb | `api/board_apis.md` |
| **Footprints** | .kicad_mod | `api/footprint_apis.md` |
| **Symbols** | .kicad_sym | `api/symbol_apis.md` |
| **Design Rules** | .kicad_dru | `api/dru_apis.md` |
| **Worksheets** | .kicad_wks | `api/wks_apis.md` |
| **Library Tables** | fp_lib_table / sym_lib_table | `api/libraries_apis.md` |

### Schematics API (`api/schematics_apis.md`)

- Schematic class with all attributes
- Schematic items: Junction, Connection, Labels (Local, Global, Hierarchical)
- Graphical shapes: Text, TextBox, Rectangle, Circle, Arc
- Hierarchical sheets and pins
- Common patterns: BOM generation, wire networks, symbol search

### Board/PCB API (`api/board_apis.md`)

- Board class with all attributes
- Trace items: Segment, Via, Arc, Target
- Graphic items: GrText, GrLine, GrRect, GrCircle, GrArc, GrPoly, GrCurve
- Zone classes: Zone, FillSettings, KeepoutSettings
- Layer and stackup configuration
- Common patterns: board outlines, copper pours, net lookup

### Footprint API (`api/footprint_apis.md`)

- Footprint class with all attributes (manufacturing, position, path)
- Pad class: SMD, through-hole, drill definitions, shapes
- Footprint graphics: FpText, FpLine, FpRect, FpCircle, FpArc, FpPoly, FpCurve
- 3D model attachment
- Common patterns: SMD resistor, through-hole connector, batch processing

### Symbol API (`api/symbol_apis.md`)

- SymbolLib and Symbol classes with all attributes
- SymbolPin: electrical types, graphical styles, alternate pins
- Symbol graphics: SyRect, SyCircle, SyArc, SyPolyLine, SyCurve, SyText, SyTextBox
- Multi-unit symbol creation
- Common patterns: IC symbols, power symbols, BOM from library

### Design Rules API (`api/dru_apis.md`)

- DesignRules and Rule classes
- Constraint types: clearance, track_width, via_diameter, disallow, length, skew
- Condition expressions: NetClass, NetName, Type, Layer, functions
- Common patterns: high voltage clearance, differential pairs, BGA areas

### Worksheet API (`api/wks_apis.md`)

- WorkSheet and Setup classes
- Drawing objects: TbText, Line, Rect, Polygon, Bitmap
- Position with corner references (ltcorner, rbcorner, etc.)
- Text variables: ${TITLE}, ${DATE}, ${REVISION}, ${#}, ${##}
- Common patterns: title blocks, borders, company logos

### Library Table API (`api/libraries_apis.md`)

- LibTable and Library classes
- Library types: KiCad, Legacy, Eagle, Altium
- Path variables: ${KIPRJMOD}, ${KICAD7_FOOTPRINT_DIR}, etc.
- Common patterns: add/remove libraries, migrate paths, merge tables

## Items Package Reference

For detailed class-level API docs on shared item types used across boards, schematics, footprints, and symbols:

| Use Case | Read File |
|----------|-----------|
| Positioning, styling, fonts, colors | `api/items/common_apis.md` |
| PCB tracks, vias, arcs, stackup | `api/items/brditems_apis.md` |
| Board/schematic graphics (text, lines, shapes) | `api/items/gritems_apis.md` |
| Footprint graphics and pads | `api/items/fpitems_apis.md` |
| Symbol body graphics | `api/items/syitems_apis.md` |
| Schematic wires, junctions, labels, symbols | `api/items/schitems_apis.md` |
| Copper zones and keepouts | `api/items/zones_apis.md` |
| PCB dimension annotations | `api/items/dimensions_apis.md` |

### When to Use Items APIs

**Always start with the file-level API** (`api/board_apis.md`, `api/schematic_apis.md`, etc.) for high-level operations. **Then reference items APIs** when you need detailed information about:

- Creating specific item types (tracks, text, shapes)
- Understanding attribute defaults and valid values
- Working with styling (stroke, fill, effects)
- Understanding coordinate/position conventions
