# 05 - Entra005 - Sponsor continuity

This use case shows that when an agent's sponsor leaves or changes role, a governed workflow hands the agent to someone accountable, tells the right people, and records it.

Alex runs a Lifecycle Workflow for **Allan**, the sponsor of **Policy-Digest-Agent**, as if Allan were leaving. The workflow makes Allan's manager, **Megan**, the agent's sponsor, removes Allan, and emails Megan. Nothing else about Allan's account changes, and the agent keeps working.

## Section 1 - Why it matters

**Contoso's concern:** every agent needs an accountable business sponsor. When that person leaves or changes role, the agent must not be left with nobody responsible, and the new sponsor must know they are now accountable.

**What Contoso can already do:** Lifecycle Workflows automate joiner, mover and leaver tasks for **people**, such as removing groups or licences. Microsoft Entra also **automatically reassigns** an agent's sponsorship to the sponsor's manager when the sponsor leaves the organization.

**What Agent 365 adds:** Lifecycle Workflow tasks built for agent sponsors, available in two ready-made templates: **Offboard agent sponsors** (shown under this name in the Microsoft Entra admin center; documented on Microsoft Learn as "Transition agent sponsorships when a sponsor leaves") and **Transition agent sponsorships when a sponsor changes roles**. Compared with the automatic reassignment, the workflow:

- also covers people who **change role**, not only those who leave;
- **emails** the manager and the agent's other sponsors about the change;
- runs at a time tied to the HR leave date or attribute change you choose;
- leaves a **workflow history** showing each task's result.

**Licensing:** the sponsor tasks and templates need Microsoft 365 E7, or Microsoft Agent 365 with at least Microsoft Entra ID P1 or Microsoft 365 E3 ([sponsor tasks](https://learn.microsoft.com/en-us/entra/id-governance/agent-sponsor-tasks)). Microsoft's [licensing table](https://learn.microsoft.com/en-us/entra/id-governance/licensing-fundamentals) also lists Lifecycle Workflows under Microsoft Entra ID Governance; confirm your tenant has what it lists.

| This use case shows, when observed | It does not show |
|---|---|
| A workflow run moved the agent's sponsorship from Allan to Megan and emailed her | That an HR system triggers the workflow at the right time. This run is started by hand (on demand). |
| Allan's account was not otherwise changed, because the workflow contains only sponsor tasks | That Megan reviewed or approved the agent's access. That is a separate decision. |
| The agent kept working while accountability changed | That Contoso's production leaver workflows already include these tasks |

## Section 2 - People and assets

| Name | Role in this use case | Microsoft access needed |
|---|---|---|
| **Alex** | Builds and runs the workflow; checks the sponsors before and after; exports the workflow history | Lifecycle Workflows Administrator |
| **Allan** | The sponsor the workflow runs for | None. He is the sponsor from [Setup 4.4](00-Setup.md#44-create-the-agent-identity-from-the-blueprint). |
| **Megan** | Allan's manager; receives the email and becomes the sponsor | None. A mailbox; she is Allan's manager from [Setup 2.2](00-Setup.md#22-create-or-map-the-accounts). |
| **Nestor** | Runs the token test before and after, to show the agent keeps working | Access to the blueprint secret ([Setup 4.3](00-Setup.md#43-add-a-temporary-credential-to-the-blueprint)) |
| **Lee** | Records the sponsor changes in the audit log | Reports Reader |
| **Adele** | Reviews the evidence independently | Read access to the evidence library |

| Asset | Built in | Used for |
|---|---|---|
| **Policy-Digest-Agent**, sponsor Allan | [Setup Section 4](00-Setup.md#section-4---the-autonomous-agent-identity) | The agent whose sponsorship moves |
| Allan's manager set to Megan | [Setup 2.2](00-Setup.md#22-create-or-map-the-accounts) | The transfer task needs it |
| Token test script `Contoso-Agent-Token-Test.ps1` | [Setup 4.6](00-Setup.md#46-create-the-token-test-script) | Shows the agent keeps working |
| Workflow `Contoso-Agent-Sponsor-Handover` | Created in [4.1](#41-create-the-sponsor-only-workflow) | The control demonstrated |
| Evidence folder | [Setup 5.3](00-Setup.md#53-create-the-evidence-library) | `05-Sponsor-Continuity/<date>-<run>` |

### Workflow at a glance

```text
Alex runs Contoso-Agent-Sponsor-Handover on demand for Allan
(template: Offboard agent sponsors; this PoC enables only Task 3)
                          |
   Task 1  Email Megan (manager) about sponsorship changes [disabled by default - see 4.1]
   Task 2  Email co-sponsors (none in this lab) [disabled by default - see 4.1]
   Task 3  Transfer agent sponsorships to manager
                          |
                          v
   Policy-Digest-Agent sponsors:  before Allan ->  after Megan*
                          |
        +-----------------+------------------+
        |                                    |
   Megan: agent now listed under       Allan: account unchanged;
   Agents you sponsor; email also      agent no longer listed
   received if Task 1 was enabled*     for him*
                          |
   Nestor: token test still issues tokens*

* Expected results to confirm and record.
```

## Section 3 - Before you start

| Check | Who | Pass condition | If it fails |
|---|---|---|---|
| Check 1 - Allan is the only sponsor | Alex | **Entra ID -> Agents -> Agent identities -> Policy-Digest-Agent**, owners and sponsors page (record the actual label): Allan is the only sponsor | Restore the Setup state (Setup 4.5). If use case 04 changed it, fix it first. |
| Check 2 - Allan sponsors nothing else | Alex | Allan sponsors no other agent, or every agent he sponsors is a test agent you are happy to move | The transfer task moves **every** agent Allan sponsors. Record each one, or stop. |
| Check 3 - Allan's manager is Megan | Alex | **Entra ID -> Users -> Allan -> Properties -> Job information:** Manager is Megan | Set it as in Setup 2.2. The transfer task fails without a manager. |
| Check 4 - Megan has a mailbox | Alex | Megan has an Exchange Online licence and mailbox | Record that the email task cannot be checked |
| Check 5 - Agent works | Nestor | The token test issues all three tokens; run it with `ENTRA005-LAB-01-STEP1` and keep the CSV | Fix the agent first (Setup 4.7) |
| Check 6 - Licensing and role | Alex | Licensing from [Section 1](#section-1---why-it-matters) is confirmed; Alex has Lifecycle Workflows Administrator | Stop and record |

## Section 4 - Set up the control

### 4.1 Create the sponsor-only workflow

**Alex**, in the [Microsoft Entra admin center](https://entra.microsoft.com):

1. Open **ID Governance -> Lifecycle workflows -> Workflows -> Create workflow**.
2. Choose the template **Offboard agent sponsors** (the name shown in the Microsoft Entra admin center). It contains only the three sponsor tasks and nothing that changes the person's own account.
3. **Basics:** **Name** `Contoso-Agent-Sponsor-Handover`. **Description** `Lab: hand over agent sponsorships`. Keep the trigger the template offers (for example the leave-date attribute); it does not matter for an on-demand run.
4. **Scope:** keep the template's rule-based scope. An on-demand run ignores scope and execution conditions and processes only the users you pick.
5. **Tasks:** the template lists three tasks. For this PoC, enable only **Transfer agent sponsorships to manager** and leave **Send email to manager about sponsorship changes** and **Send email to co-sponsors about sponsor changes** unchecked. This selectively exercises just the core governance control - the sponsorship handover itself - keeping the lab run simple and focused on what this use case needs to prove. Record the labels and the enabled/disabled state shown.
6. **Review + create:** leave the schedule **off**, so the workflow never runs automatically for anyone. Select **Create**.
7. Make sure the workflow is **enabled**. A workflow must be enabled to run on demand.
8. Record the workflow ID, template, tasks and schedule state in the tracker.

If the template is not offered, check the tenant's Agent 365 licensing. Do not start from another leaver template: those include tasks that disable or delete the person's account.

If you choose to enable **Send email to co-sponsors about sponsor changes** instead, be aware it fails outright - not a quiet skip - when the agent has no co-sponsors, and because every task in this template stops the workflow on failure by default, that failure also blocks **Transfer agent sponsorships to manager** from running. Only enable it once the agent actually has co-sponsors to notify.

## Section 5 - Run it in Contoso Labs

Record every result in the [tracker](98-Tracker.md#05---entra005---sponsor-continuity).

**Step 1 - Starting point.** Alex records the agent's sponsors (Allan only). Allan and Megan each open [My Account](https://myaccount.microsoft.com) -> **Manage agents -> Agents you sponsor**. Expected: Allan sees Policy-Digest-Agent; Megan does not. (Nestor's token test ran in Check 5.)

**Step 2 - Run the workflow for Allan.** Alex opens **Contoso-Agent-Sponsor-Handover -> Run on demand -> Add users**, selects **Allan**, selects **Add**, then **Run workflow**. He records the time.

**Step 3 - Check the handover.** When the run finishes, Alex opens the workflow's history and checks that every enabled task succeeded for Allan. Then he rechecks the agent's sponsors.

| Expected | What counts |
|---|---|
| Every enabled task shows success in the workflow history. Sponsors: **Megan** only; Allan removed. | The change came from the workflow run, shown in its history. A sponsor changed by hand is not this result. |

**Step 4 - Check the email (only if you enabled the manager-email task).** Megan checks her mailbox. Expected: an email about the sponsorship change. Record the subject and time; check junk mail if it is not in the inbox. If you left this task disabled per Section 4.1, skip this step.

**Step 5 - Check the self-service views.** Megan and Allan reopen **Manage agents -> Agents you sponsor**. Expected: Megan sees Policy-Digest-Agent; Allan no longer does.

**Step 6 - Check Allan's account is unchanged.** Alex opens **Entra ID -> Users -> Allan**. Expected: still enabled, with the same groups and licences as before.

**Step 7 - The agent keeps working.** Nestor runs the token test with `ENTRA005-LAB-01-STEP7`. Expected: all three steps **TOKEN ISSUED**.

**Evidence:**

1. **Alex** exports or captures the workflow history for the run: each task and its status. Viewing workflow history needs Lifecycle Workflows Administrator.
2. **Lee** opens **Entra ID -> Monitoring & health -> Audit logs**, filtered by time and target **Policy-Digest-Agent**, and records the sponsor changes made during the run: activity names and who or what made them.

## Section 6 - What you should see

| Evidence | What it supports |
|---|---|
| Workflow history: every enabled task succeeded for Allan | The handover came from a governed workflow, with a record |
| Sponsors changed from Allan to Megan, in the audit log at the time of the run | Accountability moved to Allan's manager |
| Megan received the email (if you enabled that task) | The new sponsor was told |
| Megan sees the agent; Allan does not | The self-service views follow the new sponsorship |
| Allan's account unchanged; token test still issues tokens | Only sponsorship changed; the agent kept working |

## Section 7 - If something goes wrong

| What you see | What to do |
|---|---|
| The sponsor templates are not offered | Check the tenant's Agent 365 licensing. Do not use another leaver template. |
| **Run on demand** is not available | The workflow is disabled. Enable it, keeping the schedule off. |
| The transfer task failed | Check Allan's Manager property (Check 3) |
| The transfer task shows **canceled**, not failed | An earlier task in the sequence failed and blocked it (`continueOnError` is false by default on every task in this template) - most likely **Send email to co-sponsors** if you enabled it on an agent with no co-sponsors. Disable that task (see Section 4.1) and rerun. |
| The email task failed, or no email arrived | Check Megan's mailbox, licence and junk mail. Note that a task can show **completed** in the workflow history without the email ever reaching mail flow - if in doubt, check your mail platform's message trace directly rather than relying on the task status alone. |
| Allan is still a sponsor after the run | Check the run finished and the transfer task succeeded; recheck after a few minutes |
| Other agents moved too | Allan sponsored more agents than recorded in Check 2. Restore each one's sponsors. |
| Allan's account changed | The workflow contains a task that is not a sponsor task. Restore Allan's account at once, record it, and rebuild the workflow from the sponsor template. |

## Section 8 - Put everything back

1. Alex makes Allan the sponsor of Policy-Digest-Agent again and removes Megan, adding Allan before removing Megan so the agent always has a sponsor. This returns the Setup state.
2. Leave Megan as Allan's manager; Setup set it.
3. Turn off or delete `Contoso-Agent-Sponsor-Handover`, unless you want to keep it.
4. Keep the blueprint secret for the later use cases.

## Section 9 - Outcome

| Action | Result |
|---|---|
| Run the sponsor workflow for Allan | "When Allan left, his agent was handed to his manager, Megan, by a governed workflow." |
| Megan's email and her My Account view | "Megan was told she is now accountable, and she can see and stop the agent herself." (Only if you enabled the manager-email task in Section 4.1 - see the note below if you did and no email arrived.) |
| Allan's account and the token test | "Only the sponsorship changed. Allan's account was untouched and the agent kept working." |
| Workflow history and audit log | "Every step of the handover is recorded." |

**Note on the manager-email task:** if you enable **Send email to manager about sponsorship changes** (Section 4.1) and the email does not arrive, check your mail platform's message trace directly rather than relying on the workflow's task status alone. The author's lab testing suggested that, for some workflow runs, the notification email may not be triggered in time even though the task reports success in the workflow history. That is a lab observation, not a documented behavior.

**Do not say:**

- that an HR event triggered this run. It was started by hand;
- that without this workflow the agent would be left with no sponsor. Entra already moves sponsorship to the manager when a sponsor leaves; the workflow adds notification, timing, role changes and a record;
- that Megan reviewed the agent's access.

Record the conclusion using the [results criteria](99-Results-and-Handoff.md#05---entra005---sponsor-continuity).

## Section 10 - Sources

| Topic | Source |
|---|---|
| The three sponsor tasks; mover and leaver only; licensing; Lifecycle Workflows Administrator | [Agent identity sponsor tasks](https://learn.microsoft.com/en-us/entra/id-governance/agent-sponsor-tasks) |
| Templates **Transition agent sponsorships when a sponsor leaves** and **... changes roles**; their tasks and triggers | [Lifecycle workflow templates](https://learn.microsoft.com/en-us/entra/id-governance/lifecycle-workflow-templates) |
| Run on demand: the workflow must be enabled; ignores execution conditions; up to 10 users | [Run a workflow on demand](https://learn.microsoft.com/en-us/entra/id-governance/on-demand-workflow) |
| Automatic sponsor transfer to the manager when a sponsor leaves | [Manage agent identities](https://learn.microsoft.com/en-us/entra/agent-id/manage-agent-identities-admin) |
| Lifecycle Workflows licensing | [Microsoft Entra ID Governance licensing](https://learn.microsoft.com/en-us/entra/id-governance/licensing-fundamentals) |
| Sponsor views in My Account | [Manage agents as an owner or sponsor](https://learn.microsoft.com/en-us/entra/agent-id/manage-agent-identities-end-user) |
