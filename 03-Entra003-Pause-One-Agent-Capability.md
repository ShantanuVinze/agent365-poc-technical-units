# 03 - Entra003 - Pause one agent capability

This use case shows that you can stop an agent getting access to one service while it keeps access to another. Nothing else about the agent changes.

In each run, **Policy-Digest-Agent** asks for two tokens: one for **Azure Resource Manager** (the Azure management service) and one for **Microsoft Graph**. A Conditional Access policy targets Azure management only. You see the Azure management token flagged, then blocked, then allowed again, while the Microsoft Graph token is issued every time.

## Section 1 - Why it matters

**Contoso's concern:** sometimes one thing an agent does needs to stop, for example during a change freeze, an investigation or a suspected misuse of one service. Disabling the whole agent, or removing people's access, would stop more than necessary.

**What Contoso can already do:** Conditional Access can block people, or apps with Microsoft Entra Workload ID Premium, from chosen services.

**What Agent 365 adds:** the same targeting works for **one agent identity**. A policy can block one agent from one service, and leave that agent's other access, other agents and all people untouched.

**Licensing:** Microsoft 365 E7, or Microsoft Agent 365 with at least Microsoft Entra ID P1 or Microsoft 365 E3 ([Conditional Access for agents](https://learn.microsoft.com/en-us/entra/identity/conditional-access/agent-id#requirements-and-licensing)).

**Why Azure management and not Microsoft Graph?** Microsoft Graph cannot be selected as a single target in Conditional Access; Microsoft treats it as an umbrella over many services. The Azure management target can be selected. The portal now lists it as **Azure Resource Manager** (app ID `797f4846-ba00-4fd7-ba43-dac1f8f63013`); Microsoft's documentation and older portals call it **Windows Azure Service Management API**. It covers Azure Resource Manager, the Azure portal (including the Microsoft Entra admin center), Azure Data Lake, the Application Insights API and the Log Analytics API. This use case therefore blocks Azure management and uses Microsoft Graph as the comparison that keeps working.

| This use case shows, when observed | It does not show |
|---|---|
| Conditional Access checks the agent's request for one specific service | That the agent can do anything in Azure or Graph. It has no Azure role and no Graph data permission; it only receives tokens. |
| The Azure management token is denied while the Microsoft Graph token is still issued | That every other service the agent might use is unaffected. Only Microsoft Graph is compared. |
| Turning the policy off restores the Azure management token, with no change to the agent | That tokens issued before the policy was turned on stop working. They stay valid until they expire, usually about an hour. |
| The block is limited to this agent | Control over single operations or single Azure resources. The policy covers the whole Azure management service group. |

## Section 2 - People and assets

| Name | Role in this use case | Microsoft access needed |
|---|---|---|
| **Alex** | Creates the policy and changes its state | Conditional Access Administrator |
| **Nestor** | Runs the token test at each step | Access to the blueprint secret ([Setup 4.3](00-Setup.md#43-add-a-temporary-credential-to-the-blueprint)) |
| **Lee** | Finds each run in the sign-in logs and records the policy result | Reports Reader |
| **Adele** | Reviews the evidence independently | Read access to the evidence library |

Allan is not needed: no person signs in during this use case.

| Asset | Built in | Used for |
|---|---|---|
| **Policy-Digest-Agent** | [Setup Section 4](00-Setup.md#section-4---the-autonomous-agent-identity) | The agent the policy targets |
| Token test script `Contoso-Agent-Token-Test.ps1` | [Setup 4.6](00-Setup.md#46-create-the-token-test-script) | Requests a blueprint token, then agent tokens for Microsoft Graph (`Agent-Token-Graph`) and Azure Resource Manager (`Agent-Token-ARM`) |
| Conditional Access policy `Contoso-CA-Agent-ARM-Pause` | Created in [4.1](#41-create-the-policy-in-report-only) | The control demonstrated |
| Evidence folder | [Setup 5.3](00-Setup.md#53-create-the-evidence-library) | `03-Pause-One-Agent-Capability/<date>-<run>` |

### Workflow at a glance

```text
Alex creates Contoso-CA-Agent-ARM-Pause
   Applies to: Policy-Digest-Agent only
   Resource:   Azure Resource Manager only | Grant: Block

Nestor runs Contoso-Agent-Token-Test.ps1 four times:

                     Agent-Token-Graph          Agent-Token-ARM
  Step 1 No policy   issued                     issued
  Step 2 Report-only issued                     issued; sign-in shows
                     (policy Not applied)*      "Report-only: Failure"*
  Step 3 On          issued*                    denied, error 53003*
  Step 4 Off         issued*                    issued again*

Lee matches each run to the agent's two sign-in events
and records the result for Contoso-CA-Agent-ARM-Pause

* Expected results to confirm and record.
```

## Section 3 - Before you start

| Check | Who | Pass condition | If it fails |
|---|---|---|---|
| Check 1 - Both tokens work | Nestor | In [Setup 4.7](00-Setup.md#47-test-that-the-agent-identity-works), both **Agent-Token-Graph** and **Agent-Token-ARM** showed **TOKEN ISSUED** | If the Azure management token failed in Setup, stop: this use case needs it. Record the error. Do not grant Azure roles to work around it. |
| Check 2 - Use case 02's policy is off | Alex | `Contoso-CA-Block-Policy-Digest-Agent` from use case 02 is **Off** or deleted | Turn it off. It blocks all resources and would deny both tokens. |
| Check 3 - No other policy blocks the agent | Alex | No other enforced policy applies to Policy-Digest-Agent, including Microsoft-managed policies such as **Block all high risk agents from accessing all resources (Preview)** ([Setup 4.5](00-Setup.md#45-check-the-agent-identitys-settings)) | Record it and agree with its owner |
| Check 4 - The target can be selected | Alex | **Conditional Access -> New policy -> Target resources -> Select resources** lists **Azure Resource Manager** (shown as **Windows Azure Service Management API** in older portals). Do not save. | Stop and record. Do not use All resources instead: that would also block Graph and remove the comparison. |

## Section 4 - Set up the control

### 4.1 Create the policy in Report-only

**Alex**, in the [Microsoft Entra admin center](https://entra.microsoft.com):

1. Open **Entra ID -> Conditional Access -> Policies -> New policy**.
2. **Name:** `Contoso-CA-Agent-ARM-Pause`.
3. **Assignments -> Users, agents or workload identities:** under **What does this policy apply to?** select **Agents**. Under **Include**, choose **Select individual agent identities**, open the **Agent identities** tab and select **Policy-Digest-Agent** only. Do not select the blueprint.
4. **Target resources -> Include -> Select resources:** select **Azure Resource Manager** only (app ID `797f4846-ba00-4fd7-ba43-dac1f8f63013`, formerly shown as **Windows Azure Service Management API**).
5. **Conditions:** none.
6. **Grant:** **Block**, then **Select**.
7. **Enable policy:** **Report-only**. Select **Create**.
8. Reopen the policy and record in the tracker: its ID, the single agent, the single resource and Block.

Only the agent is included, so no person loses access to the Azure portal or Azure management through this policy.

### 4.2 Know where to find the evidence

Each run produces two agent events: one for Microsoft Graph and one for Azure Resource Manager. **Lee** finds them as in [use case 02, 4.2](02-Entra002-Autonomous-Agent-Conditional-Access.md#42-know-where-to-find-the-evidence):

1. **Entra ID -> Monitoring & health -> Sign-in logs**, filtered by **Agent type: Agent Identity**, time range around **RequestUtc** from the CSV.
2. Match each event to its CSV row by **Correlation ID** (equal to **ClientRequestId**) or by time and resource.
3. Open each event's **Conditional Access** tab, or **Report-only** tab while the policy is in Report-only, and record the result for `Contoso-CA-Agent-ARM-Pause` only.

## Section 5 - Run it in Contoso Labs

Each step: **Alex** sets the policy state, **Nestor** runs the token test with the run label shown and saves the CSV in the evidence folder, and **Lee** records the policy result on both agent events. Record every result in the [tracker](98-Tracker.md#03---entra003---pause-one-agent-capability).

```powershell
.\Contoso-Agent-Token-Test.ps1 -TenantId "<TENANT-ID>" -BlueprintAppId "<BLUEPRINT-APP-ID>" -AgentIdentityAppId "<AGENT-IDENTITY-APP-ID>" -RunId "ENTRA003-LAB-01-STEP1" -CsvPath ".\ENTRA003-LAB-01-STEP1.csv"
```

**Step 1 - No policy.** If you already created the policy in 4.1, Alex sets it to **Off** first. Nestor runs the test with `ENTRA003-LAB-01-STEP1`.

| Script result | Sign-in result |
|---|---|
| Blueprint-Token, Agent-Token-Graph and Agent-Token-ARM: **TOKEN ISSUED** | Two successful agent events, one for each service |

**Step 2 - Report-only.** Alex sets the policy to **Report-only**. Nestor runs the test with `ENTRA003-LAB-01-STEP2`.

| Script result | Sign-in result |
|---|---|
| All three steps **TOKEN ISSUED** | Azure Resource Manager event: **Report-only: Failure**. Microsoft Graph event: **Report-only: Not applied**, or the policy is not listed at all, because it does not match Graph. |

This is the key result: the policy would block one service and not the other. A token issued in Report-only proves nothing on its own. Do not continue until you see both results.

**Step 3 - On.** Only after Step 2 passes, and with approval, Alex sets the policy to **On**. Nestor runs the test with `ENTRA003-LAB-01-STEP3`.

| Script result | Sign-in result |
|---|---|
| Blueprint-Token and Agent-Token-Graph **TOKEN ISSUED**. Agent-Token-ARM **DENIED OR FAILED**, error **53003**. | Azure Resource Manager event fails, with `Contoso-CA-Agent-ARM-Pause` as the blocking policy. Microsoft Graph event succeeds. |

**Step 4 - Off.** Alex sets the policy to **Off**. Nestor runs the test with `ENTRA003-LAB-01-STEP4`.

| Script result | Sign-in result |
|---|---|
| All three steps **TOKEN ISSUED** again | Both agent events succeed |

**Timing:** a policy change can take several minutes to take effect, and it can reach one resource before another. In the author's lab testing (a single tenant, not a Microsoft-documented behavior), a token for one resource was already following the new state while the token for the other still followed the old one, more than 5 minutes after the change, and one change from Report-only to On was enforced only after 15 to 45 minutes. If a step does not show the expected result for both resources, run it again with a suffix such as `-02` after about 5, 15 and 30 minutes, and record the times. The sign-in events themselves can take 15 to 20 minutes to appear.

## Section 6 - What you should see

| Evidence | What it supports |
|---|---|
| Step 1: both agent tokens issued | The agent gets tokens for two services on its own |
| Step 2: **Report-only: Failure** on the Azure management event, **Not applied** on the Graph event | The policy matches only the Azure management request |
| Step 3: Azure management denied with 53003 and the named policy; Graph still issued | One capability was paused while the other kept working |
| Step 4: both tokens issued again | Changing only the policy restored the paused capability |
| Both tokens denied in any step | Another policy is blocking the agent; see Section 7 |

## Section 7 - If something goes wrong

| What you see | What to do |
|---|---|
| Agent-Token-ARM fails at Step 1 | The agent cannot get an Azure management token in this tenant. Record the error and stop (Check 1). |
| Both agent tokens denied | Use case 02's policy or another policy blocks all resources. Find it on the sign-in event (Checks 2 and 3). |
| Azure management event shows **Report-only: Not applied** | Check the policy includes Policy-Digest-Agent (not the blueprint) and targets Azure Resource Manager |
| Graph is denied at Step 3 | Another policy is involved; this policy cannot target Graph. Check the Graph event's Conditional Access tab. |
| Step 3 still issues the Azure management token | Confirm the policy is **On** and wait a few minutes; the script requests a new token each time |
| 53003 without the named policy on the event | Another policy blocked the request |
| Errors 7000215, 82008 or 90002 | Credential or ID problems. Use the error table in [Setup 4.7](00-Setup.md#47-test-that-the-agent-identity-works). |

## Section 8 - Put everything back

1. Alex sets `Contoso-CA-Agent-ARM-Pause` to **Off**, or deletes it with approval. Its sign-in history remains.
2. If you turned off use case 02's policy only for this use case, leave it off; use case 02 ends with it off.
3. Nestor confirms a new run issues all three tokens.
4. Keep the blueprint secret for the later use cases. Nothing else was changed: no roles, permissions, users or other agents.

## Section 9 - Outcome

| Action | Result |
|---|---|
| Run the token test with no policy | "This agent gets access to Azure management and Microsoft Graph on its own." |
| Turn the policy on in Report-only | "The policy would stop the Azure management access only. Nothing is blocked yet." |
| Turn the policy On | "The agent can no longer get Azure management access, but it still gets Microsoft Graph access. No person was affected." |
| Turn the policy Off | "Azure management access came back by changing one policy. The agent and its credentials were untouched." |

**Do not say:**

- that the agent could perform Azure or Graph operations. It received tokens only; it has no roles or permissions;
- that the policy controls single operations or single Azure resources. It covers the whole Azure management service group;
- that other services or other agents were tested;
- that existing tokens stop working immediately.

Record the conclusion using the [results criteria](99-Results-and-Handoff.md#03---entra003---pause-one-agent-capability).

## Section 10 - Sources

| Topic | Source |
|---|---|
| Conditional Access for agents: agents using their own identity, licensing | [Conditional Access for agents](https://learn.microsoft.com/en-us/entra/identity/conditional-access/agent-id) |
| Selecting agent identities and target resources (Select resources, All agent resources, All resources) | [Target agent identities in Conditional Access](https://learn.microsoft.com/en-us/entra/identity/conditional-access/howto-target-agent-identities) |
| Microsoft Graph cannot be targeted on its own; services covered by Windows Azure Service Management API | [Target resources](https://learn.microsoft.com/en-us/entra/identity/conditional-access/concept-conditional-access-cloud-apps) |
| Policy creation path for agents | [Secure autonomous agents with Conditional Access](https://learn.microsoft.com/en-us/entra/identity/conditional-access/policy-autonomous-agents) |
| Report-only result labels | [Report-only mode](https://learn.microsoft.com/en-us/entra/identity/conditional-access/concept-conditional-access-report-only) |
| Agent sign-in logs and the Agent type filter | [Agent sign-in and audit logs](https://learn.microsoft.com/en-us/entra/agent-id/sign-in-audit-logs-agents) |
