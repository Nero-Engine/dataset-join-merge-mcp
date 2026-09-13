# Dataset Join & Merge (Remote MCP Server)

**Join two lists of JSON rows on a shared key, like a SQL join or a spreadsheet VLOOKUP, in a single tool call.** Hand it a main list and a lookup list, name the key, pick a join type, and it hands back one combined list plus an exact account of how many rows matched on each side and why the rest did not.

Built for AI agents. No install, no API key, no signup. Connect by URL and call it.

```
https://dataset-join-merge.nerolabs.workers.dev/mcp
```

Free to use while in early access.

## What it does

- **Seven join types.** `left` keeps every left row and adds the matching right fields (the VLOOKUP and enrichment case, and the default). `inner` keeps only the overlap. `right` keeps every right row. `full` keeps everything from both sides. `leftAnti` returns left rows with **no** match on the right ("which of these leads are not in the CRM yet?"). `rightAnti` is the reverse. `union` stacks both lists, no key needed.
- **Keys that match the way real data looks.** By default keys match case-insensitively with surrounding and repeated spaces ignored, and `123` matches `"123"`, so `" Ana@Example.com"` from a scraper finds `"ana@example.com"` in a billing export. Set `keyMatching` to `exact` for strict matching.
- **Different key names on each side.** `email` on the left, `contact_email` on the right. Composite keys work too: `["firstName", "lastName"]`.
- **Control over what comes across.** Copy only chosen right fields, and decide what happens when both sides have a field with the same name: keep both (the right one becomes `right_status`), keep the left value, or overwrite with the right value.
- **SQL or VLOOKUP behaviour for duplicate keys.** `multipleMatches: "all"` returns one row per matching pair; `"first"` uses only the first matching right row, so each left row appears once.

It is honest about what it could not match. Every row is tagged `_joinStatus` (`matched`, `left_only`, `right_only`) and `_matchCount`, and the summary reports match rates plus warnings for rows missing the key, duplicate keys on the right and key fields that exist on no row at all (usually a typo).

## Tools

| Tool | What it does |
|---|---|
| `list_capabilities` | Lists every join type, matching mode, conflict strategy and duplicate-key mode, the fields added to each row, and the row limits. Processes no data. |
| `join_rows` | Joins `leftRows` and `rightRows` on the key fields you name and returns the joined rows plus a summary. |

## Connect

**Claude Code**

```bash
claude mcp add --transport http dataset-join-merge https://dataset-join-merge.nerolabs.workers.dev/mcp
```

**Claude Desktop / claude.ai:** Settings, Connectors, Add custom connector, paste the URL above.

**Cursor, Windsurf, VS Code and other MCP clients**

```json
{
  "mcpServers": {
    "dataset-join-merge": {
      "url": "https://dataset-join-merge.nerolabs.workers.dev/mcp"
    }
  }
}
```

## Example

Four scraped leads on the left, three billing records on the right. The emails differ in case and spacing, and the key has a different name on each side:

```json
{
  "leftRows": [
    {"email": "Ana@Example.com ", "name": "Ana Silva", "status": "lead"},
    {"email": "ben@example.com", "name": "Ben Okafor", "status": "lead"},
    {"email": "cara@example.com", "name": "Cara Lind", "status": "lead"},
    {"email": "dev@example.com", "name": "Dev Patel", "status": "lead"}
  ],
  "rightRows": [
    {"contact_email": "ana@example.com", "plan": "Pro", "mrr": 49, "status": "active"},
    {"contact_email": "BEN@EXAMPLE.COM  ", "plan": "Starter", "mrr": 19, "status": "trial"},
    {"contact_email": "erin@example.com", "plan": "Enterprise", "mrr": 499, "status": "active"}
  ],
  "leftKeyFields": ["email"],
  "rightKeyFields": ["contact_email"],
  "joinType": "left"
}
```

Every lead comes back, the two customers enriched with their plan, and both `status` fields kept:

```json
{
  "rows": [
    {"email": "Ana@Example.com ", "name": "Ana Silva", "status": "lead", "plan": "Pro", "mrr": 49, "right_status": "active", "_joinStatus": "matched", "_matchCount": 1},
    {"email": "ben@example.com", "name": "Ben Okafor", "status": "lead", "plan": "Starter", "mrr": 19, "right_status": "trial", "_joinStatus": "matched", "_matchCount": 1},
    {"email": "cara@example.com", "name": "Cara Lind", "status": "lead", "_joinStatus": "left_only", "_matchCount": 0},
    {"email": "dev@example.com", "name": "Dev Patel", "status": "lead", "_joinStatus": "left_only", "_matchCount": 0}
  ],
  "summary": {
    "leftRecordCount": 4,
    "rightRecordCount": 3,
    "joinType": "left",
    "keyMatching": "normalized",
    "multipleMatches": "all",
    "leftKeyFields": ["email"],
    "rightKeyFields": ["contact_email"],
    "counts": {"matchedRows": 2, "passthroughRows": 2, "leftRowsMatched": 2, "rightRowsMatched": 2, "leftRowsUnmatched": 2, "rightRowsUnmatched": 1},
    "leftMatchRate": 0.5,
    "rightMatchRate": 0.667,
    "outputRowCount": 4,
    "warnings": []
  }
}
```

Change `joinType` to `leftAnti` and the same call returns only Cara and Dev, the leads with no billing record.

## Limits

- Up to **500 rows per call**, counting `leftRows` and `rightRows` together. For a left, inner or leftAnti join on bigger lists, split `leftRows` across several calls and send the same `rightRows` with each.
- Up to **500 output rows per call**. Only a many-to-many join (the same key repeated on both sides, with `multipleMatches` set to `all`) can return more rows than went in. That call is refused with the predicted row count and what to change, never cut short: `multipleMatches: "first"` always fits.

Anything over a limit returns a clear message rather than failing silently.

## Privacy

Your rows are processed in memory and never stored. To see which tools get used, each call records the tool name, row counts, whether it succeeded, the client name your app reports, the country and a one-way hashed caller ID. Your data, your arguments and your IP address are never kept in that log.

## Also available

The same engine runs on the Apify Store as [Dataset Join & Merge](https://apify.com/nerolabs/dataset-join-merge), which also reads Apify datasets, CSV, Excel and JSON files and Google Sheets by URL on either side, handles up to 100,000 rows per side, exports the result as CSV or Excel, appends it to a named dataset that builds up across runs, and can POST it to a webhook.

Built by **Nero Labs**.
