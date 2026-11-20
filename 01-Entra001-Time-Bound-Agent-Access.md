# 01 - Entra001 - Time-bound agent access

This use case shows that an agent can be given access the same governed way a person is: through an **access package**, with a recorded reason, an approver and an end date. When the end date passes, the access is removed automatically and the agent itself keeps working.

It has two scenarios:

- **Scenario 1 - Group membership for the Copilot Studio agent.** An administrator assigns an access package to **AGT01-CPS**, and the agent becomes a member of a security group.
- **Scenario 2 - Time-limited Microsoft Graph access for the autonomous agent.** Allan, as the agent's sponsor, requests access for **Policy-Digest-Agent**; Lee approves. You see the agent's Microsoft Graph request **fail before**, **work during** and **fail again after** the access period. Adding API permissions to access packages is in **preview**.

## Section 1 - Why it matters

**Contoso's concern:** agents are being given access to data and APIs, often permanently and with no record of who asked for it or why. Contoso wants agent access to be requested by an accountable person, approved, time-limited and removed automatically.

**What Contoso can already do:** with Microsoft Entra ID P2 or Microsoft Entra ID Governance, Contoso can already use access packages, approvals and expiry for **people**. For apps and agents, an administrator usually grants API permissions directly through admin consent, and those permissions stay until someone removes them.

**What Agent 365 adds:** the same access packages work for **agent identities**:

- an agent identity can be the beneficiary of an access package;
- the agent's **sponsor** can request access on the agent's behalf, and an approver decides;
- the package can deliver group membership, allowed directory roles or **API permissions** (preview);
- the access ends automatically on the end date, and the sponsor is notified as the end date approaches.

**Licensing:** Microsoft 365 E7, or Microsoft Agent 365 with at least Microsoft Entra ID P1 or Microsoft 365 E3 ([access packages for agents](https://learn.microsoft.com/en-us/entra/agent-id/agent-access-packages), [licensing](https://learn.microsoft.com/en-us/entra/id-governance/licensing-fundamentals)).

| This use case shows, when observed | It does not show |
|---|---|
| An agent identity can be the beneficiary of an access package | That tokens already issued are revoked when access ends. A token issued during the access period stays valid until it expires, usually about an hour. |
| The package delivers group membership to the agent (Scenario 1) | That the group in Scenario 1 changes what the agent can do. The group grants nothing; it shows delivery only. |
| A sponsor request and an independent approval deliver an API permission, and the agent's real Graph request succeeds only during the access period (Scenario 2) | That Copilot Studio chat changes. In chat, a Copilot Studio agent reads SharePoint as the signed-in user, not as itself. |
| Access is removed automatically at the end date, without disabling the agent | That administrators cannot grant access another way. Administrators can still assign packages or grant permissions directly. |

## Section 2 - People and assets

| Name | Role in this use case | Microsoft access needed |
|---|---|---|
| **Alex** | Creates both access packages; assigns the package in Scenario 1; checks delivery and expiry | Identity Governance Administrator (or catalog owner plus access package manager for the catalog) |
| **The tenant's Global Administrator** | Adds the Microsoft Graph permission to the Scenario 2 package (one step) | Global Administrator. Microsoft requires it to add API permissions to an access package. |
| **Allan** | Sponsor of Policy-Digest-Agent; requests access for it in Scenario 2 | None beyond Setup. He is already the agent's sponsor ([Setup 4.4](00-Setup.md#44-create-the-agent-identity-from-the-blueprint)). |
| **Lee** | Approves Allan's request in Scenario 2 | None. He is named as the approver in the package. His account must be different from Allan's. |
| **Nestor** | Runs the agent's Graph check before, during and after the access period | Access to the blueprint secret ([Setup 4.3](00-Setup.md#43-add-a-temporary-credential-to-the-blueprint)) |
| **Adele** | Reviews the evidence independently | Read access to the evidence library |

Everything below was built in Setup. This use case creates only the two access packages and their assignments.

| Asset | Built in | Used for |
|---|---|---|
| **AGT01-CPS** and its Entra agent identity | [Setup Section 10](00-Setup.md#section-10---the-copilot-studio-agent) | Scenario 1 beneficiary |
| **Policy-Digest-Agent**, sponsor Allan | [Setup Section 4](00-Setup.md#section-4---the-autonomous-agent-identity) | Scenario 2 beneficiary |
| Security group `Contoso_Finance_Allowed_Agents` (empty, grants nothing) | [Setup 3.2](00-Setup.md#32-agent-access-group-and-catalog) | Scenario 1 resource |
| Catalog `Agent Lab Catalog` | [Setup 3.2](00-Setup.md#32-agent-access-group-and-catalog) | Holds both packages |
| Access package `Agent Lab Catalog Access Package` | Created in [4.1](#41-create-the-scenario-1-package) | Scenario 1 |
| Access package `Contoso-Agent-Graph-Read` | Created in [4.2](#42-create-the-scenario-2-package) | Scenario 2 |
| Script `Contoso-Agent-Graph-Check.ps1` | Created in [4.3](#43-create-the-graph-check-script) | Scenario 2 before, during and after checks |
| Evidence folder | [Setup 5.3](00-Setup.md#53-create-the-evidence-library) | `01-Time-Bound-Agent-Access/<date>-<run>` |

### Workflow at a glance

```text
SCENARIO 1 - group membership

Alex --assigns--> Agent Lab Catalog Access Package (Member of group, 1 day)
                         |
                         v
            AGT01-CPS agent identity becomes a direct member of
            Contoso_Finance_Allowed_Agents   (type: Agent identity)
                         :
                         : after 1 day
                         v
            Assignment expired, membership removed*


SCENARIO 2 - time-limited Graph access

Nestor: Graph check BEFORE  -> no permission, Graph 403*
                         |
Allan (sponsor) --requests for Policy-Digest-Agent--> Contoso-Agent-Graph-Read
                         |
Lee --approves-->        |
                         v
            Policy-Digest-Agent receives Organization.Read.All for 2 hours
                         |
Nestor: Graph check DURING  -> permission in token, Graph 200*
                         :
                         : after 2 hours
                         v
            Assignment expired, permission removed
Nestor: Graph check AFTER   -> no permission, Graph 403*

* Expected results to confirm and record.
```

## Section 3 - Before you start

| Check | Who | Pass condition | If it fails |
|---|---|---|---|
| Check 1 - Setup is complete | Alex | Setup Sections 3.2, 4 and 10 are done: the group is empty, the catalog holds it, Policy-Digest-Agent passed its token test with Allan as sponsor, and AGT01-CPS's agent identity is recorded | Finish Setup first |
| Check 2 - Licensing | Alex | Licensing from [Section 1](#section-1---why-it-matters) is confirmed | Stop and record |
| Check 3 - Roles | Alex | Alex has Identity Governance Administrator; a Global Administrator is available for one step in 4.2; Lee and Allan use different accounts | Arrange the roles first |
| Check 4 - Nothing already assigned | Alex | In **Agent Lab Catalog**, no access package already assigns anything to AGT01-CPS or Policy-Digest-Agent | Remove the old assignment or wait for it to expire. Overlapping assignments hide expiry. |
| Check 5 - Policy-Digest-Agent has no Graph permission | Alex | **Entra ID -> Agents -> Agent identities -> Policy-Digest-Agent**: no Microsoft Graph application permission is granted. Record the actual page label. | Remove it, or record it and stop. The before check would not show a 403. |

## Section 4 - Set up the control

All steps in this section are done by **Alex** in the [Microsoft Entra admin center](https://entra.microsoft.com) unless stated otherwise. Menu labels may show **ID Governance** or **Identity Governance**, depending on the portal version.

### 4.1 Create the Scenario 1 package

1. Open **ID Governance -> Entitlement management -> Access packages -> New access package**.
2. Enter these values:

| Tab | Setting | Value |
|---|---|---|
| Basics | Name | `Agent Lab Catalog Access Package` |
| | Description | `Time-bound group membership for an Entra Agent ID lab` |
| | Catalog | `Agent Lab Catalog` |
| Resource roles | Resource | `Contoso_Finance_Allowed_Agents`, role **Member** (not Owner) |
| Requests | Who can get access | **For users, service principals, and agent identities in your directory**, scope **All agents** |
| | Who can request access | **Admin** and **Self** |
| | Require requestor justification | Yes |
| | Require approval | No |
| Requestor information | Questions | None |
| Lifecycle | Access package assignments expire | **Number of days: 1** |
| | Allow extension | No |
| | Access reviews | No |

3. Select **Create**. Record the package ID and its policy name (**Initial Policy**).
4. Reopen the policy and check that it shows: All agents; Admin and Self can request; approval No; requestor justification Yes; expires after 1 day.

**All agents** makes every agent identity eligible; it does not assign the package to any of them. Because **Self** is allowed with no approval, any agent could request this package. That is acceptable only because the group grants nothing. Close it as described in [Section 8](#section-8---put-everything-back) when you finish.

### 4.2 Create the Scenario 2 package

1. Open **ID Governance -> Entitlement management -> Access packages -> New access package**.
2. **Basics:** **Name** `Contoso-Agent-Graph-Read`. **Description** `Time-limited read access to organization details for a sponsored agent`. **Catalog** `Agent Lab Catalog`.
3. **Resource roles:** this step needs the **Global Administrator**. Select **API permissions -> Microsoft Graph -> Application permissions**, select **Organization.Read.All**, then **Update permissions**.
   - The catalog becomes a **privileged catalog** once an API permission is added. Record this.
   - If **Organization.Read.All** cannot be selected, it may be blocked for agents. Choose another read-only application permission that has a matching harmless Graph request, record it, and adjust the check in 4.3 to match.
4. **Requests:**

| Setting | Value |
|---|---|
| Who can get access | **For users, service principals, and agent identities in your directory**, scope **All agents**. API permissions can only be given to agents or service principals, not users. |
| Who can request access | **Self** and **Agent sponsor or owner**. **Admin** is always selected and cannot be cleared. Without **Agent sponsor or owner**, the sponsor cannot request for the agent: in My Access the package does not appear for Allan at all. |
| Require requestor justification | Yes |
| Require approval | Yes. **How many stages:** 1 |
| First approver | **Choose specific approvers -> Add approvers ->** Lee |
| Decision must be made in how many days? | 2 |
| Require approver justification | Yes |
| Fallback approvers | None |

5. **Requestor information:** no questions.
6. **Lifecycle:** **Access package assignments expire: Number of hours: 2**. **Allow extension:** No. **Access reviews:** No.
7. Select **Create**. Reopen the policy and record: All agents; Self and Agent sponsor or owner can request; approval required with Lee as approver; expires after 2 hours. Make sure the package is **not hidden**, so Allan can find it in My Access.

### 4.3 Create the Graph check script

**Nestor** saves this script as `Contoso-Agent-Graph-Check.ps1` next to the Setup token script. It:

- gets a fresh Microsoft Graph token for Policy-Digest-Agent, using the same token requests as the [Setup token test](00-Setup.md#46-create-the-token-test-script);
- lists the application permissions carried in that token;
- tries one harmless read, `GET /v1.0/organization`, and reports the result;
- never shows, saves or returns any token or secret.

```powershell
#Requires -Version 7.0
<#
Contoso agent Graph check.
Gets a fresh Microsoft Graph token for Policy-Digest-Agent, shows which application
permissions the token carries, and tries one read: GET /v1.0/organization.
Never prints, stores or returns token values or the secret.
#>
[CmdletBinding()]
param(
    [Parameter(Mandatory)]
    [ValidatePattern('^[0-9a-fA-F]{8}-([0-9a-fA-F]{4}-){3}[0-9a-fA-F]{12}$')]
    [string]$TenantId,

    [Parameter(Mandatory)]
    [ValidatePattern('^[0-9a-fA-F]{8}-([0-9a-fA-F]{4}-){3}[0-9a-fA-F]{12}$')]
    [string]$BlueprintAppId,

    [Parameter(Mandatory)]
    [ValidatePattern('^[0-9a-fA-F]{8}-([0-9a-fA-F]{4}-){3}[0-9a-fA-F]{12}$')]
    [string]$AgentIdentityAppId,

    [Parameter(Mandatory)]
    [ValidatePattern('^[A-Za-z0-9][A-Za-z0-9-]{2,63}$')]
    [string]$RunId,

    [securestring]$BlueprintSecret,

    [string]$CsvPath
)

$ErrorActionPreference = 'Stop'
$TokenEndpoint = "https://login.microsoftonline.com/$TenantId/oauth2/v2.0/token"
$GraphUri = 'https://graph.microsoft.com/v1.0/organization?$select=displayName'

function Request-Token {
    param([Parameter(Mandatory)][hashtable]$Body)
    try {
        $response = Invoke-RestMethod -Method Post -Uri $TokenEndpoint -Body $Body `
            -ContentType 'application/x-www-form-urlencoded'
        [pscustomobject]@{ Token = $response.access_token; ErrorCodes = '' }
    }
    catch {
        $codes = ''
        if ($_.ErrorDetails.Message) {
            try { $codes = (($_.ErrorDetails.Message | ConvertFrom-Json).error_codes -join ',') } catch { }
        }
        if (-not $codes) { $codes = 'see sign-in log' }
        [pscustomobject]@{ Token = $null; ErrorCodes = $codes }
    }
}

function Get-TokenPermissions {
    # Best-effort only: Microsoft documents that not every Microsoft API token is a
    # decodable JWT. A parsing failure here must not stop the actual authorization
    # test below, so any error is caught and reported instead of thrown.
    param([Parameter(Mandatory)][string]$Jwt)
    try {
        $payload = $Jwt.Split('.')[1].Replace('-', '+').Replace('_', '/')
        switch ($payload.Length % 4) { 2 { $payload += '==' } 3 { $payload += '=' } }
        $claims = [Text.Encoding]::UTF8.GetString([Convert]::FromBase64String($payload)) | ConvertFrom-Json
        if ($claims.roles) { ($claims.roles | Sort-Object) -join ', ' } else { '(none)' }
    }
    catch {
        '(token could not be parsed as a JWT; see Microsoft Learn on access-token formats)'
    }
}

function Test-GraphRead {
    param([Parameter(Mandatory)][string]$Token)
    try {
        $response = Invoke-RestMethod -Method Get -Uri $GraphUri -Headers @{ Authorization = "Bearer $Token" }
        [pscustomobject]@{ Status = 200; Detail = "Organization read: $($response.value[0].displayName)" }
    }
    catch {
        $status = $_.Exception.Response.StatusCode.value__
        $detail = $_.Exception.Message
        if ($_.ErrorDetails.Message) {
            try { $detail = ($_.ErrorDetails.Message | ConvertFrom-Json).error.code } catch { }
        }
        [pscustomobject]@{ Status = $status; Detail = $detail }
    }
}

$result = [ordered]@{
    RunId            = $RunId
    CheckUtc         = [datetime]::UtcNow.ToString('yyyy-MM-ddTHH:mm:ssZ')
    AgentToken       = ''
    TokenPermissions = ''
    GraphStatus      = ''
    GraphDetail      = ''
}
$plainSecret = $null
$blueprintToken = $null
$agentToken = $null

try {
    if (-not $BlueprintSecret) {
        $BlueprintSecret = Read-Host 'Blueprint client secret Value (input hidden)' -AsSecureString
    }
    $plainSecret = ConvertFrom-SecureString -SecureString $BlueprintSecret -AsPlainText

    $first = Request-Token -Body @{
        client_id     = $BlueprintAppId
        client_secret = $plainSecret
        scope         = 'api://AzureADTokenExchange/.default'
        grant_type    = 'client_credentials'
        fmi_path      = $AgentIdentityAppId
    }
    $plainSecret = $null
    $blueprintToken = $first.Token

    if (-not $blueprintToken) {
        $result.AgentToken = "NOT RUN - blueprint token failed ($($first.ErrorCodes))"
    }
    else {
        $second = Request-Token -Body @{
            client_id             = $AgentIdentityAppId
            scope                 = 'https://graph.microsoft.com/.default'
            grant_type            = 'client_credentials'
            client_assertion_type = 'urn:ietf:params:oauth:client-assertion-type:jwt-bearer'
            client_assertion      = $blueprintToken
        }
        $agentToken = $second.Token

        if (-not $agentToken) {
            $result.AgentToken = "FAILED ($($second.ErrorCodes))"
        }
        else {
            $result.AgentToken = 'ISSUED'
            $result.TokenPermissions = Get-TokenPermissions -Jwt $agentToken
            $graph = Test-GraphRead -Token $agentToken
            $result.GraphStatus = $graph.Status
            $result.GraphDetail = $graph.Detail
        }
    }
}
finally {
    $plainSecret = $null
    $blueprintToken = $null
    $agentToken = $null
    $first = $null
    $second = $null
}

[pscustomobject]$result | Format-List

if ($CsvPath) {
    [pscustomobject]$result | Export-Csv -Path $CsvPath -NoTypeInformation
    Write-Host "Sanitized results written to $CsvPath"
}
```

To run it, Nestor opens PowerShell 7 in the script's folder and replaces the run label for each check (`BEFORE`, `DURING`, `AFTER`):

```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
.\Contoso-Agent-Graph-Check.ps1 -TenantId "<TENANT-ID>" -BlueprintAppId "<BLUEPRINT-APP-ID>" -AgentIdentityAppId "<AGENT-IDENTITY-APP-ID>" -RunId "ENTRA001-LAB-01-BEFORE" -CsvPath ".\ENTRA001-LAB-01-BEFORE.csv"
```

Paste the blueprint secret Value when prompted; nothing appears as you type. The CSV contains the tenant's display name but no tokens or secrets. Store it in the evidence folder.

## Section 5 - Run it in Contoso Labs

Record every result in the [tracker](98-Tracker.md#01---entra001---time-bound-agent-access) as you go.

### Scenario 1 - Group membership for the Copilot Studio agent

**Step 1 - Record the starting point.** Alex opens **Entra ID -> Groups -> Contoso_Finance_Allowed_Agents -> Members** and captures the empty list.

**Step 2 - Assign the package to the agent.** Alex:

1. Opens **Agent Lab Catalog Access Package -> Assignments -> New assignment**.
2. Selects **Initial Policy**, then **Identities in my directory -> Add identities**.
3. Searches for AGT01-CPS's agent identity (the name recorded in [Setup 10.4](00-Setup.md#104-record-the-agents-identity)) and selects it.
4. Checks that the selected identity is the agent, not Alex, Allan, a blueprint or another app.
5. Enters the justification `Initial validation of Agent Identity entitlement management`.
6. **Assignment ends on** is blank by default: set it to one day after **Assignment starts on**. Leave **Bypass approval** as it is; it cannot be changed for this policy, which has no approval. Then selects **Add**.

The agent may not appear in the group's normal **Add members** picker. That is expected; the access package is the supported route. Do not add the agent to the group by hand.

**Step 3 - Check that access was delivered.**

1. **Access package -> Assignments:** the agent is listed with **Initial Policy** and an end date. **Delivering** is not finished; continue to the group check.
2. **Entra ID -> Groups -> Contoso_Finance_Allowed_Agents -> Members:** the agent is a direct member, **Type: Agent identity**, and its Object ID matches the one recorded in Setup 10.4.
3. Capture both pages for the evidence folder.

**Step 4 - Check expiry after one day.** After the end time, Alex refreshes **Assignments** and **Members**. The assignment shows as expired and the agent is no longer a member. If the member is still there, check again at about 15 minutes, 1 hour and 24 hours later and record the times. These are observation points, not service guarantees. Never remove the member by hand and call it expiry.

### Scenario 2 - Time-limited Graph access for the autonomous agent

**Step 5 - Check before access.** Nestor runs the Graph check with run label `ENTRA001-LAB-01-BEFORE`.

| Field | Expected |
|---|---|
| AgentToken | ISSUED |
| TokenPermissions | (none), or no `Organization.Read.All` |
| GraphStatus | **403**, with GraphDetail `Authorization_RequestDenied` |

**Step 6 - Allan requests access for the agent.** Allan signs in to [My Access](https://myaccess.microsoft.com) with his own account:

1. Selects **Access packages** and types `Contoso-Agent-Graph-Read` in **Search packages by name or description**. The list can show **No access packages available** until you search, because the package is for agents, not for Allan himself. Selects **Request**.
2. Under **Request details**, selects **Requesting for Sponsored agent**, selects **Policy-Digest-Agent** and selects **Continue**.
3. Enters the justification `Policy digest needs organization details for two hours`, then submits.
4. Records the request time.

If Allan cannot see the package or the **Requesting for Sponsored agent** option, record what he sees and the policy's **Who can request access** options. Do not replace his request with an administrator assignment and call it a sponsor request.

**Step 7 - Confirm nothing is delivered yet.** Alex opens the package's **Requests** and sees the request pending approval. There is no assignment yet.

**Step 8 - Lee approves.** Lee signs in to [My Access](https://myaccess.microsoft.com) -> **Approvals -> Pending**. He checks the requester (Allan), the beneficiary (Policy-Digest-Agent), the package and the 2-hour duration, then selects **Approve** with the justification `Approved for the lab test window`. He records the time.

**Step 9 - Check access during the period.**

1. Alex opens **Access package -> Assignments**: Policy-Digest-Agent is listed, delivered, with an end time 2 hours after delivery. Record it.
2. Alex opens **Entra ID -> Agents -> Agent identities -> Policy-Digest-Agent** and records the page that shows **Organization.Read.All** granted. Record the actual label.
3. Nestor runs the Graph check with run label `ENTRA001-LAB-01-DURING`:

| Field | Expected |
|---|---|
| TokenPermissions | Includes `Organization.Read.All` |
| GraphStatus | **200**, with GraphDetail `Organization read: <tenant name>` |

If it still shows 403, wait 5 minutes and run it again with `-DURING-02`. Delivery can take a few minutes.

**Step 10 - Check access after the period.** After the end time:

1. Alex refreshes **Assignments**: the assignment shows as expired. The permission is gone from the agent identity's page.
2. Nestor runs the Graph check with run label `ENTRA001-LAB-01-AFTER`. Expected: TokenPermissions without `Organization.Read.All`, GraphStatus **403**.
3. If access is still present, check again at about 15 minutes, 1 hour and 24 hours, and record the times.

The script always gets a new token. A token obtained during the access period would still work until it expires, usually about an hour; this test does not measure that.

## Section 6 - What you should see

| Evidence | What it supports |
|---|---|
| The agent is selectable in **Add identities** | The agent identity is an eligible beneficiary |
| Assignment with an end date, and the agent is a direct group member of type **Agent identity** | Access was delivered to the agent, not to a person |
| An end date only | A time limit is configured; expiry is not yet shown |
| Expired assignment and membership removed | The time-bound lifecycle works for the agent |
| Allan's request, Lee's approval and a delivered assignment | Sponsor request and independent approval work for an agent |
| Graph check: 403 before, 200 during, 403 after | The access package controlled what the agent could actually do, for a limited time |
| AGT01-CPS chat still works in Scenario 1 | Expected; the group grants nothing. This is not proof of any resource access. |

## Section 7 - If something goes wrong

| What you see | What to do |
|---|---|
| Agent not in the group's **Add members** picker | Expected. Use the access package assignment, never manual membership. |
| Agent not found under **Add identities** | Check the policy scope is **All agents**, the agent identity exists in this tenant and you selected the agent identity, not an app registration |
| Assignment stays **Delivering**, or the member is missing | Open the assignment's details for errors. Check the resource role is group **Member** and the beneficiary's Object ID is correct. |
| Organization.Read.All cannot be added | Confirm a Global Administrator is adding it. The permission may be blocked for agents; choose another read-only permission and record it. |
| Allan cannot find the package or request for the agent | Search for the package by name in My Access. Check the policy has **Agent sponsor or owner** under **Who can request access** (4.2), Allan is a direct sponsor of Policy-Digest-Agent (Setup 4.5), and the package is not hidden. |
| Graph check still 403 during the period | Wait 5 to 15 minutes and run again. Check the permission shows on the agent identity's page. |
| Graph check fails before Graph is called | Use the error table in [Setup 4.7](00-Setup.md#47-test-that-the-agent-identity-works) |
| Assignment expired but membership or permission remains | Look for another assignment or a separately granted permission. Record the result as incomplete. |

## Section 8 - Put everything back

1. Prefer natural expiry. If you must end early, Alex selects the assignment in **Access package -> Assignments -> Remove** and labels it **Manual removal**, never expiry.
2. Close new requests on both packages: edit each policy and clear **Self** (and, on the Scenario 2 package, **Agent sponsor or owner**) under **Who can request access**, so only Admin remains. Or delete the packages when no assignment is active.
3. Check that Policy-Digest-Agent no longer has **Organization.Read.All**, and that `Contoso_Finance_Allowed_Agents` is empty again.
4. Record in the tracker that **Agent Lab Catalog** is now a privileged catalog.
5. Leave AGT01-CPS, Policy-Digest-Agent and Allan's sponsorship unchanged. Later use cases need them.

## Section 9 - Outcome

| Action | Result |
|---|---|
| The agent identity selected as beneficiary | "The agent itself, not an administrator, receives the access." |
| Package settings: approver, justification, end date | "Agent access now follows the same governed process you use for people." |
| Allan's request and Lee's approval | "The agent's sponsor asked for the access; someone else approved it." |
| Graph check 403, then 200 | "The agent could read this data only after approval." |
| Graph check 403 again after the end time | "The access ended on its own. Nobody had to remember to remove it, and the agent kept running." |

**Do not say:**

- that access ends instantly. Tokens issued earlier stay valid until they expire;
- that Copilot Studio chat changed because of the group in Scenario 1;
- that administrators can no longer grant access in other ways.

Record the conclusion using the [results criteria](99-Results-and-Handoff.md#01---entra001---time-bound-agent-access).

## Section 10 - Sources

| Topic | Source |
|---|---|
| Access packages for agent identities: resource types, All agents scope, the three request routes, expiry and sponsor notifications, Global Administrator for API permissions, privileged catalogs | [Access packages for agent identities](https://learn.microsoft.com/en-us/entra/agent-id/agent-access-packages) |
| Adding API permissions to an access package (preview); permissions blocked for agents | [Add resource roles to an access package](https://learn.microsoft.com/en-us/entra/id-governance/entitlement-management-access-package-resources) |
| Sponsor request: **Requesting for Sponsored agent** in My Access | [Request an access package on behalf of an agent identity](https://learn.microsoft.com/en-us/entra/id-governance/entitlement-management-request-behalf#request-an-access-package-on-behalf-of-an-agent-identity) |
| **Self** replaces "Enable new requests"; Admin and Manager options | [Change request settings](https://learn.microsoft.com/en-us/entra/id-governance/entitlement-management-access-package-request-policy) |
| Agent sponsors or owners as on-behalf requestors | [targetAgentIdentitySponsorsOrOwners](https://learn.microsoft.com/en-us/graph/api/resources/targetagentidentitysponsorsorowners), [requestor settings](https://learn.microsoft.com/en-us/graph/api/resources/accesspackageassignmentrequestorsettings) |
| Approval stages, specific approvers, approver justification | [Change approval settings](https://learn.microsoft.com/en-us/entra/id-governance/entitlement-management-access-package-approval-policy) |
| Expiry in days or hours | [Change lifecycle settings](https://learn.microsoft.com/en-us/entra/id-governance/entitlement-management-access-package-lifecycle-policy) |
| Direct assignment and assignment status | [View and manage assignments](https://learn.microsoft.com/en-us/entra/id-governance/entitlement-management-access-package-assignments) |
| Licensing for agent identity governance | [Microsoft Entra ID Governance licensing](https://learn.microsoft.com/en-us/entra/id-governance/licensing-fundamentals) |
| Copilot Studio SharePoint knowledge runs as the signed-in user | [SharePoint knowledge in Copilot Studio](https://learn.microsoft.com/en-us/microsoft-copilot-studio/knowledge-add-sharepoint) |
