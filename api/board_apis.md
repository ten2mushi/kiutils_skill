# Kiutils Board API Reference

Complete API reference for all classes in `kiutils.board`, `kiutils.items.brditems`, `kiutils.items.gritems`, and `kiutils.items.zones` for manipulating `.kicad_pcb` files.

## Table of Contents

1. [Board Class](#board-class)
2. [Board Settings Classes](#board-settings-classes)
3. [Layer and Stackup Classes](#layer-and-stackup-classes)
4. [Trace Item Classes](#trace-item-classes)
5. [Graphic Item Classes](#graphic-item-classes)
6. [Zone Classes](#zone-classes)
7. [Common Patterns](#common-patterns)

---

## Board Class

Main class: `kiutils.board.Board`

### Import and Load

```python
from kiutils.board import Board

# Load from file
board = Board.from_file("project.kicad_pcb")

# Create empty board with default layers
board = Board.create_new()

# Parse from S-expression list
board = Board.from_sexpr(sexpr_list)
```

### Save

```python
board.to_file()                      # Save to original path
board.to_file("output.kicad_pcb")    # Save to new path
sexpr = board.to_sexpr()             # Get S-expression string
```

### Board Attributes

| Attribute | Type | Description |
|-----------|------|-------------|
| `version` | str | File version (YYYYMMDD format) |
| `generator` | str | Creating tool identifier |
| `general` | GeneralSettings | Board thickness settings |
| `paper` | PageSettings | Paper size settings |
| `titleBlock` | TitleBlock | Title block info (author, date, rev) |
| `layers` | list[LayerToken] | Layer definitions |
| `setup` | SetupData | Board setup (stackup, clearances, plot) |
| `properties` | dict[str, str] | Key-value properties |
| `nets` | list[Net] | Net definitions |
| `footprints` | list[Footprint] | Placed footprint instances |
| `graphicItems` | list | Graphical items (GrText, GrLine, etc.) |
| `traceItems` | list | Traces (Segment, Via, Arc) |
| `zones` | list[Zone] | Copper zones |
| `dimensions` | list[Dimension] | Dimension annotations |
| `targets` | list[Target] | Target markers |
| `groups` | list[Group] | Grouped objects |
| `filePath` | str | File path (set by from_file) |

---

## Board Settings Classes

### GeneralSettings

Board general settings.

```python
from kiutils.items.brditems import GeneralSettings

# Access
print(f"Board thickness: {board.general.thickness} mm")

# Modify
board.general.thickness = 1.6  # Standard 1.6mm board
```

#### GeneralSettings Attributes

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `thickness` | float | 1.6 | Board thickness in mm |

### SetupData

Board setup containing stackup, clearances, and plot settings.

```python
from kiutils.items.brditems import SetupData

# Access setup
setup = board.setup

# Modify clearances
setup.packToMaskClearance = 0.1
setup.solderMaskMinWidth = 0.1
setup.padToPasteClearance = 0.0

# Set aux origin
setup.auxAxisOrigin = Position(X=100, Y=100)
```

#### SetupData Attributes

| Attribute | Type | Description |
|-----------|------|-------------|
| `stackup` | Stackup | Board stackup definition |
| `packToMaskClearance` | float | Pad to solder mask clearance |
| `solderMaskMinWidth` | float | Minimum solder mask width |
| `padToPasteClearance` | float | Pad to paste clearance |
| `padToPasteClearanceRatio` | float | Paste ratio (0-100%) |
| `auxAxisOrigin` | Position | Auxiliary origin |
| `gridOrigin` | Position | Grid origin |
| `plotSettings` | PlotSettings | Plot/export settings |

### PlotSettings

Plot and export settings.

```python
from kiutils.items.brditems import PlotSettings

# Access
plot = board.setup.plotSettings

# Configure gerber settings
plot.useGerberExtensions = True
plot.useGerberAttributes = True
plot.createGerberJobFile = True
plot.outputDirectory = "gerbers/"

# Output format: 0=gerber, 1=PS, 2=SVG, 3=DXF, 4=HPGL, 5=PDF
plot.outputFormat = 0
```

#### PlotSettings Key Attributes

| Attribute | Type | Description |
|-----------|------|-------------|
| `layerSelection` | str | Hex bitset of layers to plot |
| `useGerberExtensions` | bool | Use Protel file extensions |
| `useGerberAttributes` | bool | Use X2 extensions |
| `useGerberAdvancedAttributes` | bool | Include netlist info |
| `createGerberJobFile` | bool | Create job file |
| `outputFormat` | int | Format (0=gerber, 1=PS, 2=SVG, 3=DXF, 4=HPGL, 5=PDF) |
| `outputDirectory` | str | Output path |
| `plotReference` | bool | Plot reference fields |
| `plotValue` | bool | Plot value fields |
| `mirror` | bool | Mirror output |
| `drillShape` | int | Drill mark type |

---

## Layer and Stackup Classes

### LayerToken

Layer definition.

```python
from kiutils.items.brditems import LayerToken

# Access layers
for layer in board.layers:
    print(f"{layer.ordinal}: {layer.name} ({layer.type})")

# Add inner copper layer
board.layers.append(LayerToken(
    ordinal=1,
    name="In1.Cu",
    type="signal"
))
```

#### LayerToken Attributes

| Attribute | Type | Description |
|-----------|------|-------------|
| `ordinal` | int | Layer stack order |
| `name` | str | Layer name (e.g., "F.Cu") |
| `type` | str | "signal", "power", "mixed", "jumper", "user" |
| `userName` | str | Custom user name (optional) |

#### Standard Layer Ordinals

| Ordinal | Name | Type |
|---------|------|------|
| 0 | F.Cu | signal |
| 31 | B.Cu | signal |
| 32-33 | B/F.Adhes | user |
| 34-35 | B/F.Paste | user |
| 36-37 | B/F.SilkS | user |
| 38-39 | B/F.Mask | user |
| 40-43 | Dwgs/Cmts/Eco.User | user |
| 44 | Edge.Cuts | user |
| 45 | Margin | user |
| 46-47 | B/F.CrtYd | user |
| 48-49 | B/F.Fab | user |

### Stackup

Board stackup definition.

```python
from kiutils.items.brditems import Stackup, StackupLayer

# Access stackup
if board.setup.stackup:
    for layer in board.setup.stackup.layers:
        print(f"{layer.name}: {layer.thickness}mm ({layer.type})")

# Create 4-layer stackup
stackup = Stackup()
stackup.layers = [
    StackupLayer(name="F.SilkS", type="Top Silk Screen"),
    StackupLayer(name="F.Paste", type="Top Solder Paste"),
    StackupLayer(name="F.Mask", type="Top Solder Mask", thickness=0.01),
    StackupLayer(name="F.Cu", type="copper", thickness=0.035),
    StackupLayer(name="dielectric 1", type="core", thickness=0.2, material="FR4", epsilonR=4.5),
    StackupLayer(name="In1.Cu", type="copper", thickness=0.035),
    StackupLayer(name="dielectric 2", type="prepreg", thickness=1.0, material="FR4"),
    StackupLayer(name="In2.Cu", type="copper", thickness=0.035),
    StackupLayer(name="dielectric 3", type="core", thickness=0.2, material="FR4", epsilonR=4.5),
    StackupLayer(name="B.Cu", type="copper", thickness=0.035),
    StackupLayer(name="B.Mask", type="Bottom Solder Mask", thickness=0.01),
    StackupLayer(name="B.Paste", type="Bottom Solder Paste"),
    StackupLayer(name="B.SilkS", type="Bottom Silk Screen"),
]
stackup.copperFinish = "ENIG"
board.setup.stackup = stackup
```

#### Stackup Attributes

| Attribute | Type | Description |
|-----------|------|-------------|
| `layers` | list[StackupLayer] | Layer stack |
| `copperFinish` | str | Copper finish (e.g., "ENIG", "HASL") |
| `dielectricContraints` | str | "yes" or "no" |
| `edgeConnector` | str | "yes" or "bevelled" |
| `castellatedPads` | bool | Has castellated pads |
| `edgePlating` | bool | Edge plating required |

### StackupLayer

Individual stackup layer.

```python
from kiutils.items.brditems import StackupLayer

layer = StackupLayer(
    name="F.Cu",
    type="copper",
    thickness=0.035,
    material="Copper",
    epsilonR=None,      # Only for dielectric
    lossTangent=None    # Only for dielectric
)
```

#### StackupLayer Attributes

| Attribute | Type | Description |
|-----------|------|-------------|
| `name` | str | Layer name or "dielectric N" |
| `type` | str | "copper", "core", "prepreg", etc. |
| `color` | str | Color (solder mask/silk only) |
| `thickness` | float | Thickness in mm |
| `material` | str | Material name |
| `epsilonR` | float | Dielectric constant |
| `lossTangent` | float | Loss tangent |
| `subLayers` | list[StackupSubLayer] | Stacked dielectrics |

---

## Trace Item Classes

### Segment

Track segment (straight trace).

```python
from kiutils.items.brditems import Segment
from kiutils.items.common import Position
import uuid

# Access segments
for item in board.traceItems:
    if isinstance(item, Segment):
        print(f"Track: {item.start.X},{item.start.Y} to {item.end.X},{item.end.Y}")
        print(f"  Width: {item.width}, Layer: {item.layer}, Net: {item.net}")

# Create track
track = Segment(
    start=Position(X=100.0, Y=100.0),
    end=Position(X=150.0, Y=100.0),
    width=0.25,
    layer="F.Cu",
    net=1,  # Use net NUMBER, not Net object
    locked=False,
    tstamp=str(uuid.uuid4())
)
board.traceItems.append(track)
```

#### Segment Attributes

| Attribute | Type | Description |
|-----------|------|-------------|
| `start` | Position | Start coordinates |
| `end` | Position | End coordinates |
| `width` | float | Track width in mm |
| `layer` | str | Copper layer name |
| `locked` | bool | Prevent editing |
| `net` | int | Net ordinal number |
| `tstamp` | str | Unique identifier |

### Via

Track via.

```python
from kiutils.items.brditems import Via
from kiutils.items.common import Position
import uuid

# Access vias
for item in board.traceItems:
    if isinstance(item, Via):
        print(f"Via at {item.position.X},{item.position.Y}")
        print(f"  Size: {item.size}, Drill: {item.drill}")
        print(f"  Type: {item.type}, Layers: {item.layers}")

# Create through-hole via
via = Via(
    type=None,  # None=through, "blind", "micro"
    position=Position(X=125.0, Y=100.0),
    size=0.8,
    drill=0.4,
    layers=["F.Cu", "B.Cu"],
    net=1,
    locked=False,
    tstamp=str(uuid.uuid4())
)
board.traceItems.append(via)

# Create blind via (F.Cu to In1.Cu)
blind_via = Via(
    type="blind",
    position=Position(X=130.0, Y=100.0),
    size=0.6,
    drill=0.3,
    layers=["F.Cu", "In1.Cu"],
    net=1,
    tstamp=str(uuid.uuid4())
)
board.traceItems.append(blind_via)
```

#### Via Attributes

| Attribute | Type | Description |
|-----------|------|-------------|
| `type` | str | None (through), "blind", "micro" |
| `position` | Position | Via center |
| `size` | float | Annular ring diameter |
| `drill` | float | Drill diameter |
| `layers` | list[str] | Connected layers |
| `locked` | bool | Prevent editing |
| `net` | int | Net ordinal number |
| `removeUnusedLayers` | bool | Remove unused layers |
| `keepEndLayers` | bool | Keep end layers |
| `free` | bool | Free via (can move off net) |
| `tstamp` | str | Unique identifier |

### Arc (Track Arc)

Track arc for length matching.

```python
from kiutils.items.brditems import Arc
from kiutils.items.common import Position
import uuid

# Create track arc
arc = Arc(
    start=Position(X=100.0, Y=100.0),
    mid=Position(X=105.0, Y=95.0),
    end=Position(X=110.0, Y=100.0),
    width=0.25,
    layer="F.Cu",
    net=1,
    locked=False,
    tstamp=str(uuid.uuid4())
)
board.traceItems.append(arc)
```

#### Arc Attributes

| Attribute | Type | Description |
|-----------|------|-------------|
| `start` | Position | Arc start point |
| `mid` | Position | Arc midpoint (on arc) |
| `end` | Position | Arc end point |
| `width` | float | Track width |
| `layer` | str | Copper layer |
| `locked` | bool | Prevent editing |
| `net` | int | Net ordinal number |
| `tstamp` | str | Unique identifier |

### Target

Target marker for alignment.

```python
from kiutils.items.brditems import Target
from kiutils.items.common import Position
import uuid

target = Target(
    type="plus",  # "plus" or "x"
    position=Position(X=50.0, Y=50.0),
    size=5.0,
    width=0.15,
    layer="F.Cu",
    tstamp=str(uuid.uuid4())
)
board.targets.append(target)
```

#### Target Attributes

| Attribute | Type | Description |
|-----------|------|-------------|
| `type` | str | "plus" or "x" shape |
| `position` | Position | Marker position |
| `size` | float | Marker size |
| `width` | float | Line width |
| `layer` | str | Layer name |
| `tstamp` | str | Unique identifier |

---

## Graphic Item Classes

All from `kiutils.items.gritems`.

### GrText

Graphical text annotation.

```python
from kiutils.items.gritems import GrText
from kiutils.items.common import Position, Effects
import uuid

text = GrText(
    text="REV A",
    position=Position(X=100, Y=10, angle=0),
    layer="F.SilkS",
    effects=Effects(),
    knockout=False,
    locked=False,
    tstamp=str(uuid.uuid4())
)
board.graphicItems.append(text)
```

#### GrText Attributes

| Attribute | Type | Description |
|-----------|------|-------------|
| `text` | str | Text content |
| `position` | Position | X, Y, angle |
| `layer` | str | Layer name |
| `effects` | Effects | Text formatting |
| `knockout` | bool | Inverted text |
| `locked` | bool | Prevent editing |
| `tstamp` | str | Unique identifier |

### GrTextBox

Text box with border (v7+).

```python
from kiutils.items.gritems import GrTextBox
from kiutils.items.common import Position, Effects, Stroke
import uuid

textbox = GrTextBox(
    text="Important Note",
    start=Position(X=10, Y=10),
    end=Position(X=50, Y=30),
    layer="F.SilkS",
    effects=Effects(),
    stroke=Stroke(width=0.15),
    locked=False,
    tstamp=str(uuid.uuid4())
)
board.graphicItems.append(textbox)
```

#### GrTextBox Attributes

| Attribute | Type | Description |
|-----------|------|-------------|
| `text` | str | Text content |
| `start` | Position | Top-left corner |
| `end` | Position | Bottom-right corner |
| `pts` | list[Position] | 4 corners (non-cardinal angles) |
| `angle` | float | Rotation angle |
| `layer` | str | Layer name |
| `effects` | Effects | Text formatting |
| `stroke` | Stroke | Border style |
| `locked` | bool | Prevent editing |
| `tstamp` | str | Unique identifier |

### GrLine

Graphical line.

```python
from kiutils.items.gritems import GrLine
from kiutils.items.common import Position
import uuid

line = GrLine(
    start=Position(X=0, Y=0),
    end=Position(X=100, Y=0),
    layer="Edge.Cuts",
    width=0.15,
    locked=False,
    tstamp=str(uuid.uuid4())
)
board.graphicItems.append(line)
```

#### GrLine Attributes

| Attribute | Type | Description |
|-----------|------|-------------|
| `start` | Position | Line start |
| `end` | Position | Line end |
| `layer` | str | Layer name |
| `width` | float | Line width |
| `angle` | float | Optional rotation |
| `locked` | bool | Prevent editing |
| `tstamp` | str | Unique identifier |

### GrRect

Graphical rectangle.

```python
from kiutils.items.gritems import GrRect
from kiutils.items.common import Position
import uuid

rect = GrRect(
    start=Position(X=0, Y=0),
    end=Position(X=100, Y=80),
    layer="Edge.Cuts",
    width=0.15,
    fill=None,  # None, "solid", "none"
    locked=False,
    tstamp=str(uuid.uuid4())
)
board.graphicItems.append(rect)
```

#### GrRect Attributes

| Attribute | Type | Description |
|-----------|------|-------------|
| `start` | Position | Top-left corner |
| `end` | Position | Bottom-right corner |
| `layer` | str | Layer name |
| `width` | float | Line width |
| `fill` | str | "solid", "none", or None |
| `locked` | bool | Prevent editing |
| `tstamp` | str | Unique identifier |

### GrCircle

Graphical circle.

```python
from kiutils.items.gritems import GrCircle
from kiutils.items.common import Position
import uuid

# Circle defined by center and point on circumference
circle = GrCircle(
    center=Position(X=50, Y=40),
    end=Position(X=55, Y=40),  # 5mm radius
    layer="F.SilkS",
    width=0.15,
    fill=None,
    locked=False,
    tstamp=str(uuid.uuid4())
)
board.graphicItems.append(circle)
```

#### GrCircle Attributes

| Attribute | Type | Description |
|-----------|------|-------------|
| `center` | Position | Circle center |
| `end` | Position | Point on circumference |
| `layer` | str | Layer name |
| `width` | float | Line width |
| `fill` | str | "solid", "none", or None |
| `locked` | bool | Prevent editing |
| `tstamp` | str | Unique identifier |

### GrArc

Graphical arc.

```python
from kiutils.items.gritems import GrArc
from kiutils.items.common import Position
import uuid

arc = GrArc(
    start=Position(X=10, Y=0),
    mid=Position(X=5, Y=5),
    end=Position(X=0, Y=0),
    layer="Edge.Cuts",
    width=0.15,
    locked=False,
    tstamp=str(uuid.uuid4())
)
board.graphicItems.append(arc)
```

#### GrArc Attributes

| Attribute | Type | Description |
|-----------|------|-------------|
| `start` | Position | Arc start |
| `mid` | Position | Point on arc |
| `end` | Position | Arc end |
| `layer` | str | Layer name |
| `width` | float | Line width |
| `locked` | bool | Prevent editing |
| `tstamp` | str | Unique identifier |

### GrPoly

Graphical polygon.

```python
from kiutils.items.gritems import GrPoly
from kiutils.items.common import Position
import uuid

# Triangle
poly = GrPoly(
    layer="F.SilkS",
    coordinates=[
        Position(X=0, Y=0),
        Position(X=10, Y=0),
        Position(X=5, Y=8.66),
    ],
    width=0.15,
    fill="solid",
    locked=False,
    tstamp=str(uuid.uuid4())
)
board.graphicItems.append(poly)
```

#### GrPoly Attributes

| Attribute | Type | Description |
|-----------|------|-------------|
| `layer` | str | Layer name |
| `coordinates` | list[Position] | Polygon vertices |
| `width` | float | Line width |
| `fill` | str | "solid", "none", or None |
| `locked` | bool | Prevent editing |
| `tstamp` | str | Unique identifier |

### GrCurve

Cubic Bezier curve.

```python
from kiutils.items.gritems import GrCurve
from kiutils.items.common import Position
import uuid

curve = GrCurve(
    coordinates=[
        Position(X=0, Y=0),      # Start point
        Position(X=5, Y=10),     # Control point 1
        Position(X=15, Y=10),    # Control point 2
        Position(X=20, Y=0),     # End point
    ],
    layer="F.SilkS",
    width=0.15,
    locked=False,
    tstamp=str(uuid.uuid4())
)
board.graphicItems.append(curve)
```

#### GrCurve Attributes

| Attribute | Type | Description |
|-----------|------|-------------|
| `coordinates` | list[Position] | 4 Bezier control points |
| `layer` | str | Layer name |
| `width` | float | Line width |
| `locked` | bool | Prevent editing |
| `tstamp` | str | Unique identifier |

---

## Zone Classes

All from `kiutils.items.zones`.

### Zone

Copper fill zone or keepout area.

```python
from kiutils.items.zones import Zone, FillSettings, ZonePolygon, Hatch
from kiutils.items.common import Position
import uuid

# Create copper pour zone
zone = Zone(
    net=1,  # Net ordinal
    netName="GND",
    layers=["F.Cu"],
    hatch=Hatch(style="edge", pitch=0.5),
    connectPads=None,  # None=thermal, "thru_hole_only", "full", "no"
    clearance=0.5,
    minThickness=0.25,
    locked=False,
    tstamp=str(uuid.uuid4())
)

# Define zone outline
polygon = ZonePolygon()
polygon.coordinates = [
    Position(X=0, Y=0),
    Position(X=100, Y=0),
    Position(X=100, Y=80),
    Position(X=0, Y=80),
]
zone.polygons.append(polygon)

# Set fill settings
zone.fillSettings = FillSettings(
    yes=True,
    thermalGap=0.5,
    thermalBridgeWidth=0.5,
    smoothingStyle="chamfer",
    smoothingRadius=0.2
)

board.zones.append(zone)
```

#### Zone Attributes

| Attribute | Type | Description |
|-----------|------|-------------|
| `net` | int | Net ordinal (0 for keepout) |
| `netName` | str | Net name (empty for keepout) |
| `layers` | list[str] | Zone layers |
| `name` | str | Zone name (optional) |
| `hatch` | Hatch | Outline display style |
| `priority` | int | Fill priority |
| `connectPads` | str | Pad connection type |
| `clearance` | float | Zone clearance |
| `minThickness` | float | Minimum fill width |
| `filledAreasThickness` | str | Use line width in fill |
| `keepoutSettings` | KeepoutSettings | Keepout configuration |
| `fillSettings` | FillSettings | Fill configuration |
| `polygons` | list[ZonePolygon] | Zone outlines |
| `filledPolygons` | list[FilledPolygon] | Filled polygons |
| `locked` | bool | Prevent editing |
| `tstamp` | str | Unique identifier |

### FillSettings

Zone fill configuration.

```python
from kiutils.items.zones import FillSettings

fill = FillSettings(
    yes=True,                    # Enable fill
    mode=None,                   # None=solid, "hatched"
    thermalGap=0.5,              # Gap around pads
    thermalBridgeWidth=0.5,      # Spoke width
    smoothingStyle="chamfer",    # "chamfer" or "fillet"
    smoothingRadius=0.2,         # Corner radius
    islandRemovalMode=0,         # 0=always, 1=never, 2=min area
    islandAreaMin=10.0           # Min island area (mode 2)
)
```

#### FillSettings Attributes

| Attribute | Type | Description |
|-----------|------|-------------|
| `yes` | bool | Fill enabled |
| `mode` | str | None (solid) or "hatched" |
| `thermalGap` | float | Thermal relief gap |
| `thermalBridgeWidth` | float | Thermal spoke width |
| `smoothingStyle` | str | "chamfer" or "fillet" |
| `smoothingRadius` | float | Corner smoothing radius |
| `islandRemovalMode` | int | 0=always, 1=never, 2=min area |
| `islandAreaMin` | float | Minimum island area |
| `hatchThickness` | float | Hatch line thickness |
| `hatchGap` | float | Hatch line gap |
| `hatchOrientation` | float | Hatch line angle |

### KeepoutSettings

Keepout zone configuration.

```python
from kiutils.items.zones import Zone, KeepoutSettings, ZonePolygon, Hatch

# Create keepout zone
keepout = Zone(
    net=0,
    netName="",
    layers=["F.Cu", "B.Cu"],
    hatch=Hatch(style="edge", pitch=0.5),
    tstamp=str(uuid.uuid4())
)

keepout.keepoutSettings = KeepoutSettings(
    tracks="not_allowed",
    vias="not_allowed",
    pads="allowed",
    copperpour="not_allowed",
    footprints="allowed"
)

polygon = ZonePolygon()
polygon.coordinates = [
    Position(X=40, Y=30),
    Position(X=60, Y=30),
    Position(X=60, Y=50),
    Position(X=40, Y=50),
]
keepout.polygons.append(polygon)

board.zones.append(keepout)
```

#### KeepoutSettings Attributes

| Attribute | Type | Description |
|-----------|------|-------------|
| `tracks` | str | "allowed" or "not_allowed" |
| `vias` | str | "allowed" or "not_allowed" |
| `pads` | str | "allowed" or "not_allowed" |
| `copperpour` | str | "allowed" or "not_allowed" |
| `footprints` | str | "allowed" or "not_allowed" |

### Hatch

Zone outline hatch display.

```python
from kiutils.items.zones import Hatch

hatch = Hatch(
    style="edge",  # "none", "edge", "full"
    pitch=0.5      # Hatch spacing
)
```

---

## Common Patterns

### Create Board Outline

```python
from kiutils.board import Board
from kiutils.items.gritems import GrLine, GrArc
from kiutils.items.common import Position
import uuid

def create_rectangular_outline(board, width, height, corner_radius=0):
    """Create rectangular board outline with optional rounded corners"""

    if corner_radius == 0:
        # Simple rectangle
        corners = [
            (0, 0), (width, 0), (width, height), (0, height)
        ]
        for i in range(4):
            x1, y1 = corners[i]
            x2, y2 = corners[(i + 1) % 4]
            line = GrLine(
                start=Position(X=x1, Y=y1),
                end=Position(X=x2, Y=y2),
                layer="Edge.Cuts",
                width=0.15,
                tstamp=str(uuid.uuid4())
            )
            board.graphicItems.append(line)
    else:
        # Rounded corners
        r = corner_radius
        # Top edge
        board.graphicItems.append(GrLine(
            start=Position(X=r, Y=0),
            end=Position(X=width-r, Y=0),
            layer="Edge.Cuts", width=0.15, tstamp=str(uuid.uuid4())
        ))
        # Top-right corner arc
        board.graphicItems.append(GrArc(
            start=Position(X=width-r, Y=0),
            mid=Position(X=width-r*0.293, Y=r*0.293),
            end=Position(X=width, Y=r),
            layer="Edge.Cuts", width=0.15, tstamp=str(uuid.uuid4())
        ))
        # Continue for other edges and corners...
```

### Find Net by Name

```python
def get_net_by_name(board, name):
    """Get net object by name"""
    return next((n for n in board.nets if n.name == name), None)

def get_net_number(board, name):
    """Get net number for assignment to traces/vias"""
    net = get_net_by_name(board, name)
    return net.number if net else 0

# Usage
gnd_net = get_net_number(board, "GND")
vcc_net = get_net_number(board, "VCC")
```

### Add Track Between Two Points

```python
from kiutils.items.brditems import Segment
from kiutils.items.common import Position
import uuid

def add_track(board, x1, y1, x2, y2, width, layer, net_name):
    """Add a track segment between two points"""
    net_num = get_net_number(board, net_name)

    track = Segment(
        start=Position(X=x1, Y=y1),
        end=Position(X=x2, Y=y2),
        width=width,
        layer=layer,
        net=net_num,
        tstamp=str(uuid.uuid4())
    )
    board.traceItems.append(track)
    return track
```

### Add Via at Position

```python
from kiutils.items.brditems import Via
from kiutils.items.common import Position
import uuid

def add_via(board, x, y, size, drill, net_name, layers=["F.Cu", "B.Cu"]):
    """Add a via at the specified position"""
    net_num = get_net_number(board, net_name)

    via = Via(
        position=Position(X=x, Y=y),
        size=size,
        drill=drill,
        layers=layers,
        net=net_num,
        tstamp=str(uuid.uuid4())
    )
    board.traceItems.append(via)
    return via
```

### Find Footprint by Reference

```python
def find_footprint(board, reference):
    """Find footprint by reference designator"""
    for fp in board.footprints:
        for prop in fp.properties:
            if prop.key == "Reference" and prop.value == reference:
                return fp
    return None

# Usage
r1 = find_footprint(board, "R1")
if r1:
    print(f"R1 position: ({r1.position.X}, {r1.position.Y})")
```

### Create GND Copper Pour

```python
from kiutils.items.zones import Zone, FillSettings, ZonePolygon, Hatch
from kiutils.items.common import Position
import uuid

def create_gnd_pour(board, layer, outline_points):
    """Create a GND copper pour zone"""
    gnd_net = get_net_number(board, "GND")

    zone = Zone(
        net=gnd_net,
        netName="GND",
        layers=[layer],
        hatch=Hatch(style="edge", pitch=0.5),
        clearance=0.5,
        minThickness=0.25,
        tstamp=str(uuid.uuid4())
    )

    zone.fillSettings = FillSettings(
        yes=True,
        thermalGap=0.5,
        thermalBridgeWidth=0.5
    )

    polygon = ZonePolygon()
    polygon.coordinates = [Position(X=p[0], Y=p[1]) for p in outline_points]
    zone.polygons.append(polygon)

    board.zones.append(zone)
    return zone

# Usage
create_gnd_pour(board, "F.Cu", [(0, 0), (100, 0), (100, 80), (0, 80)])
```

### List All Nets

```python
def list_nets(board):
    """List all nets in the board"""
    for net in board.nets:
        print(f"Net {net.number}: {net.name}")

def count_traces_per_net(board):
    """Count traces per net"""
    from collections import Counter
    net_counts = Counter()

    for item in board.traceItems:
        net_counts[item.net] += 1

    return net_counts
```

---

## Import Summary

```python
# Main board class
from kiutils.board import Board

# Board item classes
from kiutils.items.brditems import (
    GeneralSettings,
    LayerToken,
    StackupSubLayer,
    StackupLayer,
    Stackup,
    PlotSettings,
    SetupData,
    Segment,
    Via,
    Arc,
    Target,
)

# Graphic item classes
from kiutils.items.gritems import (
    GrText,
    GrTextBox,
    GrLine,
    GrRect,
    GrCircle,
    GrArc,
    GrPoly,
    GrCurve,
)

# Zone classes
from kiutils.items.zones import (
    KeepoutSettings,
    FillSettings,
    ZonePolygon,
    FilledPolygon,
    FillSegments,
    Hatch,
    Zone,
)

# Common items
from kiutils.items.common import (
    Position,
    Net,
    Property,
    Effects,
    Font,
    Justify,
    Stroke,
    Fill,
    Group,
    Image,
    PageSettings,
    TitleBlock,
)

# Footprint (for board.footprints)
from kiutils.footprint import Footprint

# Dimensions
from kiutils.items.dimensions import Dimension
```
