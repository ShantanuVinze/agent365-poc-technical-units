# 08 - A365001 - Apply a standard to existing agents

This use case shows how to bring agents that **already exist** under a new control standard, without rebuilding them, and how to keep the standard applied as agents come and go.

Two tools work together:

- **Scenario 1 - Tag existing agents in bulk.** An agent management rule with the **Apply template** action tags an existing agent, **AGT01-CPS**, with the custom security attribute `ControlStandard = Standard-A`. This is a one-time bulk action.
- **Scenario 2 - Let the standard follow the tag.** A Conditional Access policy targets every agent tagged `Standard-A`. Shown with **Policy-Digest-Agent**: the policy ignores it while it is untagged, applies to it as soon as it is tagged, and stops applying when the tag is removed.

The policy runs in **Report-only**.

## Section 1 - Why it matters

**Contoso's concern:** many agents were created before the organization's control standard existed. The standard must reach them without rebuilding each one, and must also cover agents created later.

**What Contoso can already do:** a Conditional Access policy can name agents one by one, but each new agent must then be added by hand.

**What Agent 365 adds:**

- **Policy templates** apply a set of policies when an agent is approved. Editing a template later affects only new approvals.
- An **agent management rule** with the **Apply template** action applies a template to agents that already exist. It is a **one-time bulk operation**, not a scheduled rule, and covers registry agents that have an agent identity.
- Conditional Access for agents can target agents **by custom security attribute**, so a policy covers every agent with the tag, including agents tagged in future.

Used together, the rule tags today's agents in one action, and the attribute-scoped policy keeps applying the standard to whatever carries the tag.

**Licensing:** Conditional Access for agents needs Microsoft 365 E7, or Microsoft Agent 365 with at least Microsoft Entra ID P1 or Microsoft 365 E3. Applying Entra policies through templates needs an **Agent 365** licence in the tenant.

| This use case shows, when observed | It does not show |
|---|---|
| The Apply template rule tagged an existing agent without rebuilding it | That the rule keeps running. It is one-time; agents created later need the rule run again, or tagging at approval (use case 07). |
| The attribute-scoped policy applies to an agent only while it carries the tag | That every template control has an attribute-based equivalent |
| The same agent moves in and out of scope by tag alone | That agents which do not sign in with their own Entra identity are covered. Microsoft notes such policies may not be enforced for them. |
| | That the policy blocks anything. It runs in Report-only. |

## Section 2 - People and assets

| Name | Role in this use case | Microsoft access needed |
|---|---|---|
| **Alex** | Builds the template and the rule; creates the policy; tags and untags Policy-Digest-Agent | AI Administrator, Attribute Assignment Administrator and Conditional Access Administrator. Attribute-filtered policies also need to read the attribute definitions (Attribute Definition Administrator, from Setup, covers this). |
| **The tenant's Global Administrator** | Builds the template with Alex, if your tenant requires it | Global Administrator, also holding Attribute Assignment Administrator |
| **Nestor** | Runs the token test | Access to the blueprint secret ([Setup 4.3](00-Setup.md#43-add-a-temporary-credential-to-the-blueprint)) |
| **Lee** | Checks the agents' attributes, the Custom Security audit log and the sign-ins | Attribute Assignment Reader, Attribute Log Reader and Reports Reader (Setup 2.3) |
| **Adele** | Reviews the evidence independently | Read access to the evidence library |

| Asset | Built in | Used for |
|---|---|---|
| Attribute `AgentGovernance / ControlStandard` (value `Standard-A`) | [Setup 3.3](00-Setup.md#33-custom-security-attribute) | The standard's tag |
| **AGT01-CPS** and its agent identity | [Setup Section 10](00-Setup.md#section-10---the-copilot-studio-agent) | Scenario 1: an existing agent tagged by the rule |
| **Policy-Digest-Agent** | [Setup Section 4](00-Setup.md#section-4---the-autonomous-agent-identity) | Scenario 2: an existing agent whose sign-in we can trigger |
| Token test script `Contoso-Agent-Token-Test.ps1` | [Setup 4.6](00-Setup.md#46-create-the-token-test-script) | Triggers Policy-Digest-Agent's sign-ins |
| Template `TPL-Contoso-Standard-A` and rule `Contoso-Apply-Standard-A` | Created in [4.1](#41-create-the-template-and-the-apply-template-rule) | Scenario 1 |
| Policy `Contoso-CA-Standard-A-Agents` | Created in [4.2](#42-create-the-attribute-scoped-policy-in-report-only) | Scenario 2 |
| Evidence folder | [Setup 5.3](00-Setup.md#53-create-the-evidence-library) | `08-Apply-a-Standard-to-Existing-Agents/<date>-<run>` |

### Workflow at a glance

```text
SCENARIO 1 - tag existing agents in bulk (one time)

Alex: rule Contoso-Apply-Standard-A, action Apply template
      template TPL-Contoso-Standard-A (ControlStandard = Standard-A)
      selected agent: AGT01-CPS -> Run
                         |
   AGT01-CPS before: no ControlStandard*
   AGT01-CPS after:  ControlStandard = Standard-A*
   Custom Security audit log shows the change*


SCENARIO 2 - the standard follows the tag

Policy Contoso-CA-Standard-A-Agents (Report-only)
   agents with ControlStandard = Standard-A
   resource: Azure Resource Manager | Block

   Policy-Digest-Agent untagged  -> "Report-only: Not applied"*
   Alex tags it Standard-A       -> "Report-only: Failure"*
   Alex removes the tag          -> "Report-only: Not applied"*

* Expected results to confirm and record.
```

## Section 3 - Before you start

| Check | Who | Pass condition | If it fails |
|---|---|---|---|
| Check 1 - The attribute exists | Alex | **Entra ID -> Custom security attributes -> AgentGovernance -> ControlStandard** exists with `Standard-A` ([Setup 3.3](00-Setup.md#33-custom-security-attribute)) | Finish Setup 3.3 |
| Check 2 - Neither agent is tagged | Lee | On **AGT01-CPS** and **Policy-Digest-Agent** (**Entra ID -> Agents -> Agent identities -> agent -> Custom security attributes**): no `ControlStandard` | Remove it, or record it and stop |
| Check 3 - AGT01-CPS is in the registry | Alex | **Microsoft 365 admin center -> Agents -> All agents** lists AGT01-CPS, and it has an agent identity ([Setup 10.4](00-Setup.md#104-record-the-agents-identity)) | The rule only covers registry agents with an agent identity. Record what is listed. |
| Check 4 - Both tokens work | Nestor | [Setup 4.7](00-Setup.md#47-test-that-the-agent-identity-works): Agent-Token-ARM showed **TOKEN ISSUED** | The policy targets Azure management; without that token Scenario 2 cannot run. Record and stop. |
| Check 5 - No other policy blocks Policy-Digest-Agent | Alex | The policies from use cases 02, 03, 06 and 07 are **Off** or in Report-only | Turn them off. An enforced policy would block the agent for another reason. |
| Check 6 - Roles and licensing | Alex | Roles in Section 2 assigned; licensing from [Section 1](#section-1---why-it-matters) confirmed | Arrange them first |

## Section 4 - Set up the control

### 4.1 Create the template and the Apply template rule

**Alex**, with the Global Administrator if required, in the [Microsoft 365 admin center](https://admin.microsoft.com):

1. Expand **Agents** and select **Settings -> Policy template -> Add a new template** (shown under this name in the admin center; some Learn docs call it "Templates").
2. **Name:** `TPL-Contoso-Standard-A`. **Description:** `Tags agents with ControlStandard = Standard-A`. Add the custom policy **Custom security attribute**: `AgentGovernance / ControlStandard`, value `Standard-A`. Select **Save template**.
3. Select **Settings -> Agent Management Rules -> Add rule**.
4. **Name:** `Contoso-Apply-Standard-A`. **Rule action:** **Apply Template**.
5. Under **Conditions**, set **Criteria** to **Display Name**, **Operator** to **is exactly**, **Value** to `AGT01-CPS`. Select **Next**, choose the template `TPL-Contoso-Standard-A`, then **Next** again to confirm the rule will target AGT01-CPS specifically.
6. Review the policies and **save** the rule. Do not run it yet.

### 4.2 Create the attribute-scoped policy in Report-only

**Alex**, in the [Microsoft Entra admin center](https://entra.microsoft.com):

1. Open **Entra ID -> Conditional Access -> Policies -> New policy**.
2. **Name:** `Contoso-CA-Standard-A-Agents`.
3. **Assignments -> Users, agents or workload identities:** under **What does this policy apply to?** select **Agents**. Under **Include**, choose **Select agent identities based on attributes**. Set **Configure** to **Yes**, choose **AgentGovernance / ControlStandard**, operator **Contains** (or **Equals**, if offered; record which), value `Standard-A`. Select **Done**.
4. **Target resources -> Include -> Select resources:** **Azure Resource Manager** (formerly shown as **Windows Azure Service Management API**).
5. **Grant:** **Block**, then **Select**.
6. **Enable policy:** **Report-only**. Select **Create**.
7. Record the policy ID and settings.

The policy names no agent. Any agent tagged `Standard-A`, existing or future, is in scope.

## Section 5 - Run it in Contoso Labs

Record every result in the [tracker](98-Tracker.md#08---a365001---apply-a-standard-to-existing-agents).

### Scenario 1 - Tag existing agents in bulk

**Step 1 - Run the rule.** Alex opens **Agents -> Settings -> Agent management rules -> Contoso-Apply-Standard-A**. In the rule details pane, he selects **AGT01-CPS** only, then runs the rule. He records the time.

**Step 2 - Check the tag.** Lee checks:

| Where | Expected |
|---|---|
| **Entra ID -> Agents -> Agent identities -> AGT01-CPS -> Custom security attributes** | `AgentGovernance / ControlStandard` = `Standard-A` |
| **Entra ID -> Monitoring & health -> Audit logs -> Custom Security** tab | An **Update attribute values assigned to a servicePrincipal** event for AGT01-CPS at the time of the run. Record who or what made the change. |

AGT01-CPS existed before the rule and was not rebuilt.

### Scenario 2 - Let the standard follow the tag

**Nestor** runs the token test from [Setup 4.6](00-Setup.md#46-create-the-token-test-script) with the run labels shown. **Lee** finds Policy-Digest-Agent's **Azure Resource Manager** sign-in event as in [use case 02, 4.2](02-Entra002-Autonomous-Agent-Conditional-Access.md#42-know-where-to-find-the-evidence) and records the result for `Contoso-CA-Standard-A-Agents`.

**Step 3 - Untagged.** Nestor runs the test with `A365001-LAB-01-STEP3`.

| Script result | Sign-in result (Azure Resource Manager event) |
|---|---|
| All three steps **TOKEN ISSUED** | **Report-only: Not applied**: the agent has no tag |

**Step 4 - Tagged.** Alex opens **Entra ID -> Agents -> Agent identities -> Policy-Digest-Agent -> Custom security attributes -> Add assignment**, selects **AgentGovernance / ControlStandard**, value `Standard-A`, and saves. He records the time. Wait a few minutes, then Nestor runs the test with `A365001-LAB-01-STEP4`.

| Script result | Sign-in result (Azure Resource Manager event) |
|---|---|
| All three steps **TOKEN ISSUED** (Report-only) | **Report-only: Failure**: the standard now applies to this agent. The agent itself was not changed. |

**Step 5 - Untagged again.** Alex removes the assignment. Wait a few minutes, then Nestor runs the test with `A365001-LAB-01-STEP5`.

| Script result | Sign-in result (Azure Resource Manager event) |
|---|---|
| All three steps **TOKEN ISSUED** | **Report-only: Not applied** again |

If a step does not show the expected result, run it again after about 5 and 15 minutes with a suffix such as `-02`, and record the times.

## Section 6 - What you should see

| Evidence | What it supports |
|---|---|
| Step 2: AGT01-CPS tagged `Standard-A`, with the audit event at the rule's run time | The Apply template rule tagged an existing agent in one action |
| Step 3: **Not applied** for the untagged agent | The policy does not reach agents without the tag |
| Step 4: **Report-only: Failure** once tagged | The standard follows the tag to an existing agent, without rebuilding it |
| Step 5: **Not applied** once untagged | Removing the tag takes the agent out of scope |

## Section 7 - If something goes wrong

| What you see | What to do |
|---|---|
| **Apply template** is not offered as a rule action | Record what is offered. Check the tenant's Agent 365 licensing. |
| AGT01-CPS cannot be selected in the rule | It is not in the registry, or it has no agent identity (Check 3) |
| The attribute policy cannot be added to the template | Alex or the Global Administrator lacks Attribute Assignment Administrator |
| The rule ran but AGT01-CPS has no tag | Record the rule's run history and the template. Do not add the tag by hand and count it as the rule's result. |
| Lee cannot see the **Custom Security** tab | He needs Attribute Log Reader |
| Step 3 shows **Report-only: Failure** | Policy-Digest-Agent is already tagged (Check 2) |
| Step 4 still shows **Not applied** | Check the tag value and the policy's attribute filter; wait and run again |
| The Graph event shows the policy | It cannot: the policy targets Azure Resource Manager only. Check you are reading the right event. |

## Section 8 - Put everything back

1. Alex removes `ControlStandard` from Policy-Digest-Agent (Step 5) and from AGT01-CPS.
2. Set `Contoso-CA-Standard-A-Agents` to **Off**, or delete it with approval.
3. Delete the rule `Contoso-Apply-Standard-A` and, if no longer needed, the template `TPL-Contoso-Standard-A`.
4. The attribute stays: attributes cannot be deleted (Setup 3.3). You can deactivate it if it is no longer needed.
5. Keep the blueprint secret for the later use cases.

## Section 9 - Outcome

| Action | Result |
|---|---|
| Run the Apply template rule on an existing agent | "Our existing agent was brought under the standard in one action, without being rebuilt." |
| The policy on an untagged agent | "The standard does not reach agents that are not tagged." |
| Tag the agent | "As soon as the agent carried the tag, the standard applied to it. Nothing else about it changed." |
| Remove the tag | "The standard follows the tag, so any agent tagged later is covered automatically." |

**Do not say:**

- that the Apply template rule keeps running by itself. It is one-time; new agents need it run again, or tagging at approval;
- that editing a template changes agents already approved;
- that every template control has an attribute-based equivalent;
- that the policy blocks anything. It is in Report-only.

Record the conclusion using the [results criteria](99-Results-and-Handoff.md#08---a365001---apply-a-standard-to-existing-agents).

## Section 10 - Sources

| Topic | Source |
|---|---|
| Agent management rules; the **Apply template** action for existing agents: registry agents with an agent identity, not blueprints, one-time bulk operation | [Agent settings](https://learn.microsoft.com/en-us/microsoft-365/admin/manage/agent-settings) |
| Policy templates: adding a template, custom security attribute policies, Attribute Assignment Administrator, changes apply to new approvals only, Entra sign-in needed for enforcement | [Policy templates](https://learn.microsoft.com/en-us/microsoft-agent-365/admin/policy-template) |
| Selecting agent identities by custom security attribute | [Target agent identities in Conditional Access](https://learn.microsoft.com/en-us/entra/identity/conditional-access/howto-target-agent-identities) |
| Attribute-scoped policies apply to agents with matching attributes, including future ones | [Secure autonomous agents with Conditional Access](https://learn.microsoft.com/en-us/entra/identity/conditional-access/policy-autonomous-agents) |
| Azure Resource Manager, formerly Windows Azure Service Management API | [Target resources](https://learn.microsoft.com/en-us/entra/identity/conditional-access/concept-conditional-access-cloud-apps) |
| **Custom Security** audit log and Attribute Log Reader | [Manage access to custom security attributes](https://learn.microsoft.com/en-us/entra/fundamentals/custom-security-attributes-manage) |
| Report-only result labels | [Report-only mode](https://learn.microsoft.com/en-us/entra/identity/conditional-access/concept-conditional-access-report-only) |
