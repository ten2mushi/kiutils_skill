# Symbol API Reference

Comprehensive API for `.kicad_sym` symbol library files.

## Quick Start

```python
from kiutils.symbol import SymbolLib, Symbol, SymbolPin
from kiutils.items.syitems import SyRect, SyText
from kiutils.items.common import Position, Effects, Stroke, Fill

# Load symbol library
lib = SymbolLib.from_file("MyLibrary.kicad_sym")

# Access symbols
for symbol in lib.symbols:
    print(symbol.entryName)

# Create new symbol
symbol = Symbol.create_new(
    id="MySymbol",
    reference="U",
    value="MyChip",
    footprint="Package_SO:SOIC-8",
    datasheet="https://example.com/datasheet.pdf"
)

# Save library
lib.to_file("MyLibrary.kicad_sym")
```

---

## SymbolLib Class

Container for symbol library files.

### Constructor / Factory Methods

```python
# Load from file
lib = SymbolLib.from_file(filepath: str, encoding: str = None) -> SymbolLib

# Parse from S-Expression
lib = SymbolLib.from_sexpr(exp: list) -> SymbolLib
```

### Attributes

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `version` | `str` | `KIUTILS_CREATE_NEW_VERSION_STR` | Version (YYYYMMDD format) |
| `generator` | `str \| None` | `None` | Creation tool identifier |
| `symbols` | `List[Symbol]` | `[]` | Symbol definitions |
| `filePath` | `str \| None` | `None` | File path (auto-set by `from_file()`) |

### Methods

```python
# Save to file
lib.to_file(filepath: str = None, encoding: str = None)

# Generate S-Expression
lib.to_sexpr(indent: int = 0, newline: bool = True) -> str
```

---

## Symbol Class

Individual symbol definition.

### Constructor / Factory Methods

```python
# Create new symbol
symbol = Symbol.create_new(
    id: str,            # Symbol ID
    reference: str,     # Reference designator (U, R, C, etc.)
    value: str,         # Value text
    footprint: str = "",    # Default footprint
    datasheet: str = ""     # Datasheet URL
) -> Symbol

# Parse from S-Expression
symbol = Symbol.from_sexpr(exp: list) -> Symbol
```

### Library ID Property

```python
# The libId property combines libraryNickname:entryName or entryName_unitId_styleId
symbol.libId  # Get: "MyLib:MySymbol" or "MySymbol_1_1"
symbol.libId = "NewLib:NewName"  # Set: updates libraryNickname and entryName
```

### Core Attributes

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `libId` | `str` (property) | - | Library identifier |
| `libraryNickname` | `str \| None` | `None` | Library name prefix |
| `entryName` | `str` | `None` | Symbol name |
| `unitId` | `int \| None` | `None` | Unit identifier (child symbols) |
| `styleId` | `int \| None` | `None` | Body style identifier (child symbols) |
| `extends` | `str \| None` | `None` | Parent symbol to extend |

### Display Settings

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `hidePinNumbers` | `bool` | `False` | Hide all pin numbers |
| `pinNames` | `bool` | `False` | Pin names token present |
| `pinNamesHide` | `bool` | `False` | Hide all pin names |
| `pinNamesOffset` | `float \| None` | `None` | Pin name offset (default 0.508mm) |

### Symbol Flags

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `inBom` | `bool \| None` | `None` | Include in BOM |
| `onBoard` | `bool \| None` | `None` | Export to PCB |
| `isPower` | `bool` | `False` | Power symbol flag |

### Collections

| Attribute | Type | Description |
|-----------|------|-------------|
| `properties` | `List[Property]` | Symbol properties (Reference, Value, Footprint, Datasheet, etc.) |
| `graphicItems` | `List` | SyArc, SyCircle, SyCurve, SyPolyLine, SyRect, SyText, SyTextBox |
| `pins` | `List[SymbolPin]` | Pin definitions |
| `units` | `List[Symbol]` | Child unit symbols |

### Methods

```python
# Generate S-Expression
symbol.to_sexpr(indent: int = 2, newline: bool = True) -> str
```

---

## SymbolPin Class

Pin definition within a symbol.

```python
from kiutils.symbol import SymbolPin
from kiutils.items.common import Position, Effects

pin = SymbolPin(
    electricalType='input',
    graphicalStyle='line',
    position=Position(X=-5.08, Y=2.54, angle=0),
    length=2.54,
    name='IN',
    number='1',
    nameEffects=Effects(),
    numberEffects=Effects(),
    hide=False
)
```

### Attributes

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `electricalType` | `str` | `"input"` | Electrical connection type |
| `graphicalStyle` | `str` | `"line"` | Graphical style |
| `position` | `Position` | `Position()` | X, Y, angle (0, 90, 180, 270) |
| `length` | `float` | `0.254` | Pin length |
| `name` | `str` | `""` | Pin name |
| `nameEffects` | `Effects \| None` | `None` | Name text styling |
| `number` | `str` | `"0"` | Pin number |
| `numberEffects` | `Effects \| None` | `None` | Number text styling |
| `hide` | `bool` | `False` | Hidden pin |
| `alternatePins` | `List[SymbolAlternativePin]` | `[]` | Alternative pin functions |

### Electrical Types

| Type | Description |
|------|-------------|
| `input` | Input pin |
| `output` | Output pin |
| `bidirectional` | Bidirectional pin |
| `tri_state` | Tri-state output |
| `passive` | Passive component |
| `free` | Not internally connected |
| `unspecified` | Unspecified type |
| `power_in` | Power input |
| `power_out` | Power output |
| `open_collector` | Open collector |
| `open_emitter` | Open emitter |
| `no_connect` | Not connected internally |

### Graphical Styles

| Style | Description |
|-------|-------------|
| `line` | Standard line |
| `inverted` | Inverted (bubble) |
| `clock` | Clock input |
| `inverted_clock` | Inverted clock |
| `input_low` | Active low input |
| `clock_low` | Active low clock |
| `output_low` | Active low output |
| `edge_clock_high` | Edge triggered clock |
| `non_logic` | Non-logic pin |

---

## SymbolAlternativePin Class

Alternative pin function definition.

```python
from kiutils.symbol import SymbolAlternativePin

alt_pin = SymbolAlternativePin(
    pinName='ALT_FUNC',
    electricalType='output',
    graphicalStyle='line'
)

# Add to a pin
pin.alternatePins.append(alt_pin)
```

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `pinName` | `str` | `""` | Alternative function name |
| `electricalType` | `str` | `"input"` | Electrical type |
| `graphicalStyle` | `str` | `"line"` | Graphical style |

---

## Symbol Graphic Items

### SyRect

Rectangle in symbol.

```python
from kiutils.items.syitems import SyRect
from kiutils.items.common import Position, Stroke, Fill

rect = SyRect(
    start=Position(X=-5.08, Y=5.08),
    end=Position(X=5.08, Y=-5.08),
    stroke=Stroke(width=0.254, type='default'),
    fill=Fill(type='background'),
    private=False  # v7+: visible only in editor
)
```

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `private` | `bool` | `False` | Editor-only visibility (v7+) |
| `start` | `Position` | `Position()` | Start corner |
| `end` | `Position` | `Position()` | End corner |
| `stroke` | `Stroke` | `Stroke()` | Outline style |
| `fill` | `Fill` | `Fill()` | Fill style |

### SyCircle

Circle in symbol.

```python
from kiutils.items.syitems import SyCircle
from kiutils.items.common import Position, Stroke, Fill

circle = SyCircle(
    center=Position(X=0, Y=0),
    radius=2.54,
    stroke=Stroke(width=0.254),
    fill=Fill(type='none'),
    private=False
)
```

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `private` | `bool` | `False` | Editor-only visibility (v7+) |
| `center` | `Position` | `Position()` | Center point |
| `radius` | `float` | `0.0` | Circle radius |
| `stroke` | `Stroke` | `Stroke()` | Outline style |
| `fill` | `Fill` | `Fill()` | Fill style |

### SyArc

Arc in symbol.

```python
from kiutils.items.syitems import SyArc
from kiutils.items.common import Position, Stroke, Fill

arc = SyArc(
    start=Position(X=-2.54, Y=0),
    mid=Position(X=0, Y=2.54),
    end=Position(X=2.54, Y=0),
    stroke=Stroke(width=0.254),
    fill=Fill(type='none'),
    private=False
)
```

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `private` | `bool` | `False` | Editor-only visibility (v7+) |
| `start` | `Position` | `Position()` | Start point |
| `mid` | `Position` | `Position()` | Midpoint |
| `end` | `Position` | `Position()` | End point |
| `stroke` | `Stroke` | `Stroke()` | Outline style |
| `fill` | `Fill` | `Fill()` | Fill style |

### SyPolyLine

Polyline/polygon in symbol.

```python
from kiutils.items.syitems import SyPolyLine
from kiutils.items.common import Position, Stroke, Fill

polyline = SyPolyLine(
    points=[
        Position(X=0, Y=0),
        Position(X=2.54, Y=2.54),
        Position(X=5.08, Y=0)
    ],
    stroke=Stroke(width=0.254),
    fill=Fill(type='none')
)
```

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `points` | `List[Position]` | `[]` | Vertex coordinates |
| `stroke` | `Stroke` | `Stroke()` | Outline style |
| `fill` | `Fill` | `Fill()` | Fill style |

### SyCurve

Cubic Bezier curve.

```python
from kiutils.items.syitems import SyCurve
from kiutils.items.common import Position, Stroke, Fill

curve = SyCurve(
    points=[
        Position(X=0, Y=0),     # Start
        Position(X=1, Y=2),     # Control 1
        Position(X=3, Y=2),     # Control 2
        Position(X=4, Y=0)      # End
    ],
    stroke=Stroke(width=0.254),
    fill=Fill(type='none')
)
```

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `points` | `List[Position]` | `[]` | 4 control points |
| `stroke` | `Stroke` | `Stroke()` | Outline style |
| `fill` | `Fill` | `Fill()` | Fill style |

### SyText

Text in symbol.

```python
from kiutils.items.syitems import SyText
from kiutils.items.common import Position, Effects

text = SyText(
    text="Label",
    position=Position(X=0, Y=0, angle=0),
    effects=Effects()
)
```

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `text` | `str` | `""` | Text content |
| `position` | `Position` | `Position()` | Location and angle |
| `effects` | `Effects` | `Effects()` | Text styling |

### SyTextBox (v7+)

Text box in symbol.

```python
from kiutils.items.syitems import SyTextBox
from kiutils.items.common import Position, Stroke, Fill, Effects

textbox = SyTextBox(
    text="Description",
    position=Position(X=-5, Y=5),
    size=Position(X=10, Y=4),
    stroke=Stroke(),
    fill=Fill(type='background'),
    effects=Effects(),
    private=False
)
```

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `text` | `str` | `""` | Text content |
| `private` | `bool` | `False` | Editor-only visibility |
| `position` | `Position` | `Position()` | Location and angle |
| `size` | `Position` | `Position()` | Width and height |
| `stroke` | `Stroke` | `Stroke()` | Border style |
| `fill` | `Fill` | `Fill()` | Background fill |
| `effects` | `Effects` | `Effects()` | Text styling |
| `uuid` | `str \| None` | `None` | Unique identifier |

---

## Fill Types

| Type | Description |
|------|-------------|
| `none` | No fill |
| `outline` | Outline only |
| `background` | Filled with background color |

---

## Common Patterns

### Create Simple IC Symbol

```python
from kiutils.symbol import SymbolLib, Symbol, SymbolPin
from kiutils.items.syitems import SyRect
from kiutils.items.common import Position, Effects, Stroke, Fill, Font, Property

lib = SymbolLib(generator='kiutils')

# Create main symbol
symbol = Symbol.create_new("MyIC", "U", "74HC04", "Package_DIP:DIP-14", "")

# Add body rectangle
symbol.graphicItems.append(
    SyRect(
        start=Position(X=-5.08, Y=5.08),
        end=Position(X=5.08, Y=-5.08),
        stroke=Stroke(width=0.254, type='default'),
        fill=Fill(type='background')
    )
)

# Add pins
symbol.pins.extend([
    SymbolPin(electricalType='input', position=Position(X=-7.62, Y=2.54, angle=0),
              length=2.54, name='IN', number='1'),
    SymbolPin(electricalType='output', position=Position(X=7.62, Y=2.54, angle=180),
              length=2.54, name='OUT', number='2'),
    SymbolPin(electricalType='power_in', position=Position(X=0, Y=7.62, angle=270),
              length=2.54, name='VCC', number='14'),
    SymbolPin(electricalType='power_in', position=Position(X=0, Y=-7.62, angle=90),
              length=2.54, name='GND', number='7'),
])

lib.symbols.append(symbol)
lib.to_file("MyLibrary.kicad_sym")
```

### Create Multi-Unit Symbol

```python
from kiutils.symbol import SymbolLib, Symbol, SymbolPin
from kiutils.items.syitems import SyRect
from kiutils.items.common import Position, Stroke, Fill

lib = SymbolLib(generator='kiutils')

# Parent symbol with common settings
parent = Symbol.create_new("74HC04", "U", "74HC04")
parent.inBom = True
parent.onBoard = True

# Unit 1 (gates A through F would be similar)
unit1 = Symbol()
unit1.libId = "74HC04_1_1"  # Sets entryName, unitId, styleId

# Add inverter triangle
unit1.graphicItems.append(
    SyPolyLine(
        points=[
            Position(X=-2.54, Y=2.54),
            Position(X=-2.54, Y=-2.54),
            Position(X=2.54, Y=0)
        ],
        stroke=Stroke(width=0.254),
        fill=Fill(type='background')
    )
)

# Add pins to unit
unit1.pins.extend([
    SymbolPin(electricalType='input', position=Position(X=-5.08, Y=0, angle=0),
              length=2.54, name='A', number='1'),
    SymbolPin(electricalType='output', graphicalStyle='inverted',
              position=Position(X=5.08, Y=0, angle=180),
              length=2.54, name='Y', number='2'),
])

parent.units.append(unit1)
# Add more units...

lib.symbols.append(parent)
lib.to_file("74HC04.kicad_sym")
```

### Create Power Symbol

```python
symbol = Symbol()
symbol.libId = "GND"
symbol.isPower = True
symbol.inBom = False
symbol.onBoard = True
symbol.hidePinNumbers = True
symbol.pinNamesHide = True
symbol.pinNames = True

# Add GND graphic
symbol.graphicItems.append(
    SyPolyLine(
        points=[
            Position(X=0, Y=0),
            Position(X=0, Y=-1.27),
            Position(X=-1.27, Y=-1.27),
            Position(X=1.27, Y=-1.27),
        ],
        stroke=Stroke(width=0.254),
        fill=Fill(type='none')
    )
)

# Add power pin
symbol.pins.append(
    SymbolPin(
        electricalType='power_in',
        graphicalStyle='line',
        position=Position(X=0, Y=0, angle=90),
        length=0,
        name='GND',
        number='1',
        hide=True
    )
)

symbol.properties.extend([
    Property(key="Reference", value="#PWR", id=0),
    Property(key="Value", value="GND", id=1),
])
```

### Find Symbols by Property

```python
lib = SymbolLib.from_file("library.kicad_sym")

# Find by value
matches = [s for s in lib.symbols
           if any(p.key == "Value" and "555" in p.value for p in s.properties)]

# Find power symbols
power_symbols = [s for s in lib.symbols if s.isPower]

# Find symbols with specific footprint
dip_symbols = [s for s in lib.symbols
               if any(p.key == "Footprint" and "DIP" in p.value for p in s.properties)]
```

### Modify Pin Properties

```python
lib = SymbolLib.from_file("library.kicad_sym")

for symbol in lib.symbols:
    for unit in symbol.units:
        for pin in unit.pins:
            if pin.name == "VCC" or pin.name == "VDD":
                pin.electricalType = "power_in"
            elif pin.name == "GND" or pin.name == "VSS":
                pin.electricalType = "power_in"

lib.to_file()
```

### Generate BOM from Library

```python
lib = SymbolLib.from_file("library.kicad_sym")

bom = []
for symbol in lib.symbols:
    if symbol.inBom:
        entry = {"name": symbol.entryName}
        for prop in symbol.properties:
            entry[prop.key] = prop.value
        bom.append(entry)

# Print BOM
for item in bom:
    print(f"{item.get('Reference', 'N/A')}: {item.get('Value', 'N/A')} - {item.get('Footprint', 'N/A')}")
```

### Add Properties to All Symbols

```python
from kiutils.items.common import Property, Effects, Font

lib = SymbolLib.from_file("library.kicad_sym")

for symbol in lib.symbols:
    # Check if property exists
    has_manuf = any(p.key == "Manufacturer" for p in symbol.properties)

    if not has_manuf:
        symbol.properties.append(
            Property(
                key="Manufacturer",
                value="",
                id=len(symbol.properties),
                effects=Effects(font=Font(width=1.27, height=1.27), hide=True)
            )
        )

lib.to_file()
```

### Clone and Modify Symbol

```python
import copy

lib = SymbolLib.from_file("library.kicad_sym")

# Find source symbol
source = next(s for s in lib.symbols if s.entryName == "OriginalIC")

# Deep copy
new_symbol = copy.deepcopy(source)
new_symbol.libId = "ModifiedIC"

# Modify value property
for prop in new_symbol.properties:
    if prop.key == "Value":
        prop.value = "ModifiedIC"

lib.symbols.append(new_symbol)
lib.to_file()
```

### Extract Pin List

```python
lib = SymbolLib.from_file("library.kicad_sym")

for symbol in lib.symbols:
    print(f"\n{symbol.entryName}:")
    print("-" * 40)

    # Collect all pins from units
    all_pins = symbol.pins.copy()
    for unit in symbol.units:
        all_pins.extend(unit.pins)

    for pin in sorted(all_pins, key=lambda p: p.number):
        print(f"  {pin.number:>4}: {pin.name:<20} ({pin.electricalType})")
```
