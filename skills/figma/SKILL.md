---
name: figma
description: |
  Read and extract information from Figma design files using the Figma Desktop MCP.
  Supports reading specific nodes, listing sections, and extracting UI component details.
  Triggers when user says "figma", "read figma", "figma design", or pastes a Figma URL.
---

# Figma Design Reader

$ARGUMENTS

## Prerequisites

- Figma Desktop app must be running
- Figma Desktop MCP is configured at `http://127.0.0.1:3845/mcp` (no API key needed)
- **Figma Dev Mode is required** — your Figma account must have Dev Mode access enabled. If you don't have Dev Mode access, please contact **Vuong Tran** to get it set up.

## Read Flow

When the user runs `/figma read <FIGMA_URL>` or `/figma <FIGMA_URL>`:

1. **Parse the Figma URL** to extract the file ID and node ID:
   - URL format: `https://www.figma.com/design/<FILE_ID>/<FILE_NAME>?node-id=<NODE_ID>&...`
   - The `<FILE_ID>` is the first path segment after `/design/`
   - The `<NODE_ID>` is from the `node-id` query parameter (e.g., `8139-101827`)
   - Convert the node ID from hyphen format to colon format: `8139-101827` → `8139:101827`
   - If the URL is split across multiple lines, concatenate them first

2. **Use the Figma Desktop MCP tools** to fetch the node data

3. **Summarize the screen** by extracting only visible, meaningful content:
   - Screen title and heading text
   - Visible list items with their titles, subtitles, and icons
   - Button labels and states
   - Navigation elements (visible only)
   - Ignore hidden elements and placeholder/template items

## List Versions Flow

When the user runs `/figma versions <FIGMA_URL>`:

1. Parse the URL to extract the file ID
2. Use the Figma Desktop MCP to fetch versions
3. Display the versions in a table with date, author, and label

## Examples

Read a specific node:
```
/figma read https://www.figma.com/design/RJB1rPsPtJKj00PLq7XiDH/06---UPF--UI-2.0----MASTER?node-id=8139-101827
```

Short form:
```
/figma https://www.figma.com/design/RJB1rPsPtJKj00PLq7XiDH/06---UPF--UI-2.0----MASTER?node-id=8139-101827
```

List versions:
```
/figma versions https://www.figma.com/design/RJB1rPsPtJKj00PLq7XiDH/06---UPF--UI-2.0----MASTER
```
