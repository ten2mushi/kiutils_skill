# kiutils.items.zones API Reference

PCB zone items: copper fill zones and keepout areas.

## Quick Start

```python
from kiutils.items.zones import Zone, FillSettings, KeepoutSettings, ZonePolygon, Hatch
from kiutils.items.common import Position

# Create a copper pour zone
zone = Zone()
zone.net = 1
zone.netName = "GND"
zone.layers = ["F.Cu"]
zone.hatch = Hatch(style="edge", pitch=0.5)

# Define zone outline
polygon = ZonePolygon()
polygon.coordinates = [
    Position(X=0, Y=0),
    Position(X=100, Y=0),
    Position(X=100, Y=100),
    Position(X=0, Y=100)
]
zone.polygons.append(polygon)

# Configure fill settings
zone.fillSettings = FillSettings(yes=True, thermalGap=0.5, thermalBridgeWidth=0.3)
```

## Class Reference

### Zone

Copper zone or keepout area.

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `locked` | `bool` | `False` | Prevent editing |
| `net` | `int` | `0` | Net ordinal number |
| `netName` | `str` | `"unknown"` | Net name (empty for keepout) |
| `layers` | `List[str]` | `[]` | Layer(s) the zone spans |
| `tstamp` | `Optional[str]` | `None` | Unique identifier |
| `name` | `Optional[str]` | `None` | Zone name |
| `hatch` | `Hatch` | `Hatch()` | Outline hatch style |
| `priority` | `Optional[int]` | `None` | Zone priority (higher fills first) |
| `connectPads` | `Optional[str]` | `None` | Pad connection: `thru_hole_only`, `full`, `no` |
| `clearance` | `float` | `0.254` | Thermal relief clearance |
| `minThickness` | `float` | `0.254` | Minimum fill width |
| `filledAreasThickness` | `Optional[str]` | `None` | Line thickness in fill calculation |
| `keepoutSettings` | `Optional[KeepoutSettings]` | `None` | Keepout configuration |
| `fillSettings` | `Optional[FillSettings]` | `None` | Fill configuration |
| `polygons` | `List[ZonePolygon]` | `[]` | Zone boundary polygons |
| `filledPolygons` | `List[FilledPolygon]` | `[]` | Filled polygon data |
| `fillSegments` | `Optional[FillSegments]` | `None` | Legacy fill segments (KiCad 4) |

**Methods:**
- `from_sexpr(exp: list) -> Zone` - Parse from `(zone ...)`
- `to_sexpr(indent=2, newline=True) -> str`

**Example:**
```python
from kiutils.items.zones import Zone, FillSettings, ZonePolygon, Hatch
from kiutils.items.common import Position

# Ground plane on bottom copper
gnd_zone = Zone()
gnd_zone.net = 1
gnd_zone.netName = "GND"
gnd_zone.layers = ["B.Cu"]
gnd_zone.hatch = Hatch(style="edge", pitch=0.508)
gnd_zone.priority = 0
gnd_zone.connectPads = None  # thermal relief
gnd_zone.clearance = 0.508
gnd_zone.minThickness = 0.254
gnd_zone.fillSettings = FillSettings(yes=True, thermalGap=0.508, thermalBridgeWidth=0.508)

# Define zone boundary
outline = ZonePolygon()
outline.coordinates = [
    Position(X=10, Y=10), Position(X=90, Y=10),
    Position(X=90, Y=90), Position(X=10, Y=90)
]
gnd_zone.polygons.append(outline)
```

---

### Hatch

Zone outline display settings.

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `style` | `str` | `"none"` | Hatch style: `none`, `edge`, `full` |
| `pitch` | `float` | `0.0` | Hatch line spacing |

---

### FillSettings

Zone fill configuration.

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `yes` | `bool` | `False` | Enable zone fill |
| `mode` | `Optional[str]` | `None` | Fill mode: `hatched` or None (solid) |
| `thermalGap` | `Optional[float]` | `None` | Gap to pad thermal relief |
| `thermalBridgeWidth` | `Optional[float]` | `None` | Thermal spoke width |
| `smoothingStyle` | `Optional[str]` | `None` | Corner smoothing: `chamfer`, `fillet` |
| `smoothingRadius` | `Optional[float]` | `None` | Corner smoothing radius |
| `islandRemovalMode` | `Optional[int]` | `None` | Island removal: 0=always, 1=never, 2=min area |
| `islandAreaMin` | `Optional[float]` | `None` | Minimum island area (mode=2) |
| `hatchThickness` | `Optional[float]` | `None` | Hatched fill line width |
| `hatchGap` | `Optional[float]` | `None` | Hatched fill line spacing |
| `hatchOrientation` | `Optional[float]` | `None` | Hatched fill line angle |
| `hatchSmoothingLevel` | `Optional[int]` | `None` | Hatch outline smoothing: 0-3 |
| `hatchSmoothingValue` | `Optional[float]` | `None` | Smoothing ratio |
| `hatchBorderAlgorithm` | `Optional[int]` | `None` | Border algorithm: 0=zone min, 1=hatch |
| `hatchMinHoleArea` | `Optional[float]` | `None` | Minimum hatch hole area |

**Methods:**
- `from_sexpr(exp: list) -> FillSettings` - Parse from `(fill ...)`
- `to_sexpr(indent=0, newline=False) -> str`

---

### KeepoutSettings

Keepout zone restrictions.

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `tracks` | `str` | `"allowed"` | Track restriction: `allowed`, `not_allowed` |
| `vias` | `str` | `"allowed"` | Via restriction: `allowed`, `not_allowed` |
| `pads` | `str` | `"allowed"` | Pad restriction: `allowed`, `not_allowed` |
| `copperpour` | `str` | `"not-allowed"` | Copper pour: `allowed`, `not_allowed` |
| `footprints` | `str` | `"not-allowed"` | Footprint restriction |

**Methods:**
- `from_sexpr(exp: list) -> KeepoutSettings` - Parse from `(keepout ...)`
- `to_sexpr(indent=0, newline=False) -> str`

---

### ZonePolygon

Zone boundary polygon.

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `coordinates` | `List[Position]` | `[]` | Polygon vertices |

**Methods:**
- `from_sexpr(exp: list) -> ZonePolygon` - Parse from `(polygon ...)`
- `to_sexpr(indent=4, newline=True) -> str`

---

### FilledPolygon

Computed fill polygon (generated by zone fill).

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `layer` | `str` | `"F.Cu"` | Layer name |
| `island` | `bool` | `False` | Is an island |
| `coordinates` | `List[Position]` | `[]` | Fill polygon vertices |

**Methods:**
- `from_sexpr(exp: list) -> FilledPolygon` - Parse from `(filled_polygon ...)`
- `to_sexpr(indent=4, newline=True) -> str`

---

### FillSegments

Legacy fill segments (KiCad 4 compatibility).

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `layer` | `str` | `"F.Cu"` | Layer name |
| `coordinates` | `List[Position]` | `[]` | Segment coordinates |

**Methods:**
- `from_sexpr(exp: list) -> FillSegments` - Parse from `(fill_segments ...)`
- `to_sexpr(indent=4, newline=True) -> str`

## Common Patterns

### Creating a ground plane

```python
from kiutils.items.zones import Zone, FillSettings, ZonePolygon, Hatch
from kiutils.items.common import Position

gnd = Zone()
gnd.net = 1
gnd.netName = "GND"
gnd.layers = ["F.Cu"]
gnd.hatch = Hatch(style="edge", pitch=0.508)
gnd.fillSettings = FillSettings(
    yes=True,
    thermalGap=0.508,
    thermalBridgeWidth=0.508,
    smoothingStyle="chamfer",
    smoothingRadius=0.254
)

outline = ZonePolygon()
outline.coordinates = [
    Position(X=0, Y=0), Position(X=100, Y=0),
    Position(X=100, Y=80), Position(X=0, Y=80)
]
gnd.polygons.append(outline)
```

### Creating a keepout zone

```python
from kiutils.items.zones import Zone, KeepoutSettings, ZonePolygon, Hatch
from kiutils.items.common import Position

keepout = Zone()
keepout.net = 0
keepout.netName = ""
keepout.layers = ["F.Cu", "B.Cu"]
keepout.hatch = Hatch(style="edge", pitch=0.5)
keepout.keepoutSettings = KeepoutSettings(
    tracks="not_allowed",
    vias="not_allowed",
    pads="allowed",
    copperpour="not_allowed",
    footprints="allowed"
)

outline = ZonePolygon()
outline.coordinates = [
    Position(X=40, Y=40), Position(X=60, Y=40),
    Position(X=60, Y=60), Position(X=40, Y=60)
]
keepout.polygons.append(outline)
```

### Multi-layer zone (internal planes)

```python
from kiutils.items.zones import Zone, FillSettings, ZonePolygon, Hatch
from kiutils.items.common import Position

power_plane = Zone()
power_plane.net = 2
power_plane.netName = "VCC"
power_plane.layers = ["In1.Cu", "In2.Cu"]  # Internal layers
power_plane.hatch = Hatch(style="full", pitch=0.508)
power_plane.connectPads = "full"  # Solid connection
power_plane.fillSettings = FillSettings(yes=True)

outline = ZonePolygon()
outline.coordinates = [
    Position(X=5, Y=5), Position(X=95, Y=5),
    Position(X=95, Y=75), Position(X=5, Y=75)
]
power_plane.polygons.append(outline)
```

## Cross-References

- Uses: [common_apis.md](common_apis.md) (Position)
- See also: [brditems_apis.md](brditems_apis.md) for tracks and vias
