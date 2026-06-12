# docsync/docsync/apps/cli/src/commands/init.ts

# init.ts

## Overview

This module implements the initialization command for the DocSync CLI. It handles user authentication, project setup, destination platform configuration (Notion, Google Docs, Confluence, GitHub Wiki, Readme.io), payment processing, and optional integrations with Slack, Linear, and GitHub. Configuration is saved to `~/.docsync/config.json` for subsequent CLI operations.

## Functions

### `question(prompt: string): Promise<string>`

Prompts the user for input via the command line and returns the response.

**Parameters:**
- `prompt` (string): The text to display to the user

**Returns:**
- `Promise<string>`: Resolves with the user's input

**Example:**
```typescript
const email = await question('Email: ')
```

---

### `initCommand(): Promise<void>`

Main initialization workflow for DocSync. Handles the complete setup process including authentication, destination selection, payment, and configuration file creation.

**Parameters:**
None

**Returns:**
- `Promise<void>`: Resolves when initialization completes or fails

**Workflow:**
1. Prompts for email and password
2. Authenticates with DocSync API (`/auth/signin`)
3. Creates a new project using current directory name
4. Prompts for destination platform selection (Notion, Google Docs, Confluence, GitHub Wiki, Readme.io)
5. Prompts for pricing plan (Solo, Corporate, Enterprise)
6. Handles OAuth flows for selected destination:
   - **Notion**: Opens browser for OAuth, collects access token and parent page ID
   - **Google Docs**: Opens browser for OAuth, collects access and refresh tokens
   - **Confluence**: Opens browser for OAuth, collects tokens, allows space selection, creates root page
   - **Readme.io**: Prompts for API key
7. Creates Stripe checkout session and opens payment URL
8. Waits for payment verification (polls `/billing/status` up to 10 times)
9. Optionally configures Slack webhook for notifications
10. Optionally configures Linear integration (OAuth + team selection)
11. Optionally configures GitHub integration (OAuth + webhook setup)
12. Writes configuration to `~/.docsync/config.json`

**Example:**
```typescript
import { initCommand } from './commands/init'

await initCommand()
```

**Error Handling:**
- API errors are displayed with `chalk.red()`
- Invalid user choices result in early termination
- Network errors are caught and displayed

## Configuration File

The command creates `~/.docsync/config.json` with the following structure:

```json
{
  "token": "string",
  "refreshToken": "string",
  "email": "string",
  "destination": "notion | googledocs | confluence | githubwiki | readme",
  "projectId": "string",
  "cloudId": "string (Confluence only)",
  "spaceId": "string (Confluence only)",
  "notionToken": "string (optional)",
  "googleToken": "string (optional)",
  "googleRefreshToken": "string (optional)",
  "confluenceToken": "string (optional)",
  "confluenceRefreshToken": "string (optional)",
  "parentPageId": "string",
  "slackWebhookUrl": "string (optional)",
  "githubToken": "string (optional)",
  "githubRepo": "string (optional)",
  "linearToken": "string (optional)",
  "linearTeamId": "string (optional)"
}
```

## API Endpoints

The following external API endpoints are called:

- `POST https://docsync-api.onrender.com/auth/signin`
- `POST https://docsync-api.onrender.com/projects/create`
- `GET https://docsync-api.onrender.com/auth/notion/url`
- `GET https://docsync-api.onrender.com/auth/google/url`
- `GET https://docsync-api.onrender.com/auth/confluence/url`
- `POST https://docsync-api.onrender.com/auth/confluence/resources`
- `POST https://docsync-api.onrender.com/auth/confluence/spaces`
- `POST https://docsync-api.onrender.com/auth/confluence/create-root-page`
- `POST https://docsync-api.onrender.com/billing/checkout`
- `GET https://docsync-api.onrender.com/billing/status`
- `GET https://docsync-api.onrender.com/auth/linear/url`
- `GET https://docsync-api.onrender.com/auth/linear/teams`
- `GET https://docsync-api.onrender.com/auth/github/url`
- `POST https://docsync-api.onrender.com/auth/github/setup-webhook`

## Notes

- Requires `child_process.exec` with `open` command (macOS/Linux) to launch browser windows
- Payment verification polls every 2 seconds for up to 20 seconds
- Readline interface (`rl`) must be manually closed on all exit paths
- Configuration directory `~/.docsync` is created if it doesn't exist
- OAuth flows redirect users to browser and require manual token copy-paste
- GitHub Wiki and Readme.io destinations are listed in the menu but only Readme.io has implementation logic
- Linear integration appears twice in the code (once as destination, once as optional notification setup)