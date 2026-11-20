# 12 - Defender002 - Show what the agent did

This use case shows that Microsoft Defender can trace **what an agent did** during a conversation, not only what the person asked and what the agent answered.

Allan asks **AGT01-CPS** a fixed question. Lee finds the agent's own activity records for that conversation in Defender **Advanced hunting**, compares them with the Purview audit record Contoso already has, and writes down exactly what the agent records add.

Detecting and investigating threats to AI agents in Defender is in **preview**.

## Section 1 - Why it matters

**Contoso's concern:** chat history shows what a person asked and what the agent answered. During an investigation, the team also needs to know what the agent did: which agent and identity acted, what it used, and where something failed.

**What Contoso can already do:** Purview audit records each Copilot interaction: who asked, which agent answered, and the files accessed.

**What Agent 365 adds:** agent **observability data**. Copilot Studio, Foundry and Agent Builder agents send it to Microsoft 365 by default; with the Microsoft 365 connector, Defender stores it in Advanced hunting:

| Advanced hunting table | Used here for |
|---|---|
| `AgentsInfo` | The agent's identifiers |
| `CloudAppEvents` | Agent 365 observability data: agent actions, tool use and data access |
| `AlertInfo`, `AlertEvidence` | Alerts involving the agent, if any |

**Licensing:** the [Agent 365 service description](https://learn.microsoft.com/en-us/office365/servicedescriptions/microsoft-agent-365/microsoft-agent-365) lists "Collect Agent 365 unified agent observability logs and allow investigation and hunting for threats for agents" as requiring Microsoft 365 E7 or Agent 365; it is not included with other Microsoft 365 plans.

| This use case shows, when observed | It does not show |
|---|---|
| Agent activity records exist for Allan's conversation and can be matched to it | That every agent platform records the same fields. Other platforms need the Agent 365 SDK. |
| Which details the agent records add beyond the audit record | That Contoso had no logs before. Purview audit is the starting point. |
| | Any content the records do not actually contain. Record only what the fields show. |

## Section 2 - People and assets

| Name | Role in this use case | Microsoft access needed |
|---|---|---|
| **Allan** | Asks the fixed question and records the time | Agent access from Setup |
| **Lee** | Runs the queries, matches the records and compares them with the audit record | Advanced hunting access in Defender; Audit Reader |
| **Alex** | Confirms Defender's set-up status | Security Administrator |
| **Adele** | Reviews the comparison independently | Read access to the evidence library |

Agent records can include parts of conversations. Limit raw records to Lee and approved reviewers, and keep only what the evidence needs.

| Asset | Built in | Used for |
|---|---|---|
| Defender security for AI agents and the Microsoft 365 connector (with Microsoft 365 activities) | [Setup Section 9](00-Setup.md#section-9---microsoft-defender) | Brings the agent records into Defender |
| **AGT01-CPS** and its Entra Agent ID | [Setup Section 10](00-Setup.md#section-10---the-copilot-studio-agent) | The agent Allan asks |
| Audit logging | [Setup 7.3](00-Setup.md#73-confirm-audit-logging-is-on) | The baseline record |
| Use case 09's DLP rule (optional) | [Use case 09](09-Purview001-Existing-Labels-and-DLP-Still-Apply.md#step-2---turn-on-the-dlp-rule) | An interaction that was restricted, for comparison |
| Evidence folder | [Setup 5.3](00-Setup.md#53-create-the-evidence-library) | `12-Show-What-the-Agent-Did/<date>-<run>` |

### Workflow at a glance

```text
Step 1  Allan asks AGT01-CPS the fixed question; records the time
                         |
Agent 365 observability -> Defender Advanced hunting
                         |
Lee:
  Step 2  AgentsInfo:     the agent's identifiers
  Step 3  CloudAppEvents: which activity types exist for the agent*
  Step 4  CloudAppEvents: records in Allan's time window*
  Step 5  Purview audit:  the same conversation
                         |
  Step 6  a table of what the agent records add*
  Step 7  optional: a conversation restricted by DLP*
  Step 8  alerts involving the agent, if any
                         |
                 Adele reviews

* Expected results to confirm and record.
```

## Section 3 - Before you start

| Check | Who | Pass condition | If it fails |
|---|---|---|---|
| Check 1 - Defender is set up | Alex | Setup 9.9 passed: Security for AI **Enable** on, Agent 365 **Done**, Microsoft 365 connector **Connected** with **Microsoft 365 activities** selected | Finish [Setup Section 9](00-Setup.md#section-9---microsoft-defender). Without the connector, `CloudAppEvents` has no agent records. |
| Check 2 - Lee can hunt | Lee | **Hunting -> Advanced hunting** opens | Arrange the role |
| Check 3 - Audit works | Lee | Purview audit search returns AGT01-CPS conversations ([Setup 7.3](00-Setup.md#73-confirm-audit-logging-is-on)) | Record that the comparison lacks its baseline |

## Section 4 - Set up the control

There is nothing to build. This use case uses the data sources turned on in Setup Section 9. Do not turn on extra collection to improve the trace.

## Section 5 - Run it in Contoso Labs

Record every result in the [tracker](98-Tracker.md#12---defender002---show-what-the-agent-did). Lee runs the queries in the [Microsoft Defender portal](https://security.microsoft.com) -> **Hunting -> Advanced hunting**, with the time range covering the conversation.

### Step 1 - Allan asks the fixed question

Allan starts a **new conversation** in Microsoft 365 Copilot with **AGT01-CPS** and records the time to the minute:

```text
For DEFENDER002-STEP1, read Contoso-Meal-General-Policy.
Return its document marker and meal allowance. Cite the source.
```

Expected: **WS03-GENERAL-104** and **80**, with a citation.

### Step 2 - Find the agent's identifiers

```kusto
AgentsInfo
| summarize arg_max(Timestamp, *) by AgentId
| where Name has "AGT01-CPS"
| project AgentId, Name, Platform, EntraAgentID, SourceAgentId, ObservabilityID
```

Record every value. The field names above (`Name`, `EntraAgentID`, `ObservabilityID`, capital ID) are what the author's lab tenant returned; they are a portability consideration, not a sign the query is wrong. Microsoft's currently published schema for this table instead names them `AgentName`, `EntraAgentId` and `ObservabilityId` (lowercase `Id`). KQL identifiers and `==` comparisons are case-sensitive, so run `.show table AgentsInfo` (or check the column list in the query editor) first, and use whichever names your own tenant actually returns:

```kusto
AgentsInfo
| summarize arg_max(Timestamp, *) by AgentId
| where AgentName =~ "AGT01-CPS"
| project AgentId, AgentName, Platform, EntraAgentId, SourceAgentId, ObservabilityId
```

`ObservabilityID`/`ObservabilityId` is the identifier Microsoft uses to link the agent with its activity data. If it is shown as an object, use the identifier value inside it. If `EntraAgentID`/`EntraAgentId` differs from the Entra Agent ID recorded in [Setup 10.4](00-Setup.md#104-record-the-agents-identity), note both.

### Step 3 - See which activity types exist for the agent

Paste the identifiers from Step 2. Leave any you do not have as empty quotes; the query skips them. The query searches every column of each record, so it finds the identifier wherever it appears.

```kusto
let id1 = "<AgentId>";
let id2 = "<EntraAgentID>";
let id3 = "<SourceAgentId>";
let id4 = "<ObservabilityID>";
CloudAppEvents
| where Timestamp > ago(2d)
| extend Raw = tostring(pack_all())
| where (isnotempty(id1) and Raw contains id1)
    or (isnotempty(id2) and Raw contains id2)
    or (isnotempty(id3) and Raw contains id3)
    or (isnotempty(id4) and Raw contains id4)
| summarize Events = count(), First = min(Timestamp), Last = max(Timestamp) by ActionType, Application
| order by Events desc
```

Record each **ActionType** and **Application** found. This guide does not assume their names.

If an ActionType is **CopilotInteraction**, that is the Copilot interaction record that Purview audit also holds. Treat it as the **baseline copy**, not as an extra agent record.

### Step 4 - Find the records for Allan's conversation

Use the same identifiers, and a window of a few minutes either side of Allan's time, for example `datetime(2026-10-06T10:00:00Z)`.

**This matches by agent identifier, person and a narrow time window, not by a shared conversation, session or trace ID** - `AgentId` identifies the agent, not the conversation. In this isolated lab, with one person using one agent at a time, that is enough to find the right record; it would be ambiguous if more than one conversation with this agent were running at the same time. If your tenant's records expose a conversation, session or trace identifier, prefer matching on that instead and note which you used.

```kusto
let id1 = "<AgentId>";
let id2 = "<EntraAgentID>";
let id3 = "<SourceAgentId>";
let id4 = "<ObservabilityID>";
let start = datetime(<UTC start>);
let finish = datetime(<UTC end>);
CloudAppEvents
| where Timestamp between (start .. finish)
| extend Raw = tostring(pack_all())
| where (isnotempty(id1) and Raw contains id1)
    or (isnotempty(id2) and Raw contains id2)
    or (isnotempty(id3) and Raw contains id3)
    or (isnotempty(id4) and Raw contains id4)
| project Timestamp, ActionType, Application, AccountDisplayName, AccountObjectId, ObjectName, RawEventData
| order by Timestamp asc
```

Expand **RawEventData** for each record. Record the agent and identity, the person, the order of activities, what the agent used (knowledge source, tool or file) and the outcome, **as far as the fields show them**. Do not assume tool inputs, outputs or full prompts are included.

If nothing is returned, run it again after about 1, 4 and 24 hours, and record the delay.

### Step 5 - Find the same conversation in Purview audit

Lee opens **Purview -> Audit -> Search**, filtered to Allan and the same time window, and finds the Copilot interaction record for Step 1. He records its fields.

This search can also be run as a script, using Microsoft Graph's audit log query API, which avoids depending on a particular Security & Compliance PowerShell role assignment:

```http
POST https://graph.microsoft.com/beta/security/auditLog/queries
{
  "displayName": "DEFENDER002-STEP5",
  "filterStartDateTime": "<UTC start>",
  "filterEndDateTime": "<UTC end>",
  "operationFilters": ["CopilotInteraction"],
  "userPrincipalNameFilters": ["<Allan's UPN>"]
}
```

This needs the delegated scope `AuditLogsQuery.Read.All`, in addition to the scopes already in use. The query runs asynchronously: poll `GET .../security/auditLog/queries/{id}` until `status` is `succeeded` (allow several minutes; in the author's lab it took about 12 minutes), then read `GET .../security/auditLog/queries/{id}/records`. Match the record whose `auditData.CopilotEventData.ConversationId` (or, for Copilot Studio agents, the `AgentId` inside it) matches the conversation found in Step 4.

### Step 6 - Write down what the agent records add

Lee makes a two-column table: the fields in the audit record (Step 5), and the **additional** fields in the agent records (Step 4, excluding any CopilotInteraction baseline copy). Name each field; "more detail" is not enough.

### Step 7 - Optional: a restricted conversation

Only if use case 09's DLP rule is still on. Allan asks use case 09's **Q-LABEL** question with `DEFENDER002-STEP7`, and Lee runs Step 4 for that time window. Record where the records show the conversation differed from Step 1, or that they do not.

### Step 8 - Alerts involving the agent

```kusto
let id1 = "<AgentId>";
let id2 = "<EntraAgentID>";
let id3 = "<SourceAgentId>";
let id4 = "<ObservabilityID>";
AlertEvidence
| where Timestamp > ago(7d)
| extend Raw = tostring(pack_all())
| where (isnotempty(id1) and Raw contains id1)
    or (isnotempty(id2) and Raw contains id2)
    or (isnotempty(id3) and Raw contains id3)
    or (isnotempty(id4) and Raw contains id4)
| distinct AlertId, Title, ServiceSource
| join kind=inner (AlertInfo | distinct AlertId, Severity) on AlertId
| project AlertId, Title, Severity, ServiceSource
```

No result is a valid outcome. This use case does not create alerts.

## Section 6 - What you should see

| Evidence | What it supports |
|---|---|
| Step 3: activity types for AGT01-CPS | The agent sends activity data that Defender can search |
| Step 4: records matched to Allan's conversation by agent, person and time | The agent's own actions can be traced for one specific conversation |
| Step 6: named fields beyond the audit record | Exactly what the agent records add for an investigation |
| Step 7: a difference for the restricted conversation | Where a restricted request stopped, if the records show it |

## Section 7 - If something goes wrong

| What you see | What to do |
|---|---|
| `CloudAppEvents` has no agent records | Check the Microsoft 365 connector includes Microsoft 365 activities (Check 1) |
| Step 2 returns nothing | The agent is not in the Agent 365 inventory; see use case 11 |
| Step 3 returns nothing | Data has not arrived yet, or the platform does not send it; wait and run again |
| Step 4 is empty but Step 3 has records | Widen the time window; check Allan's recorded time |
| Only CopilotInteraction records | The agent records add nothing beyond audit for this conversation; record that |
| Fields differ from the examples | Expected; use the fields your records actually contain |

## Section 8 - Put everything back

Nothing is changed by this use case. Delete any exported raw records that are not needed as evidence.

## Section 9 - Outcome

| Action | Result |
|---|---|
| Match agent records to Allan's question | "This is the agent's own record of Allan's request: which agent acted, as which identity, and what it used." |
| Compare with the audit record | "Your audit log already shows the question and the answer. The agent records add these specific details." |
| Show a restricted conversation, if run | "Here is where the restricted request stopped." |

**Do not say:**

- that Contoso had no logs before;
- that every agent platform records the same fields;
- that the records show content they do not actually contain.

Record the conclusion using the [results criteria](99-Results-and-Handoff.md#12---defender002---show-what-the-agent-did).

## Section 10 - Sources

| Topic | Source |
|---|---|
| Agent observability in `CloudAppEvents`; alert tables; Copilot Studio, Foundry and Agent Builder send data by default; other platforms need the Agent 365 SDK; preview | [Detect and investigate threats to AI agents](https://learn.microsoft.com/en-us/defender-xdr/security-for-ai/ai-agent-detection-protection) |
| Microsoft 365 connector components, including Microsoft 365 activities | [Enable security for AI agents](https://learn.microsoft.com/en-us/defender-xdr/security-for-ai/get-started-defender-security-for-ai) |
| `CloudAppEvents` columns | [CloudAppEvents table](https://learn.microsoft.com/en-us/defender-xdr/advanced-hunting-cloudappevents-table) |
| `AgentsInfo` identifiers: AgentId, EntraAgentID, SourceAgentId, ObservabilityID | [AgentsInfo table](https://learn.microsoft.com/en-us/defender-xdr/advanced-hunting-agentsinfo-table) |
| The Copilot interaction audit baseline | [Purview for Copilot Studio agents](https://learn.microsoft.com/en-us/purview/ai-copilot-studio) |
