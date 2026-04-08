# Confluence Reader

Read Confluence pages and search for content using the Atlassian MCP tools.

## Site Configuration

- **Cloud ID**: `ambcba.atlassian.net`

## Commands

### Read a page

```
/confluence read <URL_OR_PAGE_ID>
```

Accepts either:
- A full Confluence URL: `https://ambcba.atlassian.net/wiki/spaces/GOT/pages/4284743994/Page+Title`
- A page ID: `4284743994`

### Search for pages

```
/confluence search <query>
```

Searches Confluence using natural language via Rovo Search.

### Search with CQL

```
/confluence cql <CQL_QUERY>
```

Searches using Confluence Query Language. Examples:
- `title ~ "account number" AND space = GOT`
- `label = "testing-data" AND type = page`
- `ancestor = 3389850185 AND title ~ "mocking"`

### List child pages

```
/confluence children <URL_OR_PAGE_ID>
```

Lists all child/descendant pages under a given page.

## Execution Flow

### Parsing URLs

When given a Confluence URL, extract the page ID from the path. The URL format is:
```
https://<site>.atlassian.net/wiki/spaces/<SPACE_KEY>/pages/<PAGE_ID>/Optional+Title
```

Extract `<PAGE_ID>` (the numeric segment after `/pages/`).

If the input is purely numeric, treat it as a page ID directly.

### Reading a Page

1. Parse the URL or page ID from the user's input.
2. Call `getConfluencePage` with:
   - `cloudId`: `ambcba.atlassian.net`
   - `pageId`: the extracted page ID
   - `contentFormat`: `markdown`
3. Present the page content to the user in a readable format:
   - Show the page **title** as a heading
   - Show the page **body** content
   - If the content contains tables with test data (account numbers, bank codes, error codes, etc.), format them clearly
   - Note the page version and last updated date

### Searching

For **natural language search** (`/confluence search`):
1. Call `searchAtlassian` with the user's query.
2. Present results as a list with title, space, and link.

For **CQL search** (`/confluence cql`):
1. Call `searchConfluenceUsingCql` with:
   - `cloudId`: `ambcba.atlassian.net`
   - `cql`: the user's CQL query
   - `limit`: 10
2. Present results as a list with title, space, and link.

### Listing Children

1. Parse the URL or page ID.
2. Call `getConfluencePageDescendants` with:
   - `cloudId`: `ambcba.atlassian.net`
   - `pageId`: the extracted page ID
3. Present the child pages as a tree/list with titles and IDs.

## Integration with Write-Automation

When the `/write-automation` skill needs test data (bank accounts, error codes, mocking configurations), it can use this skill to fetch the relevant Confluence page. Common data pages:

| Data Type | How to Find |
|---|---|
| Account numbers for mocking | `/confluence search "range account number for testing"` |
| Bank/transfer error codes | `/confluence search "error code" AND space = GOT` |
| Test environment configs | `/confluence search "DEV SIT mocking"` |
| Team-specific test data | `/confluence search "<team_name> testing data"` |

To use in automation context: read the Confluence page, extract the relevant data (account numbers, error codes, etc.), and use it in test case implementation.

## Known Spaces

| Space Key | Description |
|---|---|
| GOT | GoTyme main space |
| CD | Continuous Delivery |
| STXL | SLSA / Sanlam |

## Safety

- **Read-only**: This skill only reads Confluence pages. It never creates, updates, or deletes content.
- Do not expose sensitive data (credentials, tokens, PII) from Confluence pages — summarize or redact as needed.
