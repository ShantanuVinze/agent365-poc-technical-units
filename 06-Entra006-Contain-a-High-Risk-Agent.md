# 06 - Entra006 - Contain a high-risk agent

This use case shows that a standing Conditional Access policy blocks an agent automatically when its identity becomes **high risk**, and lets it back in when the risk is cleared, without anyone editing the policy.

Lee, a security analyst, declares **Policy-Digest-Agent** compromised. Its risk becomes High, and a policy that blocks high-risk agents stops it getting new tokens. When Lee dismisses the risk, the agent gets tokens again, with the policy still on.

This is a **simulation**: Lee declares the compromise; no real attack happens. Agent risk in Conditional Access and the risky-agent APIs are in **preview**.

## Section 1 - Why it matters

**Contoso's concern:** if an agent is compromised, its access should stop as soon as the security team knows, everywhere and automatically. It should come back only when the risk is cleared.

**What Contoso can already do:** Microsoft Entra ID Protection and risk-based Conditional Access protect **people's** sign-ins, and, with Workload ID Premium, apps.

**What Agent 365 adds:** ID Protection assigns risk to **agent identities**, and Conditional Access has an **Agent risk** condition. A security analyst can **confirm an agent compromised**, which sets its risk to High and triggers any policy that blocks high-risk agents. Microsoft may already have created a policy for this in your tenant: **Block all high risk agents from accessing all resources (Preview)**, which Microsoft creates in Report-only and can turn on automatically about 30 days later.

**Licensing:** Conditional Access for agents needs Microsoft 365 E7, or Microsoft Agent 365 with at least Microsoft Entra ID P1 or Microsoft 365 E3 for an ordinary policy - but this use case's policy targets **agent risk level**, which needs **Microsoft Entra ID P2 or Microsoft 365 E5** instead. Check the current requirement for [ID Protection for agents](https://learn.microsoft.com/en-us/entra/id-protection/concept-risky-agents) and [risk-based Conditional Access for agents](https://learn.microsoft.com/en-us/entra/identity/conditional-access/policy-autonomous-agents#prerequisites); Microsoft states it will require Agent 365.

| This use case shows, when observed | It does not show |
|---|---|
| A standing policy blocks the agent's new tokens once its risk is High | That ID Protection would detect a real attack on this agent. Lee set the risk himself. |
| Clearing the risk restores access, with the policy still on | That tokens issued before the risk changed stop working at once |
| The policy does nothing while the agent's risk is None | How quickly Microsoft's own detections, which run offline, would raise the risk |
| | That agents acting for a signed-in user are contained this way. In that case, risk is attributed to the user, not the agent. |

## Section 2 - People and assets

| Name | Role in this use case | Microsoft access needed |
|---|---|---|
| **Lee** | Checks the agent's risk; declares it compromised; dismisses the risk | **Security Administrator**, for the test window only (granted in Check 4). It is the least-privileged role for the risk actions. |
| **Alex** | Creates the risk policy and changes its state; handles the Microsoft-managed policy; arranges consent | Conditional Access Administrator. For consent (4.2): Privileged Role Administrator, Cloud Application Administrator or Application Administrator. |
| **Nestor** | Runs the token test at each step | Access to the blueprint secret ([Setup 4.3](00-Setup.md#43-add-a-temporary-credential-to-the-blueprint)) |
| **Adele** | Reviews the evidence independently | Read access to the evidence library |

| Asset | Built in | Used for |
|---|---|---|
| **Policy-Digest-Agent** and its **Object ID** | [Setup Section 4](00-Setup.md#section-4---the-autonomous-agent-identity) | The agent whose risk changes. For an agent identity, the Object ID equals the App ID. |
| Token test script `Contoso-Agent-Token-Test.ps1` | [Setup 4.6](00-Setup.md#46-create-the-token-test-script) | Shows whether the agent can get tokens |
| Conditional Access policy `Contoso-CA-Agent-High-Risk-Block` | Created in [4.1](#41-create-the-risk-policy-in-report-only) | The control demonstrated |
| Microsoft Graph PowerShell commands | Prepared in [4.2](#42-prepare-the-risk-commands) | Declare the compromise and dismiss the risk |
| Evidence folder | [Setup 5.3](00-Setup.md#53-create-the-evidence-library) | `06-Contain-a-High-Risk-Agent/<date>-<run>` |

### Workflow at a glance

```text
Alex creates Contoso-CA-Agent-High-Risk-Block
   Applies to: Policy-Digest-Agent | Resources: All
   Condition:  Agent risk = High   | Grant: Block

  Step 1  No policy, risk None     tokens issued*
  Step 2  Report-only, risk None   tokens issued; policy "Not applied"*
  Step 3  Lee: confirm compromised -> risk High
          Report-only              tokens issued; policy "Report-only: Failure"*
  Step 4  Policy On, risk High     tokens denied, error 53003*
  Step 5  Lee: dismiss risk -> risk None
          Policy still On          tokens issued again; policy "Not applied"*

* Expected results to confirm and record.
```

## Section 3 - Before you start

| Check | Who | Pass condition | If it fails |
|---|---|---|---|
| Check 1 - Agent works | Nestor | [Setup 4.7](00-Setup.md#47-test-that-the-agent-identity-works) passed: all three steps **TOKEN ISSUED** | Finish Setup Section 4 first |
| Check 2 - No other policy blocks the agent | Alex | The policies from use cases 02 and 03 are **Off** | Turn them off. They would block the agent for a different reason. |
| Check 3 - Microsoft's high-risk policy | Alex | **Entra ID -> Conditional Access -> Policies**, look for policies with **Created by: Microsoft**. Record the state of **Block all high risk agents from accessing all resources (Preview)**. If it is **On**, with approval, add **Policy-Digest-Agent** to its exclusions for the test window. | If it is On and not excluded, it blocks the agent at Step 3 and you cannot tell which policy acted. Record the original state and exclusions so you can restore them. |
| Check 4 - Lee's role | Alex | Lee holds **Security Administrator** for the test window only, with approval | Arrange a time-limited assignment |
| Check 5 - The agent has no risk | Lee | **Entra ID -> ID Protection -> Risky agents**: neither **Policy-Digest-Agent** nor **Contoso-Autonomous-Blueprint** is listed, or both show risk level **None** | Record the existing detections first. Setup activity can raise risk on its own, for example a new blueprint secret being used, or the agent requesting an Azure management token it does not usually request. Dismiss the risk with approval before starting, or stop. |
| Check 6 - Licensing | Alex | Licensing from [Section 1](#section-1---why-it-matters) is confirmed. Record any licence warning on the Risky agents page. | Stop and record |

Run Check 5 again immediately before Step 1. Microsoft's detections run offline and can raise risk later.

## Section 4 - Set up the control

### 4.1 Create the risk policy in Report-only

**Alex**, in the [Microsoft Entra admin center](https://entra.microsoft.com):

1. Open **Entra ID -> Conditional Access -> Policies -> New policy**.
2. **Name:** `Contoso-CA-Agent-High-Risk-Block`.
3. **Assignments -> Users, agents or workload identities:** under **What does this policy apply to?** select **Agents**. Under **Include**, choose **Select individual agent identities**, open the **Agent identities** tab and select **Policy-Digest-Agent** only. (Microsoft's template targets all agent identities; this test uses one agent.)
4. **Target resources -> Include:** **All resources (formerly 'All cloud apps')**.
5. **Conditions -> Agent risk (Preview):** set **Configure** to **Yes** and select **High** only.
6. **Grant:** **Block**, then **Select**.
7. **Enable policy:** **Report-only**. Select **Create**.
8. Reopen the policy and record: its ID, the single agent, All resources, Agent risk High and Block.

### 4.2 Prepare the risk commands

Lee changes the agent's risk with Microsoft Graph commands. The **Risky agents** page has the same buttons, but it lists only agents that already have risk. Policy-Digest-Agent starts with none (Check 5), so the first action needs the command.

These are Microsoft Graph **beta** APIs and may change.

**What you need:**

| Item | Value |
|---|---|
| Tenant ID | **Entra ID -> Overview -> Tenant ID** |
| Agent Object ID | **Entra ID -> Agents -> Agent identities -> Policy-Digest-Agent -> Object ID** |
| Permissions | Delegated **IdentityRiskyAgent.Read.All** and **IdentityRiskyAgent.ReadWrite.All**. Both need administrator consent. |

**Step A - Install the module (Lee, once):**

```powershell
Install-Module Microsoft.Graph.Authentication -Scope CurrentUser
```

**Step B - Grant consent (Alex, once):** Alex signs in with an account that can grant consent: Privileged Role Administrator, Cloud Application Administrator or Application Administrator. Security Administrator cannot grant consent.

```powershell
Connect-MgGraph -TenantId "<TENANT-ID>" -Scopes "IdentityRiskyAgent.Read.All","IdentityRiskyAgent.ReadWrite.All" -ContextScope Process
```

In the sign-in window, check the two permissions, select **Consent on behalf of your organization**, then **Accept**. Run `Disconnect-MgGraph`. Record who consented and when.

This grants the two permissions to Microsoft's **Microsoft Graph Command Line Tools** app for the tenant. They are delegated permissions: a person can use them only within their own role, so nobody gains new rights. They are removed in [Section 8](#section-8---put-everything-back).

**Step C - Sign in and read the agent's risk (Lee):**

```powershell
Connect-MgGraph -TenantId "<TENANT-ID>" -Scopes "IdentityRiskyAgent.Read.All","IdentityRiskyAgent.ReadWrite.All" -ContextScope Process -NoWelcome
$AgentObjectId  = "<POLICY-DIGEST-AGENT-OBJECT-ID>"
$RiskyAgentsUri = "https://graph.microsoft.com/beta/identityProtection/riskyAgents"
$Body = @{ agentIds = @($AgentObjectId) } | ConvertTo-Json -Compress

$risk = Invoke-MgGraphRequest -Method GET -Uri "$RiskyAgentsUri/$AgentObjectId"
$risk.riskLevel; $risk.riskState
```

| What you see | Meaning |
|---|---|
| `riskLevel` **none** | No risk; Check 5 passes |
| A **404 Not Found** error | Usually means the agent has never had risk. Confirm it is not on the Risky agents page. |
| `riskLevel` low, medium or high | The agent already has risk. Go back to Check 5. |
| Nothing prints | Run `$risk \| ConvertTo-Json` to see the full response |

Keep this PowerShell window open. Step 3 and Step 5 in Section 5 use the commands below:

```powershell
# Step 3 only - declare the agent compromised (risk becomes High)
Invoke-MgGraphRequest -Method POST -Uri "$RiskyAgentsUri/confirmCompromised" -Body $Body -ContentType "application/json"
"confirmCompromised sent (UTC): " + (Get-Date).ToUniversalTime().ToString("o")
```

```powershell
# Step 5 only - dismiss the risk (risk becomes None)
Invoke-MgGraphRequest -Method POST -Uri "$RiskyAgentsUri/dismiss" -Body $Body -ContentType "application/json"
"dismiss sent (UTC): " + (Get-Date).ToUniversalTime().ToString("o")
```

A command that prints only the "sent" line, with no red error, was accepted (204 No Content). Re-run the two lines from Step C to check the new risk level.

**Why dismiss rather than confirm safe?** **Confirm safe** tells ID Protection the risk was a false positive. Here the risk was real, by Lee's declaration, and is now resolved, so **Dismiss** is the accurate choice. Both set the risk level to None and need the same permission and role.

## Section 5 - Run it in Contoso Labs

**Nestor** runs the token test from [Setup 4.6](00-Setup.md#46-create-the-token-test-script) at each step and saves the CSV in the evidence folder. **Lee** finds the agent's sign-in events as in [use case 02, 4.2](02-Entra002-Autonomous-Agent-Conditional-Access.md#42-know-where-to-find-the-evidence) and records the result for `Contoso-CA-Agent-High-Risk-Block`. Record every result in the [tracker](98-Tracker.md#06---entra006---contain-a-high-risk-agent).

**Step 1 - No policy, no risk.** If you already created the policy in 4.1, Alex sets it to **Off** first. Nestor runs the test with `ENTRA006-LAB-01-STEP1`. Expected: all three steps **TOKEN ISSUED**.

**Step 2 - Report-only, no risk.** Alex sets the policy to **Report-only**. Nestor runs the test with `ENTRA006-LAB-01-STEP2`.

| Script result | Sign-in result |
|---|---|
| All three steps **TOKEN ISSUED** | The policy shows **Report-only: Not applied**: the agent's risk is not High |

This is the control: the policy does nothing while the agent is healthy.

**Step 3 - Declare the compromise.** Lee runs the **Step 3** command from 4.2 and records the time. He checks:

- the risk level is **high** (re-run the Step C lines);
- **ID Protection -> Risky agents** lists Policy-Digest-Agent at **High**;
- **ID Protection -> Risk detections -> Agent detections** shows a **Confirmed compromised** detection.

Then Nestor runs the test with `ENTRA006-LAB-01-STEP3`.

| Script result | Sign-in result |
|---|---|
| All three steps **TOKEN ISSUED**: the policy is still in Report-only | The policy shows **Report-only: Failure**: it would have blocked the agent |

Confirm the High risk is visible in ID Protection before reading the policy result.

**Step 4 - Turn the policy on.** With approval, Alex sets the policy to **On**. Nestor runs the test with `ENTRA006-LAB-01-STEP4`.

| Script result | Sign-in result |
|---|---|
| Blueprint-Token **TOKEN ISSUED**. Agent-Token-Graph and Agent-Token-ARM **DENIED OR FAILED**, error **53003** | The agent's events fail, with `Contoso-CA-Agent-High-Risk-Block` as the blocking policy |

**Step 5 - Dismiss the risk, with the policy still on.** Lee runs the **Step 5** command from 4.2, or selects Policy-Digest-Agent in **ID Protection -> Risky agents -> Dismiss risk**. He checks the risk level is **none**. Alex does **not** change the policy. Nestor runs the test with `ENTRA006-LAB-01-STEP5`.

| Script result | Sign-in result |
|---|---|
| All three steps **TOKEN ISSUED** again | The policy shows **Not applied**: the risk condition is no longer met |

Turning the policy off would be a different result. Recovery here must happen with the policy **On**.

**Timing:** a risk change can take a while to affect new token requests. If Step 3, 4 or 5 does not show the expected result, run the test again after about 5, 15 and 30 minutes with a suffix such as `-02`, and record the times. Sign-in log entries for a **blocked** request (Step 4) can take noticeably longer to appear than entries for an **allowed** one (Steps 1-3, 5) - give blocked events extra time before concluding the log is missing or wrong.

## Section 6 - What you should see

| Evidence | What it supports |
|---|---|
| Step 2: **Report-only: Not applied** while risk is None | The policy leaves a healthy agent alone |
| Step 3: risk High, a **Confirmed compromised** detection, and **Report-only: Failure** | The policy reacts to the agent's risk |
| Step 4: agent tokens denied with 53003 and the named policy | A high-risk agent is blocked automatically |
| Step 5: risk None, tokens issued, policy still On | Clearing the risk restored access; nobody edited the policy |

## Section 7 - If something goes wrong

| What you see | What to do |
|---|---|
| The agent is already at risk before Step 1 | A detection fired, possibly from setup activity. Record it; dismiss it with approval, or stop. |
| Sign-in shows **Need admin approval** (AADSTS65001 or AADSTS90094) | Consent is missing. Complete 4.2 Step B. |
| Sign-in blocked for **Microsoft Graph Command Line Tools** | The tenant restricts that app. Ask its owner to allow Lee for the test window, and record it. |
| A risk command returns **403 Forbidden** | Lee's Security Administrator role is not active (for example, a time-limited role not yet activated), or consent is missing |
| A risk command returns **400** or **404** | Check you used the agent's Object ID. If it still fails, the API did not accept this agent; record it and stop. |
| The agent is not High after Step 3 | Wait and check again; record the delay |
| Step 3 shows **Report-only: Not applied** | The risk is not yet visible to Conditional Access, or the policy does not include Policy-Digest-Agent. Wait, then check the policy. |
| Step 4 still issues tokens | Check the policy is **On** and the risk is still High; run again after a few minutes |
| Step 4 denial names Microsoft's high-risk policy, not yours | Check 3 was not completed. Exclude the agent from Microsoft's policy and repeat Step 4. |
| Step 5 still denies tokens | Check the risk is None; run again after a few minutes; look for another blocking policy |

## Section 8 - Put everything back

1. Confirm Policy-Digest-Agent's risk level is **None**.
2. Alex sets `Contoso-CA-Agent-High-Risk-Block` to **Off**, or deletes it with approval.
3. Restore Microsoft's **Block all high risk agents** policy to its recorded state: remove the exclusion added in Check 3.
4. Lee runs `Disconnect-MgGraph`. Remove Lee's Security Administrator role.
5. Remove the consent from 4.2 Step B: **Entra ID -> Enterprise apps -> All applications -> Microsoft Graph Command Line Tools -> Security -> Permissions -> Admin consent**. Revoke **IdentityRiskyAgent.Read.All** and **IdentityRiskyAgent.ReadWrite.All** only, and record the label of the control you used. Do not revoke other permissions there; other administrators may rely on them.
6. The risk history stays in ID Protection for up to 90 days. Do not try to delete it.
7. Keep the blueprint secret for the later use cases.

## Section 9 - Outcome

| Action | Result |
|---|---|
| The policy in place while the agent is healthy | "The policy is already in place, and does nothing while the agent is healthy." |
| Lee declares the agent compromised | "As soon as security confirmed the agent was compromised, the same policy would block it." |
| The policy blocks the agent's tokens | "The compromised agent could no longer get new access. This was a simulation: Lee declared the compromise." |
| Lee dismisses the risk | "When the risk was cleared, the agent recovered. Nobody touched the policy." |

**Do not say:**

- that ID Protection detected a real attack;
- that tokens issued before the risk change were cut off;
- that agents acting for a signed-in user are contained this way; their risk is attributed to the user;
- that Microsoft's own high-risk policy was tested. It was excluded so that the result comes from your policy.

Record the conclusion using the [results criteria](99-Results-and-Handoff.md#06---entra006---contain-a-high-risk-agent).

## Section 10 - Sources

| Topic | Source |
|---|---|
| ID Protection for agents: detections (including Confirmed compromised, Suspicious credential usage, Unfamiliar resource access), offline detections, Risky agents report, Confirm compromise, Confirm safe, Dismiss risk, roles, licensing, 90-day retention, on-behalf-of attribution | [ID Protection for agents](https://learn.microsoft.com/en-us/entra/id-protection/concept-risky-agents) |
| Agent risk condition, Block, Report-only | [Secure autonomous agents with Conditional Access](https://learn.microsoft.com/en-us/entra/identity/conditional-access/policy-autonomous-agents) |
| Microsoft-managed **Block all high risk agents** policy; editing state and exclusions | [Microsoft-managed policies](https://learn.microsoft.com/en-us/entra/identity/conditional-access/managed-policies) |
| confirmCompromised: Object IDs, sets risk High, Security Administrator, 204 | [riskyAgent: confirmCompromised](https://learn.microsoft.com/en-us/graph/api/riskyagent-confirmcompromised?view=graph-rest-beta) |
| dismiss: sets risk None, same permission and role | [riskyAgent: dismiss](https://learn.microsoft.com/en-us/graph/api/riskyagent-dismiss?view=graph-rest-beta) |
| Reading one risky agent | [Get riskyAgent](https://learn.microsoft.com/en-us/graph/api/riskyagent-get?view=graph-rest-beta) |
| Both IdentityRiskyAgent permissions need admin consent | [Microsoft Graph permissions reference](https://learn.microsoft.com/en-us/graph/permissions-reference) |
| Roles that can grant admin consent | [Grant tenant-wide admin consent](https://learn.microsoft.com/en-us/entra/identity/enterprise-apps/grant-admin-consent) |
| Connect-MgGraph and Invoke-MgGraphRequest | [Microsoft Graph PowerShell authentication](https://learn.microsoft.com/en-us/powershell/microsoftgraph/authentication-commands) |
| Report-only result labels | [Report-only mode](https://learn.microsoft.com/en-us/entra/identity/conditional-access/concept-conditional-access-report-only) |
