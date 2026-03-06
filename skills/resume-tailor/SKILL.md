---
name: resume-tailor
description: Use after job-match confirms a good fit. Generates tailored markdown resume without fabricating experience.
---

# Resume Tailor

## Overview

Generate a tailored markdown resume based on job-match analysis. Conservative modifications only - reframe existing experience, never fabricate.

## Prerequisites

- job-match skill must have been run first
- Job posting URL must be available
- Gap analysis must be available

**Before starting:** Look for job-match score output earlier in the conversation (the `Score: X% - ...` output). If not found, tell the user: "I need a job-match analysis first. Would you like me to run /job-match now?" and stop until job-match has been completed. If the conversation is long and the score output isn't visible, ask the user to confirm the company, role, and match score before proceeding.

## Important: Working Directory

All file paths in this skill (e.g., `resumes/`) are relative to the **user's current working directory** — NOT the plugin installation directory. Never read from or write to the plugin's own folder.

## Process

1. **Select base resume**:
   - List available resume variants from `resumes/`
   - If only one variant exists → Use it
   - If multiple variants exist → Ask user which variant best fits this role
   - If unclear, ask user

2. **Read base resume** markdown file

3. **Apply modifications** (conservative only):

   **DO:**
   - Rewrite summary to address specific role requirements (using only themes that span multiple roles)
   - Swap equivalent keywords to match job posting terminology (e.g., "JS" → "JavaScript", "CI/CD" → "continuous integration")
   - Reorder bullet points (most relevant first per role)
   - Reorder skills section (required skills first)

   **Keyword swapping means equivalents only** - NOT adding new skills:
   - "Node" → "Node.js" ✓ (same thing, different name)
   - "Project management" → "Program management" ✓ (equivalent term)
   - Nothing → "WebSockets" ✗ (fabrication)
   - Nothing → "Salesforce" ✗ (fabrication)

   **DO NOT:**
   - Add skills or experience not in original resume
   - Remove any content (certifications, experience, skills, etc.)
   - Change job titles, companies, or dates
   - Fabricate metrics or achievements
   - Add fluff language
   - Invent anything
   - **Add skills from job posting that aren't in the base resume** - this is fabrication:
     - If job posting mentions a skill/tool/certification not in the resume → do NOT add it
     - Only use skills and experience that actually appear in the base resume
   - **Promote one-time experiences to summary-level themes** - summary should reflect patterns, not isolated instances:
     - If a technique appears in one bullet point → leave it there, don't elevate to summary
     - Summary should only include themes that span multiple roles or are core competencies
   - **Inflate seniority level or titles** - use language that matches actual resume experience:
     - Use terms that match the titles actually held in the resume
     - When in doubt, use the lower/more conservative framing

4. **Generate markdown resume** with frontmatter and structure:

```markdown
---
base: [variant name]
company: [Company Name]
role: [Role Title]
url: [Job Posting URL]
---

# [YOUR NAME]

[City, State/Province] | [Phone] | [Email] | [LinkedIn]
[Additional info like languages]

## Summary

[ROLE TITLE IN CAPS]

[Tailored 2-3 sentence summary addressing this specific role]

## Skills

[Reordered to prioritize skills mentioned in job posting]

## Experience

### [Title] - [Company]
[Date Range] | [Location]

- [Bullet points reordered by relevance to this role]
- [Keywords swapped to match job posting terminology]

## Education

[Standard education section]

## Certifications

[If relevant to role]
```

5. **Save resume**:
   - Generate slugs: lowercase, spaces to hyphens, strip characters not matching `[a-z0-9-]`, collapse consecutive hyphens
   - Run `mkdir -p resumes/[company-slug]` to ensure the directory exists
   - Save to `resumes/[company-slug]/[role-slug].md`

6. **Convert to PDF**:

   > **Note:** First run downloads md-to-pdf dependencies (~200MB for Chromium) and may take a minute.

   **6a. Check font preference**: Read `.job-search-helper.json` in the user's working directory. If it exists and contains a `font` key, use that value silently — do NOT ask again.

   If the file doesn't exist or has no `font` key, ask: "What font would you like for your resume? You can name any Apple system font (e.g., SF Pro, SF Pro Rounded, New York, SF Mono). Default: SF Pro Rounded"
   - If user accepts default or doesn't respond → use SF Pro Rounded
   - Save the choice to `.job-search-helper.json` (create the file if needed, merge with existing keys if present):
     ```json
     { "font": "SF Pro Rounded" }
     ```
   - Use the chosen font name directly in `font-family` in step 6c

   **6b. Create temp directory**: Run `mkdir -p .tmp` in the user's working directory.

   **6c. Preprocess the markdown** — read the saved resume file and apply these transformations (results will be written in step 6e):
   - Strip YAML frontmatter (`---...---` block), but extract `company` value for filename
   - Extract the candidate name from the `# H1` heading
   - Put language/extra info line on its own line: if a line ends with `[LinkedIn](...)` followed by `Fluent in ...`, insert `\n<br>` between them
   - Replace `## Summary` heading: remove the `## Summary` line and wrap the next non-empty line (the role title) in `<div class="role-title">...</div>`
   - Convert bare URLs (not already in markdown link syntax) to clickable `[url](url)` links

   **6d. Write temp CSS** to `.tmp/jsh-resume-style.css` with this content (substitute the user's chosen font):

   ```css
   :root {
     --navy: #13284b;
     --link-blue: #103cc0;
   }

   body {
     font-family: 'SF Pro Rounded', ui-rounded, sans-serif; /* Replace with user's chosen font */
     font-size: 11pt;
     line-height: 1.4;
     color: #000;
     margin: 0;
     padding: 0;
   }

   @page {
     size: letter;
     margin: 0.5in 0.6in;
   }

   h1 {
     font-size: 16pt;
     font-weight: bold;
     text-align: center;
     margin: 0 0 6pt 0;
   }

   h1 + p {
     text-align: center;
     margin: 0 0 6pt 0;
   }

   .role-title {
     font-size: 14pt;
     font-weight: bold;
     color: var(--navy);
     text-transform: uppercase;
     text-align: center;
     margin: 12pt 0;
   }

   h2 {
     font-size: 11pt;
     font-weight: bold;
     color: var(--navy);
     text-transform: uppercase;
     margin: 12pt 0 6pt 0;
     border: none;
   }

   h3 {
     font-size: 11pt;
     font-weight: bold;
     margin: 8pt 0 2pt 0;
   }

   h3 + p {
     font-size: 10pt;
     font-style: italic;
     margin: 0 0 4pt 0;
   }

   p {
     margin: 0 0 6pt 0;
     text-align: justify;
   }

   ul {
     margin: 0 0 6pt 0;
     padding-left: 20pt;
   }

   li {
     margin-bottom: 2pt;
   }

   a {
     color: var(--link-blue);
     text-decoration: underline;
   }

   strong {
     font-weight: bold;
   }

   h2, h3 {
     page-break-after: avoid;
   }

   li, p {
     page-break-inside: avoid;
   }
   ```

   **6e. Write preprocessed markdown** to `.tmp/jsh-resume-preprocessed.md`

   **6f. Run conversion**:

   ```bash
   npx md-to-pdf@5 .tmp/jsh-resume-preprocessed.md --stylesheet .tmp/jsh-resume-style.css --pdf-options '{"format":"Letter","margin":{"top":"0.5in","bottom":"0.5in","left":"0.6in","right":"0.6in"},"printBackground":true}'
   ```

   **6g. Move output** from `.tmp/jsh-resume-preprocessed.pdf` to `resumes/[company]/[name]-[role].pdf`
   - Use the candidate name from H1 and company from frontmatter
   - Strip special characters from filename: `,/:*?"<>|`
   - If no company, use `[Name] Resume.pdf`

   **6h. Clean up** temp files — always run this on both success and failure:
   ```bash
   rm -f .tmp/jsh-resume-style.css .tmp/jsh-resume-preprocessed.md .tmp/jsh-resume-preprocessed.pdf
   ```

7. **Show the user** the generated resume and PDF location

8. **Add to tracker** (if application-tracker skill is configured — Notion, Obsidian, or local):
   - If Notion backend is configured but database IDs are missing from CLAUDE.md, prompt user to run `/application-tracker` first instead of failing silently
   - Add application entry with Company, Role, Job Posting URL, and Stage = "Applied"
   - Confirm to user: "Added [Company] - [Role] to your Applications tracker."

## Error Handling

- If company/role extraction fails → Ask user to confirm values
- If tracker API fails → Show error, tell user to add manually
- If resume generation has issues → Show what was generated, ask for feedback
- If `npx md-to-pdf` fails (non-zero exit code) → Tell the user the markdown resume was saved successfully to `resumes/[company-slug]/[role-slug].md` and suggest manual PDF conversion (e.g., online converter or word processor)
- **Always clean up** — step 6h runs on both success and failure paths
