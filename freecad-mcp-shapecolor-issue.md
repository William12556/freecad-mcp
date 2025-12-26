Created: 2024 December 26

# FreeCAD MCP ShapeColor Attribute Issue

## Table of Contents

- [Issue Summary](<#issue summary>)
- [Technical Details](<#technical details>)
- [Affected Functions](<#affected functions>)
- [Error Message](<#error message>)
- [Root Cause](<#root cause>)
- [Affected Object Types](<#affected object types>)
- [Proposed Solution](<#proposed solution>)
- [Resolution Status](<#resolution status>)
- [Version History](<#version history>)

## Issue Summary

The `get_objects` and `get_object` tools in FreeCAD MCP version 0.1.13 fail with AttributeError when documents contain objects with ViewProviderDocumentObject view providers, which lack the ShapeColor attribute.

[Return to Table of Contents](<#table of contents>)

## Technical Details

**Repository:** https://github.com/neka-nat/freecad-mcp  
**Affected Version:** 0.1.13  
**File:** `src/freecad_mcp/server.py`  
**Issue Type:** Runtime AttributeError

[Return to Table of Contents](<#table of contents>)

## Affected Functions

### get_objects

Unconditionally accesses `obj.ViewObject.ShapeColor` without checking attribute existence.

### get_object

Unconditionally accesses `obj.ViewObject.ShapeColor` without checking attribute existence.

[Return to Table of Contents](<#table of contents>)

## Error Message

```
Failed to get objects: <Fault 1: "<class 'AttributeError'>:'Gui.ViewProviderDocumentObject' object has no attribute 'ShapeColor'">
```

[Return to Table of Contents](<#table of contents>)

## Root Cause

The implementation assumes all ViewObject instances have a ShapeColor attribute. However, certain view provider types do not support this attribute:

- `ViewProviderDocumentObject` - Used by App::Part, App::Origin
- Other generic view providers without shape rendering

When the code encounters these object types, it attempts to access a non-existent attribute, causing immediate failure.

[Return to Table of Contents](<#table of contents>)

## Affected Object Types

**View Provider Types Without ShapeColor:**

| Object Type | View Provider Type | Has ShapeColor |
|-------------|-------------------|----------------|
| App::Part | ViewProviderDocumentObject | No |
| App::Origin | ViewProviderDocumentObject | No |
| App::Line | ViewProviderGeometryObject | Yes |
| App::Plane | ViewProviderGeometryObject | Yes |
| Part::* | ViewProviderPartExt | Yes |

**Test Case:** Documents containing App::Part or App::Origin objects will trigger this error.

[Return to Table of Contents](<#table of contents>)

## Proposed Solution

Implement defensive attribute checking before accessing ShapeColor:

```python
# Current problematic code
view_object_data = {
    "ShapeColor": obj.ViewObject.ShapeColor,
    "Transparency": obj.ViewObject.Transparency,
    "Visibility": obj.ViewObject.Visibility
}

# Proposed correction
view_object_data = {}
if hasattr(obj.ViewObject, 'ShapeColor'):
    view_object_data["ShapeColor"] = obj.ViewObject.ShapeColor
if hasattr(obj.ViewObject, 'Transparency'):
    view_object_data["Transparency"] = obj.ViewObject.Transparency
if hasattr(obj.ViewObject, 'Visibility'):
    view_object_data["Visibility"] = obj.ViewObject.Visibility
```

**Alternative Approach:**

```python
view_object_data = {
    "ShapeColor": getattr(obj.ViewObject, 'ShapeColor', None),
    "Transparency": getattr(obj.ViewObject, 'Transparency', None),
    "Visibility": getattr(obj.ViewObject, 'Visibility', None)
}
```

[Return to Table of Contents](<#table of contents>)

## Resolution Status

**Status:** Identified, solution proposed  
**Date:** 2024-12-26  
**Next Steps:** Implement fix in fork and submit pull request

[Return to Table of Contents](<#table of contents>)

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2024-12-26 | Initial documentation of ShapeColor issue |

---

Copyright (c) 2025 William Watson. This work is licensed under the MIT License.
