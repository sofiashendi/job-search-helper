# Changelog

## 1.0.0

- 5 skills: onboard, job-match, resume-tailor, star-prep, application-tracker
- Application tracking via Notion, Obsidian, or local files
- MCP server auto-install during `/onboard` with OAuth for Notion
- PDF resume generation with md-to-pdf (font preference saved to `.job-search-helper.json`)
- STAR method interview prep with competency mapping
- Obsidian backend uses single Overview dashboard with Dataview tables
- 3-phase setup flow (Detect → Install → Configure) with cross-session state tracking
- Temp files use project-local `.tmp/` directory
- All file paths relative to user's working directory, not plugin folder
