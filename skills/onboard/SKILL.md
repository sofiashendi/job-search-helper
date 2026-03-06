---
name: onboard
description: Use when a user first installs the plugin or wants to import/add a resume. Converts PDF, DOCX, or markdown resumes into the expected format.
---

# Onboard

## Overview

Onboarding skill that imports a resume from any format (PDF, DOCX, or markdown) and converts it into the structured markdown format used by other skills. Supports multiple resume variants for different role types.

## Important: Working Directory

All file paths in this skill (e.g., `resumes/`) are relative to the **user's current working directory** — NOT the plugin installation directory. Never read from or write to the plugin's own folder.

## Security

- **NEVER** accept API keys, tokens, or secrets in chat
- Notion uses OAuth — no keys needed
- Obsidian only needs a vault path (a filesystem path, not sensitive)
- If the user tries to paste a secret, stop them and explain it's not needed

## Process

### 1. Get Resume Input

Ask: "To get started, I'll need your current resume. How would you like to provide it?"
- **File path** — provide a path to a PDF, DOCX, or markdown file
- **Paste text** — paste resume content directly

> **Returning users:** If the user has run `/onboard` before, `resumes/` will already contain variants. In that case, show existing variants and ask: "Would you like to add a new resume variant or replace an existing one?" Do NOT proactively search for existing resumes — only check if the user mentions they've already onboarded.

### 2. Determine Variant Type

Ask: "What type of role does this resume target? (e.g., software engineer, product manager, marketing director, data analyst)"

The user's answer becomes the variant label. Generate filename slug from it: `[role-slug].md` (e.g., `software-engineer.md`, `product-manager.md`, `marketing-director.md`)

#### If file path provided:

Detect format by extension:

**PDF (`.pdf`):**
- Use the Read tool to read the PDF file — Claude can read PDFs natively
- If the extracted text is garbled or incomplete, inform the user: "The PDF text extraction didn't capture your resume well. Could you paste the text directly instead?"

**DOCX (`.docx`):**
- Convert using mammoth:
  ```bash
  mkdir -p .tmp && npx mammoth@1 --output-format=markdown [file-path] --output=.tmp/jsh-resume-extracted.md
  ```
- Read the extracted markdown from `.tmp/jsh-resume-extracted.md`
- Clean up: `rm .tmp/jsh-resume-extracted.md`

**Markdown (`.md`):**
- Read the file directly with the Read tool

#### If paste text:

Accept the pasted content as-is and proceed to restructuring.

### 3. Restructure into Standard Format

Parse the extracted text and restructure it into this format:

```markdown
# [FULL NAME]

[City, State/Province] | [Phone] | [Email] | [LinkedIn URL]
[Additional info line if applicable, e.g., languages spoken]

## Summary

[ROLE TITLE IN CAPS]

[2-3 sentence professional summary highlighting core strengths and years of experience]

## Skills

**[Category]:** [Skill 1], [Skill 2], [Skill 3]
**[Category]:** [Skill 1], [Skill 2], [Skill 3]

## Experience

### [Job Title] - [Company Name]
[Start Date]-[End Date] | [Location]

- [Achievement/responsibility bullet]
- [Achievement/responsibility bullet]

### [Job Title] - [Company Name]
[Start Date]-[End Date] | [Location]

- [Achievement/responsibility bullet]

## Education

### [Degree] - [Institution]
[Years]

## Certifications

[If applicable]
```

**Rules:**
- Preserve ALL content from the original — do not remove, summarize, or fabricate anything
- Reformat into the structure above (headings, bullet points, consistent formatting)
- Group skills by relevant categories for the role (e.g., Languages/Frameworks/Tools for engineers; Platforms/Certifications for marketers; Tools/Methodologies for PMs)
- Ensure experience entries are in reverse chronological order
- Keep bullet points as-is — do not rewrite, merge, or embellish

### 4. Review and Save

Show the restructured resume to the user. Ask: "Does this look correct? Any edits before I save?"

After approval, save to `resumes/[variant-slug].md`

### 5. Set Up Application Tracking (Optional)

After saving the resume, ask:

"Would you like to set up application tracking? You can track applications in Notion, Obsidian, or local files."

Present these options:
- Notion (I'll install it for you)
- Obsidian (I'll install it for you)
- Local files (no setup needed)
- Skip for now

**If Notion:**
1. Run via Bash: `claude mcp add --transport http notion https://mcp.notion.com/mcp`
2. Update the `## Application Tracking` section of CLAUDE.md (create it if it doesn't exist):
   ```markdown
   ## Application Tracking (Optional)

   Backend: Notion
   Status: MCP installed, pending restart
   ```

**If Obsidian:**
1. Ask the user for their Obsidian vault path (just a filesystem path, not sensitive)
2. Validate the path exists via Bash: `test -d "/path/to/vault" && echo "OK" || echo "NOT FOUND"`
3. If the path doesn't exist, ask the user to correct it
4. Run via Bash: `claude mcp add obsidian -- npx -y obsidian-mcp /path/to/vault`
5. Update the `## Application Tracking` section of CLAUDE.md (create it if it doesn't exist):
   ```markdown
   ## Application Tracking (Optional)

   Backend: Obsidian
   Status: MCP installed, pending restart
   ```

**If Local files:**
1. Run via Bash: `mkdir -p applications/ star-stories/`
2. Update the `## Application Tracking` section of CLAUDE.md (create it if it doesn't exist):
   ```markdown
   ## Application Tracking (Optional)

   Backend: Local
   Status: configured
   ```

**After running `claude mcp add`** (Notion or Obsidian), do not attempt to use MCP tools in this session — proceed directly to Step 6.

**If Skip:** Do nothing, proceed to next steps.

### 6. Next Steps

After saving, suggest:
- "Want to add another resume variant? Run `/onboard` again."
- "Ready to evaluate a job posting? Provide a URL and run `/job-match`."

**Additional messaging based on tracking choice:**
- If **Notion** was installed: "All set! Restart Claude Code, then run `/mcp` to authenticate with Notion. After that, tracking is ready."
- If **Obsidian** was installed: "All set! Restart Claude Code — tracking will be ready when you need it."
- If **Local files** or **Skip**: no additional restart messaging needed

Only show the "Want to track applications? Run `/application-tracker`" suggestion if tracking was **skipped** in Step 5.

## Error Handling

- If mammoth fails on DOCX → ask user to paste text directly
- If PDF text is unreadable → provide a specific reason:
  - **Scanned/image PDF** (no text extracted at all): "This appears to be a scanned PDF with no selectable text. Please paste your resume text directly, or provide a DOCX or markdown version."
  - **Encrypted PDF** (Read tool returns permission error): "This PDF is password-protected. Please provide an unencrypted version, or paste the text directly."
  - **Incomplete extraction** (text extracted but sections missing or garbled): "The PDF text extraction captured some content but appears incomplete. Here's what I got: [show extracted text]. Could you paste the full text directly so I can fill in the gaps?"
- If resume structure is ambiguous → ask user to clarify sections (e.g., "Is this a job title or company name?")
