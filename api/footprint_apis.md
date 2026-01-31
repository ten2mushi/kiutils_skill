# Footprint API Reference

Comprehensive API for `.kicad_mod` footprint files.

## Quick Start

```python
from kiutils.footprint import Footprint, Pad, Model
from kiutils.items.fpitems import FpText, FpLine, FpRect, FpCircle, FpArc, FpPoly
from kiutils.items.common import Position, Effects

# Load footprint
fp = Footprint.from_file("component.kicad_mod")

# Create new footprint
fp = Footprint.create_new("MyLib:R_0805", "0805", type="smd")

# Save footprint
fp.to_file("output.kicad_mod")
```

---

## Footprint Class

Main container for footprint definitions.

### Constructor / Factory Methods

```python
# Load from file
fp = Footprint.from_file(filepath: str, encoding: str = None) -> Footprint

# Create new footprint
fp = Footprint.create_new(
    library_id: str,    # e.g., "Connector:Conn01x02"
    value: str,         # Value text
    type: str = 'other',  # 'smd', 'through_hole', or 'other'
    reference: str = 'REF**'
) -> Footprint

# Parse from S-Expression
fp = Footprint.from_sexpr(exp: list) -> Footprint
```

### Key Attributes

| Attribute | Type | Description |
|-----------|------|-------------|
| `libId` | `str` (property) | Library identifier `<libraryNickname>:<entryName>` |
| `libraryNickname` | `str \| None` | Library name prefix |
| `entryName` | `str` | Footprint name |
| `version` | `str \| None` | Version stamp (YYYYMMDD) |
| `generator` | `str \| None` | Creation tool identifier |
| `layer` | `str` | Canonical layer (default: `"F.Cu"`) |
| `description` | `str \| None` | Footprint description |
| `tags` | `str \| None` | Search keywords |
| `properties` | `Dict[str, str]` | Key-value property pairs |
| `locked` | `bool` | Edit lock flag |
| `placed` | `bool` | Placement status flag |

### Position & Path (Board Context)

| Attribute | Type | Description |
|-----------|------|-------------|
| `position` | `Position \| None` | Board placement coordinates |
| `path` | `str \| None` | Hierarchical schematic path |
| `tstamp` | `str \| None` | Unique identifier |

### Manufacturing Settings

| Attribute | Type | Description |
|-----------|------|-------------|
| `solderMaskMargin` | `float \| None` | Solder mask distance from pads |
| `solderPasteMargin` | `float \| None` | Solder paste distance from pads |
| `solderPasteRatio` | `float \| None` | Paste size percentage |
| `clearance` | `float \| None` | Copper clearance from pads |
| `zoneConnect` | `int \| None` | Zone connection mode (0-3) |
| `thermalWidth` | `float \| None` | Thermal relief spoke width |
| `thermalGap` | `float \| None` | Thermal relief gap distance |

### Collections

| Attribute | Type | Description |
|-----------|------|-------------|
| `graphicItems` | `List` | FpText, FpLine, FpRect, FpCircle, FpArc, FpPoly, FpCurve, Image |
| `pads` | `List[Pad]` | Pad definitions |
| `zones` | `List[Zone]` | Keep-out zones |
| `models` | `List[Model]` | 3D model associations |
| `groups` | `List[Group]` | Grouped objects |
| `privateLayers` | `List[str]` | Private layer assignments (v7+) |
| `netTiePadGroups` | `List[str]` | Net tie groups (v7+) |
| `attributes` | `Attributes` | Footprint attributes |

### Methods

```python
# Save to file
fp.to_file(filepath: str = None, encoding: str = None)

# Generate S-Expression
fp.to_sexpr(indent: int = 0, newline: bool = True, layerInFirstLine: bool = False) -> str
```

---

## Attributes Class

Footprint configuration flags.

```python
from kiutils.footprint import Attributes

attrs = Attributes(
    type='smd',                    # 'smd' or 'through_hole'
    boardOnly=False,               # Board-only footprint
    excludeFromPosFiles=False,     # Exclude from position files
    excludeFromBom=False,          # Exclude from BOM
    allowMissingCourtyard=False    # Allow missing courtyard (v7+)
)
```

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `type` | `str \| None` | `None` | `"smd"` or `"through_hole"` |
| `boardOnly` | `bool` | `False` | No schematic reference |
| `excludeFromPosFiles` | `bool` | `False` | Exclude from pick-and-place |
| `excludeFromBom` | `bool` | `False` | Exclude from BOM |
| `allowMissingCourtyard` | `bool` | `False` | Suppress courtyard DRC (v7+) |

---

## Pad Class

Individual pad definition.

```python
from kiutils.footprint import Pad, DrillDefinition
from kiutils.items.common import Position, Net

# SMD pad
pad = Pad(
    number='1',
    type='smd',
    shape='rect',
    position=Position(X=0, Y=0),
    size=Position(X=1.2, Y=0.6),
    layers=['F.Cu', 'F.Paste', 'F.Mask']
)

# Through-hole pad
pad = Pad(
    number='1',
    type='thru_hole',
    shape='circle',
    position=Position(X=0, Y=0),
    size=Position(X=1.8, Y=1.8),
    drill=DrillDefinition(diameter=1.0),
    layers=['*.Cu', '*.Mask']
)
```

### Core Attributes

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `number` | `str` | `"x"` | Pad identifier |
| `type` | `str` | `"smd"` | `thru_hole`, `smd`, `connect`, `np_thru_hole` |
| `shape` | `str` | `"rect"` | `circle`, `rect`, `oval`, `trapezoid`, `roundrect`, `custom` |
| `position` | `Position` | `Position()` | X, Y, angle |
| `size` | `Position` | `Position()` | Width, height |
| `layers` | `List[str]` | `[]` | Layer assignments |
| `locked` | `bool` | `False` | Edit lock |

### Drill Attributes

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `drill` | `DrillDefinition \| None` | `None` | Drill specifications |

### Shape Modifiers

| Attribute | Type | Description |
|-----------|------|-------------|
| `roundrectRatio` | `float \| None` | Corner radius ratio (0-1) |
| `chamferRatio` | `float \| None` | Chamfer size ratio (0-1) |
| `chamfer` | `List[str]` | `top_left`, `top_right`, `bottom_left`, `bottom_right` |

### Properties

| Attribute | Type | Description |
|-----------|------|-------------|
| `property` | `str \| None` | `pad_prop_bga`, `pad_prop_fiducial_glob`, `pad_prop_fiducial_loc`, `pad_prop_testpoint`, `pad_prop_heatsink`, `pad_prop_castellated` |
| `pinFunction` | `str \| None` | Schematic pin name |
| `pinType` | `str \| None` | Pin electrical type |

### Manufacturing Overrides

| Attribute | Type | Description |
|-----------|------|-------------|
| `solderMaskMargin` | `float \| None` | Mask clearance override |
| `solderPasteMargin` | `float \| None` | Paste margin override |
| `solderPasteMarginRatio` | `float \| None` | Paste percentage override |
| `clearance` | `float \| None` | Copper clearance override |
| `zoneConnect` | `int \| None` | Zone connection override (0-3) |
| `thermalWidth` | `float \| None` | Thermal spoke width override |
| `thermalGap` | `float \| None` | Thermal gap override |
| `dieLength` | `float \| None` | Die length measurement |

### Advanced

| Attribute | Type | Description |
|-----------|------|-------------|
| `net` | `Net \| None` | Net connection (board context) |
| `tstamp` | `str \| None` | Unique identifier |
| `removeUnusedLayers` | `bool` | Remove copper from unconnected layers |
| `keepEndLayers` | `bool` | Retain top/bottom when removing layers |
| `customPadOptions` | `PadOptions \| None` | Custom pad settings |
| `customPadPrimitives` | `List` | Custom pad drawing objects |

---

## DrillDefinition Class

Drill specifications for pads.

```python
from kiutils.footprint import DrillDefinition
from kiutils.items.common import Position

# Round drill
drill = DrillDefinition(diameter=1.0)

# Oval drill (slot)
drill = DrillDefinition(
    oval=True,
    diameter=1.0,
    width=2.0
)

# Offset drill
drill = DrillDefinition(
    diameter=1.0,
    offset=Position(X=0.5, Y=0)
)
```

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `oval` | `bool` | `False` | Oval/slot drill |
| `diameter` | `float` | `0.0` | Drill diameter |
| `width` | `float \| None` | `None` | Slot width (oval only) |
| `offset` | `Position \| None` | `None` | Offset from pad center |

---

## PadOptions Class

Custom pad configuration.

```python
from kiutils.footprint import PadOptions

opts = PadOptions(
    clearance='outline',   # 'outline' or 'convexhull'
    anchor='rect'          # 'rect' or 'circle'
)
```

---

## Model Class

3D model association.

```python
from kiutils.footprint import Model
from kiutils.items.common import Coordinate

model = Model(
    path="${KICAD7_3DMODEL_DIR}/Resistor_SMD.3dshapes/R_0805.wrl",
    pos=Coordinate(0.0, 0.0, 0.0),
    scale=Coordinate(1.0, 1.0, 1.0),
    rotate=Coordinate(0.0, 0.0, 0.0),
    hide=False,
    opacity=1.0  # 0.0-1.0
)
```

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `path` | `str` | `""` | 3D model file path |
| `pos` | `Coordinate` | `(0,0,0)` | 3D position offset |
| `scale` | `Coordinate` | `(1,1,1)` | Scale factors |
| `rotate` | `Coordinate` | `(0,0,0)` | Rotation angles |
| `hide` | `bool` | `False` | Visibility flag |
| `opacity` | `float \| None` | `None` | Opacity (0.0-1.0) |

---

## Footprint Graphic Items

### FpText

Text element in footprint.

```python
from kiutils.items.fpitems import FpText
from kiutils.items.common import Position, Effects

text = FpText(
    type='reference',        # 'reference', 'value', or 'user'
    text='R1',
    position=Position(X=0, Y=-1.5),
    layer='F.SilkS',
    effects=Effects(),
    hide=False,
    knockout=False
)
```

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `type` | `str` | `"reference"` | `reference`, `value`, `user` |
| `text` | `str` | `"%REF"` | Text content |
| `position` | `Position` | `Position()` | Location and angle |
| `layer` | `str` | `"F.Cu"` | Layer assignment |
| `effects` | `Effects` | `Effects()` | Text styling |
| `hide` | `bool` | `False` | Hidden flag |
| `knockout` | `bool` | `False` | Inverted text |
| `tstamp` | `str \| None` | `None` | Unique ID |
| `renderCache` | `RenderCache \| None` | `None` | TrueType cache (v7+) |

### FpLine

Line element.

```python
from kiutils.items.fpitems import FpLine
from kiutils.items.common import Position, Stroke

line = FpLine(
    start=Position(X=-1, Y=0),
    end=Position(X=1, Y=0),
    layer='F.SilkS',
    width=0.12,          # KiCad < 7
    stroke=Stroke(...)   # KiCad >= 7
)
```

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `start` | `Position` | `Position()` | Start point |
| `end` | `Position` | `Position()` | End point |
| `layer` | `str` | `"F.Cu"` | Layer |
| `width` | `float \| None` | `0.12` | Line width (< v7) |
| `stroke` | `Stroke \| None` | `None` | Stroke style (v7+) |
| `locked` | `bool` | `False` | Edit lock |
| `tstamp` | `str \| None` | `None` | Unique ID |

### FpRect

Rectangle element.

```python
from kiutils.items.fpitems import FpRect
from kiutils.items.common import Position

rect = FpRect(
    start=Position(X=-1, Y=-0.5),
    end=Position(X=1, Y=0.5),
    layer='F.SilkS',
    width=0.12,
    fill='none'  # 'solid' or 'none'
)
```

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `start` | `Position` | `Position()` | Top-left corner |
| `end` | `Position` | `Position()` | Bottom-right corner |
| `layer` | `str` | `"F.Cu"` | Layer |
| `width` | `float \| None` | `0.12` | Line width (< v7) |
| `stroke` | `Stroke \| None` | `None` | Stroke style (v7+) |
| `fill` | `str \| None` | `None` | `solid` or `none` |
| `locked` | `bool` | `False` | Edit lock |
| `tstamp` | `str \| None` | `None` | Unique ID |

### FpCircle

Circle element.

```python
from kiutils.items.fpitems import FpCircle
from kiutils.items.common import Position

circle = FpCircle(
    center=Position(X=0, Y=0),
    end=Position(X=1, Y=0),    # Point on circumference
    layer='F.SilkS',
    width=0.12,
    fill='none'
)
```

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `center` | `Position` | `Position()` | Center point |
| `end` | `Position` | `Position()` | Point on circumference |
| `layer` | `str` | `"F.Cu"` | Layer |
| `width` | `float \| None` | `0.12` | Line width (< v7) |
| `stroke` | `Stroke \| None` | `None` | Stroke style (v7+) |
| `fill` | `str \| None` | `None` | `solid` or `none` |
| `locked` | `bool` | `False` | Edit lock |
| `tstamp` | `str \| None` | `None` | Unique ID |

### FpArc

Arc element (defined by start, mid, end points).

```python
from kiutils.items.fpitems import FpArc
from kiutils.items.common import Position

arc = FpArc(
    start=Position(X=-1, Y=0),
    mid=Position(X=0, Y=-1),
    end=Position(X=1, Y=0),
    layer='F.SilkS',
    width=0.12
)
```

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `start` | `Position` | `Position()` | Arc start |
| `mid` | `Position` | `Position()` | Arc midpoint |
| `end` | `Position` | `Position()` | Arc end |
| `layer` | `str` | `"F.Cu"` | Layer |
| `width` | `float \| None` | `0.12` | Line width (< v7) |
| `stroke` | `Stroke \| None` | `None` | Stroke style (v7+) |
| `locked` | `bool` | `False` | Edit lock |
| `tstamp` | `str \| None` | `None` | Unique ID |

### FpPoly

Polygon element.

```python
from kiutils.items.fpitems import FpPoly
from kiutils.items.common import Position

poly = FpPoly(
    coordinates=[
        Position(X=0, Y=0),
        Position(X=1, Y=0),
        Position(X=0.5, Y=1)
    ],
    layer='F.SilkS',
    width=0.12,
    fill='solid'
)
```

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `coordinates` | `List[Position]` | `[]` | Polygon vertices |
| `layer` | `str` | `"F.Cu"` | Layer |
| `width` | `float \| None` | `0.12` | Line width (< v7) |
| `stroke` | `Stroke \| None` | `None` | Stroke style (v7+) |
| `fill` | `str \| None` | `None` | `solid` or `none` |
| `locked` | `bool` | `False` | Edit lock |
| `tstamp` | `str \| None` | `None` | Unique ID |

### FpCurve

Cubic Bezier curve.

```python
from kiutils.items.fpitems import FpCurve
from kiutils.items.common import Position

curve = FpCurve(
    coordinates=[
        Position(X=0, Y=0),     # Start
        Position(X=0.5, Y=1),   # Control 1
        Position(X=1.5, Y=1),   # Control 2
        Position(X=2, Y=0)      # End
    ],
    layer='F.SilkS',
    width=0.12
)
```

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `coordinates` | `List[Position]` | `[]` | 4 control points |
| `layer` | `str` | `"F.Cu"` | Layer |
| `width` | `float \| None` | `0.12` | Line width (< v7) |
| `stroke` | `Stroke \| None` | `None` | Stroke style (v7+) |
| `locked` | `bool` | `False` | Edit lock |
| `tstamp` | `str \| None` | `None` | Unique ID |

### FpTextBox (v7+)

Text box with border.

```python
from kiutils.items.fpitems import FpTextBox
from kiutils.items.common import Position, Effects, Stroke

textbox = FpTextBox(
    text="Description",
    start=Position(X=-2, Y=-1),
    end=Position(X=2, Y=1),
    layer='F.SilkS',
    effects=Effects(),
    stroke=Stroke()
)
```

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `text` | `str` | `""` | Text content |
| `locked` | `bool` | `False` | Edit lock |
| `start` | `Position \| None` | `None` | Top-left (cardinal angles) |
| `end` | `Position \| None` | `None` | Bottom-right (cardinal angles) |
| `pts` | `List[Position]` | `[]` | 4 corners (non-cardinal angles) |
| `angle` | `float \| None` | `None` | Rotation angle |
| `layer` | `str` | `"F.Cu"` | Layer |
| `effects` | `Effects \| None` | `None` | Text styling |
| `stroke` | `Stroke \| None` | `None` | Border style |
| `tstamp` | `str \| None` | `None` | Unique ID |
| `renderCache` | `RenderCache \| None` | `None` | TrueType cache |

---

## Common Patterns

### Create SMD Resistor Footprint

```python
from kiutils.footprint import Footprint, Pad, Model, Attributes
from kiutils.items.fpitems import FpText, FpLine
from kiutils.items.common import Position, Effects, Font

fp = Footprint.create_new("MyLib:R_0805", "0805", type="smd")

# Add pads
fp.pads = [
    Pad(number='1', type='smd', shape='rect',
        position=Position(X=-0.95, Y=0),
        size=Position(X=0.7, Y=0.8),
        layers=['F.Cu', 'F.Paste', 'F.Mask']),
    Pad(number='2', type='smd', shape='rect',
        position=Position(X=0.95, Y=0),
        size=Position(X=0.7, Y=0.8),
        layers=['F.Cu', 'F.Paste', 'F.Mask'])
]

# Add silkscreen
fp.graphicItems.append(
    FpLine(start=Position(X=-0.25, Y=-0.5),
           end=Position(X=0.25, Y=-0.5),
           layer='F.SilkS', width=0.12)
)

# Add courtyard
fp.graphicItems.append(
    FpRect(start=Position(X=-1.5, Y=-0.65),
           end=Position(X=1.5, Y=0.65),
           layer='F.CrtYd', width=0.05)
)

fp.to_file("R_0805.kicad_mod")
```

### Create Through-Hole Connector

```python
from kiutils.footprint import Footprint, Pad, DrillDefinition

fp = Footprint.create_new("MyLib:PinHeader_1x04", "1x04", type="through_hole")

# Add through-hole pads
for i in range(4):
    shape = 'rect' if i == 0 else 'circle'  # Pin 1 is square
    fp.pads.append(
        Pad(number=str(i+1), type='thru_hole', shape=shape,
            position=Position(X=0, Y=i*2.54),
            size=Position(X=1.7, Y=1.7),
            drill=DrillDefinition(diameter=1.0),
            layers=['*.Cu', '*.Mask'])
    )

fp.to_file("PinHeader_1x04.kicad_mod")
```

### Find Pads by Number

```python
fp = Footprint.from_file("component.kicad_mod")

# Find specific pad
pad1 = next((p for p in fp.pads if p.number == '1'), None)

# Find all power pads
power_pads = [p for p in fp.pads if p.pinType == 'power_in']
```

### Modify Pad Properties

```python
fp = Footprint.from_file("component.kicad_mod")

for pad in fp.pads:
    # Increase all solder mask margins
    pad.solderMaskMargin = 0.1

    # Set thermal relief for all pads
    pad.zoneConnect = 1
    pad.thermalWidth = 0.3
    pad.thermalGap = 0.3

fp.to_file("modified.kicad_mod")
```

### Add 3D Model

```python
from kiutils.footprint import Model
from kiutils.items.common import Coordinate

fp = Footprint.from_file("component.kicad_mod")

fp.models.append(
    Model(
        path="${KICAD7_3DMODEL_DIR}/Package_SO.3dshapes/SOIC-8.wrl",
        pos=Coordinate(0.0, 0.0, 0.0),
        scale=Coordinate(1.0, 1.0, 1.0),
        rotate=Coordinate(0.0, 0.0, 0.0)
    )
)

fp.to_file("with_3d.kicad_mod")
```

### Batch Process Footprint Library

```python
import os
from kiutils.footprint import Footprint

lib_path = "MyLibrary.pretty"
for filename in os.listdir(lib_path):
    if filename.endswith(".kicad_mod"):
        fp = Footprint.from_file(os.path.join(lib_path, filename))

        # Update all footprints
        fp.attributes.excludeFromBom = False
        fp.description = f"Updated: {fp.description or ''}"

        fp.to_file()
```
