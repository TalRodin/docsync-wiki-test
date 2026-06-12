# docsync/docsync/apps/cli/src/commands/init.ts

# init.ts

## Overview

Command-line initialization workflow for DocSync that handles user authentication, project creation, payment processing, and integration setup. Guides users through selecting a documentation destination (Notion, Google Docs, Confluence, GitHub Wiki, or Readme.io), configuring OAuth integrations, and setting up optional notifications through Slack, Linear, and GitHub.

## Functions

### `question(prompt: string): Promise<string>`

Prompts the user for input via the command line and returns their response.

**Parameters:**
- `prompt` (string) - The text to display to the user

**Returns:**
- `Promise<string>` - The user's input as a string

**Usage:**
```typescript
const email = await question('Email: ')
```

---

### `initCommand(): Promise<void>`

Main initialization command that orchestrates the entire DocSync setup process.

**Parameters:**
None

**Returns:**
- `Promise<void>` - Completes when initialization is finished or fails

**Usage:**
```typescript
import { initCommand } from './commands/init'

await initCommand()
```

**Process Flow:**
1. Collects email and password credentials
2. Authenticates user via `/auth/signin` endpoint
3. Creates a new project using current directory name
4. Prompts for documentation destination selection (1-5)
5. Handles destination-specific OAuth flows:
   - **Notion**: Opens browser for authorization, collects access token and parent page ID
   - **Google Docs**: Opens browser for OAuth, collects access and refresh tokens
   - **Confluence**: Opens browser for OAuth, collects tokens, lists available spaces, creates root page
   - **GitHub Wiki**: Standard setup
   - **Readme.io**: Collects API key
6. Prompts for billing plan selection (Solo/Corporate/Enterprise)
7. Creates Stripe checkout session and opens payment URL
8. Polls `/billing/status` endpoint to verify payment (max 10 attempts, 2s interval)
9. Optionally configures Slack webhook notifications
10. Optionally configures Linear integration for issue creation
11. Optionally configures GitHub integration for PR comments
12. Saves configuration to `~/.docsync/config.json`

**Configuration File Structure:**
```json
{
  "token": "session_access_token",
  "refreshToken": "session_refresh_token",
  "email": "user@example.com",
  "destination": "notion|googledocs|confluence|githubwiki|readme",
  "projectId": "uuid",
  "cloudId": "confluence_cloud_id",
  "spaceId": "confluence_space_id",
  "notionToken": "notion_access_token",
  "googleToken": "google_access_token",
  "googleRefreshToken": "google_refresh_token",
  "confluenceToken": "confluence_access_token",
  "confluenceRefreshToken": "confluence_refresh_token",
  "parentPageId": "parent_page_id",
  "slackWebhookUrl": "https://hooks.slack.com/...",
  "githubToken": "github_access_token",
  "githubRepo": "username/repo",
  "linearToken": "linear_access_token",
  "linearTeamId": "linear_team_id"
}
```

## API Endpoints Used

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/auth/signin` | POST | User authentication |
| `/projects/create` | POST | Create new project |
| `/auth/notion/url` | GET | Get Notion OAuth URL |
| `/auth/google/url` | GET | Get Google OAuth URL |
| `/auth/confluence/url` | GET | Get Confluence OAuth URL |
| `/auth/confluence/resources` | POST | Fetch Confluence cloud resources |
| `/auth/confluence/spaces` | POST | List available Confluence spaces |
| `/auth/confluence/create-root-page` | POST | Create root documentation page |
| `/billing/checkout` | POST | Create Stripe checkout session |
| `/billing/status` | GET | Verify payment completion |
| `/auth/linear/url` | GET | Get Linear OAuth URL |
| `/auth/linear/teams` | GET | List Linear teams |
| `/auth/github/url` | GET | Get GitHub OAuth URL |
| `/auth/github/setup-webhook` | POST | Configure GitHub webhook |

## Notes

- All OAuth flows use browser-based authorization with manual token copy-paste
- Configuration is stored in plaintext at `~/.docsync/config.json` containing sensitive tokens
- Payment verification uses polling with 20-second timeout (10 attempts × 2 seconds)
- Browser opening uses macOS-specific `open` command via `child_process.exec`
- No validation on user input for email, passwords, or tokens
- Error handling terminates process on authentication or network failures
- Linear integration code appears duplicated in two separate conditional blocks
- Readme.io API key is collected but not saved to configuration file
- Confluence automatically creates a "DocSync" root page in the selected space