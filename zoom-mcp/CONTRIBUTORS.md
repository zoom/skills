# Contributor Guide — zoom-mcp Skill

This document lists everything that needs to be verified or completed before this skill
is ready to merge. Each item is also marked inline in the relevant file with
`[CONTRIBUTOR NEEDED]`.

**PR:** https://github.com/zoom/skills/compare/main...rossmayz:skills:add-zoom-mcp-skill

---

## How to Contribute

1. Open the PR above → click **Files changed** to review the full content
2. For each item below, add the answer as a PR comment (reference the file + line),
   or push a commit directly to the branch `add-zoom-mcp-skill` on the fork
3. Check off items as they're resolved

---

## Items Needed

### 1. Agentic Retrieval — Recap meeting field names
**Files:** `references/tools.md`, `examples/transcript-retrieval.md`, `SKILL.md`

The `search_meetings` tool uses AI Companion's Agentic Retrieval API and returns a
"Recap meeting" result type. What are the exact JSON field names for:

- [ ] The inline AI summary text (what is the field called?)
- [ ] The docs array (field name, and each entry's structure — title + URL field names)
- [ ] The recording reference (field name and structure)
- [ ] The whiteboards (field name and structure)

**Who knows:** Zoom MCP Server engineering team, or inspect a live server response.

---

### 2. Agentic Retrieval — View recording field names
**Files:** `references/tools.md`, `examples/transcript-retrieval.md`

The `search_meetings` tool also returns a "View recording" result type. What fields does
it contain? How does it differ from the Recap meeting result?

- [ ] Full field list for View recording result type

**Who knows:** Zoom MCP Server engineering team.

---

### 3. Zoom Doc tool — exact tool name
**Files:** `SKILL.md`, `references/tools.md`, `examples/create-zoom-doc.md`

The MCP Server has a tool to create Zoom Docs. We've used `create_zoom_doc` as a
placeholder. What is the real tool name?

- [ ] Confirm by running `zoom-mcp:list_available_tools` against the live server

**Who knows:** Anyone with a valid OAuth token can run `list_available_tools` to get
the authoritative tool list.

---

### 4. Zoom Doc tool — parameters
**Files:** `references/tools.md`, `examples/create-zoom-doc.md`

Once the tool name is confirmed, what are the full input parameters?

- [ ] Required parameters (title? content?)
- [ ] Optional parameters (owner, folder, template, permissions?)
- [ ] What does the response contain? (URL of created doc? Doc ID?)

**Who knows:** Run `zoom-mcp:get_tool_details` with the correct tool name, or ask the
engineering team.

---

### 5. Total tool count
**Files:** `references/tools.md`, `concepts/mcp-architecture.md`

The original documented tool list had 12 tools. The Zoom Doc creation tool may be
a 13th (or it may already be in the 12 under a different name).

- [ ] Confirm total tool count by running `zoom-mcp:list_available_tools`

---

### 6. Agentic Retrieval API — public documentation
**File:** `concepts/mcp-architecture.md`

Is there a public developer documentation page for the Agentic Retrieval API on
https://developers.zoom.us/docs/? If so, linking it would give contributors and users
an authoritative reference.

- [ ] URL of public Agentic Retrieval API docs (or confirm none exist yet)

**Who knows:** Zoom Developer Platform / AI Companion team.

---

### 7. OAuth scopes for Zoom Docs
**File:** `concepts/oauth-setup.md`

The current scope list covers meetings and recordings. Does creating or accessing Zoom
Docs require an additional OAuth scope?

- [ ] Scope required for Zoom Doc creation (if any)
- [ ] Scope required for reading Zoom Doc URLs from Agentic Retrieval results (if any)

**Who knows:** Zoom OAuth / Platform team.

---

## Items That Are Complete

The following are documented and do not need further verification:

- Server endpoints (Streamable HTTP + SSE URLs — confirmed live via curl)
- All 12 originally documented tools and their parameters
- OAuth setup flow (User-managed OAuth app)
- AI Companion prerequisite (Smart Recording + Meeting Summary)
- Error codes (-32001, -32602, HTTP 400/401/403/404/429)
- Meeting lifecycle workflows (create, update, delete, list)
- Recording file types (MP4, M4A, VTT, TRANSCRIPT, CHAT, TIMELINE)
- Transcript download pattern (Bearer token on download_url)
- RUNBOOK.md preflight checklist
- Troubleshooting guide
