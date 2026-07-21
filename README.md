<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="images/Trello_logo_dark.svg">
    <img src="images/Trello_logo_light.svg" alt="Trello" width="200">
  </picture>
</p>

<h1 align="center">Trello MCP Server</h1>

<p align="center">
  <b>The official Model Context Protocol (MCP) server for Trello: a cloud-hosted bridge that gives your AI tools secure, real-time access to your Trello content, including boards, lists, cards, checklists and more.</b>
</p>

<!-- Line 1 · Project -->
<p align="center">
  <a href="https://github.com/atlassian/trello-mcp-server"><img src="https://img.shields.io/badge/Official-Trello-1558BC?logo=trello&logoColor=white" alt="Official Trello Server"></a>
  <a href="LICENSE"><img src="https://img.shields.io/github/license/atlassian/trello-mcp-server?label=License&color=1558BC" alt="License: Apache 2.0"></a>
  <a href="https://github.com/atlassian/trello-mcp-server"><img src="https://img.shields.io/badge/Status-Available-2EBC4F" alt="Status: Available"></a>
</p>

<!-- Line 2 · Protocol & access -->
<p align="center">
  <a href="https://modelcontextprotocol.io"><img src="https://img.shields.io/badge/Model_Context_Protocol-compatible-000000?logo=modelcontextprotocol&logoColor=white" alt="Model Context Protocol compatible"></a>
  <a href="#data-and-security"><img src="https://img.shields.io/badge/Auth-OAuth_2.0-2EBC4F" alt="Auth: OAuth 2.0"></a>
  <a href="https://www.atlassian.com/cloud"><img src="https://img.shields.io/badge/Hosting-Atlassian_Cloud-1558BC?logo=atlassian&logoColor=white" alt="Hosting: Atlassian Cloud"></a>
</p>

The official Trello MCP (Model Context Protocol) is a cloud-based bridge between your Trello account and compatible external AI assistants. Once configured, it lets those tools interact with your Trello boards, cards, and tasks using natural language.

With Trello MCP, you can manage work without constantly switching between apps. Your AI assistant can read Trello data, search across it, and take actions on your behalf based on the permissions you grant.

**At launch:** Each Trello MCP connection supports one workspace. Multi-workspace support is planned for a future release.

## Supported AI platforms

Trello MCP supports any app with MCP support, including:

* [OpenAI ChatGPT](https://chatgpt.com/apps/trello/asdk_app_6a20b18a639081918c1b438f8381b27e)
* [Claude](https://support.claude.com/en/articles/11175166-get-started-with-custom-connectors-using-remote-mcp)
* [Cursor](https://cursor.com/docs/mcp#installing-mcp-servers)
* [Visual Studio Code](https://code.visualstudio.com/docs/copilot/customization/mcp-servers#_add-an-mcp-server)
* [Google Gemini CLI](https://github.com/google-gemini/gemini-cli/blob/main/docs/tools/mcp-server.md)

For setup details, refer to your AI client's MCP documentation or built-in instructions.

## Before you start

Ensure your environment meets the necessary requirements to successfully connect to Trello MCP.

### Prerequisites

* A Trello account on any plan
* Access to a supported AI client
* A modern browser to complete the authorisation flow

## How to connect

**For ChatGPT users:** Add Trello MCP from our [listing page](https://chatgpt.com/apps/trello/asdk_app_6a20b18a639081918c1b438f8381b27e).

**For Claude users:** Add Trello MCP from our [listing page](https://claude.ai/directory/connectors/trello).

Listing on other AI platform marketplaces may become available over time. Until then, you can connect by manually adding the MCP URL in your AI client.

1. Add the Trello MCP server URL to your AI client's MCP settings: `https://mcp.trello.com/v1`
2. Start the connection flow from your AI client.
3. On the consent screen, choose the Trello workspace you want to authorise.
4. Review and approve the permissions you want to grant.

**Tip:** If you later want to use a feature that requires a permission you did not grant, disconnect Trello MCP and reconnect it with the updated permissions.

## Agent skill

This repo ships a [`trello-use`](skills/trello-use/SKILL.md) agent skill that guides AI agents on how to call the Trello MCP tools correctly — the ARI id format every tool expects, UTC date handling, Inbox vs. board tools, and ordered creation. Loading it up front avoids common malformed-id and timezone errors.

Install it with the [`skills`](https://github.com/vercel-labs/skills) CLI:

```bash
# Install the trello-use skill from this repo
npx skills install atlassian/trello-mcp-server

# Or install globally (available to all your projects)
npx skills install atlassian/trello-mcp-server -g
```

The CLI detects your agent (Claude Code, Cursor, and others) and installs the skill where that agent looks for it.

## Data and security

* **Authentication** - Trello MCP uses OAuth 2.0 for secure authentication and access control.
* **Workspace-scoped access** - Your AI assistant can only access the workspace you explicitly authorise on the consent screen.
* **Permission management** - All actions respect your existing Trello permissions. Your AI assistant cannot perform actions beyond what you can do in Trello directly.
* **No destructive operations** - Destructive delete operations are not supported. Your assistant can archive cards and lists, but cannot permanently delete them.
* **Revocable access** - You can revoke your AI assistant's access at any time.

## Admin controls

Organization admins can manage how Trello MCP is used through the Atlassian Administration.

* **[Control Atlassian Rovo MCP server settings](https://support.atlassian.com/security-and-access-policies/docs/control-atlassian-rovo-mcp-server-settings/)** - Manage which MCP server domains are allowed or blocked for your organization.
* **[Configure Atlassian Rovo MCP server permission](https://support.atlassian.com/security-and-access-policies/docs/Configure-Atlassian-Rovo-MCP-server-permission/)** - Control what level of access (Read, Write, Search) is allowed when users connect to MCP servers, including Trello MCP.
* **Admin MCP access controls** - Enterprise organizations managed through [admin.atlassian.com](http://admin.atlassian.com/) can configure MCP access controls in Atlassian Administration.

Trello MCP server uses the same admin control framework as the Atlassian Rovo MCP server. Separate Trello-specific admin tooling is not required.

## What you can do

Once connected, you can ask your AI assistant to perform a variety of tasks.

### Capture and organise ideas

* "Create a Trello board for the project plan we just discussed."
* "Add these books to my reading list board in Trello."

### View your work

* "What Trello tasks are due this week for me?"
* "Show me everything that's overdue across my boards."
* "What's the current progress on my kitchen renovation board?"

### Update tasks

* "Add a checklist called 'Prep Steps' to my Quarterly Review card."
* "Mark my 'Submit report' card as done."
* "Move 'Fix login bug' card to the In Progress list on my Sprint Board."

### Clean up and triage

* "Archive everything in my Inbox that's older than a week."
* "Move everything in my Inbox to the corresponding boards."
* "Help me consolidate these three boards into one."

### Schedule focus time

* "Block 2 hours tomorrow morning for 'Draft social copy'."
* "When's a good time today for me to rehearse my presentation?"

### Personal insights and board enrichment

* "Pull every Trello card I marked as done over the last year and write a recap I can use in my annual performance review."
* "Review the tasks on my wedding planning board and add anything missing."

## Supported capabilities

| Category             | What you can do                                                                    |
| -------------------- | ---------------------------------------------------------------------------------- |
| **Member**     | View your profile and defaults                                                     |
| **Workspaces** | View workspace details                                                             |
| **Boards**     | View and create boards; view labels                                                |
| **Lists**      | View and move lists                                                                |
| **Cards**      | View, create, update, move, archive, and mark cards done; attach and detach labels |
| **Checklists** | View, create, and update checklists; add and update checklist items                |
| **Search**     | Search for cards and boards by keyword                                             |
| **Planner**    | View calendar events; create focus-time events; link and unlink cards to events    |
| **Inbox**      | View, create, update, and archive Inbox cards                                      |

**Planner and focus time**

Users can connect their Google or Outlook calendar to Planner to read calendar events. However, creating focus time in Planner requires Trello Premium or enterprise.

## More capabilities coming soon

Trello MCP is actively being developed. The current release includes core tools for managing boards, lists, cards, checklists, and search. Additional tools are planned for upcoming releases to expand what your AI assistant can do with Trello:

| Category                     | What's coming                                         |
| ---------------------------- | ----------------------------------------------------- |
| **Activity & History** | View action history for cards, lists, and boards      |
| **Comments**           | View, add, and edit comments on cards                 |
| **Attachments**        | View, upload, and download card attachments           |
| **Custom Fields**      | View field definitions; get and set values on cards   |
| **Labels**             | Create, view, and update labels                       |
| **Boards**             | Edit board details (name, visibility); archive boards |
| **Cards**              | Copy cards                                            |
| **Members**            | View and manage board, workspace, and card members    |
| **Planner**            | View events linked to specific cards                  |
| **Workspaces**         | Create workspace                                      |

We'll update this page as new capabilities become available.

## Trello MCP vs. Atlassian Rovo MCP

|                        | Trello MCP                                        | Atlassian Rovo MCP                                                            |
| ---------------------- | ------------------------------------------------- | ----------------------------------------------------------------------------- |
| **Access**       | Trello data only                                  | Jira, Confluence, Compass, Bitbucket, and more Atlassian product data         |
| **Who it's for** | Trello users connecting to external AI assistants | Atlassian Cloud users connecting external AI assistants to Atlassian products |
| **Scope**        | One Trello workspace per connection at launch     | Atlassian cloud site scope                                                    |

In a future release, Trello data may also be available through the Atlassian Rovo MCP server for customers who use Trello alongside other Atlassian products.

## FAQ

### Do I need a paid Trello plan to use Trello MCP?

No. Trello MCP works with all Trello plans. However, creating focus time in Planner requires Trello Premium or Enterprise. Reading calendar events from a connected Google or Outlook calendar in Planner is available more broadly.

### What is a Trello MCP server?

A Trello MCP server is the secure connection layer that allows AI assistants such as Claude or ChatGPT to access Trello data based on the permissions you approve.

### What permissions can I grant?

Trello MCP can request Read and Write permissions. Read lets AI assistants view your existing work as context. Write lets them create and update work on your behalf using your existing Trello permissions. Search lets them find relevant Trello content without changing it.

### What happens if I do not grant all permissions?

Some Trello MCP features may not work. If you later want to use a feature that depends on a permission you did not grant, disconnect Trello MCP and reconnect it with the updated permissions.

### Why can't I grant Inbox and Planner permissions for a workspace outside my organization?

Inbox and Planner permissions operate at the account level and are tied to your organization's management controls. If you select a workspace that is not owned by your organization, those permissions may be disabled. You can still grant Read and Write permissions for that workspace's boards and cards.

### I see "Access to this domain is restricted." What does that mean?

Your organization admin has blocked the MCP server domain in your organization's settings. Contact your organization admin to request access or review [domain restriction settings](https://support.atlassian.com/security-and-access-policies/docs/control-atlassian-rovo-mcp-server-settings/).

### I see "Permissions are blocked for this workspace." What should I do?

Your admin has not allowed MCP permissions for that workspace or product access combination. Contact your organization admin or review [MCP permission settings](https://support.atlassian.com/security-and-access-policies/docs/Configure-Atlassian-Rovo-MCP-server-permission/).

### I see "No Trello workspaces found" or "No workspaces available." Why?

This can happen if you signed in with a different Atlassian account than the one you use for Trello, if you no longer belong to any Trello workspace, or if your access has changed. In some cases, you may still be able to connect Inbox and Planner even without a workspace, but boards, cards, and lists will not be available until you join or create a workspace.

### I see "You no longer have access to this workspace." What happened?

Your access to that workspace has been removed. Select another workspace if available, or contact the workspace admin to restore access.

### I see "Something went wrong" on the permissions screen. What should I do?

This usually means the permissions data could not be loaded. Refresh the page and try again. If the issue continues, contact [Trello support](https://support.atlassian.com/trello).

### Can my AI assistant permanently delete boards or cards?

No. Trello MCP does not support destructive delete operations. Your assistant can archive cards and lists, but cannot permanently delete them.

### Is my data secure?

Yes. Trello MCP uses OAuth 2.0 for authentication. Your AI assistant can only access the workspace you explicitly authorise on the consent screen. You can revoke access at any time.

### I'm a Trello Enterprise admin. How do I control whether my users can use Trello MCP?

Go to Atlassian Administration → Rovo → Rovo MCP server → Permissions. Click "Edit details" under Read or Write, and uncheck `read_trello` or `write_trello`.

### As Enterprise admin, can I restrict which MCP server domains my users connect to without disabling Trello MCP entirely?

Yes. Go to Atlassian Administration → Rovo → Rovo MCP server → Domains to allow or block specific MCP server domains.

### Does the API token authentication setting in Atlassian Administration apply to Trello MCP?

No. The "Authentication" tab under Atlassian Administration > Rovo > Rovo MCP server is not relevant to Trello MCP. Trello does not currently support API token-based authentication for MCP connections. Trello MCP uses OAuth consent (users authorise via the consent screen when connecting their AI client).

However, if your organization also uses the **Atlassian Rovo MCP** (for Jira/Confluence), the API token authentication setting does apply there. These are separate configurations, and changes to the Authentication tab will not affect Trello MCP access.
