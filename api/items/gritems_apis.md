# kiutils.items.gritems API Reference

Graphical items for boards and schematics: text, lines, rectangles, circles, arcs, polygons, and curves.

## Quick Start

```python
from kiutils.items.gritems import GrText, GrLine, GrRect, GrCircle, GrArc, GrPoly
from kiutils.items.common import Position, Effects

# Graphical text on silkscreen
text = GrText(
    text="Hello KiCad",
    position=Position(X=100.0, Y=100.0, angle=0),
    layer="F.SilkS"
)

# Line on edge cuts
line = GrLine(
    start=Position(X=0, Y=0),
    end=Position(X=100, Y=0),
    layer="Edge.Cuts",
    width=0.1
)

# Rectangle outline
rect = GrRect(
    start=Position(X=10, Y=10),
    end=Position(X=50, Y=50),
    layer="F.SilkS",
    width=0.12
)
```

## Class Reference

### GrText

Graphical text annotation.

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `text` | `str` | `""` | Text content |
| `knockout` | `bool` | `False` | Inverted text (transparent text, colored background) |
| `position` | `Position` | `Position()` | X, Y coordinates and optional angle |
| `layer` | `Optional[str]` | `None` | Canonical layer name |
| `effects` | `Effects` | `Effects()` | Text display settings |
| `tstamp` | `Optional[str]` | `None` | Unique identifier |
| `locked` | `bool` | `False` | Prevent moving |
| `renderCache` | `Optional[RenderCache]` | `None` | TrueType font cache (v7+) |

**Methods:**
- `from_sexpr(exp: list) -> GrText` - Parse from `(gr_text ...)`
- `to_sexpr(indent=2, newline=True) -> str`

**Example:**
```python
from kiutils.items.gritems import GrText
from kiutils.items.common import Position, Effects, Font

text = GrText(text="REV A", position=Position(X=150, Y=100, angle=90))
text.layer = "F.SilkS"
text.effects.font = Font(height=1.5, width=1.5)
```

---

### GrTextBox

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

**Note:** For cardinal angles (0, 90, 180, 270) or no angle, use `start`/`end`. For non-cardinal angles, use `pts` with 4 positions.

**Methods:**
- `from_sexpr(exp: list) -> GrTextBox` - Parse from `(gr_text_box ...)`
- `to_sexpr(indent=2, newline=True) -> str`

---

### GrLine

Graphical line segment.

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `start` | `Position` | `Position()` | Start coordinates |
| `end` | `Position` | `Position()` | End coordinates |
| `angle` | `Optional[float]` | `None` | Rotational angle |
| `layer` | `Optional[str]` | `None` | Canonical layer name |
| `width` | `Optional[float]` | `0.12` | Line width (pre-v7) |
| `tstamp` | `Optional[str]` | `None` | Unique identifier |
| `locked` | `bool` | `False` | Prevent moving |

**Methods:**
- `from_sexpr(exp: list) -> GrLine` - Parse from `(gr_line ...)`
- `to_sexpr(indent=2, newline=True) -> str`

---

### GrRect

Graphical rectangle.

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `start` | `Position` | `Position()` | Upper left corner |
| `end` | `Position` | `Position()` | Lower right corner |
| `layer` | `Optional[str]` | `None` | Canonical layer name |
| `width` | `Optional[float]` | `0.12` | Line width (pre-v7) |
| `fill` | `Optional[str]` | `None` | Fill type: `solid`, `none` |
| `tstamp` | `Optional[str]` | `None` | Unique identifier |
| `locked` | `bool` | `False` | Prevent moving |

**Methods:**
- `from_sexpr(exp: list) -> GrRect` - Parse from `(gr_rect ...)`
- `to_sexpr(indent=2, newline=True) -> str`

---

### GrCircle

Graphical circle.

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `center` | `Position` | `Position()` | Center coordinates |
| `end` | `Position` | `Position()` | Point on circumference |
| `layer` | `Optional[str]` | `None` | Canonical layer name |
| `width` | `Optional[float]` | `0.12` | Line width (pre-v7) |
| `fill` | `Optional[str]` | `None` | Fill type: `solid`, `none` |
| `tstamp` | `Optional[str]` | `None` | Unique identifier |
| `locked` | `bool` | `False` | Prevent moving |

**Methods:**
- `from_sexpr(exp: list) -> GrCircle` - Parse from `(gr_circle ...)`
- `to_sexpr(indent=2, newline=True) -> str`

**Example:**
```python
from kiutils.items.gritems import GrCircle
from kiutils.items.common import Position

# Circle with center at (50, 50) and radius 10mm
circle = GrCircle(
    center=Position(X=50, Y=50),
    end=Position(X=60, Y=50),  # radius = 10
    layer="F.SilkS",
    width=0.15
)
```

---

### GrArc

Graphical arc defined by start, midpoint, and end.

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `start` | `Position` | `Position()` | Arc start |
| `mid` | `Position` | `Position()` | Arc midpoint |
| `end` | `Position` | `Position()` | Arc end |
| `layer` | `Optional[str]` | `None` | Canonical layer name |
| `width` | `Optional[float]` | `0.12` | Line width (pre-v7) |
| `tstamp` | `Optional[str]` | `None` | Unique identifier |
| `locked` | `bool` | `False` | Prevent moving |

**Methods:**
- `from_sexpr(exp: list) -> GrArc` - Parse from `(gr_arc ...)`
- `to_sexpr(indent=2, newline=True) -> str`

---

### GrPoly

Graphical polygon.

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `layer` | `Optional[str]` | `None` | Canonical layer name |
| `coordinates` | `List[Position]` | `[]` | Polygon vertices |
| `width` | `Optional[float]` | `0.12` | Line width (pre-v7) |
| `fill` | `Optional[str]` | `None` | Fill type: `solid`, `none` |
| `tstamp` | `Optional[str]` | `None` | Unique identifier |
| `locked` | `bool` | `False` | Prevent moving |

**Methods:**
- `from_sexpr(exp: list) -> GrPoly` - Parse from `(gr_poly ...)`
- `to_sexpr(indent=2, newline=True, pts_newline=False) -> str`

**Example:**
```python
from kiutils.items.gritems import GrPoly
from kiutils.items.common import Position

# Triangle
triangle = GrPoly(layer="F.SilkS", fill="solid")
triangle.coordinates = [
    Position(X=0, Y=0),
    Position(X=10, Y=0),
    Position(X=5, Y=10)
]
```

---

### GrCurve

Graphical cubic Bezier curve.

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `coordinates` | `List[Position]` | `[]` | Control points (4 points for cubic Bezier) |
| `layer` | `Optional[str]` | `None` | Canonical layer name |
| `width` | `Optional[float]` | `0.12` | Line width (pre-v7) |
| `tstamp` | `Optional[str]` | `None` | Unique identifier |
| `locked` | `bool` | `False` | Prevent moving |

**Methods:**
- `from_sexpr(exp: list) -> GrCurve` - Parse from `(gr_curve ...)`
- `to_sexpr(indent=2, newline=True) -> str`

## Common Patterns

### Drawing board outline

```python
from kiutils.items.gritems import GrLine, GrArc
from kiutils.items.common import Position

# Rectangular board 100x80mm with rounded corners
width, height, radius = 100, 80, 5

edges = [
    # Top edge
    GrLine(start=Position(X=radius, Y=0), end=Position(X=width-radius, Y=0), layer="Edge.Cuts", width=0.1),
    # Right edge
    GrLine(start=Position(X=width, Y=radius), end=Position(X=width, Y=height-radius), layer="Edge.Cuts", width=0.1),
    # Bottom edge
    GrLine(start=Position(X=width-radius, Y=height), end=Position(X=radius, Y=height), layer="Edge.Cuts", width=0.1),
    # Left edge
    GrLine(start=Position(X=0, Y=height-radius), end=Position(X=0, Y=radius), layer="Edge.Cuts", width=0.1),
]
```

### Adding silkscreen annotations

```python
from kiutils.items.gritems import GrText, GrRect
from kiutils.items.common import Position, Effects, Font

# Project label
label = GrText(text="MY PROJECT v1.0", position=Position(X=50, Y=5))
label.layer = "F.SilkS"
label.effects = Effects()
label.effects.font = Font(height=2.0, width=2.0, bold=True)

# Box around components
box = GrRect(
    start=Position(X=10, Y=10),
    end=Position(X=90, Y=70),
    layer="F.SilkS",
    width=0.15
)
```

### Creating logo from polygon

```python
from kiutils.items.gritems import GrPoly
from kiutils.items.common import Position

logo = GrPoly(layer="F.SilkS", fill="solid")
logo.coordinates = [
    Position(X=0, Y=0),
    Position(X=5, Y=0),
    Position(X=7.5, Y=4),
    Position(X=5, Y=8),
    Position(X=0, Y=8),
    Position(X=2.5, Y=4)
]
```

## Cross-References

- Uses: [common_apis.md](common_apis.md) (Position, Effects, Stroke)
- See also: [fpitems_apis.md](fpitems_apis.md) for footprint graphics, [syitems_apis.md](syitems_apis.md) for symbol graphics
