# Results and Handoff

This file turns what you recorded in the [tracker](98-Tracker.md) into a conclusion for each use case, explains how to present it, and lists what to decide before using a control in production.

## Section 1 - Assemble the evidence

**What this section does:** gathers the evidence for one use case in one place before anyone writes a conclusion.

Each person supplies what they recorded:

| Name | Supplies |
|---|---|
| **Nestor** | Token test CSV files, agent configuration and published versions, Copilot Studio activity |
| **Allan** | His questions and requests, with times, and the answers and citations he received |
| **Lee** | Sign-in, audit, alert and hunting results, matched to the times Allan and Nestor recorded |
| **Alex** | Policy, rule and workflow settings, with their IDs, and the final state after Section 8 of the use case |
| **Adele** | Purview settings, and an independent review of the whole package |

Save the package in the evidence library, in the use case's folder: `NN-Short-Name/<date>-<run>`, for example `02-Autonomous-Agent-Conditional-Access/20261006-01`.

**What counts as evidence:**

- A result you observed, with its time, matched to the step that caused it. A screenshot of a setting shows configuration, not a result.
- A run label echoed by the agent or the script identifies the run. It does not prove the step worked.
- A result for the named policy or rule. A failure caused by something else, such as a credential error or another policy, is not this use case's result.

Never include passwords, client secret values, access tokens or Authorization headers.

## Section 2 - Write the conclusion

**What this section does:** gives, for each use case, the conclusion that each kind of evidence supports. Find the row that matches what you observed and record its conclusion in the tracker's **Summary**. If several rows apply, record each one. If no row matches, record exactly what you saw.

Each use case's **Section 9 - Outcome** lists what not to say. Those limits apply to every conclusion below.

### 01 - Entra001 - Time-bound agent access

| Evidence | Conclusion |
|---|---|
| Package and policy configured only | Set up; no result yet |
| Assignment with an end date, and the agent is a direct group member of type **Agent identity** | Access delivered to the agent |
| An end date, but the end time has not passed | Time limit configured; expiry not yet shown |
| Expired assignment and the agent removed from the group | Time-bound access works for the agent |
| Allan's request, Lee's approval and a delivered assignment | Sponsor request and independent approval work for an agent |
| Graph check 403 before, 200 during and 403 after the approved period | The approved access controlled what the agent could do, for a limited time |
| Access removed early by an administrator | Manual removal; not an expiry result |

### 02 - Entra002 - Conditional Access for an autonomous agent

| Evidence | Conclusion |
|---|---|
| Step 1: tokens issued and agent sign-ins found | The agent signs in on its own |
| Step 2: tokens issued and **Report-only: Failure** for the named policy on the agent's events | Conditional Access evaluated the agent and would have blocked it |
| Step 3: agent tokens denied with 53003 by the named policy; blueprint token still issued | The policy blocked this agent's new tokens |
| Step 4: tokens issued again | Access restored by changing only the policy |
| A Graph 403, a credential error or a failed blueprint token | A different failure; not a Conditional Access result |

### 03 - Entra003 - Pause one agent capability

| Evidence | Conclusion |
|---|---|
| Step 1: Graph and Azure management tokens issued | Not ready until both tokens are issued |
| Step 2: **Report-only: Failure** on the Azure management event; **Not applied** on the Graph event | The policy matches only the Azure management request |
| Step 3: Azure management denied with 53003 by the named policy; Graph still issued | One capability paused while the other kept working |
| Step 4: both tokens issued again | The paused capability restored by changing only the policy |
| Both tokens denied, or the denial not attributed to the named policy | A different or wider control; not this result |

### 04 - Entra004 - Sponsor stops an agent

| Evidence | Conclusion |
|---|---|
| Agent shows Disabled; the audit log shows Allan made the change | A sponsor stopped the agent without an administrator role |
| No agent token issued after the stop, with no credential error | The stopped agent could not get new tokens |
| Allan has no enable option, or his attempt is refused | A sponsor cannot restart the agent |
| Allan can re-enable the agent | Not shown: Allan holds owner or administrator rights. Check Setup 4.5. |
| Alex re-enables it and all tokens are issued again | The owner restarted the agent with nothing rebuilt |

### 05 - Entra005 - Sponsor continuity

| Evidence | Conclusion |
|---|---|
| Workflow history shows every enabled task succeeded for Allan | The handover came from a governed workflow |
| Sponsors changed from Allan to Megan, in the audit log at the run time | Accountability moved to Allan's manager |
| Megan received the email (only if the manager-email task was enabled) | The new sponsor was told |
| Megan sees the agent in **Agents you sponsor**; Allan does not | The self-service views follow the new sponsorship |
| Allan's account unchanged and tokens still issued | Only sponsorship changed; the agent kept working |
| Sponsor changed by hand, not by the workflow run | Manual change; not this result |

### 06 - Entra006 - Contain a high-risk agent

| Evidence | Conclusion |
|---|---|
| **Report-only: Not applied** while the agent's risk is None | The policy leaves a healthy agent alone |
| Risk High, a **Confirmed compromised** detection, and **Report-only: Failure** | The policy reacts to the agent's risk |
| Policy On: agent tokens denied with 53003 by the named policy | A high-risk agent was blocked automatically (simulated compromise) |
| Risk dismissed to None; tokens issued with the policy still On | Clearing the risk restored access without editing the policy |
| Access restored only by turning the policy off | Policy rollback; not recovery through risk |
| Tokens denied by Microsoft's own high-risk policy | A different policy; not this result |

### 07 - Entra007 - Approval-gated agent access

| Evidence | Conclusion |
|---|---|
| Approval-Demo-Agent **Pending review** with no tag | This publishing request awaited administrator review; the approval tag had not been assigned |
| Tag `Approved` with a **Custom Security** audit event at approval time | Approval with the template tagged the agent |
| **Report-only: Failure** for untagged Policy-Digest-Agent | This policy would block the tested untagged agent requests if enforced |
| **Report-only: Not applied** once tagged, with nothing else changed | The tag excludes those requests from this block policy; it does not grant access or prove approval history |
| Tag present but the policy result unchanged after waiting | Scope by tag not shown; check the policy's attribute filter |

### 08 - A365001 - Apply a standard to existing agents

| Evidence | Conclusion |
|---|---|
| AGT01-CPS tagged `Standard-A`, with an audit event at the rule's run time | The Apply template rule tagged an existing agent in one action |
| **Not applied** for untagged Policy-Digest-Agent | The policy does not reach agents without the tag |
| **Report-only: Failure** once tagged | The standard followed the tag to an existing agent, without rebuilding it |
| **Not applied** again once untagged | Removing the tag takes the agent out of scope |
| Tagged, but still **Not applied** after waiting | Scope by tag not shown; check the filter and the assignment |

### 09 - Purview001 - Existing labels and DLP still apply

| Evidence | Conclusion |
|---|---|
| Citation shows the document's sensitivity label | The agent shows existing labels on what it cites |
| Step 3: labelled content not used; general question still answered | The existing label-based DLP rule applies to this agent, for this label |
| Step 4: labelled content answered again | The change came from the DLP rule alone |
| Step 3 unchanged from Step 1 after four hours | DLP effect not shown; check scope, label and location |
| Both questions fail at Step 3 | Agent or channel problem; not a DLP result |

### 10 - Purview002 - Insider risk for agents

Record the run label, and whether each alert came from Microsoft's default policy or the optional 4B custom policy. Both are ways to show the same capability, not two separate claims: the default policy is what a customer gets out of the box; the custom policy, if used, demonstrates the same mechanism on a controllable schedule.

| Evidence | Conclusion |
|---|---|
| The agent's steps in the audit log, with the files accessed | The agent's activity is recorded |
| Activity explorer shows the label or classifier on the agent's interaction | Purview recognized the sensitive content the agent handled |
| A Risky Agents alert for the agent, matched to the test steps | Insider risk management flagged the agent's behaviour |
| The alert came from the default policy | Shown with no extra configuration, exactly as a customer would see it |
| The alert came from the 4B custom policy only | The same detection mechanism is real and investigable. The default policy did not alert for the same activity, and the reason is not established because Microsoft does not publish its inputs |
| Lee's investigation of that alert, reviewed by Adele | An agent alert can be investigated like a person's |
| No alert after the waiting period | No alert produced. Show the recorded audit and classification evidence instead. Not proof that the control failed, nor that there is no risk. |
| A DLP block, a model refusal or a user's alert only | A different outcome; not an agent alert |

### 11 - Defender001 - Agent security posture

| Evidence | Conclusion |
|---|---|
| Risk level, indicators and recommendations recorded for each Contoso agent | Defender explains how each agent is ranked |
| Counts per risk level, with **Not evaluated** kept separate | Assessment coverage measured on that date |
| **Not evaluated** agents with the reason per platform | Coverage gap explained |
| A recommendation cleared after an approved fix | The fix was applied and rechecked; the risk level may or may not change |
| Portal and `AgentsInfo` totals agree, or the difference is explained | The inventory is consistent |

### 12 - Defender002 - Show what the agent did

| Evidence | Conclusion |
|---|---|
| Agent records matched to Allan's conversation by agent, person and time | The agent's own actions can be traced for one conversation |
| Named fields in the agent records that the audit record does not have | What the agent records add to an investigation |
| A restricted conversation shows where it differed from the normal one | Where the restricted request stopped, as far as the records show |
| No agent records after the checks and waiting | Activity data not available for this agent; record the gap |

Purview audit is the existing record of the conversation. Do not describe the agent records as Contoso's first view of agent use.

### 13 - Defender003 - Stop an unsafe agent action

| Evidence | Conclusion |
|---|---|
| Audit pass (Allan, on AGT03-CPS): each tool-call scenario's action executes and a behavior is recorded | Defender evaluated the agent's tool calls, and recorded the risky ones |
| Block pass (Nestor, on AGT02-CPS), Scenarios B-E: Nestor told it was blocked with `SecurityWebhookBlocked`, the risky outcome did not happen, and a behavior is recorded | The block rule stopped the risky tool call before it ran, for this agent |
| Block pass, Scenario A: ordinary email delivered | The block affected the risky action, not the agent |
| Whether an alert appeared during the block pass | Record it either way; Microsoft does not say whether block rules suppress alerts |
| A `ContentFiltered` block | A Copilot Studio content filter result; not evidence that the Defender rule worked. Microsoft's pages disagree on whether it is recorded as a behavior |
| A refusal by the agent with no behavior recorded and no error code | Model refusal; not Defender prevention |
| No behavior in the audit pass after the checks | Scenario not shown for this agent; do not use its block-pass step as proof |

### 14 - A365002 - Withdraw a shared tool

| Evidence | Conclusion |
|---|---|
| Step 2: server registry status **Blocked** | The tenant-wide block is configured; runtime effect still needs observation |
| Step 3: tool call failed or not made; knowledge question answered; agent still published | Tool withdrawal was observed on AGT01-CPS with Nestor, without changing the agent during the test |
| Time from the block to the change | How quickly the block took effect in this tenant |
| Step 4: tool works again with no change to the agent | Unblocking restored the tool |
| Both questions fail after the block | Agent or channel problem; not this result |
| The tool still works four hours after the block | Withdrawal not shown for this tool and agent |

## Section 3 - Present the result

**What this section does:** gives the order in which to present a use case, so the audience sees both what Contoso already had and what Agent 365 adds.

1. **The concern:** the business question, from the use case's Section 1.
2. **What Contoso can already do:** the existing capability, from the use case's Section 1.
3. **The evidence:** the configuration and the observed result, with dates, from the tracker.
4. **What Agent 365 adds:** exactly what the agent-specific capability added in this run.
5. **The limits:** what the result does not show, from the use case's Section 9 **Do not say** list.

Use the words in the use case's **Section 9 - Outcome**, and only for steps that gave the expected result. If a step did not, show what you recorded and say what is still missing. Never show an image or a result from a different run as if it were this one.

## Section 4 - Decide what to use in production

**What this section does:** lists what must be decided before a control demonstrated here is used outside the lab. A lab result shows that a control works; it does not approve it for production.

| Decision | What you need |
|---|---|
| Licences and roles | The licences and the least-privileged roles for the people who will run the control |
| Scope | Which agents, people and resources the control applies to, approved by their owners |
| State | Whether a policy starts in Report-only, and who approves turning it on |
| Credentials | Certificates or federated credentials instead of client secrets for agent identity blueprints |
| Monitoring | Who reviews sign-ins, alerts and reports, and how they escalate |
| Lab-only items | Leave behind the test documents, classifiers, test inputs and temporary tools. They exist only to produce a lab result. |

## Section 5 - Close the exercise

**What this section does:** confirms that Contoso Labs is back in its Setup state and the evidence is kept.

1. Confirm each use case's **Section 8 - Put everything back** is complete, and record it in the tracker.
2. Confirm the agents still work: Allan gets the required answer from AGT01-CPS ([Setup 10.2](00-Setup.md#102-test-the-agent)), and Nestor's token test issues all three tokens ([Setup 4.7](00-Setup.md#47-test-that-the-agent-identity-works)).
3. Delete or rotate the blueprint secret from [Setup 4.3](00-Setup.md#43-add-a-temporary-credential-to-the-blueprint) when you no longer need it.
4. Lee keeps alerts and records under Contoso's retention rules. Dismissing an alert is an investigation action, not a way to undo a test.
5. Adele reviews the evidence library and the tracker, and records her review date in the tracker's **Summary**.
6. Keep the filled-in tracker and the evidence private. Publish only the blank guide.
