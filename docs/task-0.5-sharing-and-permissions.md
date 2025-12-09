# Task 0.5 — Sharing Power Apps and Power Automate Flows

**Status:** 📝 Documentation  
**Date Created:** 8 December 2025

## Problem

Users getting permission errors when trying to use the Power App and run flows:
```
JoinGameFlow.Run failed: {"error":{"code":"Forbidden","message":"The user with object id '...' does not have permission to access this."}}
```

## Solution Overview

To allow users to run your Power App and its associated flows without permission issues, you need to:

1. **Share the Power Automate flow** with users or configure it to run under embedded connections
2. **Share the Power App** with users
3. **Share the SharePoint lists** with appropriate permissions
4. **Configure flow connections properly**

## Step-by-Step Fix

### Option A: Share the Flow with Everyone (Recommended for Many Users)

**1. Share the Power Automate Flow with Your Organization**

For each flow (`JoinGameFlow`, `UpdateARecordAndPostToTeams`, etc.):

1. Go to [Power Automate](https://make.powerautomate.com)
2. Find your flow in **My flows**
3. Click the **three dots (...)** next to the flow → **Share**
4. In the **Add people** field, type your organization name or domain
   - Look for options like "Everyone at [YourCompany]" or your organization's group
   - Alternatively, add a security group that contains all users
5. Set permissions:
   - ✅ **User** (can run the flow) — this is what you want
   - ⚠️ Avoid **Co-owner** unless users need to edit the flow
6. Click **Share**
7. Repeat for all flows

**Important:** With this approach, users will still need to authorize their own connections to SharePoint/Teams when they first use the app. See Option B to avoid this.

---

### Option B: Use "Run-only users" Connections (Avoids User Connection Setup)

This approach allows the flow to run using **your connections** instead of requiring each user to set up their own. Users won't see connection authorization prompts.

**1. Configure Each Flow Action to Use Run-only Connections**

1. Go to [Power Automate](https://make.powerautomate.com)
2. Edit your flow (e.g., `JoinGameFlow`)
3. For **each action** that uses a connector (SharePoint, Teams, etc.):
   - Click the **⋮** (three dots) on the action card
   - Select **Settings**
   - Expand the **Connection** dropdown
   - Look for and select: **"Use this connection ([your name])"** or **"Provided by run-only user"**
   - If you don't see these options, the connector may not support it (proceed to step 4)
4. Click **Save** at the top of the flow
5. **Share the flow** (see Option A above) - this is still required!

**What this does:**
- Flow runs with YOUR SharePoint/Teams connections
- Users don't need to authorize connections themselves
- Users only need "User" permission on the flow (not their own connections)

**Limitations:**
- Not all connectors support this (some Premium connectors don't)
- The flow will always run as YOU in SharePoint/Teams (audit trails show your name)
- If you leave the organization, flows will stop working

**Note:** This setting is in the **flow editor**, not in Power Apps. When you add the flow to Power Apps, you just select it - there are no additional connection options in the Power Apps interface.

---

### 2. Share the Power App

**In Power Apps Studio:**

1. Open your app → Click **Share** (top right)
2. Add users or groups:
   - Enter email addresses or group names
   - Option: Share with "Everyone in [Organization]"
3. Permissions:
   - ✅ **User** (can use the app)
   - ⚠️ **Co-owner** (can edit the app) — only if needed
4. Optional: Check **Send an email invitation**
5. Click **Share**

**Important Settings:**
- Under **Data permissions**, ensure users have access to:
  - SharePoint lists (Players, Games, etc.)
  - Any other data sources used

---

### 3. SharePoint List Permissions

Users need at least **Edit** permissions on the SharePoint lists:

**For each list (`Players`, `Games`, `GameTurns`):**

1. Go to SharePoint site
2. Navigate to the list
3. Click **⚙️ Settings** → **List settings**
4. Under **Permissions and Management** → **Permissions for this list**
5. Click **Grant Permissions**
6. Add users/groups
7. Set permission level:
   - **Edit** (can add/update items) — recommended
   - **Contribute** (can add/update/delete items)
   - **Read** (view only) — not sufficient for your flows
8. Uncheck **Send an email invitation** if desired
9. Click **Share**

---

### 4. Testing Checklist

After configuring sharing:

- [ ] Flow owner can run the app successfully
- [ ] Test user can open the app via shared link
- [ ] Test user can trigger flows without permission errors
- [ ] SharePoint list items update correctly
- [ ] Teams messages post successfully
- [ ] App displays confirmation/success messages

---

## Recommended Configuration for Your Use Case

Given that this is a game app where **many users** need to interact:

### **Best Practice Setup (For Many Users):**

1. **Flows:** 
   - Share with "Everyone at [Organization]" or a broad security group
   - Configure actions to use "run-only user" connections (Option B)
2. **Power App:** 
   - Share with "Everyone at [Organization]" or the same broad group
   - Permission: "User" (can use)
3. **SharePoint Lists:** 
   - Grant "Edit" permissions to "Everyone" or the same group
   - Or use existing site permissions if appropriate
4. **Teams:** 
   - Ensure the channel is accessible to all players

### **Security Note:**

This configuration allows all users to:
- Update SharePoint list items via the app/flows
- Post to the Teams channel
- View all game data

If you need more granular control:
- Use PowerFx formulas to filter data by user
- Add conditional logic in flows to validate user permissions
- Consider using Row-level security in SharePoint (advanced)

---

## Common Issues and Solutions

### Issue: "Connection not found" errors

**Cause:** User hasn't authorized connections  
**Solution:** 
- First time: User must click "Allow" when prompted
- Subsequent: Use embedded connections (Option B above)

### Issue: Flow works for owner but not other users

**Cause:** Flow not shared or connections not embedded  
**Solution:** Follow Option A or Option B above

### Issue: "User does not have permission to access this"

**Cause:** SharePoint list permissions insufficient  
**Solution:** Grant Edit permissions on all lists used by the flow

### Issue: Teams message posts but shows flow owner's name

**Cause:** Teams connector uses flow owner's connection  
**Solution:** This is expected behavior. To show actual user:
- Add a "Created By" parameter to the flow
- Pass `User().FullName` or `User().Email` from Power Apps
- Include in Teams message: `"Action by: @{triggerBody()['text_2']}"`

---

## Quick Steps Summary

**To allow everyone to use your app without individual setup:**

1. **In Power Automate** - For each flow:
   - Share → Add "Everyone at [Org]" → Permission: User → Share
   - Edit flow → Each action → ⋮ Settings → Connection: Use run-only user → Save

2. **In Power Apps**:
   - Share → Add "Everyone at [Org]" → Share

3. **In SharePoint**:
   - Each list → Settings → Permissions → Add "Everyone" → Edit permission

4. **Test** with a non-owner account

---

## Quick Reference: Sharing Links

After sharing the Power App, you'll get a link like:
```
https://apps.powerapps.com/play/[app-id]?tenantId=[tenant-id]
```

For deep links with parameters:
```
https://apps.powerapps.com/play/[app-id]?tenantId=[tenant-id]&agentId=agent-001&gameId=game-123
```

Users can bookmark these links or access via Teams app integration.

---

## Next Steps

- [ ] Share all flows with target users
- [ ] Share Power App with target users
- [ ] Configure SharePoint list permissions
- [ ] Test with non-owner user account
- [ ] Document any role-specific permissions needed
- [ ] Consider creating a Teams app tab for easier access

---

## Related Documentation

- [Task 0.2 — SharePoint Lists](task-0.2-sharepoint-lists.md)
- [Task 0.3 — Power Apps Deep Link](task-0.3-power-apps-deep-link.md)
- [Task 0.4 — Power Automate Flow](task-0.4-power-automate-flow.md)
