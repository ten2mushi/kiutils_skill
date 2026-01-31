# kiutils.items.schitems API Reference

Schematic items: junctions, connections, labels, text, and hierarchical elements for KiCad schematic files.

## Quick Start

```python
from kiutils.items.schitems import Junction, Connection, LocalLabel, GlobalLabel, Text
from kiutils.items.common import Position, Stroke

# Wire connection
wire = Connection(type="wire")
wire.points = [Position(X=50.8, Y=50.8), Position(X=76.2, Y=50.8)]
wire.stroke = Stroke(width=0, type="default")

# Wire junction
junction = Junction(position=Position(X=76.2, Y=50.8), diameter=0)

# Local label (net name on wire)
label = LocalLabel(text="VCC", position=Position(X=50.8, Y=50.8, angle=0))

# Global label (visible across sheets)
global_label = GlobalLabel(
    text="CLK",
    shape="input",
    position=Position(X=100.0, Y=50.0, angle=0)
)
```

## Class Reference

### Junction

Wire junction point (connection node).

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `position` | `Position` | `Position()` | X, Y coordinates |
| `diameter` | `float` | `0` | Junction diameter (0 = default) |
| `color` | `ColorRGBA` | `ColorRGBA()` | Junction color (all 0 = default) |
| `uuid` | `Optional[str]` | `None` | Unique identifier |

**Methods:**
- `from_sexpr(exp: list) -> Junction` - Parse from `(junction ...)`
- `to_sexpr(indent=2, newline=True) -> str`

---

### NoConnect

Unused pin marker (X symbol).

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `position` | `Position` | `Position()` | X, Y coordinates |
| `uuid` | `Optional[str]` | `None` | Unique identifier |

**Methods:**
- `from_sexpr(exp: list) -> NoConnect` - Parse from `(no_connect ...)`
- `to_sexpr(indent=2, newline=True) -> str`

---

### Connection

Wire or bus connection.

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `type` | `str` | `"wire"` | Connection type: `wire` or `bus` |
| `points` | `List[Position]` | `[]` | Start and end coordinates |
| `stroke` | `Stroke` | `Stroke()` | Line style |
| `uuid` | `Optional[str]` | `None` | Unique identifier |

**Methods:**
- `from_sexpr(exp: list) -> Connection` - Parse from `(wire ...)` or `(bus ...)`
- `to_sexpr(indent=2, newline=True) -> str`

**Example:**
```python
from kiutils.items.schitems import Connection
from kiutils.items.common import Position, Stroke

# Horizontal wire
wire = Connection(type="wire")
wire.points = [Position(X=25.4, Y=50.8), Position(X=50.8, Y=50.8)]
wire.stroke = Stroke(width=0, type="default")
```

---

### BusEntry

Bus entry point.

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `position` | `Position` | `Position()` | Entry position |
| `size` | `Position` | `Position()` | X/Y distance to endpoint |
| `stroke` | `Stroke` | `Stroke()` | Line style |
| `uuid` | `Optional[str]` | `None` | Unique identifier |

**Methods:**
- `from_sexpr(exp: list) -> BusEntry` - Parse from `(bus_entry ...)`
- `to_sexpr(indent=2, newline=True) -> str`

---

### BusAlias

Named bus alias defining member signals.

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `name` | `str` | `""` | Bus alias name |
| `members` | `List[str]` | `[]` | Member signal names |

**Methods:**
- `from_sexpr(exp: list) -> BusAlias` - Parse from `(bus_alias ...)`
- `to_sexpr(indent=2, newline=True) -> str`

---

### PolyLine

Graphical polyline (decorative, not electrical).

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `points` | `List[Position]` | `[]` | Vertex coordinates (min 2) |
| `stroke` | `Stroke` | `Stroke()` | Line style |
| `uuid` | `Optional[str]` | `None` | Unique identifier |

**Methods:**
- `from_sexpr(exp: list) -> PolyLine` - Parse from `(polyline ...)`
- `to_sexpr(indent=2, newline=True) -> str`

---

### Text

Graphical text annotation.

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `text` | `str` | `""` | Text content |
| `position` | `Position` | `Position()` | Position and angle |
| `effects` | `Effects` | `Effects()` | Text display style |
| `uuid` | `Optional[str]` | `None` | Unique identifier |

**Methods:**
- `from_sexpr(exp: list) -> Text` - Parse from `(text ...)`
- `to_sexpr(indent=2, newline=True) -> str`

---

### TextBox

Text box with border (v7+).

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `text` | `str` | `""` | Text content |
| `position` | `Position` | `Position()` | Position and angle |
| `size` | `Position` | `Position()` | Width and height |
| `stroke` | `Stroke` | `Stroke()` | Border style |
| `fill` | `Fill` | `Fill()` | Fill settings |
| `effects` | `Effects` | `Effects()` | Text style |
| `uuid` | `Optional[str]` | `None` | Unique identifier |

**Methods:**
- `from_sexpr(exp: list) -> TextBox` - Parse from `(text_box ...)`
- `to_sexpr(indent=2, newline=True) -> str`

---

### LocalLabel

Wire/bus label (local to sheet).

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `text` | `str` | `""` | Label text (net name) |
| `position` | `Position` | `Position()` | Position and angle |
| `effects` | `Effects` | `Effects()` | Text style |
| `uuid` | `Optional[str]` | `None` | Unique identifier |
| `fieldsAutoplaced` | `bool` | `False` | Properties auto-placed |

**Methods:**
- `from_sexpr(exp: list) -> LocalLabel` - Parse from `(label ...)`
- `to_sexpr(indent=2, newline=True) -> str`

---

### GlobalLabel

Global label (visible across all sheets).

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `text` | `str` | `""` | Label text |
| `shape` | `str` | `"input"` | Shape: `input`, `output`, `bidirectional`, `tri_state`, `passive` |
| `fieldsAutoplaced` | `bool` | `False` | Properties auto-placed |
| `position` | `Position` | `Position()` | Position and angle |
| `effects` | `Effects` | `Effects()` | Text style |
| `uuid` | `Optional[str]` | `None` | Unique identifier |
| `properties` | `List[Property]` | `[]` | Inter-sheet reference properties |

**Methods:**
- `from_sexpr(exp: list) -> GlobalLabel` - Parse from `(global_label ...)`
- `to_sexpr(indent=2, newline=True) -> str`

---

### HierarchicalLabel

Label connecting to hierarchical sheets.

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `text` | `str` | `""` | Label text |
| `shape` | `str` | `"input"` | Shape: `input`, `output`, `bidirectional`, `tri_state`, `passive` |
| `position` | `Position` | `Position()` | Position and angle |
| `effects` | `Effects` | `Effects()` | Text style |
| `uuid` | `Optional[str]` | `None` | Unique identifier |
| `fieldsAutoplaced` | `bool` | `False` | Properties auto-placed |

**Methods:**
- `from_sexpr(exp: list) -> HierarchicalLabel` - Parse from `(hierarchical_label ...)`
- `to_sexpr(indent=2, newline=True) -> str`

---

### SchematicSymbol

Symbol instance placed in schematic.

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `libName` | `Optional[str]` | `None` | Library name |
| `libId` | `str` | `""` | Library identifier |
| `position` | `Position` | `Position()` | Position and rotation |
| `unit` | `int` | `1` | Symbol unit number |
| `inBom` | `bool` | `True` | Include in BOM |
| `onBoard` | `bool` | `True` | Include on board |
| `dnp` | `bool` | `False` | Do Not Populate (v7+) |
| `fieldsAutoplaced` | `bool` | `False` | Fields auto-placed |
| `uuid` | `Optional[str]` | `None` | Unique identifier |
| `properties` | `List[Property]` | `[]` | Symbol properties |
| `pins` | `Dict[str, str]` | `{}` | Pin UUID mappings |
| `mirror` | `Optional[str]` | `None` | Mirror: `x` or `y` |
| `instances` | `List[SymbolProjectInstance]` | `[]` | Project instances (v7+) |

**Methods:**
- `from_sexpr(exp: list) -> SchematicSymbol` - Parse from `(symbol ...)`
- `to_sexpr(indent=2, newline=True) -> str`

## Common Patterns

### Creating a simple wire connection

```python
from kiutils.items.schitems import Connection, Junction
from kiutils.items.common import Position, Stroke

# Wire from point A to point B
wire = Connection(type="wire")
wire.points = [Position(X=25.4, Y=50.8), Position(X=76.2, Y=50.8)]
wire.stroke = Stroke(width=0, type="default")

# Add junction at connection point
junction = Junction(position=Position(X=76.2, Y=50.8))
```

### Adding labels to wires

```python
from kiutils.items.schitems import LocalLabel, GlobalLabel
from kiutils.items.common import Position, Effects

# Local net label
net_label = LocalLabel(text="VCC", position=Position(X=50.8, Y=50.8, angle=0))

# Global label for multi-sheet designs
global_net = GlobalLabel(
    text="RESET_N",
    shape="input",
    position=Position(X=25.4, Y=25.4, angle=180)
)
```

### Working with hierarchical designs

```python
from kiutils.items.schitems import HierarchicalLabel
from kiutils.items.common import Position

# Create hierarchical connection
hlabel = HierarchicalLabel(
    text="DATA_BUS",
    shape="bidirectional",
    position=Position(X=152.4, Y=76.2, angle=0)
)
```

### Creating bus connections

```python
from kiutils.items.schitems import Connection, BusEntry, BusAlias
from kiutils.items.common import Position, Stroke

# Define bus alias
bus_alias = BusAlias(name="DATA", members=["D0", "D1", "D2", "D3", "D4", "D5", "D6", "D7"])

# Create bus
bus = Connection(type="bus")
bus.points = [Position(X=25.4, Y=25.4), Position(X=25.4, Y=100.0)]
bus.stroke = Stroke(width=0, type="default")

# Bus entry to tap off signal
entry = BusEntry(position=Position(X=25.4, Y=50.8))
entry.size = Position(X=2.54, Y=2.54)
```

## Cross-References

- Uses: [common_apis.md](common_apis.md) (Position, Effects, Stroke, Fill, Property, ColorRGBA)
- See also: [syitems_apis.md](syitems_apis.md) for symbol body graphics
