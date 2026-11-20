# 02 - Entra002 - Conditional Access for an autonomous agent

This use case shows that a Conditional Access policy can check, and block, an agent that runs on its own, with no signed-in user. The policy targets one agent, **Policy-Digest-Agent**, not its blueprint, not other agents and not any person.

You run the same token test four times: with no policy, with the policy in **Report-only**, with the policy **On**, and with the policy **Off** again. You see the agent's token request succeed, get flagged, get blocked and succeed again.

## Section 1 - Why it matters

**Contoso's concern:** autonomous agents act without a person signing in, so the controls that protect people's sign-ins do not cover them. Contoso wants to be able to stop one agent's access quickly, without touching people's access or every other agent.

**What Contoso can already do:** Conditional Access protects people's sign-ins. With Microsoft Entra Workload ID Premium, it can also cover ordinary service principals (apps).

**What Agent 365 adds:** Conditional Access treats **agent identities** as their own kind of policy subject. A policy can:

- target one agent identity, all agent identities, or every agent created from one blueprint;
- block the agent's own token requests, with Report-only mode to preview the effect first;
- leave people and other agents untouched.

**Licensing:** Microsoft 365 E7, or Microsoft Agent 365 with at least Microsoft Entra ID P1 or Microsoft 365 E3 ([Conditional Access for agents](https://learn.microsoft.com/en-us/entra/identity/conditional-access/agent-id#requirements-and-licensing)).

**How the agent signs in.** Policy-Digest-Agent first gets a **blueprint token**, an intermediate exchange token that cannot call any service. It then uses that to request its own **agent token** for a resource such as Microsoft Graph. Conditional Access evaluates that second request, made by the agent itself. That is the request this use case blocks.

| This use case shows, when observed | It does not show |
|---|---|
| Conditional Access evaluates a request made by an agent with no signed-in user | That agents acting on behalf of a user are blocked. When an agent acts for a user, the user is the subject and user policies apply. |
| A policy aimed at one agent identity blocks that agent's new tokens | That tokens issued before the policy was turned on stop working. They stay valid until they expire, usually about an hour. |
| Turning the policy off restores access, with no change to the agent or its credentials | That the agent can read any data. It has no Microsoft Graph data permissions; a Graph 403 is not a Conditional Access result. |
| Report-only shows the effect before anything is blocked | That the policy can target Microsoft Graph alone. Microsoft Graph cannot be selected as a single target; use case 03 targets one resource. |

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
| **Contoso-Autonomous-Blueprint** and **Policy-Digest-Agent** | [Setup Section 4](00-Setup.md#section-4---the-autonomous-agent-identity) | The agent the policy targets |
| Token test script `Contoso-Agent-Token-Test.ps1` | [Setup 4.6](00-Setup.md#46-create-the-token-test-script) | Requests a blueprint token, then agent tokens for Microsoft Graph and Azure Resource Manager |
| Conditional Access policy `Contoso-CA-Block-Policy-Digest-Agent` | Created in [4.1](#41-create-the-policy-in-report-only) | The control demonstrated |
| Evidence folder | [Setup 5.3](00-Setup.md#53-create-the-evidence-library) | `02-Autonomous-Agent-Conditional-Access/<date>-<run>` |

### Workflow at a glance

```text
Alex creates Contoso-CA-Block-Policy-Digest-Agent
   Applies to: Policy-Digest-Agent only | Resources: All | Grant: Block

Nestor runs Contoso-Agent-Token-Test.ps1 four times:

  Step 1  No policy     Blueprint token issued
                        Agent tokens (Graph, ARM) issued
  Step 2  Report-only   Agent tokens still issued;
                        sign-in shows "Report-only: Failure"*
  Step 3  On            Blueprint token issued;
                        agent tokens denied, error 53003*
  Step 4  Off           Agent tokens issued again*

Lee matches each run to Policy-Digest-Agent's sign-in events
and records the result for Contoso-CA-Block-Policy-Digest-Agent

* Expected results to confirm and record.
```

## Section 3 - Before you start

| Check | Who | Pass condition | If it fails |
|---|---|---|---|
| Check 1 - Agent works | Nestor | Setup 4.7 passed: all three steps of the token test show **TOKEN ISSUED** | Finish Setup Section 4 first |
| Check 2 - Licensing | Alex | Licensing from [Section 1](#section-1---why-it-matters) is confirmed | Stop and record |
| Check 3 - No other policy blocks the agent | Alex | **Entra ID -> Conditional Access -> Policies:** no enforced policy applies to Policy-Digest-Agent. Include policies created by Microsoft, such as **Block all high risk agents from accessing all resources (Preview)** ([Setup 4.5](00-Setup.md#45-check-the-agent-identitys-settings)). | Record it and agree with its owner. Another enforced policy would block the agent at Step 1 and hide this policy's effect. |
| Check 4 - Roles | Alex | Alex has Conditional Access Administrator; Lee has Reports Reader | Arrange the roles first |

## Section 4 - Set up the control

### 4.1 Create the policy in Report-only

**Alex**, in the [Microsoft Entra admin center](https://entra.microsoft.com):

1. Open **Entra ID -> Conditional Access -> Policies -> New policy**.
2. **Name:** `Contoso-CA-Block-Policy-Digest-Agent`.
3. **Assignments -> Users, agents or workload identities:** under **What does this policy apply to?** select **Agents**. Under **Include**, choose **Select individual agent identities**, open the **Agent identities** tab and select **Policy-Digest-Agent** only.
4. Do **not** select `Contoso-Autonomous-Blueprint`. A policy on a blueprint applies to every agent created from it.
5. **Target resources -> Include:** **All resources (formerly 'All cloud apps')**.
6. **Conditions:** none.
7. **Grant:** **Block**, then **Select**.
8. **Enable policy:** **Report-only**. Select **Create**.
9. Reopen the policy and record in the tracker: its ID, the single agent, All resources, Block and Report-only.

**All resources** is safe here because the policy includes only this one agent and starts in Report-only.

### 4.2 Know where to find the evidence

For every run in Section 5, Nestor's CSV file gives the time (**RequestUtc**) and an ID (**ClientRequestId**) for each request. **Lee** then:

1. Opens **Entra ID -> Monitoring & health -> Sign-in logs**. Filters **Agent type** to **Agent Identity** and sets the time range around RequestUtc. Agent events can appear on any tab; start with **Service principal sign-ins**. They are also listed under **Entra ID -> Agents -> Sign-in logs**.
2. Finds the **Policy-Digest-Agent** events for Microsoft Graph and Azure Resource Manager. Matches them by **Correlation ID** (equal to ClientRequestId in the CSV) or by time.
3. Opens each event's **Conditional Access** tab, or its **Report-only** tab while the policy is in Report-only, and records the result for `Contoso-CA-Block-Policy-Digest-Agent`.

The blueprint's own sign-in is a separate event. The policy applies to the agent's events, not the blueprint's.

## Section 5 - Run it in Contoso Labs

Each step: **Alex** sets the policy state, **Nestor** runs the token test with the run label shown and saves the CSV in the evidence folder, and **Lee** records the policy result as in [4.2](#42-know-where-to-find-the-evidence). Record every result in the [tracker](98-Tracker.md#02---entra002---conditional-access-for-an-autonomous-agent).

```powershell
.\Contoso-Agent-Token-Test.ps1 -TenantId "<TENANT-ID>" -BlueprintAppId "<BLUEPRINT-APP-ID>" -AgentIdentityAppId "<AGENT-IDENTITY-APP-ID>" -RunId "ENTRA002-LAB-01-STEP1" -CsvPath ".\ENTRA002-LAB-01-STEP1.csv"
```

**Step 1 - No policy.** The policy must not apply yet. If you already created it in 4.1, Alex sets it to **Off** first. Nestor runs the test with `ENTRA002-LAB-01-STEP1`.

| Script result | Sign-in result |
|---|---|
| Blueprint-Token, Agent-Token-Graph and Agent-Token-ARM: **TOKEN ISSUED** | Policy-Digest-Agent events succeed; the policy is not listed or shows **Off** |

**Step 2 - Report-only.** Alex sets the policy to **Report-only** (as created in 4.1). Nestor runs the test with `ENTRA002-LAB-01-STEP2`.

| Script result | Sign-in result |
|---|---|
| All three steps **TOKEN ISSUED**; nothing is blocked | Both Policy-Digest-Agent events show the policy as **Report-only: Failure**: the policy would have blocked them |

A token issued in Report-only proves nothing on its own. The **Report-only: Failure** result on the agent's events is what counts. Do not continue until you see it.

**Step 3 - On.** Only after Step 2 passes, and with approval, Alex sets the policy to **On**. Nestor runs the test with `ENTRA002-LAB-01-STEP3`.

| Script result | Sign-in result |
|---|---|
| Blueprint-Token **TOKEN ISSUED**. Agent-Token-Graph and Agent-Token-ARM **DENIED OR FAILED**, error **53003** | Both Policy-Digest-Agent events fail, with `Contoso-CA-Block-Policy-Digest-Agent` as the blocking policy |

The blueprint token is still issued, because the policy applies to the agent, not the blueprint. A failure without the named policy on the sign-in event does not count.

**Step 4 - Off.** Alex sets the policy to **Off**. Nestor runs the test with `ENTRA002-LAB-01-STEP4`.

| Script result | Sign-in result |
|---|---|
| All three steps **TOKEN ISSUED** again | Policy-Digest-Agent events succeed; the policy no longer applies |

Nothing about the agent, its secret or its permissions changed between Steps 3 and 4. Only the policy state changed.

**Timing:** a policy change can take several minutes to take effect, and it can reach one resource before another. In the author's lab testing (a single tenant, not a Microsoft-documented behavior), a token for one resource was already following the new state while the token for the other still followed the old one, more than 5 minutes after the change, and one change from Report-only to On was enforced only after 15 to 45 minutes. If a step does not show the expected result for both resources, run it again with a suffix such as `-02` after about 5, 15 and 30 minutes, and record the times. The sign-in events themselves can take 15 to 20 minutes to appear.

## Section 6 - What you should see

| Evidence | What it supports |
|---|---|
| Step 1: tokens issued, agent sign-ins succeed | The agent can sign in on its own, with no user |
| Step 2: tokens issued, **Report-only: Failure** on the agent's events | Conditional Access evaluated the agent's request and would have blocked it |
| Step 3: agent tokens denied with 53003, named policy on the agent's events; blueprint token still issued | The policy blocked this agent's new tokens, and only this agent's |
| Step 4: tokens issued again | Access came back by changing only the policy |
| A Graph 403 when the agent calls Graph | Not a Conditional Access result. The agent has no Graph data permission. |

## Section 7 - If something goes wrong

| What you see | What to do |
|---|---|
| Step 1 already shows 53003 | Another enforced policy blocks the agent. Find it on the sign-in event; see Check 3. |
| Step 2 shows **Report-only: Not applied** | The policy does not match this request. Check that Policy-Digest-Agent (not the blueprint) is included and that there are no exclusions. |
| Step 2 shows no result for the policy at all | You may be looking at the blueprint's event. Find the Policy-Digest-Agent events at the same time. |
| Step 3 still issues tokens | Confirm the policy is **On** and wait a few minutes; the script requests a new token each time |
| Step 4 still blocked | Confirm the policy is **Off** and check for another blocking policy on the sign-in event |
| Errors 7000215, 82008 or 90002 | Credential or ID problems. Use the error table in [Setup 4.7](00-Setup.md#47-test-that-the-agent-identity-works). |

## Section 8 - Put everything back

1. Alex sets `Contoso-CA-Block-Policy-Digest-Agent` to **Off**, or deletes it with approval. Its sign-in history remains.
2. Nestor confirms a new run issues all three tokens.
3. Keep the blueprint secret: use cases 03 to 08 use it. Delete it after your last use case ([Setup 4.3](00-Setup.md#43-add-a-temporary-credential-to-the-blueprint)).
4. Nothing else was changed: no permissions, users or other agents.

## Section 9 - Outcome

| Action | Result |
|---|---|
| Run the token test with no policy | "This agent signs in on its own, with no person involved." |
| Turn the policy on in Report-only | "Conditional Access saw this agent's request and showed us it would block it, before we blocked anything." |
| Turn the policy On | "The agent can no longer get new tokens. People and other agents were not affected." |
| Turn the policy Off | "Access came back by changing one policy. The agent and its credentials were untouched." |

**Do not say:**

- that existing tokens stop working immediately. They stay valid until they expire;
- that a Graph 403 proves the policy worked;
- that agents acting for a signed-in user are blocked by this policy;
- that the policy blocks Microsoft Graph alone. It blocks All resources for this agent.

Record the conclusion using the [results criteria](99-Results-and-Handoff.md#02---entra002---conditional-access-for-an-autonomous-agent).

## Section 10 - Sources

| Topic | Source |
|---|---|
| Conditional Access for agents: access patterns, blueprint targeting, the intermediate token exchange, when Conditional Access is evaluated, licensing | [Conditional Access for agents](https://learn.microsoft.com/en-us/entra/identity/conditional-access/agent-id) |
| Policy creation path for agents: Agents, agent identity picker, All resources, Block, Report-only | [Secure autonomous agents with Conditional Access](https://learn.microsoft.com/en-us/entra/identity/conditional-access/policy-autonomous-agents) |
| Report-only result labels | [Report-only mode](https://learn.microsoft.com/en-us/entra/identity/conditional-access/concept-conditional-access-report-only) |
| Agent sign-in logs, the **Agent type** filter, events across all tabs | [Agent sign-in and audit logs](https://learn.microsoft.com/en-us/entra/agent-id/sign-in-audit-logs-agents) |
| The blueprint and agent token requests | [Autonomous agent OAuth flow](https://learn.microsoft.com/en-us/entra/agent-id/agent-autonomous-app-oauth-flow) |
| Microsoft-managed policies | [Microsoft-managed policies](https://learn.microsoft.com/en-us/entra/identity/conditional-access/managed-policies) |
| Microsoft Graph cannot be a single target | [Target resources](https://learn.microsoft.com/en-us/entra/identity/conditional-access/concept-conditional-access-cloud-apps) |
