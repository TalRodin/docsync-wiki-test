# docsync/docsync/apps/api/src/services/dispatcher.ts

# dispatcher.ts

## Overview

Centralized service for routing document synchronization operations to various documentation platforms. Handles both creation and update operations, maintains mapping records in Supabase to track document relationships across platforms.

## Functions

### `dispatchDoc()`

Routes document content to the specified destination platform, creating new documents or updating existing ones based on stored mappings.

**Parameters:**

- `projectId` (string) — Unique identifier for the project
- `filePath` (string) — Path of the source file being synchronized
- `content` (string) — Document content to sync
- `destination` (string) — Target platform (`'notion'`, `'googledocs'`, `'confluence'`, `'linear'`, `'readme'`, or `'githubwiki'`)
- `token` (string) — Authentication token for the destination platform
- `parentPageId` (string, optional) — ID of parent page for hierarchical platforms
- `spaceId` (string, optional) — Confluence space identifier
- `githubToken` (string, optional) — GitHub authentication token
- `githubRepo` (string, optional) — GitHub repository in `owner/repo` format
- `linearToken` (string, optional) — Linear API token
- `linearTeamId` (string, optional) — Linear team identifier
- `readmeApiKey` (string, optional) — Readme.io API key

**Returns:**

`Promise<void>` — Resolves when the document is successfully dispatched

**Usage Example:**

```typescript
import { dispatchDoc } from './services/dispatcher'

// Sync to Notion
await dispatchDoc(
  'proj-123',
  'docs/api-guide.md',
  '# API Guide\n\nContent here...',
  'notion',
  'notion-token-xyz',
  'parent-page-id'
)

// Sync to GitHub Wiki
await dispatchDoc(
  'proj-123',
  'docs/setup.md',
  '# Setup Instructions\n\n...',
  'githubwiki',
  'main-token',
  undefined,
  undefined,
  'github-token-abc',
  'myorg/myrepo'
)

// Sync to Linear
await dispatchDoc(
  'proj-123',
  'docs/roadmap.md',
  '# Roadmap\n\n...',
  'linear',
  'unused-token',
  undefined,
  undefined,
  undefined,
  undefined,
  'linear-token-def',
  'team-id-456'
)
```

## Notes

**Required Environment Variables:**

- `CONFLUENCE_URL` — Base URL for Confluence instance (required for Confluence destination)
- `CONFLUENCE_EMAIL` — Email for Confluence authentication (required for Confluence destination)

**Platform-Specific Token Requirements:**

- **Notion**: Requires `token` and `parentPageId` (for creation)
- **Google Docs**: Requires `token`
- **Confluence**: Requires `token`, `spaceId` (for creation), and environment variables
- **Linear**: Requires `linearToken` and `linearTeamId` (throws error if missing)
- **Readme.io**: Requires `readmeApiKey` (throws error if missing)
- **GitHub Wiki**: Requires `githubToken` and `githubRepo` (throws error if missing)

**Database Schema:**

The function expects a `doc_mappings` table in Supabase with columns:
- `project_id` (string)
- `file_path` (string)
- `destination` (string)
- `doc_id` (string)

**Behavior:**

- Queries existing mappings to determine create vs. update operation
- Creates new mapping records after successful document creation
- Updates are performed when a mapping exists for the project/file/destination combination
- GitHub Wiki always performs an upsert operation regardless of mapping status