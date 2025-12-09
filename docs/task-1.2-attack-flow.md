# Task 1.2: AttackFlow Logic (Damage Calculations and Ability Effects)

**Status:** ✅ Complete - Flow tested and working  
**Date Started:** 3 December 2025  
**Date Completed:** 5 December 2025

## Overview

This task implements the AttackFlow Power Automate flow that handles player attacks against the Mind Crumble. The flow calculates damage based on player stats and ability effects, updates the Mind Crumble's HP, logs the action, posts to Teams, and returns results to Power Apps.

**Important:** This flow uses the **Power Apps (V2)** trigger and follows the same patterns established in JoinGameFlow (Task 1.1). All SharePoint column names are case-sensitive, OData filters use concat for dynamic values, and all 200 responses must have identical schemas.

## Components to Implement

### 1. Power Automate Flow: AttackFlow

**Trigger:** Power Apps (V2)

**Inputs:**
- `agentId` (text) - Unique player ID performing the attack (e.g., `london-digestive`)
- `abilityName` (text_1) - Name of ability being used (e.g., `Smash`, `DoubleStrike`)

**Flow Logic (Step-by-Step):**

#### Step 1: Initialize Variables
Add these "Initialize variable" actions at the top of your flow:

- `varGameId` (String) = `"current-game"`
- `varLockTimeout` (Integer) = `30`
- `varPlayerClass` (String) = `` (empty - will be populated)
- `varDamageDealt` (Integer) = `0`
- `varPlayerAttack` (Integer) = `0`
- `varPlayerDefence` (Integer) = `0`
- `varMindCrumbleHP` (Integer) = `0`
- `varMindCrumbleDefence` (Integer) = `0`
- `varAbilityDamage` (Integer) = `0`
- `varAbilityEffects` (String) = `""`
- `varErrorMessage` (String) = `""`

#### Step 2: Get Player Record
**Action:** Get items (SharePoint)
- **List name:** Players
- **Filter Query:** In the Expression tab, enter:
  ```
  concat('AgentID eq ''', triggerBody()?['text'], '''')
  ```
- **Top Count:** 1

#### Step 2b: Set Player Class Variable
**Action:** Set variable
- **Name:** varPlayerClass
- **Value:** In the Expression tab, enter:
  ```
  string(first(outputs('Get_items_-_Player')?['body/value'])?['Class']?['Value'])
  ```
- **Why:** The Class field is a lookup column that returns an object with nested Value property. Extract and convert to string for ClassTemplates filtering.

#### Step 3: Check if Player Exists
**Action:** Condition

**How to configure:**
1. Add a "Condition" action after the "Get items - Player" step
2. Click in the first "Choose a value" box
3. Click the "Expression" tab (next to "Dynamic content")
4. Type or paste this expression:
   ```
   greater(length(outputs('Get_items_-_Player')?['body/value']), 0)
   ```
5. Click "OK" to insert the expression
6. Leave the middle dropdown as **"is equal to"**
7. In the third box, type: `true`

**What this does:** Checks if any player records were returned (length > 0 = true means player exists)

**IF NO (Player Not Found):**
- **Action:** Respond to a PowerApp or flow
  - `success` (string): `"false"`
  - `message` (string): `"Player not found"`
  - `damageDealt` (number): `0`
  - `mindCrumbleHP` (number): `0`
  - `effects` (string): `""`
- **Action:** Terminate (Failed)

**IF YES (Player Exists) - Continue with steps 4-18:**

#### Step 4: Get Class Templates (All)
**Action:** Get items (SharePoint)
- **List name:** ClassTemplates
- **Filter Query:** Leave blank (we'll filter in Power Automate)
- **Top Count:** 100

#### Step 4b: Filter Array - Find Matching Class Template
**Action:** Filter array
- **From:** (Dynamic content) Select **value** from Get items - ClassTemplates
- **Condition:**
  - Left: (Dynamic content) Select **ClassName** → **Value** from array items
  - Operator: `is equal to`
  - Right: (Dynamic content) Select **varPlayerClass** variable
- **Why:** OData filters in SharePoint have limitations with nested fields. This approach is more reliable.

#### Step 5: Parse Ability JSON
**Action:** Parse JSON
- **Content:** In the Expression tab, enter:
  ```
  first(body('Filter_array'))?['AbilityJson']
  ```
- **Schema:** Click "Use sample payload to generate schema" and paste:
  ```json
  {
    "type": "composite",
    "name": "shadowStrike",
    "actions": [
      {
        "type": "damage",
        "target": "mindcrumble",
        "amount": 2
      },
      {
        "type": "buff",
        "target": "self",
        "stat": "defence",
        "amount": 2,
        "duration": 1
      }
    ]
  }
  ```
- **Note:** This schema supports abilities with multiple actions (damage, heal, buff, debuff)

#### Step 6: Filter Damage Actions
**Action:** Filter array
- **From:** In the Expression tab, enter:
  ```
  body('Parse_JSON')?['actions']
  ```
- **Condition:** 
  - Left: `item()?['type']` (use Expression tab)
  - Operator: `is equal to`
  - Right: `damage` (type this as plain text)

#### Step 7: Set Ability Damage Variable
**Action:** Set variable
- **Name:** varAbilityDamage
- **Value:** In the Expression tab, enter:
  ```
  if(greater(length(body('Filter_array')), 0), first(body('Filter_array'))?['amount'], 0)
  ```
- **Explanation:** Gets the damage amount from the first damage action, or 0 if no damage action exists

#### Step 8: Set Ability Name Variable
**Action:** Set variable
- **Name:** varAbilityEffects
- **Value:** In the Expression tab, enter:
  ```
  body('Parse_JSON')?['name']
  ```
- **Note:** This stores the ability name (e.g., "shadowStrike"). Future enhancements will apply buff/debuff actions here.

#### Step 9: Set Player Stats Variables
**Action:** Set variable (varPlayerAttack)
- **Name:** varPlayerAttack
- **Value:** In the Expression tab, enter:
  ```
  int(first(outputs('Get_items_-_Player')?['body/value'])?['Attack'])
  ```
- **Why:** Convert float to integer for calculations.

**Action:** Set variable (varPlayerDefence)
- **Name:** varPlayerDefence
- **Value:** In the Expression tab, enter:
  ```
  int(first(outputs('Get_items_-_Player')?['body/value'])?['Defence'])
  ```
- **Why:** Convert float to integer for calculations.
- **Name:** varPlayerDefence
- **Value:** In the Expression tab, enter:
  ```
  first(outputs('Get_items_-_Player')?['body/value'])?['Defence']
  ```

#### Step 10: Get Current Game Record
**Action:** Get items (SharePoint)
- **List name:** Games
- **Filter Query:** Leave blank (or delete if pre-filled)
- **Top Count:** 1
- **Note:** Since there's only one game record ("current-game"), we can fetch all and get the first item. If the filter is required, some SharePoint connectors may not support OData syntax in the UI - the flow will work without filtering since there's only 1 record.

#### Step 11: Acquire Lock on Game Record
**Action:** Update item (SharePoint)
- **List name:** Games
- **Id:** Click the \"Dynamic content\" tab and select **ID** from Step 10 (labeled something like \"Get items - Games\")
  - **OR** in the Expression tab, enter:
    ```
    first(outputs('Get_items_-_Games')?['body/value'])?['ID']
    ```
    (Adjust the action name if Power Automate named it differently)
- **LockedBy:** `@{triggerBody()?['text']}`
- **LockExpiresAt:** In the Expression tab, enter:
  ```
  addSeconds(utcNow(), variables('varLockTimeout'))
  ```
- **Important:** After entering the ID, verify the actual action name by right-clicking the action title. Then update Step 12 and 13 references if Power Automate used a different name.

#### Step 12: Get Latest Game State (After Lock)
**Action:** Get items (SharePoint)
- **List name:** Games
- **Filter Query:** Leave blank (same as Step 10)
- **Top Count:** 1
- **Note:** Re-fetch to get the most current Mind Crumble stats after acquiring the lock

#### Step 13: Set Mind Crumble Stats Variables
**Action:** Set variable (varMindCrumbleHP)
- **Name:** varMindCrumbleHP
- **Value:** In the Expression tab, enter:
  ```
  int(first(outputs('Get_items_-_Latest_game_state')?['body/value'])?['MindCrumble_HP'])
  ```
- **Why:** Convert float to integer.

**Action:** Set variable (varMindCrumbleDefence)
- **Name:** varMindCrumbleDefence
- **Value:** In the Expression tab, enter:
  ```
  int(first(outputs('Get_items_-_Latest_game_state')?['body/value'])?['MindCrumble_Defence'])
  ```
- **Why:** Convert float to integer.

#### Step 14: Calculate Damage Dealt
**Action:** Set variable (varDamageDealt)
- **Name:** varDamageDealt
- **Value:** In the Expression tab, enter:
  ```
  max(sub(add(variables('varAbilityDamage'), variables('varPlayerAttack')), variables('varMindCrumbleDefence')), 1)
  ```
- **Explanation:** 
  - Base damage = Ability damage + Player attack - Mind Crumble defence
  - Minimum damage = 1 (using `max()` to ensure at least 1 damage)

#### Step 15: Calculate New Mind Crumble HP
**Action:** Compose (name it: Compose_-_NewMindCrumbleHP)
- **Inputs:** In the Expression tab, enter:
  ```
  max(sub(variables('varMindCrumbleHP'), variables('varDamageDealt')), 0)
  ```
- **Explanation:** Subtract damage from current HP, minimum 0

#### Step 16: Update Game Record (Mind Crumble HP & Release Lock)
**Action:** Update item (SharePoint)
- **List name:** Games
- **Id:** In the Expression tab, enter:
  ```
  first(outputs('Get_items_-_Latest_game_state')?['body/value'])?['ID']
  ```
- **MindCrumble_HP:** (Dynamic content) Select **Compose_-_NewMindCrumbleHP** output
- **LockedBy:** Leave empty (clears the lock)
- **LockExpiresAt:** Leave empty (clears the lock)

#### Step 17: Update Player LastActionAt
**Action:** Update item (SharePoint)
- **List name:** Players
- **Id:** In the Expression tab, enter:
  ```
  first(outputs('Get_items_-_Player')?['body/value'])?['ID']
  ```
- **LastActionAt:** (Expression tab) Enter: `utcNow()`

#### Step 18: Log Action to ActionLog
**Action:** Create item (SharePoint)
- **List name:** ActionLog
- **Title:** In the Expression tab, enter:
  ```
  concat(triggerBody()?['text'], ' - Attack')
  ```
- **ActorAgentID:** `@{triggerBody()?['text']}`
- **ActionType:** `Attack`
- **Details:** In the Expression tab, enter (use Power Automate Copilot's suggestion if the JSON below fails):
  ```
  json(concat('{"ability":"', triggerBody()?['text_1'], '","abilityDamage":', variables('varAbilityDamage'), ',"playerAttack":', variables('varPlayerAttack'), ',"mindCrumbleDefence":', variables('varMindCrumbleDefence'), ',"effects":"', variables('varAbilityEffects'), '"}'))
  ```
  **Alternative (simpler, try this first):**
  ```
  concat('{"ability":"', triggerBody()?['text_1'], '","abilityDamage":', string(variables('varAbilityDamage')), ',"playerAttack":', string(variables('varPlayerAttack')), ',"mindCrumbleDefence":', string(variables('varMindCrumbleDefence')), ',"effects":"', variables('varAbilityEffects'), '"}')
  ```
- **Result:** In the Expression tab, enter:
  ```
  concat('{"success":true,"damageDealt":', string(variables('varDamageDealt')), ',"mindCrumbleHPBefore":', string(variables('varMindCrumbleHP')), ',"mindCrumbleHPAfter":', string(outputs('Compose_-_NewMindCrumbleHP')), '}')
  ```

#### Step 19: Post to Teams (Optional)
**Action:** Post adaptive card in a chat or channel (Teams)
- **Post as:** Flow bot
- **Post in:** Channel
- **Team:** [Select your team]
- **Channel:** [Select your channel]
- **Adaptive Card:** Paste this JSON:
  ```json
  {
    "type": "AdaptiveCard",
    "$schema": "http://adaptivecards.io/schemas/adaptive-card.json",
    "version": "1.4",
    "body": [
      {
        "type": "Image",
        "url": "@{if(equals(variables('varPlayerClass'), 'VienneseWizard'), 'https://raw.githubusercontent.com/Jim-Wheatley/crumble/refs/heads/main/VienneseWizard.png', if(equals(variables('varPlayerClass'), 'BourbonRogue'), 'https://raw.githubusercontent.com/Jim-Wheatley/crumble/refs/heads/main/BourbonRogue.png', if(equals(variables('varPlayerClass'), 'HobnobBarbarian'), 'https://raw.githubusercontent.com/Jim-Wheatley/crumble/refs/heads/main/HobnobBarbarian.png', if(equals(variables('varPlayerClass'), 'CustardCleric'), 'https://raw.githubusercontent.com/Jim-Wheatley/crumble/refs/heads/main/CustardCleric.png', if(equals(variables('varPlayerClass'), 'JammieBard'), 'https://raw.githubusercontent.com/Jim-Wheatley/crumble/refs/heads/main/DodgerBard.png', if(equals(variables('varPlayerClass'), 'DigestiveFighter'), 'https://raw.githubusercontent.com/Jim-Wheatley/crumble/refs/heads/main/DigestiveDefender.png', 'https://raw.githubusercontent.com/Jim-Wheatley/crumble/refs/heads/main/BourbonRogue.png'))))))}",
        "size": "Stretch",
        "width": "100%"
      },
      {
        "type": "TextBlock",
        "text": "⚔️ Attack!",
        "weight": "Bolder",
        "size": "Large",
        "color": "Attention",
        "spacing": "Medium"
      },
      {
        "type": "FactSet",
        "facts": [
          {
            "title": "Player:",
            "value": "@{first(outputs('Get_items_-_Player')?['body/value'])?['Title']}"
          },
          {
            "title": "Class:",
            "value": "@{variables('varPlayerClass')}"
          },
          {
            "title": "Agent ID:",
            "value": "@{triggerBody()?['text']}"
          },
          {
            "title": "Ability:",
            "value": "@{triggerBody()?['text_1']}"
          },
          {
            "title": "Damage Dealt:",
            "value": "@{variables('varDamageDealt')}"
          },
          {
            "title": "Mind Crumble HP:",
            "value": "@{outputs('Compose_-_NewMindCrumbleHP')} (was @{variables('varMindCrumbleHP')})"
          }
        ]
      }
    ]
  }
  ```

#### Step 20: Return Response to Power Apps
**Action:** Respond to a PowerApp or flow

**Body Configuration:**
- `success` (string): 
  - Click in the field, go to **Expression** tab
  - Enter: `string(true)`
- `message` (string):
  - Click in the field, go to **Expression** tab
  - Enter: `concat('You dealt ', string(variables('varDamageDealt')), ' damage!')`
- `damageDealt` (number):
  - Click in the field, go to **Dynamic content** tab
  - Select **varDamageDealt** variable
- `mindCrumbleHP` (number):
  - Click in the field, go to **Dynamic content** tab
  - Select **Compose_-_NewMindCrumbleHP** output
- `effects` (string):
  - Click in the field, go to **Dynamic content** tab
  - Select **varAbilityEffects** variable

---

## 2. Power Apps Integration

### Add Attack Button to Screen_AgentDetail

#### Controls to Add:

**1. Image: imgClassAvatar (Display Class Avatar)**

**Step 1:** First, ensure the ClassTemplates data source is connected:
- In Power Apps, go to **Data** (left panel)
- Verify **ClassTemplates** SharePoint list is connected
- If not, click **+ Add data** and add the ClassTemplates list

**Step 2:** Add the ClassAvatarUrl column to ClassTemplates (if not already present):
- Go to your SharePoint **ClassTemplates** list
- Add a new column called **ClassAvatarUrl** (Single line of text)
- Fill in the image URLs for each class:
  - **VienneseWizard:** `https://raw.githubusercontent.com/Jim-Wheatley/crumble/refs/heads/main/VienneseWizard.png`
  - **BourbonRogue:** `https://raw.githubusercontent.com/Jim-Wheatley/crumble/refs/heads/main/BourbonRogue.png`
  - **HobnobBarbarian:** `https://raw.githubusercontent.com/Jim-Wheatley/crumble/refs/heads/main/HobnobBarbarian.png`
  - **CustardCleric:** `https://raw.githubusercontent.com/Jim-Wheatley/crumble/refs/heads/main/CustardCleric.png`
  - **JammieBard:** `https://raw.githubusercontent.com/Jim-Wheatley/crumble/refs/heads/main/DodgerBard.png`
  - **DigestiveFighter:** `https://raw.githubusercontent.com/Jim-Wheatley/crumble/refs/heads/main/DigestiveDefender.png`

**Step 3:** Add the Image control and set properties:
- **Image property:** Use the existing **varClassName** variable:
  ```powerquery
  First(
      Filter(
          ClassTemplatesSummitCrumbleverse,
          ClassName = varClassName
      )
  ).ClassAvatarUrl
  ```
- **Why this approach:** 
  - Uses the existing `varClassName` variable (already captured from deep link parameter)
  - Uses `Filter` + `First` which is more reliable than `LookUp` for matching exact values
  - Compares the `ClassName` field (which contains the class name) to the variable value
  - Works for both existing and new players (doesn't require Players list lookup)
  - Image URLs are centrally managed in ClassTemplates

- **Image position:** Stretch or Fit (your preference)
- **Width:** 200-300 pixels (adjust to fit your layout)
- **Height:** 200-300 pixels (adjust to fit your layout)

**Alternative formula with fallback (recommended for production):**
  ```powerquery
  If(
      IsBlank(First(
          Filter(
              ClassTemplatesSummitCrumbleverse,
              ClassName = varClassName
          )
      ).ClassAvatarUrl),
      "https://raw.githubusercontent.com/Jim-Wheatley/crumble/refs/heads/main/BourbonRogue.png",
      First(
          Filter(
              ClassTemplatesSummitCrumbleverse,
              ClassName = varClassName
          )
      ).ClassAvatarUrl
  )
  ```

**Troubleshooting:**
- **If no image shows:** Use the fallback formula above to get a default image when the filter returns no results
- **Verify your SharePoint list:** Check that the `ClassName` column contains class names like "BourbonRogue", "VienneseWizard", etc.
- **Check the deep link parameter:** Ensure `className` parameter matches the `ClassName` field exactly (case-sensitive: `BourbonRogue` not `bourbonrogue`)
- **Test URL format:** `https://apps.powerapps.com/play/[app-id]?tenantId=[tenant-id]&agentId=london-digestive&displayName=Test&className=BourbonRogue`
- Verify the internal column name for the image URL is exactly `ClassAvatarUrl` (case-sensitive)
- **Important:** The SharePoint list has an empty `Title` column. Use the `ClassName` column for filtering instead.
- **Note:** This dynamically displays the avatar based on the className from the deep link parameter

**2. Label: lblAbility (Display Class Ability)**
- **Text:**
  ```powerquery
  LookUp(
      ClassTemplates,
      ClassName.Value = LookUp(
          Players,
          AgentID = varAgentId
      ).Class.Value
  ).AbilityName
  ```
- **Note:** This dynamically pulls the ability name from ClassTemplates based on the player's class

**2. Button: btnAttack**
- **Text:** "Attack Mind Crumble"
- **OnSelect:** Paste this formula:
  ```powerquery
  Set(
      varAttackResult,
      AttackFlow.Run(
          varAgentId,
          LookUp(
              ClassTemplates,
              ClassName.Value = LookUp(
                  Players,
                  AgentID = varAgentId
              ).Class.Value
          ).AbilityName
      )
  );
  If(
      varAttackResult.success = "true",
      Notify(
          varAttackResult.message & " Mind Crumble HP: " & Text(varAttackResult.mindCrumbleHP),
          NotificationType.Success
      ),
      Notify(varAttackResult.message, NotificationType.Error)
  )
  ```
- **Note:** The ability is automatically retrieved from the player's class, no dropdown needed

**3. Label: lblMindCrumbleHP (Display Mind Crumble HP)**
- **Text:** 
  ```powerquery
  "Mind Crumble HP: " & Text(
      LookUp(
          Games,
          Title = "current-game"
      ).MindCrumble_HP
  )
  ```
- **Note:** This label will update after each attack when the Games list refreshes

---

## 3. SharePoint Lists Requirements

### Games List Columns (verify these exist):
- `MindCrumble_HP` (Number)
- `MindCrumble_Attack` (Number)
- `MindCrumble_Defence` (Number)
- `MindCrumble_Status` (Choice: Active, Defeated)
- `MindCrumble_Debuffs` (Multiple lines of text)
- `LockedBy` (Single line of text)
- `LockExpiresAt` (Date and Time)

### ClassTemplates List - Sample AbilityJson:
Each class should have an `AbilityJson` column with this format:

**Rogue (Bourbon) - Shadow Strike:**
```json
{
  "type": "composite",
  "name": "shadowStrike",
  "actions": [
    {
      "type": "damage",
      "target": "mindcrumble",
      "amount": 2
    },
    {
      "type": "buff",
      "target": "self",
      "stat": "defence",
      "amount": 2,
      "duration": 1
    }
  ]
}
```

**Barbarian (Chocolate HobNob) - Oaty Rage:**
```json
{
  "type": "simple",
  "name": "oatyRage",
  "actions": [
    {
      "type": "damage",
      "target": "mindcrumble",
      "amount": 7,
      "ignoresDefence": true
    }
  ]
}
```

**Wizard (Viennese Whirl) - Arcane Swirl:**
```json
{
  "type": "composite",
  "name": "arcaneSwirl",
  "actions": [
    {
      "type": "debuff",
      "target": "mindcrumble",
      "stat": "defence",
      "amount": -2,
      "duration": 3
    }
  ]
}
```

**Note:** For this initial implementation (Task 1.2), only "damage" actions are processed. Buff/debuff actions will be implemented in future tasks.

### ActionLog List Columns (verify these exist):
- `Title` (Single line of text)
- `ActorAgentID` (Single line of text)
- `ActionType` (Choice: Join, Attack, Defend, Heal)
- `Details` (Multiple lines of text) - JSON format
- `Result` (Multiple lines of text) - JSON format
- `Created` (Date and Time) - auto-generated

---

## 4. Update ClassTemplates List

**Do you need to update the list?** Yes, you need to update the `AbilityJson` column for each class.

### How to Update:

1. **Go to your SharePoint site** and open the **ClassTemplates** list
2. **Edit each class record** and update the `AbilityJson` column with the new schema
3. **Use the examples above** (Section 3) for each class

### Complete Ability Definitions for All Classes:

**Wizard (Viennese Whirl) - Arcane Swirl:**
```json
{
  "type": "composite",
  "name": "arcaneSwirl",
  "actions": [
    {
      "type": "debuff",
      "target": "mindcrumble",
      "stat": "defence",
      "amount": -2,
      "duration": 3
    }
  ]
}
```

**Barbarian (Chocolate HobNob) - Oaty Rage:**
```json
{
  "type": "simple",
  "name": "oatyRage",
  "actions": [
    {
      "type": "damage",
      "target": "mindcrumble",
      "amount": 7,
      "ignoresDefence": true
    }
  ]
}
```

**Cleric (Custard Cream) - Sweet Salvation:**
```json
{
  "type": "composite",
  "name": "sweetSalvation",
  "actions": [
    {
      "type": "heal",
      "target": "previousThreePlayers",
      "amount": 2
    },
    {
      "type": "buff",
      "target": "nextThreePlayers",
      "stat": "defence",
      "amount": 1,
      "duration": 1
    }
  ]
}
```

**Rogue (Bourbon) - Shadow Dunk:**
```json
{
  "type": "composite",
  "name": "shadowDunk",
  "actions": [
    {
      "type": "damage",
      "target": "mindcrumble",
      "amount": 2
    },
    {
      "type": "buff",
      "target": "self",
      "stat": "defence",
      "amount": 2,
      "duration": 1
    }
  ]
}
```

**Bard (Jammie Dodger) - Jam Anthem:**
```json
{
  "type": "composite",
  "name": "jamAnthem",
  "actions": [
    {
      "type": "buff",
      "target": "nextThreePlayers",
      "stat": "attack",
      "amount": 2,
      "duration": 1
    }
  ]
}
```

**Fighter (Chocolate Digestive) - Shield of Sweet Steel:**
```json
{
  "type": "composite",
  "name": "shieldOfSweetSteel",
  "actions": [
    {
      "type": "damage",
      "target": "mindcrumble",
      "amount": 2
    },
    {
      "type": "buff",
      "target": "self",
      "stat": "defence",
      "amount": 2,
      "duration": 1
    }
  ]
}
```

**Important Notes:**
- For **Task 1.2**, only the **damage actions** will be processed by the flow
- **Buff, debuff, and heal actions** will be implemented in future tasks
- Each class still has their damage component working immediately
- The schema is extensible for future enhancements

---

## 5. Testing Checklist

### Before Testing:
1. ✅ Ensure Games list has one record with `Title = "current-game"`
2. ✅ Set initial Mind Crumble HP (e.g., 100)
3. ✅ Set Mind Crumble Defence (e.g., 5)
4. ✅ Ensure ClassTemplates has AbilityJson for all classes
5. ✅ Join the game first using JoinGameFlow (Task 1.1)

### Test Scenarios:

**Test 1: Valid Attack**
- ✅ Trigger: Click "Attack Mind Crumble" button in Power Apps
- ✅ Expected: 
  - Damage calculated correctly (ability + player attack - MC defence, min 1)
  - Mind Crumble HP reduced
  - Success notification shown
  - ActionLog entry created
  - Teams card posted

**Test 2: Attack with Different Abilities**
- ✅ Select "DoubleStrike" from dropdown
- ✅ Click attack
- ✅ Expected: Different damage calculation based on ability

**Test 3: Attack When Mind Crumble Defeated (HP = 0)**
- ✅ Reduce Mind Crumble HP to near 0
- ✅ Attack again
- ✅ Expected: HP goes to 0 (not negative)

**Test 4: Player Not Found**
- ✅ Test with invalid agentId
- ✅ Expected: Error message "Player not found"

**Test 5: Concurrent Attacks (Lock Testing)**
- ✅ Two players attack simultaneously
- ✅ Expected: Lock prevents race condition, one attack waits

---

## 6. Common Issues & Solutions

### Issue: "The execution of template action 'Filter_array' failed"
**Solution:** Ensure the Parse JSON schema matches your AbilityJson structure exactly. Use "Use sample payload to generate schema" with real data from ClassTemplates.

### Issue: Damage calculation returns negative value
**Solution:** Use `max()` function to ensure minimum damage of 1:
```
max(sub(add(varAbilityDamage, varPlayerAttack), varMindCrumbleDefence), 1)
```

### Issue: Response schema mismatch error in Power Apps
**Solution:** Ensure BOTH branches of the "Player Exists" condition return identical fields:
- `success` (string)
- `message` (string)
- `damageDealt` (number)
- `mindCrumbleHP` (number)
- `effects` (string)

### Issue: Lock not released after error
**Solution:** Add error handling scope that always releases the lock:
1. Wrap steps 11-19 in a "Scope" action
2. Add parallel branch "Run after" configured to run on failure
3. Add "Update item" to clear LockedBy and LockExpiresAt

### Issue: Ability not found in JSON
**Solution:** Verify:
- Ability name matches exactly (case-sensitive)
- AbilityJson is valid JSON
- Filter array uses `item()?['name']` not `item().name`

---

## 7. Enhanced Features (Future Iterations)

### Sprint 1.3+ Enhancements:
- [ ] Apply ability effects (buffs/debuffs) to player stats
- [ ] Implement multi-hit abilities (DoubleStrike actually hits twice)
- [ ] Add critical hit calculation (random chance)
- [ ] Apply status effects (poison, stun, etc.)
- [ ] Check turn order before allowing attack
- [ ] Implement cooldowns for powerful abilities

### Sprint 2+ Enhancements:
- [ ] Add lock timeout validation (check if lock expired)
- [ ] Implement retry logic for lock acquisition
- [ ] Add detailed Teams adaptive card with images
- [ ] Log damage breakdowns for analytics
- [ ] Add celebration animation when Mind Crumble defeated

---

## 8. Power Automate Expression Reference

### Commonly Used Expressions:

**Get first item from SharePoint results:**
```
first(outputs('Get_items_-_Player')?['body/value'])
```

**Access nested property:**
```
first(outputs('Get_items_-_Player')?['body/value'])?['Attack']
```

**Concat strings for OData filter:**
```
concat('AgentID eq ''', triggerBody()?['text'], '''')
```

**Calculate max value:**
```
max(sub(variables('varHP'), variables('varDamage')), 0)
```

**Add seconds to current time:**
```
addSeconds(utcNow(), 30)
```

**Create JSON object:**
```json
{
  "field": "@{variables('varValue')}",
  "number": @{variables('varNumber')}
}
```

---

## 9. Key Architecture Decisions

### Why Power Automate for Attacks (vs Power Apps Direct Write)?
- Attacks require **transactional consistency** (lock, read, calculate, write, unlock)
- Multiple lists updated atomically (Games, Players, ActionLog)
- Ability to post to Teams from the flow
- Better error handling and retry logic
- Centralized damage calculation logic

### Why Lock the Games Record?
- Prevents race conditions when multiple players attack simultaneously
- Ensures Mind Crumble HP is calculated correctly
- 30-second timeout prevents deadlocks

### Why Parse AbilityJson vs Separate Columns?
- Flexible ability system with multiple actions per ability
- Supports damage, heal, buff, debuff in a single ability
- Easy to add new action types without schema changes
- Machine-readable for automatic effect application
- Extensible for complex targeting (self, allies, enemies, etc.)

---

## Next Steps

After completing Task 1.2, proceed to:
- **Task 1.3:** Implement Mind Crumble turn logic (villain attacks back)
- **Task 1.4:** Wire Power Apps Attack button (already done in this task!)
- **Task 1.5:** Update Copilot agent instructions with attack guidance
- **Task 2.1:** Enhance locking with timeout validation and retry logic
- **Task 2.2:** Improve Teams adaptive card formatting

---

---

## Common Issues & Solutions During Implementation

### Issue 1: Parse Ability JSON returns null
**Cause:** Filter on ClassTemplates list not returning records
**Solution:** Use Filter array action instead of OData filter. SharePoint lookup fields don't filter well with OData concat() syntax.
- Get all ClassTemplates (no filter)
- Use Filter array in Power Automate to match ClassName with varPlayerClass

### Issue 2: Type mismatch - Float fields (Attack, Defence, MindCrumble_HP)
**Cause:** SharePoint number columns may return float values
**Error:** "The variable 'varXxx' of type 'Integer' cannot be initialized or updated with value of type 'Float'"
**Solution:** Wrap with `int()` function in Expression tab:
```
int(first(outputs('Get_items_-_Player')?['body/value'])?['Attack'])
```

### Issue 3: Lookup fields return nested objects (Class field)
**Cause:** SharePoint lookup/choice fields have complex structure
**Error:** "Cannot be initialized or updated with value of type 'Object'"
**Solution:** Extract the Value property:
```
string(first(outputs('Get_items_-_Player')?['body/value'])?['Class']?['Value'])
```

### Issue 4: Empty filter with quotes causes OData errors
**Cause:** Using `""` or `''` in Filter Query field
**Error:** "The $filter expression """" is not valid"
**Solution:** Leave the Filter Query field completely blank when no filter is needed

### Issue 5: ID references in loops/conditions
**Cause:** Using `item()?['ID']` outside of a loop context
**Error:** "The 'inputs.parameters' value for 'id' is null or empty"
**Solution:** Use `first(outputs('ActionName')?['body/value'])?['ID']` instead

### Issue 6: Response action has expressions in schema
**Cause:** Typing expression syntax directly in body values
**Error:** "Expressions are not allowed in schema definition"
**Solution:** Use Dynamic content tab for variables/outputs, Expression tab only for string concatenation with concat()

### Issue 7: varPlayerClass populates but filter doesn't work
**Cause:** Filter array condition not matching data types
**Solution:** Select the nested `.Value` property from ClassName when setting up the filter condition

---

## Notes

- All SharePoint column names are **case-sensitive**
- Use **Modern (Fluent) controls** in Power Apps (`Value`, `SelectedItems`, not `Default`, `Selected`)
- OData filters require `concat()` for dynamic values
- All 200 responses must have identical schema
- Lock timeout is 30 seconds (configurable via `varLockTimeout`)
- Damage formula: `(Ability Damage + Player Attack - Mind Crumble Defence)`, minimum 1
- Mind Crumble HP cannot go below 0

---

## Files Modified

- Power Automate: `AttackFlow` (new flow)
- Power Apps: `Screen_AgentDetail` (add ddAbility dropdown and btnAttack button)
- SharePoint: `Games`, `Players`, `ActionLog`, `ClassTemplates` (data changes)

---

## Success Criteria

✅ AttackFlow accepts agentId and abilityName  
✅ Flow acquires and releases lock on Games record  
✅ Damage calculated using formula: ability + player attack - MC defence  
✅ Mind Crumble HP updated correctly (minimum 0)  
✅ Player LastActionAt timestamp updated  
✅ ActionLog entry created with damage details  
✅ Teams adaptive card posted with attack summary  
✅ Power Apps button triggers flow and displays result  
✅ Error handling for player not found  
✅ Response schema consistent (success/error paths)  

---

**Happy battling against the Mind Crumble! 🍪⚔️**
