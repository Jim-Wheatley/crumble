# Task 0.4 — Power Automate Flow: Update SharePoint + Post to Teams

- Flow name: `UpdateARecordAndPostToTeams`
- Purpose: From Power Apps, update a SharePoint list item (e.g., HP) and post a message to Teams, then respond back to Power Apps.
- Scope: Connected to Power App from Task 0.3 and lists from Task 0.2.

## Flow Design
- Type: `Instant cloud flow`
- Trigger: `PowerApps (V2)`
- Inputs added on trigger:
  - `Text` titled `recordId` → auto variable: `triggerBody()['text']`
  - `Text` titled `newValue` → auto variable: `triggerBody()['text_1']`

## Steps
1. PowerApps (V2) — Trigger
   - Accepts `recordId` (numeric SharePoint ID) and `newValue` (e.g., new HP).

2. Get item — SharePoint
   - Site Address: Select the correct SharePoint site
   - List Name: Select target list (e.g., `Players`)
   - Id: `int(triggerBody()['text'])`

3. Update item — SharePoint
   - Site Address: same as above
   - List Name: same as above
   - Id: `int(triggerBody()['text'])`
   - Field to update (example):
     - `HP`: `triggerBody()['text_1']`
     - Alternatively, calculate: `sub(outputs('Get_item')?['body/HP'], 10)`
   - Ensure any required columns are populated (reuse values from `Get item` for untouched fields).

4. Post message in a chat or channel — Teams
   - Team: select your Team
   - Channel: select target Channel (e.g., `General`)
   - Message examples:
     - Basic: `Player HP updated to @{outputs('Update_item')?['body/HP']}`
     - With IDs: `Record @{triggerBody()['text']} HP set to @{triggerBody()['text_1']}`

5. Respond to PowerApps — Return confirmation
   - Output `Text`
   - Name: `result`
   - Value: `concat('Updated HP to ', outputs('Update_item')?['body/HP'])`

## Power Apps Wiring (Task 0.3)
- Add flow to app: Power Automate panel → Add flow → `UpdateARecordAndPostToTeams`
- Example test button on `Screen_AgentDetails`:
  - OnSelect:
    ```
    Set(rec, LookUp(Players, AgentId = varAgentId));
    Set(flowResult, UpdateARecordAndPostToTeams.Run(Text(rec.ID), Text(rec.HP - 10)));
    Notify(flowResult.result, NotificationType.Success);
    ```
- Simple hard-coded test:
  ```
  Set(flowResult, UpdateARecordAndPostToTeams.Run("5", "50"));
  Notify(flowResult.result, NotificationType.Success);
  ```

## Testing Checklist
- Use a valid numeric SharePoint `ID` (not the custom `AgentId`).
- Confirm Teams message posts with dynamic content.
- Verify the SharePoint item’s `HP` (or target field) changes.
- See a confirmation toast in Power Apps from the flow response.

## Troubleshooting Notes (Observed)
- No dynamic content in Id field: Save the flow, then edit; or use `int(triggerBody()['text'])` in expression tab.
- InvalidOpenApi… integer required: Do not pass `@triggerBody()`; pass the specific field: `int(triggerBody()['text'])`.
- Item Not Found: Ensure you pass an existing numeric `ID` (create a test item if needed).
- `recordId` vs auto variable names: Power Automate uses `text`/`text_1` for the first/second inputs unless you bind differently.
- Response warning: Use dynamic content in `Respond to PowerApps` (e.g., outputs from `Update item`) to avoid the warning.

## Improvements (Next)
- Error handling: Add scopes or `Configure run after` to return friendly errors.
- Adaptive card: Replace plain message with an Adaptive Card (Sprint 2.2).
- Concurrency: Introduce lock fields or ETag handling where needed.
