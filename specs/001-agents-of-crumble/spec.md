# 001 - Agents of the Crumbleverse

## Overview
Forge your agentic-hero biscuit to take on the dreaded Mind Crumble. Teams create a Copilot-powered "biscuit agent" and use a Power Apps bridge to trigger game actions. The game uses Power Apps, Power Automate, SharePoint lists (as the canonical data store), and Teams for real-time feedback.

## Goals
- Make a playful, low-friction workshop where participants build agent UIs and trigger abilities that update shared game state.
- Provide reliable, auditable game logic and real-time feedback to Teams.

## Primary Users
- Player (workshop participant)
- Facilitator (organiser)
- Observer
Forge your agentic-hero biscuit to take on the dreaded Mind Crumble. Teams create a Copilot-powered "biscuit agent" and use a Power Apps bridge to trigger game actions. The game uses Power Apps, Power Automate, SharePoint lists (as the canonical data store), and Teams for real-time feedback.
- Copilot Studio cannot invoke Power Automate directly in this environment. A Power Apps canvas app will act as the bridge. Copilot agents must include deep links with `agentId` to open the Power App.
AC2: Pressing Attack/Ability in Power Apps invokes a Power Automate flow that updates SharePoint and posts a Teams adaptive card with the action result.
AC3: Mind Crumble acts after each player action and updates are consistent and logged in SharePoint.
- FR7: Admin Tools — facilitator can reset games and configure the Mind Crumble.

## Data Model (SharePoint lists)
- ClassTemplates: ClassName (Title), BaseHP, BaseAttack, BaseDefence, AbilityName, AbilityJson, Description, ImageUrl, TemplateVersion
- Players: PlayerID (Text, primary key), DisplayName, Class, HP, MaxHP, Attack, Defence, TurnOrder, AbilityState (JSON), ProfilePicUrl, LastActionAt
- Games: ID (Text, fixed value: "current-game"), Status, TurnPointer, CreatedBy, PlayerCount, MindCrumble_HP (Number), MindCrumble_Attack (Number), MindCrumble_Defence (Number), MindCrumble_Status (Choice), MindCrumble_Debuffs (JSON)
- ActionLog: Timestamp, ActorID, ActionType, Details(JSON), Result(JSON)

**ID Format Convention:**
Players create their own unique `PlayerID` using the pattern: `{region}-{biscuitname}` (e.g., `london-digestive`, `paris-macaroon`, `tokyo-pocky`). This human-readable ID is easier for workshop participants to create and embed in their Copilot agent deep links than numeric or GUID values.

**Single-Game Model:**
The game runs one instance at a time. The `Games` list contains a single row with ID `current-game` that is reset between workshop sessions. This eliminates the need to manage multiple game states or pass `gameId` in deep links.

Purpose: `ClassTemplates` stores canonical base stats and ability definitions for each biscuit class. Power Apps and Power Automate read `ClassTemplates` to populate `Players` on creation and to interpret abilities during combat.

Note: For a minimal workshop setup, the Mind Crumble villain will be seeded as a row in `ClassTemplates` (ClassName = "MindCrumble") containing base HP and AbilityJson. This avoids creating an extra `MonsterTemplates` list when only one villain exists.

Sample Mind Crumble AbilityJson (two-mode behavior):
```json
{
    "type":"conditional",
    "name":"crumbBehavior",
    "conditions":[
        {"when":"hp<=5","actions":[{"type":"heal","target":"self","amount":1,"repeatFor":"playersWaiting"}]},
        {"when":"hp>5","actions":[{"type":"damage","target":"all","amount":3}]} 
    ]
}
```

## Acceptance Criteria
- AC1: A participant creates a unique player ID (e.g., `london-digestive`), embeds it in their Copilot agent deep link, clicks the link, and sees the Power App pre-populated with their agent stats.
- AC2: Pressing Attack/Ability in Power Apps invokes a Power Automate flow that updates SharePoint and posts a Teams adaptive card with the action result.
- AC3: Mind Crumble acts after each player action and updates are consistent and logged in SharePoint.
- AC4: Facilitator can reset the game and restore initial stats.

---
