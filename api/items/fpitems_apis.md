# kiutils.items.fpitems API Reference

Footprint graphical items: text, text boxes, lines, rectangles, circles, arcs, polygons, and curves used in footprint definitions.

**Note:** The `Pad` class is in `kiutils.footprint`, not fpitems.

## Quick Start

```python
from kiutils.items.fpitems import FpText, FpLine, FpRect, FpCircle, FpArc, FpPoly
from kiutils.items.common import Position, Effects, Stroke

# Reference designator text
ref_text = FpText(
    type="reference",
    text="R1",
    position=Position(X=0, Y=-2.5),
    layer="F.SilkS"
)

# Footprint outline on silkscreen
line = FpLine(
    start=Position(X=-1, Y=-0.5),
    end=Position(X=1, Y=-0.5),
    layer="F.SilkS",
    width=0.12
)

# Courtyard rectangle
rect = FpRect(
    start=Position(X=-1.5, Y=-1),
    end=Position(X=1.5, Y=1),
    layer="F.CrtYd",
    width=0.05
)
```

## Class Reference

### FpText

Footprint text (reference, value, or user text).

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `type` | `str` | `"reference"` | Text type: `reference`, `value`, `user` |
| `text` | `str` | `"%REF"` | Text content |
| `position` | `Position` | `Position()` | X, Y coordinates and optional angle |
| `layer` | `str` | `"F.Cu"` | Canonical layer name |
| `knockout` | `bool` | `False` | Inverted text style |
| `hide` | `bool` | `False` | Hide text |
| `effects` | `Effects` | `Effects()` | Text display settings |
| `tstamp` | `Optional[str]` | `None` | Unique identifier |
| `renderCache` | `Optional[RenderCache]` | `None` | TrueType font cache (v7+) |

**Methods:**
- `from_sexpr(exp: list) -> FpText` - Parse from `(fp_text ...)`
- `to_sexpr(indent=2, newline=True) -> str`

**Example:**
```python
from kiutils.items.fpitems import FpText
from kiutils.items.common import Position, Effects, Font

ref = FpText(type="reference", text="U1", position=Position(X=0, Y=-3))
ref.layer = "F.SilkS"
ref.effects.font = Font(height=1.0, width=1.0)
```

---

### FpTextBox

Text box with line-wrapped text (v7+).

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `locked` | `bool` | `False` | Prevent moving |
| `text` | `str` | `""` | Text content |
| `start` | `Optional[Position]` | `None` | Top-left corner (cardinal angles) |
| `end` | `Optional[Position]` | `None` | Bottom-right corner (cardinal angles) |
| `pts` | `List[Position]` | `[]` | Four corners (non-cardinal angles) |
| `angle` | `Optional[float]` | `None` | Rotation in degrees |
| `layer` | `str` | `"F.Cu"` | Canonical layer name |
| `tstamp` | `Optional[str]` | `None` | Unique identifier |
| `effects` | `Optional[Effects]` | `None` | Text style |
| `stroke` | `Optional[Stroke]` | `None` | Border style |
| `renderCache` | `Optional[RenderCache]` | `None` | TrueType font cache |

**Methods:**
- `from_sexpr(exp: list) -> FpTextBox` - Parse from `(fp_text_box ...)`
- `to_sexpr(indent=2, newline=True) -> str`

---

### FpLine

Footprint line segment.

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `start` | `Position` | `Position()` | Start coordinates |
| `end` | `Position` | `Position()` | End coordinates |
| `layer` | `str` | `"F.Cu"` | Canonical layer name |
| `width` | `Optional[float]` | `0.12` | Line width (pre-v7) |
| `stroke` | `Optional[Stroke]` | `None` | Line style (v7+) |
| `locked` | `bool` | `False` | Prevent editing |
| `tstamp` | `Optional[str]` | `None` | Unique identifier |

**Methods:**
- `from_sexpr(exp: list) -> FpLine` - Parse from `(fp_line ...)`
- `to_sexpr(indent=2, newline=True) -> str`

---

### FpRect

Footprint rectangle.

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `start` | `Position` | `Position()` | Upper left corner |
| `end` | `Position` | `Position()` | Lower right corner |
| `layer` | `str` | `"F.Cu"` | Canonical layer name |
| `width` | `Optional[float]` | `0.12` | Line width (pre-v7) |
| `stroke` | `Optional[Stroke]` | `None` | Line style (v7+) |
| `fill` | `Optional[str]` | `None` | Fill type: `solid`, `none` |
| `locked` | `bool` | `False` | Prevent editing |
| `tstamp` | `Optional[str]` | `None` | Unique identifier |

**Methods:**
- `from_sexpr(exp: list) -> FpRect` - Parse from `(fp_rect ...)`
- `to_sexpr(indent=2, newline=True) -> str`

---

### FpCircle

Footprint circle.

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `center` | `Position` | `Position()` | Center coordinates |
| `end` | `Position` | `Position()` | Point on circumference |
| `layer` | `str` | `"F.Cu"` | Canonical layer name |
| `width` | `Optional[float]` | `0.12` | Line width (pre-v7) |
| `stroke` | `Optional[Stroke]` | `None` | Line style (v7+) |
| `fill` | `Optional[str]` | `None` | Fill type: `solid`, `none` |
| `locked` | `bool` | `False` | Prevent editing |
| `tstamp` | `Optional[str]` | `None` | Unique identifier |

**Methods:**
- `from_sexpr(exp: list) -> FpCircle` - Parse from `(fp_circle ...)`
- `to_sexpr(indent=2, newline=True) -> str`

---

### FpArc

Footprint arc defined by start, midpoint, and end.

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `start` | `Position` | `Position()` | Arc start |
| `mid` | `Position` | `Position()` | Arc midpoint |
| `end` | `Position` | `Position()` | Arc end |
| `layer` | `str` | `"F.Cu"` | Canonical layer name |
| `width` | `Optional[float]` | `0.12` | Line width (pre-v7) |
| `stroke` | `Optional[Stroke]` | `None` | Line style (v7+) |
| `locked` | `bool` | `False` | Prevent editing |
| `tstamp` | `Optional[str]` | `None` | Unique identifier |

**Methods:**
- `from_sexpr(exp: list) -> FpArc` - Parse from `(fp_arc ...)`
- `to_sexpr(indent=2, newline=True) -> str`

---

### FpPoly

Footprint polygon.

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `layer` | `str` | `"F.Cu"` | Canonical layer name |
| `coordinates` | `List[Position]` | `[]` | Polygon vertices |
| `width` | `Optional[float]` | `0.12` | Line width (pre-v7) |
| `stroke` | `Optional[Stroke]` | `None` | Line style (v7+) |
| `fill` | `Optional[str]` | `None` | Fill type: `solid`, `none` |
| `locked` | `bool` | `False` | Prevent editing |
| `tstamp` | `Optional[str]` | `None` | Unique identifier |

**Methods:**
- `from_sexpr(exp: list) -> FpPoly` - Parse from `(fp_poly ...)`
- `to_sexpr(indent=2, newline=True) -> str`

---

### FpCurve

Footprint cubic Bezier curve.

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `coordinates` | `List[Position]` | `[]` | Control points |
| `layer` | `str` | `"F.Cu"` | Canonical layer name |
| `width` | `Optional[float]` | `0.12` | Line width (pre-v7) |
| `stroke` | `Optional[Stroke]` | `None` | Line style (v7+) |
| `locked` | `bool` | `False` | Prevent editing |
| `tstamp` | `Optional[str]` | `None` | Unique identifier |

**Methods:**
- `from_sexpr(exp: list) -> FpCurve` - Parse from `(fp_curve ...)`
- `to_sexpr(indent=2, newline=True) -> str`

## Common Patterns

### Creating a resistor footprint outline

```python
from kiutils.items.fpitems import FpLine, FpText
from kiutils.items.common import Position, Effects, Font

# Silkscreen outline for 0805 resistor
lines = [
    FpLine(start=Position(X=-1, Y=-0.625), end=Position(X=1, Y=-0.625), layer="F.SilkS", width=0.12),
    FpLine(start=Position(X=-1, Y=0.625), end=Position(X=1, Y=0.625), layer="F.SilkS", width=0.12),
]

# Reference designator
ref = FpText(type="reference", text="REF**", position=Position(X=0, Y=-1.5))
ref.layer = "F.SilkS"
ref.effects.font = Font(height=1.0, width=1.0)
```

### Using stroke for KiCad 7+ styling

```python
from kiutils.items.fpitems import FpLine
from kiutils.items.common import Position, Stroke

# KiCad 7+ style with stroke
line = FpLine(
    start=Position(X=0, Y=0),
    end=Position(X=10, Y=0),
    layer="F.SilkS"
)
line.width = None  # Clear legacy width
line.stroke = Stroke(width=0.15, type="solid")
```

### Adding courtyard and fab layers

```python
from kiutils.items.fpitems import FpRect
from kiutils.items.common import Position

# Courtyard (0.25mm outside component)
courtyard = FpRect(
    start=Position(X=-1.75, Y=-1.0),
    end=Position(X=1.75, Y=1.0),
    layer="F.CrtYd",
    width=0.05
)

# Fabrication layer (component outline)
fab = FpRect(
    start=Position(X=-1.0, Y=-0.5),
    end=Position(X=1.0, Y=0.5),
    layer="F.Fab",
    width=0.1
)
```

## Cross-References

- Uses: [common_apis.md](common_apis.md) (Position, Effects, Stroke)
- See also: [gritems_apis.md](gritems_apis.md) for board graphics
- Note: For pads, see `kiutils.footprint.Pad`
