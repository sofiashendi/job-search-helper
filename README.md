# Job Search Helper

AI-powered job search toolkit for Claude Code. Score job postings against your resume, generate tailored resumes, prepare for behavioral interviews, and optionally track everything in Notion, Obsidian, or local files.

## Prerequisites

- [Claude Code CLI](https://claude.ai/code)
- Node.js 18+ (used automatically for document conversion, no manual setup needed)
- macOS or Linux (Windows is not currently supported)
- Notion or Obsidian (optional; local file tracking works out of the box)
- [Dataview](https://github.com/blacksmithgu/obsidian-dataview) Obsidian plugin (required if using Obsidian tracking)

## Installation

### From the Plugin Marketplace

```bash
# Add the marketplace
/plugin marketplace add sofiashendi/job-search-helper

# Install the plugin
/plugin install job-search-toolkit@job-search-helper
```

You can also browse available plugins via `/plugin` > Discover tab.

### From GitHub

```bash
# Clone the repo
git clone https://github.com/sofiashendi/job-search-helper.git

# Load as a local plugin
claude --plugin-dir ./job-search-helper
```

Or for permanent local installation:

```bash
/plugin marketplace add ./job-search-helper
/plugin install job-search-toolkit@job-search-helper
```

## Setup

### 1. Import your resume

Run `/onboard` in Claude Code to import your resume from PDF, DOCX, or markdown. It will be converted to the expected format and saved to `resumes/`.

> **First-run note:** The first time you generate a PDF resume, `md-to-pdf` will download Chromium (~200MB). This is a one-time download and may take 1-2 minutes.

### 2. Set up application tracking (optional)

Run `/application-tracker` in Claude Code. The skill will detect available integrations (Notion, Obsidian) and guide you through setup. If neither is available, it defaults to local file tracking — no configuration needed.

## Workflow

```bash
claude

# 0. (First time) Import your resume
> /onboard

# 1. Evaluate a job posting
> /job-match https://example.com/jobs/posting-url

# 2. If Apply/Consider, generate tailored resume
> /resume-tailor

# 3. When you land an interview, prep STAR stories
> /star-prep
```

## Available Skills

| Skill | Description |
|-------|-------------|
| `/onboard` | Import resumes from PDF, DOCX, or markdown into the expected format |
| `/job-match` | Score job postings against your resume (weighted scoring, gap analysis) |
| `/resume-tailor` | Generate tailored resumes with PDF conversion (conservative reframing, never fabricates) |
| `/star-prep` | Interview prep when you've landed an interview — creates and maps STAR stories to role competencies |
| `/application-tracker` | Track applications and STAR stories (Notion, Obsidian, or local files) |

## Scoring System

| Score | Rating | Recommendation |
|-------|--------|----------------|
| 80%+ | Strong Match | Apply |
| 65-79% | Good Match | Consider |
| <65% | Weak Match | Skip |

Weights: Required Skills (50%), Experience (30%), Nice-to-haves (20%)

## Dependencies (auto-installed)

These packages are installed automatically on first use — no manual setup needed:

- `mammoth` — DOCX resume import
- `md-to-pdf` — PDF generation

## Directory Structure

```
.claude-plugin/
  marketplace.json          # Plugin marketplace definition
skills/                           # Skill definitions
  onboard/
  job-match/
  resume-tailor/
  star-prep/
  application-tracker/
resumes/            # Your base resume(s) go here
star-method-guide.md        # STAR method reference guide
applications/               # Local application tracking (created on use)
resumes/                    # Generated tailored resumes (created on use)
star-stories/               # Local STAR stories (created on use)
```

## Privacy

Resumes and generated files are stored locally on your machine. Nothing is sent to external servers unless you configure optional Notion or Obsidian integration. Tailored resumes are saved to `resumes/[company]/`.

## Contributing

This project does not accept external contributions.

## License

MIT
