# Library Table API Reference

Comprehensive API for `fp_lib_table` and `sym_lib_table` library management files.

## Quick Start

```python
from kiutils.libraries import LibTable, Library

# Load footprint library table
fp_table = LibTable.from_file("fp_lib_table")

# Load symbol library table
sym_table = LibTable.from_file("sym_lib_table")

# Create new library table
table = LibTable.create_new(type='fp_lib_table')

# Add library entry
table.libs.append(
    Library(
        name="MyLibrary",
        type="KiCad",
        uri="${KIPRJMOD}/MyLibrary.pretty",
        description="My custom footprint library"
    )
)

# Save
table.to_file("fp_lib_table")
```

---

## LibTable Class

Container for library table entries.

### Constructor / Factory Methods

```python
# Load from file
table = LibTable.from_file(filepath: str, encoding: str = None) -> LibTable

# Create new empty table
table = LibTable.create_new(type: str = 'sym_lib_table') -> LibTable

# Parse from S-Expression
table = LibTable.from_sexpr(exp: list) -> LibTable
```

### Attributes

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `type` | `str` | `'sym_lib_table'` | Table type |
| `libs` | `List[Library]` | `[]` | Library entries |
| `filePath` | `str \| None` | `None` | File path (auto-set by `from_file()`) |

### Table Types

| Type | Description |
|------|-------------|
| `fp_lib_table` | Footprint library table |
| `sym_lib_table` | Symbol library table |

### Methods

```python
# Save to file
table.to_file(filepath: str = None, encoding: str = None)

# Generate S-Expression
table.to_sexpr(indent: int = 0, newline: bool = True) -> str
```

---

## Library Class

Individual library entry.

```python
from kiutils.libraries import Library

lib = Library(
    name="MyFootprints",
    type="KiCad",
    uri="${KIPRJMOD}/MyFootprints.pretty",
    options="",
    description="Project-specific footprints",
    active=True
)
```

### Attributes

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `name` | `str` | `""` | Display name in KiCad |
| `type` | `str` | `"KiCad"` | Library type |
| `uri` | `str` | `""` | Path to library |
| `options` | `str` | `""` | Additional options |
| `description` | `str` | `""` | Library description |
| `active` | `bool` | `True` | Library loaded status |

### Library Types

| Type | Description |
|------|-------------|
| `KiCad` | Native KiCad library format |
| `Legacy` | Legacy KiCad format |
| `Eagle` | Eagle library format |
| `Altium` | Altium library format |
| `CADSTAR` | CADSTAR library format |

---

## Path Variables

Common KiCad path variables:

| Variable | Description |
|----------|-------------|
| `${KIPRJMOD}` | Project directory |
| `${KICAD7_SYMBOL_DIR}` | KiCad 7 symbol libraries |
| `${KICAD7_FOOTPRINT_DIR}` | KiCad 7 footprint libraries |
| `${KICAD7_3DMODEL_DIR}` | KiCad 7 3D models |
| `${KICAD_SYMBOL_DIR}` | Default symbol directory |
| `${KICAD_FOOTPRINT_DIR}` | Default footprint directory |
| `${KIUSER_SYMBOL_DIR}` | User symbol directory |
| `${KIUSER_FOOTPRINT_DIR}` | User footprint directory |

---

## Library Table File Locations

### Global Libraries (System-wide)

| OS | fp_lib_table | sym_lib_table |
|----|--------------|---------------|
| Linux | `~/.config/kicad/7.0/fp_lib_table` | `~/.config/kicad/7.0/sym_lib_table` |
| macOS | `~/Library/Preferences/kicad/7.0/fp_lib_table` | `~/Library/Preferences/kicad/7.0/sym_lib_table` |
| Windows | `%APPDATA%\kicad\7.0\fp_lib_table` | `%APPDATA%\kicad\7.0\sym_lib_table` |

### Project Libraries

Located in project directory alongside `.kicad_pro` file.

---

## Common Patterns

### Create Project Library Table

```python
from kiutils.libraries import LibTable, Library

# Create footprint library table
fp_table = LibTable.create_new(type='fp_lib_table')

# Add project-specific library
fp_table.libs.append(
    Library(
        name="ProjectFootprints",
        type="KiCad",
        uri="${KIPRJMOD}/footprints.pretty",
        description="Project footprint library"
    )
)

# Add external library
fp_table.libs.append(
    Library(
        name="MyConnectors",
        type="KiCad",
        uri="/home/user/KiCadLibs/Connectors.pretty",
        description="Custom connector footprints"
    )
)

fp_table.to_file("fp_lib_table")
```

### Create Symbol Library Table

```python
sym_table = LibTable.create_new(type='sym_lib_table')

sym_table.libs.append(
    Library(
        name="ProjectSymbols",
        type="KiCad",
        uri="${KIPRJMOD}/symbols.kicad_sym",
        description="Project symbol library"
    )
)

sym_table.to_file("sym_lib_table")
```

### Add Library to Existing Table

```python
table = LibTable.from_file("fp_lib_table")

# Check if library already exists
existing_names = [lib.name for lib in table.libs]
if "NewLibrary" not in existing_names:
    table.libs.append(
        Library(
            name="NewLibrary",
            type="KiCad",
            uri="${KIPRJMOD}/NewLibrary.pretty",
            description="Added new library"
        )
    )

table.to_file()
```

### Remove Library

```python
table = LibTable.from_file("fp_lib_table")

# Remove by name
table.libs = [lib for lib in table.libs if lib.name != "ObsoleteLibrary"]

table.to_file()
```

### Disable Library (Keep Entry)

```python
table = LibTable.from_file("fp_lib_table")

for lib in table.libs:
    if lib.name == "TemporarilyDisabled":
        lib.active = False

table.to_file()
```

### Enable All Libraries

```python
table = LibTable.from_file("fp_lib_table")

for lib in table.libs:
    lib.active = True

table.to_file()
```

### List All Libraries

```python
table = LibTable.from_file("fp_lib_table")

print(f"Library Table: {table.type}")
print("-" * 60)

for lib in table.libs:
    status = "Active" if lib.active else "Disabled"
    print(f"[{status}] {lib.name}")
    print(f"    Type: {lib.type}")
    print(f"    URI: {lib.uri}")
    if lib.description:
        print(f"    Description: {lib.description}")
    print()
```

### Find Libraries by Path Pattern

```python
import re

table = LibTable.from_file("fp_lib_table")

# Find all project-local libraries
project_libs = [lib for lib in table.libs if "${KIPRJMOD}" in lib.uri]

# Find all external libraries
external_libs = [lib for lib in table.libs if "${KIPRJMOD}" not in lib.uri]

print(f"Project libraries: {len(project_libs)}")
print(f"External libraries: {len(external_libs)}")
```

### Migrate Library Paths

```python
table = LibTable.from_file("fp_lib_table")

# Update paths from old location to new
for lib in table.libs:
    lib.uri = lib.uri.replace(
        "/old/path/to/libs",
        "/new/path/to/libs"
    )

table.to_file()
```

### Update KiCad Version Variables

```python
table = LibTable.from_file("fp_lib_table")

# Update from KiCad 6 to KiCad 7 paths
for lib in table.libs:
    lib.uri = lib.uri.replace("KICAD6_", "KICAD7_")

table.to_file()
```

### Merge Library Tables

```python
# Load both tables
global_table = LibTable.from_file("/path/to/global/fp_lib_table")
project_table = LibTable.from_file("/path/to/project/fp_lib_table")

# Merge (project overrides global)
global_names = {lib.name for lib in global_table.libs}
merged = LibTable.create_new(type='fp_lib_table')

# Add all global libraries
merged.libs.extend(global_table.libs)

# Add project libraries (skip duplicates)
for lib in project_table.libs:
    if lib.name not in global_names:
        merged.libs.append(lib)
    else:
        # Replace global with project version
        merged.libs = [l for l in merged.libs if l.name != lib.name]
        merged.libs.append(lib)

merged.to_file("merged_fp_lib_table")
```

### Validate Library Paths

```python
import os

table = LibTable.from_file("fp_lib_table")

# Expand common variables for validation
def expand_path(uri):
    uri = uri.replace("${KIPRJMOD}", os.getcwd())
    uri = uri.replace("${HOME}", os.path.expanduser("~"))
    return uri

print("Library Path Validation:")
for lib in table.libs:
    expanded = expand_path(lib.uri)
    exists = os.path.exists(expanded)
    status = "OK" if exists else "MISSING"
    print(f"[{status}] {lib.name}: {expanded}")
```

### Export Library List

```python
import json

table = LibTable.from_file("fp_lib_table")

# Export as JSON
libs_json = []
for lib in table.libs:
    libs_json.append({
        "name": lib.name,
        "type": lib.type,
        "uri": lib.uri,
        "description": lib.description,
        "active": lib.active
    })

with open("libraries.json", "w") as f:
    json.dump(libs_json, f, indent=2)
```

### Import Libraries from JSON

```python
import json

# Load JSON configuration
with open("libraries.json") as f:
    libs_config = json.load(f)

# Create new table
table = LibTable.create_new(type='fp_lib_table')

for config in libs_config:
    table.libs.append(
        Library(
            name=config["name"],
            type=config.get("type", "KiCad"),
            uri=config["uri"],
            description=config.get("description", ""),
            active=config.get("active", True)
        )
    )

table.to_file("fp_lib_table")
```

### Batch Create Libraries from Directory

```python
import os

# Scan directory for KiCad libraries
lib_dir = "/path/to/libraries"
table = LibTable.create_new(type='fp_lib_table')

for item in os.listdir(lib_dir):
    if item.endswith(".pretty"):
        name = item.replace(".pretty", "")
        table.libs.append(
            Library(
                name=name,
                type="KiCad",
                uri=os.path.join(lib_dir, item),
                description=f"Auto-added: {name}"
            )
        )

table.to_file("auto_fp_lib_table")
```

### Sort Libraries Alphabetically

```python
table = LibTable.from_file("fp_lib_table")

table.libs.sort(key=lambda lib: lib.name.lower())

table.to_file()
```

### Deduplicate Libraries

```python
table = LibTable.from_file("fp_lib_table")

seen_names = set()
unique_libs = []

for lib in table.libs:
    if lib.name not in seen_names:
        seen_names.add(lib.name)
        unique_libs.append(lib)
    else:
        print(f"Removing duplicate: {lib.name}")

table.libs = unique_libs
table.to_file()
```

### Create Backup Before Modification

```python
import shutil
from datetime import datetime

filepath = "fp_lib_table"

# Create backup
backup_name = f"{filepath}.backup.{datetime.now().strftime('%Y%m%d_%H%M%S')}"
shutil.copy(filepath, backup_name)

# Now safely modify
table = LibTable.from_file(filepath)
# ... modifications ...
table.to_file()
```
