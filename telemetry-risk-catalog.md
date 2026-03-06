# n8n Telemetry Event Risk Catalog

## Context
Catalog of all RudderStack telemetry events (`rudder_schema.tracks.event_text`) in the n8n codebase, categorized by security risk in the context of an unauthorized session (user accidentally authenticated into another user's instance).

## Risk Categories
- **CRITICAL** — credential secrets exposed, data exfiltrated, or destructive action
- **HIGH** — sensitive data accessed but not necessarily exfiltrated
- **MEDIUM** — instance data modified or accessed
- **LOW** — read-only browsing, UI navigation, no data exposure
- **NONE** — system/infrastructure events with no security relevance

---

## Consolidated Risk Table

### CRITICAL

| Event Name | Trigger | What It Implies | Source |
|---|---|---|---|
| `User deleted credentials` | Credential deleted via API | Unauthorized user destroyed credential data | telemetry.event-relay.ts:561 |
| `User deleted workflow` | Workflow deleted | Unauthorized user destroyed workflow data | telemetry.event-relay.ts:696 |
| `User deleted user` | Admin deletes a user | Unauthorized user removed another user account | telemetry.event-relay.ts:1211 |
| `User deleted project` | Team project deleted | Unauthorized user destroyed project and its resources | telemetry.event-relay.ts:179 |
| `User changed role` | Admin changes user's instance role | Unauthorized user escalated/changed privileges | telemetry.event-relay.ts:1130 |
| `User updated instance policies` | Admin changes security settings (2FA enforcement, workflow sharing/publishing) | Unauthorized user weakened security policies | telemetry.event-relay.ts:1439 |
| `User updated single sign on settings` | Admin saves SAML/OIDC settings | Unauthorized user changed SSO config, potential account takeover vector | SamlSettingsForm.vue:127, OidcSettingsForm.vue:183 |
| `User updated Ldap settings` | Admin changes LDAP config | Unauthorized user changed auth backend config | telemetry.event-relay.ts:600 |
| `User updated external secrets settings` | External secrets provider settings saved | Unauthorized user accessed/modified vault credentials | telemetry.event-relay.ts:342 |
| `User created external secrets connection` | External secrets connection created | Unauthorized user connected a secrets vault | telemetry.event-relay.ts:364 |
| `User updated external secrets connection` | External secrets connection updated | Unauthorized user modified secrets vault connection | telemetry.event-relay.ts:377 |
| `User deleted external secrets connection` | External secrets connection deleted | Unauthorized user severed secrets vault connection | telemetry.event-relay.ts:390 |
| `User exported workflow` | Workflow JSON exported/downloaded | Unauthorized user exfiltrated workflow data (may contain credential references, logic, API URLs) | ActionsDropdownMenu.vue:290, useWorkflowCommands.ts:365 |
| `User downloaded version` | Workflow version downloaded from history | Unauthorized user exfiltrated historical workflow data | WorkflowHistory.vue:434 |
| `User updated provisioning settings` | SSO user role provisioning changed | Unauthorized user changed how roles are auto-assigned | useUserRoleProvisioningForm.ts:61 |
| `API key created` | Public API key created | Unauthorized user created persistent API access (backdoor) | telemetry.event-relay.ts:419 |
| `API key deleted` | Public API key deleted | Unauthorized user removed API key (covering tracks or denial of service) | telemetry.event-relay.ts:428 |
| `User invited new user` | Admin invites new user | Unauthorized user created an invitation (potential persistent access) | telemetry.event-relay.ts:1228 |
| `User updated log streaming destination` | Log streaming destination changed | Unauthorized user could redirect audit logs to attacker-controlled endpoint | EventDestinationSettingsModal.vue:383 |
| `User updated source control settings` | Source control settings changed | Unauthorized user modified git/source control config (potential code injection) | telemetry.event-relay.ts:207 |

### HIGH

| Event Name | Trigger | What It Implies | Source |
|---|---|---|---|
| `User created workflow` | Workflow created | Unauthorized user could create a workflow that uses existing credentials to exfiltrate data to attacker-controlled endpoint | telemetry.event-relay.ts:663 |
| `User saved workflow` | Workflow saved | Unauthorized user could modify existing workflow to exfiltrate credential-bound data | telemetry.event-relay.ts:778 |
| `User clicked execute workflow button` | Execute Workflow clicked | Unauthorized user triggered workflow execution — if workflow was crafted/modified, this is the exfiltration moment | useRunWorkflow.ts:567 |
| `User clicked execute node button` | Execute single node clicked | Unauthorized user triggered node execution using bound credentials | NodeView.vue:1113, useNodeExecution.ts:403 |
| `Manual node exec finished` | Manual single-node execution finished | Unauthorized user executed a node with credentials and saw results | telemetry.event-relay.ts:963 |
| `Manual workflow exec finished` | Manual workflow execution finished | Unauthorized user executed a workflow with credentials and saw results | telemetry.event-relay.ts:976 |
| `User set workflow active status` | Workflow activated/deactivated | Unauthorized user could activate a crafted workflow for persistent exfiltration | useWorkflowActivate.ts:188 |
| `User saved credentials` | Credential saved (non-OAuth or OAuth) | Unauthorized user modified credential values | CredentialEdit.vue:878,1180, useCredentialOAuth.ts:274 |
| `User created credentials` | New credential created (backend) | Unauthorized user added credentials to the instance | telemetry.event-relay.ts:513, CredentialEdit.vue:985 |
| `User updated credentials` | Credential updated via API | Unauthorized user modified existing credential values | telemetry.event-relay.ts:548 |
| `User updated cred sharing` | Credential sharing changed | Unauthorized user changed who can access credentials | telemetry.event-relay.ts:532 |
| `User retrieved all users` | API lists all users | Unauthorized user enumerated all user accounts | telemetry.event-relay.ts:1146 |
| `User retrieved user` | API retrieves single user | Unauthorized user accessed user profile data | telemetry.event-relay.ts:1139 |
| `User retrieved all workflows` | API lists all workflows | Unauthorized user enumerated all workflows | telemetry.event-relay.ts:1190 |
| `User retrieved workflow` | API retrieves single workflow | Unauthorized user accessed workflow definition | telemetry.event-relay.ts:1170 |
| `User retrieved workflow version` | API retrieves workflow version | Unauthorized user accessed historical workflow data | telemetry.event-relay.ts:1180 |
| `User retrieved execution` | API retrieves execution data | Unauthorized user accessed execution results (may contain processed data) | telemetry.event-relay.ts:1153 |
| `User retrieved all executions` | API lists all executions | Unauthorized user enumerated execution history | telemetry.event-relay.ts:1163 |
| `User changed personal settings` | User updates profile/password | Unauthorized user changed another user's password or profile | telemetry.event-relay.ts:1197 |
| `User clicked create API key button` | Create API key button clicked | Unauthorized user attempted to create API key | SettingsApiView.vue:47 |
| `User clicked delete API key button` | Delete API key button clicked | Unauthorized user attempted to delete API key | SettingsApiView.vue:102 |
| `User updated workflow sharing` | Workflow sharing changed | Unauthorized user changed workflow access controls | telemetry.event-relay.ts:708 |
| `User reloaded external secrets` | External secrets provider reloaded | Unauthorized user triggered secrets refresh | telemetry.event-relay.ts:354 |
| `User invoked API` | Any public API endpoint called | Unauthorized user accessed the API (path/method logged) | telemetry.event-relay.ts:408 |
| `User opened Executions log` | Executions log page opened | Unauthorized user viewed execution history (may reveal data flow) | ExecutionsView.vue:44 |
| `User opened read-only execution` | Execution details viewed | Unauthorized user viewed execution data | usePostMessageHandler.ts:111 |
| `User opened workflow history` | Workflow history panel opened | Unauthorized user browsed historical workflow versions | WorkflowHistory.vue:169 |
| `User cloned version` | Workflow version cloned | Unauthorized user copied historical workflow version | WorkflowHistory.vue:438 |
| `User restored version` | Workflow version restored | Unauthorized user restored a previous workflow version | WorkflowHistory.vue:442 |
| `User viewed worker view` | Admin views workers/orchestration | Unauthorized user saw infrastructure details | WorkerList.vue:44 |
| `User changed AI Usage settings` | AI data sharing settings changed | Unauthorized user changed data sharing policy | SettingsAIView.vue:67 |
| `User copied ndv data` | Data copied from NDV output | Unauthorized user copied execution/node output data | RunDataJsonActions.vue:181 |
| `User copied webhook URL` | Webhook URL copied | Unauthorized user obtained webhook endpoint URL | TriggerPanel.vue:374, NodeWebhooks.vue:186 |

### MEDIUM

| Event Name | Trigger | What It Implies | Source |
|---|---|---|---|
| `User archived workflow` | Workflow archived | Unauthorized user archived a workflow | telemetry.event-relay.ts:676 |
| `User unarchived workflow` | Workflow unarchived | Unauthorized user unarchived a workflow | telemetry.event-relay.ts:688 |
| `User clicked execute node button in modal` | Execute node in AI modal clicked | Unauthorized user triggered node execution from AI modal | FromAiParametersModal.vue:101 |
| `User duplicated workflow` | Workflow duplicated | Unauthorized user copied a workflow | DuplicateWorkflowDialog.vue:128 |
| `User imported workflow` | Workflow imported from file | Unauthorized user imported external workflow | useCanvasOperations.ts:2660 |
| `User inserted workflow template` | Template inserted into workflow | Unauthorized user added template content | useCanvasOperations.ts:3099, setupTemplate.store.ts:200 |
| `User pasted nodes` | Nodes pasted into canvas | Unauthorized user added nodes via paste | useCanvasOperations.ts:2650 |
| `User deleted node` | Node deleted from workflow | Unauthorized user modified workflow structure | useCanvasOperations.ts:519 |
| `User added node to workflow canvas` | Node added to canvas | Unauthorized user modified workflow | nodeCreator.store.ts:438 |
| `User set node enabled status` | Node enabled/disabled | Unauthorized user changed node behavior | useNodeHelpers.ts:733 |
| `User updated workflow settings` | Workflow settings saved | Unauthorized user changed workflow config | WorkflowSettings.vue:571 |
| `User clicked retry execution button` | Execution retry clicked | Unauthorized user re-ran past execution | useExecutionCommands.ts:163, etc. |
| `User clicked debug execution button` | Debug execution clicked | Unauthorized user accessed debug mode | useExecutionDebugging.ts:153 |
| `User confirmed stop many executions` | Bulk execution stop confirmed | Unauthorized user stopped running executions | StopManyExecutionsModal.vue:77 |
| `User created project` | Team project created | Unauthorized user created a project | telemetry.event-relay.ts:189 |
| `Project settings updated` | Project settings/members changed | Unauthorized user modified project config | telemetry.event-relay.ts:164 |
| `User added member to project` | Member added to project | Unauthorized user changed project membership | ProjectSettings.vue:159,306 |
| `User changed member role on project` | Member role changed | Unauthorized user changed project permissions | ProjectSettings.vue:195,321 |
| `User removed member from project` | Member removed from project | Unauthorized user removed project access | ProjectSettings.vue:233,314 |
| `User changed project name` | Project renamed | Unauthorized user modified project | ProjectSettings.vue:302 |
| `User selected sharee to add` | Sharee added to workflow | Unauthorized user expanded workflow access | WorkflowShareModal.ee.vue:139 |
| `User selected sharee to remove` | Sharee removed from workflow | Unauthorized user restricted workflow access | WorkflowShareModal.ee.vue:146 |
| `User published version from canvas` | Workflow version published | Unauthorized user published a workflow version | WorkflowPublishModal.vue:210 |
| `User published version from history` | Workflow published from history | Unauthorized user published historical version | WorkflowHistory.vue:315 |
| `User unpublished workflow from history` | Workflow unpublished | Unauthorized user deactivated published workflow | WorkflowHistory.vue:352 |
| `User created variable` | Variable created | Unauthorized user added instance variable | telemetry.event-relay.ts:311 |
| `User updated variable` | Variable updated | Unauthorized user modified instance variable | telemetry.event-relay.ts:318 |
| `User deleted variable` | Variable deleted | Unauthorized user deleted instance variable | telemetry.event-relay.ts:325 |
| `User started push via UI` | Source control push started | Unauthorized user pushed changes to git | telemetry.event-relay.ts:258 |
| `User finished push via UI` | Source control push finished | Unauthorized user completed push to git | telemetry.event-relay.ts:275 |
| `User started pull via UI` | Source control pull started | Unauthorized user pulled changes from git | telemetry.event-relay.ts:222 |
| `User finished pull via UI` | Source control pull finished | Unauthorized user completed pull from git | telemetry.event-relay.ts:234 |
| `User pulled via API` | Source control pull via API | Unauthorized user pulled via API | telemetry.event-relay.ts:244 |
| `User gave MCP access to workflow` | MCP access granted to workflow | Unauthorized user exposed workflow via MCP | useMcp.ts:39 |
| `User toggled MCP access` | MCP access toggled | Unauthorized user changed MCP settings | useMcp.ts:43 |
| `User connected to MCP server` | MCP connection established | Unauthorized user connected via MCP | mcp.constants.ts:9 |
| `User called mcp tool` | MCP tool invoked | Unauthorized user used MCP tools | mcp.constants.ts:10 |
| `cnr package install finished` | Community node installed | Unauthorized user installed arbitrary code | telemetry.event-relay.ts:449 |
| `cnr package updated` | Community node updated | Unauthorized user updated community node | telemetry.event-relay.ts:471 |
| `cnr package deleted` | Community node deleted | Unauthorized user removed community node | telemetry.event-relay.ts:490 |
| `user started cnr package install` | Community node install started | Unauthorized user initiated package install | useInstallNode.ts:66 |
| `user started cnr package deletion` | Community node deletion started | Unauthorized user initiated package removal | CommunityPackageManageConfirmModal.vue:122 |
| `user started cnr package update` | Community node update started | Unauthorized user initiated package update | CommunityPackageManageConfirmModal.vue:149 |
| `User submitted builder message` | Message sent to AI builder | Unauthorized user used AI builder (may expose instance context) | builder.store.ts:652 |
| `User sent message in Assistant` | Assistant message sent | Unauthorized user interacted with AI assistant | assistant.store.ts:660 |
| `User ran test` | Evaluation test run | Unauthorized user executed evaluation test | test-runner.service.ee.ts:548 |
| `User deleted a run` | Test run deleted | Unauthorized user deleted test data | test-runs.controller.ee.ts:83 |
| `User set workflow description` | Workflow description updated | Unauthorized user modified workflow metadata | WorkflowDescriptionModal.vue:64 |
| `User edited workflow tags` | Workflow tags changed | Unauthorized user modified workflow metadata | WorkflowDetails.vue:177 |
| `User extracted nodes to sub-workflow` | Nodes extracted to sub-workflow | Unauthorized user restructured workflows | useWorkflowExtraction.ts:534 |
| `User created data table` | Data table created | Unauthorized user created data storage | AddDataTableModal.vue:276 |
| `User deleted data table` | Data table deleted | Unauthorized user destroyed data | DataTableActions.vue:137 |
| `User downloaded data table CSV` | Data table exported as CSV | Unauthorized user exfiltrated tabular data | DataTableActions.vue:117 |
| `User edited data table content` | Data table cell edited | Unauthorized user modified stored data | useDataTableOperations.ts:301 |
| `User deleted rows in data table` | Data table rows deleted | Unauthorized user destroyed data | useDataTableOperations.ts:371 |
| `Workflow execution with dynamic credentials` | Execution using dynamic credentials | Unauthorized user ran workflow with dynamic creds | telemetry/index.ts:185 |
| `User created agent` | Chat agent created | Unauthorized user created AI agent | chat.store.ts:835 |
| `User claimed OpenAI credits` | Free AI credits claimed | Unauthorized user claimed resources | useFreeAiCredits.ts:59 |
| `User registered for license community plus` | Community plus registration | Unauthorized user registered for license | telemetry.event-relay.ts:299 |
| `User successfully created new role` | Custom role created | Unauthorized user created new permission role | ProjectRoleView.vue:183 |
| `User updated role` | Custom role updated | Unauthorized user modified permissions | ProjectRoleView.vue:239 |
| `User successfully deleted role` | Custom role deleted | Unauthorized user deleted permission role | ProjectRoleView.vue:319 |
| `User moved content` | Workflow/credential moved between projects | Unauthorized user reorganized resources | WorkflowsView.vue:1685 |
| `User created folder` | Folder created | Unauthorized user modified file structure | WorkflowsView.vue:1301 |
| `User renamed folder` | Folder renamed | Unauthorized user modified organization | WorkflowsView.vue:1365,1754 |
| `User deleted folder` | Folder deleted | Unauthorized user destroyed organization | WorkflowsView.vue:533 |

### LOW

| Event Name | Trigger | What It Implies | Source |
|---|---|---|---|
| `User opened Credential modal` | Credential create/edit modal opened | User viewed credential configuration UI — secrets are redacted server-side, not visible | CredentialsSelectModal.vue:56, NodeCredentials.vue:351,495, etc. |
| `User viewed credential tab` | Tab switched in credential editor | User navigated credential details — secrets remain redacted | CredentialEdit.vue:602 |
| `User opened node modal` | NDV opened | User browsed node configuration | NodeDetailsView.vue:619 |
| `User closed node modal` | NDV closed | User closed node view | NodeDetailsView.vue:499 |
| `User opened workflow settings` | Settings dialog opened | User viewed workflow settings | WorkflowSettings.vue:733 |
| `User opened sharing modal` | Share dialog opened | User viewed sharing options | WorkflowCard.vue:357 |
| `User opened Expression Editor` | Expression editor opened | User opened expression editing | ParameterInput.vue:811 |
| `User closed Expression Editor` | Expression editor closed | User closed expression editing | ExpressionParameterInput.vue:116 |
| `User opened nodes panel` | Node creator panel opened | User browsed available nodes | nodeCreator.store.ts:337 |
| `User entered nodes panel search term` | Search in node panel | User searched for nodes | nodeCreator.store.ts:389 |
| `User viewed node category` | Category opened in panel | User browsed node categories | nodeCreator.store.ts:402,419 |
| `User viewed node actions` | Node actions viewed | User viewed node actions | nodeCreator.store.ts:411 |
| `User clicked custom API from node actions` | Custom API clicked | User clicked custom API option | nodeCreator.store.ts:415 |
| `User opened focus panel` | Focus panel opened | User opened focus panel | NodeView.vue:927 |
| `User closed focus panel` | Focus panel closed | User closed focus panel | FocusPanel.vue:333 |
| `User focused focus panel` | Focus sidebar activated | User interacted with focus | FocusSidebar.vue:110 |
| `User moved parameters pane` | NDV pane resized | User adjusted UI layout | NodeDetailsView.vue:395 |
| `User changed ndv run linking` | Run linking toggled | User changed run display | NodeDetailsView.vue:441 |
| `User changed ndv run dropdown` | Run dropdown changed | User browsed execution runs | NodeDetailsView.vue:510 |
| `User changed ndv input dropdown` | Input source changed | User changed input display | NodeDetailsView.vue:540 |
| `User clicked ndv link` | NDV link clicked | User navigated within NDV | Various NDV files |
| `User clicked ndv button` | NDV button clicked | User interacted with NDV | RunData.vue:1098 |
| `User changed ndv branch` | Output branch changed | User browsed output branches | RunData.vue:1086 |
| `User changed ndv page` | Pagination changed | User browsed data pages | RunData.vue:1125 |
| `User changed ndv page size` | Page size changed | User adjusted display | RunData.vue:1148 |
| `User changed ndv item view` | View mode changed (table/json/schema) | User changed data view | RunData.vue:1179 |
| `User opened ndv edit state` | Edit state entered | User entered edit mode | RunData.vue:955 |
| `User closed ndv edit state` | Edit state exited | User exited edit mode | RunData.vue:1020 |
| `User clicked pin data icon` | Pin data icon clicked | User interacted with pinning | RunData.vue:1045 |
| `Ndv data pinning success` | Data pinned successfully | User pinned test data | usePinnedData.ts:239 |
| `Ndv data pinning failure` | Pin data failed | Pin attempt failed | usePinnedData.ts:253 |
| `User unpinned ndv data` | Data unpinned | User removed pinned data | usePinnedData.ts:303 |
| `User viewed data mapping tooltip` | Mapping tooltip shown | User hovered over mapping help | InputPanel.vue:282 |
| `User dragged data for mapping` | Data dragged for mapping | User performed data mapping | RunDataJson.vue:118 |
| `User switched parameter mode` | Fixed/expression mode switched | User changed parameter input mode | ParameterInput.vue:1203 |
| `User chose input data mode` | Input mode selected | User selected input mode | ParameterInput.vue:947 |
| `User set node operation or mode` | Node operation changed | User configured node operation | ParameterInput.vue:1021 |
| `User turned on fromAI override` | fromAI enabled | User enabled AI override | ResourceLocator.vue:958 |
| `User turned off fromAI override` | fromAI disabled | User disabled AI override | ResourceLocator.vue:973 |
| `User added focused param` | Focus param added | User focused on parameter | ParameterInput.vue:1179 |
| `User removed focused param` | Focus param removed | User unfocused parameter | FocusPanel.vue:325 |
| `User viewed node settings` | Settings tab viewed | User viewed node settings | NodeSettingsTabs.vue:148 |
| `User autocompleted code` | Code autocomplete used | User used autocomplete | useAutocompleteTelemetry.ts:91 |
| `User imported curl command` | cURL import executed | User imported cURL command | ImportCurlModal.vue:75 |
| `User selected credential from node modal` | Credential selected in dropdown | User browsed credential list | NodeCredentials.vue:380 |
| `User clicked credential modal docs link` | Docs link clicked | User opened documentation | CredentialConfig.vue:240 |
| `User clicked add cred button` | Add credential button clicked | User initiated credential creation | CredentialsView.vue:135 |
| `User clicked quick connect button` | Quick connect clicked | User initiated OAuth quick connect | useQuickConnect.ts:139 |
| `User opened cred setup` | Credential setup modal opened | User opened template credential setup | SetupWorkflowCredentialsModal.vue:49 |
| `User closed cred setup` | Credential setup closed | User closed credential setup | SetupWorkflowCredentialsModal.vue:53 |
| `User clicked review credentials` | Review credentials link clicked | User clicked review after pull | sourceControl.utils.ts:84 |
| `User clicked review variables` | Review variables link clicked | User clicked review after pull | sourceControl.utils.ts:61 |
| `User toggled sidebar` | Sidebar toggled | User toggled navigation sidebar | ui.store.ts:594 |
| `User clicked help resource` | Help link clicked | User accessed help | MainSidebar.vue:253 |
| `User clicked insights link from side menu` | Insights link clicked | User navigated to insights | MainSidebar.vue:297 |
| `User clicked on what's new section` | What's new clicked | User viewed updates | MainSidebar.vue:303 |
| `User clicked on what's new notification` | Notification clicked | User clicked version notification | versions.store.ts:213 |
| `User clicked on update button` | Update button clicked | User clicked update CTA | VersionUpdateCTA.vue:25 |
| `User clicked upgrade CTA` | Upgrade CTA clicked | User clicked upgrade | usePageRedirectionHelper.ts:64 |
| `User clicked button on usage page` | Usage page button clicked | User interacted with usage page | SettingsUsageAndPlan.vue:191 |
| `User skipped community plus` | Community plus skipped | User dismissed enrollment | CommunityPlusEnrollmentModal.vue:55 |
| `User toggled log view` | Log view toggled | User interacted with log panel | useLogsPanelLayout.ts:72 |
| `User toggled log view sub pane` | Log sub pane toggled | User interacted with log panel | logs.store.ts:92 |
| `User selected node in log view` | Node selected in log | User browsed log entries | useLogsSelection.ts:53 |
| `User toggled n8n reference option` | Attribution toggled | User toggled n8n reference | telemetry/index.ts:196 |
| `User tidied up canvas` | Auto-arrange canvas | User organized canvas | useCanvasOperations.ts:243 |
| `User duplicated nodes` | Nodes duplicated | User copied nodes | useCanvasOperations.ts:2655 |
| `User copied nodes` | Nodes copied | User copied nodes to clipboard | useCanvasOperations.ts:2881 |
| `User hit undo` | Undo pressed | User undid action | useHistoryHelper.ts:84 |
| `User hit redo` | Redo pressed | User redid action | useHistoryHelper.ts:86 |
| `User hit undo in NDV` | Undo in NDV | User undid NDV action | useHistoryHelper.ts:96 |
| `User inserted workflow note` | Sticky note added | User added note | useCanvasOperations.ts:1059 |
| `User deleted workflow note` | Sticky note deleted | User removed note | useCanvasOperations.ts:514 |
| `User clicked chat open button` | Chat pane opened | User opened chat | useCanvasOperations.ts:2958 |
| `User started nodes to sub-workflow extraction` | Extraction started | User began extraction | useWorkflowExtraction.ts:527 |
| `User chose sub-workflow` | Sub-workflow selected | User selected sub-workflow | WorkflowSelectorParameterInput.vue:194 |
| `User clicked create new sub-workflow button` | Create sub-workflow clicked | User initiated sub-workflow | WorkflowSelectorParameterInput.vue:299 |
| `User added workflow input field` | Input field added | User added workflow input | FixedCollectionParameterNew.vue:416 |
| `User changed workflow input field type` | Input field type changed | User changed input type | FixedCollectionParameterNew.vue:423 |
| `user opened suggested actions checklist` | Checklist opened | User viewed production checklist | WorkflowProductionChecklist.vue:291 |
| `user clicked ignore suggested action` | Suggestion ignored | User dismissed suggestion | WorkflowProductionChecklist.vue:262 |
| `user clicked ignore suggested actions for all workflows` | All suggestions ignored | User dismissed all suggestions | WorkflowProductionChecklist.vue:282 |
| `User named version from history` | Version named | User labeled a version | WorkflowHistory.vue:408 |
| `User opened version in new tab` | Version opened in new tab | User viewed version | WorkflowHistory.vue:430,469 |
| `User selected version` | Version selected | User browsed versions | WorkflowHistory.vue:545 |
| `User attempted to save locked workflow` | Save attempted on locked workflow | User tried to save locked workflow | useWorkflowSaving.ts:256 |
| `User saved new workflow from template` | Workflow saved from template | User created from template | useWorkflowSaving.ts:503 |
| `User clicked add workflow button` | Add workflow button clicked | User initiated workflow creation | WorkflowsView.vue:891 |
| `User clicked empty page option` | Empty page option clicked | User selected option | WorkflowsView.vue:898 |
| `User clicked on workflow from insights table` | Workflow clicked in insights | User navigated to workflow | InsightsTableWorkflows.vue:141 |
| `User updated insights time range` | Insights time range changed | User adjusted analytics view | InsightsDataRangePicker.vue:128 |
| `User sorted insights table` | Insights table sorted | User sorted analytics data | InsightsTableWorkflows.vue:147 |
| `User changed template filters` | Template filters changed | User browsed templates | TemplatesSearchView.vue:158 |
| `User viewed template detail` | Template detail viewed | User viewed template | recommendedTemplates.store.ts:49 |
| `User viewed template cell` | Template card viewed | User browsed templates | recommendedTemplates.store.ts:55 |
| `User clicked on templates` | Templates link clicked | User navigated to templates | experiments/utils.ts:57 |
| `user clicked cnr install button` | CNR install button clicked | User initiated community node install | SettingsCommunityNodesView.vue:90 |
| `user viewed cnr settings page` | CNR settings viewed | User viewed community nodes | SettingsCommunityNodesView.vue:112 |
| `user clicked cnr browse button` | CNR browse button clicked | User browsed community nodes | CommunityPackageInstallModal.vue:50 |
| `user clicked cnr docs link` | CNR docs link clicked | User viewed docs | CommunityPackageInstallModal.vue:91 |
| `user clicked to browse the cnr package documentation` | CNR package docs browsed | User viewed package docs | CommunityPackageCard.vue:70 |
| `User clicked to open community node update modal` | Update modal opened | User viewed update options | ui.store.ts:544 |
| `User filtered executions with custom data` | Execution filter used | User filtered executions | ExecutionsFilter.vue:138 |
| `User initiated stop many executions` | Bulk stop initiated | User started bulk stop | ExecutionStopAllText.vue:24 |
| `User searched workflows in commit modal` | Commit modal search | User searched in commit | SourceControlPushModal.vue:588 |
| `User filtered by status in commit modal` | Status filter in commit modal | User filtered by status | SourceControlPushModal.vue:584 |
| `User clicks compare workflows` | Compare workflows clicked | User viewed diff | SourceControlPushModal.vue:756 |
| `User clicked connected clients tab` | MCP clients tab clicked | User viewed MCP clients | SettingsMCPView.vue:73 |
| `User clicked connect workflows from mcp settings` | MCP connect workflows clicked | User browsed MCP settings | SettingsMCPView.vue:194 |
| `User copied MCP connection parameter` | MCP param copied | User copied MCP config | McpConnectPopover.vue:61 |
| `User dismissed mcp workflows dialog` | MCP dialog dismissed | User closed MCP dialog | MCPConnectWorkflowsModal.vue:37,59 |
| `User selected workflow from list` | Workflow selected in MCP | User selected workflow for MCP | MCPConnectWorkflowsModal.vue:48 |
| `Assistant session started` | AI assistant session began | User opened assistant | assistant.store.ts:265 |
| `User opened assistant` | Assistant panel opened | User opened assistant | assistant.store.ts:693 |
| `User gave feedback` | Thumbs up/down on assistant | User rated assistant | AskAssistantChat.vue:50 |
| `User clicked solution card action` | Undo/redo on solution | User interacted with solution | AskAssistantChat.vue:63 |
| `User executed node after assistant suggestion` | Node executed after suggestion | User followed suggestion | assistant.store.ts:575 |
| `User rated workflow generation` | AI generation rated | User rated AI output | AskAssistantBuild.vue:289 |
| `User submitted workflow generation feedback` | AI feedback submitted | User provided AI feedback | AskAssistantBuild.vue:296 |
| `Workflow diff view opened` | Diff modal opened | User viewed AI changes | AIBuilderDiffModal.vue:28 |
| `Workflow diff view closed` | Diff modal closed | User closed diff view | AIBuilderDiffModal.vue:33 |
| `End of response from builder` | Builder response finished | AI builder completed response | builder.store.ts:392 |
| `Workflow builder journey` | Builder journey tracking (various sub-events) | User progressed through builder | builder.store.ts:1141 |
| `User sent chat hub message` | Chat hub message sent | User interacted with chat | chat.store.ts:530 |
| `User edited chat hub message` | Chat hub message edited | User edited chat message | chat.store.ts:627 |
| `User regenerated chat hub message` | Chat hub message regenerated | User regenerated response | chat.store.ts:677 |
| `User clicked chat hub suggested prompt` | Suggested prompt clicked | User used suggested prompt | ChatView.vue:713 |
| `User clicked new chat button` | New chat started | User started new chat | ChatSidebarContent.vue:135 |
| `User selected model or agent` | Model/agent selected | User chose model | ModelSelector.vue:167 |
| `User encountered an error` | Error shown in NDV | User saw error message | RunData.vue:776 |
| `User clicked on RAG callout` | RAG callout clicked | User interacted with RAG | useCalloutHelpers.ts:82 |
| `User inserted tutorial template` | Tutorial template inserted | User added tutorial template | useCalloutHelpers.ts:86 |
| `User responded to personalization questions` | Personalization survey completed | User completed survey | telemetry.event-relay.ts:1263 |
| `User responded value survey score` | NPS score submitted | User rated experience | NpsSurvey.vue:56 |
| `User responded value survey feedback` | NPS feedback submitted | User provided feedback | NpsSurvey.vue:67 |
| `User executed command bar command` | Command bar used | User ran command | useCommandBar.ts:231 |
| `User renamed data table` | Data table renamed | User modified table name | DataTableBreadcrumbs.vue:88 |
| `User deleted data table column` | Data table column deleted | User modified table | useDataTableOperations.ts:141 |
| `User renamed data table column` | Data table column renamed | User modified table | useDataTableOperations.ts:180 |
| `User added data table column` | Data table column added | User modified table | useDataTableOperations.ts:211 |
| `User added row to data table` | Data table row added | User added data | useDataTableOperations.ts:265 |
| `User viewed tests tab` | Tests tab viewed | User browsed tests | EvaluationsRootView.vue:84 |
| `User added execution annotation tag` | Annotation tag added | User annotated execution | WorkflowExecutionAnnotationTags.ee.vue:67 |
| `User clicked add role` | Add role clicked | User browsed roles | ProjectRolesView.vue:240 |
| `User duplicated role` | Role duplicated | User browsed roles | ProjectRolesView.vue:178 |
| `User clicked delete role` | Delete role clicked | User initiated role deletion | ProjectRoleView.vue:302 |
| `User clicked add custom role from role selector` | Custom role option clicked | User browsed role options | ProjectMembersRoleCell.vue:150 |
| `User clicked project variables tab` | Variables tab clicked | User browsed project vars | ProjectTabs.vue:145 |
| `User clicked add variable button` | Add variable clicked | User initiated variable creation | ProjectVariables.vue:84 |
| `User clicked header add variable button` | Header add variable clicked | User initiated variable creation | ProjectHeader.vue:327 |
| `ai.focusedNodes.added` | Focused node added to AI | User added node focus | focusedNodes.store.ts:114 |
| `ai.focusedNodes.removed` | Focused node removed from AI | User removed node focus | focusedNodes.store.ts:184 |
| `ai.focusedNodes.chatSent` | Chat sent with focused nodes | User sent focused chat | builder.store.ts:659 |
| `Ai code generation finished` | AI code gen completed | AI generated code | telemetry/index.ts:164 |
| `Ai Transform code generation finished` | AI transform code gen completed | AI generated transform | telemetry/index.ts:178 |
| `User opened ready to run workflow` | Ready-to-run workflow opened | User opened starter | readyToRunWorkflows.store.ts:59 |
| `User executed ready to run workflow` | Ready-to-run workflow executed | User ran starter | readyToRunWorkflows.store.ts:65 |
| `User opened ready to run AI workflow` | Ready-to-run AI workflow opened | User opened AI starter | readyToRun.store.ts:99 |
| `User executed ready to run AI workflow` | Ready-to-run AI workflow executed | User ran AI starter | readyToRun.store.ts:63 |
| `User executed ready to run AI workflow successfully` | AI starter succeeded | Execution completed | readyToRun.store.ts:69 |
| `User created ready to run workflows` | Starter workflows created | System created starters | readyToRunWorkflows.store.ts:49 |
| `User dismissed ready to run workflows callout` | Starter callout dismissed | User dismissed UI | readyToRunWorkflows.store.ts:55 |
| `User opened AI template workflow` | AI template opened | User viewed AI template | aiTemplatesStarterCollection.store.ts:81 |
| `User executed AI template` | AI template executed | User ran AI template | aiTemplatesStarterCollection.store.ts:87 |
| `User created AI templates starter collection` | AI starter collection created | System created collection | aiTemplatesStarterCollection.store.ts:71 |
| `User dismissed AI templates starter collection callout` | AI starter callout dismissed | User dismissed UI | aiTemplatesStarterCollection.store.ts:77 |
| `User clicked from scratch in empty state` | From scratch clicked | User chose blank canvas | EmptyStateBuilderPrompt.vue:49 |
| `User submitted empty state builder prompt` | Empty state prompt submitted | User used builder prompt | emptyStateBuilderPrompt.store.ts:66 |
| `User imported workflow from empty state` | Workflow imported from empty state | User imported template | emptyStateBuilderPrompt.store.ts:103 |
| `User completed all setup steps` | All setup steps done | User finished setup guide | SetupPanelCards.vue:31 |
| `User completed setup step` | Single setup step done | User completed step | SetupCard.vue:49 |
| `User executed test AI workflow` | Test AI workflow run | User tested AI workflow | executionFinished.ts:92 |
| `User executed tutorial template` | Tutorial template run | User ran tutorial | executionFinished.ts:111 |
| `App selection page viewed` | App selection shown | User viewed app selection | credentialsAppSelection.store.ts:52 |
| `App selection search performed` | App selection searched | User searched apps | credentialsAppSelection.store.ts:56 |
| `App selection completed` | App selection done | User completed selection | credentialsAppSelection.store.ts:67 |
| `User was recommended personalized templates` | Templates recommended | System recommended templates | personalizedTemplates.store.ts:99 |
| `User clicked on personalized template callout` | Template callout clicked | User clicked callout | personalizedTemplates.store.ts:105 |
| `User dismissed personalized template callout` | Template callout dismissed | User dismissed callout | personalizedTemplates.store.ts:111 |
| `User clicked on personalization card` | Personalization card clicked | User interacted | personalizedTemplatesV3.store.ts:74 |
| `User viewed personalization modal` | Personalization modal shown | User viewed modal | personalizedTemplatesV3.store.ts:78 |
| `User viewed personalization tooltip` | Personalization tooltip shown | Tooltip displayed | personalizedTemplatesV3.store.ts:82 |
| `User dismissed personalization tooltip` | Tooltip dismissed | User dismissed | personalizedTemplatesV3.store.ts:86 |
| `User clicked on template from modal` | Template from modal clicked | User selected template | personalizedTemplatesV3.store.ts:90 |
| `User clicked on templates repo from modal` | Templates repo clicked | User navigated to repo | personalizedTemplatesV3.store.ts:96 |
| `User clicked on node minicard` | Node minicard clicked | User clicked minicard | templateRecoV2.store.ts:59 |
| `User visited template recommendation modal tab` | Template reco tab visited | User browsed tab | templateRecoV2.store.ts:65 |
| `User clicked on template recommendation tile` | Template reco tile clicked | User clicked template | templateRecoV2.store.ts:71 |
| `User clicked on template recommendation video` | Template reco video clicked | User clicked video | templateRecoV2.store.ts:77 |
| `User clicked on template recommendation see more` | Template reco see more clicked | User expanded | templateRecoV2.store.ts:83 |
| `User viewed resource center tooltip` | Resource center tooltip shown | Tooltip displayed | resourceCenter.store.ts:51 |
| `User dismissed resource center tooltip` | Resource center tooltip dismissed | User dismissed | resourceCenter.store.ts:55 |
| `User visited resource center` | Resource center opened | User opened resource center | resourceCenter.store.ts:89 |
| `User visited resource center section` | Resource center section visited | User browsed section | resourceCenter.store.ts:93 |
| `User clicked on resource center tile` | Resource center tile clicked | User clicked tile | resourceCenter.store.ts:97 |
| `User visited template repo` | Template repo visited | User visited templates | resourceCenter.store.ts:101 |

### NONE

| Event Name | Trigger | What It Implies | Source |
|---|---|---|---|
| `Instance started` | Server startup | System event — instance booted | telemetry.event-relay.ts:1060 |
| `User instance stopped` | Instance shutdown | System event — instance stopped | telemetry.event-relay.ts:1071 |
| `Session started` | Session/telemetry init | System event — session began | telemetry/index.ts:65, telemetry.event-relay.ts:1067 |
| `pulse` | 6-hour heartbeat | System pulse — no user action | telemetry/index.ts:152 |
| `Workflow execution count` | Periodic aggregate | System batch of execution counts | telemetry/index.ts:126 |
| `Instance attempted to refresh license` | License refresh | System license check | telemetry.event-relay.ts:289 |
| `Owner finished instance setup` | Initial setup completed | System setup event | telemetry.event-relay.ts:1075 |
| `User signed up` | User completed signup | Account creation event | telemetry.event-relay.ts:1248 |
| `User attempted to login` | Login form submitted | Auth event | SigninView.vue:146 |
| `User is part of experiment` | Experiment assignment | System A/B test assignment | posthog.store.ts:105 |
| `Workflow first prod success` | First production success | Milestone event | telemetry.event-relay.ts:1083 |
| `Instance first prod failure` | First production failure | Milestone event | telemetry.event-relay.ts:1095 |
| `Workflow first data fetched` | First data fetch | Milestone event | telemetry.event-relay.ts:1110 |
| `Workflow execution errored` | Execution error (auto-tracked) | System error tracking | telemetry/index.ts:193 |
| `Manual exec errored` | Frontend execution error | System error tracking | workflows.store.ts:1137 |
| `Instance FE emitted error` | Frontend error | System error | useToast.ts:84 |
| `Instance FE emitted paired item error` | Paired item error | System error | executionFinished.ts:360 |
| `Instance compacted workflow history` | History compaction | System maintenance | telemetry.event-relay.ts:1413 |
| `User hit concurrency limit` | Concurrency limit reached | System limit event | concurrency-control.service.ts:85 |
| `User hit data table storage limit` | Storage limit reached | System limit event | data-table-size-validator.service.ts:62 |
| `User ran out of free AI credits` | AI credits exhausted | System limit event | telemetry.event-relay.ts:841 |
| `User sent transactional email` | Email sent successfully | System email event | telemetry.event-relay.ts:1283 |
| `Instance failed to send transactional email to user` | Email send failed | System email failure | telemetry.event-relay.ts:1271 |
| `User clicked invite link from email` | Invite link clicked | System onboarding event | telemetry.event-relay.ts:1382 |
| `User clicked password reset link from email` | Password reset link clicked | System auth event | telemetry.event-relay.ts:1388 |
| `User requested password reset while logged out` | Password reset requested | System auth event | telemetry.event-relay.ts:1396 |
| `Ldap general sync finished` | LDAP sync completed | System sync event | telemetry.event-relay.ts:578 |
| `Ldap login sync failed` | LDAP sync failed | System error | telemetry.event-relay.ts:617 |
| `User login failed since ldap disabled` | Login failed, LDAP disabled | System auth event | telemetry.event-relay.ts:623 |
| `Sso user project access update` | SSO provisioning update | System provisioning | telemetry.event-relay.ts:635 |
| `Sso user instance role update` | SSO role update | System provisioning | telemetry.event-relay.ts:646 |
| `Test run finished` | Test run completed | System event | test-runner.service.ee.ts:807 |
| `User shown value survey` | NPS survey shown | System event | NpsSurvey.vue:121 |
| `User set filters in ${resourceKey} list` | List filter changed | UI state | ResourcesListLayout.vue:409 |
| `User changed sorting in ${resourceKey} list` | List sort changed | UI state | ResourcesListLayout.vue:454 |
| `User opened workflow from onboarding template with ID ${id}` | Onboarding workflow opened | System onboarding | useWorkflowInitialization.ts:303 |

---

## Notes

### Events NOT in RudderStack (audit/log-streaming only)
These internal events do NOT appear in `rudder_schema.tracks`:
- `user-logged-in`
- `user-login-failed`
- `user-mfa-enabled`
- `user-mfa-disabled`
- `user-reinvited`

### Dynamic event names
Some events use template strings and will appear with variable content:
- `User opened workflow from onboarding template with ID ${id}` → `User opened workflow from onboarding template with ID 123`
- `User hit ${type}` → `User hit undo` or `User hit redo`
- `User set filters in ${resourceKey} list` → `User set filters in workflows list`
- `User changed sorting in ${resourceKey} list` → `User changed sorting in workflows list`
- `User clicked ${summaryTitles[type]}` → depends on insight type

### Credential secrets are NOT exposed via UI
The backend **redacts** all credential fields marked with `typeOptions: { password: true }` before sending them in API responses. The actual values are replaced with a sentinel string (`CREDENTIAL_BLANKING_VALUE`). There is no "show password" toggle in the UI. Sharees receive no credential data at all. The real secrets never leave the server in API responses — they are only used server-side during workflow execution and restored from the DB via `unredact()` on save.

**Caveat:** Fields not marked `typeOptions: { password: true }` in their credential type definition would be returned as plaintext. An ESLint rule catches common sensitive field names missing this annotation, but coverage depends on credential type authors.

### Workflow creation/execution is a credential exfiltration vector
A user with workflow edit + execute permissions can create or modify a workflow to use existing credentials (e.g., HTTP Request node with a bound API credential sending data to an attacker endpoint). The credential secret itself is never visible in the UI, but the workflow execution uses the real decrypted credential, making the data accessible to the external endpoint. This is why `User created workflow`, `User saved workflow`, and execution events are rated HIGH.

### What's NOT tracked
- **Workflow JSON viewing** — reading a workflow via the editor doesn't fire a telemetry event beyond the route-level page view. The `User retrieved workflow` event only fires for Public API calls.
- **Execution data viewing** — viewing execution input/output data in the NDV doesn't have a dedicated event beyond opening the NDV itself.
