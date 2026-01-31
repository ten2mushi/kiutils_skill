# kiutils.items.syitems API Reference

Symbol graphical items: rectangles, circles, arcs, polylines, curves, text, and text boxes used in symbol library definitions.

## Quick Start

```python
from kiutils.items.syitems import SyRect, SyCircle, SyPolyLine, SyText, SyArc
from kiutils.items.common import Position, Fill, Stroke, Effects

# Symbol body rectangle
rect = SyRect(
    start=Position(X=-5.08, Y=5.08),
    end=Position(X=5.08, Y=-5.08)
)
rect.fill = Fill(type="background")

# Circle (e.g., for op-amp output)
circle = SyCircle(
    center=Position(X=6.35, Y=0),
    radius=0.508
)

# Polyline for symbol outline
poly = SyPolyLine()
poly.points = [
    Position(X=-5.08, Y=5.08),
    Position(X=5.08, Y=0),
    Position(X=-5.08, Y=-5.08),
    Position(X=-5.08, Y=5.08)
]
poly.fill = Fill(type="background")
```

## Class Reference

### SyRect

Symbol rectangle.

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `private` | `bool` | `False` | Only visible in symbol editor (v7+) |
| `start` | `Position` | `Position()` | Start point coordinates |
| `end` | `Position` | `Position()` | End point coordinates |
| `stroke` | `Stroke` | `Stroke()` | Outline style |
| `fill` | `Fill` | `Fill()` | Fill settings |

**Methods:**
- `from_sexpr(exp: list) -> SyRect` - Parse from `(rectangle ...)`
- `to_sexpr(indent=6, newline=True) -> str`

**Example:**
```python
from kiutils.items.syitems import SyRect
from kiutils.items.common import Position, Fill, Stroke

# IC body
body = SyRect(
    start=Position(X=-7.62, Y=7.62),
    end=Position(X=7.62, Y=-7.62)
)
body.stroke = Stroke(width=0.254, type="solid")
body.fill = Fill(type="background")
```

---

### SyCircle

Symbol circle.

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `private` | `bool` | `False` | Only visible in symbol editor (v7+) |
| `center` | `Position` | `Position()` | Center coordinates |
| `radius` | `float` | `0.0` | Circle radius |
| `stroke` | `Stroke` | `Stroke()` | Outline style |
| `fill` | `Fill` | `Fill()` | Fill settings |

**Methods:**
- `from_sexpr(exp: list) -> SyCircle` - Parse from `(circle ...)`
- `to_sexpr(indent=6, newline=True) -> str`

---

### SyArc

Symbol arc defined by start, midpoint, and end.

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `private` | `bool` | `False` | Only visible in symbol editor (v7+) |
| `start` | `Position` | `Position()` | Arc start |
| `mid` | `Position` | `Position()` | Arc midpoint |
| `end` | `Position` | `Position()` | Arc end |
| `stroke` | `Stroke` | `Stroke()` | Outline style |
| `fill` | `Fill` | `Fill()` | Fill settings |

**Methods:**
- `from_sexpr(exp: list) -> SyArc` - Parse from `(arc ...)`
- `to_sexpr(indent=6, newline=True) -> str`

---

### SyPolyLine

Symbol polyline (one or more connected lines, optionally forming a polygon).

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `points` | `List[Position]` | `[]` | List of vertices |
| `stroke` | `Stroke` | `Stroke()` | Outline style |
| `fill` | `Fill` | `Fill()` | Fill settings |

**Methods:**
- `from_sexpr(exp: list) -> SyPolyLine` - Parse from `(polyline ...)`
- `to_sexpr(indent=6, newline=True) -> str`

**Example:**
```python
from kiutils.items.syitems import SyPolyLine
from kiutils.items.common import Position, Fill

# Triangle (op-amp symbol)
triangle = SyPolyLine()
triangle.points = [
    Position(X=-3.81, Y=3.81),
    Position(X=3.81, Y=0),
    Position(X=-3.81, Y=-3.81),
    Position(X=-3.81, Y=3.81)  # Close the shape
]
triangle.fill = Fill(type="background")
```

---

### SyCurve

Symbol cubic Bezier curve.

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `points` | `List[Position]` | `[]` | Four control points |
| `stroke` | `Stroke` | `Stroke()` | Outline style |
| `fill` | `Fill` | `Fill()` | Fill settings |

**Methods:**
- `from_sexpr(exp: list) -> SyCurve` - Parse from `(curve ...)`
- `to_sexpr(indent=6, newline=True) -> str`

---

### SyText

Symbol text annotation.

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `text` | `str` | `""` | Text content |
| `position` | `Position` | `Position()` | X, Y coordinates and angle |
| `effects` | `Effects` | `Effects()` | Text display settings |

**Methods:**
- `from_sexpr(exp: list) -> SyText` - Parse from `(text ...)`
- `to_sexpr(indent=6, newline=True) -> str`

---

### SyTextBox

Symbol text box (v7+).

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `text` | `str` | `""` | Text content |
| `private` | `bool` | `False` | Only visible in symbol editor |
| `position` | `Position` | `Position()` | Position and angle |
| `size` | `Position` | `Position()` | Width and height (X, Y) |
| `stroke` | `Stroke` | `Stroke()` | Border style |
| `fill` | `Fill` | `Fill()` | Fill settings |
| `effects` | `Effects` | `Effects()` | Text style |
| `uuid` | `Optional[str]` | `None` | Unique identifier |

**Methods:**
- `from_sexpr(exp: list) -> SyTextBox` - Parse from `(text_box ...)`
- `to_sexpr(indent=2, newline=True) -> str`

## Common Patterns

### Creating an op-amp symbol body

```python
from kiutils.items.syitems import SyPolyLine, SyText
from kiutils.items.common import Position, Fill, Stroke

# Triangle body
body = SyPolyLine()
body.points = [
    Position(X=-3.81, Y=3.81),
    Position(X=3.81, Y=0),
    Position(X=-3.81, Y=-3.81),
    Position(X=-3.81, Y=3.81)
]
body.stroke = Stroke(width=0.254, type="default")
body.fill = Fill(type="background")

# Plus sign for non-inverting input
plus = SyText(text="+", position=Position(X=-2.54, Y=-1.27))

# Minus sign for inverting input
minus = SyText(text="-", position=Position(X=-2.54, Y=1.27))
```

### Creating an IC body with pin markers

```python
from kiutils.items.syitems import SyRect, SyCircle
from kiutils.items.common import Position, Fill, Stroke

# IC body
body = SyRect(
    start=Position(X=-10.16, Y=10.16),
    end=Position(X=10.16, Y=-10.16)
)
body.stroke = Stroke(width=0.254, type="default")
body.fill = Fill(type="background")

# Pin 1 indicator circle
pin1_marker = SyCircle(
    center=Position(X=-8.89, Y=8.89),
    radius=0.762
)
pin1_marker.fill = Fill(type="outline")
```

### Using private elements (v7+)

```python
from kiutils.items.syitems import SyRect

# Element only visible in symbol editor
private_rect = SyRect(
    start=Position(X=-12, Y=12),
    end=Position(X=12, Y=-12),
    private=True
)
```

## Cross-References

- Uses: [common_apis.md](common_apis.md) (Position, Effects, Stroke, Fill)
- See also: [schitems_apis.md](schitems_apis.md) for schematic symbol instances
- Note: For symbol pins, see `kiutils.symbol.SymbolPin`
