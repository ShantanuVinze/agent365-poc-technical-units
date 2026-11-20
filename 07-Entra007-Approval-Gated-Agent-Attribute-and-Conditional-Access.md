# 07 - Entra007 - Approval-gated agent access

This use case shows how **administrator approval** through the Microsoft 365 admin center's publishing workflow can tag an agent as approved. A separate Conditional Access test shows, in **Report-only**, how that tag excludes an autonomous agent from a block policy.

The approval tag is a policy selection attribute. The policy checks the tag; it does not check the approval history. Scenario 1 demonstrates tagging at approval, and Scenario 2 demonstrates policy selection on a different agent.

It has two scenarios:

- **Scenario 1 - Approval tags the agent.** Nestor submits a new agent for approval. Alex approves it with a policy template, and the template gives the agent the custom security attribute `AgentApprovalStatus = Approved`.
- **Scenario 2 - The policy separates tagged from untagged agents.** An "allow only approved agents" policy, in **Report-only**, shows that it would block requests from an agent without the tag and excludes those requests once the agent carries the tag.

The policy stays in **Report-only** throughout. If enforced, it would block matching requests made with untagged agent identities. A tagged agent is excluded from this policy; other policies and permissions still determine its access.

## Section 1 - Why it matters

**Contoso's concern:** agents should receive administrator review before organizational distribution. Contoso wants approval to assign a tag, and a policy to identify autonomous agent requests that lack that tag.

**What Contoso can already do:** Conditional Access can target people by group and apps by name. Conditional Access for agents **cannot** use groups to choose agents.

**What Agent 365 adds:**

- the Microsoft 365 admin center **Requests** workflow, where an administrator reviews agents submitted for organizational distribution through this publishing route;
- **policy templates** applied during approval, which can assign a **custom security attribute** to the agent;
- Conditional Access for agents that can **include all agent identities and exclude those with a given attribute**, Microsoft's documented "allow only specific agents" pattern.

| Governance requirement | How it is demonstrated in this use case |
|---|---|
| Record approval through a controlled publishing process | The template applies `AgentApprovalStatus = Approved` to Approval-Demo-Agent at approval |
| Identify requests without the approval tag | Report-only flags Policy-Digest-Agent's requests while it is untagged |
| Exclude requests carrying the approval tag from this block policy | Report-only shows **Not applied** after Alex manually assigns the same tag to Policy-Digest-Agent |

**Licensing:** Conditional Access for agents needs Microsoft 365 E7, or Microsoft Agent 365 with at least Microsoft Entra ID P1 or Microsoft 365 E3. Applying Entra policies through a template needs an **Agent 365** licence in the tenant ([policy templates](https://learn.microsoft.com/en-us/microsoft-agent-365/admin/policy-template)).

| This use case shows, when observed | It does not show |
|---|---|
| Approving an agent with the template tags it **Approved**, and the audit log shows the tag arriving at approval time | That the policy blocks anything. It stays in Report-only. |
| The policy would block an untagged agent and leave a tagged one alone | That agents acting for a signed-in user are covered. When an agent acts for a user, the user's policies apply. |
| The same tag gives the same policy result, whether approval or an administrator applied it | That agents using API keys or other non-Entra sign-ins are covered. Conditional Access never sees them. |
| | That all publishing routes require approval, or that the tag can only be assigned through approval. Scenario 2 assigns it manually. |
| | That exclusion from this policy grants access. Other policies and permissions still apply. |
| | That editing the template later changes agents already approved. Template changes apply to new approvals only. |

## Section 2 - People and assets

| Name | Role in this use case | Microsoft access needed |
|---|---|---|
| **Nestor** | Builds and submits **Approval-Demo-Agent**; runs the token test | Copilot Studio access from Setup; access to the blueprint secret ([Setup 4.3](00-Setup.md#43-add-a-temporary-credential-to-the-blueprint)) |
| **Alex** | Builds the template; approves the agent; creates the policy; tags Policy-Digest-Agent in Scenario 2 | AI Administrator, Attribute Assignment Administrator and Conditional Access Administrator |
| **The tenant's Global Administrator** | Builds the template with Alex | Global Administrator, also holding Attribute Assignment Administrator. Microsoft requires both administrators to hold it for attribute policies in templates. |
| **Lee** | Checks the agents' attributes, the Custom Security audit log and the sign-ins | Attribute Assignment Reader, Attribute Log Reader and Reports Reader (Setup 2.3) |
| **Adele** | Reviews the evidence independently | Read access to the evidence library |

Nestor builds the agent; Alex approves it. The person who builds an agent must not approve it.

| Asset | Built in | Used for |
|---|---|---|
| Attribute `AgentAttributes / AgentApprovalStatus` (values `New`, `In_Review`, `Approved`) | [Setup 3.4](00-Setup.md#34-approval-attribute) | The approval tag |
| **Policy-Digest-Agent** | [Setup Section 4](00-Setup.md#section-4---the-autonomous-agent-identity) | Scenario 2: an agent whose own sign-in we can trigger |
| Token test script `Contoso-Agent-Token-Test.ps1` | [Setup 4.6](00-Setup.md#46-create-the-token-test-script) | Triggers Policy-Digest-Agent's sign-ins |
| Power Platform environment | [Setup Section 8](00-Setup.md#section-8---power-platform-environment) | Where Approval-Demo-Agent is built |
| Template `TPL-AGT-PoC Approved agents` | Created in [4.1](#41-create-the-approval-template) | Applies the tag at approval |
| Agent **Approval-Demo-Agent** | Created in [Step 1](#scenario-1---approval-tags-the-agent) | Scenario 1: the agent that is approved |
| Policy `CA-AGT-01 Allow only approved agents` | Created in [4.2](#42-create-the-policy-in-report-only) | The control demonstrated in Scenario 2 |
| Evidence folder | [Setup 5.3](00-Setup.md#53-create-the-evidence-library) | `07-Approval-Gated-Agent-Access/<date>-<run>` |

### Workflow at a glance

```text
SCENARIO 1 - approval tags the agent

Nestor builds Approval-Demo-Agent and submits it for approval
                         |
   Microsoft 365 admin center -> Agents -> All agents -> Requests
   Approval-Demo-Agent: Pending review*
                         |
Alex approves it with template TPL-AGT-PoC Approved agents
                         |
   Before: no AgentApprovalStatus*
   After:  AgentAttributes / AgentApprovalStatus = Approved*
   Custom Security audit log: attribute set at approval time*


SCENARIO 2 - the policy separates tagged from untagged agents

CA-AGT-01 (Report-only): all agent identities, except those tagged Approved -> Block

   Policy-Digest-Agent, no tag      -> tokens issued; "Report-only: Failure"*
   Alex tags it Approved
   Policy-Digest-Agent, tagged      -> tokens issued; "Report-only: Not applied"*
   Alex removes the tag

* Expected results to confirm and record.
```

## Section 3 - Before you start

| Check | Who | Pass condition | If it fails |
|---|---|---|---|
| Check 1 - Approval is required for the tested route | Alex | In the **Microsoft 365 admin center -> Agents -> Settings**, confirm administrator review is required for the organizational publishing route used in Scenario 1. Record each setting name and value as shown. | Record the gap and stop Scenario 1 if this route does not require review. Other publishing or sharing routes are outside this test. |
| Check 2 - The attribute exists | Alex | **Entra ID -> Custom security attributes -> AgentAttributes -> AgentApprovalStatus** exists with `New`, `In_Review` and `Approved` ([Setup 3.4](00-Setup.md#34-approval-attribute)) | Finish Setup 3.4 |
| Check 3 - Roles | Alex | Alex holds AI Administrator, Attribute Assignment Administrator and Conditional Access Administrator. The Global Administrator also holds Attribute Assignment Administrator. Lee holds Attribute Assignment Reader, Attribute Log Reader and Reports Reader. | Arrange the roles. A Global Administrator does not have attribute rights by default. |
| Check 4 - Licensing | Alex | Licensing from [Section 1](#section-1---why-it-matters) is confirmed | Stop and record |
| Check 5 - No other policy blocks Policy-Digest-Agent | Alex | The policies from use cases 02, 03 and 06 are **Off** | Turn them off. They would block the agent for another reason in Scenario 2. |
| Check 6 - Policy-Digest-Agent is untagged | Lee | **Entra ID -> Agents -> Agent identities -> Policy-Digest-Agent -> Custom security attributes**: no `AgentApprovalStatus` | Remove it, or record it and stop |

## Section 4 - Set up the control

### 4.1 Create the approval template

**Alex**, with the **Global Administrator**, in the [Microsoft 365 admin center](https://admin.microsoft.com):

1. Expand **Agents** and select **Settings -> Templates -> Add a new template**.
2. **Name:** `TPL-AGT-PoC Approved agents`. **Description:** `Tags approved agents with AgentApprovalStatus = Approved`.
3. Answer the question about whether the template applies to agents with their own access, choosing the option for agents that sign in with their own Entra identity. Record the wording shown.
4. Add the custom policy **Custom security attribute**: `AgentAttributes / AgentApprovalStatus`, value `Approved`.
5. Select **Save template**.
6. Record the template and its policies. Check that you could choose the **value** `Approved`, not only the attribute. If you could not, record exactly what the policy assigns.

The template can include only policies that already exist in Entra; the attribute comes from Setup 3.4.

### 4.2 Create the policy in Report-only

**Alex**, in the [Microsoft Entra admin center](https://entra.microsoft.com):

1. Open **Entra ID -> Conditional Access -> Policies -> New policy**.
2. **Name:** `CA-AGT-01 Allow only approved agents`.
3. **Assignments -> Users, agents or workload identities:** under **What does this policy apply to?** select **Agents**. Under **Include**, select **All agent identities**.
4. Under **Exclude**, select **Select agent identities based on attributes**. Set **Configure** to **Yes**, choose **AgentApprovalStatus**, operator **Contains**, value **Approved**, then **Done**.
5. **Target resources -> Include:** **All resources (formerly 'All cloud apps')**.
6. **Grant:** **Block**, then **Select**.
7. **Enable policy:** **Report-only**. Select **Create**.
8. Record the policy ID and settings.

**Never set this policy to On in this use case.** It includes every agent identity in the tenant. In Report-only it only records what it would do. Each untagged agent's sign-in will show **Report-only: Failure**; that is reporting, not blocking.

## Section 5 - Run it in Contoso Labs

Record every result in the [tracker](98-Tracker.md#07---entra007---approval-gated-agent-access).

### Scenario 1 - Approval tags the agent

**Step 1 - Build and submit the agent.** Nestor, in [Copilot Studio](https://copilotstudio.microsoft.com), in the environment from Setup Section 8:

1. Creates an agent named `Approval-Demo-Agent` with the description `Lab agent for approval testing`. No knowledge, tools or actions are needed.
2. Publishes it, adds the **Teams and Microsoft 365 Copilot** channel and, under **Availability options**, submits it to be shown to everyone in the organization, which sends it for administrator approval. Record the labels shown. **This must go through Publish -> Channels -> Availability options, not the general Share dialog** - Share looks similar (it also offers an "Everyone in organization" choice) but only grants chat access; it never sends the agent for approval, regardless of which permission level is chosen there.
3. Records the agent's **Entra Agent ID** from **Settings -> Advanced -> Metadata**.

**Step 2 - Before approval.** Lee opens **Entra ID -> Agents -> Agent identities -> Approval-Demo-Agent -> Custom security attributes**. Expected: no `AgentApprovalStatus`.

Alex opens the **Microsoft 365 admin center -> Agents -> All agents -> Requests**. Expected: `Approval-Demo-Agent` is listed as **Pending review**.

**Step 3 - Alex approves with the template.**

1. Alex selects `Approval-Demo-Agent` in **Requests** and reviews its details: description, owner, data sources, tools and permissions.
2. Selects **Publish to store**.
3. In the wizard: selects only the `Contoso-Agent365-Lab-Users` group as the users who can install it; at the template step, selects `TPL-AGT-PoC Approved agents`; reviews the permissions. **Select Next twice more** (past the group and template steps) to reach the final review screen.
4. Selects **Publish** and records the time.

**Step 4 - After approval.** Lee checks:

| Where | Expected |
|---|---|
| **Entra ID -> Agents -> Agent identities -> Approval-Demo-Agent -> Custom security attributes** | `AgentAttributes / AgentApprovalStatus` contains `Approved` |
| **Entra ID -> Monitoring & health -> Audit logs -> Custom Security** tab | An **Update attribute values assigned to a servicePrincipal** event for Approval-Demo-Agent at, or just after, the approval time. Record who or what made the change. |

The tag alone shows the agent is tagged. The audit event at approval time shows that **approval** tagged it. A manual edit by a person would not count for Scenario 1.

### Scenario 2 - The policy separates tagged from untagged agents

Approval-Demo-Agent is a Copilot Studio agent. Its sign-ins use a blueprint Microsoft manages, so the token test cannot sign in as it. Scenario 2 uses **Policy-Digest-Agent**, whose sign-in the token test can trigger, and gives it the same tag that approval gave Approval-Demo-Agent.

**Nestor** runs the token test from [Setup 4.6](00-Setup.md#46-create-the-token-test-script) with the run labels shown. **Lee** finds the agent's sign-in events as in [use case 02, 4.2](02-Entra002-Autonomous-Agent-Conditional-Access.md#42-know-where-to-find-the-evidence) and records the result for `CA-AGT-01 Allow only approved agents`.

**Step 5 - An untagged agent.** Policy-Digest-Agent has no tag (Check 6). Nestor runs the test with `ENTRA007-LAB-01-STEP5`.

| Script result | Sign-in result |
|---|---|
| All three steps **TOKEN ISSUED**: the policy is in Report-only | `CA-AGT-01` shows **Report-only: Failure**: it would block these requests from the untagged agent if enforced |

**Step 6 - Tag it Approved.** Alex opens **Entra ID -> Agents -> Agent identities -> Policy-Digest-Agent -> Custom security attributes -> Add assignment**, selects **AgentAttributes / AgentApprovalStatus**, adds the value `Approved` and saves. He records the time. This stands in for what approval did to Approval-Demo-Agent in Step 3.

**Step 7 - A tagged agent.** Wait a few minutes, then Nestor runs the test with `ENTRA007-LAB-01-STEP7`.

| Script result | Sign-in result |
|---|---|
| All three steps **TOKEN ISSUED** | `CA-AGT-01` shows **Report-only: Not applied**: the tag excludes this agent |

If Step 7 still shows **Report-only: Failure**, wait about 5 and 15 minutes and run again with `-02`, recording the times.

**Step 8 - Remove the tag.** Alex removes `AgentApprovalStatus` from Policy-Digest-Agent, so later use cases start from the Setup state.

## Section 6 - What you should see

| Evidence | What it supports |
|---|---|
| Step 2: Approval-Demo-Agent **Pending review**, no tag | This publishing request was awaiting administrator review; the approval tag had not been assigned |
| Step 4: tag `Approved`, with the Custom Security audit event at approval time | Approval with the template tagged the agent |
| Step 5: **Report-only: Failure** for the untagged agent | This policy would block the tested untagged agent requests if enforced |
| Step 7: **Report-only: Not applied** once tagged | The tag excludes the tested agent requests from this block policy |
| Steps 5 and 7 together, with nothing else changed | The tag changes whether this policy applies; it does not establish overall access or approval history |

## Section 7 - If something goes wrong

| What you see | What to do |
|---|---|
| Approval-Demo-Agent never appears in **Requests** | The approval requirement (Check 1) is not in force for this route; record it |
| The template is not offered in the wizard | It was not saved, or the tenant lacks the Agent 365 licence for Entra policies in templates |
| The attribute policy cannot be added to the template | Alex or the Global Administrator lacks Attribute Assignment Administrator, or Setup 3.4 was not done |
| Approved, but no tag on Approval-Demo-Agent | Record the template and the audit log. Do not add the tag by hand and call it approval tagging. If the template wizard would not let you choose the option for agents with their own identity when you built it, check your tenant's Agent 365 service plan provisioning status (Entra ID Protection for agents / Entra Conditional Access for agents, under Billing -> Licenses, or via an admin script) - if either still shows Pending, the template can only be scoped to non-agentic service principals, so approving an agent identity will not tag it. |
| The tag is there, but the audit event shows a person's manual edit | Scenario 1 is not shown. Repeat with a new agent. |
| Lee cannot see the **Custom Security** tab | He needs Attribute Log Reader (Check 3) |
| Step 5 shows **Report-only: Not applied** | Policy-Digest-Agent already has the tag; see Check 6 |
| Step 7 still shows **Report-only: Failure** | Check the tag value is exactly `Approved` and the policy uses **Contains** and `Approved`; wait and run again |
| The agent is denied (53003) | Another policy is On, or `CA-AGT-01` was set On by mistake. Set it back to Report-only and see Check 5. |
| The agent was approved before the template existed | Templates apply to new approvals only. The **Apply template** action in an agent management rule can apply one to existing agents; that is a different result. |

## Section 8 - Put everything back

1. Keep `CA-AGT-01` in **Report-only**, or set it to **Off**. Never set it to **On** in this use case.
2. Make sure Policy-Digest-Agent has no `AgentApprovalStatus` (Step 8).
3. Remove Approval-Demo-Agent from the store, or delete it in Copilot Studio, if it is no longer needed.
4. Keep or delete `TPL-AGT-PoC Approved agents`. Editing it later affects new approvals only.
5. The attribute stays: attributes cannot be deleted (Setup 3.4).
6. Keep the blueprint secret for the later use cases.

## Section 9 - Outcome

| Action | Result |
|---|---|
| The agent waits in **Requests** | "This agent's organizational publishing request waited for administrator review." |
| Approval with the template | "Approving this agent with the template assigned its approval tag automatically." |
| The policy on an untagged agent | "Report-only showed that this policy would block the autonomous agent's tested requests while it lacked the tag." |
| The policy on a tagged agent | "Assigning the tag excluded the same requests from this block policy. Other policies and permissions still determine access." |

**Do not say:**

- that every publishing or sharing route requires approval, or that approval cannot be bypassed;
- that the tag proves approval history or can only be assigned by the approval workflow;
- that **Not applied** grants access or means other policies allow the request;
- that the policy blocks anything. It is in Report-only;
- that Scenario 2 used the approved Copilot Studio agent. It used Policy-Digest-Agent with the same tag;
- that agents acting for a signed-in user, or using API keys, are covered by this policy;
- that one attribute also drives Defender or Purview policies; only Entra Conditional Access was shown.

Record the conclusion using the [results criteria](99-Results-and-Handoff.md#07---entra007---approval-gated-agent-access).

## Section 10 - Sources

| Topic | Source |
|---|---|
| Requests: Pending review, Pending update, Pending activate; Publish to store; the publishing wizard with a template step | [Manage agent requests](https://learn.microsoft.com/en-us/microsoft-365/admin/manage/agent-requests) |
| Templates: Add a new template; policies must exist in Entra first; Attribute Assignment Administrator for both administrators; AI Administrator limits; Agent 365 licence; changes apply to new approvals only; policies need Entra sign-in to be enforced | [Policy templates](https://learn.microsoft.com/en-us/microsoft-agent-365/admin/policy-template) |
| Agent settings and the **Apply template** action for existing agents | [Agent settings](https://learn.microsoft.com/en-us/microsoft-365/admin/manage/agent-settings) |
| "Allow only specific agents": include all agent identities, exclude by attribute with **Contains** | [Secure autonomous agents with Conditional Access](https://learn.microsoft.com/en-us/entra/identity/conditional-access/policy-autonomous-agents) |
| Agent access patterns, licensing, no group targeting for agents | [Conditional Access for agents](https://learn.microsoft.com/en-us/entra/identity/conditional-access/agent-id) |
| Custom security attributes; Global Administrator has no attribute rights by default | [Add or deactivate custom security attributes](https://learn.microsoft.com/en-us/entra/fundamentals/custom-security-attributes-add) |
| **Custom Security** audit log, its activities and the Attribute Log Reader role | [Manage access to custom security attributes](https://learn.microsoft.com/en-us/entra/fundamentals/custom-security-attributes-manage) |
| Report-only result labels | [Report-only mode](https://learn.microsoft.com/en-us/entra/identity/conditional-access/concept-conditional-access-report-only) |
