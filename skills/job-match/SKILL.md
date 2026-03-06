---
name: job-match
description: Use when evaluating a job posting to determine fit before applying. Scores match percentage and identifies gaps.
---

# Job Match Scoring

## Overview

Score job postings against your resume to determine fit. Outputs match percentage, category breakdown, gaps, and recommendation (Apply/Consider/Skip).

## Important: Working Directory

All file paths in this skill (e.g., `resumes/`, `applications/`) are relative to the **user's current working directory** — NOT the plugin installation directory. Never read from or write to the plugin's own folder.

## Process

1. **Get job posting**: Fetch URL with WebFetch. If fetch fails or content seems incomplete, ask user to paste the text.

2. **Check for previous application**:

   First, extract company name and role title from the job posting.

   If the application-tracker skill is configured (Notion, Obsidian, or local), check for duplicate applications:
   - Normalize company and role to lowercase for case-insensitive comparison (exact match only, not fuzzy)
   - **Notion**: Query with `{"and": [{"property": "Company", "title": {"equals": "[company]"}}, {"property": "Role", "rich_text": {"equals": "[role]"}}]}` filter
   - **Obsidian**: Search `Job Search/Applications/` for `[Company] - [Role].md`
   - **Local**: Glob `applications/[company-slug]-[role-slug].md`
   - **If duplicate found** (results array not empty):
     - Calculate days since last update
     - Alert user:
       ```
       WARNING: Previous application detected — You applied to [Company] - [Role] [X] days ago.
       Note: Reapplying after 90 days is generally acceptable.
       ```
     - Ask user: "Continue with job-match analysis anyway?" (Yes/No)
     - If No: End process here
     - If Yes: Continue to step 3

   **If no duplicate or no application-tracker integration:** Continue to step 3

3. **Parse requirements**: Categorize each requirement:
   - **Required**: "must have", "required", "X+ years", "must"
   - **Nice-to-have**: "preferred", "bonus", "ideally", "plus", "nice to have"

4. **Select resume to match against**:
   - List available resume variants from `resumes/`
   - Exclude any file with `sample: true` in its YAML frontmatter
   - If no variants remain after exclusion, tell the user: "No resume found. Run /onboard first to import your resume." and stop
   - If only one variant exists → Use it
   - If multiple variants exist → Ask user which variant best fits this role
   - If unclear → Ask user

5. **Score each requirement** against resume:
   - Full match (skill/experience clearly present): 100%
   - Partial match (related but not exact): 50%
   - No match: 0%
   - Unclear: Flag for clarifying question

6. **Calculate weighted score**:
   - Required Skills: 50% weight
   - Experience Relevance: 30% weight
   - Nice-to-haves: 20% weight

   **Per-item scoring:** Each item scores 0% (no match), 50% (partial), or 100% (full). Category score = average of item scores.

   **Weight redistribution:** If a category has 0 items (e.g., no nice-to-haves listed), redistribute its weight proportionally to other categories. Example: 0 nice-to-haves → Required Skills gets 62.5% (50/80), Experience gets 37.5% (30/80).

   **Example calculation:** 3 required skills (100%, 50%, 100%) = 83.3%. Experience = 80%. 2 nice-to-haves (100%, 0%) = 50%. Final = (83.3 x 0.5) + (80 x 0.3) + (50 x 0.2) = 41.7 + 24 + 10 = 75.7% → Good Match (Consider).

7. **Output results** in this format:

```
Score: [X]% - [Strong Match|Good Match|Weak Match] ([Apply|Consider|Skip])

Breakdown:
- Required Skills: [X]% ([N]/[M] matched)
- Experience: [X]% ([notes on fit])
- Nice-to-haves: [X]% ([N]/[M] matched)

Gaps:
- [Requirement] - not found in resume

Questions:
- [Ambiguous requirement] - [clarifying question]?
```

8. **After answering questions**, recalculate and provide final score.

## Thresholds

| Score | Rating | Recommendation |
|-------|--------|----------------|
| 80%+ | Strong Match | Apply |
| 65-79% | Good Match | Consider |
| <65% | Weak Match | Skip |

## Resume Locations

Base resumes in `resumes/`. Users create one variant per role type they're targeting (e.g., `software-engineer.md`, `product-manager.md`, `marketing-director.md`). Run `/onboard` to import your own.

## After Scoring

If recommendation is Apply or Consider, ask:
"Would you like me to generate a tailored resume for this role?"

If yes, invoke the resume-tailor skill.

## Error Handling

- If WebFetch fails or returns empty/garbled content: Ask the user to paste the job posting text directly
- If tracker query fails (Notion/Obsidian API error): Log the error, skip duplicate check, and continue with scoring
- If no resume found in `resumes/` (or only sample): Tell the user "No resume found. Run /onboard first to import your resume." and stop
