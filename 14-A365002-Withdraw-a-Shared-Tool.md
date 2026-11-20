# 14 - A365002 - Withdraw a shared tool

This use case demonstrates a **tenant-wide tool block** by testing its effect on **AGT01-CPS**, without deleting or changing that agent during the block and unblock tests.

AGT01-CPS is given a read-only **User** MCP server tool. Alex blocks that server in the Microsoft 365 admin center. The agent can no longer use the tool, but it still exists and still answers from its SharePoint knowledge. Alex unblocks the server and the tool works again, with no change to the agent.

**A block applies to every user and every agent in the tenant.** MCP tools and the User server are in **preview**.

The tenant-wide scope is Microsoft's documented control behavior. This lab observes tool withdrawal and recovery on one agent; it does not test every agent or user in the tenant.

## Section 1 - Why it matters

**Contoso's concern:** if a shared tool becomes untrustworthy, for example during a supplier incident, Contoso needs to withdraw it from all agents at once, without finding and deleting each agent.

**What Contoso can already do:** Power Platform data policies control which connectors makers can use, environment by environment.

**What Agent 365 adds:** a tenant-wide tool registry in the Microsoft 365 admin center (**Agents -> Tools -> Registry**). It lists the plugins, skills, MCP servers and connectors available to agents. **Block** "prevents the selected tool from being used by agents or workflows", and **Unblock** restores it.

Blocking an MCP server disables **all** the tools it exposes, and also blocks plugins that depend on it and linked connectors.

**Licensing:** the [Agent 365 service description](https://learn.microsoft.com/en-us/office365/servicedescriptions/microsoft-agent-365/microsoft-agent-365) lists "Control which tools (including Microsoft MCP servers) agents can access tenant-wide" as requiring Microsoft 365 E7 or Agent 365; it is not included with other Microsoft 365 plans. Using MCP tools also needs a Microsoft 365 Copilot licence.

| This use case shows, when observed | It does not show |
|---|---|
| A tenant-wide block stopped this agent from using the MCP server | That the agent's other routes to the same data are closed. Another tool or a direct API may still reach it. |
| The agent itself, and its knowledge, kept working | An instant effect. Timing is recorded, not assumed. |
| Unblocking restored the tool without changing the agent | Control of Power Platform connectors used directly by makers. Those are governed by Power Platform data policies. |
| | That every agent and user was tested. This run tests AGT01-CPS with Nestor. |

## Section 2 - People and assets

| Name | Role in this use case | Microsoft access needed |
|---|---|---|
| **Nestor** | Adds and later removes the tool; asks the test questions; checks the agent's activity | Copilot Studio access and Microsoft 365 Copilot licence from Setup |
| **Alex** | Blocks and unblocks the server | AI Administrator |
| **Adele** | Reviews the evidence independently | Read access to the evidence library |

Nestor asks the questions himself because Copilot Studio's **Activity** page shows only your own conversations unless an administrator turns on transcript sharing.

| Asset | Built in | Used for |
|---|---|---|
| **AGT01-CPS**, using generative orchestration | [Setup Section 10](00-Setup.md#section-10---the-copilot-studio-agent) | The agent that uses the tool |
| `Contoso-Meal-General-Policy.docx` | [Setup Section 6](00-Setup.md#section-6---test-documents) | The knowledge question that should keep working |
| **User** MCP server (server ID `mcp_MeServer`) | Added in [4.1](#41-give-agt01-cps-the-user-tool) | The tool that is withdrawn |
| Evidence folder | [Setup 5.3](00-Setup.md#53-create-the-evidence-library) | `14-Withdraw-a-Shared-Tool/<date>-<run>` |

The User server is one of Microsoft's workload-specific MCP servers. Microsoft keeps them for backward compatibility and does not recommend them for production or new projects. It is used here only because it is read-only and simple to demonstrate.

### Workflow at a glance

```text
Nestor
   Q-TOOL      -> AGT01-CPS -> User MCP server -> display name
   Q-KNOWLEDGE -> AGT01-CPS -> SharePoint knowledge -> policy answer
                         |
Alex: Microsoft 365 admin center -> Agents -> Tools -> Registry
                         |
        +----------------+-----------------+
        |                                  |
  Step 1 Available                   Step 3 Blocked (whole tenant)
  Q-TOOL: name returned*             Q-TOOL: tool unavailable*
  Q-KNOWLEDGE: answered*             Q-KNOWLEDGE: still answered*
                                           |
                                     Step 4 Unblocked
                                     Q-TOOL: name returned again,
                                     no agent change*

* Expected results to confirm and record.
```

## Section 3 - Before you start

| Check | Who | Pass condition | If it fails |
|---|---|---|---|
| Check 1 - Safe to block | Alex | No other agent or user in the tenant relies on the User MCP server | Do not run. The block is tenant-wide. |
| Check 2 - Registry and Block available | Alex | **Microsoft 365 admin center -> Agents -> Tools -> Registry** lists MCP servers, and **Block** is offered | Record **Not available in this tenant** and stop |
| Check 3 - The agent works | Nestor | AGT01-CPS gives the required answer to the first question in [Setup 10.2](00-Setup.md#102-test-the-agent) | Fix the agent first |

## Section 4 - Set up the control

### 4.1 Give AGT01-CPS the User tool

**Nestor**, in Copilot Studio:

1. Open **AGT01-CPS -> Tools -> Add a tool -> Model Context Protocol**. Search for **User** (publisher Microsoft) and select it.
2. Create the connection when prompted, using your own account. Select **Add**.
3. In the agent's instructions, **replace** the line `Answer only from the approved SharePoint knowledge documents.` with:

   ```text
   Answer policy questions only from the approved SharePoint knowledge documents.
   Lab only: when asked for the user's own display name, use the User tool and return only the display name.
   ```

   Record the original line; it is restored in Section 8.
4. Publish.

### 4.2 The block

There is nothing to configure in advance. The control is the **Block** action on the server's registry entry, used at Step 2. Before Step 2, Alex finds the User server in **Agents -> Tools -> Registry** and records its name, type, publisher and status.

## Section 5 - Run it in Contoso Labs

Record every result in the [tracker](98-Tracker.md#14---a365002---withdraw-a-shared-tool).

**The questions.** Nestor asks them in Microsoft 365 Copilot, choosing **AGT01-CPS**, with a **new conversation** for each question. Replace `<step>` with the step number, for example `STEP1`, and record the time.

```text
Q-TOOL: For A365002-LAB-01-<step>, use the User tool to get my display name. Return only the display name.
```

```text
Q-KNOWLEDGE: For A365002-LAB-01-<step>, read Contoso-Meal-General-Policy. Return its document marker and meal allowance. Cite the source.
```

**After every question**, Nestor opens **AGT01-CPS -> Activity** in Copilot Studio, opens that conversation and records whether the User tool was called and whether the call succeeded.

### Step 1 - Before the block

Nestor asks both questions.

| Question | Expected |
|---|---|
| Q-TOOL | Nestor's display name; Activity shows a successful User tool call |
| Q-KNOWLEDGE | **WS03-GENERAL-104** and **80**, with a citation |

### Step 2 - Block the server

Alex opens **Agents -> Tools -> Registry**, selects the User server, selects **Block** and confirms. Alex records the time. Expected: the status shows **Blocked**.

### Step 3 - While blocked

Nestor asks both questions at about 15 minutes, 1 hour and 4 hours after Step 2, stopping once the result changes.

| Question | Expected |
|---|---|
| Q-TOOL | No display name from the tool. The agent says it cannot use the tool, and Activity shows the call failed or was not made. |
| Q-KNOWLEDGE | Still answered, with a citation |
| The agent | Still listed and published; nothing in it was changed |

If both questions fail, the agent or the channel has a problem; this is not a tool-withdrawal result.

### Step 4 - Unblock the server

Alex selects the User server and **Unblock**, and records the time. Nestor asks both questions again, repeating at the same intervals if needed. Expected: the status shows **Available**, Q-TOOL returns the display name again, and no change was made to the agent.

## Section 6 - What you should see

| Evidence | What it supports |
|---|---|
| Step 2: registry status **Blocked** | The tenant-wide server block is configured |
| Step 3: tool unavailable, knowledge answered | The block's effect was observed on AGT01-CPS while its knowledge still worked |
| Step 3: time between the block and the change | How quickly the block took effect in this tenant |
| Step 4: tool works again, agent unchanged | Unblocking restored the tool with no agent rework |

## Section 7 - If something goes wrong

| What you see | What to do |
|---|---|
| **Block** is not offered | Check Alex has AI Administrator; otherwise record **Not available in this tenant** |
| The User tool is not in the Copilot Studio list | Check Nestor's Microsoft 365 Copilot licence and that generative orchestration is on |
| The tool is not called at Step 1 | Check the instruction line from 4.1 and that the agent was republished |
| Q-TOOL still works 4 hours after the block | Record **Not observed**; confirm the status is **Blocked** |
| Q-TOOL gives a name at Step 3, but Activity shows no tool call | The model answered without the tool. Treat it as a failed tool call, not a working one. |
| The tool still fails after unblocking | Check again later; confirm the status is **Available** |

## Section 8 - Put everything back

1. Alex confirms the User server status is **Available**.
2. Nestor removes the User tool, restores the original instruction line from 4.1, and republishes.
3. Nestor confirms Q-KNOWLEDGE still gets the required answer.

## Section 9 - Outcome

| Action | Result |
|---|---|
| Steps 2 and 3 | "We applied a tenant-wide block and confirmed that AGT01-CPS could no longer use this tool. Its knowledge still worked." |
| Step 4 | "Unblocking restored it, with no change to the agent." |

**Do not say:**

- that the lab verified withdrawal for every agent or user; only AGT01-CPS with Nestor was tested;
- that the block closes every route to the underlying data;
- that it controls Power Platform connectors used directly by makers;
- that the effect is instant.

Record the conclusion using the [results criteria](99-Results-and-Handoff.md#14---a365002---withdraw-a-shared-tool).

## Section 10 - Sources

| Topic | Source |
|---|---|
| Tools page, Registry, Block and Unblock; Power Platform connector usage (preview) | [Tools page in the Microsoft 365 admin center](https://learn.microsoft.com/en-us/microsoft-365/admin/manage/agent-tools-overview) |
| AI Administrator or Global Administrator required; blocking an MCP server disables all its tools and blocks dependent plugins and linked connectors | [Manage plugins, skills and MCP servers](https://learn.microsoft.com/en-us/microsoft-365/admin/manage/manage-plugins-skills-mcp-servers) |
| A blocked MCP server is blocked for every user and agent; Microsoft 365 Copilot licence required | [Work IQ MCP overview](https://learn.microsoft.com/en-us/microsoft-agent-365/tooling-servers-overview) |
| User server ID `mcp_MeServer`; workload-specific servers are for backward compatibility (preview) | [User reference](https://learn.microsoft.com/en-us/microsoft-copilot-studio/mcp-user-tools) |
| The Activity page shows your own conversations unless transcript sharing is enabled | [Review agent activity](https://learn.microsoft.com/en-us/microsoft-copilot-studio/authoring-review-activity) |
