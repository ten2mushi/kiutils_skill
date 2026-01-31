# kiutils.items.dimensions API Reference

PCB dimension annotations: aligned, leader, center, orthogonal, and radial dimensions.

## Quick Start

```python
from kiutils.items.dimensions import Dimension, DimensionFormat, DimensionStyle
from kiutils.items.common import Position

# Create an aligned dimension
dim = Dimension(type="aligned", layer="F.SilkS")
dim.pts = [Position(X=10, Y=10), Position(X=50, Y=10)]
dim.height = 5.0
dim.format = DimensionFormat(units=2, precision=2)  # mm with 2 decimal places
dim.style = DimensionStyle(thickness=0.15, arrowLength=1.27)
```

## Class Reference

### Dimension

PCB dimension annotation.

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `locked` | `bool` | `False` | Prevent editing |
| `type` | `str` | `"aligned"` | Dimension type: `aligned`, `leader`, `center`, `orthogonal`, `radial` (v7) |
| `layer` | `str` | `"F.Cu"` | Canonical layer name |
| `tstamp` | `Optional[str]` | `None` | Unique identifier |
| `pts` | `List[Position]` | `[]` | Dimension reference points |
| `height` | `Optional[float]` | `None` | Distance from baseline (aligned) |
| `orientation` | `Optional[float]` | `None` | Rotation angle (orthogonal) |
| `leaderLength` | `Optional[float]` | `None` | Knee distance (radial) |
| `grText` | `Optional[GrText]` | `None` | Dimension text (all except center) |
| `format` | `Optional[DimensionFormat]` | `None` | Value formatting |
| `style` | `DimensionStyle` | `DimensionStyle()` | Dimension line style |

**Methods:**
- `from_sexpr(exp: list) -> Dimension` - Parse from `(dimension ...)`
- `to_sexpr(indent=2, newline=True) -> str`

**Dimension Types:**

| Type | Description | Required Points |
|------|-------------|-----------------|
| `aligned` | Measures distance along a line | 2 points + height |
| `orthogonal` | Measures horizontal or vertical distance | 2 points + orientation |
| `leader` | Arrow pointing to a feature with text | 2+ points |
| `center` | Marks center of circle/arc | 2 points (center + edge) |
| `radial` | Measures radius (v7+) | 2 points + leaderLength |

**Example:**
```python
from kiutils.items.dimensions import Dimension, DimensionFormat, DimensionStyle
from kiutils.items.gritems import GrText
from kiutils.items.common import Position, Effects

# Horizontal dimension
dim = Dimension(type="aligned", layer="Dwgs.User")
dim.pts = [Position(X=0, Y=0), Position(X=100, Y=0)]
dim.height = 10.0

# Configure format
dim.format = DimensionFormat(
    units=2,           # mm
    unitsFormat=1,     # bare suffix
    precision=2,       # 2 decimal places
    suppressZeroes=True
)

# Configure style
dim.style = DimensionStyle(
    thickness=0.15,
    arrowLength=1.27,
    textPositionMode=0,    # outside
    extensionHeight=0.5,
    keepTextAligned=True
)

# Add dimension text
dim.grText = GrText(text="100.00 mm", position=Position(X=50, Y=-5))
```

---

### DimensionFormat

Dimension value formatting.

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `prefix` | `Optional[str]` | `None` | Text before value |
| `suffix` | `Optional[str]` | `None` | Text after value |
| `units` | `int` | `3` | Units: 0=inches, 1=mils, 2=mm, 3=auto |
| `unitsFormat` | `int` | `1` | Suffix display: 0=none, 1=bare, 2=parentheses |
| `precision` | `int` | `4` | Significant digits |
| `overrideValue` | `Optional[str]` | `None` | Custom text instead of measurement |
| `suppressZeroes` | `bool` | `False` | Remove trailing zeros |

**Methods:**
- `from_sexpr(exp: list) -> DimensionFormat` - Parse from `(format ...)`
- `to_sexpr(indent=4, newline=True) -> str`

---

### DimensionStyle

Dimension line and text style.

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `thickness` | `float` | `0.0` | Line thickness |
| `arrowLength` | `float` | `0.0` | Arrow length |
| `textPositionMode` | `int` | `0` | Text position: 0=outside, 1=inline, 2=manual |
| `extensionHeight` | `Optional[float]` | `None` | Extension line overhang |
| `textFrame` | `Optional[int]` | `None` | Leader text frame: 0=none, 1=rect, 2=circle, 3=rounded |
| `extensionOffset` | `Optional[float]` | `None` | Gap from feature to extension line |
| `keepTextAligned` | `bool` | `False` | Keep text aligned with dimension line |

**Methods:**
- `from_sexpr(exp: list) -> DimensionStyle` - Parse from `(style ...)`
- `to_sexpr(indent=4, newline=True) -> str`

## Common Patterns

### Linear dimension (aligned)

```python
from kiutils.items.dimensions import Dimension, DimensionFormat, DimensionStyle
from kiutils.items.common import Position

dim = Dimension(type="aligned", layer="Dwgs.User")
dim.pts = [Position(X=10, Y=50), Position(X=90, Y=50)]
dim.height = 8.0
dim.format = DimensionFormat(units=2, precision=2)
dim.style = DimensionStyle(thickness=0.15, arrowLength=1.5, keepTextAligned=True)
```

### Orthogonal dimension (horizontal/vertical only)

```python
from kiutils.items.dimensions import Dimension, DimensionStyle
from kiutils.items.common import Position

dim = Dimension(type="orthogonal", layer="Dwgs.User")
dim.pts = [Position(X=10, Y=10), Position(X=50, Y=30)]  # Diagonal points
dim.orientation = 0  # 0 = horizontal measurement
dim.style = DimensionStyle(thickness=0.15, arrowLength=1.5)
```

### Leader dimension (callout)

```python
from kiutils.items.dimensions import Dimension, DimensionFormat, DimensionStyle
from kiutils.items.gritems import GrText
from kiutils.items.common import Position, Effects

dim = Dimension(type="leader", layer="F.SilkS")
dim.pts = [Position(X=30, Y=30), Position(X=50, Y=10)]  # Arrow to text
dim.format = DimensionFormat(overrideValue="Important!")
dim.style = DimensionStyle(
    thickness=0.15,
    arrowLength=1.5,
    textFrame=1  # Rectangle around text
)
dim.grText = GrText(text="Important!", position=Position(X=50, Y=10))
```

### Center mark

```python
from kiutils.items.dimensions import Dimension, DimensionStyle
from kiutils.items.common import Position

dim = Dimension(type="center", layer="Dwgs.User")
dim.pts = [Position(X=50, Y=50), Position(X=55, Y=50)]  # Center + edge point
dim.style = DimensionStyle(thickness=0.15, arrowLength=2.0)
```

### Custom formatted dimension

```python
from kiutils.items.dimensions import Dimension, DimensionFormat
from kiutils.items.common import Position

dim = Dimension(type="aligned", layer="Dwgs.User")
dim.pts = [Position(X=0, Y=0), Position(X=25.4, Y=0)]
dim.height = 5.0
dim.format = DimensionFormat(
    prefix="Length: ",
    suffix=" (nom)",
    units=2,
    precision=3,
    suppressZeroes=False
)
# Displays: "Length: 25.400 mm (nom)"
```

## Cross-References

- Uses: [common_apis.md](common_apis.md) (Position), [gritems_apis.md](gritems_apis.md) (GrText)
- See also: [brditems_apis.md](brditems_apis.md) for board setup
