# Design Rules API Reference

Comprehensive API for `.kicad_dru` custom design rules files.

## Quick Start

```python
from kiutils.dru import DesignRules, Rule, Constraint

# Load design rules
dru = DesignRules.from_file("project.kicad_dru")

# Create new design rules
dru = DesignRules.create_new()

# Add a rule
dru.rules.append(
    Rule(
        name="HV_Clearance",
        condition="A.NetClass == 'HV' || B.NetClass == 'HV'",
        constraints=[Constraint(type='clearance', min='2mm')]
    )
)

# Save design rules
dru.to_file("project.kicad_dru")
```

---

## DesignRules Class

Main container for custom design rules.

### Constructor / Factory Methods

```python
# Load from file
dru = DesignRules.from_file(filepath: str, encoding: str = None) -> DesignRules

# Create empty design rules
dru = DesignRules.create_new() -> DesignRules

# Parse from S-Expression
dru = DesignRules.from_sexpr(exp: list) -> DesignRules
```

### Attributes

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `version` | `int` | `1` | File format version |
| `rules` | `List[Rule]` | `[]` | Collection of design rules |
| `filePath` | `str \| None` | `None` | File path (auto-set by `from_file()`) |

### Methods

```python
# Save to file
dru.to_file(filepath: str = None, encoding: str = None)

# Generate S-Expression
dru.to_sexpr(indent: int = 0, newline: bool = False) -> str
```

---

## Rule Class

Individual custom design rule definition.

```python
from kiutils.dru import Rule, Constraint

rule = Rule(
    name="My_Rule",
    condition="A.NetClass == 'Power'",
    constraints=[Constraint(type='track_width', min='0.5mm')],
    layer='F.Cu',          # Optional: specific layer
    severity='error'       # Optional: v7+ (warning, error, exclusion, ignore)
)
```

### Attributes

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `name` | `str` | `""` | Rule name |
| `constraints` | `List[Constraint]` | `[]` | Associated constraints |
| `condition` | `str` | `""` | Rule condition expression |
| `layer` | `str \| None` | `None` | Target canonical layer |
| `severity` | `str \| None` | `None` | Severity level (v7+) |

### Severity Levels (v7+)

| Value | Description |
|-------|-------------|
| `warning` | Show as warning in DRC |
| `error` | Show as error in DRC |
| `exclusion` | Exclude from DRC |
| `ignore` | Ignore this rule |

---

## Constraint Class

Design rule constraint specification.

```python
from kiutils.dru import Constraint

# Simple clearance constraint
constraint = Constraint(
    type='clearance',
    min='0.2mm'
)

# Range constraint
constraint = Constraint(
    type='track_width',
    min='0.15mm',
    opt='0.25mm',
    max='1.0mm'
)

# Element-specific constraint
constraint = Constraint(
    type='disallow',
    elements=['via', 'micro_via']
)
```

### Attributes

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `type` | `str` | `"clearance"` | Constraint type |
| `min` | `str \| None` | `None` | Minimum allowed value |
| `opt` | `str \| None` | `None` | Optimum value |
| `max` | `str \| None` | `None` | Maximum allowed value |
| `elements` | `List[str]` | `[]` | Element types to include |

### Constraint Types

| Type | Description |
|------|-------------|
| `annular_width` | Width of annular ring |
| `clearance` | Clearance between items |
| `courtyard_clearance` | Courtyard-to-courtyard clearance |
| `diff_pair_gap` | Gap between differential pairs |
| `diff_pair_uncoupled` | Uncoupled differential pair length |
| `disallow` | Disallow specific elements |
| `edge_clearance` | Clearance from board edges |
| `length` | Track/net length |
| `hole_clearance` | Clearance from holes |
| `hole_size` | Hole size constraints |
| `silk_clearance` | Silkscreen clearance |
| `skew` | Length difference (skew) |
| `track_width` | Track width constraints |
| `via_count` | Number of vias |
| `via_diameter` | Via diameter constraints |

### Element Types

For `disallow` and element-specific constraints:

| Element | Description |
|---------|-------------|
| `buried_via` | Buried vias |
| `micro_via` | Micro vias |
| `via` | Standard vias |
| `graphic` | Graphic items |
| `hole` | Holes |
| `pad` | Pads |
| `text` | Text items |
| `track` | Tracks |
| `zone` | Zones |

---

## Condition Expressions

Design rule conditions use KiCad's expression syntax. Items can be accessed as `A` (first item) and `B` (second item) for clearance rules, or just `A` for single-item rules.

### Common Properties

| Property | Description |
|----------|-------------|
| `A.NetClass` | Net class name |
| `A.NetName` | Net name |
| `A.Net` | Net number |
| `A.Type` | Item type |
| `A.Layer` | Layer name |
| `A.isPlated()` | Is plated hole |
| `A.Pad_Type` | Pad type (PTH, SMD, etc.) |

### Functions

| Function | Description |
|----------|-------------|
| `A.inDiffPair('*')` | Item is in any diff pair |
| `AB.isCoupledDiffPair()` | A and B are coupled diff pair |
| `A.existsOnLayer('F.Cu')` | Item exists on layer |
| `A.memberOf('group')` | Item is member of group |
| `A.insideCourtyard('U1')` | Item inside footprint courtyard |
| `A.insideArea('zone')` | Item inside rule area |
| `A.enclosedByArea('zone')` | Item fully enclosed by area |
| `A.intersectsArea('zone')` | Item intersects area |

### Operators

| Operator | Description |
|----------|-------------|
| `==` | Equal |
| `!=` | Not equal |
| `<`, `>`, `<=`, `>=` | Comparisons |
| `&&` | Logical AND |
| `\|\|` | Logical OR |
| `!` | Logical NOT |

---

## Common Patterns

### High Voltage Clearance

```python
from kiutils.dru import DesignRules, Rule, Constraint

dru = DesignRules.create_new()

dru.rules.append(
    Rule(
        name="HV_Clearance",
        condition="A.NetClass == 'HighVoltage' || B.NetClass == 'HighVoltage'",
        constraints=[Constraint(type='clearance', min='2mm')]
    )
)

dru.to_file("project.kicad_dru")
```

### Differential Pair Constraints

```python
dru.rules.append(
    Rule(
        name="USB_DiffPair",
        condition="A.inDiffPair('USB*')",
        constraints=[
            Constraint(type='track_width', min='0.1mm', opt='0.12mm'),
            Constraint(type='diff_pair_gap', min='0.1mm', opt='0.12mm')
        ]
    )
)
```

### Disallow Vias in Area

```python
dru.rules.append(
    Rule(
        name="NoVias_BGA",
        condition="A.insideCourtyard('U1')",
        constraints=[Constraint(type='disallow', elements=['via', 'buried_via', 'micro_via'])]
    )
)
```

### Board Edge Clearance

```python
dru.rules.append(
    Rule(
        name="Edge_Keep_Out",
        condition="A.Type != 'Pad'",
        constraints=[Constraint(type='edge_clearance', min='0.5mm')]
    )
)
```

### Length Matching

```python
dru.rules.append(
    Rule(
        name="DDR_Length_Match",
        condition="A.NetClass == 'DDR_Data'",
        constraints=[
            Constraint(type='skew', max='2mm'),
            Constraint(type='length', max='50mm')
        ]
    )
)
```

### Net Class Specific Track Width

```python
dru.rules.append(
    Rule(
        name="Power_Tracks",
        condition="A.NetClass == 'Power'",
        constraints=[
            Constraint(type='track_width', min='0.5mm', opt='1.0mm'),
            Constraint(type='via_diameter', min='0.8mm')
        ]
    )
)
```

### Layer-Specific Rules

```python
dru.rules.append(
    Rule(
        name="Inner_Clearance",
        layer='In1.Cu',
        condition="",  # Applies to all items on layer
        constraints=[Constraint(type='clearance', min='0.25mm')]
    )
)
```

### Silkscreen Clearance from Pads

```python
dru.rules.append(
    Rule(
        name="Silk_Pad_Clearance",
        condition="A.Type == 'Text' && B.Type == 'Pad'",
        constraints=[Constraint(type='silk_clearance', min='0.15mm')]
    )
)
```

### Complete DRU File Example

```python
from kiutils.dru import DesignRules, Rule, Constraint

dru = DesignRules.create_new()

# General clearance
dru.rules.append(
    Rule(name="Default_Clearance", condition="",
         constraints=[Constraint(type='clearance', min='0.15mm')])
)

# Power nets
dru.rules.append(
    Rule(name="Power_Width", condition="A.NetClass == 'Power'",
         constraints=[Constraint(type='track_width', min='0.3mm', opt='0.5mm')])
)

# High-speed signals
dru.rules.append(
    Rule(name="HighSpeed_Length", condition="A.NetClass == 'HighSpeed'",
         constraints=[
             Constraint(type='length', max='100mm'),
             Constraint(type='skew', max='5mm')
         ])
)

# BGA area
dru.rules.append(
    Rule(name="BGA_NoVia", condition="A.insideCourtyard('U1')",
         constraints=[Constraint(type='disallow', elements=['via'])])
)

dru.to_file("project.kicad_dru")
```

### Read and Modify Existing Rules

```python
dru = DesignRules.from_file("project.kicad_dru")

# Find and modify a rule
for rule in dru.rules:
    if rule.name == "HV_Clearance":
        for constraint in rule.constraints:
            if constraint.type == 'clearance':
                constraint.min = '3mm'

# Add new rule
dru.rules.append(
    Rule(name="New_Rule", condition="A.NetName == '/RESET'",
         constraints=[Constraint(type='clearance', min='0.5mm')])
)

dru.to_file()
```

### List All Rules

```python
dru = DesignRules.from_file("project.kicad_dru")

for rule in dru.rules:
    print(f"Rule: {rule.name}")
    print(f"  Condition: {rule.condition}")
    print(f"  Layer: {rule.layer}")
    for c in rule.constraints:
        print(f"  Constraint: {c.type} min={c.min} opt={c.opt} max={c.max}")
    print()
```
