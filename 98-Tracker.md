# Tracker

This file is where you record the real values and results as you work through [Setup](00-Setup.md) and the use cases: account names, IDs, URLs, times and what you saw. It follows the same order as the guide: a summary, then Setup, then one table for each use case.

## How to use this file

- **Keep one tracker for Contoso Labs.** Fill in the empty cells as you go. When you repeat a step, for example a `-02` run, add a row under it.
- **Keep the filled-in tracker private.** The published copy stays blank. Store your filled-in copy in the evidence library from [Setup 5.3](00-Setup.md#53-create-the-evidence-library) and do not publish it.
- **Never record secrets.** No passwords, client secret values, access tokens or Authorization headers. Record names, IDs, times and results only.
- **Every table uses the same columns:** **Ref** points to the Setup section or use-case step; **Record** says what to write down; **Who** is the person who records it.

Use these values in the **Status** column:

| Status | Meaning |
|---|---|
| Not started | Nothing done yet |
| Done | A setup item is complete |
| As expected | A step gave the expected result |
| Not as expected | A step gave a different result. Record what you saw in **Value or result**. |
| Blocked | The step cannot continue. Record why. |
| Not run | An optional step, or a use case you chose not to run |

## Summary

One row for each use case you run. Fill it in after you have applied the [results criteria](99-Results-and-Handoff.md).

| Use case | Run label | Conclusion | Reviewed by Adele (date) | Status |
|---|---|---|---|---|
| [01 - Time-bound agent access](#01---entra001---time-bound-agent-access) | | | | Not started |
| [02 - Conditional Access for an autonomous agent](#02---entra002---conditional-access-for-an-autonomous-agent) | | | | Not started |
| [03 - Pause one agent capability](#03---entra003---pause-one-agent-capability) | | | | Not started |
| [04 - Sponsor stops an agent](#04---entra004---sponsor-stops-an-agent) | | | | Not started |
| [05 - Sponsor continuity](#05---entra005---sponsor-continuity) | | | | Not started |
| [06 - Contain a high-risk agent](#06---entra006---contain-a-high-risk-agent) | | | | Not started |
| [07 - Approval-gated agent access](#07---entra007---approval-gated-agent-access) | | | | Not started |
| [08 - Apply a standard to existing agents](#08---a365001---apply-a-standard-to-existing-agents) | | | | Not started |
| [09 - Existing labels and DLP still apply](#09---purview001---existing-labels-and-dlp-still-apply) | | | | Not started |
| [10 - Insider risk for agents](#10---purview002---insider-risk-for-agents) | | | | Not started |
| [11 - Agent security posture](#11---defender001---agent-security-posture) | | | | Not started |
| [12 - Show what the agent did](#12---defender002---show-what-the-agent-did) | | | | Not started |
| [13 - Stop an unsafe agent action](#13---defender003---stop-an-unsafe-agent-action) | | | | Not started |
| [14 - Withdraw a shared tool](#14---a365002---withdraw-a-shared-tool) | | | | Not started |

## Setup

### Tenant, people and roles

| Ref | Record | Who | Value or result | Date and time | Status |
|---|---|---|---|---|---|
| Setup 1 | Tenant name | Alex | | | Not started |
| Setup 1 | Subscriptions in the tenant | Alex | | | Not started |
| Setup 1 | Licence warnings, region restrictions or preview notices, and how each was resolved | Alex | | | Not started |
| Setup 2.2 | Real account used for **Alex** | Alex | | | Not started |
| Setup 2.2 | Real account used for **Adele** | Alex | | | Not started |
| Setup 2.2 | Real account used for **Nestor** | Alex | | | Not started |
| Setup 2.2 | Real account used for **Allan** | Alex | | | Not started |
| Setup 2.2 | Real account used for **Lee** | Alex | | | Not started |
| Setup 2.2 | Real account used for **Megan**; Megan is Allan's manager | Alex | | | Not started |
| Setup 2.3 | Each role assignment: person, role, start and end date | Alex | | | Not started |

### Groups and the autonomous agent

| Ref | Record | Who | Value or result | Date and time | Status |
|---|---|---|---|---|---|
| Setup 3.1 | `Contoso-Agent365-Lab-Users` created with Alex, Nestor and Allan | Alex | | | Not started |
| Setup 3.2 | `Contoso_Finance_Allowed_Agents` Object ID; no members; added to **Agent Lab Catalog** | Alex | | | Not started |
| Setup 3.3 | **AgentGovernance / ControlStandard** with value `Standard-A` | Alex | | | Not started |
| Setup 3.4 | **AgentAttributes / AgentApprovalStatus** with `New`, `In_Review`, `Approved` | Alex | | | Not started |
| Setup 4.2 | `Contoso-Autonomous-Blueprint` App ID | Alex | | | Not started |
| Setup 4.3 | Blueprint secret: where it is stored and when it expires (never the value) | Alex | | | Not started |
| Setup 4.4 | `Policy-Digest-Agent` App ID | Alex | | | Not started |
| Setup 4.5 | Policy-Digest-Agent owners (Alex only) and sponsors (Allan only) | Alex | | | Not started |
| Setup 4.5 | Microsoft-managed Conditional Access policies and their state, including **Block all high risk agents from accessing all resources (Preview)** | Alex | | | Not started |
| Setup 4.7 | Token test `SETUP-LAB-01`: result of Blueprint-Token, Agent-Token-Graph and Agent-Token-ARM | Nestor | | | Not started |
| Setup 4.7 | Matching sign-in events: time, name, status and resource | Lee | | | Not started |

### SharePoint, documents, labels and audit

| Ref | Record | Who | Value or result | Date and time | Status |
|---|---|---|---|---|---|
| Setup 5.1 | Site URL; Adele is a member; Nestor and Allan are visitors | Alex | | | Not started |
| Setup 5.2 | Knowledge folder URL | Alex | | | Not started |
| Setup 5.3 | Evidence library URL; only Alex, Adele and Lee can open it | Alex | | | Not started |
| Setup 6 | URL of `Contoso-Meal-General-Policy.docx`; Allan confirmed its check values | Alex, Allan | | | Not started |
| Setup 6 | URL of `Contoso-Board-Confidential.docx`; Allan confirmed its check values | Alex, Allan | | | Not started |
| Setup 6 | URL of `Contoso-Project-AgentRisk.docx`; Allan confirmed its check values | Alex, Allan | | | Not started |
| Setup 7.1 | Label name applied to `Contoso-Board-Confidential.docx`; Allan sees it in Word | Adele | | | Not started |
| Setup 7.2 | `Contoso-LabRoutingMarker`: matches the routing marker, no match on `PUBLIC-24681357`; reindex requested | Adele | | | Not started |
| Setup 7.3 | `UnifiedAuditLogIngestionEnabled` is **True**; Lee can search the audit log | Adele, Lee | | | Not started |

### Power Platform, Defender and the Copilot Studio agent

| Ref | Record | Who | Value or result | Date and time | Status |
|---|---|---|---|---|---|
| Setup 8 | Environment name, URL and ID; restricted to `Contoso-Agent365-Lab-Users` | Alex | | | Not started |
| Setup 8 | Nestor's role names; Allan's role with the ChatBotReaders privilege | Alex | | | Not started |
| Setup 9.1 | Error behavior decision and who decided; any existing threat detection provider | Alex | | | Not started |
| Setup 9.2 | Status of Agent 365, Microsoft 365 connector and Copilot Studio in the Security for AI checklist | Alex | | | Not started |
| Setup 9.4 | Prompt evidence collection setting; real-time protection rules present | Alex | | | Not started |
| Setup 9.5 | Endpoint URL | Alex | | | Not started |
| Setup 9.6 | `Contoso-Copilot-RTP-Lab` App ID | Alex | | | Not started |
| Setup 9.9 | Checks passed for the steps completed | Alex | | | Not started |
| Setup 10.1 | Knowledge option and folder URL used for AGT01-CPS; generative orchestration on | Nestor | | | Not started |
| Setup 10.2 | Test answers in the test pane: WS03-GENERAL-104, 80, 40; Lighthouse | Nestor | | | Not started |
| Setup 10.3 | Published version; channel label shown; shared with Allan only; Allan's answer | Nestor, Allan | | | Not started |
| Setup 10.4 | AGT01-CPS Entra Agent ID; agent identity display name and Object ID; owners and sponsors | Nestor, Alex | | | Not started |
| Setup 11 | Final check complete | Alex | | | Not started |

## 01 - Entra001 - Time-bound agent access

Follow [use case 01](01-Entra001-Time-Bound-Agent-Access.md).

| Ref | Record | Who | Value or result | Date and time | Status |
|---|---|---|---|---|---|
| Section 3 | Checks 1 to 5 passed | Alex | | | Not started |
| 4.1 | Scenario 1 package ID and policy name | Alex | | | Not started |
| 4.2 | Scenario 2 package ID; API permission added; **Agent Lab Catalog** is now privileged; **Who can request access** options shown | Alex, Global Administrator | | | Not started |
| 4.2 | Policy settings: All agents; Lee approves; expires after 2 hours | Alex | | | Not started |
| Step 1 | Group members before: empty | Alex | | | Not started |
| Step 2 | Assignment created for AGT01-CPS's agent identity | Alex | | | Not started |
| Step 3 | Assignment delivered with an end date; agent is a direct member, **Type: Agent identity**, matching Object ID | Alex | | | Not started |
| Step 4 | After one day: assignment expired; agent no longer a member | Alex | | | Not started |
| Step 5 | Graph check `ENTRA001-LAB-01-BEFORE`: 403 | Nestor | | | Not started |
| Step 6 | Allan's request, with **Requesting for Sponsored agent**, and its time | Allan | | | Not started |
| Step 7 | Request pending approval; no assignment yet | Alex | | | Not started |
| Step 8 | Lee's approval and its time | Lee | | | Not started |
| Step 9 | Assignment delivered with a 2-hour end time; **Organization.Read.All** shown on the agent; Graph check: 200 | Alex, Nestor | | | Not started |
| Step 10 | After the end time: Graph check 403; permission removed; times of any rechecks | Alex, Nestor | | | Not started |
| Section 8 | Everything put back | Alex | | | Not started |

## 02 - Entra002 - Conditional Access for an autonomous agent

Follow [use case 02](02-Entra002-Autonomous-Agent-Conditional-Access.md). For each step, Nestor records the token test results and Lee records the policy result on the agent's sign-in events.

| Ref | Record | Who | Value or result | Date and time | Status |
|---|---|---|---|---|---|
| Section 3 | Checks 1 to 4 passed | Alex | | | Not started |
| 4.1 | `Contoso-CA-Block-Policy-Digest-Agent` ID; one agent; All resources; Block; Report-only | Alex | | | Not started |
| Step 1 | No policy: all tokens issued; agent sign-ins succeed | Nestor, Lee | | | Not started |
| Step 2 | Report-only: tokens issued; **Report-only: Failure** on the agent's events | Nestor, Lee | | | Not started |
| Step 3 | On: agent tokens denied with 53003 by the named policy; blueprint token still issued | Nestor, Lee | | | Not started |
| Step 4 | Off: all tokens issued again | Nestor, Lee | | | Not started |
| Section 8 | Policy Off or deleted; secret kept for later use cases | Alex | | | Not started |

## 03 - Entra003 - Pause one agent capability

Follow [use case 03](03-Entra003-Pause-One-Agent-Capability.md).

| Ref | Record | Who | Value or result | Date and time | Status |
|---|---|---|---|---|---|
| Section 3 | Checks 1 to 4 passed; use case 02's policy is Off | Alex | | | Not started |
| 4.1 | `Contoso-CA-Agent-ARM-Pause` ID; one agent; Azure Resource Manager; Block | Alex | | | Not started |
| Step 1 | No policy: Graph and Azure management tokens issued | Nestor, Lee | | | Not started |
| Step 2 | Report-only: **Report-only: Failure** on the Azure management event; **Not applied** on the Graph event | Nestor, Lee | | | Not started |
| Step 3 | On: Azure management denied with 53003 by the named policy; Graph still issued | Nestor, Lee | | | Not started |
| Step 4 | Off: both tokens issued again | Nestor, Lee | | | Not started |
| Section 8 | Policy Off or deleted | Alex | | | Not started |

## 04 - Entra004 - Sponsor stops an agent

Follow [use case 04](04-Entra004-Sponsor-Stops-an-Agent.md).

| Ref | Record | Who | Value or result | Date and time | Status |
|---|---|---|---|---|---|
| Section 3 | Checks 1 to 5 passed: Allan only a sponsor; Alex an owner; no blocking policy | Alex, Allan | | | Not started |
| Step 1 | Starting point: all three tokens issued | Nestor | | | Not started |
| Step 2 | Allan disables the agent in My Account; the time | Allan | | | Not started |
| Step 3 | No agent token issued, and no credential error | Nestor | | | Not started |
| Step 4 | What Allan sees when he looks for **Enable agent** | Allan | | | Not started |
| Step 5 | Alex's reason for the restart; agent shows **Enabled** | Alex | | | Not started |
| Step 6 | All three tokens issued again | Nestor | | | Not started |
| Evidence | Audit log entries for Allan's and Alex's changes; failed and successful sign-ins | Lee | | | Not started |
| Section 8 | Agent enabled; sponsors and owners as in Setup | Alex | | | Not started |

## 05 - Entra005 - Sponsor continuity

Follow [use case 05](05-Entra005-Sponsor-Continuity.md).

| Ref | Record | Who | Value or result | Date and time | Status |
|---|---|---|---|---|---|
| Section 3 | Checks 1 to 6 passed; token test `ENTRA005-LAB-01-STEP1` issued all tokens | Alex, Nestor | | | Not started |
| 4.1 | `Contoso-Agent-Sponsor-Handover` workflow ID; template; the task labels and which are enabled; schedule state | Alex | | | Not started |
| Step 1 | Sponsors before: Allan only; what Allan and Megan see in **Agents you sponsor** | Alex, Allan, Megan | | | Not started |
| Step 2 | Workflow run on demand for Allan; the time | Alex | | | Not started |
| Step 3 | Every enabled task succeeded; sponsors now Megan | Alex | | | Not started |
| Step 4 | Megan's email: subject and time (only if the manager-email task was enabled in 4.1) | Megan | | | Not started |
| Step 5 | Megan sees the agent; Allan no longer does | Megan, Allan | | | Not started |
| Step 6 | Allan's account unchanged | Alex | | | Not started |
| Step 7 | All three tokens issued | Nestor | | | Not started |
| Evidence | Workflow history export; sponsor changes in the audit log | Alex, Lee | | | Not started |
| Section 8 | Allan is the only sponsor again (added before Megan was removed); workflow off or deleted | Alex | | | Not started |

## 06 - Entra006 - Contain a high-risk agent

Follow [use case 06](06-Entra006-Contain-a-High-Risk-Agent.md).

| Ref | Record | Who | Value or result | Date and time | Status |
|---|---|---|---|---|---|
| Section 3 | Checks 1 to 6 passed; state of Microsoft's high-risk policy and the exclusion made | Alex, Lee | | | Not started |
| Section 3 | Lee's Security Administrator window: start and end | Alex | | | Not started |
| 4.1 | `Contoso-CA-Agent-High-Risk-Block` ID; one agent; All resources; Agent risk High; Block | Alex | | | Not started |
| 4.2 | Who consented to the risk command permissions, and when | Lee | | | Not started |
| Step 1 | No policy, no risk: all tokens issued | Nestor | | | Not started |
| Step 2 | Report-only, no risk: **Report-only: Not applied** | Nestor, Lee | | | Not started |
| Step 3 | Compromise declared: the time; risk High; **Confirmed compromised** detection; **Report-only: Failure** | Lee, Nestor | | | Not started |
| Step 4 | On: agent tokens denied with 53003 by the named policy | Nestor, Lee | | | Not started |
| Step 5 | Risk dismissed: the time; risk None; tokens issued with the policy still On | Lee, Nestor | | | Not started |
| Section 8 | Policy Off or deleted; exclusion removed from Microsoft's policy; consent removed; Lee's role removed | Alex, Lee | | | Not started |

## 07 - Entra007 - Approval-gated agent access

Follow [use case 07](07-Entra007-Approval-Gated-Agent-Attribute-and-Conditional-Access.md).

| Ref | Record | Who | Value or result | Date and time | Status |
|---|---|---|---|---|---|
| Section 3 | Checks 1 to 6 passed; Policy-Digest-Agent has no `AgentApprovalStatus` | Alex, Lee | | | Not started |
| 4.1 | `TPL-AGT-PoC Approved agents` and its policies; the value `Approved` could be chosen | Alex | | | Not started |
| 4.2 | `CA-AGT-01` ID and settings; state Report-only | Alex | | | Not started |
| Step 1 | Approval-Demo-Agent submitted; its Entra Agent ID | Nestor | | | Not started |
| Step 2 | Before approval: **Pending review**; no tag | Lee | | | Not started |
| Step 3 | Approved with the template; the time | Alex | | | Not started |
| Step 4 | Tag `Approved` on the agent; **Custom Security** audit event at approval time | Lee | | | Not started |
| Step 5 | Untagged Policy-Digest-Agent: **Report-only: Failure** for the tested requests; this policy would block them if enforced | Nestor, Lee | | | Not started |
| Step 6 | Policy-Digest-Agent tagged `Approved`; the time | Alex | | | Not started |
| Step 7 | Tagged: **Report-only: Not applied**; the tag excludes the tested requests from this policy, not a grant of access | Nestor, Lee | | | Not started |
| Step 8 | Tag removed from Policy-Digest-Agent | Alex | | | Not started |
| Section 8 | `CA-AGT-01` in Report-only or Off; Policy-Digest-Agent untagged; Approval-Demo-Agent removed if no longer needed | Alex, Nestor | | | Not started |

## 08 - A365001 - Apply a standard to existing agents

Follow [use case 08](08-A365001-Apply-a-Standard-to-Existing-Agents.md).

| Ref | Record | Who | Value or result | Date and time | Status |
|---|---|---|---|---|---|
| Section 3 | Checks 1 to 6 passed; neither agent tagged; other agent policies Off or Report-only | Alex, Lee | | | Not started |
| 4.1 | `TPL-Contoso-Standard-A`; rule `Contoso-Apply-Standard-A` and its criteria options | Alex | | | Not started |
| 4.2 | `Contoso-CA-Standard-A-Agents` ID and settings; state Report-only | Alex | | | Not started |
| Step 1 | Rule run on AGT01-CPS only; the time | Alex | | | Not started |
| Step 2 | AGT01-CPS tagged `Standard-A`; **Custom Security** audit event and who made the change | Lee | | | Not started |
| Step 3 | Untagged Policy-Digest-Agent: **Not applied** on the Azure Resource Manager event | Nestor, Lee | | | Not started |
| Step 4 | Tagged `Standard-A`: **Report-only: Failure** | Alex, Nestor, Lee | | | Not started |
| Step 5 | Untagged again: **Not applied** | Alex, Nestor, Lee | | | Not started |
| Section 8 | Tag removed from both agents; policy Off or deleted; rule deleted | Alex | | | Not started |

## 09 - Purview001 - Existing labels and DLP still apply

Follow [use case 09](09-Purview001-Existing-Labels-and-DLP-Still-Apply.md).

| Ref | Record | Who | Value or result | Date and time | Status |
|---|---|---|---|---|---|
| Section 3 | Checks passed; existing DLP policies for the Microsoft 365 Copilot location | Adele | | | Not started |
| Step 1 | Before the rule: Q-LABEL answer, citation and label shown; Q-GENERAL answer | Allan | | | Not started |
| Step 2 | `Contoso-DLP-Copilot-Confidential-Label` created and on; action label shown; status | Adele | | | Not started |
| Step 3 | After the rule: Q-LABEL not answered from the document; Q-GENERAL still answered; times of any reruns | Allan | | | Not started |
| Step 4 | Rule off: Q-LABEL answered again | Adele, Allan | | | Not started |
| Evidence | Audit records for each step, with the files accessed | Lee | | | Not started |
| Section 8 | DLP rule deleted or left off | Adele | | | Not started |

## 10 - Purview002 - Insider risk for agents

Follow [the use case](10-Purview002-Insider-Risk-for-Agents.md). Record the run label, and separately record whether any result came from the default policy or the 4B custom policy.

| Ref | Record | Who | Value or result | Date and time | Status |
|---|---|---|---|---|---|
| Run label | Run label used | Adele | | | Not started |
| Section 3 | Checks passed | Adele | | | Not started |
| Section 4 | **Risky Agents (preview)** default policy: name, status, scope and detections. Nothing changed. | Adele | | | Not started |
| 4.1 | Label applied to `Contoso-Project-AgentRisk.docx`; the time | Adele | | | Not started |
| 4B (optional) | Custom policy name, scope, indicators and thresholds, if built | Adele | | | Not started |
| Step 1 | Existing Risky Agents alerts and their IDs | Lee | | | Not started |
| Step 2 | Harmless question: answer and time | Allan | | | Not started |
| Step 3 | WS03-AGENTRISK-207 and Harbor, with a citation; the time | Allan | | | Not started |
| Step 4 | Sensitive values returned, or what blocked them; the time | Allan | | | Not started |
| Step 5 (optional) | Risky prompt: whether the agent refused | Allan | | | Not started |
| 5B (optional) | Bulk documents and/or automated run used; which agent and flow | Nestor | | | Not started |
| Step 6 | Audit records; Activity explorer classification; any alert | Lee | | | Not started |
| Step 7 | Alert ID, policy it came from, contributing activity and Lee's notes, or **No alert** | Lee, Adele | | | Not started |
| Section 8 | Agent works normally; label removed; custom policy/flow decision recorded; IRM role changes recorded | Allan, Adele, Alex | | | Not started |

## 11 - Defender001 - Agent security posture

Follow [use case 11](11-Defender001-Agent-Security-Posture.md). Record the date and time of every view.

| Ref | Record | Who | Value or result | Date and time | Status |
|---|---|---|---|---|---|
| Section 3 | Checks passed | Lee | | | Not started |
| Step 1 | Columns shown in **Assets -> AI agents -> Agents** | Lee | | | Not started |
| Step 2 | For each Contoso agent: risk level, active risk indicators, recommendations, alerts | Lee | | | Not started |
| Step 3 | For each recommendation: description, evidence, remediation steps, exposed assets | Lee | | | Not started |
| Step 4 | Counts: High, Medium, Low, No known risk, Not evaluated; Not evaluated by platform | Lee | | | Not started |
| Step 5 (optional) | Fix applied, approval, and the risk level and recommendation before and after | Nestor, Lee | | | Not run |
| Step 6 | `AgentsInfo` total by platform and status; difference from Step 4 | Lee | | | Not started |
| Section 8 | Step 5 fix reversed if it affects other use cases | Nestor | | | Not run |

## 12 - Defender002 - Show what the agent did

Follow [use case 12](12-Defender002-Show-What-the-Agent-Did.md).

| Ref | Record | Who | Value or result | Date and time | Status |
|---|---|---|---|---|---|
| Section 3 | Checks passed | Lee | | | Not started |
| Step 1 | Allan's question and its time, to the minute | Allan | | | Not started |
| Step 2 | AgentId, EntraAgentId, SourceAgentId, ObservabilityId | Lee | | | Not started |
| Step 3 | ActionType and Application values found; whether CopilotInteraction is among them | Lee | | | Not started |
| Step 4 | Records matched to Allan's conversation; what they show; delay before they appeared | Lee | | | Not started |
| Step 5 | Purview audit record for the same conversation, and its fields | Lee | | | Not started |
| Step 6 | Fields in the agent records that the audit record does not have, by name | Lee | | | Not started |
| Step 7 (optional) | Restricted conversation: where it differed from Step 1 | Allan, Lee | | | Not run |
| Step 8 | Alerts involving the agent | Lee | | | Not started |

## 13 - Defender003 - Stop an unsafe agent action

Follow [use case 13](13-Defender003-Stop-an-Unsafe-Agent-Action.md).

| Ref | Record | Who | Value or result | Date and time | Status |
|---|---|---|---|---|---|
| Section 3 | Checks 1 to 5 passed; who approved the test link and the untrusted-domain mailbox, where each came from, and when | Alex | | | Not started |
| 4.1 | Email tool added to AGT03-CPS and AGT02-CPS; original instruction line; published version | Nestor | | | Not started |
| 4.2 | `Contoso-RTP-Block-AGT02-CPS`: scope and detection types | Alex | | | Not started |
| Audit pass | Scenario A (baseline): ordinary email delivered | Allan, Lee | | | Not started |
| Audit pass | Scenario B (Evasion techniques, 4 variants): behaviors recorded | Allan, Lee | | | Not started |
| Audit pass | Scenario C (Malicious content propagation): behavior recorded; any alert | Allan, Lee | | | Not started |
| Audit pass | Scenario D (Secret exfiltration, 3 variants): behaviors recorded; any alert | Allan, Lee | | | Not started |
| Audit pass | Scenario E (Unsafe email domain): behavior recorded; any alert | Allan, Lee | | | Not started |
| Audit pass | Scenario F (Jailbreak): alert, if any | Allan, Lee | | | Not started |
| Audit pass | Scenario G (XPIA): alert, if any | Allan, Lee | | | Not started |
| Scenarios H, I (optional) | Anonymized IP, Suspicious IP: safe test approach agreed, or recorded Not validated | Allan, Lee | | | Not run |
| Block pass | Scenario A repeated: still delivered | Nestor, Lee | | | Not started |
| Block pass | Scenarios B to E repeated: each blocked; behavior recorded | Nestor, Lee | | | Not started |
| Section 8 | Rule disabled or deleted; tool removed; instruction line restored; test emails deleted | Alex, Nestor | | | Not started |

## 14 - A365002 - Withdraw a shared tool

Follow [use case 14](14-A365002-Withdraw-a-Shared-Tool.md). For each question, Nestor also records whether **Activity** shows the User tool called and whether it succeeded.

| Ref | Record | Who | Value or result | Date and time | Status |
|---|---|---|---|---|---|
| Section 3 | Checks 1 to 3 passed | Alex, Nestor | | | Not started |
| 4.1 | User tool added; original instruction line; published version | Nestor | | | Not started |
| 4.2 | User server in the registry: name, type, publisher, status | Alex | | | Not started |
| Step 1 | Q-TOOL: display name, tool call succeeded; Q-KNOWLEDGE: WS03-GENERAL-104 and 80 | Nestor | | | Not started |
| Step 2 | Server blocked; status **Blocked** | Alex | | | Not started |
| Step 3 | AGT01-CPS with Nestor: Q-TOOL no name from the tool; Q-KNOWLEDGE still answered; time from the block to the change | Nestor | | | Not started |
| Step 4 | Server unblocked; Q-TOOL works again; time from the unblock to the change | Alex, Nestor | | | Not started |
| Section 8 | Server **Available**; tool removed; instruction line restored; Q-KNOWLEDGE still answered | Alex, Nestor | | | Not started |
