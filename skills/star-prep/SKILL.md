---
name: star-prep
description: Use when user has confirmed an interview for a specific role (e.g., "I got an interview at X for Y role"). Helps create and refine STAR stories mapped to role competencies.
---

# STAR Interview Prep

## Overview

For detailed guidance on the STAR method framework, see `star-method-guide.md` in the project root.

Prepare for a confirmed interview by creating, reviewing, and mapping STAR stories to the role's competencies. Adapts based on whether the user already has stories or is starting from scratch.

## Trigger

This skill activates when a user confirms they have an interview — e.g., "I got an interview at Google for SWE role", "just heard back from Stripe, got the interview!", "interviewing at Acme next week for PM".

## Important: Working Directory

All file paths in this skill (e.g., `resumes/`, `star-stories/`) are relative to the **user's current working directory** — NOT the plugin installation directory. Never read from or write to the plugin's own folder.

## Process

### 1. Extract Interview Context

From the user's message, extract:
- **Company name**
- **Role title**

If either is unclear, ask: "What company and role is the interview for?"

### 2. Load Resume Variant

List available resume variants from `resumes/` (excluding any with `sample: true` in frontmatter).
- If only one non-sample variant exists → use it automatically
- If multiple variants exist → ask the user which to use

Read the resume and extract:
- Role being targeted
- Key responsibilities mentioned
- Technical skills emphasized
- Leadership indicators

### 3. Check for Existing STAR Stories

Load stories from the configured application-tracker backend:
- **Notion**: Query the STAR Stories database
- **Obsidian**: Read from `Job Search/STAR Stories/`
- **Local** (default): Read from `star-stories/` directory

Stories may or may not exist — both cases are fine. If the user says "I already have stories" and wants to paste them, accept the paste and create the files, then continue to step 4.

### 4. Map Stories to Competency Categories

| Category | Indicators in Resume/Role |
|----------|--------------------------|
| Leadership | Team size, mentoring, decisions, strategy |
| Problem-solving | Complex challenges, analytical thinking, root cause analysis |
| Conflict | Disagreements, pushback, negotiation |
| Failure | Mistakes, lessons learned, recovery |
| Growth | Feedback, skill development, adaptation |
| Collaboration | Cross-functional, stakeholders, partnerships |
| Delivery | Deadlines, prioritization, results, shipping |

Stories can map to multiple categories (e.g., a story about leading a cross-functional project could count under both Leadership and Collaboration). Count the story under each applicable category.

If no stories exist yet, all categories show as gaps — that's expected.

### 5. Show Coverage Analysis

```
## STAR Coverage for [Role] at [Company]

| Category | Covered By | Gap? |
|----------|------------|------|
| Leadership | [Story name] or "None" | Yes/No |
| Problem-solving | ... | ... |
...

### Gaps to Address
- [Category]: [Why this matters for the role]
```

### 6. Create Stories for Gaps

For each gap, guide the user through creating a new story conversationally:

**Step 1 — Identify the story**:
"For [competency], think of a specific time when [scenario]. What was happening? Where were you working?"

**Step 2 — Clarify the task**:
"What was your specific responsibility in this situation? What did success look like?"

**Step 3 — Draw out actions**:
"Walk me through what you actually did, step by step. What decisions did you make and why?"

**Step 4 — Capture results**:
"What was the outcome? Any metrics, feedback, or lessons learned?"

**Step 5 — Format and review**:
Show the formatted STAR entry:
```
**Name**: [Descriptive title]

**Situation**: [1-2 sentences from Step 1]

**Task**: [1-2 sentences from Step 2]

**Action**: [2-4 sentences from Step 3, first-person]

**Result**: [1-2 sentences from Step 4, with metrics if available]
```

Ask: "Does this capture your story accurately? Any edits?"

**Step 6 — Save** via the configured application-tracker backend:
- **Notion**: Create entry in the STAR Stories database
- **Obsidian**: Create note in `Job Search/STAR Stories/`
- **Local** (default): Run `mkdir -p star-stories/` then save as `star-stories/[slug].md` with the following format:

```markdown
---
name: [Descriptive title]
category: [competency category]
created: [date]
---

## Situation
[1-2 sentences]

## Task
[1-2 sentences]

## Action
[2-4 sentences]

## Result
[1-2 sentences]
```

### 7. Improve Existing Stories (if any)

If stories were loaded in step 3, review them for quality issues:
- **Incomplete**: Missing any S/T/A/R section
- **Too verbose**: Any section >3 sentences
- **Vague actions**: Uses "worked on", "helped with", "was involved in"
- **Missing metrics**: Result lacks quantifiable outcomes
- **Generic situation**: No specific role, company, or timeframe

For each issue found, show the suggested improvement and ask: "Apply these improvements?"

Save improvements via the configured backend.

### Quality Criteria

| Section | Target | Watch For |
|---------|--------|-----------|
| Situation | 1-2 sentences | Generic context, missing stakes |
| Task | 1-2 sentences | Unclear responsibility, no success criteria |
| Action | 2-4 sentences | "We" instead of "I", vague steps |
| Result | 1-2 sentences | No metrics, no lessons learned |

**Quality thresholds:**
- **Good**: All four STAR sections present, uses "I" not "we" in Action, Result includes a metric or concrete outcome, no section exceeds 3 sentences
- **Needs Improvement**: Any of the above criteria fails

## Common Behavioral Questions by Category

Use these to prompt story recall:

**Leadership**:
- "Tell me about a time you had to make a difficult decision for your team"
- "Describe how you've mentored or developed someone"
- "How have you handled underperformance?"

**Problem-solving**:
- "Describe a complex problem you solved"
- "Tell me about a difficult decision you had to make with incomplete information"
- "How do you approach diagnosing the root cause of an issue?"

**Conflict**:
- "Tell me about a disagreement with a colleague"
- "How have you handled pushback on your ideas?"
- "Describe working with a difficult stakeholder"

**Failure**:
- "Tell me about a mistake you made"
- "Describe a project that didn't go as planned"
- "What's a time you missed a deadline?"

**Growth**:
- "Tell me about feedback that changed how you work"
- "How have you developed a new skill recently?"
- "Describe adapting to a significant change"

**Collaboration**:
- "How do you work with stakeholders from different teams or backgrounds?"
- "Describe a cross-functional project you led"
- "Tell me about aligning competing priorities"

**Delivery**:
- "Tell me about delivering under a tight deadline"
- "How do you prioritize when everything is urgent?"
- "Describe shipping something you're proud of"

## Error Handling

- If stories fail to load from backend (API/MCP error): Inform the user, suggest switching to local files or retrying
- If saving a story fails: Show the formatted story to the user so they can save it manually, and report the error
- If backend is unreachable mid-session: Continue working in memory and attempt to save at the end
