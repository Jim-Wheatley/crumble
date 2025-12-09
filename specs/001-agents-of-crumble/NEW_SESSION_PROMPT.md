# New Copilot Session Prompt Template

Use this prompt to start a fresh Copilot chat session for any task in the Agents of the Crumbleverse project:

---

## Prompt for New Session:

```
You are an expert on Power Apps and Power Automate. I am a novice with these tools and need beginner-level, step-by-step instructions to complete tasks. With code snippets and clear direction of where to place the snippets. 

I'm working on "Agents of the Crumbleverse" - a workshop game built with Power Apps + Power Automate + SharePoint Lists + Teams.

**Project Context:**
- Location: `/Users/jim.wheatley/Library/CloudStorage/GoogleDrive-wheatleyux@gmail.com/My Drive/jimdev/agents of the crumbleverse`
- Specs are in: `specs/001-agents-of-crumble/` (spec.md, plan.md, tasks.md)
- Stack: Power Apps (primary UI), Power Automate (transactional flows), SharePoint Lists (persistence), Teams (adaptive cards)
- No Copilot Studio or Dataverse access - using SharePoint Lists instead
- Players create unique IDs using pattern: `{region}-{biscuitname}` (e.g., `london-digestive`)

**Key Architecture Decisions:**
1. Power Apps is the primary player UI (reads `agentId` query param from deep links; always loads the single active game)
   - **Using Modern (Fluent) controls** - different properties than Classic controls (e.g., `Value` vs `Default`, `SelectedItems` vs `Selected`)
   - App starts directly on `Screen_AgentDetail` (no home screen needed)
   - Deep link params read in App OnStart using `Param("agentId")`, `Param("displayName")`, `Param("className")`
2. Power Apps writes directly to SharePoint for safe single-row operations (registration, profile edits)
3. Power Automate flows handle transactional operations (locks, turn-order, multi-list updates, Teams cards)
   - Use Power Apps (V2) trigger for better input handling
   - OData filters require concat for dynamic values: `concat('ColumnName eq ''', variable, '''')`
   - All 200 responses must have identical schema (return same fields in success/error paths)
4. SharePoint Lists: `Players`, `Games` (single row: "current-game", includes MindCrumble fields), `ActionLog`, `ClassTemplates` (includes base stats + MindCrumble row)
   - Column names are case-sensitive (use internal names like `ClassName`, `AgentID`)
   - Games list has exactly one record with `Title = "current-game"`
5. Concurrency: Use lock fields (`LockedBy`, `LockExpiresAt`) in Power Automate flows (30-second timeout)

**Task I'm working on:**
[PASTE TASK FROM tasks.md HERE - e.g., "Task 1.2: Implement AttackFlow logic"]

**What I need:**
[DESCRIBE WHAT YOU NEED - e.g., "Step-by-step Power Automate flow design with HTTP actions for SharePoint", "Power Apps formulas for the Attack button", "PnP script to create SharePoint lists", etc.]

Please read the spec files first to understand the full context, then help me implement this task.
```

---

## Quick Task Reference (copy the relevant task):

### Sprint 0
- **Task 0.2:** Provision SharePoint lists (Players, Games, ActionLog) ✅
- **Task 0.2b:** Create ClassTemplates list and seed base stats for all classes and MindCrumble ✅
- **Task 0.3:** Create Power Apps canvas app skeleton and verify deep link parsing ✅
- **Task 0.4:** Create sample Power Automate flow that updates a record and posts to Teams ✅

### Sprint 1
- **Task 1.1:** Implement JoinGameFlow and wire to Power Apps ✅ (see `/docs/task-1.1-join-game-flow.md`)

https://apps.powerapps.com/play/<YourAppId>?tenantId=<YourTenantId>&agentId=london-digestive&displayName=Test%20Player&className=Bourbon%20Rogue
- **Task 1.2:** Implement AttackFlow logic (damage calculations and ability effects)
- **Task 1.3:** Implement Mind Crumble turn logic
- **Task 1.4:** Wire Power Apps Attack button to AttackFlow
- **Task 1.5:** Update Copilot agent instructions to include deep link template and ID creation guidance
- **Task 1.6:** Update player creation flows/apps to read ClassTemplates and apply base stats

### Sprint 2
- **Task 2.1:** Implement locking / concurrency handling (prefer Power Automate for transactional paths)
- **Task 2.2:** Add Teams adaptive card formatting and logging
- **Task 2.3:** Create facilitator admin flows (reset game) and UI

### Sprint 3
- **Task 3.1:** Prepare facilitator guide and slide deck
- **Task 3.2:** Run pilot workshop and gather feedback
- **Task 3.3:** Finalize spec and repo

---

## Example Filled Prompt:

```
I'm working on "Agents of the Crumbleverse" - a workshop game built with Power Apps + Power Automate + SharePoint Lists + Teams.

**Project Context:**
- Location: `/Users/jim.wheatley/Library/CloudStorage/GoogleDrive-wheatleyux@gmail.com/My Drive/jimdev/agents of the crumbleverse`
- Specs are in: `specs/001-agents-of-crumble/` (spec.md, plan.md, tasks.md)
- Stack: Power Apps (primary UI), Power Automate (transactional flows), SharePoint Lists (persistence), Teams (adaptive cards)
- No Copilot Studio or Dataverse access - using SharePoint Lists instead
- Players create unique IDs using pattern: `{region}-{biscuitname}` (e.g., `london-digestive`)

**Key Architecture Decisions:**
1. Power Apps is the primary player UI (reads `agentId` query param from deep links; always loads the single active game)
   - **Using Modern (Fluent) controls** - different properties than Classic controls (e.g., `Value` vs `Default`, `SelectedItems` vs `Selected`)
   - App starts directly on `Screen_AgentDetail` (no home screen needed)
   - Deep link params read in App OnStart using `Param("agentId")`, `Param("displayName")`, `Param("className")`
2. Power Apps writes directly to SharePoint for safe single-row operations (registration, profile edits)
3. Power Automate flows handle transactional operations (locks, turn-order, multi-list updates, Teams cards)
   - Use Power Apps (V2) trigger for better input handling
   - OData filters require concat for dynamic values: `concat('ColumnName eq ''', variable, '''')`
   - All 200 responses must have identical schema (return same fields in success/error paths)
4. SharePoint Lists: `Players`, `Games` (single row: "current-game", includes MindCrumble fields), `ActionLog`, `ClassTemplates` (includes base stats + MindCrumble row)
   - Column names are case-sensitive (use internal names like `ClassName`, `AgentID`)
   - Games list has exactly one record with `Title = "current-game"`
5. Concurrency: Use lock fields (`LockedBy`, `LockExpiresAt`) in Power Automate flows (30-second timeout)

**Task I'm working on:**
Task 1.2: Implement AttackFlow logic (owner: flow-dev). AC: damage calculations and ability effects applied.

**What I need:**
Step-by-step Power Automate flow design for AttackFlow that:
1. Accepts agentId as input (game is always "current-game")
2. Acquires lock on the single Games record
3. Reads Player and Games records from SharePoint
4. Parses AbilityJson from ClassTemplates
5. Computes damage with buffs/debuffs
6. Updates Games (MindCrumble HP) and ActionLog
7. Posts adaptive card to Teams
8. Releases lock and returns result

Please read the spec files first to understand the full context, then help me implement this task.
```
