---
name: application-tracker
description: Track job applications and STAR interview stories using Notion, Obsidian, or local files. Other skills reference this automatically when configured.
---

# Application Tracker

## Overview

Track job applications and STAR interview stories with your preferred backend. When configured, other skills (job-match, resume-tailor, star-prep) automatically use the tracker for storage and duplicate detection.

Supported backends:
- **Notion** — databases via Notion MCP server
- **Obsidian** — markdown notes via Obsidian MCP server
- **Local files** — markdown files in the project directory (no setup required)

## Important: Working Directory

All file paths in this skill (e.g., `applications/`, `star-stories/`) are relative to the **user's current working directory** — NOT the plugin installation directory. Never read from or write to the plugin's own folder.

## Setup Flow

The setup flow spans up to two sessions when MCP installation is needed (requires a restart). CLAUDE.md tracks state between sessions via the `## Application Tracking` section.

### Phase 1: Detect Existing MCP Servers

1. Read the project `CLAUDE.md` and check for `Backend:` and `Status:` in the `## Application Tracking` section:
   - If `Status: configured` → backend is ready, skip setup entirely
   - If `Status: MCP installed, pending restart` → check if MCP tools are now available (Phase 3)
   - If `Backend:` exists but no status → proceed to Phase 3 for that backend
   - If no `Backend:` line exists (or no `## Application Tracking` section) → continue detection below

2. If the `ToolSearch` tool is available, use it to discover MCP tools by keyword — this catches all installation variants (Docker, npm, Claude Code plugin, community packages):
   - Run `ToolSearch` with query `"notion"` — look at returned tool names
   - Run `ToolSearch` with query `"obsidian"` — look at returned tool names

3. If `ToolSearch` is unavailable or returns no results for both, no MCP servers are installed.

4. If tools are found, identify the variant from the tool names returned and extract the full prefix (everything before the distinguishing suffix, e.g., `mcp__MCP_DOCKER__` or `mcp__notion__`):

**Notion variants:**

| Variant | Indicator (tool name contains) | Key tools (prefix + suffix) |
|---------|-------------------------------|----------------------------|
| Notion API (Docker/npm) | `API-post-page` | `[prefix]API-query-data-source`, `[prefix]API-post-page`, `[prefix]API-patch-page` |
| Notion Plugin (makenotion) | `create_page` | `[prefix]search`, `[prefix]create_page`, `[prefix]update_page`, `[prefix]query_database` |
| Notion Community (suekou) | `notion_query_database` | `[prefix]notion_search`, `[prefix]notion_create_page`, `[prefix]notion_update_page` |

**Obsidian variants:**

| Variant | Indicator (tool name contains) | Key tools (prefix + suffix) |
|---------|-------------------------------|----------------------------|
| Standard (Docker/MarkusPfundstein w/ prefix) | `obsidian_list_files` | `[prefix]obsidian_simple_search`, `[prefix]obsidian_append_content`, `[prefix]obsidian_patch_content` |
| Bare (MarkusPfundstein direct) | `list_files_in_vault` without `obsidian_` prefix | `[prefix]search`, `[prefix]append_content`, `[prefix]patch_content` |
| Plugin (iansinnott) | `view` or `str_replace` | `[prefix]view`, `[prefix]create`, `[prefix]get_workspace_files` |

5. Store the discovered prefix and variant for use in all subsequent tool calls.
6. If MCP tools were found → skip to "Ask the User (MCP detected)" below
7. If no MCP tools found → continue to "Ask the User (no MCP detected)"

### Ask the User

**MCP detected** — ask which backend to use:

- If both Notion and Obsidian detected:
  "I detected both Notion and Obsidian MCP servers. Which would you like to use for tracking applications and STAR stories?"
  - Notion
  - Obsidian
  - Local files (no MCP needed)

- If only one detected:
  "I detected [Notion/Obsidian]. Would you like to use it for tracking, or prefer local files?"
  - [Notion/Obsidian]
  - Local files

If the user picks a detected backend → skip to Phase 3.

**No MCP detected** — ask what they want:

"No Notion or Obsidian MCP servers detected. I can set one up for you automatically, or you can use local markdown files."
- Notion (I'll install it for you)
- Obsidian (I'll install it for you)
- Local files (no setup needed)

If the user picks Notion or Obsidian → continue to Phase 2.
If the user picks Local files → skip to Phase 3.

### Phase 2: Install MCP Server (requires restart)

> **Note:** MCP servers are usually installed during `/onboard`. This phase is a fallback for users who skipped onboarding or want to switch backends.

**Notion:**
1. Run via Bash: `claude mcp add --transport http notion https://mcp.notion.com/mcp`
2. Update the `## Application Tracking` section of CLAUDE.md:
   ```markdown
   ## Application Tracking (Optional)

   Backend: Notion
   Status: MCP installed, pending restart
   ```
3. Tell the user:
   > All set! Restart Claude Code, then run `/mcp` to authenticate with Notion. After that, run `/application-tracker` to complete configuration.

**Obsidian:**
1. Ask the user for their Obsidian vault path (this is just a filesystem path, not sensitive)
2. Validate the path exists via Bash: `test -d "/path/to/vault" && echo "OK" || echo "NOT FOUND"`
3. If the path doesn't exist, ask the user to correct it
4. Run via Bash: `claude mcp add obsidian -- npx -y obsidian-mcp /path/to/vault`
5. Update the `## Application Tracking` section of CLAUDE.md:
   ```markdown
   ## Application Tracking (Optional)

   Backend: Obsidian
   Status: MCP installed, pending restart
   ```
6. Tell the user:
   > All set! Restart Claude Code, then run `/application-tracker` to complete configuration.

**After running `claude mcp add`**, STOP. Do not attempt to use MCP tools in this session — they won't be available until after the restart.

### Phase 3: Configure Backend (MCP available)

This phase runs when MCP tools are available (either detected in Phase 1, or after a restart following Phase 2).

Follow the backend-specific setup instructions below (Notion Backend, Obsidian Backend, or Local Files Backend).

After successful configuration, update CLAUDE.md to mark setup as complete:
```markdown
Status: configured
```

---

## Notion Backend

### Prerequisites

- Notion MCP server connected to Claude Code (installed automatically in Phase 2, or pre-existing)
- OAuth authentication completed (via `/mcp` after install)
- Two Notion databases must be created (see schemas below)

### Setup

#### 1. Create the Applications Database

Create a new Notion database with these properties:

| Property | Type | Description |
|----------|------|-------------|
| Company | Title | Company name |
| Role | Rich text | Job title/role |
| Job Posting | URL | Link to job posting |
| Stage | Status | Application stage (e.g., Applied, Interview, Offer, Rejected) |
| Last update | Date | Last activity date |

#### 2. Create the STAR Stories Database

Create a new Notion database with these properties:

| Property | Type | Description |
|----------|------|-------------|
| Name | Title | Descriptive story title |
| Situation | Rich text | Context and background |
| Task | Rich text | Your responsibility |
| Action | Rich text | Steps you took |
| Result | Rich text | Outcomes and metrics |

#### 3. Get Database IDs

1. Open each database in Notion
2. Copy the URL — it looks like one of these:
   - `https://www.notion.so/[database-id]?v=...` (current format)
   - `https://www.notion.so/[workspace]/[database-id]?v=...` (older format with workspace slug)
3. The database ID is the 32-character hex string before `?v=` (add hyphens: `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`)

#### 4. Save Database IDs to CLAUDE.md

Add the database IDs to the Application Tracking section of CLAUDE.md so skills can find them, and update the status to `configured`:

```markdown
## Application Tracking (Optional)

Backend: Notion
Status: configured

NOTION_APPLICATIONS_DB_ID=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
NOTION_STAR_STORIES_DB_ID=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
```

### API Patterns

Use the tool names discovered in the "Detect Available Backends" section. The JSON payloads below are the same across all Notion variants — only the tool name changes:

| Operation | Notion API (Docker/npm) | Notion Plugin (makenotion) | Notion Community (suekou) |
|-----------|------------------------|---------------------------|--------------------------|
| Query DB | `[prefix]API-query-data-source` | `[prefix]query_database` | `[prefix]notion_query_database` |
| Create page | `[prefix]API-post-page` | `[prefix]create_page` | `[prefix]notion_create_page` |
| Update page | `[prefix]API-patch-page` | `[prefix]update_page` | `[prefix]notion_update_page` |

#### Query a Database (e.g., check for duplicates)

```json
{
  "database_id": "<APPLICATIONS_DB_ID>",
  "filter": {
    "and": [
      {"property": "Company", "title": {"equals": "[COMPANY]"}},
      {"property": "Role", "rich_text": {"equals": "[ROLE]"}}
    ]
  }
}
```

#### Create a Page (e.g., add application)

```json
{
  "parent": {"database_id": "<APPLICATIONS_DB_ID>"},
  "properties": {
    "Company": {"title": [{"text": {"content": "[COMPANY]"}}]},
    "Role": {"rich_text": [{"text": {"content": "[ROLE]"}}]},
    "Job Posting": {"url": "[URL]"},
    "Stage": {"status": {"name": "Applied"}}
  }
}
```

#### Update a Page (e.g., improve STAR entry)

```json
{
  "page_id": "<PAGE_ID>",
  "properties": {
    "Action": {"rich_text": [{"text": {"content": "[improved text]"}}]}
  }
}
```

#### Create a STAR Story

```json
{
  "parent": {"database_id": "<STAR_STORIES_DB_ID>"},
  "properties": {
    "Name": {"title": [{"text": {"content": "[title]"}}]},
    "Situation": {"rich_text": [{"text": {"content": "[situation]"}}]},
    "Task": {"rich_text": [{"text": {"content": "[task]"}}]},
    "Action": {"rich_text": [{"text": {"content": "[action]"}}]},
    "Result": {"rich_text": [{"text": {"content": "[result]"}}]}
  }
}
```

### Validation

After setup, verify access by querying each database with an empty filter:

```json
{
  "database_id": "<APPLICATIONS_DB_ID>",
  "filter": {}
}
```

If the query succeeds (even with 0 results), the database is correctly configured. Repeat for the STAR Stories database. If either query fails, check database IDs and integration permissions.

### Troubleshooting

- **"Notion tools not found"**: Ensure your Notion MCP server is running and connected
- **"Database not found"**: Verify database IDs are correct and the integration has access
- **"Permission denied"**: Share the Notion databases with your integration
- **API errors**: Check that property names match exactly (case-sensitive)

---

## Obsidian Backend

### Prerequisites

- Obsidian MCP server connected to Claude Code (installed automatically in Phase 2, or pre-existing)
- Obsidian vault must be accessible
- **Dataview** community plugin must be installed and enabled (for database-style table views in the Overview dashboard)

### Setup

#### 0. Verify Dataview Plugin

Before creating the dashboard, check if Dataview is installed by asking the user:

"The Overview dashboard uses Dataview for table views. Is the Dataview community plugin installed and enabled in your Obsidian vault?"

- If **yes** → continue to step 1
- If **no** → guide them: "Install Dataview from Obsidian → Settings → Community Plugins → Browse → search 'Dataview' → Install → Enable. Let me know when it's ready."
- If **unsure** → "You can check in Obsidian → Settings → Community Plugins. If Dataview isn't listed there, install it from Browse."

Do not proceed until the user confirms Dataview is available.

#### 1. Create Folder Structure with Overview Dashboard

Create the following structure in your Obsidian vault. The **Overview** note is a single dashboard with Dataview tables for both Applications and STAR Stories — one click to see everything.

```
Job Search/
  Overview.md          ← dashboard with both Dataview tables
  Applications/
    [Company] - [Role].md
  STAR Stories/
    [Story Name].md
```

**Important:** `Overview.md` should appear first in the sidebar, above the `Applications/` and `STAR Stories/` folders. If Obsidian sorts folders before files, the user can change this in Settings → Files and Links → sort order, or use the **Folder Notes** plugin to make clicking "Job Search" open `Overview.md` directly.

Create the **Overview dashboard**:

```
[prefix]obsidian_append_content:
  filepath: "Job Search/Overview.md"
  content: |
    ---
    cssclasses:
      - database-view
    ---

    ## Applications

    ```dataview
    TABLE
      company AS "Company",
      role AS "Role",
      stage AS "Stage",
      url AS "Job Posting",
      last_update AS "Last Updated"
    FROM "Job Search/Applications"
    SORT last_update DESC
    ```

    ## STAR Stories

    ```dataview
    TABLE
      category AS "Category",
      name AS "Story",
      created AS "Created"
    FROM "Job Search/STAR Stories"
    SORT category ASC, created DESC
    ```
```

#### 2. Verify Access

Test that MCP tools can access the vault by listing the folders:

```
[prefix]obsidian_list_files_in_dir: dirpath "Job Search/Applications"
```

If the listing succeeds, the folders are ready. If it fails, re-run the folder creation step above.

#### 3. Update CLAUDE.md

Update the Application Tracking section of CLAUDE.md with the status:

```markdown
## Application Tracking (Optional)

Backend: Obsidian
Status: configured
```

### File Formats

#### Applications

One note per application in `Job Search/Applications/[Company] - [Role].md`:

```markdown
---
company: [Company Name]
role: [Role Title]
url: [Job Posting URL]
stage: Applied
last_update: [YYYY-MM-DD]
---

# [Company Name] - [Role Title]

**Job Posting:** [URL]
**Stage:** Applied
**Last Updated:** [YYYY-MM-DD]
```

#### STAR Stories

One note per story in `Job Search/STAR Stories/[Story Name].md`:

```markdown
---
name: [Descriptive title]
category: [competency category]
created: [YYYY-MM-DD]
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

### MCP Tool Patterns

Use the tool names discovered in the "Detect Available Backends" section. The examples below use `[prefix]` as a placeholder — replace with the actual discovered prefix (e.g., `mcp__MCP_DOCKER__`):

| Operation | Standard (w/ obsidian_ prefix) | Bare (MarkusPfundstein direct) | Plugin (iansinnott) |
|-----------|-------------------------------|-------------------------------|---------------------|
| Search | `[prefix]obsidian_simple_search` | `[prefix]search` | `[prefix]get_workspace_files` |
| Create/Append | `[prefix]obsidian_append_content` | `[prefix]append_content` | `[prefix]create` |
| Update/Patch | `[prefix]obsidian_patch_content` | `[prefix]patch_content` | `[prefix]str_replace` |
| List files | `[prefix]obsidian_list_files_in_dir` | `[prefix]list_files_in_vault` | `[prefix]get_workspace_files` |

#### Check for Duplicates

```
[prefix]obsidian_simple_search: query "[Company] - [Role]"
```

Check results for matching filenames in `Job Search/Applications/`.

#### Create Application

```
[prefix]obsidian_append_content:
  filepath: "Job Search/Applications/[Company] - [Role].md"
  content: [full note content with frontmatter]
```

#### Update Application Stage

```
[prefix]obsidian_patch_content:
  filepath: "Job Search/Applications/[Company] - [Role].md"
  target: "stage: [old-stage]"
  replacement: "stage: [new-stage]"
  operation: "replace"
  target_type: "string"
```

Also update the `last_update` field and the body text.

#### List Applications

```
[prefix]obsidian_list_files_in_dir: dirpath "Job Search/Applications"
```

#### Create STAR Story

```
[prefix]obsidian_append_content:
  filepath: "Job Search/STAR Stories/[Story Name].md"
  content: [full note content with frontmatter]
```

#### Update STAR Story

```
[prefix]obsidian_patch_content:
  filepath: "Job Search/STAR Stories/[Story Name].md"
  target: [section to update]
  replacement: [improved text]
  operation: "replace"
  target_type: "string"
```

### Troubleshooting

- **"Obsidian tools not found"**: Ensure the Obsidian MCP server is running and connected
- **"Folder not found"**: Create the `Job Search/Applications/` and `Job Search/STAR Stories/` folders in your vault
- **"File not found"**: Check that filenames match exactly (Obsidian is case-sensitive on some systems)

---

## Local Files Backend

### Overview

No setup required. Local files work out of the box and are the default when neither Notion nor Obsidian is configured. Data is stored as markdown files in the project directory.

On setup, create directories and update CLAUDE.md:

```bash
mkdir -p applications/
mkdir -p star-stories/
```

```markdown
## Application Tracking (Optional)

Backend: Local
Status: configured
```

### Storage

Verify the directories were created by listing them. If either is missing, re-run the `mkdir -p` commands.

**Slug rules:** lowercase, spaces to hyphens, strip characters not matching `[a-z0-9-]`, collapse consecutive hyphens. Example: "Acme Corp." → `acme-corp`, "Senior Software Engineer (Remote)" → `senior-software-engineer-remote`.

#### Applications

One file per application in `applications/[company-slug]-[role-slug].md`:

```markdown
---
company: [Company Name]
role: [Role Title]
url: [Job Posting URL]
stage: Applied
last_update: [YYYY-MM-DD]
---

# [Company Name] - [Role Title]

**Job Posting:** [URL]
**Stage:** Applied
**Last Updated:** [YYYY-MM-DD]
```

#### STAR Stories

One file per story in `star-stories/[slug].md`:

```markdown
---
name: [Descriptive title]
category: [competency category]
created: [YYYY-MM-DD]
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

### Operations

#### Check for Duplicate Applications

Use Glob to list files in `applications/` and check for `[company-slug]-[role-slug].md`. If found, read the file and check the `last_update` frontmatter field.

#### Create Application

Write a new markdown file to `applications/[company-slug]-[role-slug].md`.

#### Update Application Stage

Read the existing file, update the `stage` and `last_update` fields in both frontmatter and body, then write back.

#### List Applications

Use Glob to list all `applications/*.md` files.

#### Create/Update STAR Story

Write to `star-stories/[slug].md`. Read first if updating.

---

## How Other Skills Reference This

All skills that need tracker integration follow this exact algorithm:

1. Read the project `CLAUDE.md`
2. Look for `Backend: <value>` and `Status: <value>` in the `## Application Tracking` section:
   - If `Status: MCP installed, pending restart` → tell user to restart Claude Code and re-run `/application-tracker`
   - If `Status: configured` (or no status but Backend is set):
     - `Notion` → use Notion backend
     - `Obsidian` → use Obsidian backend
     - `Local` → use local files backend
3. If no explicit backend is set:
   - If the `ToolSearch` tool is available, run `ToolSearch` with query `"notion"` → if tools found, use Notion
   - If the `ToolSearch` tool is available, run `ToolSearch` with query `"obsidian"` → if tools found, use Obsidian
   - If `ToolSearch` is unavailable or neither found → use local files
4. For Notion: also read `NOTION_APPLICATIONS_DB_ID` and `NOTION_STAR_STORIES_DB_ID` from the `## Application Tracking` section of CLAUDE.md
5. Use tool discovery from the "Detect Available Backends" section above to determine the correct variant and tool name prefix (e.g., if ToolSearch returns `mcp__MCP_DOCKER__API-query-data-source`, the prefix is `mcp__MCP_DOCKER__`)

For duplicate detection (job-match) and application tracking (resume-tailor):
- **Notion**: Query the Applications database
- **Obsidian**: Search `Job Search/Applications/` for matching notes
- **Local**: Check `applications/` directory for matching files
