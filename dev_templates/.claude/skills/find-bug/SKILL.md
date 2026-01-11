---
name: find-bug
description: Find bug code and recommend to fix if it's required.
---

# Find bug

Find and report target bug.

## Workflow

**User provides**
- What is the bug
- (Optional) Where error happened

**Claude code does**
- Research the bug with what and where
- Report reason and reproduction
- Ask user if fix recommendations needed (using AskUserQuestion)
- (If yes) Suggest fix recommendations

## User Input

User should input:
- What is the bug
- (Optional) Where error happened

If "what is the bug" is not provided, stop workflow and ask user to clarify.

## Claude Code Investigation

Find these **facts**:
- [ ] What happened?
- [ ] Where happened?
- [ ] Scope of influence
  - [ ] Function level?
  - [ ] Single module level?
  - [ ] Multiple modules level?
- [ ] How to reproduce?

After investigation:
1. Use AskUserQuestion to ask: "Do you need fix recommendations?"
2. Create report:
   - [ ] What is the reason? (Use what and where)
   - [ ] (If user said yes) How can user fix? (Use scope and reproduction)

## Report Template

```
# <title>

## Point
{Conclusion: core reason of the bug}

## Reason
{Why this bug happened}

## Scope of Influence
{Show scope of Influence}

## Reproduction Example
{Show how bug occurs}

## Steps to Reproduce (Optional)
{Write in Gherkin style. Skip if simple bug}
* Given: ...
* When: ...
* Then: ...

## Fix Recommendations (Optional)
{Include only if user requested}

### Plan 1
{content}

### Plan 2
{content}
```

## Example

```
# Panic happened in SampleService

## Point
`nil pointer exception` of `user` in line 500.

## Reason
When argument passed around x < 0, repository layer panics

## Scope of Influence
All of modules that use UserRepository.find

## Reproduction Example
`u, err := UserRepository.find(x)` << When `x` is -1, panics

## Steps to Reproduce
* Given: In UserService layer, `x` is passed as x < 0
* When: `x` is passed to UserRepository.find
* Then: UserRepository.find panics

## Fix Recommendations

### Plan 1
(Recommended) Validate in repository layer and return nil.
Because many 100+ modules uses target repository

### Plan 2
Guard in service layer.
Repository layer should not validate data range.
```
