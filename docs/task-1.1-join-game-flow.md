# Task 1.1: Join Game Flow and TurnOrder Assignment

**Status:** ✅ Complete  
**Date Completed:** 3 December 2025

## Overview

Implemented the Join Game flow that allows players to register for the game with turn order assignment. Players arrive via deep links with their unique `agentId`, `displayName`, and `className` parameters, and the system creates their player record with stats from the ClassTemplates list.

**Important:** This implementation uses **Modern (Fluent) controls** in Power Apps, which have different property names and behaviors compared to Classic controls. See "Key Learnings & Solutions" section for syntax differences.

## Components Implemented

### 1. Power Automate Flow: JoinGameFlow

**Trigger:** Power Apps (V2)

**Inputs:**
- `agentId` (text) - Unique player ID (e.g., `london-digestive`)
- `displayName` (text_1) - Player's display name
- `className` (text_2) - Selected biscuit class (e.g., `Bourbon Rogue`)

**Flow Logic:**

1. **Initialize Variables**
   - `varGameId` = "current-game"
   - `varLockTimeout` = 30 (seconds)
   - `varPlayerExists` = false
   - `varNextTurnOrder` = 0
   - `varPlayerClass` = "" (empty string - will be populated from input)

2. **Get Class Stats from ClassTemplates**
   - Filter Query: `ClassName eq 'className'` (using concat for dynamic OData filter)
   - Handles class names with spaces correctly

3. **Set Class Variable**
   - Store className from input in `varPlayerClass` for use in Teams card
   - Value: `triggerBody()?['text_2']`

4. **Check if Player Already Exists**
   - Filter Query: `AgentID eq 'agentId'`
   - Uses length of results array to determine existence

5. **Condition: Player ID Exists**
   - **TRUE branch:** Return error "Player already exists"
   - **FALSE branch:** Create new player (steps 6-13)

6. **Get Current Game Record**
   - Filter: `Title eq 'current-game'`
   - Returns the single active game instance

7. **Acquire Lock**
   - Update Games record:
     - `LockedBy` = agentId
     - `LockExpiresAt` = current time + 30 seconds

8. **Get Current Player Count**
   - Retrieve all players from Players list
   - Calculate next turn order: `length(players) + 1`

9. **Create New Player Record**
   - `PlayerID` = agentId
   - `DisplayName` = displayName
   - `Class` = className
   - `HP` = BaseHP from ClassTemplates
   - `MaxHP` = BaseHP from ClassTemplates
   - `Attack` = BaseAttack from ClassTemplates
   - `Defence` = BaseDefence from ClassTemplates
   - `TurnOrder` = varNextTurnOrder
   - `AbilityState` = AbilityJson from ClassTemplates
   - `LastActionAt` = current UTC time

10. **Update Games PlayerCount**
    - Set `PlayerCount` = varNextTurnOrder

11. **Release Lock**
    - Clear `LockedBy` and `LockExpiresAt` fields

12. **Log Action**
    - Create entry in ActionLog:
      - `ActorID` = agentId
      - `ActionType` = "Join"
      - `Details` = JSON with className and turnOrder
      - `Result` = JSON with success message

13. **Post Adaptive Card to Teams** (Optional)
    - Posts a formatted card showing the new player's details
    - Displays player class image, name, agent ID, and turn order
    - Runs after player is created and action is logged

14. **Return Response**
    - `success` (string): "true" or "false"
    - `message` (string): Success/error message
    - `turnorder` (number): Assigned turn order

### 2. Power Apps Integration

**App OnStart:**
```powerquery
Set(varAgentId, Coalesce(Param("agentId"), "preview-" & Text(Now(), "[$-en-US]yyyymmddhhmmss")));
Set(varDisplayName, Coalesce(Param("displayName"), User().FullName));
Set(varClassName, Coalesce(Param("className"), "Bourbon Rogue"));
```

**Controls:**
- `txtDisplayName` (Text Input)
  - Property: `Value` = `varDisplayName`
  
- `ddClassName` (Dropdown - Modern)
  - Property: `Items` = `["Digestive","HobNob","Oreo","Custard Cream","Bourbon Rogue","Jammie Dodger"]`
  
- `btnJoinGame` (Button)
  - Property: `OnSelect`:
    ```powerquery
    Set(
        varJoinResult,
        JoinGameFlow.Run(
            varAgentId,
            txtDisplayName.Text,
            First(ddClassName.SelectedItems).Value
        )
    );
    If(
        varJoinResult.success = "true",
        Notify(
            varJoinResult.message & " (Turn: " & Text(varJoinResult.turnorder) & ")",
            NotificationType.Success
        ),
        Notify(varJoinResult.message, NotificationType.Error)
    )
    ```

**Deep Link Format:**
```
https://apps.powerapps.com/play/<AppId>?tenantId=<TenantId>&agentId=london-digestive&displayName=Test%20Player&className=Bourbon%20Rogue
```

## Key Learnings & Solutions

### OData Filter Syntax
- SharePoint filter queries require proper OData syntax
- Spaces in values must be handled with concat:
  ```
  concat('ClassName eq ''', triggerBody()?['text_2'], '''')
  ```
- Column names are case-sensitive

### Modern vs Classic Controls
- **Modern Controls:**
  - Text Input: `Value` property (not `Default`)
  - Dropdown: `DefaultSelectedItems` expects table (e.g., `[{Value: varClassName}]`)
  - Dropdown selection: `First(ddClassName.SelectedItems).Value`
  
- **Classic Controls:**
  - Text Input: `Default` property
  - Dropdown: `Default` property, `Selected.Value`

### Condition Expressions
- Must use evaluated expressions, not strings
- Correct: `length(outputs('ActionName')?['body/value'])`
- Wrong: `"length(outputs('ActionName')?['body/value'])"`

### Response Schema Consistency
- All 200 responses must have identical schema
- Required fields: `success` (string), `message` (string), `turnorder` (number)

### Variable Initialization
- `Initialize variable` must come early in flow (before branching)
- `Set variable` can be used anywhere after initialization

### Action Name References
- Power Automate expressions must reference exact action names
- Example: `outputs('Check_if_player_exists')?['body/value']` 
- Action names use underscores for spaces (e.g., `Get_items_-_current_player_count`)

### User Experience Note
- Provide beginner-level, step-by-step instructions with clear code snippets
- Specify exact locations (e.g., "In the Expression tab, enter..." vs "Use this expression")
- Include both success and error scenarios in testing

## Testing

### Test Scenarios Verified:
1. ✅ New player joins with unique agentId → Player created with stats and turn order
2. ✅ Existing player attempts to join → Error message returned
3. ✅ Deep link parameters passed correctly → App populates fields
4. ✅ Preview mode without deep link → App generates test agentId
5. ✅ Class name with spaces → Correctly filtered from ClassTemplates
6. ✅ Lock acquired and released → Games record updated correctly
7. ✅ ActionLog entry created → Join action logged with details
8. ✅ Teams card posted → Adaptive card displays player image, name, class, agent ID, and turn order

### SharePoint Lists Updated:
- `Players` - New player row with stats from ClassTemplates
- `Games` - PlayerCount incremented, lock acquired/released
- `ActionLog` - Join action logged

## Teams Adaptive Card

The flow posts an adaptive card to Teams displaying:
- Player class image (dynamically selected based on class)
- "🎉 New Player Joined!" header
- Player name, class, agent ID, and turn order in FactSet format

Card JSON includes nested if-statements to select the correct image URL for each class (VienneseWizard, BourbonRogue, HobnobBarbarian, CustardCleric, JammieBard, DigestiveFighter).

## Prerequisites

### SharePoint Lists Required:
- `Players` (with columns: AgentID, Title, Class, HP, MaxHP, Attack, Defence, TurnOrder, AbilityState, LastActionAt)
- `Games` (with columns: Title, PlayerCount, LockedBy, LockExpiresAt, + MindCrumble fields)
- `ActionLog` (with columns: ActorAgentID, ActionType, Details, Result)
- `ClassTemplates` (with columns: ClassName, BaseHP, BaseAttack, BaseDefence, AbilityJson)

### Games List Setup:
- Must contain one record with Title = "current-game"
- Initial values:
  - PlayerCount = 0
  - Status = "Active"

### Teams Setup:
- Flow must be connected to your Teams instance
- Specify the target Team and Channel for adaptive card posting (see "Post adaptive card in a chat or channel" action)

## Files Modified

- Power Automate: `JoinGameFlow` (14 steps including Teams card posting)
- Power Apps: App OnStart, Screen_AgentDetail (Join button)

## Next Steps

- [ ] Task 1.2: Implement AttackFlow logic
- [ ] Bind ddClassName.Items to ClassTemplates SharePoint list for dynamic options
- [ ] Add lock timeout validation before acquiring lock
- [ ] Add Teams adaptive card notification on player join

## Notes

- Using Modern (Fluent) controls in Power Apps
- `Screen_Home` can be removed - app starts directly on `Screen_AgentDetail`
- Deep link params are URL-encoded (spaces = %20)
- Preview mode generates unique agentId per session for testing
