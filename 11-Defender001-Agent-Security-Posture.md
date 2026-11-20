# 11 - Defender001 - Agent security posture

This use case shows that Microsoft Defender can tell the security team **which agents to review first, and why**, and which agents it **could not assess** yet.

Lee opens Defender's **AI agents** inventory, records each Contoso agent's risk level, risk indicators and recommendations, and counts how many agents were assessed and how many were not. If a lab agent has a genuine recommendation, Nestor applies the fix and Lee checks the reassessment.

AI agent posture risk and the `AgentsInfo` hunting table are in **preview**.

## Section 1 - Why it matters

**Contoso's concern:** with many agents, a plain list does not tell the security team where to spend limited review time. Agents that could not be assessed must not be mistaken for safe.

**What Contoso can already do:** Microsoft Defender protects devices, identities, email and cloud apps, with an inventory and recommendations for each.

**What Agent 365 adds:** Defender's **security for AI agents**, turned on automatically when the tenant is onboarded to Agent 365. Defender lists the agents, gives each a **risk level** built from **risk indicators** (derived from the agent's configuration, runtime activity, access, tools, settings and active alerts), and lists **security recommendations**, which are calculated separately.

| Risk level | Meaning |
|---|---|
| High / Medium / Low | Active risk indicators of that combined severity |
| **No known risk** | Assessed, and no active risk indicator found |
| **Not evaluated** | Defender did not have enough information to assess the agent |

**Licensing:** the [Agent 365 service description](https://learn.microsoft.com/en-us/office365/servicedescriptions/microsoft-agent-365/microsoft-agent-365) lists "Identify and remediate agent misconfigurations and exposure risks" as requiring Microsoft 365 E7 or Agent 365; it is not included with other Microsoft 365 plans.

| This use case shows, when observed | It does not show |
|---|---|
| Defender ranks the Contoso agents by their documented risk indicators | That every indicator applies to every agent platform. Microsoft notes not all indicators apply to all platforms. |
| How many agents were assessed and how many were **Not evaluated** | That an agent with **No known risk** is safe |
| A recommendation's evidence, fix and reassessment | That applying a fix lowers the risk level at once. Recommendations and risk levels are calculated separately. |

## Section 2 - People and assets

| Name | Role in this use case | Microsoft access needed |
|---|---|---|
| **Lee** | Reviews the inventory, risk and recommendations; runs the query | Security Reader, or a Defender role that shows **Assets -> AI agents** and Advanced hunting. Record the role used. |
| **Alex** | Confirms Defender's set-up status; approves any fix | Security Administrator |
| **Nestor** | Applies an approved fix to a Copilot Studio agent | Copilot Studio access from Setup |
| **Adele** | Reviews the findings independently | Read access to the evidence library |

| Asset | Built in | Used for |
|---|---|---|
| Defender security for AI agents and the Microsoft 365 connector | [Setup Section 9](00-Setup.md#section-9---microsoft-defender) | The inventory and its data |
| **AGT01-CPS** | [Setup Section 10](00-Setup.md#section-10---the-copilot-studio-agent) | An agent to review |
| **Policy-Digest-Agent**, and **Approval-Demo-Agent** if use case 07 was run | [Setup Section 4](00-Setup.md#section-4---the-autonomous-agent-identity), use case 07 | Agents to review, if listed |
| Evidence folder | [Setup 5.3](00-Setup.md#53-create-the-evidence-library) | `11-Agent-Security-Posture/<date>-<run>` |

### Workflow at a glance

```text
Lee: Defender portal -> Assets -> AI agents -> Agents
                         |
        +----------------+-----------------+
        |                                  |
  Contoso agents                     Whole inventory
  risk level, indicators,            count by risk level:
  recommendations, alerts*           High / Medium / Low /
        |                            No known risk / Not evaluated*
        |                                  |
  Genuine recommendation?            AgentsInfo query:
  yes -> Nestor applies the fix      agents by platform and status*
  -> Lee rechecks after
     24 and 48 hours*
                         |
                 Adele reviews the evidence

* Expected results to confirm and record.
```

## Section 3 - Before you start

| Check | Who | Pass condition | If it fails |
|---|---|---|---|
| Check 1 - Defender is set up | Alex | Setup 9.9 passed: **Settings -> Security for AI -> Get started** shows **Enable** on, Agent 365 **Done** and the Microsoft 365 connector **Connected** | Finish [Setup Section 9](00-Setup.md#section-9---microsoft-defender). The inventory needs the connector. |
| Check 2 - Lee can see the inventory | Lee | **Assets -> AI agents** opens and lists agents | Arrange the role. Agents can take time to appear after onboarding. |
| Check 3 - The Contoso agents are listed | Lee | AGT01-CPS is listed; record whether Policy-Digest-Agent and Approval-Demo-Agent are. Record each agent's platform and publish status. | Record which agents are missing |

## Section 4 - Set up the control

There is nothing to build. The control is Defender's posture assessment, turned on in Setup Section 9. Do not change an agent's settings to create a finding; the only change in this use case is an optional fix for a genuine recommendation (Step 5).

## Section 5 - Run it in Contoso Labs

Record every result in the [tracker](98-Tracker.md#11---defender001---agent-security-posture). Note the date and time of every view.

**Step 1 - Set up the inventory view.** Lee opens the [Microsoft Defender portal](https://security.microsoft.com) -> **Assets -> AI agents -> Agents**. He selects **Customize columns** and shows **Risk level**, **Risk indicators**, **Recommendations** and **Active alerts**.

**Step 2 - Review the Contoso agents.** For each listed Contoso agent, Lee filters by **Agent name**, selects it, then **Open Agent page**. He records the risk level, each active risk indicator, recommendations, alerts and the tools shown, and takes a screenshot.

**Step 3 - Review the recommendations.** On each agent page, Lee opens **Security recommendations**. For each one he records the description, the evidence, the **Remediation steps** and the number of **Exposed assets**.

**Step 4 - Count coverage.** Lee clears the name filter, then filters by **Risk level** one value at a time and records the count for **High**, **Medium**, **Low**, **No known risk** and **Not evaluated**. He then filters the Not evaluated agents by **Platform** and records the count for each.

**Step 5 - Optional: apply a genuine fix.** Only if a Contoso lab agent has a real recommendation, and Alex approves: Nestor applies its **Remediation steps** to the agent and republishes it. Lee rechecks the agent at about 24 and 48 hours and records the recommendation and risk level before and after. If no lab agent has a recommendation, record that; it is a valid result. Do not weaken an agent's settings to create one.

**Step 6 - Query the inventory.** In **Hunting -> Advanced hunting**, Lee runs:

```kusto
AgentsInfo
| summarize arg_max(Timestamp, *) by AgentId
| where LifecycleStatus != "Deleted"
| summarize Agents = count() by Platform, PublishedStatus
| order by Agents desc
```

He compares the total with the portal count from Step 4 and records any difference. `AgentsInfo` holds agent configuration, not risk levels; use the portal for risk. Local agents found on devices are discovered separately and can make the counts differ.

## Section 6 - What you should see

| Evidence | What it supports |
|---|---|
| Steps 2 and 3: risk level, indicators and recommendations per agent | Defender explains why each agent is ranked as it is |
| Step 4: counts per risk level, with **Not evaluated** separate | The security team knows how much of the estate was actually assessed |
| Step 5: before and after views of a genuine fix | A recommendation can be acted on and rechecked |
| Step 6: inventory counts by platform and status | The portal and the hunting data agree, or the difference is explained |

## Section 7 - If something goes wrong

| What you see | What to do |
|---|---|
| The AI agents page is empty | Check Setup 9.9. Data can take time to appear after onboarding. |
| A Contoso agent is missing | Check its platform and whether it is registered in Agent 365 |
| Many agents are **Not evaluated** | Record them by platform. Defender assesses an indicator only when platform support and data exist. |
| High risk but no recommendations | Possible: no available recommendation matches its indicators |
| The recommendation remains after the fix | Reassessment takes time; check again later and record the time |
| The query count differs from the portal | Check deleted, draft and local agents; record the difference |

## Section 8 - Put everything back

1. If the fix in Step 5 would affect other use cases, Nestor reverses it and republishes.
2. Leave Defender's settings as Setup left them.

## Section 9 - Outcome

| Action | Result |
|---|---|
| Show an agent's risk indicators | "This is why this agent is ranked high: these are its actual indicators." |
| Show the risk-level counts | "This many agents were assessed. This many could not yet be assessed, and we do not count those as safe." |
| Show a fix and its recheck | "Defender recommended this change; we made it, and here is the reassessment." |

**Do not say:**

- that **No known risk** means safe;
- that every agent was assessed;
- that a fix always lowers the risk level;
- that every indicator applies to every agent platform.

Record the conclusion using the [results criteria](99-Results-and-Handoff.md#11---defender001---agent-security-posture).

## Section 10 - Sources

| Topic | Source |
|---|---|
| Inventory: prerequisites including the Microsoft 365 connector, **Assets -> AI agents**, columns, filters, agent page, local agents | [AI agent inventory](https://learn.microsoft.com/en-us/defender-xdr/security-for-ai/ai-agent-inventory) |
| Risk levels including No known risk and Not evaluated; risk indicators; recommendations calculated separately (preview) | [AI agent posture risk](https://learn.microsoft.com/en-us/defender-xdr/security-for-ai/ai-agent-risk-assessment) |
| Security for AI turned on with Agent 365 onboarding | [Enable security for AI agents](https://learn.microsoft.com/en-us/defender-xdr/security-for-ai/get-started-defender-security-for-ai) |
| `AgentsInfo` columns and values (preview) | [AgentsInfo table](https://learn.microsoft.com/en-us/defender-xdr/advanced-hunting-agentsinfo-table) |
