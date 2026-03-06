---
name: job-search-toolkit
description: AI-powered job search workflow. Import resumes, score job postings, tailor resumes, prep for behavioral interviews, and track applications.
---

# Job Search Toolkit

A complete job search workflow powered by Claude Code skills.

## Available Skills

### /onboard
**Trigger:** User first installs the plugin or wants to import/add a resume
Import resumes from PDF, DOCX, or markdown into the expected format. Supports multiple variants for different role types.

### /job-match
**Trigger:** User provides a job posting URL or asks to evaluate a role
Score job postings against your resume with weighted criteria. Get a match percentage, gap analysis, and Apply/Consider/Skip recommendation.

### /resume-tailor
**Trigger:** After job-match confirms fit, or user asks to tailor a resume
Generate a tailored markdown resume from your base resume. Conservative modifications only — reframes existing experience, never fabricates.

### /star-prep
**Trigger:** User confirms they have an interview (e.g., "got an interview at Google for SWE role")
Creates and maps STAR stories to role competencies, guides conversational story creation for gaps, and improves existing stories.

### /application-tracker
**Trigger:** User wants to set up or configure application tracking
Track applications and STAR stories using Notion, Obsidian, or local files. When configured, other skills use it automatically.

## Typical Workflow

```
0. /onboard                  → Import resume (first time only)
1. /job-match [url]        → Score: 85% - Apply
2. /resume-tailor          → resumes/company-role.md + PDF
3. /star-prep              → STAR stories for your interview
```
