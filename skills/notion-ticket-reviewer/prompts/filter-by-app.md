# Filter by App / Dashboard

## Prompt Template

```
Enter plan mode - Review notion tickets
Filter: App = "[APP_NAME]", Status = "New"
```

## Description

Filter Bug Report tickets by App / Dashboard to see issues relevant to a specific application.

## Available Apps

| App | Description |
|-----|-------------|
| **APP** | Patient mobile application |
| **Dashboard** | Coach/Admin web dashboard |
| **Admin** | System administration panel |
| **Web** | Public-facing website |

## Example Usage

### Patient App
```
Enter plan mode - Review notion tickets for database [DATABASE_ID]
Filter: App = "APP", Status = "New"
```

### Dashboard
```
Enter plan mode - Review notion tickets for database [DATABASE_ID]
Filter: App = "Dashboard"
```

### Multiple Apps
```
Enter plan mode - Review notion tickets for database [DATABASE_ID]
Filter: App = "APP" OR App = "Dashboard"
```

### App + Type
```
Enter plan mode - Review notion tickets for database [DATABASE_ID]
Filter: App = "APP", Type = "Bug"
```

## Curl Commands

### Get API Key from .env
```bash
NOTION_API_KEY=$(grep -E "^NOTION_API_KEY=" .env | cut -d'=' -f2)
```

### Filter by Single App
```bash
curl -s -X POST "https://api.notion.com/v1/databases/[DATABASE_ID]/query" \
  -H "Authorization: Bearer $NOTION_API_KEY" \
  -H "Notion-Version: 2022-06-28" \
  -H "Content-Type: application/json" \
  -d '{
    "filter": {
      "and": [
        {"property": "App / Dashboard", "multi_select": {"contains": "APP"}},
        {"property": "Status", "status": {"equals": "New"}}
      ]
    },
    "page_size": 100
  }'
```

### Filter by Multiple Apps
```bash
curl -s -X POST "https://api.notion.com/v1/databases/[DATABASE_ID]/query" \
  -H "Authorization: Bearer $NOTION_API_KEY" \
  -H "Notion-Version: 2022-06-28" \
  -H "Content-Type: application/json" \
  -d '{
    "filter": {
      "and": [
        {
          "or": [
            {"property": "App / Dashboard", "multi_select": {"contains": "APP"}},
            {"property": "App / Dashboard", "multi_select": {"contains": "Dashboard"}}
          ]
        },
        {"property": "Status", "status": {"equals": "New"}}
      ]
    },
    "page_size": 100
  }'
```

## API Filter Reference

### Single App
```json
{
  "filter": {
    "and": [
      {
        "property": "App / Dashboard",
        "multi_select": {
          "contains": "APP"
        }
      },
      {
        "property": "Status",
        "status": {
          "equals": "New"
        }
      }
    ]
  }
}
```

### Multiple Apps
```json
{
  "filter": {
    "and": [
      {
        "or": [
          {"property": "App / Dashboard", "multi_select": {"contains": "APP"}},
          {"property": "App / Dashboard", "multi_select": {"contains": "Dashboard"}}
        ]
      },
      {
        "property": "Status",
        "status": {
          "equals": "New"
        }
      }
    ]
  }
}
```

## Output Format

```
Found [N] tickets for app [APP_NAME]:

### /patient (3 issues)
1. #2 My profile detail page
2. #3 Calendar gap between elements
3. #4 All component sizes

### /patient/chat (1 issue)
4. #7 Chatroom's latest message

### /patient/survey (2 issues)
5. #5 Padding sizes
6. #6 Survey check effect
```

## Common Use Cases

### Daily Bug Triage (APP)
```
Enter plan mode - Review notion tickets
Filter: App = "APP", Status = "New", Type = "Bug"
```

### Dashboard Issues
```
Enter plan mode - Review notion tickets
Filter: App = "Dashboard", Status = "New"
```

### All Frontend Issues
```
Enter plan mode - Review notion tickets
Filter: App = "APP" OR App = "Dashboard" OR App = "Web"
```

### Resolved Issues for QA
```
Enter plan mode - Review notion tickets
Filter: App = "APP", Status = "Resolved"
```

## Related

- [review-all.md](./review-all.md) - Review all tickets
- [filter-by-project.md](./filter-by-project.md) - Filter by project
