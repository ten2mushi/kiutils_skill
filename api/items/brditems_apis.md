# kiutils.items.brditems API Reference

PCB board-specific items: tracks, vias, arcs, stackup configuration, and board settings.

## Quick Start

```python
from kiutils.items.brditems import Segment, Via, Arc, Target, Stackup, SetupData
from kiutils.items.common import Position

# Create a track segment
track = Segment(
    start=Position(X=100.0, Y=100.0),
    end=Position(X=110.0, Y=100.0),
    width=0.25,
    layer="F.Cu",
    net=1
)

# Create a via
via = Via(
    position=Position(X=105.0, Y=100.0),
    size=0.8,
    drill=0.4,
    layers=["F.Cu", "B.Cu"],
    net=1
)

# Create a curved track (arc)
arc = Arc(
    start=Position(X=100.0, Y=100.0),
    mid=Position(X=105.0, Y=95.0),
    end=Position(X=110.0, Y=100.0),
    width=0.25,
    layer="F.Cu",
    net=1
)
```

## Class Reference

### Segment

Track segment (straight copper trace).

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `start` | `Position` | `Position()` | Start coordinates |
| `end` | `Position` | `Position()` | End coordinates |
| `width` | `float` | `0.1` | Track width in mm |
| `layer` | `str` | `"F.Cu"` | Canonical layer name |
| `locked` | `bool` | `False` | Prevent editing |
| `net` | `int` | `0` | Net ordinal number |
| `tstamp` | `str` | `""` | Unique identifier |

**Methods:**
- `from_sexpr(exp: list) -> Segment` - Parse from `(segment ...)`
- `to_sexpr(indent=2, newline=True) -> str`

---

### Via

Track via connecting copper layers.

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `type` | `Optional[str]` | `None` | Via type: `None` (through-hole), `blind`, or `micro` |
| `locked` | `bool` | `False` | Prevent editing |
| `position` | `Position` | `Position()` | Center coordinates |
| `size` | `float` | `0.0` | Annular ring diameter |
| `drill` | `float` | `0.0` | Drill diameter |
| `layers` | `List[str]` | `[]` | Connected layers (e.g., `["F.Cu", "B.Cu"]`) |
| `removeUnusedLayers` | `bool` | `False` | Remove unused layers |
| `keepEndLayers` | `bool` | `False` | Keep end layers |
| `free` | `bool` | `False` | Via can move outside assigned net |
| `net` | `int` | `0` | Net ordinal number |
| `tstamp` | `Optional[str]` | `None` | Unique identifier |

**Methods:**
- `from_sexpr(exp: list) -> Via` - Parse from `(via ...)`
- `to_sexpr(indent=2, newline=True) -> str`

**Example:**
```python
via = Via(
    type="blind",
    position=Position(X=50.0, Y=50.0),
    size=0.6,
    drill=0.3,
    layers=["F.Cu", "In1.Cu"],
    net=5
)
```

---

### Arc

Track arc for length-matching on differential pairs.

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `start` | `Position` | `Position()` | Arc start coordinates |
| `mid` | `Position` | `Position()` | Arc midpoint |
| `end` | `Position` | `Position()` | Arc end coordinates |
| `width` | `float` | `0.2` | Track width |
| `layer` | `str` | `"F.Cu"` | Canonical layer name |
| `locked` | `bool` | `False` | Prevent editing |
| `net` | `int` | `0` | Net ordinal number |
| `tstamp` | `Optional[str]` | `None` | Unique identifier |

**Methods:**
- `from_sexpr(exp: list) -> Arc` - Parse from `(arc ...)`
- `to_sexpr(indent=2, newline=True) -> str`

---

### Target

Target marker (alignment mark) on PCB.

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `type` | `str` | `"plus"` | Marker shape: `plus` or `x` |
| `position` | `Position` | `Position()` | Marker position |
| `size` | `float` | `0` | Marker size |
| `width` | `float` | `0.1` | Line width |
| `layer` | `str` | `"F.Cu"` | Canonical layer name |
| `tstamp` | `Optional[str]` | `None` | Unique identifier |

**Methods:**
- `from_sexpr(exp: list) -> Target` - Parse from `(target ...)`
- `to_sexpr(indent=2, newline=True) -> str`

---

### GeneralSettings

Board general settings (thickness).

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `thickness` | `float` | `1.6` | Board thickness in mm |

**Methods:**
- `from_sexpr(exp: list) -> GeneralSettings` - Parse from `(general ...)`
- `to_sexpr(indent=2, newline=True) -> str`

---

### LayerToken

Layer definition in board layer stack.

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `ordinal` | `int` | `0` | Layer stack ordering |
| `name` | `str` | `"F.Cu"` | Internal layer name |
| `type` | `str` | `"signal"` | Layer type: `jumper`, `mixed`, `power`, `signal`, `user` |
| `userName` | `Optional[str]` | `None` | Custom user name |

**Methods:**
- `from_sexpr(exp: list) -> LayerToken` - Parse from `(<nr> "<name>" <type>)`
- `to_sexpr(indent=4, newline=True) -> str`

---

### StackupSubLayer

Sublayer for stacked dielectrics.

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `thickness` | `float` | `0.1` | Sublayer thickness |
| `material` | `Optional[str]` | `None` | Material description |
| `epsilonR` | `Optional[float]` | `None` | Dielectric constant |
| `lossTangent` | `Optional[float]` | `None` | Dielectric loss tangent |

**Note:** Cannot parse from S-expression - assign values manually.

---

### StackupLayer

Single layer in board stackup.

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `name` | `str` | `""` | Layer name or "dielectric ID" |
| `type` | `str` | `""` | Layer description |
| `color` | `Optional[str]` | `None` | Layer color (solder mask/silkscreen) |
| `thickness` | `Optional[float]` | `None` | Layer thickness |
| `material` | `Optional[str]` | `None` | Material description |
| `epsilonR` | `Optional[float]` | `None` | Dielectric constant |
| `lossTangent` | `Optional[float]` | `None` | Dielectric loss tangent |
| `subLayers` | `List[StackupSubLayer]` | `[]` | Stacked dielectric sublayers |

**Methods:**
- `from_sexpr(exp: list) -> StackupLayer` - Parse from `(layer ...)`
- `to_sexpr(indent=6, newline=True) -> str`

---

### Stackup

Board stackup configuration.

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `layers` | `List[StackupLayer]` | `[]` | Layer settings list |
| `copperFinish` | `Optional[str]` | `None` | Copper finish (e.g., "ENIG") |
| `dielectricContraints` | `Optional[str]` | `None` | Meet dielectric requirements: `yes`/`no` |
| `edgeConnector` | `Optional[str]` | `None` | Edge connector: `yes` or `bevelled` |
| `castellatedPads` | `bool` | `False` | Castellated pads on board edges |
| `edgePlating` | `bool` | `False` | Plated board edges |

**Methods:**
- `from_sexpr(exp: list) -> Stackup` - Parse from `(stackup ...)`
- `to_sexpr(indent=4, newline=True) -> str`

---

### PlotSettings

Plotting and printing settings.

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `layerSelection` | `str` | `""` | Hex bit set of layers to plot |
| `plotOnAllLayersSelection` | `Optional[str]` | `None` | Layers plotted on all (v7+) |
| `disableApertMacros` | `bool` | `False` | Disable aperture macros in Gerber |
| `useGerberExtensions` | `bool` | `False` | Use Protel file extensions |
| `useGerberAttributes` | `bool` | `False` | Use X2 extensions |
| `useGerberAdvancedAttributes` | `bool` | `False` | Include netlist info |
| `createGerberJobFile` | `bool` | `False` | Create job file |
| `svgPrecision` | `float` | `0.0` | SVG units precision |
| `plotFameRef` | `bool` | `False` | Plot border and title block |
| `viasOnMask` | `bool` | `False` | Tent vias |
| `mode` | `int` | `1` | Plot mode: 1=normal, 2=outline |
| `useAuxOrigin` | `bool` | `False` | Use user-defined origin |
| `hpglPenNumber` | `int` | `0` | HPGL pen number |
| `hpglPenSpeed` | `int` | `0` | HPGL pen speed |
| `hpglPenDiameter` | `float` | `0.0` | HPGL pen diameter |
| `dxfPolygonMode` | `bool` | `False` | DXF polygon mode |
| `dxfImperialUnits` | `bool` | `False` | DXF imperial units |
| `dxfUsePcbnewFont` | `bool` | `False` | DXF use vector font |
| `psNegative` | `bool` | `False` | PostScript negative output |
| `psA4Output` | `bool` | `False` | PostScript A4 size |
| `plotReference` | `bool` | `False` | Plot hidden reference text |
| `plotValue` | `bool` | `False` | Plot hidden value text |
| `plotInvisibleText` | `bool` | `False` | Plot other hidden text |
| `sketchPadsOnFab` | `bool` | `False` | Sketch mode for pads |
| `subtractMaskFromSilk` | `bool` | `False` | Subtract mask from silkscreen |
| `outputFormat` | `int` | `0` | Format: 0=Gerber, 1=PS, 2=SVG, 3=DXF, 4=HPGL, 5=PDF |
| `mirror` | `bool` | `False` | Mirror plot |
| `drillShape` | `int` | `0` | Drill mark type |
| `scaleSelection` | `int` | `1` | Scale selection |
| `outputDirectory` | `str` | `""` | Output path relative to project |

**Methods:**
- `from_sexpr(exp: list) -> PlotSettings` - Parse from `(pcbplotparams ...)`
- `to_sexpr(indent=4, newline=True) -> str`

---

### SetupData

Board setup section containing stackup and clearance settings.

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `stackup` | `Optional[Stackup]` | `None` | Board stackup |
| `packToMaskClearance` | `float` | `0.0` | Pad to solder mask clearance |
| `solderMaskMinWidth` | `Optional[float]` | `None` | Minimum solder mask width |
| `padToPasteClearance` | `Optional[float]` | `None` | Pad to paste clearance |
| `padToPasteClearanceRatio` | `Optional[float]` | `None` | Paste ratio (0-100%) |
| `auxAxisOrigin` | `Optional[Position]` | `None` | Auxiliary origin |
| `gridOrigin` | `Optional[Position]` | `None` | Grid origin |
| `plotSettings` | `Optional[PlotSettings]` | `None` | Plot settings |

**Methods:**
- `from_sexpr(exp: list) -> SetupData` - Parse from `(setup ...)`
- `to_sexpr(indent=2, newline=True) -> str`

## Common Patterns

### Creating a track route

```python
from kiutils.items.brditems import Segment
from kiutils.items.common import Position

# Route from (100, 100) to (120, 100) to (120, 120)
segments = [
    Segment(
        start=Position(X=100.0, Y=100.0),
        end=Position(X=120.0, Y=100.0),
        width=0.25, layer="F.Cu", net=1, tstamp="uuid1"
    ),
    Segment(
        start=Position(X=120.0, Y=100.0),
        end=Position(X=120.0, Y=120.0),
        width=0.25, layer="F.Cu", net=1, tstamp="uuid2"
    )
]
```

### Adding vias for layer transitions

```python
from kiutils.items.brditems import Via
from kiutils.items.common import Position

# Through-hole via
th_via = Via(
    position=Position(X=110.0, Y=100.0),
    size=0.8,
    drill=0.4,
    layers=["F.Cu", "B.Cu"],
    net=1
)

# Blind via (F.Cu to In1.Cu)
blind_via = Via(
    type="blind",
    position=Position(X=115.0, Y=100.0),
    size=0.6,
    drill=0.3,
    layers=["F.Cu", "In1.Cu"],
    net=1
)
```

### Configuring board stackup

```python
from kiutils.items.brditems import Stackup, StackupLayer

stackup = Stackup()
stackup.layers = [
    StackupLayer(name="F.Cu", type="copper", thickness=0.035),
    StackupLayer(name="dielectric 1", type="prepreg", thickness=0.2, epsilonR=4.5),
    StackupLayer(name="In1.Cu", type="copper", thickness=0.035),
    # ... more layers
]
stackup.copperFinish = "ENIG"
```

## Cross-References

- Uses: [common_apis.md](common_apis.md) (Position)
- See also: [zones_apis.md](zones_apis.md) for copper zones, [gritems_apis.md](gritems_apis.md) for board graphics
