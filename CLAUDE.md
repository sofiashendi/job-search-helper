# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Purpose

Job search toolkit powered by Claude Code skills. Evaluate job postings, generate tailored resumes, prep for behavioral interviews, and optionally track applications in Notion, Obsidian, or local files.

## Resume Variants

Run `/onboard` to import your resume from PDF, DOCX, or markdown. Your resume files are saved to `resumes/`.

Create a variant for each type of role you're targeting. People often apply to different kinds of positions. For example, both management and individual contributor roles, or roles across different fields entirely. Each variant highlights the experience and skills most relevant to that role type.

Examples:

- `product-manager.md` - Strategy, roadmap ownership, cross-functional leadership
- `software-engineer.md` - Hands-on technical work, system design, individual contributions
- `marketing-director.md` - Campaign management, brand strategy, team leadership
- `data-analyst.md` - Analysis, visualization, business insights

Name your files however makes sense for your search. The `/onboard` skill will guide you through creating variants.

## Skills

### /onboard
Import your resume from PDF, DOCX, or markdown into the expected format. Supports multiple variants for different role types.

### /job-match
Evaluate a job posting URL against your resume. Outputs:
- Match score (%) with Apply/Consider/Skip recommendation
- Breakdown by category (Required Skills, Experience, Nice-to-haves)
- Gaps and clarifying questions

### /resume-tailor
After job-match confirms fit, generates tailored markdown resume:
- Saves markdown and PDF to `resumes/[company]/` with YAML frontmatter (base, company, role, url)
- Converts to PDF using `npx md-to-pdf` (auto-installed on first run)
- Optionally adds job to application tracker (if configured)

### /star-prep
Interview prep when you've landed an interview:
- Creates and maps STAR stories to role competencies
- Guides conversational story creation for gaps
- Reviews and improves existing stories

### /application-tracker
Track applications and STAR stories using Notion, Obsidian, or local files. See skill documentation for setup instructions.

## Workflow

0. (First time) Run `/onboard` to import your resume
1. Provide job posting URL
2. Run `/job-match` to score fit
3. If Apply/Consider, run `/resume-tailor` (optionally tracks application)
4. Review generated resume
5. When you land an interview, prep with `/star-prep`

## Application Tracking (Optional)

Application tracking setup is included in `/onboard`. If you skipped it or want to change backends, run `/application-tracker`.

- **Notion** - Stores applications and STAR stories in Notion databases.
- **Obsidian** - Stores as markdown notes in your vault.
- **Local files** - No setup needed. Stores as markdown files in `applications/` and `star-stories/`.

## Changelog

**Automated rule — Claude Code MUST follow this on every commit:**

Before creating any git commit:
1. Update `CHANGELOG.md` under the current version section with a brief summary of what changed. Group entries under `### Added`, `### Changed`, `### Fixed`, or `### Removed` as appropriate. If the current version section doesn't exist, create it below the `# Changelog` heading.
2. Keep the version number in sync across all three files: `CHANGELOG.md`, `.claude-plugin/plugin.json` (`version` field), and `.claude-plugin/marketplace.json` (`plugins[0].version` field). If bumping the version, update all three.

Never skip this step.
