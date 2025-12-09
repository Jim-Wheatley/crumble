# Task 0.3 — Power Apps Canvas App with Deep Link Parsing

**Status:** ✓ Complete  
**Owner:** Front-end  
**Date Completed:** 2 December 2025

## Overview

Created a Power Apps canvas app skeleton that reads `agentId` from URL parameters and displays agent data from SharePoint with fallback to sample placeholder data.

## Acceptance Criteria

✓ App reads `agentId` parameter from deep link  
✓ Shows placeholder/real agent data (Name, Class, HP, Defence)  
✓ Handles missing or invalid parameters gracefully  
✓ Works with SharePoint `Summit - Crumbleverse - Players` list

## Architecture

**Screens:**
- `Screen_Home` — Landing screen with hidden Timer for navigation
- `Screen_AgentDetail` — Displays agent information

**Data Flow:**
1. `App.OnStart` initializes variables and sample agent
2. `Screen_Home.OnVisible` captures params and triggers Timer navigation
3. `tmrNavigate` (Timer) performs Navigate to AgentDetail
4. `Screen_AgentDetail.OnVisible` looks up SharePoint data or falls back to sample

**Key Variables:**
- `varAgentId` — Agent ID from URL param
- `varAgentIdDecoded` — URL-decoded version
- `varSampleAgent` — Fallback sample data
- `varAgentSP` — SharePoint lookup result
- `varAgent` — Final agent record displayed in UI
- `varShouldNavigateToAgent` — Flag for Timer-based navigation

## Implementation Details

### App.OnStart (Initialize variables and sample data)

```
Set(varAgentId, Param("agentId"));
Set(varGameId, Param("gameId"));
Set(varSampleAgent,
  {
    AgentId: "sample-1",
    Title: "Crumb",
    Class: {Value: "Biscuit"},
    HP: 12,
    Defence: 3,
    AvatarUrl: "",
    Abilities: Table({Name:"Nibble",Power:2})
  }
);
Set(varShouldNavigateToAgent, !IsBlank(varAgentId))
```

**Note:** `Class` is a Choice column in SharePoint, requiring `{Value: "..."}` structure.

### Timer-Based Navigation Pattern

Power Apps restricts `Navigate()` in `OnStart` and `OnVisible`. Workaround:

**Timer Control (`tmrNavigate` on `Screen_Home`):**
- `Duration`: `10` (ms)
- `AutoStart`: `false`
- `Repeat`: `false`
- `Start`: `varStartNavTimer`

**`tmrNavigate.OnTimerEnd`:**
```
Set(varStartNavTimer, false);
If(varShouldNavigateToAgent,
   Set(varShouldNavigateToAgent, false);
   Navigate(Screen_AgentDetail, ScreenTransition.None)
)
```

**`Screen_Home.OnVisible`:**
```
If(IsBlank(varAgentId), Set(varAgentId, Param("agentId")));
If(varShouldNavigateToAgent || !IsBlank(varAgentId), Set(varStartNavTimer, true))
```

### SharePoint Lookup with Fallback

**`Screen_AgentDetail.OnVisible`:**
```
/* Capture agentId from param if needed */
If(IsBlank(varAgentId), Set(varAgentId, Param("agentId")));

/* Decode URL-encoded characters (e.g., %20 → space) */
Set(varAgentIdDecoded, If(!IsBlank(varAgentId), Substitute(varAgentId, "%20", " "), varAgentId));

/* Try SharePoint lookup */
Set(varAgentSP, LookUp('Summit - Crumbleverse - Players', AgentID = varAgentIdDecoded));

/* Use SharePoint result if found, otherwise use sample */
Set(varAgent, If(!IsBlank(varAgentSP), varAgentSP, varSampleAgent))
```

### Control Bindings

**Choice Column Handling:**  
SharePoint Choice columns return records with `.Value` property, not plain text.

```
lblTitle.Text = varAgent.Title
lblClass.Text = varAgent.Class.Value  /* Choice column */
lblHP.Text = "HP: " & Text(varAgent.HP)
lblDefence.Text = "Def: " & Text(varAgent.Defence)
```

## SharePoint Integration

**Data Source:** `'Summit - Crumbleverse - Players'`

**Column Mapping:**
- `AgentID` (text) — Unique identifier for lookup
- `Title` (text) — Agent display name
- `Class` (Choice) — Agent class (requires `.Value`)
- `HP` (number) — Hit points
- `Defence` (number) — Defence stat
- `AvatarUrl` (text, optional) — Image URL
- `Abilities` (not implemented) — Future: table of abilities

**Key Learnings:**
- SharePoint's `Title` column is special and always exists; avoid hiding and creating custom `Name` columns
- Choice columns must use `.Value` property in formulas
- `App.OnStart` may not run reliably in all scenarios; reinforce initialization in `Screen.OnVisible`

## Testing

### Test Scenarios

**1. Valid AgentID (SharePoint record exists):**
- Set test ID in `Screen_Home.OnVisible`: `Set(varAgentId, "actual-id-from-sharepoint");`
- Run preview → Should display SharePoint data

**2. Invalid AgentID (no matching record):**
- Set test ID: `Set(varAgentId, "nonexistent-999");`
- Run preview → Should display sample agent (`Crumb`)

**3. No agentId param (blank):**
- Set: `Set(varAgentId, Blank());`
- Run preview → Should stay on Home screen or show sample agent

**4. Deep Link Testing (Published App):**
- Template: `https://apps.powerapps.com/play/{AppId}?tenantId={TenantId}&agentId={AgentID}&gameId={GameID}`
- Test with valid, invalid, and missing `agentId` params

### Debug Tools

**Debug Label (`lblDebug`):**
```
"AgentId: " & varAgentIdDecoded & " | SP: " & If(!IsBlank(varAgentSP), "FOUND", "NOT FOUND") & " | Agent: " & varAgent.Title
```

Shows:
- Agent ID being looked up
- Whether SharePoint returned data
- Current agent name displayed

**Test Gallery (`galTest`):**  
Displays all SharePoint records for verification — remove before production.

## Production Checklist

Before deploying:
- [ ] Remove or hide `lblDebug`
- [ ] Remove or hide `galTest`
- [ ] Remove test ID hardcoding from `Screen_Home.OnVisible`
- [ ] Publish app and test with real deep link URLs
- [ ] Optional: Re-enable error controls (`lblError`, `btnRetry`)
- [ ] Share app link template with content team

## Known Issues & Future Enhancements

**Known Issues:**
- `App.OnStart` doesn't always run on app resume; mitigated by param checks in `Screen.OnVisible`
- Preview mode requires multiple refreshes for variable updates to take effect

**Future Enhancements (Pass 3+):**
- Error UI with retry button (controls exist but disabled)
- Loading spinner while fetching SharePoint data
- Refresh button to reload agent data
- Support for `Abilities` table display
- Telemetry logging to `ActionLog` SharePoint list
- Handle network errors and permissions issues gracefully
- Support for additional params (`gameId` captured but not used yet)

## Files Modified

- Power Apps Canvas App: `AgentsOfCrumble_Skeleton_v1` (not in repo)
- Documentation: `/docs/task-0.3-power-apps-deep-link.md`
- Original spec: `/specs/001-agents-of-crumble/README.md` (Pass 1 notes)

## Related Tasks

- **Task 0.2** — Provisioned SharePoint lists (dependency)
- **Task 1.4** — Wire Attack button to AttackFlow (next)
- **Task 1.5** — Update agent instructions with deep link template (handoff)

## Handoff Notes

**For Content Team (Task 1.5):**  
Deep link template to include in agent instructions:
```
https://apps.powerapps.com/play/{AppId}?tenantId={TenantId}&agentId={AgentID}&gameId={GameID}
```

Replace `{AppId}` and `{TenantId}` with actual values from published app.

**For Infra Team:**  
Ensure SharePoint list `Summit - Crumbleverse - Players` has:
- `AgentID` column (text, required)
- `Title` column (default SharePoint text column)
- `Class` column (Choice type)
- Appropriate permissions for app service account

---

**Task Completed:** 2 December 2025  
**Duration:** ~4 hours (including troubleshooting Choice columns and navigation restrictions)
