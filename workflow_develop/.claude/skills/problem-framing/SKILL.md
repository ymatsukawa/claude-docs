---
name: problem-framing
description: Analyze current vs desired state (AsIs-ToBe) to identify gaps before implementation
---

# Problem Framing Skill

Frame problems using AsIs-ToBe analysis before implementation.

## Step 1: Gather Context from User

Ask the user for:
- **Why**: The motivation or reason behind the request
- **Goal**: The desired end state (ToBe)

## Step 2: Perform AsIs-ToBe Analysis

### AsIs (Current State)
Analyze and document the current situation:
- What exists now?
- How does it currently work?
- What are the current limitations?

### ToBe (Desired State)
Define the target state:
- What should exist after completion?
- How should it work?
- What capabilities should be added?

## Step 3: Identify the Gap (Problem)

Compare AsIs and ToBe to find:

```
Problem = ToBe - AsIs
```

Break down into:
1. **Already achieved**: Parts of the goal that exist now
2. **Not yet achieved**: Parts of the goal that are missing
3. **Gap analysis**: Specific differences between current and desired

## Step 4: Output Problem Statement

Present findings in this format:

```markdown
## Problem Statement

### Why (Motivation)
[User's reason for the change]

### Goal (ToBe)
[Desired end state]

### Current State (AsIs)
[What exists now]

### Gap (Problem to Solve)
- [Gap item 1]
- [Gap item 2]
- [Gap item 3]

### Already Achieved
- [What's already in place]

### Not Yet Achieved
- [What needs to be built/changed]
```

## Example

**User request**:
"I want to add dark mode to my app because to reduce eye strain for users at night."

**Analysis**:
- Why: Reduce eye strain for users at night
- Goal (ToBe): App supports light and dark themes
- Current (AsIs): App has only light theme hardcoded
- Gap: Theme switching mechanism, dark color palette, user preference storage

**Problem Statement**:
- Already achieved: UI components exist, styling system in place
- Not yet achieved: Theme context, dark colors, toggle UI, preference persistence

## Error
When user's input is vague about goal or why
- abort all steps
- ask again "what is goal or why(background)"
