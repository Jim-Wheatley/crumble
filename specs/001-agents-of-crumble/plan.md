# Plan — Agents of the Crumbleverse

## Tech Choices
- UI: Power Apps canvas app (primary player-facing UI). Copilot agents provide deep links to the Power App but do not host the full interactive UI in this tenant.
- Bridge/UI: Power Apps canvas app (single app with parameterized character rendering)
- Orchestration: Power Automate flows (triggered by Power Apps)
- Persistence: SharePoint lists (primary)
- Real-time feedback: Microsoft Teams adaptive cards; optional Power BI streaming dataset
- Auth: Azure AD

## Architecture Summary
 - `ClassTemplates` holds canonical class stats and ability JSON; Power Apps should cache templates on `App.OnStart` (e.g., `ClearCollect(colClassTemplates, ClassTemplates)`) and reference them when pre-populating player fields.

## Implementation Plan (high level)
- Risk: Concurrency issues with many simultaneous button presses. Mitigation: prefer using Power Automate for transactional/multi-item updates (locks, turn-order, multi-row updates). When safe (single-row writes like player registration or profile updates) Power Apps can write directly to SharePoint using `Patch`.
- Risk: Dataverse not available. Mitigation: SharePoint lists are used; accept weaker transactionality and use Power Automate flows for critical sections.

1. Build Power Apps canvas app that reads `agentId` (e.g., `london-digestive`) query param and renders the character sheet. The app always loads the single active game (ID: `current-game`). Players create their own unique IDs using the pattern `{region}-{biscuitname}` for easy workshop setup.
2. Implement direct-write scenarios in Power Apps (registration, profile edits) using `Patch` to SharePoint lists.
3. Create Power Automate flows for transactional scenarios: `JoinGameFlow` (acquire lock, set TurnOrder), `AttackFlow` (acquire lock, compute results, write updates to multiple lists, release lock), `MindCrumbleFlow` (villain turn logic), `AdminResetFlow`.
4. Create Copilot agent templates (instructions + deep link). The agent includes a deep link with `agentId` to open Power Apps.
5. Implement concurrency/locking and validation inside flows (ETag/If-Match or lock rows via `LockedBy`/`LockExpiresAt`) and provide retries + failure handling back to the app.
6. Add Teams adaptive card formatting and optional Power BI integration.
7. QA, workshops, and facilitator materials.

## Risks & Mitigations
 - Risk: Players must manually create and embed their unique ID in the deep link. Mitigation: Provide clear workshop instructions with ID pattern examples (`{region}-{biscuitname}`) and a template deep link that players can copy and customize.
 - Risk: Concurrency issues with many simultaneous button presses. Mitigation: use Power Automate flows to serialize critical updates or an Azure Function if flows are insufficient.
 - Risk: Dataverse not available. Mitigation: Use SharePoint lists but accept weaker concurrency guarantees and adjust locking logic.
