---
name: clean-coding
description: Apply when writing or reviewing code. Small, focused tidying that makes the next change easier.
---

# Clean Coding

Small improvements (minutes to an hour) for **low coupling and high cohesion**. Not big refactoring—elemental cleanup that makes the next change easier.

## 1. Guard Early

Exit early with guard clauses instead of nesting.

```
// NG: nested conditions
if (condition)
  if (nested) return result
  main_process()

// OK: guards at top
if (not condition) return
if (not nested) return
main_process()
```

## 2. Delete Dead Code

Remove unused code. Verify it's truly dead first.

## 3. Normalize Symmetry

Same operation → same pattern. Inconsistency implies different intent.

## 4. Reading Order

Public/high-level first, private/details after.

## 5. Cohesion Order

Code that changes together stays together.

## 6. Declaration Near Use

Declare and initialize variables close to where they're used.

## 7. Explaining Variables

Extract complex expressions into named variables.

```
// NG
return Point((x1+x2)/2 + offset*cos(angle), (y1+y2)/2 + offset*sin(angle))

// OK
center_x = (x1 + x2) / 2 + offset * cos(angle)
center_y = (y1 + y2) / 2 + offset * sin(angle)
return Point(center_x, center_y)
```

## 8. Explaining Constants

Name magic numbers meaningfully: `ACTIVE_STATUS = 1` not `STATUS = 1`.

## 9. Explicit Parameters

Make required inputs explicit, not buried in config objects.

## 10. Chunk Statements

Blank lines between logical sections.

## 11. Extract Helper

Extract shared code with clear purpose into helper functions.

## 12. Consolidate Before Refactoring

Over-fragmented code hides the big picture. Inline first to see the whole, then re-extract meaningfully.

Signs: repeated parameter lists, repeated conditionals, unclear helper names, shared mutable state.

## 13. Comment Why, Not What

Document context, constraints, and tradeoffs—not what the code literally does.

```
// NG: restates code
// Set max to 100 if greater than 100

// OK: explains why
// UI crashes with 100+ elements (see github: #789)
if max > 100 then max = 100
```
