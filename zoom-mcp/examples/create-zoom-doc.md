# Experimental: Create a Zoom Doc

A Zoom Doc is Zoom's native document format — comparable to a Notion page. The Zoom MCP
Server can create Zoom Docs directly, making it easy to turn meeting content into a
persistent, shareable document without leaving the workflow.

This page is quarantined on purpose. Do not route here automatically until live MCP tool
discovery confirms the tool name and parameter schema.

**Primary use cases:**
- Generate action item lists from a meeting AI summary
- Create structured meeting notes as a Zoom Doc
- Capture decisions and next steps in a format the team can collaborate on

---

## Workflow: Search Meeting → Create Zoom Doc

### Step 1: Find the meeting and get the AI summary

```
zoom-mcp:search_meetings
  query: "Q1 planning"
  startDate: "2025-02-01"
  endDate: "2025-02-28"
```

The Recap meeting result includes the inline AI summary. Use that content as the source
for the Zoom Doc.

### Step 2: Create the Zoom Doc

> **[CONTRIBUTOR NEEDED]** Verify the exact tool name and parameter schema. The tool name
> `create_zoom_doc` below is a placeholder. Run `zoom-mcp:list_available_tools` to find
> the correct name, or confirm with the Zoom MCP Server engineering team. Parameters
> needed: title, body/content, and any optional fields (owner, folder, permissions).

```
zoom-mcp:create_zoom_doc  ← verify tool name
  title: "Q1 Planning — Action Items"
  content: "[action items extracted from AI summary]"
```

The tool returns the **URL of the created Zoom Doc**, which can be shared with the team
or opened directly in Zoom.

---

## Zoom Doc in Search Results

When a Zoom Doc was attached to a meeting, the Agentic Retrieval result includes it. The
Recap meeting result contains a docs array with the URL for each attached Zoom Doc — no
separate fetch required. This means Claude can surface relevant documents directly from
a search result.

---

## Example Prompts

**After a meeting:**
> "Create a Zoom Doc with the action items from today's all-hands meeting."

**From a search:**
> "Find the Q1 planning session and create a Zoom Doc summarizing the key decisions."

**Structured notes:**
> "Get the AI summary from last Tuesday's product review and create a Zoom Doc with
> sections for Decisions, Action Items, and Open Questions."

---

> **[CONTRIBUTOR NEEDED]** Complete example showing the full request and response for
> `create_zoom_doc` (or the correct tool name), including all available parameters and
> what the returned Zoom Doc URL looks like. Source: Zoom MCP Server engineering team
> or live tool testing.
