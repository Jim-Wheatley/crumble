# Tasks — Agents of the Crumbleverse

## Sprint 0 — Setup & Proof-of-Concept
 - Task 0.2b: Create `ClassTemplates` list and seed base stats for all classes and `MindCrumble` (owner: infra/content). AC: `ClassTemplates` contains rows for all six biscuit classes and a `MindCrumble` row with AbilityJson.
 Task 0.2: Provision SharePoint lists (owner: infra). AC: `Players`, `Games` (contains MindCrumble fields), `ActionLog` exist.
 Task 0.2b: Create `ClassTemplates` list and seed base stats for all classes and `MindCrumble` (owner: infra/content). AC: `ClassTemplates` contains rows for all six biscuit classes and a `MindCrumble` row with AbilityJson.
 Task 0.3: Create Power Apps canvas app skeleton and verify deep link parsing (owner: front-end). AC: app reads `agentId` param and shows placeholder data.
 Task 0.4: Create sample Power Automate flow that updates a record and posts to Teams (owner: flow-dev). AC: test button triggers Teams message.

## Sprint 1 — Core Combat Mechanics
- Task 1.1: Implement Join Game flow and TurnOrder assignment (owner: flow-dev). AC: joining assigns `TurnOrder`.
- Task 1.2: Implement AttackFlow logic (owner: flow-dev). AC: damage calculations and ability effects applied.
- Task 1.3: Implement Mind Crumble turn logic (owner: flow-dev). AC: Mind Crumble fields on the `Games` record are updated after each player.
 - Task 1.4: Wire Power Apps Attack button to AttackFlow (owner: front-end). AC: pressing button triggers flow and returns result.
 - Task 1.5: Update Copilot agent instructions to include deep link template and ID creation guidance (owner: content). AC: agent instructions contain working template link and examples showing players how to create their unique ID using the `{region}-{biscuitname}` pattern.
 - Task 1.6: Update player creation flows/apps to read `ClassTemplates` and apply base stats (owner: front-end/flow-dev). AC: new players are created with stats populated from `ClassTemplates`.

## Sprint 2 — Resilience & UX
 - Task 2.1: Implement locking / concurrency handling (owner: infra). AC: race conditions prevented during concurrent presses. Prefer Power Automate flows for transactional paths.
 - Task 2.2: Add Teams adaptive card formatting and logging (owner: flow-dev). AC: action posts well-formatted card and ActionLog populated.
 - Task 2.3: Create facilitator admin flows (reset game) and UI (owner: ops). AC: facilitator can reset the game.

## Sprint 3 — Workshop Materials & QA
- Task 3.1: Prepare facilitator guide and slide deck (owner: content). AC: deliver deck and script.
- Task 3.2: Run pilot workshop and gather feedback (owner: PO). AC: successful run with ≤ 50 players and documented issues.
- Task 3.3: Finalize spec and repo (owner: dev). AC: `specs/`, `tasks.md`, `plan.md` committed.

## Details & Notes
- Provide `agentLinkTemplate` example to content team: `https://apps.powerapps.com/play/{AppId}?tenantId={TenantId}&agentId={region-biscuitname}`
- Example filled deep link: `https://apps.powerapps.com/play/e/abc123/a/def456?agentId=london-digestive`
- ID pattern guidance for participants: Create a unique player ID by combining your region/city and biscuit name with a hyphen (e.g., `paris-macaroon`, `tokyo-pocky`, `berlin-pretzel`). This ID will be your character identifier throughout the game.
- Single-game model: The game always uses one active game record (ID: `current-game`). No need to track or select different games.
- If distributing via Teams, include Teams deep links for an integrated experience.
- Note: Mind Crumble data is stored on the `Games` item (fields like `MindCrumble_HP`, `MindCrumble_Defence`); flows should update the `Games` item.
