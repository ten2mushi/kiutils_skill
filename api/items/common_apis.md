# kiutils.items.common API Reference

Base types shared across symbol libraries, footprint libraries, schematics, boards, and worksheets.

## Quick Start

```python
from kiutils.items.common import Position, Stroke, Effects, Font, Fill, Net, ColorRGBA

# Position with X, Y coordinates and optional angle
pos = Position(X=10.0, Y=20.0, angle=90.0)

# Stroke for line styling
stroke = Stroke(width=0.15, type="solid", color=ColorRGBA(R=255, G=0, B=0, A=1))

# Text effects with font and justification
effects = Effects()
effects.font = Font(height=1.27, width=1.27, bold=True)
effects.justify.horizontally = "left"

# Fill for shapes
fill = Fill(type="background")

# Net reference
net = Net(number=1, name="GND")
```

## Class Reference

### Position

2D position with optional rotation angle. Units are millimeters.

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `X` | `float` | `0.0` | Horizontal position |
| `Y` | `float` | `0.0` | Vertical position |
| `angle` | `Optional[float]` | `None` | Rotation angle in degrees (tenths of degree for symbol text) |
| `unlocked` | `bool` | `False` | Whether position can be edited independently |

**Methods:**
- `from_sexpr(exp: list) -> Position` - Parse from S-expression

**Note:** Position does not have a `to_sexpr()` method as it's embedded in other tokens.

---

### Coordinate

3D position for 3D model placement.

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `X` | `float` | `0.0` | X-axis position |
| `Y` | `float` | `0.0` | Y-axis position |
| `Z` | `float` | `0.0` | Z-axis position |

**Methods:**
- `from_sexpr(exp: list) -> Coordinate` - Parse from `(xyz X Y Z)`
- `to_sexpr(indent=0, newline=False) -> str`

---

### ColorRGBA

RGBA color definition.

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `R` | `int` | `0` | Red channel (0-255) |
| `G` | `int` | `0` | Green channel (0-255) |
| `B` | `int` | `0` | Blue channel (0-255) |
| `A` | `int` | `0` | Alpha channel (0=transparent) |
| `precision` | `Optional[int]` | `None` | Decimal precision for alpha output |

**Methods:**
- `from_sexpr(exp: list) -> ColorRGBA` - Parse from `(color R G B A)`
- `to_sexpr(indent=0, newline=False) -> str`

---

### Stroke

Line styling for graphical objects.

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `width` | `float` | `0.0` | Line width in mm |
| `type` | `Optional[str]` | `None` | Line style: `dash`, `dash_dot`, `dash_dot_dot`, `dot`, `default`, `solid` |
| `color` | `Optional[ColorRGBA]` | `None` | Line color (optional since KiCad 7) |

**Methods:**
- `from_sexpr(exp: list) -> Stroke` - Parse from `(stroke ...)`
- `to_sexpr(indent=2, newline=True) -> str`

**Example:**
```python
stroke = Stroke(width=0.25, type="dash")
stroke.color = ColorRGBA(R=0, G=0, B=255, A=1)
```

---

### Font

Text font attributes.

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `face` | `Optional[str]` | `None` | Font family name or "KiCad Font" (v7+) |
| `height` | `float` | `1.0` | Font height in mm |
| `width` | `float` | `1.0` | Font width in mm |
| `thickness` | `Optional[float]` | `None` | Line thickness for stroke font |
| `bold` | `bool` | `False` | Bold text |
| `italic` | `bool` | `False` | Italic text |
| `lineSpacing` | `Optional[float]` | `None` | Line spacing ratio |
| `color` | `Optional[ColorRGBA]` | `None` | Text color (v7+) |

**Methods:**
- `from_sexpr(exp: list) -> Font` - Parse from `(font ...)`
- `to_sexpr(indent=0, newline=False) -> str`

---

### Justify

Text justification settings.

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `horizontally` | `Optional[str]` | `None` | `left`, `right`, or `center` |
| `vertically` | `Optional[str]` | `None` | `top` or `bottom` |
| `mirror` | `bool` | `False` | Mirror text |

**Methods:**
- `from_sexpr(exp: list) -> Justify` - Parse from `(justify ...)`
- `to_sexpr(indent=0, newline=False) -> str` - Returns empty string if no justification set

---

### Effects

Text display effects combining font and justification.

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `font` | `Font` | `Font()` | Font settings |
| `justify` | `Justify` | `Justify()` | Justification settings |
| `hide` | `bool` | `False` | Hide text |
| `href` | `Optional[str]` | `None` | Hyperlink URL (v7+) |

**Methods:**
- `from_sexpr(exp: list) -> Effects` - Parse from `(effects ...)`
- `to_sexpr(indent=0, newline=True) -> str`

**Example:**
```python
effects = Effects()
effects.font.height = 1.27
effects.font.width = 1.27
effects.font.bold = True
effects.justify.horizontally = "left"
effects.hide = False
```

---

### Fill

Shape fill settings.

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `type` | `str` | `"none"` | Fill type: `none`, `outline`, `background` |
| `color` | `Optional[ColorRGBA]` | `None` | Fill color (v7+) |

**Methods:**
- `from_sexpr(exp: list) -> Fill` - Parse from `(fill ...)`
- `to_sexpr(indent=4, newline=True) -> str`

---

### Net

Net reference by number and name.

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `number` | `int` | `0` | Net number (0 = no net) |
| `name` | `str` | `""` | Net name |

**Methods:**
- `from_sexpr(exp: list) -> Net` - Parse from `(net number "name")`
- `to_sexpr(indent=0, newline=False) -> str`

---

### Group

Named group of items.

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `name` | `str` | `""` | Group name |
| `locked` | `bool` | `False` | Prevent moving |
| `id` | `str` | `""` | Unique identifier |
| `members` | `List[str]` | `[]` | UUIDs of member items |

**Methods:**
- `from_sexpr(exp: list) -> Group` - Parse from `(group ...)`
- `to_sexpr(indent=2, newline=True) -> str`

---

### PageSettings

Drawing page size and orientation.

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `paperSize` | `str` | `"A4"` | Paper size: `A0`-`A5`, `A`-`E`, or `User` |
| `width` | `Optional[float]` | `None` | Custom width (when `paperSize="User"`) |
| `height` | `Optional[float]` | `None` | Custom height (when `paperSize="User"`) |
| `portrait` | `bool` | `False` | Portrait orientation |

**Methods:**
- `from_sexpr(exp: list) -> PageSettings` - Parse from `(paper ...)`
- `to_sexpr(indent=2, newline=True) -> str`

---

### TitleBlock

Document title block information.

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `title` | `Optional[str]` | `None` | Document title |
| `date` | `Optional[str]` | `None` | Date (YYYY-MM-DD format) |
| `revision` | `Optional[str]` | `None` | Revision string |
| `company` | `Optional[str]` | `None` | Company name |
| `comments` | `Dict[int, str]` | `{}` | Comments (keys 1-9) |

**Methods:**
- `from_sexpr(exp: list) -> TitleBlock` - Parse from `(title_block ...)`
- `to_sexpr(indent=2, newline=True) -> str`

---

### Property

Symbol/schematic property (key-value pair).

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `key` | `str` | `""` | Property name (must be unique) |
| `value` | `str` | `""` | Property value |
| `id` | `Optional[int]` | `None` | Unique ID (optional since v7) |
| `position` | `Position` | `Position(angle=0)` | Property position |
| `effects` | `Optional[Effects]` | `None` | Text display effects |
| `showName` | `bool` | `False` | Show property name (v7+) |

**Methods:**
- `from_sexpr(exp: list) -> Property` - Parse from `(property ...)`
- `to_sexpr(indent=4, newline=True) -> str`

---

### RenderCache

Cache for TrueType font rendering (v7+).

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `text` | `str` | `""` | Cached text string |
| `id` | `int` | `0` | Cache identifier |
| `polygons` | `List[RenderCachePolygon]` | `[]` | Polygon outlines |

**Methods:**
- `from_sexpr(exp: list) -> RenderCache` - Parse from `(render_cache ...)`
- `to_sexpr(indent=4, newline=True) -> str`

---

### RenderCachePolygon

Polygon used by RenderCache.

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `pts` | `List[Position]` | `[]` | Polygon vertices |

**Methods:**
- `from_sexpr(exp: list) -> RenderCachePolygon` - Parse from `(polygon ...)`
- `to_sexpr(indent=6, newline=True) -> str`

---

### Image

Embedded PNG image.

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `position` | `Position` | `Position()` | Image position |
| `scale` | `Optional[float]` | `None` | Scale factor |
| `data` | `List[str]` | `[]` | Base64-encoded PNG data |
| `uuid` | `Optional[str]` | `None` | Unique identifier |
| `layer` | `Optional[str]` | `None` | Layer name (PCB/footprint only) |

**Methods:**
- `from_sexpr(exp: list) -> Image` - Parse from `(image ...)`
- `to_sexpr(indent=2, newline=True) -> str`

---

### ProjectInstance (Abstract Base Class)

Base class for symbol and sheet project instances (v7+).

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `name` | `str` | `""` | Project name |

**Abstract Methods:**
- `from_sexpr(cls, exp: list) -> ProjectInstance`
- `to_sexpr(indent=2, newline=True) -> str`

## Common Patterns

### Creating styled text

```python
from kiutils.items.common import Effects, Font, Justify

effects = Effects()
effects.font = Font(height=1.5, width=1.5, thickness=0.15, bold=True)
effects.justify = Justify(horizontally="left", vertically="top")
```

### Setting stroke and fill together

```python
from kiutils.items.common import Stroke, Fill, ColorRGBA

stroke = Stroke(width=0.2, type="solid")
fill = Fill(type="outline")

# With custom colors (v7+)
stroke.color = ColorRGBA(R=0, G=100, B=200, A=1)
fill.color = ColorRGBA(R=200, G=200, B=255, A=1)
```

### Working with properties

```python
from kiutils.items.common import Property, Position, Effects

prop = Property(
    key="Reference",
    value="R1",
    position=Position(X=0, Y=-2.54, angle=0)
)
prop.effects = Effects()
prop.effects.font.height = 1.27
prop.effects.font.width = 1.27
```

## Cross-References

- Used by: All other `kiutils.items` modules
- See also: [brditems_apis.md](brditems_apis.md), [fpitems_apis.md](fpitems_apis.md), [schitems_apis.md](schitems_apis.md)
