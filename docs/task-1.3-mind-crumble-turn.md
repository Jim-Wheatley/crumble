# Task 1.3: Mind Crumble Turn Logic

**Status:** 🚧 In Progress  
**Date Started:** 6 December 2025  
**Date Completed:** _Pending_

## Overview

This task implements the MindCrumbleTurnFlow Power Automate flow that handles the Mind Crumble's turn after each player action. The Mind Crumble has conditional behavior based on its HP: when HP ≤ 5, it heals itself for each waiting player; when HP > 5, it attacks all players for 3 damage. This flow updates player HP, logs actions, posts to Teams, and returns results.

**Important:** This flow uses the **Power Apps (V2)** trigger and follows the same patterns established in JoinGameFlow (Task 1.1) and AttackFlow (Task 1.2). All SharePoint column names are case-sensitive, OData filters use concat for dynamic values, and all 200 responses must have identical schemas.

**When to Call This Flow:**
- Automatically triggered **after** a successful AttackFlow completes
- Can be called manually from Power Apps for testing
- Should NOT run if Mind Crumble is already defeated (HP = 0)

---

## Components to Implement

### 1. Power Automate Flow: MindCrumbleTurnFlow

**Trigger:** Power Apps (V2)

**Inputs:**
- `triggerAgentId` (text) - The agentId of the player who triggered the Mind Crumble's turn (for logging purposes)

**Flow Logic (Step-by-Step):**

#### Step 1: Initialize Variables
Add these "Initialize variable" actions at the top of your flow:

- `varGameId` (String) = `"current-game"`
- `varLockTimeout` (Integer) = `30`
- `varMindCrumbleHP` (Integer) = `0`
- `varMindCrumbleAttack` (Integer) = `0`
- `varMindCrumbleDefence` (Integer) = `0`
- `varMindCrumbleAction` (String) = `""` (will be "heal" or "attack")
- `varDamageAmount` (Integer) = `3` (damage dealt to all players when HP > 5)
- `varHealAmount` (Integer) = `1` (heal per waiting player when HP ≤ 5)
- `varTotalHealing` (Integer) = `0`
- `varPlayersAffected` (Integer) = `0`
- `varActionDetails` (String) = `""`
- `varErrorMessage` (String) = `""`

#### Step 2: Get Current Game Record
**Action:** Get items (SharePoint)
- **List name:** Games
- **Filter Query:** Leave blank (there's only one record: "current-game")
- **Top Count:** 1

#### Step 3: Check if Game Exists
**Action:** Condition

**How to configure:**
1. Add a "Condition" action
2. In the first "Choose a value" box, click "Expression" tab
3. Enter:
   ```
   greater(length(outputs('Get_items_-_Games')?['body/value']), 0)
   ```
4. Click "OK"
5. Middle dropdown: **"is equal to"**
6. Third box: `true`

**IF NO (Game Not Found):**
- **Action:** Respond to a PowerApp or flow
  - `success` (string): `"false"`
  - `message` (string): `"Game not found"`
  - `action` (string): `""`
  - `playersAffected` (number): `0`
  - `details` (string): `""`
- **Action:** Terminate (Failed)

**IF YES (Game Exists) - Continue with steps 4-22:**

#### Step 4: Set Mind Crumble Stats Variables
**Action:** Set variable (varMindCrumbleHP)
- **Name:** varMindCrumbleHP
- **Value:** In the Expression tab, enter:
  ```
  int(first(outputs('Get_items_-_Games')?['body/value'])?['MindCrumble_HP'])
  ```

**Action:** Set variable (varMindCrumbleAttack)
- **Name:** varMindCrumbleAttack
- **Value:** In the Expression tab, enter:
  ```
  int(first(outputs('Get_items_-_Games')?['body/value'])?['MindCrumble_Attack'])
  ```

**Action:** Set variable (varMindCrumbleDefence)
- **Name:** varMindCrumbleDefence
- **Value:** In the Expression tab, enter:
  ```
  int(first(outputs('Get_items_-_Games')?['body/value'])?['MindCrumble_Defence'])
  ```

#### Step 5: Check if Mind Crumble is Already Defeated
**Action:** Condition

**How to configure:**
1. Add a "Condition" action
2. In the first "Choose a value" box, click "Dynamic content" tab
3. Select **varMindCrumbleHP** variable
4. Middle dropdown: **"is equal to"**
5. Third box: `0`

**IF YES (Mind Crumble Defeated - HP = 0):**
- **Action:** Respond to a PowerApp or flow
  - `success` (string): `"true"`
  - `message` (string): `"Mind Crumble is already defeated!"`
  - `action` (string): `"defeated"`
  - `playersAffected` (number): `0`
  - `details` (string): `"Mind Crumble HP is 0"`
- **Action:** Terminate (Succeeded)

**IF NO (Mind Crumble Still Active) - Continue with steps 6-22:**

#### Step 6: Acquire Lock on Game Record
**Action:** Update item (SharePoint)
- **List name:** Games
- **Id:** In the Expression tab, enter:
  ```
  first(outputs('Get_items_-_Games')?['body/value'])?['ID']
  ```
- **LockedBy:** `@{triggerBody()?['text']}`
- **LockExpiresAt:** In the Expression tab, enter:
  ```
  addSeconds(utcNow(), variables('varLockTimeout'))
  ```

#### Step 7: Determine Mind Crumble Action (Conditional Behavior)
**Action:** Condition

**How to configure:**
1. Add a "Condition" action
2. In the first "Choose a value" box, click "Dynamic content" tab
3. Select **varMindCrumbleHP** variable
4. Middle dropdown: **"is less than or equal to"**
5. Third box: `5`

**What this does:** Checks if Mind Crumble HP ≤ 5 (heal mode) or HP > 5 (attack mode)

---

### Branch A: IF YES (HP ≤ 5 - Heal Mode)

#### Step 8a: Set Action Variable to "heal"
**Action:** Set variable
- **Name:** varMindCrumbleAction
- **Value:** `heal`

#### Step 9a: Get All Active Players
**Action:** Get items (SharePoint)
- **List name:** Players
- **Filter Query:** Leave blank (get all players)
- **Top Count:** 100

#### Step 10a: Calculate Total Healing
**Action:** Set variable (varTotalHealing)
- **Name:** varTotalHealing
- **Value:** In the Expression tab, enter:
  ```
  mul(length(outputs('Get_items_-_All_Players')?['body/value']), variables('varHealAmount'))
  ```
- **Explanation:** Total healing = number of players × 1 HP per player

#### Step 11a: Set Players Affected Count
**Action:** Set variable
- **Name:** varPlayersAffected
- **Value:** In the Expression tab, enter:
  ```
  length(outputs('Get_items_-_All_Players')?['body/value'])
  ```

#### Step 12a: Calculate New Mind Crumble HP (After Heal)
**Action:** Compose (name it: Compose_-_NewMindCrumbleHP_Heal)
- **Inputs:** In the Expression tab, enter:
  ```
  add(variables('varMindCrumbleHP'), variables('varTotalHealing'))
  ```
- **Explanation:** New HP = Current HP + Total Healing

#### Step 13a: Update Game Record (Heal & Release Lock)
**Action:** Update item (SharePoint)
- **List name:** Games
- **Id:** In the Expression tab, enter:
  ```
  first(outputs('Get_items_-_Games')?['body/value'])?['ID']
  ```
- **MindCrumble_HP:** (Dynamic content) Select **Compose_-_NewMindCrumbleHP_Heal** output
- **LockedBy:** Leave empty (clears the lock)
- **LockExpiresAt:** Leave empty (clears the lock)

#### Step 14a: Set Action Details
**Action:** Set variable
- **Name:** varActionDetails
- **Value:** In the Expression tab, enter:
  ```
  concat('Mind Crumble healed for ', string(variables('varTotalHealing')), ' HP (', string(variables('varPlayersAffected')), ' players × ', string(variables('varHealAmount')), ' HP each)')
  ```

---

### Branch B: IF NO (HP > 5 - Attack Mode)

#### Step 8b: Set Action Variable to "attack"
**Action:** Set variable
- **Name:** varMindCrumbleAction
- **Value:** `attack`

#### Step 9b: Get All Active Players
**Action:** Get items (SharePoint)
- **List name:** Players
- **Filter Query:** Leave blank (get all players)
- **Top Count:** 100

#### Step 10b: Set Players Affected Count
**Action:** Set variable
- **Name:** varPlayersAffected
- **Value:** In the Expression tab, enter:
  ```
  length(outputs('Get_items_-_All_Players_Attack')?['body/value'])
  ```

#### Step 11b: Apply Damage to Each Player (Loop)
**Action:** Apply to each
- **Select an output from previous steps:** (Dynamic content) Select **value** from "Get items - All Players Attack" (Step 9b)

**Inside the loop, add these actions:**

**Step 11b-1: Calculate Player New HP**
**Action:** Compose (name it: Compose_-_PlayerNewHP)
- **Inputs:** In the Expression tab, enter:
  ```
  max(sub(int(item()?['HP']), variables('varDamageAmount')), 0)
  ```
- **Explanation:** New HP = Current HP - 3 damage, minimum 0

**Step 11b-2: Update Player HP**
**Action:** Update item (SharePoint)
- **List name:** Players
- **Id:** In the Expression tab, enter:
  ```
  item()?['ID']
  ```
- **HP:** (Dynamic content) Select **Compose_-_PlayerNewHP** output

#### Step 12b: Set Action Details
**Action:** Set variable
- **Name:** varActionDetails
- **Value:** In the Expression tab, enter:
  ```
  concat('Mind Crumble attacked all players for ', string(variables('varDamageAmount')), ' damage (', string(variables('varPlayersAffected')), ' players affected)')
  ```

#### Step 13b: Release Lock on Game Record
**Action:** Update item (SharePoint)
- **List name:** Games
- **Id:** In the Expression tab, enter:
  ```
  first(outputs('Get_items_-_Games')?['body/value'])?['ID']
  ```
- **LockedBy:** Leave empty (clears the lock)
- **LockExpiresAt:** Leave empty (clears the lock)

---

### Steps After Both Branches (Continue from Step 15)

#### Step 15: Log Action to ActionLog
**Action:** Create item (SharePoint)
- **List name:** ActionLog
- **Title:** In the Expression tab, enter:
  ```
  concat('Mind Crumble Turn - ', variables('varMindCrumbleAction'))
  ```
- **ActorAgentID:** `Mind Crumble`
- **ActionType:** In the Expression tab, enter:
  ```
  if(equals(variables('varMindCrumbleAction'), 'heal'), 'Heal', 'Attack')
  ```
- **Details:** In the Expression tab, enter:
  ```
  concat('{"action":"', variables('varMindCrumbleAction'), '","playersAffected":', string(variables('varPlayersAffected')), ',"amount":', if(equals(variables('varMindCrumbleAction'), 'heal'), string(variables('varTotalHealing')), string(variables('varDamageAmount'))), '}')
  ```
- **Result:** In the Expression tab, enter:
  ```
  concat('{"mindCrumbleHP":', string(if(equals(variables('varMindCrumbleAction'), 'heal'), outputs('Compose_-_NewMindCrumbleHP_Heal'), variables('varMindCrumbleHP'))), ',"success":true}')
  ```

#### Step 16: Post to Teams (Optional)
**Action:** Post adaptive card in a chat or channel (Teams)
- **Post as:** Flow bot
- **Post in:** Channel
- **Team:** [Select your team]
- **Channel:** [Select your channel]
- **Adaptive Card:** Paste this JSON:
  ```json
  {
    "$schema": "http://adaptivecards.io/schemas/adaptive-card.json",
    "type": "AdaptiveCard",
    "version": "1.4",
    "body": [
      {
        "type": "Container",
        "style": "emphasis",
        "items": [
          {
            "type": "ColumnSet",
            "columns": [
              {
                "type": "Column",
                "width": "auto",
                "items": [
                  {
                    "type": "Image",
                    "url": "https://raw.githubusercontent.com/microsoft/fluentui-emoji/main/assets/Cookie/3D/cookie_3d.png",
                    "size": "Medium"
                  }
                ]
              },
              {
                "type": "Column",
                "width": "stretch",
                "items": [
                  {
                    "type": "TextBlock",
                    "text": "🌀 Mind Crumble's Turn",
                    "weight": "Bolder",
                    "size": "Large",
                    "color": "@{if(equals(variables('varMindCrumbleAction'), 'heal'), 'Good', 'Attention')}"
                  },
                  {
                    "type": "TextBlock",
                    "text": "@{variables('varActionDetails')}",
                    "wrap": true,
                    "size": "Medium"
                  }
                ]
              }
            ]
          }
        ]
      },
      {
        "type": "FactSet",
        "facts": [
          {
            "title": "Action:",
            "value": "@{if(equals(variables('varMindCrumbleAction'), 'heal'), '💚 Heal', '⚔️ Attack')}"
          },
          {
            "title": "Mind Crumble HP:",
            "value": "@{if(equals(variables('varMindCrumbleAction'), 'heal'), outputs('Compose_-_NewMindCrumbleHP_Heal'), variables('varMindCrumbleHP'))}"
          },
          {
            "title": "Players Affected:",
            "value": "@{variables('varPlayersAffected')}"
          },
          {
            "title": "Triggered by:",
            "value": "@{triggerBody()?['text']}"
          }
        ]
      }
    ]
  }
  ```

#### Step 17: Return Response to Power Apps
**Action:** Respond to a PowerApp or flow

**Body Configuration:**
- `success` (string): 
  - Click in the field, go to **Expression** tab
  - Enter: `string(true)`
- `message` (string):
  - Click in the field, go to **Dynamic content** tab
  - Select **varActionDetails** variable
- `action` (string):
  - Click in the field, go to **Dynamic content** tab
  - Select **varMindCrumbleAction** variable
- `playersAffected` (number):
  - Click in the field, go to **Dynamic content** tab
  - Select **varPlayersAffected** variable
- `details` (string):
  - Click in the field, go to **Expression** tab
  - Enter: 
    ```
    if(equals(variables('varMindCrumbleAction'), 'heal'), concat('Healed: +', string(variables('varTotalHealing')), ' HP'), concat('Damage: -', string(variables('varDamageAmount')), ' HP to all'))
    ```

---

## 2. Update AttackFlow to Call MindCrumbleTurnFlow

### Modify AttackFlow (Task 1.2)

After **Step 18** (Log Action to ActionLog) and before **Step 19** (Post to Teams), add these new steps:

#### New Step 18a: Check if Mind Crumble is Still Alive
**Action:** Condition

**How to configure:**
1. Add a "Condition" action
2. In the first "Choose a value" box, click "Dynamic content" tab
3. Select **Compose_-_NewMindCrumbleHP** output (from Step 15 of AttackFlow)
4. Middle dropdown: **"is greater than"**
5. Third box: `0`

**IF YES (Mind Crumble Still Alive - HP > 0):**

#### New Step 18a-1: Call MindCrumbleTurnFlow
**Action:** Run a Child Flow (Power Automate)
- **Child Flow:** Select **MindCrumbleTurnFlow** from the dropdown
- **triggerAgentId:** `@{triggerBody()?['text']}`

#### New Step 18a-2: Store Mind Crumble Turn Result
**Action:** Compose (name it: Compose_-_MindCrumbleTurnResult)
- **Inputs:** (Dynamic content) Select the entire output from "MindCrumbleTurnFlow" action

**IF NO (Mind Crumble Defeated):**
- No action needed - continue to Teams post

**Important:** After adding this condition, the Teams adaptive card (Step 19) and Response action (Step 20) should be **outside** the condition (after both branches merge) so they always execute.

---

## 3. Power Apps Integration (Optional Manual Trigger for Testing)

### Add Test Button to Screen_AgentDetail

**Note:** In production, this flow is called automatically by AttackFlow. This button is for testing only.

#### Button: btnMindCrumbleTurn (Test Only)
- **Text:** "🧪 Test Mind Crumble Turn"
- **Visible:** `false` (or `true` during testing, set to `false` in production)
- **OnSelect:** Paste this formula:
  ```powerquery
  // Test trigger for Mind Crumble turn
  Set(
      varMCTurnResult,
      MindCrumbleTurnFlow.Run(gblAgentId)
  );
  
  // Refresh to see updated HP
  Refresh(Players);
  Refresh(Games);
  
  // Show result
  Notify(
      varMCTurnResult.message,
      If(
          varMCTurnResult.success,
          NotificationType.Success,
          NotificationType.Error
      )
  );
  ```

---

## 4. SharePoint Lists Requirements

### Games List Columns (verify these exist):
- `MindCrumble_HP` (Number) - ✅ Should already exist from Task 1.2
- `MindCrumble_Attack` (Number) - ✅ Should already exist from Task 1.2
- `MindCrumble_Defence` (Number) - ✅ Should already exist from Task 1.2
- `MindCrumble_Status` (Choice: Active, Defeated) - ✅ Should already exist from Task 1.2
- `MindCrumble_Debuffs` (Multiple lines of text) - ✅ Should already exist from Task 1.2
- `LockedBy` (Single line of text) - ✅ Should already exist from Task 1.1
- `LockExpiresAt` (Date and Time) - ✅ Should already exist from Task 1.1

### Players List Columns (verify these exist):
- `HP` (Number) - Should already exist from Task 0.2/1.1
- `MaxHP` (Number) - Should already exist from Task 0.2/1.1
- `AgentID` (Single line of text) - ✅ Should already exist from Task 1.1

### ActionLog List Columns (verify these exist):
- `Title` (Single line of text) - ✅ Should already exist from Task 1.2
- `ActorAgentID` (Single line of text) - ✅ Should already exist from Task 1.2
- `ActionType` (Choice: Join, Attack, Defend, Heal) - ✅ Should already exist from Task 1.2
- `Details` (Multiple lines of text) - ✅ Should already exist from Task 1.2
- `Result` (Multiple lines of text) - ✅ Should already exist from Task 1.2
- `Created` (Date and Time) - auto-generated

### ClassTemplates List - Mind Crumble AbilityJson

**Do you need to update the list?** Yes, you need to add a Mind Crumble template row if it doesn't exist.

#### How to Add Mind Crumble Template:

1. **Go to your SharePoint site** and open the **ClassTemplates** list
2. **Create a new item** with these values:
   - **ClassName (Title):** `MindCrumble`
   - **BaseHP:** `30` (or adjust based on your game balance)
   - **BaseAttack:** `5`
   - **BaseDefence:** `3`
   - **AbilityName:** `Crumb Behavior`
   - **AbilityJson:** Paste this JSON:
     ```json
     {
       "type": "conditional",
       "name": "crumbBehavior",
       "conditions": [
         {
           "when": "hp<=5",
           "actions": [
             {
               "type": "heal",
               "target": "self",
               "amount": 1,
               "repeatFor": "playersWaiting"
             }
           ]
         },
         {
           "when": "hp>5",
           "actions": [
             {
               "type": "damage",
               "target": "all",
               "amount": 3
             }
           ]
         }
       ]
     }
     ```
   - **Description:** `The dreaded Mind Crumble - attacks all players when strong, heals when weak`
   - **ImageUrl:** (Optional) URL to a villain image
   - **TemplateVersion:** `1.0`

3. **Save the item**

**Note:** For Task 1.3, this JSON is for documentation purposes. The conditional logic is implemented directly in the Power Automate flow (Steps 7-14). Future enhancements could parse this JSON dynamically.

---

## 5. Testing Checklist

### Before Testing:
1. ✅ Ensure Games list has one record with `Title = "current-game"`
2. ✅ Set initial Mind Crumble HP (e.g., 30)
3. ✅ Set Mind Crumble Attack (e.g., 5)
4. ✅ Set Mind Crumble Defence (e.g., 3)
5. ✅ Ensure at least 2-3 players have joined the game (from Task 1.1)
6. ✅ Ensure players have HP > 0

### Test Scenarios:

**Test 1: Mind Crumble Attack Mode (HP > 5)**
- ✅ Set Mind Crumble HP to 20
- ✅ Trigger: Use test button or complete an AttackFlow
- ✅ Expected: 
  - All players lose 3 HP
  - Player HP cannot go below 0
  - ActionLog entry created with ActionType = "Attack"
  - Teams card posted showing "⚔️ Attack"
  - Response message: "Mind Crumble attacked all players for 3 damage (X players affected)"

**Test 2: Mind Crumble Heal Mode (HP ≤ 5)**
- ✅ Reduce Mind Crumble HP to 4
- ✅ Ensure 3 players exist in the game
- ✅ Trigger: Use test button or complete an AttackFlow
- ✅ Expected: 
  - Mind Crumble heals 3 HP (1 HP × 3 players)
  - New Mind Crumble HP = 7 (4 + 3)
  - ActionLog entry created with ActionType = "Heal"
  - Teams card posted showing "💚 Heal"
  - Response message: "Mind Crumble healed for 3 HP (3 players × 1 HP each)"

**Test 3: Mind Crumble Already Defeated**
- ✅ Set Mind Crumble HP to 0
- ✅ Trigger: Use test button
- ✅ Expected: 
  - Flow returns success but no action taken
  - Response message: "Mind Crumble is already defeated!"
  - No players or game record updated
  - No Teams card posted

**Test 4: Boundary Test - HP Exactly 5**
- ✅ Set Mind Crumble HP to exactly 5
- ✅ Trigger: Use test button
- ✅ Expected: 
  - Mind Crumble heals (HP ≤ 5 condition triggers)
  - Not attack mode

**Test 5: Player HP Cannot Go Negative**
- ✅ Set a player's HP to 2
- ✅ Set Mind Crumble HP to 20 (attack mode)
- ✅ Trigger: Use test button
- ✅ Expected: 
  - Player HP becomes 0 (not -1)
  - Player record updated with HP = 0

**Test 6: Integration Test - Full Attack Sequence**
- ✅ Set Mind Crumble HP to 10
- ✅ Player attacks Mind Crumble (AttackFlow)
- ✅ AttackFlow completes successfully
- ✅ Expected: 
  - AttackFlow reduces Mind Crumble HP
  - MindCrumbleTurnFlow automatically triggers
  - Mind Crumble attacks all players (if HP > 5 after damage)
  - Both actions logged in ActionLog
  - Two Teams cards posted (one for player attack, one for Mind Crumble turn)

**Test 7: Concurrent Locks**
- ✅ Trigger two Mind Crumble turns simultaneously (e.g., two players attack at the same time)
- ✅ Expected: 
  - Lock prevents race condition
  - Both flows complete successfully (second waits for first to release lock)

---

## 6. Common Issues & Solutions

### Issue: "The execution of template action 'Apply_to_each' failed"
**Solution:** Ensure the "Get items - All Players" action returns valid data. Check that the dynamic content reference matches the exact action name (e.g., `Get_items_-_All_Players_Attack` vs `Get_items_-_All_Players`).

### Issue: Response schema mismatch - different fields in heal vs attack branches
**Solution:** Ensure BOTH branches end with identical "Respond to PowerApp" fields:
- `success` (string)
- `message` (string)
- `action` (string)
- `playersAffected` (number)
- `details` (string)

**Move the "Respond to PowerApp" action OUTSIDE both condition branches** so it executes once after both paths merge.

### Issue: varTotalHealing not available in "IF NO" branch
**Solution:** Initialize `varTotalHealing` at the top (Step 1) so it's available in both branches. In the attack branch, you won't use this variable (it stays 0).

### Issue: Compose_-_NewMindCrumbleHP_Heal not found in ActionLog step
**Solution:** This output only exists in the heal branch. For the ActionLog step (Step 15), use this expression for Result:
```
concat('{"mindCrumbleHP":', string(if(equals(variables('varMindCrumbleAction'), 'heal'), outputs('Compose_-_NewMindCrumbleHP_Heal'), variables('varMindCrumbleHP'))), ',"success":true}')
```
This checks which action was taken and uses the appropriate value.

### Issue: Lock not released after error in Apply to each loop
**Solution:** Add a "Scope" action around Steps 6-14 and configure "Run after" to release lock on failure:
1. Wrap Steps 6-14 in a "Scope" action
2. Add parallel branch after the scope
3. Configure "Run after" → Run if scope fails
4. Add "Update item" to clear LockedBy and LockExpiresAt

### Issue: Child flow call from AttackFlow fails
**Solution:** Ensure MindCrumbleTurnFlow is saved and published. Verify the trigger is "Power Apps (V2)" (not manual trigger). Check that the input parameter name matches exactly: `triggerAgentId`.

### Issue: Players HP goes negative
**Solution:** Use `max()` function in Compose_-_PlayerNewHP:
```
max(sub(int(item()?['HP']), variables('varDamageAmount')), 0)
```

---

## 7. Enhanced Features (Future Iterations)

### Sprint 1.4+ Enhancements:
- [ ] Parse AbilityJson from ClassTemplates dynamically (instead of hardcoded logic)
- [ ] Implement variable damage based on Mind Crumble Attack stat
- [ ] Apply player defence to reduce Mind Crumble damage
- [ ] Add status effects (e.g., "burning" debuff that deals damage over time)
- [ ] Implement Mind Crumble special abilities at different HP thresholds

### Sprint 2+ Enhancements:
- [ ] Add Mind Crumble_Status field update (Active → Defeated when HP = 0)
- [ ] Trigger celebratory Teams card when Mind Crumble is defeated
- [ ] Log individual player damage details (who took how much damage)
- [ ] Add retry logic for lock acquisition with exponential backoff
- [ ] Implement turn order validation (only allow Mind Crumble turn after valid player turn)

---

## 8. Power Automate Expression Reference

### Commonly Used Expressions in This Task:

**Check if array has items:**
```
greater(length(outputs('Get_items_-_Games')?['body/value']), 0)
```

**Count items in array:**
```
length(outputs('Get_items_-_All_Players')?['body/value'])
```

**Multiply variables:**
```
mul(length(outputs('Get_items_-_All_Players')?['body/value']), variables('varHealAmount'))
```

**Add variables:**
```
add(variables('varMindCrumbleHP'), variables('varTotalHealing'))
```

**Subtract with minimum value (max function):**
```
max(sub(int(item()?['HP']), variables('varDamageAmount')), 0)
```

**Conditional value (if-then-else):**
```
if(equals(variables('varMindCrumbleAction'), 'heal'), 'Heal', 'Attack')
```

**Concat strings:**
```
concat('Mind Crumble healed for ', string(variables('varTotalHealing')), ' HP')
```

**Access item in loop:**
```
item()?['HP']
item()?['ID']
```

**Get first item from array:**
```
first(outputs('Get_items_-_Games')?['body/value'])
```

**Convert to integer:**
```
int(first(outputs('Get_items_-_Games')?['body/value'])?['MindCrumble_HP'])
```

**Convert to string:**
```
string(variables('varTotalHealing'))
```

**Add seconds to current time:**
```
addSeconds(utcNow(), variables('varLockTimeout'))
```

---

## 9. Key Architecture Decisions

### Why Separate Flow for Mind Crumble Turn (vs Inline in AttackFlow)?
- **Modularity:** Mind Crumble logic can be reused for other triggers (e.g., time-based turns, special events)
- **Testability:** Can test Mind Crumble behavior independently without player attacks
- **Clearer separation of concerns:** Player action vs villain reaction
- **Easier to extend:** Future enhancements (different villain behaviors, multiple villains) won't bloat AttackFlow

### Why Conditional Logic in Flow (vs Parsing AbilityJson)?
- **Simpler for initial implementation:** Hardcoded conditions are easier to debug
- **Performance:** No JSON parsing overhead for every turn
- **Clear behavior:** Explicit if-then-else is easier to understand than generic JSON interpreter
- **Future-proof:** Can migrate to JSON parsing in later sprints without changing the API

### Why Heal Based on Player Count?
- **Game balance:** Prevents Mind Crumble from being invincible when low on HP
- **Encourages strategy:** Players must coordinate to defeat Mind Crumble before it heals too much
- **Scalable:** Works with any number of players (more players = more healing, but also more attackers)

### Why Attack All Players (vs Random or Highest HP)?
- **Fairness:** All players share the risk equally
- **Simplicity:** No complex targeting logic needed
- **Workshop-friendly:** Easy for participants to understand and predict
- **Creates urgency:** Encourages players to act quickly before taking too much damage

### Why Lock During Mind Crumble Turn?
- **Prevents race conditions:** Multiple players attacking simultaneously won't cause duplicate Mind Crumble turns
- **Atomic updates:** Ensures all player HP updates complete before next player action
- **Consistent with Task 1.2:** Same locking pattern as AttackFlow for maintainability

---

## 10. Flow Diagram (Visual Reference)

```
┌─────────────────────────────────────────────────────────────┐
│ MindCrumbleTurnFlow                                         │
├─────────────────────────────────────────────────────────────┤
│ 1. Initialize variables                                     │
│ 2. Get current game record                                  │
│ 3. Check if game exists → NO: Return error                  │
│                         → YES: Continue                      │
│ 4. Set Mind Crumble stats (HP, Attack, Defence)             │
│ 5. Check if defeated (HP = 0) → YES: Return "defeated"      │
│                                → NO: Continue                │
│ 6. Acquire lock on Games record                             │
│ 7. Check HP ≤ 5?                                            │
│    ├─ YES (Heal Mode):                                      │
│    │   8a. Set action = "heal"                              │
│    │   9a. Get all players                                  │
│    │   10a. Calculate total healing (players × 1 HP)        │
│    │   11a. Set players affected count                      │
│    │   12a. Calculate new Mind Crumble HP (HP + healing)    │
│    │   13a. Update Games (new HP & release lock)            │
│    │   14a. Set action details message                      │
│    │                                                         │
│    └─ NO (Attack Mode):                                     │
│        8b. Set action = "attack"                            │
│        9b. Get all players                                  │
│        10b. Set players affected count                      │
│        11b. Loop through each player:                       │
│             - Calculate new HP (HP - 3, min 0)              │
│             - Update player HP                              │
│        12b. Set action details message                      │
│        13b. Update Games (release lock)                     │
│                                                              │
│ 15. Log action to ActionLog                                 │
│ 16. Post adaptive card to Teams                             │
│ 17. Return response to Power Apps                           │
└─────────────────────────────────────────────────────────────┘
```

---

## 11. Integration with AttackFlow

### Updated AttackFlow Sequence (Task 1.2 + Task 1.3):

```
┌─────────────────────────────────────────────────────────────┐
│ AttackFlow (Modified)                                       │
├─────────────────────────────────────────────────────────────┤
│ ... (Steps 1-17 from Task 1.2 - unchanged) ...              │
│ 18. Log action to ActionLog                                 │
│ 18a. Check if Mind Crumble still alive (HP > 0)?            │
│      ├─ YES:                                                │
│      │   18a-1. Call MindCrumbleTurnFlow (child flow)       │
│      │   18a-2. Store result in variable                    │
│      └─ NO: Continue (Mind Crumble defeated)                │
│ 19. Post to Teams (player attack summary)                   │
│ 20. Return response to Power Apps                           │
└─────────────────────────────────────────────────────────────┘
```

### Example Response from AttackFlow (After Integration):

**Player Attack + Mind Crumble Turn Success:**
```json
{
  "success": "true",
  "message": "You dealt 8 damage!",
  "damageDealt": 8,
  "mindCrumbleHP": 12,
  "effects": "Smash"
}
```

**Then, the child flow (MindCrumbleTurnFlow) executes automatically:**
```json
{
  "success": "true",
  "message": "Mind Crumble attacked all players for 3 damage (4 players affected)",
  "action": "attack",
  "playersAffected": 4,
  "details": "Damage: -3 HP to all"
}
```

---

## 12. Testing with Sample Data

### Scenario 1: Heal Mode Triggered

**Initial State:**
- Mind Crumble HP: 4
- Players in game: 3
  - Player A (london-digestive): HP = 15
  - Player B (paris-bourbon): HP = 12
  - Player C (tokyo-pocky): HP = 18

**Action:** Trigger MindCrumbleTurnFlow

**Expected Results:**
- Mind Crumble HP: 7 (4 + 3 healing)
- Player HP unchanged
- ActionLog: ActionType = "Heal", Details = `{"action":"heal","playersAffected":3,"amount":3}`
- Teams card: "💚 Heal - Mind Crumble healed for 3 HP (3 players × 1 HP each)"

---

### Scenario 2: Attack Mode Triggered

**Initial State:**
- Mind Crumble HP: 20
- Players in game: 3
  - Player A (london-digestive): HP = 15
  - Player B (paris-bourbon): HP = 12
  - Player C (tokyo-pocky): HP = 18

**Action:** Trigger MindCrumbleTurnFlow

**Expected Results:**
- Mind Crumble HP: 20 (unchanged)
- Player A HP: 12 (15 - 3)
- Player B HP: 9 (12 - 3)
- Player C HP: 15 (18 - 3)
- ActionLog: ActionType = "Attack", Details = `{"action":"attack","playersAffected":3,"amount":3}`
- Teams card: "⚔️ Attack - Mind Crumble attacked all players for 3 damage (3 players affected)"

---

### Scenario 3: Player HP Goes to Zero

**Initial State:**
- Mind Crumble HP: 20
- Players in game: 2
  - Player A (london-digestive): HP = 5
  - Player B (paris-bourbon): HP = 2

**Action:** Trigger MindCrumbleTurnFlow

**Expected Results:**
- Mind Crumble HP: 20 (unchanged)
- Player A HP: 2 (5 - 3)
- Player B HP: 0 (2 - 3, clamped to 0, not -1)
- ActionLog: ActionType = "Attack", Details = `{"action":"attack","playersAffected":2,"amount":3}`

---

## 13. Troubleshooting Checklist

Before asking for help, verify:

- [ ] MindCrumbleTurnFlow is saved and published
- [ ] Trigger type is "Power Apps (V2)" (not Manual)
- [ ] Input parameter is named exactly `triggerAgentId` (text)
- [ ] All variables are initialized in Step 1
- [ ] SharePoint action names match the expressions (e.g., `Get_items_-_Games`)
- [ ] Both condition branches have identical response schemas
- [ ] Lock is released in BOTH branches (heal and attack)
- [ ] `max()` function is used to prevent negative HP
- [ ] `int()` function wraps all HP/Attack/Defence values
- [ ] ActionLog step uses `if()` to handle heal vs attack results
- [ ] Teams card references correct outputs based on action type
- [ ] AttackFlow has been updated to call MindCrumbleTurnFlow (Step 18a)

---

## Next Steps

After completing Task 1.3, proceed to:
- **Task 1.4:** Wire Power Apps Attack button (already done in Task 1.2!)
- **Task 1.5:** Update Copilot agent instructions with attack and Mind Crumble behavior
- **Task 1.6:** Update player creation to read ClassTemplates
- **Task 2.1:** Enhance locking with timeout validation and retry logic
- **Task 2.2:** Improve Teams adaptive card formatting with images and better styling
- **Task 2.3:** Create facilitator admin flows (reset game, configure Mind Crumble stats)

---

## Notes

- All SharePoint column names are **case-sensitive**
- Use **Modern (Fluent) controls** in Power Apps (`Value`, `SelectedItems`, not `Default`, `Selected`)
- OData filters require `concat()` for dynamic values
- All 200 responses must have identical schema
- Lock timeout is 30 seconds (configurable via `varLockTimeout`)
- Heal formula: `1 HP × number of players`
- Attack formula: `3 damage to all players`
- Player HP cannot go below 0
- Mind Crumble HP threshold: ≤ 5 (heal), > 5 (attack)

---

## Files Modified

- Power Automate: `MindCrumbleTurnFlow` (new flow)
- Power Automate: `AttackFlow` (modified - add child flow call)
- SharePoint: `ClassTemplates` (add MindCrumble row)
- Power Apps: `Screen_AgentDetail` (optional test button)

---

## Success Criteria

✅ MindCrumbleTurnFlow accepts triggerAgentId input  
✅ Flow checks Mind Crumble HP and determines action (heal vs attack)  
✅ When HP ≤ 5: Mind Crumble heals 1 HP per player  
✅ When HP > 5: Mind Crumble deals 3 damage to all players  
✅ Player HP updated correctly (minimum 0)  
✅ Games record lock acquired and released  
✅ ActionLog entry created with correct ActionType  
✅ Teams adaptive card posted with action summary  
✅ Response schema consistent across all branches  
✅ AttackFlow calls MindCrumbleTurnFlow automatically after successful attack  
✅ Mind Crumble turn only triggers if Mind Crumble is still alive (HP > 0)  

---

**Good luck implementing the Mind Crumble's turn logic! 🌀🍪**
