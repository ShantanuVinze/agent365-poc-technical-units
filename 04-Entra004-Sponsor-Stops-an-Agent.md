# 04 - Entra004 - Sponsor stops an agent

This use case shows that the business person accountable for an agent, its **sponsor**, can stop the agent themselves, without an IT ticket or an administrator role. The sponsor cannot restart it; someone with technical authority does that.

Allan, the sponsor of **Policy-Digest-Agent**, disables the agent from his own My Account page. The agent can then no longer get new tokens. Allan cannot re-enable it. Alex, the agent's owner, re-enables it, and the agent works again with nothing rebuilt.

## Section 1 - Why it matters

**Contoso's concern:** when an agent behaves unexpectedly, the person accountable for it should be able to stop it at once. Restarting it should need someone with technical authority, so a stop is not undone casually.

**What Contoso can already do:** an app or service principal can be disabled only by an administrator or one of its owners. Business owners have no self-service way to stop it.

**What Agent 365 adds:** agent identities have a **sponsor**, a business person who can stop the agents they sponsor from the self-service **My Account** page, without any administrator role. Sponsors can disable an agent, change its sponsors or soft-delete it, but **cannot re-enable or restore** it. Owners and administrators can.

**Licensing:** Microsoft 365 E7, or Microsoft Agent 365 with at least Microsoft Entra ID P1 or Microsoft 365 E3 ([identity governance licensing](https://learn.microsoft.com/en-us/entra/id-governance/licensing-fundamentals)).

| This use case shows, when observed | It does not show |
|---|---|
| A sponsor with no administrator role stops the agent getting new tokens | That tokens the agent already holds stop at once. They may keep working until they expire, usually about an hour. Microsoft Graph can reject them sooner for apps that support continuous access evaluation; this test does not measure that. |
| The same sponsor cannot restart the agent | That administrators cannot stop or restart agents in other ways |
| The owner restarts it, with no new agent or credential | How a Microsoft-managed agent, such as a Copilot Studio agent, behaves when disabled. Microsoft warns such products might fail or fall back. |

## Section 2 - People and assets

| Name | Role in this use case | Microsoft access needed |
|---|---|---|
| **Allan** | Sponsor of Policy-Digest-Agent: stops it, then tries to restart it | None. He is the sponsor from [Setup 4.4](00-Setup.md#44-create-the-agent-identity-from-the-blueprint) and must have **no** owner or administrator rights over the agent. |
| **Alex** | Owner of Policy-Digest-Agent: checks the starting state and restarts the agent | Owner of the agent (from Setup 4.4). Agent ID Administrator is an alternative. |
| **Nestor** | Runs the token test before, during and after the stop | Access to the blueprint secret ([Setup 4.3](00-Setup.md#43-add-a-temporary-credential-to-the-blueprint)) |
| **Lee** | Finds the stop and restart in the audit log, and the failed sign-ins | Reports Reader |
| **Adele** | Reviews the evidence independently | Read access to the evidence library |

| Asset | Built in | Used for |
|---|---|---|
| **Policy-Digest-Agent**, owner Alex, sponsor Allan | [Setup Section 4](00-Setup.md#section-4---the-autonomous-agent-identity) | The agent that is stopped and restarted |
| Token test script `Contoso-Agent-Token-Test.ps1` | [Setup 4.6](00-Setup.md#46-create-the-token-test-script) | Shows whether the agent can get tokens |
| My Account **Manage agents** page | Microsoft | Where Allan stops the agent and Alex restarts it |
| Evidence folder | [Setup 5.3](00-Setup.md#53-create-the-evidence-library) | `04-Sponsor-Stops-an-Agent/<date>-<run>` |

This use case creates no new control. The control is the sponsor role itself, set up in Setup.

### Workflow at a glance

```text
Step 1  Nestor: token test             -> all tokens issued*

Step 2  Allan (sponsor), My Account -> Manage agents -> Agents you sponsor
        -> Policy-Digest-Agent -> Disable
                                       -> agent shows Disabled*

Step 3  Nestor: token test             -> no agent token issued*

Step 4  Allan looks for Enable         -> not available to him*

Step 5  Alex (owner), My Account -> Manage agents -> Agents you own
        -> Policy-Digest-Agent -> Enable
                                       -> agent shows Enabled*

Step 6  Nestor: token test             -> all tokens issued again*

Lee records the stop and restart in the audit log
and the failed sign-ins from Step 3.

* Expected results to confirm and record.
```

## Section 3 - Before you start

| Check | Who | Pass condition | If it fails |
|---|---|---|---|
| Check 1 - Agent works | Nestor | [Setup 4.7](00-Setup.md#47-test-that-the-agent-identity-works) passed: all three steps **TOKEN ISSUED** | Finish Setup Section 4 first |
| Check 2 - Allan is only a sponsor | Alex | **Entra ID -> Agents -> Agent identities -> Policy-Digest-Agent**, on its owners and sponsors page (record the actual label): Allan is a sponsor, assigned directly, not through a group. Allan is **not** an owner of the agent or its blueprint, and holds none of these roles: Agent ID Administrator, Cloud Application Administrator, Application Administrator, Global Administrator. | Fix it first. If Allan has any of these rights, the restart test proves nothing. |
| Check 3 - Alex can restart it | Alex | Alex is an owner of Policy-Digest-Agent | Fix it before Step 2. Never disable the agent without a confirmed way back. |
| Check 4 - No policy blocks the agent | Alex | The policies from use cases 02 and 03 are **Off**, and no other enforced policy applies to Policy-Digest-Agent, including Microsoft-managed ones ([Setup 4.5](00-Setup.md#45-check-the-agent-identitys-settings)) | Turn them off or record them. A blocking policy would hide the effect of the stop. |
| Check 5 - Allan can see the agent | Allan | In [My Account](https://myaccount.microsoft.com), **Manage agents -> Agents you sponsor** lists Policy-Digest-Agent as enabled. If **Manage agents** is missing, select **Use new version** in the banner. | Check Allan's sponsorship (Check 2) and that he is signed in with his own account |

## Section 4 - Set up the control

Nothing to configure. The control is the sponsor role, which Setup already gave to Allan. Checks 2 and 3 confirm the separation between Allan (can stop) and Alex (can restart).

## Section 5 - Run it in Contoso Labs

**Nestor** runs the token test from [Setup 4.6](00-Setup.md#46-create-the-token-test-script) with the run label shown, and saves the CSV in the evidence folder. Record every result in the [tracker](98-Tracker.md#04---entra004---sponsor-stops-an-agent).

**Step 1 - Starting point.** Nestor runs the test with `ENTRA004-LAB-01-STEP1`. Expected: Blueprint-Token, Agent-Token-Graph and Agent-Token-ARM all **TOKEN ISSUED**.

**Step 2 - Allan stops the agent.** Allan, in [My Account](https://myaccount.microsoft.com) with his own account:

1. Selects **Manage agents -> Agents you sponsor -> Policy-Digest-Agent**.
2. Selects **Disable** and confirms.
3. Records the time.

Expected: the agent shows as **Disabled** in My Account and in **Entra ID -> Agents -> Agent identities**.

Do not use **Delete**. A sponsor can soft-delete an agent, but only an owner or administrator can restore it.

**Step 3 - The agent cannot get tokens.** Nestor runs the test with `ENTRA004-LAB-01-STEP3`.

| Expected | What counts |
|---|---|
| No agent token is issued. Either **Blueprint-Token** fails and both agent-token rows show **NOT RUN**, or Blueprint-Token is issued and both agent tokens show **DENIED OR FAILED**. | Counts as a stop when the agent shows **Disabled** and the error is **not** a credential or ID error (7000215, 82008 or 90002). Record the error codes shown. |

If tokens are still issued, wait about 5 minutes and run again with `-02`. A policy or disable change can take a few minutes to take effect.

**Step 4 - Allan tries to restart it.** Allan reopens **Policy-Digest-Agent** in My Account and looks for **Enable**. Expected: the option is not available to him, or the attempt is refused. Record exactly what he sees.

If Allan **can** re-enable the agent, he has more than sponsor rights. Recheck Check 2; the separation is not shown.

**Step 5 - Alex restarts it.** Alex records why the agent is being restored, then, in [My Account](https://myaccount.microsoft.com) with his own account:

1. Selects **Manage agents -> Agents you own -> Policy-Digest-Agent**.
2. Selects **Enable**.

Expected: the agent shows as **Enabled**. An Agent ID Administrator who is not an owner can do the same in **Entra ID -> Agents -> Agent identities -> Policy-Digest-Agent**; record the label of the enable control.

**Step 6 - The agent works again.** Nestor runs the test with `ENTRA004-LAB-01-STEP6`. Expected: all three steps **TOKEN ISSUED**. Nothing was rebuilt and no secret changed.

**Lee** then collects:

1. **Entra ID -> Monitoring & health -> Audit logs**, filtered by time and target **Policy-Digest-Agent**: the change made by **Allan** at Step 2 and by **Alex** at Step 5. Record the activity name shown and who made each change.
2. **Entra ID -> Monitoring & health -> Sign-in logs**, filtered by **Agent type**: the failed events from Step 3 and the successful events from Step 6, matched to the CSV by **Correlation ID** or time. Record the failure reason shown.

## Section 6 - What you should see

| Evidence | What it supports |
|---|---|
| Step 2: the agent shows Disabled; the audit log shows Allan made the change | A sponsor stopped the agent without an administrator role |
| Step 3: no agent token issued, with no credential error | The stopped agent cannot get new tokens |
| Step 4: no enable option for Allan | A sponsor cannot restart the agent |
| Step 5: the agent shows Enabled; the audit log shows Alex made the change | Restarting needed the owner |
| Step 6: all tokens issued again | The agent came back with nothing rebuilt |

## Section 7 - If something goes wrong

| What you see | What to do |
|---|---|
| Allan does not see **Manage agents** | Select **Use new version** in My Account. Check he is a sponsor of the agent and signed in as himself. |
| **Disable** is missing for Allan | Record what he sees. Check his sponsorship is direct, not through a group. |
| Tokens still issued after Step 2 | Check the agent shows Disabled. Run again after about 5 minutes. |
| Step 3 shows 7000215, 82008 or 90002 | A credential or ID problem, not a stop. Fix it using [Setup 4.7](00-Setup.md#47-test-that-the-agent-identity-works) and repeat Step 3. |
| Allan can re-enable the agent | He has owner or administrator rights. Fix Check 2 and repeat from Step 1. |
| Alex cannot re-enable the agent | Check he is an owner. Do not recreate the agent. An Agent ID Administrator can enable it from the admin center. |
| Tokens not issued after Step 5 | Check the agent shows Enabled, run again after about 5 minutes and look for a blocking policy on the sign-in event |

## Section 8 - Put everything back

1. Confirm Policy-Digest-Agent shows **Enabled** and Step 6 issued all tokens.
2. Leave Allan as sponsor and Alex as owner. Use case 05 needs Allan as sponsor.
3. Keep the blueprint secret for the later use cases.

## Section 9 - Outcome

| Action | Result |
|---|---|
| Allan disables the agent from My Account | "The business sponsor stopped the agent himself, with no administrator role and no IT ticket." |
| The token test fails after the stop | "Once stopped, the agent could not get new access." |
| Allan looks for a restart option | "The sponsor cannot restart it. Restarting needs technical authority." |
| Alex re-enables the agent | "The owner restarted it after review. Nothing was rebuilt and no credential changed." |

**Do not say:**

- that the stop cut off tokens the agent already held;
- that disabling a Copilot Studio agent stops its chat in a particular way; that was not tested;
- that administrators cannot stop or restart agents another way.

Record the conclusion using the [results criteria](99-Results-and-Handoff.md#04---entra004---sponsor-stops-an-agent).

## Section 10 - Sources

| Topic | Source |
|---|---|
| Sponsor rights: disable, change sponsors, soft-delete; cannot re-enable or restore. Owner rights include re-enabling. | [Agent owners, sponsors and managers](https://learn.microsoft.com/en-us/entra/agent-id/agent-owners-sponsors-managers) |
| My Account **Manage agents**: **Disable agent**, **Enable agent**, sponsors cannot re-enable, the **Use new version** banner | [Manage agents as an owner or sponsor](https://learn.microsoft.com/en-us/entra/agent-id/manage-agent-identities-end-user) |
| Administrator disable in **Entra ID -> Agents -> Agent identities**; Agent ID Administrator; Microsoft products that might fail when an identity is disabled | [Disable agent identities](https://learn.microsoft.com/en-us/entra/agent-id/disable-agent-identities) |
| Continuous access evaluation for workload identities; disabling as a revocation event for Microsoft Graph | [Continuous access evaluation for workload identities](https://learn.microsoft.com/en-us/entra/identity/conditional-access/concept-continuous-access-evaluation-workload) |
| Agent audit and sign-in logs | [Agent sign-in and audit logs](https://learn.microsoft.com/en-us/entra/agent-id/sign-in-audit-logs-agents) |
| Licensing for agent identity governance | [Microsoft Entra ID Governance licensing](https://learn.microsoft.com/en-us/entra/id-governance/licensing-fundamentals) |
