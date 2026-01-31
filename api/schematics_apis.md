# Kiutils Schematics API Reference

Complete API reference for all classes in `kiutils.schematic` and `kiutils.items.schitems` for manipulating `.kicad_sch` files.

## Table of Contents

1. [Schematic Class](#schematic-class)
2. [Symbol Classes](#symbol-classes)
3. [Connection Classes](#connection-classes)
4. [Label Classes](#label-classes)
5. [Graphical Shape Classes](#graphical-shape-classes)
6. [Hierarchical Sheet Classes](#hierarchical-sheet-classes)
7. [Instance Classes](#instance-classes)
8. [Common Patterns](#common-patterns)

---

## Schematic Class

Main class: `kiutils.schematic.Schematic`

### Import and Load

```python
from kiutils.schematic import Schematic

# Load from file
sch = Schematic.from_file("project.kicad_sch")

# Create empty schematic
sch = Schematic.create_new()

# Parse from string
sch = Schematic.from_sexpr(sexpr_list)
```

### Save

```python
sch.to_file()                    # Save to original path
sch.to_file("output.kicad_sch")  # Save to new path
sexpr = sch.to_sexpr()           # Get S-expression string
```

### Schematic Attributes

| Attribute | Type | Description |
|-----------|------|-------------|
| `version` | str | File version (YYYYMMDD format) |
| `generator` | str | Creating tool identifier |
| `uuid` | str | Universally unique identifier |
| `paper` | PageSettings | Paper size settings |
| `titleBlock` | TitleBlock | Title block info (author, date, rev) |
| `libSymbols` | list[Symbol] | Embedded symbol definitions |
| `schematicSymbols` | list[SchematicSymbol] | Symbol instances on schematic |
| `junctions` | list[Junction] | Wire junctions |
| `noConnects` | list[NoConnect] | No-connect markers |
| `busEntries` | list[BusEntry] | Bus entry connectors |
| `busAliases` | list[BusAlias] | Bus alias definitions |
| `graphicalItems` | list[Connection\|PolyLine] | Wires, buses, polylines |
| `shapes` | list[Arc\|Circle\|Rectangle] | Graphical shapes (v7+) |
| `images` | list[Image] | Embedded images |
| `texts` | list[Text] | Text annotations |
| `textBoxes` | list[TextBox] | Text boxes (v7+) |
| `labels` | list[LocalLabel] | Local wire labels |
| `globalLabels` | list[GlobalLabel] | Global labels (cross-sheet) |
| `hierarchicalLabels` | list[HierarchicalLabel] | Hierarchical labels |
| `netclassFlags` | list[NetclassFlag] | Netclass flags (v7+) |
| `sheets` | list[HierarchicalSheet] | Sub-sheet definitions |
| `sheetInstances` | list[HierarchicalSheetInstance] | Sheet instance paths |
| `symbolInstances` | list[SymbolInstance] | Symbol instance data |
| `filePath` | str | File path (set by from_file) |

---

## Symbol Classes

### SchematicSymbol

Symbol instance placed on schematic.

```python
from kiutils.items.schitems import SchematicSymbol

# Access symbols
for sym in sch.schematicSymbols:
    print(f"Library: {sym.libraryNickname}")
    print(f"Entry: {sym.entryName}")
    print(f"Full ID: {sym.libId}")
    print(f"Position: ({sym.position.X}, {sym.position.Y})")
```

#### SchematicSymbol Attributes

| Attribute | Type | Description |
|-----------|------|-------------|
| `libId` | str | Library:symbol ID (property) |
| `libraryNickname` | str | Library name part of ID |
| `entryName` | str | Symbol name part of ID |
| `libName` | str | Modified symbol name (when edited) |
| `position` | Position | X, Y, angle |
| `unit` | int | Symbol unit number (for multi-unit) |
| `inBom` | bool | Include in BOM |
| `onBoard` | bool | Include on PCB |
| `dnp` | bool | Do-not-populate flag (v7+) |
| `fieldsAutoplaced` | bool | Auto-placed fields |
| `uuid` | str | Unique identifier |
| `properties` | list[Property] | Symbol properties |
| `pins` | dict[str, str] | Pin number to UUID mapping |
| `mirror` | str | Mirror mode: "x" or "y" |
| `instances` | list[SymbolProjectInstance] | Project instances (v7+) |

#### Accessing Properties

```python
# Get property value by name (returns None if not found)
ref = sym.property("Reference")      # e.g., "R1"
value = sym.property("Value")        # e.g., "10k"
footprint = sym.property("Footprint")
datasheet = sym.property("Datasheet")

# Iterate all properties
for prop in sym.properties:
    print(f"{prop.key}: {prop.value}")

# Modify property
for prop in sym.properties:
    if prop.key == "Value":
        prop.value = "22k"
        break
```

### SymbolProjectInstance

Project-specific instance data (v7+).

```python
from kiutils.items.schitems import SymbolProjectInstance, SymbolProjectPath

# Attributes
instance.name               # Project name
instance.paths              # list[SymbolProjectPath]
```

### SymbolProjectPath

Path to symbol instance within project.

```python
# Attributes
path.sheetInstancePath      # Path string (e.g., "/")
path.reference              # Reference designator
path.unit                   # Unit number
```

---

## Connection Classes

### Connection

Wires and buses.

```python
from kiutils.items.schitems import Connection
from kiutils.items.common import Position, Stroke
import uuid

# Filter wires from graphicalItems
for item in sch.graphicalItems:
    if isinstance(item, Connection):
        if item.type == 'wire':
            print(f"Wire: {[(p.X, p.Y) for p in item.points]}")
        elif item.type == 'bus':
            print(f"Bus: {[(p.X, p.Y) for p in item.points]}")

# Create wire
wire = Connection(
    type='wire',
    points=[
        Position(X=50.8, Y=50.8),
        Position(X=101.6, Y=50.8)
    ],
    stroke=Stroke(width=0, type='default'),
    uuid=str(uuid.uuid4())
)
sch.graphicalItems.append(wire)

# Create bus
bus = Connection(
    type='bus',
    points=[
        Position(X=100, Y=100),
        Position(X=200, Y=100)
    ],
    stroke=Stroke(width=0, type='default'),
    uuid=str(uuid.uuid4())
)
sch.graphicalItems.append(bus)
```

#### Connection Attributes

| Attribute | Type | Description |
|-----------|------|-------------|
| `type` | str | "wire" or "bus" |
| `points` | list[Position] | Coordinates (min 2) |
| `stroke` | Stroke | Line style |
| `uuid` | str | Unique identifier |

### PolyLine

Graphical polyline.

```python
from kiutils.items.schitems import PolyLine

polyline = PolyLine(
    points=[
        Position(X=0, Y=0),
        Position(X=10, Y=0),
        Position(X=10, Y=10)
    ],
    stroke=Stroke(width=0.15),
    uuid=str(uuid.uuid4())
)
sch.graphicalItems.append(polyline)
```

#### PolyLine Attributes

| Attribute | Type | Description |
|-----------|------|-------------|
| `points` | list[Position] | List of coordinates |
| `stroke` | Stroke | Line style |
| `uuid` | str | Unique identifier |

### Junction

Wire junction point.

```python
from kiutils.items.schitems import Junction
from kiutils.items.common import Position, ColorRGBA

# Access junctions
for junc in sch.junctions:
    print(f"Junction at ({junc.position.X}, {junc.position.Y})")

# Create junction
junction = Junction(
    position=Position(X=76.2, Y=50.8),
    diameter=0,  # 0 = default size
    color=ColorRGBA(),
    uuid=str(uuid.uuid4())
)
sch.junctions.append(junction)
```

#### Junction Attributes

| Attribute | Type | Description |
|-----------|------|-------------|
| `position` | Position | X, Y coordinates |
| `diameter` | float | Diameter (0 = default) |
| `color` | ColorRGBA | Junction color (0,0,0,0 = default) |
| `uuid` | str | Unique identifier |

### NoConnect

No-connect marker.

```python
from kiutils.items.schitems import NoConnect

# Access
for nc in sch.noConnects:
    print(f"NC at ({nc.position.X}, {nc.position.Y})")

# Create
nc = NoConnect(
    position=Position(X=50.8, Y=50.8),
    uuid=str(uuid.uuid4())
)
sch.noConnects.append(nc)
```

#### NoConnect Attributes

| Attribute | Type | Description |
|-----------|------|-------------|
| `position` | Position | X, Y coordinates |
| `uuid` | str | Unique identifier |

### BusEntry

Bus entry connector.

```python
from kiutils.items.schitems import BusEntry

# Access
for entry in sch.busEntries:
    print(f"Entry at ({entry.position.X}, {entry.position.Y})")
    print(f"Size: ({entry.size.X}, {entry.size.Y})")

# Create
entry = BusEntry(
    position=Position(X=100, Y=100),
    size=Position(X=2.54, Y=2.54),
    stroke=Stroke(width=0),
    uuid=str(uuid.uuid4())
)
sch.busEntries.append(entry)
```

#### BusEntry Attributes

| Attribute | Type | Description |
|-----------|------|-------------|
| `position` | Position | Entry position |
| `size` | Position | X/Y distance to endpoint |
| `stroke` | Stroke | Line style |
| `uuid` | str | Unique identifier |

### BusAlias

Named bus alias.

```python
from kiutils.items.schitems import BusAlias

# Access
for alias in sch.busAliases:
    print(f"Bus: {alias.name}, Members: {alias.members}")

# Create
alias = BusAlias(
    name="DATA_BUS",
    members=["D0", "D1", "D2", "D3", "D4", "D5", "D6", "D7"]
)
sch.busAliases.append(alias)
```

#### BusAlias Attributes

| Attribute | Type | Description |
|-----------|------|-------------|
| `name` | str | Bus alias name |
| `members` | list[str] | Member signal names |

---

## Label Classes

### LocalLabel

Local wire/bus label (same sheet only).

```python
from kiutils.items.schitems import LocalLabel
from kiutils.items.common import Position, Effects

# Access
for label in sch.labels:
    print(f"Label: {label.text}")

# Create
label = LocalLabel(
    text="NET_NAME",
    position=Position(X=100, Y=50, angle=0),
    effects=Effects(),
    uuid=str(uuid.uuid4()),
    fieldsAutoplaced=False
)
sch.labels.append(label)
```

#### LocalLabel Attributes

| Attribute | Type | Description |
|-----------|------|-------------|
| `text` | str | Label text |
| `position` | Position | X, Y, angle |
| `effects` | Effects | Text formatting |
| `uuid` | str | Unique identifier |
| `fieldsAutoplaced` | bool | Auto-placed flag |

### GlobalLabel

Global label (cross-sheet connection).

```python
from kiutils.items.schitems import GlobalLabel

# Access
for glabel in sch.globalLabels:
    print(f"Global: {glabel.text} ({glabel.shape})")

# Create
glabel = GlobalLabel(
    text="VCC",
    shape="input",  # See shapes below
    position=Position(X=127.0, Y=50.8, angle=0),
    effects=Effects(),
    uuid=str(uuid.uuid4()),
    fieldsAutoplaced=False,
    properties=[]
)
sch.globalLabels.append(glabel)
```

#### GlobalLabel Attributes

| Attribute | Type | Description |
|-----------|------|-------------|
| `text` | str | Label text |
| `shape` | str | Visual shape (see table) |
| `position` | Position | X, Y, angle |
| `effects` | Effects | Text formatting |
| `uuid` | str | Unique identifier |
| `fieldsAutoplaced` | bool | Auto-placed flag |
| `properties` | list[Property] | Additional properties |

#### Label Shapes

| Shape | Description |
|-------|-------------|
| `"input"` | Input arrow pointing in |
| `"output"` | Output arrow pointing out |
| `"bidirectional"` | Diamond shape |
| `"tri_state"` | Tri-state marker |
| `"passive"` | Rectangle (passive signal) |

### HierarchicalLabel

Label for hierarchical sheet connections.

```python
from kiutils.items.schitems import HierarchicalLabel

# Access
for hlabel in sch.hierarchicalLabels:
    print(f"Hierarchical: {hlabel.text} ({hlabel.shape})")

# Create
hlabel = HierarchicalLabel(
    text="DATA_BUS",
    shape="bidirectional",
    position=Position(X=100, Y=50, angle=0),
    effects=Effects(),
    uuid=str(uuid.uuid4()),
    fieldsAutoplaced=False
)
sch.hierarchicalLabels.append(hlabel)
```

#### HierarchicalLabel Attributes

| Attribute | Type | Description |
|-----------|------|-------------|
| `text` | str | Label text |
| `shape` | str | Visual shape (same as GlobalLabel) |
| `position` | Position | X, Y, angle |
| `effects` | Effects | Text formatting |
| `uuid` | str | Unique identifier |
| `fieldsAutoplaced` | bool | Auto-placed flag |

---

## Graphical Shape Classes

### Text

Graphical text annotation.

```python
from kiutils.items.schitems import Text
from kiutils.items.common import Position, Effects

# Access
for txt in sch.texts:
    print(f"Text: {txt.text}")

# Create
text = Text(
    text="Design Notes",
    position=Position(X=50, Y=200, angle=0),
    effects=Effects(),
    uuid=str(uuid.uuid4())
)
sch.texts.append(text)
```

#### Text Attributes

| Attribute | Type | Description |
|-----------|------|-------------|
| `text` | str | Text content |
| `position` | Position | X, Y, angle |
| `effects` | Effects | Text formatting |
| `uuid` | str | Unique identifier |

### TextBox

Text box with border (v7+).

```python
from kiutils.items.schitems import TextBox
from kiutils.items.common import Position, Stroke, Fill, Effects

textbox = TextBox(
    text="Important Note",
    position=Position(X=50, Y=50, angle=0),
    size=Position(X=30, Y=20),  # Width, Height
    stroke=Stroke(width=0.15),
    fill=Fill(type="none"),
    effects=Effects(),
    uuid=str(uuid.uuid4())
)
sch.textBoxes.append(textbox)
```

#### TextBox Attributes

| Attribute | Type | Description |
|-----------|------|-------------|
| `text` | str | Text content |
| `position` | Position | X, Y, angle |
| `size` | Position | Width (X), Height (Y) |
| `stroke` | Stroke | Border style |
| `fill` | Fill | Fill style |
| `effects` | Effects | Text formatting |
| `uuid` | str | Unique identifier |

### Rectangle

Graphical rectangle (v7+).

```python
from kiutils.items.schitems import Rectangle
from kiutils.items.common import Position, Stroke, Fill

rect = Rectangle(
    start=Position(X=10, Y=10),
    end=Position(X=50, Y=30),
    stroke=Stroke(width=0.15),
    fill=Fill(type="none"),
    uuid=str(uuid.uuid4())
)
sch.shapes.append(rect)
```

#### Rectangle Attributes

| Attribute | Type | Description |
|-----------|------|-------------|
| `start` | Position | Start corner |
| `end` | Position | End corner |
| `stroke` | Stroke | Border style |
| `fill` | Fill | Fill style |
| `uuid` | str | Unique identifier |

### Circle

Graphical circle (v7+).

```python
from kiutils.items.schitems import Circle

circle = Circle(
    center=Position(X=25, Y=25),
    radius=10.0,
    stroke=Stroke(width=0.15),
    fill=Fill(type="none"),
    uuid=str(uuid.uuid4())
)
sch.shapes.append(circle)
```

#### Circle Attributes

| Attribute | Type | Description |
|-----------|------|-------------|
| `center` | Position | Center point |
| `radius` | float | Radius in mm |
| `stroke` | Stroke | Border style |
| `fill` | Fill | Fill style |
| `uuid` | str | Unique identifier |

### Arc

Graphical arc (v7+).

```python
from kiutils.items.schitems import Arc

arc = Arc(
    start=Position(X=10, Y=20),
    mid=Position(X=15, Y=15),
    end=Position(X=20, Y=20),
    stroke=Stroke(width=0.15),
    fill=Fill(type="none"),
    uuid=str(uuid.uuid4())
)
sch.shapes.append(arc)
```

#### Arc Attributes

| Attribute | Type | Description |
|-----------|------|-------------|
| `start` | Position | Start point |
| `mid` | Position | Midpoint on arc |
| `end` | Position | End point |
| `stroke` | Stroke | Border style |
| `fill` | Fill | Fill style |
| `uuid` | str | Unique identifier |

### NetclassFlag

Netclass assignment flag (v7+).

```python
from kiutils.items.schitems import NetclassFlag

flag = NetclassFlag(
    text="HighSpeed",
    length=2.54,
    shape="round",  # "round", "rectangle", "dot", "diamond"
    position=Position(X=100, Y=50, angle=0),
    effects=Effects(),
    uuid=str(uuid.uuid4()),
    fieldsAutoplaced=False,
    properties=[]
)
sch.netclassFlags.append(flag)
```

#### NetclassFlag Attributes

| Attribute | Type | Description |
|-----------|------|-------------|
| `text` | str | Netclass name |
| `length` | float | Flag length |
| `shape` | str | "round", "rectangle", "dot", "diamond" |
| `position` | Position | X, Y, angle |
| `effects` | Effects | Text formatting |
| `uuid` | str | Unique identifier |
| `fieldsAutoplaced` | bool | Auto-placed flag |
| `properties` | list[Property] | Additional properties |

---

## Hierarchical Sheet Classes

### HierarchicalSheet

Sub-sheet definition.

```python
from kiutils.items.schitems import HierarchicalSheet
from kiutils.items.common import Property

# Access sheets
for sheet in sch.sheets:
    print(f"Sheet: {sheet.sheetName.value}")
    print(f"File: {sheet.fileName.value}")
    print(f"Position: ({sheet.position.X}, {sheet.position.Y})")
    print(f"Size: {sheet.width} x {sheet.height}")

    for pin in sheet.pins:
        print(f"  Pin: {pin.name} ({pin.connectionType})")
```

#### HierarchicalSheet Attributes

| Attribute | Type | Description |
|-----------|------|-------------|
| `position` | Position | Sheet position |
| `width` | float | Sheet width |
| `height` | float | Sheet height |
| `fieldsAutoplaced` | bool | Auto-placed fields |
| `stroke` | Stroke | Border style |
| `fill` | ColorRGBA | Fill color |
| `uuid` | str | Unique identifier |
| `sheetName` | Property | Sheet name property |
| `fileName` | Property | File name property |
| `properties` | list[Property] | Additional properties |
| `pins` | list[HierarchicalPin] | Sheet pins |
| `instances` | list[HierarchicalSheetProjectInstance] | Project instances (v7+) |

### HierarchicalPin

Pin on hierarchical sheet.

```python
from kiutils.items.schitems import HierarchicalPin

# Access pins
for pin in sheet.pins:
    print(f"Pin: {pin.name}")
    print(f"Type: {pin.connectionType}")
    print(f"Position: ({pin.position.X}, {pin.position.Y})")

# Create pin
pin = HierarchicalPin(
    name="DATA",
    connectionType="bidirectional",  # "input", "output", "bidirectional", etc.
    position=Position(X=100, Y=50, angle=0),
    effects=Effects(),
    uuid=str(uuid.uuid4())
)
sheet.pins.append(pin)
```

#### HierarchicalPin Attributes

| Attribute | Type | Description |
|-----------|------|-------------|
| `name` | str | Pin name (must match hierarchical label) |
| `connectionType` | str | "input", "output", "bidirectional", etc. |
| `position` | Position | X, Y, angle |
| `effects` | Effects | Text formatting |
| `uuid` | str | Unique identifier |

### HierarchicalSheetProjectInstance

Sheet instance within project (v7+).

```python
# Attributes
instance.name   # Project name
instance.paths  # list[HierarchicalSheetProjectPath]
```

### HierarchicalSheetProjectPath

Path to sheet instance (v7+).

```python
# Attributes
path.sheetInstancePath  # Path string
path.page               # Page number string
```

---

## Instance Classes

### HierarchicalSheetInstance

Sheet instance data (root schematic only).

```python
from kiutils.items.schitems import HierarchicalSheetInstance

# Access
for inst in sch.sheetInstances:
    print(f"Path: {inst.instancePath}")
    print(f"Page: {inst.page}")

# Create (for new schematic)
inst = HierarchicalSheetInstance(
    instancePath="/",
    page="1"
)
sch.sheetInstances.append(inst)
```

#### HierarchicalSheetInstance Attributes

| Attribute | Type | Description |
|-----------|------|-------------|
| `instancePath` | str | Path to sheet (e.g., "/") |
| `page` | str | Page number |

### SymbolInstance

Symbol instance data (legacy, pre-v7).

```python
from kiutils.items.schitems import SymbolInstance

# Access
for inst in sch.symbolInstances:
    print(f"Path: {inst.path}")
    print(f"Reference: {inst.reference}")
    print(f"Value: {inst.value}")

# Attributes
inst.path       # Instance path
inst.reference  # Reference designator
inst.unit       # Unit number
inst.value      # Component value
inst.footprint  # Footprint ID
```

---

## Common Patterns

### Generate BOM

```python
from kiutils.schematic import Schematic
from collections import defaultdict

def generate_bom(schematic_path):
    """Generate BOM from schematic"""
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

def generate_grouped_bom(schematic_path):
    """Generate grouped BOM"""
    sch = Schematic.from_file(schematic_path)
    groups = defaultdict(list)

    for sym in sch.schematicSymbols:
        ref = sym.property("Reference")
        value = sym.property("Value")
        footprint = sym.property("Footprint")

        if ref and not ref.startswith("#"):
            key = (value or "", footprint or "")
            groups[key].append(ref)

    result = []
    for (value, footprint), refs in sorted(groups.items()):
        result.append({
            "Value": value,
            "Footprint": footprint,
            "Quantity": len(refs),
            "References": ", ".join(sorted(refs))
        })
    return result
```

### Create Wire Network with Junction

```python
from kiutils.schematic import Schematic
from kiutils.items.schitems import Connection, Junction
from kiutils.items.common import Position, Stroke
import uuid

def add_t_junction(sch, x, y, length_h, length_v):
    """Add T-shaped wire network with junction at (x, y)"""

    # Horizontal wire
    wire_h = Connection(
        type='wire',
        points=[
            Position(X=x - length_h/2, Y=y),
            Position(X=x + length_h/2, Y=y)
        ],
        stroke=Stroke(width=0, type='default'),
        uuid=str(uuid.uuid4())
    )
    sch.graphicalItems.append(wire_h)

    # Vertical wire
    wire_v = Connection(
        type='wire',
        points=[
            Position(X=x, Y=y),
            Position(X=x, Y=y + length_v)
        ],
        stroke=Stroke(width=0, type='default'),
        uuid=str(uuid.uuid4())
    )
    sch.graphicalItems.append(wire_v)

    # Junction at intersection
    junc = Junction(
        position=Position(X=x, Y=y),
        diameter=0,
        uuid=str(uuid.uuid4())
    )
    sch.junctions.append(junc)
```

### Find Symbols by Reference Pattern

```python
import re

def find_symbols_by_pattern(sch, pattern):
    """Find symbols matching reference pattern (e.g., 'R*', 'C1?')"""
    regex = re.compile(pattern.replace('*', '.*').replace('?', '.'))

    results = []
    for sym in sch.schematicSymbols:
        ref = sym.property("Reference")
        if ref and regex.match(ref):
            results.append(sym)
    return results

# Examples
resistors = find_symbols_by_pattern(sch, "R*")
caps_10x = find_symbols_by_pattern(sch, "C1?")
```

### Get All Wire Endpoints

```python
def get_wire_endpoints(sch):
    """Get all unique wire endpoints"""
    from kiutils.items.schitems import Connection

    points = set()
    for item in sch.graphicalItems:
        if isinstance(item, Connection) and item.type == 'wire':
            for pt in item.points:
                points.add((pt.X, pt.Y))

    # Also include junctions
    for junc in sch.junctions:
        points.add((junc.position.X, junc.position.Y))

    return points
```

### Update All Symbol Values

```python
def update_symbol_values(sch, old_value, new_value):
    """Replace value in all matching symbols"""
    count = 0
    for sym in sch.schematicSymbols:
        for prop in sym.properties:
            if prop.key == "Value" and prop.value == old_value:
                prop.value = new_value
                count += 1
    return count

# Usage
updated = update_symbol_values(sch, "100n", "100nF")
print(f"Updated {updated} symbols")
```

### Export Net List

```python
def export_simple_netlist(sch):
    """Export simplified net list"""
    nets = {}

    # Collect labels
    for label in sch.labels:
        nets.setdefault(label.text, []).append(
            f"Label@({label.position.X},{label.position.Y})"
        )

    for glabel in sch.globalLabels:
        nets.setdefault(glabel.text, []).append(
            f"GlobalLabel@({glabel.position.X},{glabel.position.Y})"
        )

    return nets
```

### Create New Schematic from Scratch

```python
from kiutils.schematic import Schematic
from kiutils.items.schitems import (
    Connection, Junction, LocalLabel, GlobalLabel,
    HierarchicalSheetInstance
)
from kiutils.items.common import Position, Stroke, Effects
import uuid

def create_empty_schematic():
    """Create a new empty schematic"""
    sch = Schematic.create_new()
    sch.uuid = str(uuid.uuid4())
    return sch

# Usage
sch = create_empty_schematic()

# Add elements...
wire = Connection(
    type='wire',
    points=[Position(X=50, Y=50), Position(X=100, Y=50)],
    stroke=Stroke(width=0, type='default'),
    uuid=str(uuid.uuid4())
)
sch.graphicalItems.append(wire)

# Save
sch.to_file("new_schematic.kicad_sch")
```

---

## Import Summary

```python
# Main schematic class
from kiutils.schematic import Schematic

# All schematic item classes
from kiutils.items.schitems import (
    # Connections
    Junction,
    NoConnect,
    BusEntry,
    BusAlias,
    Connection,
    PolyLine,

    # Text elements
    Text,
    TextBox,

    # Labels
    LocalLabel,
    GlobalLabel,
    HierarchicalLabel,

    # Shapes (v7+)
    Rectangle,
    Arc,
    Circle,
    NetclassFlag,

    # Symbols
    SchematicSymbol,
    SymbolProjectInstance,
    SymbolProjectPath,

    # Hierarchical sheets
    HierarchicalSheet,
    HierarchicalPin,
    HierarchicalSheetInstance,
    HierarchicalSheetProjectInstance,
    HierarchicalSheetProjectPath,

    # Instances
    SymbolInstance,
)

# Common items
from kiutils.items.common import (
    Position,
    Stroke,
    Fill,
    Effects,
    Font,
    Justify,
    Property,
    ColorRGBA,
)
```
