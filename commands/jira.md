# Jira: Manage Jira tickets

Interact with Jira tickets using the Jira MCP server (`@aashari/mcp-server-atlassian-jira`).

## Configuration

- **Jira instance**: `ambcba.atlassian.net`
- **Project prefix**: `PNL` (for iOS payment/mobile team tickets)
- **Credentials**: stored in `~/.claude.json` → `mcpServers.jira.env`
  - `ATLASSIAN_JIRA_BASE_URL`: `https://ambcba.atlassian.net`
  - `ATLASSIAN_JIRA_EMAIL`: `duy.le1@tyme.com`
  - `ATLASSIAN_JIRA_API_TOKEN`: Jira API token (generate at https://id.atlassian.com/manage-profile/security/api-tokens)

## Available Operations

### Get ticket details
Use MCP `jira` tools to fetch issue details:
- Search issues: `jira_search_issues` with JQL query
- Get issue: `jira_get_issue` with issue key (e.g., `PNL-31765`)

### Update ticket description
Use MCP `jira` tools to update issue fields. The description uses **Atlassian Document Format (ADF)**.

To update a ticket description with markdown content:
1. Convert the markdown to ADF format (see ADF conversion rules below)
2. Use the Jira MCP `put` endpoint: `/rest/api/3/issue/{issueKey}` with the ADF body

### ADF Conversion Rules (Markdown to Atlassian Document Format)

ADF is a JSON format. Key mappings:

| Markdown | ADF node type |
|----------|--------------|
| `# Heading` | `heading` (attrs: level 1-6) |
| `## Heading` | `heading` (attrs: level 2) |
| Paragraph text | `paragraph` with `text` nodes |
| `**bold**` | text mark: `strong` |
| `*italic*` | text mark: `em` |
| `` `code` `` | text mark: `code` |
| `- item` | `bulletList` → `listItem` → `paragraph` |
| `1. item` | `orderedList` → `listItem` → `paragraph` |
| `\| table \|` | `table` → `tableRow` → `tableHeader`/`tableCell` |
| ``` code block ``` | `codeBlock` (attrs: language) |
| `---` | `rule` |
| `[link](url)` | text mark: `link` (attrs: href) |

Example ADF structure:
```json
{
  "version": 1,
  "type": "doc",
  "content": [
    {
      "type": "heading",
      "attrs": { "level": 2 },
      "content": [{ "type": "text", "text": "Summary" }]
    },
    {
      "type": "paragraph",
      "content": [
        { "type": "text", "text": "Normal text " },
        { "type": "text", "text": "bold text", "marks": [{ "type": "strong" }] }
      ]
    },
    {
      "type": "bulletList",
      "content": [
        {
          "type": "listItem",
          "content": [
            {
              "type": "paragraph",
              "content": [{ "type": "text", "text": "First item" }]
            }
          ]
        }
      ]
    }
  ]
}
```

### Fallback: Direct REST API via curl

If MCP tools are unavailable, use curl directly:

```bash
# Get issue
curl -s -u "$JIRA_EMAIL:$JIRA_API_TOKEN" \
  "https://ambcba.atlassian.net/rest/api/3/issue/PNL-31765"

# Update description (ADF format)
curl -s -X PUT \
  -u "$JIRA_EMAIL:$JIRA_API_TOKEN" \
  -H "Content-Type: application/json" \
  "https://ambcba.atlassian.net/rest/api/3/issue/PNL-31765" \
  -d '{"fields":{"description":<ADF_JSON>}}'
```

Read credentials from `~/.claude.json`:
```python
import json
with open(os.path.expanduser("~/.claude.json")) as f:
    config = json.load(f)
jira_env = config["mcpServers"]["jira"]["env"]
JIRA_URL = jira_env["ATLASSIAN_JIRA_BASE_URL"]
JIRA_EMAIL = jira_env["ATLASSIAN_JIRA_EMAIL"]
JIRA_TOKEN = jira_env["ATLASSIAN_JIRA_API_TOKEN"]
```

## Usage Examples

When user says `/jira update PNL-31765 description`:
1. Read the ticket description content (from conversation or file)
2. Convert markdown to ADF
3. PUT to `/rest/api/3/issue/PNL-31765` with ADF body
4. Confirm update with link: `https://ambcba.atlassian.net/browse/PNL-31765`

When user says `/jira get PNL-31765`:
1. Fetch issue details via MCP or REST API
2. Display summary, status, description, assignee

## Safety
- **NEVER** print or log the Jira API token value
- Always read token from config, never hardcode
- Only modify fields explicitly requested by the user
