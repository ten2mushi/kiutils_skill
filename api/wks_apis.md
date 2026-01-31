# Worksheet API Reference

Comprehensive API for `.kicad_wks` worksheet/title block template files.

## Quick Start

```python
from kiutils.wks import WorkSheet, Setup, TbText, Line, Rect
from kiutils.wks import WksPosition, WksFont, TextSize

# Load worksheet
ws = WorkSheet.from_file("my_template.kicad_wks")

# Create new worksheet
ws = WorkSheet.create_new()

# Add title block text
ws.drawingObjects.append(
    TbText(
        text="${TITLE}",
        name="Title",
        position=WksPosition(X=110, Y=27, corner='rbcorner')
    )
)

# Save worksheet
ws.to_file("my_template.kicad_wks")
```

---

## WorkSheet Class

Main container for worksheet definitions.

### Constructor / Factory Methods

```python
# Load from file
ws = WorkSheet.from_file(filepath: str, encoding: str = None) -> WorkSheet

# Create new empty worksheet
ws = WorkSheet.create_new() -> WorkSheet

# Parse from S-Expression
ws = WorkSheet.from_sexpr(exp: list) -> WorkSheet
```

### Attributes

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `version` | `str` | `KIUTILS_CREATE_NEW_VERSION_STR` | Version (YYYYMMDD format) |
| `generator` | `str` | `KIUTILS_CREATE_NEW_GENERATOR_STR` | Creation tool |
| `setup` | `Setup` | `Setup()` | Page setup configuration |
| `drawingObjects` | `List` | `[]` | TbText, Line, Rect, Polygon, Bitmap |
| `filePath` | `str \| None` | `None` | File path (auto-set by `from_file()`) |

### Methods

```python
# Save to file
ws.to_file(filepath: str = None)

# Generate S-Expression
ws.to_sexpr(indent: int = 0, newline: bool = True) -> str
```

---

## Setup Class

Page configuration and defaults.

```python
from kiutils.wks import Setup, TextSize

setup = Setup(
    textSize=TextSize(width=1.5, height=1.5),
    lineWidth=0.15,
    textLineWidth=0.15,
    leftMargin=10.0,
    rightMargin=10.0,
    topMargin=10.0,
    bottomMargin=10.0
)
```

### Attributes

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `textSize` | `TextSize` | `TextSize()` | Default text dimensions |
| `lineWidth` | `float` | `0.15` | Default line width |
| `textLineWidth` | `float` | `0.15` | Default text stroke width |
| `leftMargin` | `float` | `10.0` | Left margin |
| `rightMargin` | `float` | `10.0` | Right margin |
| `topMargin` | `float` | `10.0` | Top margin |
| `bottomMargin` | `float` | `10.0` | Bottom margin |

---

## TextSize Class

Default text dimensions.

```python
from kiutils.wks import TextSize

text_size = TextSize(
    width=1.5,
    height=1.5
)
```

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `width` | `float` | `1.5` | Text width |
| `height` | `float` | `1.5` | Text height |

---

## WksPosition Class

Position coordinates with optional corner reference.

```python
from kiutils.wks import WksPosition

# Position relative to right-bottom corner
pos = WksPosition(X=110, Y=27, corner='rbcorner')

# Position relative to left-top (default)
pos = WksPosition(X=10, Y=10)
```

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `X` | `float` | `0.0` | X coordinate |
| `Y` | `float` | `0.0` | Y coordinate |
| `corner` | `str \| None` | `None` | Reference corner |

### Corner Values

| Value | Description |
|-------|-------------|
| `ltcorner` | Left-top corner |
| `lbcorner` | Left-bottom corner |
| `rtcorner` | Right-top corner |
| `rbcorner` | Right-bottom corner |

---

## WksFont Class

Font styling for worksheet text.

```python
from kiutils.wks import WksFont, WksFontSize

font = WksFont(
    linewidth=0.15,
    size=WksFontSize(width=1.5, height=1.5),
    bold=False,
    italic=False
)
```

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `linewidth` | `float \| None` | `None` | Font stroke width |
| `size` | `WksFontSize \| None` | `None` | Font dimensions |
| `bold` | `bool` | `False` | Bold style |
| `italic` | `bool` | `False` | Italic style |

---

## WksFontSize Class

Font size dimensions.

```python
from kiutils.wks import WksFontSize

size = WksFontSize(width=1.5, height=1.5)
```

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `width` | `float` | `1.0` | Font width |
| `height` | `float` | `1.0` | Font height |

---

## TbText Class

Title block text element.

```python
from kiutils.wks import TbText, WksPosition, WksFont, WksFontSize
from kiutils.items.common import Justify

text = TbText(
    text="${TITLE}",
    name="Title",
    position=WksPosition(X=110, Y=27, corner='rbcorner'),
    option='notonpage1',  # or 'page1only' or None
    rotate=0,
    font=WksFont(size=WksFontSize(width=3, height=3), bold=True),
    justify=Justify(horizontal='left'),
    maxlen=50,
    maxheight=5,
    repeat=1,
    incrx=0,
    incry=5,
    incrlabel=1,
    comment="Title text"
)
```

### Attributes

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `text` | `str` | `""` | Text content (supports variables) |
| `name` | `str` | `""` | Element name |
| `position` | `WksPosition` | `WksPosition()` | Position coordinates |
| `option` | `str \| None` | `None` | Page visibility option |
| `rotate` | `float \| None` | `None` | Rotation angle |
| `font` | `WksFont` | `WksFont()` | Text styling |
| `justify` | `Justify \| None` | `None` | Text alignment |
| `maxlen` | `float \| None` | `None` | Maximum text length |
| `maxheight` | `float \| None` | `None` | Maximum text height |
| `repeat` | `int \| None` | `None` | Repeat count |
| `incrx` | `float \| None` | `None` | X increment per repeat |
| `incry` | `float \| None` | `None` | Y increment per repeat |
| `incrlabel` | `int \| None` | `None` | Label increment per repeat |
| `comment` | `str \| None` | `None` | Element comment |

### Option Values

| Value | Description |
|-------|-------------|
| `None` | Show on all pages |
| `notonpage1` | Show on all pages except page 1 |
| `page1only` | Show only on page 1 |

### Text Variables

| Variable | Description |
|----------|-------------|
| `${TITLE}` | Document title |
| `${DATE}` | Document date |
| `${REVISION}` | Revision number |
| `${COMPANY}` | Company name |
| `${COMMENT1}` - `${COMMENT9}` | Comment fields |
| `${FILENAME}` | File name |
| `${PAPER}` | Paper size |
| `${SHEETNAME}` | Sheet name |
| `${#}` | Current page number |
| `${##}` | Total page count |
| `${LAYER}` | Layer name (PCB) |
| `${ISSUE_DATE}` | Issue date |
| `${KICAD_VERSION}` | KiCad version |

---

## Line Class

Line element.

```python
from kiutils.wks import Line, WksPosition

line = Line(
    name="Border_Line",
    start=WksPosition(X=0, Y=0, corner='ltcorner'),
    end=WksPosition(X=0, Y=0, corner='rbcorner'),
    option=None,
    lineWidth=0.35,
    repeat=1,
    incrx=0,
    incry=0,
    comment="Outer border"
)
```

### Attributes

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `name` | `str` | `""` | Element name |
| `start` | `WksPosition` | `WksPosition()` | Start position |
| `end` | `WksPosition` | `WksPosition()` | End position |
| `option` | `str \| None` | `None` | Page visibility |
| `lineWidth` | `float \| None` | `None` | Line width |
| `repeat` | `int \| None` | `None` | Repeat count |
| `incrx` | `float \| None` | `None` | X increment |
| `incry` | `float \| None` | `None` | Y increment |
| `comment` | `str \| None` | `None` | Comment |

---

## Rect Class

Rectangle element.

```python
from kiutils.wks import Rect, WksPosition

rect = Rect(
    name="Title_Box",
    start=WksPosition(X=110, Y=34, corner='rbcorner'),
    end=WksPosition(X=0, Y=0, corner='rbcorner'),
    option=None,
    lineWidth=0.15,
    repeat=1,
    incrx=0,
    incry=0,
    comment="Title block rectangle"
)
```

### Attributes

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `name` | `str` | `""` | Element name |
| `start` | `WksPosition` | `WksPosition()` | Start corner |
| `end` | `WksPosition` | `WksPosition()` | End corner |
| `option` | `str \| None` | `None` | Page visibility |
| `lineWidth` | `float \| None` | `None` | Line width |
| `repeat` | `int \| None` | `None` | Repeat count |
| `incrx` | `float \| None` | `None` | X increment |
| `incry` | `float \| None` | `None` | Y increment |
| `comment` | `str \| None` | `None` | Comment |

---

## Polygon Class

Polygon element (note: not fully implemented in KiCad GUI).

```python
from kiutils.wks import Polygon, WksPosition

polygon = Polygon(
    name="Arrow",
    position=WksPosition(X=50, Y=50),
    option=None,
    rotate=0,
    coordinates=[
        WksPosition(X=0, Y=0),
        WksPosition(X=5, Y=2.5),
        WksPosition(X=0, Y=5)
    ],
    repeat=1,
    incrx=0,
    incry=0,
    comment="Arrow shape"
)
```

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `name` | `str` | `""` | Element name |
| `position` | `WksPosition` | `WksPosition()` | Base position |
| `option` | `str \| None` | `None` | Page visibility |
| `rotate` | `float \| None` | `None` | Rotation angle |
| `coordinates` | `List[WksPosition]` | `[]` | Vertex positions |
| `repeat` | `int \| None` | `None` | Repeat count |
| `incrx` | `float \| None` | `None` | X increment |
| `incry` | `float \| None` | `None` | Y increment |
| `comment` | `str \| None` | `None` | Comment |

---

## Bitmap Class

Embedded image element.

```python
from kiutils.wks import Bitmap, WksPosition

bitmap = Bitmap(
    name="Logo",
    position=WksPosition(X=10, Y=10, corner='rbcorner'),
    option=None,
    scale=1.0,
    repeat=1,
    incrx=0,
    incry=0,
    comment="Company logo",
    pngdata=[
        "89 50 4e 47 0d 0a 1a 0a ...",  # PNG data as hex strings
    ]
)
```

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `name` | `str` | `""` | Element name |
| `position` | `WksPosition` | `WksPosition()` | Position |
| `option` | `str \| None` | `None` | Page visibility |
| `scale` | `float` | `1.0` | Image scale factor |
| `repeat` | `int \| None` | `None` | Repeat count |
| `incrx` | `float \| None` | `None` | X increment |
| `incry` | `float \| None` | `None` | Y increment |
| `comment` | `str \| None` | `None` | Comment |
| `pngdata` | `List[str]` | `[]` | PNG data as hex strings (32-byte aligned) |

---

## Common Patterns

### Create Basic Title Block

```python
from kiutils.wks import WorkSheet, Setup, TbText, Line, Rect, WksPosition, WksFont, WksFontSize

ws = WorkSheet.create_new()

# Title block rectangle
ws.drawingObjects.append(
    Rect(name="TitleBox",
         start=WksPosition(X=110, Y=34, corner='rbcorner'),
         end=WksPosition(X=0, Y=0, corner='rbcorner'),
         lineWidth=0.35)
)

# Title text
ws.drawingObjects.append(
    TbText(text="${TITLE}",
           name="Title",
           position=WksPosition(X=55, Y=27, corner='rbcorner'),
           font=WksFont(size=WksFontSize(width=3, height=3), bold=True))
)

# Date
ws.drawingObjects.append(
    TbText(text="${DATE}",
           name="Date",
           position=WksPosition(X=108, Y=5, corner='rbcorner'))
)

# Revision
ws.drawingObjects.append(
    TbText(text="Rev: ${REVISION}",
           name="Revision",
           position=WksPosition(X=20, Y=5, corner='rbcorner'))
)

# Sheet number
ws.drawingObjects.append(
    TbText(text="Sheet ${#}/${##}",
           name="SheetNum",
           position=WksPosition(X=55, Y=5, corner='rbcorner'))
)

ws.to_file("my_titleblock.kicad_wks")
```

### Add Border with Margin Lines

```python
ws = WorkSheet.create_new()

# Outer border (thick)
ws.drawingObjects.extend([
    Line(name="Border_Left",
         start=WksPosition(X=0, Y=0, corner='ltcorner'),
         end=WksPosition(X=0, Y=0, corner='lbcorner'),
         lineWidth=0.35),
    Line(name="Border_Top",
         start=WksPosition(X=0, Y=0, corner='ltcorner'),
         end=WksPosition(X=0, Y=0, corner='rtcorner'),
         lineWidth=0.35),
    Line(name="Border_Right",
         start=WksPosition(X=0, Y=0, corner='rtcorner'),
         end=WksPosition(X=0, Y=0, corner='rbcorner'),
         lineWidth=0.35),
    Line(name="Border_Bottom",
         start=WksPosition(X=0, Y=0, corner='lbcorner'),
         end=WksPosition(X=0, Y=0, corner='rbcorner'),
         lineWidth=0.35),
])

ws.to_file("bordered.kicad_wks")
```

### Create Zone Markers

```python
from kiutils.wks import TbText, WksPosition

# Zone letters along left edge
for i, letter in enumerate('ABCDEFGH'):
    ws.drawingObjects.append(
        TbText(text=letter,
               name=f"Zone_{letter}",
               position=WksPosition(X=5, Y=20 + i*30, corner='ltcorner'),
               font=WksFont(size=WksFontSize(width=2, height=2)))
    )

# Zone numbers along top edge
for i in range(1, 9):
    ws.drawingObjects.append(
        TbText(text=str(i),
               name=f"Zone_{i}",
               position=WksPosition(X=20 + i*30, Y=5, corner='ltcorner'),
               font=WksFont(size=WksFontSize(width=2, height=2)))
    )
```

### Add Company Logo

```python
import base64

# Read PNG file
with open("logo.png", "rb") as f:
    png_bytes = f.read()

# Convert to hex strings (32 bytes per line)
hex_data = png_bytes.hex()
pngdata = []
for i in range(0, len(hex_data), 64):  # 64 hex chars = 32 bytes
    chunk = hex_data[i:i+64]
    formatted = ' '.join(chunk[j:j+2] for j in range(0, len(chunk), 2)) + ' '
    pngdata.append(formatted)

ws.drawingObjects.append(
    Bitmap(name="CompanyLogo",
           position=WksPosition(X=15, Y=15, corner='rbcorner'),
           scale=0.5,
           pngdata=pngdata)
)
```

### Multi-Page Headers

```python
# Header only on first page
ws.drawingObjects.append(
    TbText(text="FIRST PAGE ONLY",
           name="FirstPageHeader",
           position=WksPosition(X=105, Y=5, corner='rtcorner'),
           option='page1only',
           font=WksFont(bold=True))
)

# Header on pages 2+
ws.drawingObjects.append(
    TbText(text="CONTINUATION - ${TITLE}",
           name="ContHeader",
           position=WksPosition(X=105, Y=5, corner='rtcorner'),
           option='notonpage1',
           font=WksFont(bold=True))
)
```

### Repeated Row Numbers

```python
# Row numbers down left margin
ws.drawingObjects.append(
    TbText(text="1",
           name="RowNum",
           position=WksPosition(X=5, Y=15, corner='ltcorner'),
           repeat=10,
           incrx=0,
           incry=25,
           incrlabel=1)  # Increment label by 1 each repeat
)
```

### Load and Modify Existing Template

```python
ws = WorkSheet.from_file("existing.kicad_wks")

# Find and modify title text
for obj in ws.drawingObjects:
    if isinstance(obj, TbText) and obj.name == "Title":
        obj.font.bold = True
        obj.font.size = WksFontSize(width=4, height=4)

# Add new element
ws.drawingObjects.append(
    TbText(text="CONFIDENTIAL",
           name="Confidential",
           position=WksPosition(X=200, Y=10, corner='ltcorner'),
           font=WksFont(size=WksFontSize(width=5, height=5), bold=True))
)

ws.to_file()
```

### Adjust Page Margins

```python
ws = WorkSheet.create_new()

# Set margins
ws.setup.leftMargin = 15.0
ws.setup.rightMargin = 10.0
ws.setup.topMargin = 10.0
ws.setup.bottomMargin = 10.0

# Set default text size
ws.setup.textSize = TextSize(width=1.27, height=1.27)
ws.setup.lineWidth = 0.15
ws.setup.textLineWidth = 0.1

ws.to_file("custom_margins.kicad_wks")
```

### Export Worksheet Info

```python
ws = WorkSheet.from_file("template.kicad_wks")

print(f"Version: {ws.version}")
print(f"Generator: {ws.generator}")
print(f"\nSetup:")
print(f"  Margins: L={ws.setup.leftMargin} R={ws.setup.rightMargin} T={ws.setup.topMargin} B={ws.setup.bottomMargin}")
print(f"  Text Size: {ws.setup.textSize.width} x {ws.setup.textSize.height}")

print(f"\nDrawing Objects ({len(ws.drawingObjects)}):")
for obj in ws.drawingObjects:
    obj_type = type(obj).__name__
    name = getattr(obj, 'name', 'unnamed')
    print(f"  {obj_type}: {name}")
```
