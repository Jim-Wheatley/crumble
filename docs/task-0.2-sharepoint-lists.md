# Task 0.2 — Provision SharePoint Lists

**Status:** ✓ Complete  
**Owner:** Infra  
**Date Completed:** November 2025 (estimated)

## Overview

Created SharePoint lists to serve as the canonical data store for the Agents of the Crumbleverse game. Lists store player stats, game state, class templates, and action logs.

## Acceptance Criteria

✓ `Players` list exists with required columns  
✓ `Games` list exists with MindCrumble fields  
✓ `ActionLog` list exists for event tracking  
✓ `ClassTemplates` list exists with base stats for all classes and MindCrumble

## SharePoint Lists Created

All lists prefixed with `Summit - Crumbleverse -` for clarity on shared SharePoint site.

### 1. Summit - Crumbleverse - Players

**Purpose:** Store individual player/agent stats and state.

**Columns:**
- `Title` (Single line of text, default) — Player display name
- `AgentID` (Single line of text, required) — Unique player identifier (e.g., `london-digestive`)
- `Class` (Choice) — Player class (Biscuit, Warrior, etc.)
- `HP` (Number) — Current hit points
- `MaxHP` (Number) — Maximum hit points
- `Attack` (Number) — Attack stat
- `Defence` (Number) — Defence stat
- `TurnOrder` (Number) — Turn sequence in game
- `AbilityState` (Multiple lines of text) — JSON for ability cooldowns/state
- `ProfilePicUrl` (Hyperlink) — Optional avatar image URL
- `LastActionAt` (Date/Time) — Timestamp of last action

**ID Format Convention:**  
Players create their own `AgentID` using pattern: `{region}-{biscuitname}` (e.g., `london-digestive`, `paris-macaroon`). This human-readable ID is easier for workshop participants than GUIDs.

### 2. Summit - Crumbleverse - Games

**Purpose:** Store game session state and Mind Crumble villain stats.

**Columns:**
- `Title` (Single line of text, default) — Game name/ID (e.g., `game-alpha`)
- `GameID` (Single line of text, required) — Unique game identifier
- `Status` (Choice) — Game state: NotStarted, InProgress, Completed
- `TurnPointer` (Number) — Current turn index
- `CreatedBy` (Person) — Facilitator who created the game
- `PlayerCount` (Number) — Number of players in game
- `MindCrumble_HP` (Number) — Mind Crumble current HP
- `MindCrumble_MaxHP` (Number) — Mind Crumble maximum HP
- `MindCrumble_Attack` (Number) — Mind Crumble attack stat
- `MindCrumble_Defence` (Number) — Mind Crumble defence stat
- `MindCrumble_Status` (Choice) — Mind Crumble status: Normal, Weakened, Enraged
- `MindCrumble_Debuffs` (Multiple lines of text) — JSON for debuff state

**Note:** Mind Crumble stats stored directly in `Games` record to avoid separate monster table for single villain.

### 3. Summit - Crumbleverse - ActionLog

**Purpose:** Audit trail of all game actions for debugging and replay.

**Columns:**
- `Title` (Single line of text, default) — Auto-generated summary
- `Timestamp` (Date/Time, required) — When action occurred
- `GameID` (Single line of text) — Game session identifier
- `ActorID` (Single line of text) — Player or "MindCrumble"
- `ActionType` (Choice) — Attack, Ability, Heal, MindCrumbleTurn, GameReset
- `Details` (Multiple lines of text) — JSON with action parameters
- `Result` (Multiple lines of text) — JSON with outcomes (damage dealt, HP changes, etc.)

**Example ActionLog entry:**
```json
{
  "ActorID": "london-digestive",
  "ActionType": "Attack",
  "Details": {"target": "MindCrumble", "ability": "Nibble"},
  "Result": {"damage": 4, "targetHP": 16, "actorHP": 12}
}
```

### 4. Summit - Crumbleverse - ClassTemplates

**Purpose:** Canonical base stats and ability definitions for all classes.

**Columns:**
- `Title` (Single line of text, default) — Class name (e.g., "Biscuit", "MindCrumble")
- `ClassName` (Single line of text) — Alternative class identifier
- `BaseHP` (Number) — Starting hit points
- `BaseAttack` (Number) — Base attack stat
- `BaseDefence` (Number) — Base defence stat
- `AbilityName` (Single line of text) — Primary ability name
- `AbilityJson` (Multiple lines of text) — JSON ability definition
- `Description` (Multiple lines of text) — Class flavor text
- `ImageUrl` (Hyperlink) — Class icon/avatar URL
- `TemplateVersion` (Number) — Version for schema evolution

**Mind Crumble Template Example:**
```json
{
  "ClassName": "MindCrumble",
  "BaseHP": 20,
  "BaseAttack": 5,
  "BaseDefence": 2,
  "AbilityJson": {
    "type": "conditional",
    "name": "crumbBehavior",
    "conditions": [
      {"when": "hp<=5", "actions": [{"type": "heal", "target": "self", "amount": 1, "repeatFor": "playersWaiting"}]},
      {"when": "hp>5", "actions": [{"type": "damage", "target": "all", "amount": 3}]}
    ]
  }
}
```

## Permissions & Access

**Service Account:**  
Power Apps and Power Automate flows use app-based authentication or a dedicated service account with:
- Read/Write access to all four lists
- Member of SharePoint site

**Workshop Participants:**  
- Read-only access to `ClassTemplates`
- No direct SharePoint access (all updates via Power Apps/Power Automate)

## Configuration Notes

**Choice Column Values:**

`Class` (Players list):
- Biscuit
- Warrior
- Mage
- Healer
- Tank
- Rogue

`Status` (Games list):
- NotStarted
- InProgress
- Completed
- Paused

`MindCrumble_Status` (Games list):
- Normal
- Weakened
- Enraged

`ActionType` (ActionLog):
- Attack
- Ability
- Heal
- MindCrumbleTurn
- GameReset
- PlayerJoin

## Testing & Validation

**Validation Steps:**
1. ✓ All lists created with correct column types
2. ✓ AgentID column set as indexed for performance
3. ✓ Choice columns populated with allowed values
4. ✓ Test record created in each list
5. ✓ Power Apps can connect and read data
6. ✓ Power Automate can write to lists

**Test Data:**
- Sample player: `AgentID = "sample-1"`, `Class = "Biscuit"`, `HP = 12`
- Sample game: `GameID = "test-game"`, `Status = "NotStarted"`
- ClassTemplates seeded with all six biscuit classes + MindCrumble

## Known Issues & Limitations

**SharePoint Limitations:**
- List view threshold: 5000 items (not an issue for workshop scale ~50 players)
- Concurrent update conflicts: mitigated with ETag/version checking in flows
- JSON columns stored as text: parsing required in Power Automate

**Schema Evolution:**
- `TemplateVersion` column allows for future schema updates
- JSON fields provide flexibility without schema changes
- Consider versioned backups before major updates

## Related Tasks

- **Task 0.1** — Initialize speckit project (dependency)
- **Task 0.3** — Power Apps connects to these lists (next)
- **Task 0.2b** — Seed ClassTemplates with all class data (follow-up)
- **Task 1.1** — Join Game flow uses Players list
- **Task 1.2** — AttackFlow updates Players and Games lists

## Future Enhancements

- Add `GameID` lookup column to Players for easier filtering
- Create SharePoint views for facilitator dashboard
- Add calculated columns for derived stats (e.g., damage modifiers)
- Implement row-level security for multi-game scenarios
- Add audit columns (CreatedBy, ModifiedBy) to all lists

---

**Task Completed:** November 2025  
**Duration:** ~2 hours (list creation + testing)  
**SharePoint Site:** [Link to site]
