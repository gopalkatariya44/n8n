# n8n REST API Endpoint Risk Catalog

> Generated for incident analysis: unauthorized session access investigation.
> Use with Traefik HTTP access logs (method, path, status) to score session exposure.

## Risk Categories

| Level | Definition |
|-------|-----------|
| **CRITICAL** | Returns credential secrets in plaintext, exports sensitive data, or performs irreversible destructive actions |
| **HIGH** | Returns sensitive metadata (user list with emails, instance settings with secrets), modifies permissions or security settings |
| **MEDIUM** | Modifies instance data (creates/updates/deletes workflows, executions, projects) or returns non-secret instance data |
| **LOW** | Read-only access to non-sensitive data (workflow list, node types, tags, community nodes) |
| **NONE** | Infrastructure endpoints with no security relevance (healthz, settings, license, telemetry proxy, static assets) |

---

## Consolidated Endpoint Table (sorted by risk)

### CRITICAL

| Method | Path Pattern | What It Does | Response Contains | Source File |
|--------|-------------|-------------|-------------------|------------|
| GET | `/rest/ldap/config` | Get LDAP configuration | **Decrypted `bindingAdminPassword` in plaintext**, connectionUrl, baseDn, bindingAdminDn | `modules/ldap.ee/ldap.controller.ee.ts:19` |
| PUT | `/rest/ldap/config` | Update LDAP config | **Decrypted `bindingAdminPassword` in response** | `modules/ldap.ee/ldap.controller.ee.ts:39` |
| GET | `/rest/credentials?includeData=true` | List credentials with data | Decrypted credential data: **non-password fields in plaintext** (hostnames, usernames, client IDs, regions). Password fields replaced with blank sentinel. Requires `credential:update` scope. | `credentials/credentials.controller.ts:66` |
| GET | `/rest/credentials/:credentialId?includeData=true` | Get credential with data | Same as above for single credential. **Non-password fields in plaintext.** | `credentials/credentials.controller.ts:110` |
| GET | `/rest/executions/:id` | Get execution details | **Full node output data** from workflow run — may contain API responses with tokens, PII, database results, any data processed by nodes | `executions/executions.controller.ts:90` |
| GET | `/api/v1/executions/:id` | Get execution (Public API) | Same — **full node output data** when `includeData=true`. Public API does NOT use redaction service | `public-api/v1/handlers/executions/executions.handler.ts:71` |
| GET | `/api/v1/executions` | List executions (Public API) | **Bulk execution data** when `includeData=true` — all node outputs across all executions | `public-api/v1/handlers/executions/executions.handler.ts:102` |
| GET | `/rest/workflows/:workflowId/executions/last-successful` | Get last successful execution | **Full execution data** with all node outputs | `workflows/workflows.controller.ts:808` |
| GET | `/rest/variables/` | List all variables | **Variable values in plaintext** — users commonly store secrets (API keys, passwords) as variables | `environments.ee/variables/variables.controller.ee.ts:21` |
| GET | `/rest/variables/:id` | Get variable | **Variable value in plaintext** | `environments.ee/variables/variables.controller.ee.ts:49` |
| GET | `/api/v1/variables` | List variables (Public API) | **Variable values in plaintext** | `public-api/v1/handlers/variables/variables.handler.ts:55` |
| GET | `/rest/mfa/qr` | Get MFA setup data | **TOTP `secret` and `recoveryCodes` in plaintext** — allows attacker to clone user's 2FA | `controllers/mfa.controller.ts:64` |
| POST | `/rest/api-keys/` | Create API key | **`rawApiKey` in plaintext** — full API key shown once on creation | `controllers/api-keys.controller.ts:43` |
| GET | `/rest/mcp/api-key` | Get/create MCP API key | **API key in plaintext** | `modules/mcp/mcp.settings.controller.ts:57` |
| POST | `/rest/mcp/api-key/rotate` | Rotate MCP API key | **New API key in plaintext** | `modules/mcp/mcp.settings.controller.ts:63` |
| POST | `/rest/ai/free-credits` | Claim free AI credits | **Creates credential with API key** | `controllers/ai.controller.ts:208` |
| POST | `/rest/workflows/:workflowId/run` | Execute workflow manually | **Execution results with node outputs** — may contain any data the workflow processes | `workflows/workflows.controller.ts:668` |
| GET | `/rest/debug/multi-main-setup` | Debug multi-main info | **NO AUTH (skipAuth)**. Exposes instanceId, leaderKey, all active workflows, activation errors | `controllers/debug.controller.ts:17` |
| POST | `/rest/e2e/reset` | Reset entire database | **NO AUTH (skipAuth)**. Truncates all tables, recreates users. Only loaded when `N8N_E2E_TESTS=true` | `controllers/e2e.controller.ts:193` |
| POST | `/rest/source-control/push-workfolder` | Push to git remote | **Exports all workflows/credentials/variables to external git** — data exfiltration vector | `modules/source-control.ee/source-control.controller.ee.ts:173` |
| POST | `/rest/quick-connect/` | Quick-connect credential data | **May return API keys/tokens from external provider** | `modules/quick-connect/quick-connect.controller.ts:11` |

### HIGH

| Method | Path Pattern | What It Does | Response Contains | Source File |
|--------|-------------|-------------|-------------------|------------|
| GET | `/rest/users/` | List all users | **User emails, names, roles**, project relations, invite URLs for pending users | `controllers/users.controller.ts:112` |
| GET | `/api/v1/users` | List users (Public API) | **User emails, names, roles** (cleaned but includes email) | `public-api/v1/handlers/users/users.handler.ee.ts:50` |
| GET | `/api/v1/users/:id` | Get user (Public API) | **User email, name, role** | `public-api/v1/handlers/users/users.handler.ee.ts:27` |
| GET | `/rest/users/:id/password-reset-link` | Generate password reset link | **Usable password reset URL** — allows account takeover | `controllers/users.controller.ts:152` |
| POST | `/rest/users/:id/invite-link` | Generate invite link | **Invite URL with JWT** | `controllers/users.controller.ts:174` |
| DELETE | `/rest/users/:id` | Delete a user | **Irreversible user deletion** | `controllers/users.controller.ts:223` |
| PATCH | `/rest/users/:id/role` | Change user role | **Privilege escalation** — can grant admin role | `controllers/users.controller.ts:345` |
| POST | `/rest/invitations/` | Send user invitations | **Creates new user accounts**, sends invite emails | `controllers/invitation.controller.ts:49` |
| PUT | `/rest/credentials/:credentialId/share` | Share credential | **Grants credential access to other projects** | `credentials/credentials.controller.ts:322` |
| PUT | `/rest/workflows/:workflowId/share` | Share workflow | **Grants workflow access to other projects**, sends notifications | `workflows/workflows.controller.ts:709` |
| PUT | `/rest/credentials/:credentialId/transfer` | Transfer credential | **Changes credential ownership** | `credentials/credentials.controller.ts:414` |
| PUT | `/rest/workflows/:workflowId/transfer` | Transfer workflow | **Changes workflow ownership** | `workflows/workflows.controller.ts:791` |
| PUT | `/api/v1/workflows/:id/transfer` | Transfer workflow (Public API) | **Changes workflow ownership** | `public-api/v1/handlers/workflows/workflows.handler.ts:66` |
| PUT | `/api/v1/credentials/:id/transfer` | Transfer credential (Public API) | **Changes credential ownership** | `public-api/v1/handlers/credentials/credentials.handler.ts:159` |
| POST | `/rest/license/activate` | Activate license | Returns `managementToken` JWT for license management | `license/license.controller.ts:58` |
| POST | `/rest/license/renew` | Renew license | Returns `managementToken` JWT | `license/license.controller.ts:70` |
| GET | `/rest/projects/:projectId` | Get project details | **Member emails, names, roles** in relations array | `controllers/project.controller.ts:181` |
| POST | `/rest/projects/:projectId/users` | Add users to project | **Modifies project membership** | `controllers/project.controller.ts:227` |
| PATCH | `/rest/projects/:projectId/users/:userId` | Change user role in project | **Modifies project permissions** | `controllers/project.controller.ts:271` |
| DELETE | `/rest/projects/:projectId/users/:userId` | Remove user from project | **Removes access** | `controllers/project.controller.ts:299` |
| POST | `/rest/owner/setup` | Set up instance owner | **NO AUTH (skipAuth)**. Creates owner account on first setup | `controllers/owner.controller.ts:25` |
| GET | `/rest/settings` | Get frontend settings (authed) | instanceId, posthog apiKey, telemetry writeKey, deployment type, license info, enterprise features | `server.ts:465` |
| POST | `/rest/settings/security/` | Update security settings | **Modifies personal space publishing/sharing policies** | `controllers/security-settings.controller.ts:45` |
| GET | `/rest/sso/saml/config` | Get SAML config | SAML preferences, entityID, returnUrl | `modules/sso-saml/saml.controller.ee.ts:50` |
| POST | `/rest/sso/saml/config` | Set SAML config | **Modifies SAML SSO configuration** | `modules/sso-saml/saml.controller.ee.ts:63` |
| POST | `/rest/sso/saml/config/toggle` | Toggle SAML login | **Enables/disables SAML authentication** | `modules/sso-saml/saml.controller.ee.ts:72` |
| GET | `/rest/sso/oidc/config` | Get OIDC config | OIDC config (clientSecret redacted) | `modules/sso-oidc/oidc.controller.ee.ts:28` |
| POST | `/rest/sso/oidc/config` | Set OIDC config | **Modifies OIDC SSO configuration** | `modules/sso-oidc/oidc.controller.ee.ts:39` |
| GET | `/rest/sso/provisioning/config` | Get SCIM provisioning config | Provisioning configuration | `modules/provisioning.ee/provisioning.controller.ee.ts:14` |
| PATCH | `/rest/sso/provisioning/config` | Update provisioning config | **Modifies SCIM provisioning** | `modules/provisioning.ee/provisioning.controller.ee.ts:24` |
| POST | `/rest/mfa/enforce-mfa` | Enforce MFA for all users | **Forces all users to enable MFA** | `controllers/mfa.controller.ts:28` |
| GET | `/rest/api-keys/` | List API keys | API key metadata (keys redacted, but IDs visible) | `controllers/api-keys.controller.ts:74` |
| DELETE | `/rest/api-keys/:id` | Delete API key | **Revokes API key** | `controllers/api-keys.controller.ts:84` |
| GET | `/rest/external-secrets/providers` | List external secret providers | Provider metadata, connection state | `modules/external-secrets.ee/external-secrets.controller.ee.ts:32` |
| GET | `/rest/external-secrets/providers/:provider` | Get external secret provider | Provider details, properties | `modules/external-secrets.ee/external-secrets.controller.ee.ts:38` |
| POST | `/rest/external-secrets/providers/:provider` | Set provider settings | **Configures secret provider connection** | `modules/external-secrets.ee/external-secrets.controller.ee.ts:57` |
| GET | `/rest/eventbus/destination` | List log streaming destinations | Destination configs (may include connection details) | `modules/log-streaming.ee/log-streaming.controller.ts:39` |
| POST | `/rest/eventbus/destination` | Create log streaming destination | **Creates external logging destination** | `modules/log-streaming.ee/log-streaming.controller.ts:51` |
| DELETE | `/rest/eventbus/destination` | Delete log streaming destination | **Removes audit trail** | `modules/log-streaming.ee/log-streaming.controller.ts:107` |
| POST | `/rest/source-control/preferences` | Set source control preferences | **Configures git integration** (repo URL, keys) | `modules/source-control.ee/source-control.controller.ee.ts:44` |
| POST | `/rest/source-control/pull-workfolder` | Pull from git remote | **Imports workflows/credentials from external git** | `modules/source-control.ee/source-control.controller.ee.ts:208` |
| GET | `/rest/source-control/reset-workfolder` | Reset work folder | **Resets local state to remote git** | `modules/source-control.ee/source-control.controller.ee.ts:224` |
| PATCH | `/rest/roles/:slug` | Update custom role | **Modifies role permissions** | `controllers/role.controller.ts:80` |
| DELETE | `/rest/roles/:slug` | Delete custom role | **Removes role** | `controllers/role.controller.ts:92` |
| POST | `/rest/roles/` | Create custom role | **Creates new role with permissions** | `controllers/role.controller.ts:103` |
| PATCH | `/rest/mcp/settings` | Toggle MCP access | **Enables/disables MCP server** | `modules/mcp/mcp.settings.controller.ts:38` |
| DELETE | `/rest/mcp/oauth-clients/:clientId` | Delete MCP OAuth client | **Revokes OAuth client** | `modules/mcp/mcp.oauth-clients.controller.ts:59` |

### MEDIUM

| Method | Path Pattern | What It Does | Response Contains | Source File |
|--------|-------------|-------------|-------------------|------------|
| GET | `/rest/credentials/` | List credentials (no includeData) | Credential metadata: id, name, type, timestamps. No secrets | `credentials/credentials.controller.ts:66` |
| GET | `/rest/credentials/:credentialId` | Get credential (no includeData) | Credential metadata + scopes. No secrets | `credentials/credentials.controller.ts:110` |
| POST | `/rest/credentials/` | Create credential | Created credential entity (data encrypted in DB) | `credentials/credentials.controller.ts:176` |
| PATCH | `/rest/credentials/:credentialId` | Update credential | Updated metadata (data/shared stripped from response) | `credentials/credentials.controller.ts:205` |
| DELETE | `/rest/credentials/:credentialId` | Delete credential | `true` | `credentials/credentials.controller.ts:289` |
| POST | `/rest/credentials/test` | Test credential connection | Pass/fail result only | `credentials/credentials.controller.ts:138` |
| POST | `/api/v1/credentials` | Create credential (Public API) | Sanitized (no data field) | `public-api/v1/handlers/credentials/credentials.handler.ts:95` |
| PATCH | `/api/v1/credentials/:id` | Update credential (Public API) | Sanitized (no data field) | `public-api/v1/handlers/credentials/credentials.handler.ts:113` |
| DELETE | `/api/v1/credentials/:id` | Delete credential (Public API) | Sanitized (no data field) | `public-api/v1/handlers/credentials/credentials.handler.ts:174` |
| GET | `/rest/workflows/` | List all workflows | Workflow metadata, nodes (with credential refs, not secrets), connections | `workflows/workflows.controller.ts:279` |
| GET | `/rest/workflows/:workflowId` | Get workflow | Full workflow entity, nodes, connections, used credentials metadata | `workflows/workflows.controller.ts:358` |
| POST | `/rest/workflows/` | Create workflow | Created workflow entity | `workflows/workflows.controller.ts:103` |
| PATCH | `/rest/workflows/:workflowId` | Update workflow | Updated workflow entity | `workflows/workflows.controller.ts:448` |
| DELETE | `/rest/workflows/:workflowId` | Delete workflow | `true` | `workflows/workflows.controller.ts:514` |
| POST | `/rest/workflows/:workflowId/activate` | Activate workflow | Workflow entity (now active) | `workflows/workflows.controller.ts:603` |
| POST | `/rest/workflows/:workflowId/deactivate` | Deactivate workflow | Workflow entity (now inactive) | `workflows/workflows.controller.ts:637` |
| POST | `/rest/workflows/:workflowId/archive` | Archive workflow | Workflow entity | `workflows/workflows.controller.ts:535` |
| POST | `/rest/workflows/:workflowId/unarchive` | Unarchive workflow | Workflow entity | `workflows/workflows.controller.ts:569` |
| GET | `/rest/workflows/from-url` | Import workflow from URL | Fetched workflow JSON (nodes, connections) | `workflows/workflows.controller.ts:320` |
| POST | `/rest/workflows/with-node-types` | Find workflows by node types (admin) | Matching workflows with count | `workflows/workflows.controller.ts:820` |
| GET | `/api/v1/workflows` | List all workflows (Public API) | **Bulk**: full workflow definitions, nodes, connections, settings, staticData, pinData | `public-api/v1/handlers/workflows/workflows.handler.ts:162` |
| GET | `/api/v1/workflows/:id` | Get workflow (Public API) | Full workflow entity | `public-api/v1/handlers/workflows/workflows.handler.ts:99` |
| GET | `/api/v1/workflows/:id/:versionId` | Get workflow version (Public API) | Historical workflow version | `public-api/v1/handlers/workflows/workflows.handler.ts:135` |
| PUT | `/api/v1/workflows/:id` | Update workflow (Public API) | Updated workflow | `public-api/v1/handlers/workflows/workflows.handler.ts:299` |
| DELETE | `/api/v1/workflows/:id` | Delete workflow (Public API) | Deleted workflow | `public-api/v1/handlers/workflows/workflows.handler.ts:83` |
| POST | `/api/v1/workflows/:id/activate` | Activate workflow (Public API) | Updated workflow | `public-api/v1/handlers/workflows/workflows.handler.ts:331` |
| POST | `/api/v1/workflows/:id/deactivate` | Deactivate workflow (Public API) | Updated workflow | `public-api/v1/handlers/workflows/workflows.handler.ts:358` |
| GET | `/rest/executions/` | List executions | Execution summaries (no full data): id, status, timestamps, workflowId | `executions/executions.controller.ts:37` |
| POST | `/rest/executions/:id/stop` | Stop execution | Stop result | `executions/executions.controller.ts:105` |
| POST | `/rest/executions/stopMany` | Stop multiple executions | `{ stopped: count }` | `executions/executions.controller.ts:121` |
| POST | `/rest/executions/:id/retry` | Retry execution | Execution result | `executions/executions.controller.ts:132` |
| POST | `/rest/executions/delete` | Delete executions (bulk) | Deletion result | `executions/executions.controller.ts:141` |
| PATCH | `/rest/executions/:id` | Update execution annotations | Updated execution | `executions/executions.controller.ts:150` |
| DELETE | `/api/v1/executions/:id` | Delete execution (Public API) | Deleted execution | `public-api/v1/handlers/executions/executions.handler.ts:25` |
| POST | `/api/v1/executions/:id/retry` | Retry execution (Public API) | Retry result | `public-api/v1/handlers/executions/executions.handler.ts:166` |
| POST | `/api/v1/executions/:id/stop` | Stop execution (Public API) | Stop result | `public-api/v1/handlers/executions/executions.handler.ts:257` |
| POST | `/api/v1/executions/stop` | Stop many executions (Public API) | `{ stopped: count }` | `public-api/v1/handlers/executions/executions.handler.ts:285` |
| POST | `/rest/variables/` | Create variable | Created variable (licensed) | `environments.ee/variables/variables.controller.ee.ts:30` |
| PATCH | `/rest/variables/:id` | Update variable | Updated variable (licensed) | `environments.ee/variables/variables.controller.ee.ts:58` |
| DELETE | `/rest/variables/:id` | Delete variable | `true` (licensed) | `environments.ee/variables/variables.controller.ee.ts:78` |
| POST | `/api/v1/variables` | Create variable (Public API) | 201 | `public-api/v1/handlers/variables/variables.handler.ts:20` |
| PATCH | `/api/v1/variables/:id` | Update variable (Public API) | 204 | `public-api/v1/handlers/variables/variables.handler.ts:33` |
| DELETE | `/api/v1/variables/:id` | Delete variable (Public API) | 204 | `public-api/v1/handlers/variables/variables.handler.ts:46` |
| GET | `/rest/projects/` | List projects | Array of project objects | `controllers/project.controller.ts:49` |
| POST | `/rest/projects/` | Create project | Created project with role and scopes | `controllers/project.controller.ts:59` |
| PATCH | `/rest/projects/:projectId` | Update project | void | `controllers/project.controller.ts:216` |
| DELETE | `/rest/projects/:projectId` | Delete project | void | `controllers/project.controller.ts:319` |
| PATCH | `/rest/me/` | Update own profile | PublicUser (updated) | `controllers/me.controller.ts:44` |
| PATCH | `/rest/me/password` | Change own password | `{ success: true }` | `controllers/me.controller.ts:168` |
| PATCH | `/rest/me/settings` | Update own settings | Settings object | `controllers/me.controller.ts:278` |
| PATCH | `/rest/users/:id/settings` | Update user settings | Settings object | `controllers/users.controller.ts:202` |
| POST | `/rest/mfa/enable` | Enable MFA | void (sets cookie) | `controllers/mfa.controller.ts:106` |
| POST | `/rest/mfa/disable` | Disable MFA | void (sets cookie) | `controllers/mfa.controller.ts:147` |
| PATCH | `/rest/api-keys/:id` | Update API key | `{ success: true }` | `controllers/api-keys.controller.ts:97` |
| POST | `/rest/community-packages/` | Install community package | InstalledPackages entity | `modules/community-packages/community-packages.controller.ts:49` |
| DELETE | `/rest/community-packages/` | Uninstall community package | void | `modules/community-packages/community-packages.controller.ts:204` |
| PATCH | `/rest/community-packages/` | Update community package | Updated package | `modules/community-packages/community-packages.controller.ts:259` |
| GET | `/rest/workflow-history/workflow/:workflowId` | List workflow versions | Version metadata list | `workflows/workflow-history/workflow-history.controller.ts:22` |
| GET | `/rest/workflow-history/workflow/:workflowId/version/:versionId` | Get workflow version | Full historical workflow snapshot | `workflows/workflow-history/workflow-history.controller.ts:39` |
| POST | `/rest/workflow-history/workflow/:workflowId/versions` | Get multiple versions | Version objects | `workflows/workflow-history/workflow-history.controller.ts:57` |
| PATCH | `/rest/workflow-history/workflow/:workflowId/versions/:versionId` | Update version metadata | Updated version | `workflows/workflow-history/workflow-history.controller.ts:78` |
| POST | `/rest/source-control/disconnect` | Disconnect source control | Disconnect result | `modules/source-control.ee/source-control.controller.ee.ts:154` |
| POST | `/rest/source-control/generate-key-pair` | Generate SSH key pair | Preferences with new publicKey | `modules/source-control.ee/source-control.controller.ee.ts:259` |
| PATCH | `/rest/source-control/preferences` | Update source control prefs | Updated preferences | `modules/source-control.ee/source-control.controller.ee.ts:109` |
| POST | `/api/v1/source-control/pull` | Pull from source control (Public API) | Import result | `public-api/v1/handlers/source-control/source-control.handler.ts:19` |
| POST | `/rest/projects/:projectId/data-tables/` | Create data table | Data table entity | `modules/data-table/data-table.controller.ts:100` |
| PATCH | `/rest/projects/:projectId/data-tables/:dataTableId` | Update data table | Updated data table | `modules/data-table/data-table.controller.ts:137` |
| DELETE | `/rest/projects/:projectId/data-tables/:dataTableId` | Delete data table | Result | `modules/data-table/data-table.controller.ts:161` |
| GET | `/rest/projects/:projectId/data-tables/:dataTableId/rows` | Get data table rows | Paginated row data | `modules/data-table/data-table.controller.ts:278` |
| GET | `/rest/projects/:projectId/data-tables/:dataTableId/download-csv` | Download data table as CSV | CSV content | `modules/data-table/data-table.controller.ts:303` |
| POST | `/rest/projects/:projectId/data-tables/:dataTableId/insert` | Insert rows | Inserted rows | `modules/data-table/data-table.controller.ts:344` |
| POST | `/rest/projects/:projectId/data-tables/:dataTableId/upsert` | Upsert rows | Upsert result | `modules/data-table/data-table.controller.ts:373` |
| PATCH | `/rest/projects/:projectId/data-tables/:dataTableId/rows` | Update rows | Update result | `modules/data-table/data-table.controller.ts:426` |
| DELETE | `/rest/projects/:projectId/data-tables/:dataTableId/rows` | Delete rows | Delete result | `modules/data-table/data-table.controller.ts:479` |
| POST | `/rest/projects/:projectId/data-tables/:dataTableId/columns` | Add column | Updated columns | `modules/data-table/data-table.controller.ts:202` |
| DELETE | `/rest/projects/:projectId/data-tables/:dataTableId/columns/:columnId` | Delete column | Result | `modules/data-table/data-table.controller.ts:218` |
| PATCH | `/rest/projects/:projectId/data-tables/:dataTableId/columns/:columnId/move` | Reorder column | Updated columns | `modules/data-table/data-table.controller.ts:234` |
| PATCH | `/rest/projects/:projectId/data-tables/:dataTableId/columns/:columnId/rename` | Rename column | Updated columns | `modules/data-table/data-table.controller.ts:256` |
| POST | `/api/v1/data-tables` | Create data table (Public API) | Data table (201) | `public-api/v1/handlers/data-tables/data-tables.handler.ts:138` |
| PATCH | `/api/v1/data-tables/:dataTableId` | Update data table (Public API) | Data table | `public-api/v1/handlers/data-tables/data-tables.handler.ts:194` |
| DELETE | `/api/v1/data-tables/:dataTableId` | Delete data table (Public API) | 204 | `public-api/v1/handlers/data-tables/data-tables.handler.ts:230` |
| POST | `/api/v1/data-tables/:dataTableId/rows` | Insert rows (Public API) | Rows | `public-api/v1/handlers/data-tables/data-tables.rows.handler.ts:112` |
| PATCH | `/api/v1/data-tables/:dataTableId/rows` | Update rows (Public API) | Result | `public-api/v1/handlers/data-tables/data-tables.rows.handler.ts:143` |
| POST | `/api/v1/data-tables/:dataTableId/rows/upsert` | Upsert rows (Public API) | Result | `public-api/v1/handlers/data-tables/data-tables.rows.handler.ts:175` |
| DELETE | `/api/v1/data-tables/:dataTableId/rows` | Delete rows (Public API) | Result | `public-api/v1/handlers/data-tables/data-tables.rows.handler.ts:207` |
| POST | `/rest/projects/:projectId/folders/` | Create folder | Folder entity | `controllers/folder.controller.ts:58` |
| PATCH | `/rest/projects/:projectId/folders/:folderId` | Update folder | void | `controllers/folder.controller.ts:123` |
| DELETE | `/rest/projects/:projectId/folders/:folderId` | Delete folder | void | `controllers/folder.controller.ts:145` |
| PUT | `/rest/projects/:projectId/folders/:folderId/transfer` | Transfer folder | Transfer result | `controllers/folder.controller.ts:207` |
| POST | `/rest/tags/` | Create tag | Tag entity | `controllers/tags.controller.ts:28` |
| PATCH | `/rest/tags/:id` | Update tag | Tag entity | `controllers/tags.controller.ts:41` |
| DELETE | `/rest/tags/:id` | Delete tag | `true` | `controllers/tags.controller.ts:54` |
| POST | `/api/v1/tags` | Create tag (Public API) | Tag (201) | `public-api/v1/handlers/tags/tags.handler.ts:18` |
| PATCH | `/api/v1/tags/:id` | Update tag (Public API) | Tag | `public-api/v1/handlers/tags/tags.handler.ts:33` |
| DELETE | `/api/v1/tags/:id` | Delete tag (Public API) | Tag | `public-api/v1/handlers/tags/tags.handler.ts:55` |
| POST | `/rest/annotation-tags/` | Create annotation tag | Tag entity | `controllers/annotation-tags.controller.ee.ts:18` |
| PATCH | `/rest/annotation-tags/:id` | Update annotation tag | Tag entity | `controllers/annotation-tags.controller.ee.ts:26` |
| DELETE | `/rest/annotation-tags/:id` | Delete annotation tag | `true` | `controllers/annotation-tags.controller.ee.ts:37` |
| PUT | `/api/v1/workflows/:id/tags` | Update workflow tags (Public API) | Tags array | `public-api/v1/handlers/workflows/workflows.handler.ts:408` |
| PUT | `/api/v1/executions/:id/tags` | Update execution tags (Public API) | Tags array | `public-api/v1/handlers/executions/executions.handler.ts:226` |
| POST | `/api/v1/users` | Create/invite user (Public API) | Invited user | `public-api/v1/handlers/users/users.handler.ee.ts:83` |
| DELETE | `/api/v1/users/:id` | Delete user (Public API) | 204 | `public-api/v1/handlers/users/users.handler.ee.ts:99` |
| PATCH | `/api/v1/users/:id/role` | Change user role (Public API) | 204 | `public-api/v1/handlers/users/users.handler.ee.ts:107` |
| POST | `/rest/external-secrets/providers/:provider/connect` | Toggle provider connection | `{}` | `modules/external-secrets.ee/external-secrets.controller.ee.ts:65` |
| POST | `/rest/external-secrets/providers/:provider/update` | Sync secrets from provider | `{ updated: boolean }` | `modules/external-secrets.ee/external-secrets.controller.ee.ts:73` |
| POST | `/rest/secret-providers/connections/` | Create secret provider connection | SecretProviderConnection | `modules/external-secrets.ee/secrets-providers-connections.controller.ee.ts:79` |
| PATCH | `/rest/secret-providers/connections/:providerKey` | Update connection | SecretProviderConnection | `modules/external-secrets.ee/secrets-providers-connections.controller.ee.ts:94` |
| DELETE | `/rest/secret-providers/connections/:providerKey` | Delete connection | 204 | `modules/external-secrets.ee/secrets-providers-connections.controller.ee.ts:111` |
| POST | `/rest/secret-providers/projects/:projectId/connections` | Create project secret connection | SecretProviderConnection | `modules/external-secrets.ee/secrets-providers-project.controller.ee.ts:53` |
| PATCH | `/rest/secret-providers/projects/:projectId/connections/:providerKey` | Update project connection | SecretProviderConnection | `modules/external-secrets.ee/secrets-providers-project.controller.ee.ts:103` |
| DELETE | `/rest/secret-providers/projects/:projectId/connections/:providerKey` | Delete project connection | 204 | `modules/external-secrets.ee/secrets-providers-project.controller.ee.ts:123` |
| POST | `/rest/workflows/:workflowId/test-runs/new` | Start test run | `{ success: true }` (202) | `evaluation.ee/test-runs.controller.ee.ts:105` |
| DELETE | `/rest/workflows/:workflowId/test-runs/:id` | Delete test run | `{ success: true }` | `evaluation.ee/test-runs.controller.ee.ts:74` |
| POST | `/rest/workflows/:workflowId/test-runs/:id/cancel` | Cancel test run | `{ success: true }` (202) | `evaluation.ee/test-runs.controller.ee.ts:88` |
| PATCH | `/rest/mcp/workflows/:workflowId/toggle-access` | Toggle MCP workflow access | `{ id, settings, versionId }` | `modules/mcp/mcp.settings.controller.ts:92` |
| POST | `/rest/ai/build` | AI workflow builder (streaming) | JSON-lines stream of AI chunks | `controllers/ai.controller.ts:48` |
| POST | `/rest/ai/chat` | AI chat assistance (streaming) | JSON-lines stream | `controllers/ai.controller.ts:131` |
| POST | `/rest/ai/chat/apply-suggestion` | Apply AI suggestion | ApplySuggestionResponse | `controllers/ai.controller.ts:167` |
| POST | `/rest/ai/usage-settings` | Update AI usage settings | void | `controllers/ai.controller.ts:310` |
| POST | `/rest/chat/conversations/send` | Send chat message | `{ status: 'streaming' }` | `modules/chat-hub/chat-hub.controller.ts:136` |
| POST | `/rest/chat/conversations/:sessionId/messages/:messageId/edit` | Edit chat message | `{ status: 'streaming' }` | `modules/chat-hub/chat-hub.controller.ts:157` |
| POST | `/rest/chat/conversations/:sessionId/messages/:messageId/regenerate` | Regenerate AI response | `{ status: 'streaming' }` | `modules/chat-hub/chat-hub.controller.ts:181` |
| PATCH | `/rest/chat/conversations/:sessionId` | Update conversation | ChatHubConversationResponse | `modules/chat-hub/chat-hub.controller.ts:238` |
| DELETE | `/rest/chat/conversations/:sessionId` | Delete conversation | 204 | `modules/chat-hub/chat-hub.controller.ts:253` |
| POST | `/rest/chat/tools` | Create chat tool | Tool DTO | `modules/chat-hub/chat-hub.controller.ts:272` |
| PATCH | `/rest/chat/tools/:toolId` | Update chat tool | Tool DTO | `modules/chat-hub/chat-hub.controller.ts:284` |
| DELETE | `/rest/chat/tools/:toolId` | Delete chat tool | 204 | `modules/chat-hub/chat-hub.controller.ts:299` |
| POST | `/rest/chat/agents` | Create chat agent | Agent DTO | `modules/chat-hub/chat-hub.controller.ts:322` |
| POST | `/rest/chat/agents/:agentId` | Update chat agent | Agent DTO | `modules/chat-hub/chat-hub.controller.ts:332` |
| DELETE | `/rest/chat/agents/:agentId` | Delete chat agent | 204 | `modules/chat-hub/chat-hub.controller.ts:343` |
| POST | `/rest/chat/settings` | Update chat provider settings | Provider settings | `modules/chat-hub/chat-hub.settings.controller.ts:46` |
| POST | `/rest/data-tables/uploads/` | Upload CSV | `{ originalName, id, rowCount, columnCount, columns }` | `modules/data-table/data-table-uploads.controller.ts:17` |
| POST | `/rest/credential-resolvers/` | Create credential resolver | CredentialResolver | `modules/dynamic-credentials.ee/credential-resolvers.controller.ts:62` |
| PATCH | `/rest/credential-resolvers/:id` | Update credential resolver | CredentialResolver | `modules/dynamic-credentials.ee/credential-resolvers.controller.ts:108` |
| DELETE | `/rest/credential-resolvers/:id` | Delete credential resolver | `{ success: true }` | `modules/dynamic-credentials.ee/credential-resolvers.controller.ts:140` |
| POST | `/rest/breaking-changes/report/refresh` | Refresh breaking changes cache | Report result | `modules/breaking-changes/breaking-changes.controller.ts:46` |
| POST | `/api/v1/audit` | Generate security audit | Audit results | `public-api/v1/handlers/audit/audit.handler.ts:9` |

### LOW

| Method | Path Pattern | What It Does | Response Contains | Source File |
|--------|-------------|-------------|-------------------|------------|
| GET | `/rest/credentials/for-workflow` | Credentials usable in workflow | `{ id, name, type, scopes }` — no secrets | `credentials/credentials.controller.ts:89` |
| GET | `/rest/credentials/new` | Generate unique credential name | `{ name }` | `credentials/credentials.controller.ts:97` |
| GET | `/api/v1/credentials` | List credentials (Public API) | id, name, type, timestamps — no data/secrets | `public-api/v1/handlers/credentials/credentials.handler.ts:42` |
| GET | `/api/v1/credentials/schema/:credentialTypeName` | Get credential type schema | JSON Schema (field definitions, not values) | `public-api/v1/handlers/credentials/credentials.handler.ts:203` |
| GET | `/rest/workflows/new` | Generate unique workflow name | `{ name }` | `workflows/workflows.controller.ts:304` |
| GET | `/rest/workflows/:workflowId/exists` | Check if workflow exists | `{ exists: boolean }` | `workflows/workflows.controller.ts:442` |
| GET | `/rest/workflows/:workflowId/collaboration/write-lock` | Get write lock info | Write lock object | `workflows/workflows.controller.ts:503` |
| GET | `/rest/active-workflows/` | List active workflow IDs | Array of ID strings | `controllers/active-workflows.controller.ts:10` |
| GET | `/rest/active-workflows/error/:id` | Get activation error | Error details or null | `controllers/active-workflows.controller.ts:15` |
| GET | `/rest/workflow-stats/:id/counts/` | Get execution counts | `{ productionSuccess, productionError, manualSuccess, manualError }` | `controllers/workflow-statistics.controller.ts:53` |
| GET | `/rest/workflow-stats/:id/times/` | Get execution times | Timestamps | `controllers/workflow-statistics.controller.ts:58` |
| GET | `/rest/workflow-stats/:id/data-loaded/` | Check data loaded | `{ dataLoaded: boolean }` | `controllers/workflow-statistics.controller.ts:63` |
| GET | `/api/v1/workflows/:id/tags` | Get workflow tags (Public API) | Tag array | `public-api/v1/handlers/workflows/workflows.handler.ts:381` |
| GET | `/api/v1/executions/:id/tags` | Get execution tags (Public API) | Tag array | `public-api/v1/handlers/executions/executions.handler.ts:203` |
| GET | `/rest/tags/` | List tags | Tag array with optional usage count | `controllers/tags.controller.ts:23` |
| GET | `/api/v1/tags` | List tags (Public API) | Tag array with cursor | `public-api/v1/handlers/tags/tags.handler.ts:71` |
| GET | `/api/v1/tags/:id` | Get tag (Public API) | Tag entity | `public-api/v1/handlers/tags/tags.handler.ts:94` |
| GET | `/rest/annotation-tags/` | List annotation tags | Tag array | `controllers/annotation-tags.controller.ee.ts:10` |
| POST | `/rest/node-types/` | Get node type descriptions | INodeTypeDescription array | `controllers/node-types.controller.ts:36` |
| POST | `/rest/node-types/by-identifier` | Get node types by identifier | INodeTypeDescription array | `controllers/node-types.controller.ts:54` |
| POST | `/rest/dynamic-node-parameters/options` | Get dynamic node parameter options | INodePropertyOptions array | `controllers/dynamic-node-parameters.controller.ts:18` |
| POST | `/rest/dynamic-node-parameters/resource-locator-results` | Resource locator search | Resource locator results | `controllers/dynamic-node-parameters.controller.ts:67` |
| POST | `/rest/dynamic-node-parameters/resource-mapper-fields` | Resource mapper fields | Resource mapper fields | `controllers/dynamic-node-parameters.controller.ts:105` |
| POST | `/rest/dynamic-node-parameters/local-resource-mapper-fields` | Local resource mapper fields | Local resource fields | `controllers/dynamic-node-parameters.controller.ts:133` |
| POST | `/rest/dynamic-node-parameters/action-result` | Node action handler | NodeParameterValueType | `controllers/dynamic-node-parameters.controller.ts:157` |
| GET | `/rest/community-packages/` | List installed packages | Package list with versions | `modules/community-packages/community-packages.controller.ts:165` |
| GET | `/rest/community-node-types/:name` | Get community node type | CommunityNodeType or null | `modules/community-packages/community-node-types.controller.ts:11` |
| GET | `/rest/community-node-types/` | List community node types | CommunityNodeType array | `modules/community-packages/community-node-types.controller.ts:16` |
| GET | `/rest/projects/count` | Get project counts | Count data | `controllers/project.controller.ts:54` |
| GET | `/rest/projects/my-projects` | Get user's projects | Projects with role and scopes | `controllers/project.controller.ts:96` |
| GET | `/rest/projects/personal` | Get personal project | Project with scopes | `controllers/project.controller.ts:158` |
| GET | `/rest/projects/:projectId/data-tables/` | List data tables | `{ data, count }` | `modules/data-table/data-table.controller.ts:123` |
| GET | `/rest/projects/:projectId/data-tables/:dataTableId/columns` | Get columns | Column definitions | `modules/data-table/data-table.controller.ts:182` |
| GET | `/api/v1/data-tables` | List data tables (Public API) | `{ data, nextCursor }` | `public-api/v1/handlers/data-tables/data-tables.handler.ts:71` |
| GET | `/api/v1/data-tables/:dataTableId` | Get data table (Public API) | Data table with columns | `public-api/v1/handlers/data-tables/data-tables.handler.ts:167` |
| GET | `/api/v1/data-tables/:dataTableId/rows` | Get rows (Public API) | `{ data, nextCursor }` | `public-api/v1/handlers/data-tables/data-tables.rows.handler.ts:68` |
| GET | `/rest/data-tables-global/` | List all data tables globally | `{ data, count }` | `modules/data-table/data-table-aggregate.controller.ts:15` |
| GET | `/rest/data-tables-global/limits` | Get data tables limits | Size/limit info | `modules/data-table/data-table-aggregate.controller.ts:25` |
| GET | `/rest/projects/:projectId/folders/:folderId/tree` | Get folder tree | Tree array | `controllers/folder.controller.ts:79` |
| GET | `/rest/projects/:projectId/folders/:folderId/credentials` | Get folder credentials | Credential references | `controllers/folder.controller.ts:99` |
| GET | `/rest/projects/:projectId/folders/` | List folders | `{ count, data }` | `controllers/folder.controller.ts:167` |
| GET | `/rest/projects/:projectId/folders/:folderId/content` | Get folder content | `{ totalSubFolders, totalWorkflows }` | `controllers/folder.controller.ts:183` |
| GET | `/rest/roles/` | List roles | Role definitions grouped by type | `controllers/role.controller.ts:33` |
| GET | `/rest/roles/:slug` | Get role | Role DTO | `controllers/role.controller.ts:70` |
| GET | `/rest/roles/:slug/assignments` | Get role assignments | Assignment data | `controllers/role.controller.ts:59` |
| GET | `/rest/roles/:slug/assignments/:projectId/members` | Get project members by role | Member list | `controllers/role.controller.ts:47` |
| GET | `/rest/license/` | Get license data | Plan name, usage counts | `license/license.controller.ts:21` |
| GET | `/rest/settings/security/` | Get security settings | Personal space publishing/sharing counts | `controllers/security-settings.controller.ts:22` |
| GET | `/rest/source-control/preferences` | Get source control prefs | Preferences (privateKey redacted), repositoryUrl, branchName | `modules/source-control.ee/source-control.controller.ee.ts:37` |
| GET | `/rest/source-control/get-branches` | List git branches | Branch name array | `modules/source-control.ee/source-control.controller.ee.ts:164` |
| GET | `/rest/source-control/get-status` | Get file status | File status array | `modules/source-control.ee/source-control.controller.ee.ts:234` |
| GET | `/rest/source-control/status` | Alias for get-status | Same | `modules/source-control.ee/source-control.controller.ee.ts:247` |
| GET | `/rest/source-control/remote-content/:type/:id` | Get remote file content | `{ content, type }` | `modules/source-control.ee/source-control.controller.ee.ts:274` |
| GET | `/rest/external-secrets/secrets` | List secret names | Secret names only (not values) | `modules/external-secrets.ee/external-secrets.controller.ee.ts:88` |
| POST | `/rest/external-secrets/providers/:provider/test` | Test secret provider | `{ success: boolean }` | `modules/external-secrets.ee/external-secrets.controller.ee.ts:45` |
| GET | `/rest/secret-providers/types/` | List provider types | SecretProviderTypeResponse array | `modules/external-secrets.ee/secrets-providers-types.controller.ee.ts:40` |
| GET | `/rest/secret-providers/types/:type` | Get provider type schema | SecretProviderTypeResponse | `modules/external-secrets.ee/secrets-providers-types.controller.ee.ts:50` |
| GET | `/rest/secret-providers/connections/` | List connections | SecretProviderConnectionListItem array | `modules/external-secrets.ee/secrets-providers-connections.controller.ee.ts:124` |
| GET | `/rest/secret-providers/connections/:providerKey` | Get connection | SecretProviderConnection | `modules/external-secrets.ee/secrets-providers-connections.controller.ee.ts:134` |
| POST | `/rest/secret-providers/connections/:providerKey/test` | Test connection | TestSecretProviderConnectionResponse | `modules/external-secrets.ee/secrets-providers-connections.controller.ee.ts:146` |
| POST | `/rest/secret-providers/connections/:providerKey/reload` | Reload secrets | ReloadSecretProviderConnectionResponse | `modules/external-secrets.ee/secrets-providers-connections.controller.ee.ts:157` |
| GET | `/rest/secret-providers/projects/:projectId/connections` | List project connections | SecretProviderConnectionListItem array | `modules/external-secrets.ee/secrets-providers-project.controller.ee.ts:75` |
| GET | `/rest/secret-providers/projects/:projectId/connections/:providerKey` | Get project connection | SecretProviderConnection | `modules/external-secrets.ee/secrets-providers-project.controller.ee.ts:87` |
| POST | `/rest/secret-providers/projects/:projectId/connections/:providerKey/test` | Test project connection | TestSecretProviderConnectionResponse | `modules/external-secrets.ee/secrets-providers-project.controller.ee.ts:136` |
| POST | `/rest/secret-providers/projects/:projectId/reload` | Reload project secrets | ReloadSecretProviderConnectionResponse | `modules/external-secrets.ee/secrets-providers-project.controller.ee.ts:149` |
| GET | `/rest/secret-providers/completions/secrets/global` | Global secret name completions | SecretCompletionsResponse | `modules/external-secrets.ee/secrets-providers-completions.controller.ee.ts:53` |
| GET | `/rest/secret-providers/completions/secrets/project/:projectId` | Project secret name completions | SecretCompletionsResponse | `modules/external-secrets.ee/secrets-providers-completions.controller.ee.ts:62` |
| GET | `/rest/workflows/:workflowId/test-runs` | List test runs | Paginated test run summaries | `evaluation.ee/test-runs.controller.ee.ts:45` |
| GET | `/rest/workflows/:workflowId/test-runs/:id` | Get test run | Test run summary | `evaluation.ee/test-runs.controller.ee.ts:52` |
| GET | `/rest/workflows/:workflowId/test-runs/:id/test-cases` | Get test cases | Test case executions | `evaluation.ee/test-runs.controller.ee.ts:65` |
| GET | `/rest/insights/summary` | Get insights summary | InsightsSummary | `modules/insights/insights.controller.ts:24` |
| GET | `/rest/insights/by-workflow` | Insights per workflow | InsightsByWorkflow | `modules/insights/insights.controller.ts:40` |
| GET | `/rest/insights/by-time` | Insights over time | InsightsByTime array | `modules/insights/insights.controller.ts:60` |
| GET | `/rest/insights/by-time/time-saved` | Time-saved insights | RestrictedInsightsByTime array | `modules/insights/insights.controller.ts:83` |
| GET | `/rest/eventbus/eventnames` | List event names | Event name strings | `modules/log-streaming.ee/log-streaming.controller.ts:34` |
| GET | `/rest/eventbus/testmessage` | Send test log message | boolean | `modules/log-streaming.ee/log-streaming.controller.ts:97` |
| GET | `/rest/mcp/workflows` | List MCP-eligible workflows | `{ count, data }` | `modules/mcp/mcp.settings.controller.ts:68` |
| GET | `/rest/mcp/oauth-clients/` | List MCP OAuth clients | ListOAuthClientsResponseDto | `modules/mcp/mcp.oauth-clients.controller.ts:27` |
| GET | `/rest/credential-resolvers/` | List credential resolvers | CredentialResolver array | `modules/dynamic-credentials.ee/credential-resolvers.controller.ts:35` |
| GET | `/rest/credential-resolvers/types` | List resolver types | CredentialResolverType array | `modules/dynamic-credentials.ee/credential-resolvers.controller.ts:48` |
| GET | `/rest/credential-resolvers/:id` | Get credential resolver | CredentialResolver | `modules/dynamic-credentials.ee/credential-resolvers.controller.ts:88` |
| GET | `/rest/ldap/sync` | Get LDAP sync history | Sync history records | `modules/ldap.ee/ldap.controller.ee.ts:57` |
| POST | `/rest/ldap/test-connection` | Test LDAP connection | void or error | `modules/ldap.ee/ldap.controller.ee.ts:26` |
| POST | `/rest/ldap/sync` | Trigger LDAP sync | void | `modules/ldap.ee/ldap.controller.ee.ts:65` |
| POST | `/rest/ai/ask-ai` | Ask AI about node/expression | AskAiResponsePayload | `controllers/ai.controller.ts:181` |
| GET | `/rest/ai/sessions` | Get AI builder sessions | Session array | `controllers/ai.controller.ts:240` |
| GET | `/rest/ai/build/credits` | Get AI credit balance | Credits response | `controllers/ai.controller.ts:260` |
| POST | `/rest/ai/build/truncate-messages` | Truncate AI messages | `{ success: boolean }` | `controllers/ai.controller.ts:274` |
| POST | `/rest/ai/build/clear-session` | Clear AI session | `{ success: boolean }` | `controllers/ai.controller.ts:294` |
| GET | `/rest/chat/conversations` | List conversations | ChatHubConversationsResponse | `modules/chat-hub/chat-hub.controller.ts:69` |
| GET | `/rest/chat/conversations/:sessionId` | Get conversation | ChatHubConversationResponse | `modules/chat-hub/chat-hub.controller.ts:78` |
| GET | `/rest/chat/conversations/:sessionId/messages/:messageId/attachments/:index` | Get attachment | Binary file stream | `modules/chat-hub/chat-hub.controller.ts:88` |
| POST | `/rest/chat/conversations/:sessionId/messages/:messageId/stop` | Stop AI generation | 204 | `modules/chat-hub/chat-hub.controller.ts:207` |
| POST | `/rest/chat/conversations/:sessionId/reconnect` | Reconnect chat stream | ChatReconnectResponse | `modules/chat-hub/chat-hub.controller.ts:224` |
| POST | `/rest/chat/models` | Get available chat models | ChatModelsResponse | `modules/chat-hub/chat-hub.controller.ts:58` |
| GET | `/rest/chat/tools` | List chat tools | Tool DTO array | `modules/chat-hub/chat-hub.controller.ts:265` |
| GET | `/rest/chat/agents` | List chat agents | Agent DTO array | `modules/chat-hub/chat-hub.controller.ts:310` |
| GET | `/rest/chat/agents/:agentId` | Get chat agent | Agent DTO | `modules/chat-hub/chat-hub.controller.ts:316` |
| GET | `/rest/chat/settings` | Get chat settings | `{ providers }` | `modules/chat-hub/chat-hub.settings.controller.ts:23` |
| GET | `/rest/chat/settings/:provider` | Get provider settings | `{ settings }` | `modules/chat-hub/chat-hub.settings.controller.ts:30` |
| GET | `/rest/breaking-changes/report` | Get breaking changes report | Report result | `modules/breaking-changes/breaking-changes.controller.ts:34` |
| GET | `/rest/breaking-changes/report/:ruleId` | Get specific rule results | Rule result | `modules/breaking-changes/breaking-changes.controller.ts:62` |
| POST | `/rest/webhooks/find` | Find webhook by method+path | Webhook entity or null | `webhooks/webhooks.controller.ts:12` |
| GET | `/rest/module-settings/` | Get module settings | Module settings | `controllers/module-settings.controller.ts:12` |
| GET | `/rest/dynamic-templates/` | Fetch dynamic templates | `{ templates }` | `controllers/dynamic-templates.controller.ts:11` |
| GET | `/rest/cta/become-creator` | Check become-creator CTA | Boolean | `controllers/cta.controller.ts:15` |
| GET | `/rest/binary-data/` | Stream binary data | Binary file stream | `controllers/binary-data.controller.ts:13` |
| GET | `/rest/api-keys/scopes` | Get available API key scopes | Scope string array | `controllers/api-keys.controller.ts:119` |
| POST | `/rest/mfa/can-enable` | Check if MFA can be enabled | void | `controllers/mfa.controller.ts:56` |
| POST | `/rest/mfa/verify` | Verify MFA code | void | `controllers/mfa.controller.ts:191` |
| GET | `/rest/orchestration/worker/status` | Request worker status | Worker status | `controllers/orchestration.controller.ts:19` |

### NONE

| Method | Path Pattern | What It Does | Response Contains | Source File |
|--------|-------------|-------------|-------------------|------------|
| GET | `/healthz` | Health check | `{ status: 'ok' }` | `abstract-server.ts:134` |
| GET | `/healthz/readiness` | Readiness check | `{ status: 'ok' }` or 503 | `abstract-server.ts:140` |
| GET | `/rest/settings` (unauthed) | Public frontend settings | Locale, auth method, SSO status, setup needed | `server.ts:465` |
| POST | `/rest/login` | Log in | PublicUser (no password/mfaSecret) | `controllers/auth.controller.ts:53` |
| GET | `/rest/login` | Check session | PublicUser or empty | `controllers/auth.controller.ts:184` |
| POST | `/rest/logout` | Log out | `{ loggedOut: true }` | `controllers/auth.controller.ts:265` |
| GET | `/rest/resolve-signup-token` | Validate invite token | `{ inviter: { firstName, lastName } }` | `controllers/auth.controller.ts:199` |
| POST | `/rest/invitations/accept` | Accept invitation | PublicUser | `controllers/invitation.controller.ts:165` |
| POST | `/rest/invitations/:id/accept` | Accept invitation (legacy) | PublicUser | `controllers/invitation.controller.ts:211` |
| POST | `/rest/forgot-password` | Request password reset | void (no email disclosure) | `controllers/password-reset.controller.ts:61` |
| GET | `/rest/resolve-password-token` | Validate reset token | void | `controllers/password-reset.controller.ts:169` |
| POST | `/rest/change-password` | Change password via token | void | `controllers/password-reset.controller.ts:194` |
| POST | `/rest/owner/setup` | Instance owner setup | PublicUser (first-time only) | `controllers/owner.controller.ts:25` |
| POST | `/rest/owner/dismiss-banner` | Dismiss banner | void | `controllers/owner.controller.ts:32` |
| POST | `/rest/me/survey` | Save survey answers | `{ success: true }` | `controllers/me.controller.ts:233` |
| PATCH | `/rest/user-settings/nps-survey` | Update NPS survey | void | `controllers/user-settings.controller.ts:42` |
| GET | `/rest/events/session-started` | Record session start | void | `events/events.controller.ts:11` |
| GET | `/rest/credential-translation` | Credential i18n | Translation JSON | `controllers/translation.controller.ts:26` |
| GET | `/rest/node-translation-headers` | Node i18n headers | Headers JSON | `controllers/translation.controller.ts:48` |
| GET | `/rest/third-party-licenses/` | Third-party licenses | License markdown text | `controllers/third-party-licenses.controller.ts:13` |
| POST | `/rest/license/enterprise/request_trial` | Request trial | void | `license/license.controller.ts:26` |
| POST | `/rest/license/enterprise/community-registered` | Register community | Registration result | `license/license.controller.ts:43` |
| GET | `/rest/sso/saml/metadata` | SAML SP metadata (public) | XML metadata | `modules/sso-saml/saml.controller.ee.ts:40` |
| GET | `/rest/sso/saml/acs` | SAML ACS (GET) | Redirect/HTML | `modules/sso-saml/saml.controller.ee.ts:86` |
| POST | `/rest/sso/saml/acs` | SAML ACS (POST) | Redirect/HTML | `modules/sso-saml/saml.controller.ee.ts:94` |
| GET | `/rest/sso/saml/initsso` | Initiate SAML SSO | Redirect | `modules/sso-saml/saml.controller.ee.ts:171` |
| POST | `/rest/sso/saml/config/test` | Test SAML config | Redirect URL | `modules/sso-saml/saml.controller.ee.ts:197` |
| GET | `/rest/sso/oidc/login` | OIDC login redirect | HTTP redirect | `modules/sso-oidc/oidc.controller.ee.ts:52` |
| GET | `/rest/sso/oidc/callback` | OIDC callback | Sets cookie, redirects | `modules/sso-oidc/oidc.controller.ee.ts:73` |
| GET | `/rest/oauth1-credential/auth` | OAuth1 auth URL | Authorization URL | `controllers/oauth/oauth1-credential.controller.ts:23` |
| GET | `/rest/oauth1-credential/callback` | OAuth1 callback | HTML template | `controllers/oauth/oauth1-credential.controller.ts:42` |
| GET | `/rest/oauth2-credential/auth` | OAuth2 auth URL | Authorization URL | `controllers/oauth/oauth2-credential.controller.ts:25` |
| GET | `/rest/oauth2-credential/callback` | OAuth2 callback | HTML template | `controllers/oauth/oauth2-credential.controller.ts:38` |
| POST | `/rest/telemetry/proxy/:version/track` | Telemetry proxy (no auth) | Proxied response | `controllers/telemetry.controller.ts:28` |
| POST | `/rest/telemetry/proxy/:version/identify` | Telemetry proxy (no auth) | Proxied response | `controllers/telemetry.controller.ts:33` |
| POST | `/rest/telemetry/proxy/:version/page` | Telemetry proxy (no auth) | Proxied response | `controllers/telemetry.controller.ts:38` |
| GET | `/rest/telemetry/rudderstack/sourceConfig` | RudderStack config (no auth) | Config JSON | `controllers/telemetry.controller.ts:42` |
| ALL | `/rest/ph/*` | PostHog proxy (no auth) | Proxied response | `controllers/posthog.controller.ts:8` |
| GET | `/rest/binary-data/signed` | Signed binary data (no auth, JWT required) | Binary stream | `controllers/binary-data.controller.ts:30` |
| GET | `/.well-known/oauth-authorization-server` | OAuth metadata (no auth) | OAuth metadata JSON | `modules/mcp/mcp.oauth.controller.ts:65` |
| GET | `/.well-known/oauth-protected-resource/mcp-server/http` | Protected resource metadata | Resource metadata | `modules/mcp/mcp.oauth.controller.ts:100` |
| ALL | `/mcp-oauth/register` | MCP OAuth registration | Client registration | `modules/mcp/mcp.oauth.controller.ts:31` |
| ALL | `/mcp-oauth/authorize` | MCP OAuth authorization | Auth response | `modules/mcp/mcp.oauth.controller.ts:36` |
| ALL | `/mcp-oauth/token` | MCP OAuth token exchange | Token response | `modules/mcp/mcp.oauth.controller.ts:41` |
| ALL | `/mcp-oauth/revoke` | MCP OAuth revocation | Revocation response | `modules/mcp/mcp.oauth.controller.ts:47` |
| HEAD | `/mcp-server/http` | MCP auth discovery | 401 with WWW-Authenticate | `modules/mcp/mcp.controller.ts:60` |
| POST | `/mcp-server/http` | MCP JSON-RPC (OAuth auth) | JSON-RPC responses | `modules/mcp/mcp.controller.ts:70` |
| GET | `/rest/consent/details` | OAuth consent details | `{ clientName, clientId }` | `modules/mcp/mcp.auth.consent.controller.ts:18` |
| POST | `/rest/consent/approve` | OAuth consent approval | `{ status, redirectUrl }` | `modules/mcp/mcp.auth.consent.controller.ts:44` |
| OPTIONS | `/rest/credentials/:id/revoke` | CORS preflight | Headers | `modules/dynamic-credentials.ee/dynamic-credentials.controller.ts:72` |
| DELETE | `/rest/credentials/:id/revoke` | Revoke dynamic credential (external auth) | 204 | `modules/dynamic-credentials.ee/dynamic-credentials.controller.ts:77` |
| OPTIONS | `/rest/credentials/:id/authorize` | CORS preflight | Headers | `modules/dynamic-credentials.ee/dynamic-credentials.controller.ts:109` |
| POST | `/rest/credentials/:id/authorize` | Authorize dynamic credential (external auth) | Auth URL | `modules/dynamic-credentials.ee/dynamic-credentials.controller.ts:114` |
| OPTIONS | `/rest/workflows/:workflowId/execution-status` | CORS preflight | Headers | `modules/dynamic-credentials.ee/workflow-status.controller.ts:28` |
| GET | `/rest/workflows/:workflowId/execution-status` | Workflow execution readiness (external auth) | Status + credential metadata | `modules/dynamic-credentials.ee/workflow-status.controller.ts:42` |
| PATCH | `/rest/e2e/feature` | Toggle E2E feature (no auth, E2E only) | void | `controllers/e2e.controller.ts:210` |
| PATCH | `/rest/e2e/quota` | Set E2E quotas (no auth, E2E only) | void | `controllers/e2e.controller.ts:216` |
| PATCH | `/rest/e2e/queue-mode` | Toggle E2E queue (no auth, E2E only) | Result | `controllers/e2e.controller.ts:222` |
| GET | `/rest/e2e/env-feature-flags` | Get E2E flags (no auth, E2E only) | Flags | `controllers/e2e.controller.ts:228` |
| PATCH | `/rest/e2e/env-feature-flags` | Set E2E flags (no auth, E2E only) | Result | `controllers/e2e.controller.ts:233` |
| POST | `/rest/e2e/push` | E2E push (no auth, E2E only) | void | `controllers/e2e.controller.ts:204` |
| POST | `/rest/e2e/gc` | E2E garbage collection (no auth, E2E only) | Result | `controllers/e2e.controller.ts:272` |

---

## Key Findings for Incident Analysis

### Most Dangerous Endpoints (if accessed by wrong user)

1. **`GET /rest/ldap/config`** — Returns LDAP admin password in plaintext. If accessed, attacker has directory service credentials.

2. **`GET /rest/credentials?includeData=true`** and **`GET /rest/credentials/:id?includeData=true`** — Returns non-password credential fields (hostnames, usernames, client IDs, regions) in plaintext. Password fields are blanked but still reveal infrastructure topology.

3. **`GET /rest/executions/:id`** — Execution data contains ALL node outputs from workflow runs. This is the most likely source of data exposure: API responses, database results, processed PII, tokens returned by external services.

4. **`GET /rest/variables/`** — Variable values are returned in plaintext. Users commonly store API keys, passwords, and connection strings as variables.

5. **`GET /rest/mfa/qr`** — TOTP secret and recovery codes. If accessed during another user's MFA setup window, attacker can clone their 2FA.

6. **`POST /rest/api-keys/`** — If called, creates a persistent API key for the victim's account. The raw key is returned and can be used even after the session incident is resolved.

7. **`POST /rest/source-control/push-workfolder`** — Can export all instance data to an attacker-controlled git repository.

8. **`GET /rest/debug/multi-main-setup`** — Unauthenticated. Exposes cluster topology to anyone.

### Credential Decryption Flow (for `includeData=true`)

```
Controller receives includeData=true
  → Service calls addDecryptedDataToCredentials()
    → Checks user has credential:update scope
      → YES: decrypt() with includeRawData=false
        → redact() replaces password fields with CREDENTIAL_BLANKING_VALUE
        → replaces oauthTokenData with boolean true
        → NON-PASSWORD FIELDS RETURNED IN PLAINTEXT
      → NO: data field is undefined (no decryption)
```

### Public API Execution Data (No Redaction)

The internal REST API uses `ExecutionRedactionServiceProxy` to optionally redact node output data. The Public API (`/api/v1/executions`) does NOT use this redaction service — execution data is returned unredacted when `includeData=true`.
