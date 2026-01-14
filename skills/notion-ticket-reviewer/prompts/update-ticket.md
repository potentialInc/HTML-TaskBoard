# Update Notion Tickets via Curl

## Prompt Template

```
Update ticket [PAGE_ID] status to [STATUS]
```

Or:

```
Add comment to ticket [PAGE_ID]: [COMMENT_TEXT]
```

## Description

Update ticket status and add comments to Notion Bug Report tickets using curl commands. Works independently of the MCP server.

## Prerequisites

1. **NOTION_API_KEY** must be set in `.env` file or environment variable
2. **Page ID** of the ticket to update

## Update Status

### Set to "Fixing"

```bash
curl -s -X PATCH "https://api.notion.com/v1/pages/[PAGE_ID]" \
  -H "Authorization: Bearer $NOTION_API_KEY" \
  -H "Notion-Version: 2022-06-28" \
  -H "Content-Type: application/json" \
  -d '{
    "properties": {
      "Status": {
        "status": {
          "name": "Fixing"
        }
      }
    }
  }'
```

### Set to "Resolved"

```bash
curl -s -X PATCH "https://api.notion.com/v1/pages/[PAGE_ID]" \
  -H "Authorization: Bearer $NOTION_API_KEY" \
  -H "Notion-Version: 2022-06-28" \
  -H "Content-Type: application/json" \
  -d '{
    "properties": {
      "Status": {
        "status": {
          "name": "Resolved"
        }
      }
    }
  }'
```

### Set to "Closed"

```bash
curl -s -X PATCH "https://api.notion.com/v1/pages/[PAGE_ID]" \
  -H "Authorization: Bearer $NOTION_API_KEY" \
  -H "Notion-Version: 2022-06-28" \
  -H "Content-Type: application/json" \
  -d '{
    "properties": {
      "Status": {
        "status": {
          "name": "Closed"
        }
      }
    }
  }'
```

### Set to "Won't Fix"

```bash
curl -s -X PATCH "https://api.notion.com/v1/pages/[PAGE_ID]" \
  -H "Authorization: Bearer $NOTION_API_KEY" \
  -H "Notion-Version: 2022-06-28" \
  -H "Content-Type: application/json" \
  -d '{
    "properties": {
      "Status": {
        "status": {
          "name": "Won'\''t Fix"
        }
      }
    }
  }'
```

## Add Dev's Comment

### Update Dev's Comment Property

```bash
curl -s -X PATCH "https://api.notion.com/v1/pages/[PAGE_ID]" \
  -H "Authorization: Bearer $NOTION_API_KEY" \
  -H "Notion-Version: 2022-06-28" \
  -H "Content-Type: application/json" \
  -d '{
    "properties": {
      "Dev'\''s Comment": {
        "rich_text": [
          {
            "text": {
              "content": "Your developer comment here"
            }
          }
        ]
      }
    }
  }'
```

### Add Page Comment (Discussion)

```bash
curl -s -X POST "https://api.notion.com/v1/comments" \
  -H "Authorization: Bearer $NOTION_API_KEY" \
  -H "Notion-Version: 2022-06-28" \
  -H "Content-Type: application/json" \
  -d '{
    "parent": {
      "page_id": "[PAGE_ID]"
    },
    "rich_text": [
      {
        "text": {
          "content": "Your comment text here"
        }
      }
    ]
  }'
```

### Implementation Complete Comment

```bash
curl -s -X POST "https://api.notion.com/v1/comments" \
  -H "Authorization: Bearer $NOTION_API_KEY" \
  -H "Notion-Version: 2022-06-28" \
  -H "Content-Type: application/json" \
  -d '{
    "parent": {
      "page_id": "[PAGE_ID]"
    },
    "rich_text": [
      {
        "text": {
          "content": "Fix implemented.\n\nFiles modified:\n- file1.ts\n- file2.ts\n\nChanges:\n- Description of changes\n\nTesting:\n- Tested locally\n- Ready for QA"
        }
      }
    ]
  }'
```

## Update Multiple Properties

### Status + Dev's Comment

```bash
curl -s -X PATCH "https://api.notion.com/v1/pages/[PAGE_ID]" \
  -H "Authorization: Bearer $NOTION_API_KEY" \
  -H "Notion-Version: 2022-06-28" \
  -H "Content-Type: application/json" \
  -d '{
    "properties": {
      "Status": {
        "status": {
          "name": "Resolved"
        }
      },
      "Dev'\''s Comment": {
        "rich_text": [
          {
            "text": {
              "content": "Fix implemented - ready for QA verification"
            }
          }
        ]
      }
    }
  }'
```

## Retrieve Comments

### Get All Comments on a Page

```bash
curl -s -X GET "https://api.notion.com/v1/comments?block_id=[PAGE_ID]" \
  -H "Authorization: Bearer $NOTION_API_KEY" \
  -H "Notion-Version: 2022-06-28"
```

## Workflow: Fix a Bug

### Step 1: Start Work

```bash
# Set status to "Fixing"
curl -s -X PATCH "https://api.notion.com/v1/pages/[PAGE_ID]" \
  -H "Authorization: Bearer $NOTION_API_KEY" \
  -H "Notion-Version: 2022-06-28" \
  -H "Content-Type: application/json" \
  -d '{"properties": {"Status": {"status": {"name": "Fixing"}}}}'
```

### Step 2: Implement Changes

(Make code changes in the codebase)

### Step 3: Complete Work

```bash
# Set status to "Resolved" and add Dev's Comment
curl -s -X PATCH "https://api.notion.com/v1/pages/[PAGE_ID]" \
  -H "Authorization: Bearer $NOTION_API_KEY" \
  -H "Notion-Version: 2022-06-28" \
  -H "Content-Type: application/json" \
  -d '{
    "properties": {
      "Status": {"status": {"name": "Resolved"}},
      "Dev'\''s Comment": {"rich_text": [{"text": {"content": "Fixed navigation to profile page. Modified: PatientLayout.tsx"}}]}
    }
  }'
```

## Response Handling

### Success Response

```json
{
  "object": "page",
  "id": "page-id-here",
  "properties": {
    "Status": {
      "status": {
        "name": "Fixing"
      }
    }
  }
}
```

### Error Response

```json
{
  "object": "error",
  "status": 400,
  "code": "validation_error",
  "message": "Status is not a property that exists"
}
```

## Status Values Reference

| Status | Description |
|--------|-------------|
| **New** | Newly reported, ready to be fixed |
| **Fixing** | Currently being worked on |
| **Resolved** | Fix complete, needs QA verification |
| **Closed** | Verified and closed |
| **Won't Fix** | Not going to be fixed |

## Troubleshooting

### "Could not find property"
- Ensure "Status" property exists and is type "status" (not "select")
- Property names are case-sensitive
- "Dev's Comment" includes an apostrophe

### "Validation error"
- Check status value matches exactly (e.g., "Fixing" not "fixing")
- Verify page_id format (UUID with or without dashes)

### "Object not found"
- Verify page_id is correct
- Ensure integration has access to the page

## Related

- [review-tickets.md](./review-tickets.md) - Review tickets via curl
- [fix-all-by-status.md](./fix-all-by-status.md) - Fix all tickets sequentially
- [fix-single.md](./fix-single.md) - Fix a single ticket
