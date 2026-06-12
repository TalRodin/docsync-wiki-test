# docsync/docsync/apps/api/src/routes/docs.ts

# docs.ts

## Overview

Express router that handles documentation generation requests. Generates technical documentation from code using Claude AI, dispatches the documentation to specified destinations (Notion, Google Docs, Confluence, GitHub, Linear, ReadMe), and sends notifications via Slack and Linear.

## Routes

### POST `/generate`

Generates documentation from source code and optionally publishes it to external platforms.

**Middleware:**
- `planCheck` - Validates user's subscription plan before processing

**Request Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `filePath` | string | Yes | Path to the file being documented |
| `code` | string | Yes | Source code to generate documentation from |
| `projectId` | string | No | Project identifier for tracking doc mappings |
| `destination` | string | No | Target platform: `notion`, `google`, `confluence`, `github`, `linear`, or `readme` |
| `notionToken` | string | No | Notion API authentication token |
| `googleToken` | string | No | Google API authentication token |
| `confluenceToken` | string | No | Confluence API authentication token |
| `parentPageId` | string | No | Parent page ID for Notion/Confluence |
| `spaceId` | string | No | Space ID for Confluence |
| `slackWebhookUrl` | string | No | Slack webhook URL for notifications |
| `githubToken` | string | No | GitHub API authentication token |
| `githubRepo` | string | No | GitHub repository in `owner/repo` format |
| `linearToken` | string | No | Linear API authentication token |
| `linearTeamId` | string | No | Linear team identifier |
| `readmeApiKey` | string | No | ReadMe API key |

**Response:**

Success (200):
```json
{
  "doc": "Generated documentation content..."
}
```

Error (400):
```json
{
  "error": "filePath and code are required"
}
```

Error (500):
```json
{
  "error": "Error message..."
}
```

**Usage Example:**

```typescript
// Basic documentation generation
fetch('/docs/generate', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    filePath: 'src/utils/helper.ts',
    code: 'export function add(a: number, b: number) { return a + b; }'
  })
});

// Generate and publish to Notion with Slack notification
fetch('/docs/generate', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    filePath: 'src/utils/helper.ts',
    code: 'export function add(a: number, b: number) { return a + b; }',
    projectId: 'proj_123',
    destination: 'notion',
    notionToken: 'secret_xxx',
    parentPageId: 'page_456',
    slackWebhookUrl: 'https://hooks.slack.com/services/xxx'
  })
});
```

**Behavior:**

1. Validates required fields (`filePath` and `code`)
2. Generates documentation using Claude AI service
3. If destination is specified and appropriate token exists, dispatches documentation to target platform
4. Sends Slack notification (failures are silently caught)
5. For Linear destination with token and team ID:
   - Checks for existing doc mapping in database
   - Updates existing Linear doc if mapping found
   - Creates new Linear doc and saves mapping if not found
6. Returns generated documentation

## Dependencies

**Services:**
- `generateDoc` - Claude AI service for documentation generation
- `dispatchDoc` - Multi-platform document dispatcher
- `sendSlackNotification` - Slack notification service
- `createLinearDoc`, `updateLinearDoc` - Linear integration (lazy loaded)
- `supabase` - Database client for doc mappings (lazy loaded)

**Middleware:**
- `planCheck` - Subscription plan validation

## Notes

**Error Handling:**
- Slack notifications fail silently to avoid blocking the main response
- Linear doc creation/update failures are caught and ignored
- Main errors return 500 status with error message

**Database Operations:**
- Linear doc mappings are stored in `doc_mappings` table with fields: `project_id`, `file_path`, `destination`, `doc_id`
- Linear services and Supabase client are dynamically imported only when needed

**Token Selection:**
- Route accepts multiple authentication tokens but only uses the one matching the specified destination
- Token is selected from `notionToken`, `googleToken`, or `confluenceToken` based on availability

**Security:**
- Protected by `planCheck` middleware to enforce subscription limits
- All API tokens are expected to be provided by the client