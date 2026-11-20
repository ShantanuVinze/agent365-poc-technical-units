# Agent 365 Value Demonstration Guide

> **Disclaimer.** This guide is a **personal project** by its author. It was created in a personal capacity, is **not Microsoft intellectual property**, and is **not an official Microsoft publication**. It is not reviewed, endorsed or supported by Microsoft, and it does not represent Microsoft's views, positions or product roadmap. The views and observations here are the author's own. Microsoft product names are used only to describe the products. All organizations, people, documents and values in the guide are fictional. The guide is provided "as is", without warranty, and you use it at your own risk. Always check the linked Microsoft documentation for current licensing, behavior and preview status.

This guide shows, with evidence, what **Microsoft Agent 365** adds to the governance and protection of AI agents. It builds on the identity, security and compliance tools an organization already has, such as Microsoft 365 E5.

It follows **Contoso**, a fictional organization, as its IT team builds test agents, turns on controls and checks the results.

**Who it is for:** an IT generalist with Level-200 knowledge. You do not need to be an expert in Microsoft Entra, Purview or Defender. Each step says who does it, where, what to enter and what you should see.

## 1. What you will achieve

- Answer specific business questions about controlling AI agents, one use case at a time.
- See each control work in a test tenant, including tests that are not safe to run in production.
- Record the evidence and reach a clear, honest conclusion.

You do not need to run every use case. Choose the ones that answer your questions.

## 2. Environment

Everything in this guide runs in one Microsoft 365 tenant, **Contoso Labs**, with one set of users. It is a non-production tenant, so controls can be turned on and off and safe simulations can be run.

You can run the same steps in any tenant you choose. In a tenant other than a test tenant, get the tenant owner's approval before every change, and use test users and test agents.

## 3. The Contoso team

| Name | Role | Responsibility |
|---|---|---|
| **Alex** | Environment and Identity Administrator | Licences, accounts and the evidence folder; creates and changes Microsoft Entra settings and policies |
| **Adele** | Purview Administrator | Microsoft Purview settings and policies; independent review of the evidence |
| **Nestor** | Agent Creator | Builds, creates and publishes the agents; runs the agent tests |
| **Allan** | Business Test User | Uses the agent as a normal employee; acts as the agent's business sponsor where a use case needs one |
| **Lee** | Security and Compliance Analyst | Checks sign-in logs, alerts and reports, and records the findings |
| **Megan** | Business Manager | Allan's manager; takes over as the agent's sponsor in one use case |

- These people are fictional. Record which real account plays each person in the tracker.
- The Microsoft administrator roles each person needs are listed in [Setup](00-Setup.md#people-and-roles) and in each use case.

## 4. The agents

Each agent has a fixed placeholder name. A placeholder with an empty **Used in** column is reserved for use cases that will be added later. When a new use case needs another agent, add a row here with the next number.

| Name | What it is | Used in |
|---|---|---|
| **AGT01-CPS** | A Microsoft Copilot Studio agent built during [Setup](00-Setup.md#agent-setup), Standard harness. It answers questions from Contoso policy documents. | Use cases 01, 09, 10, 12, 14 |
| **AGT02-CPS** | A second agent built in Microsoft Copilot Studio, Standard harness ([Setup 10.5](00-Setup.md#105-a-second-agent-for-an-automated-run-and-the-block-pass-optional)) | Use cases 10 (Section 5B, automated run), 13 (block pass) |
| **AGT03-CPS** | A third Copilot Studio agent, Standard harness, built only because AGT01-CPS's original default (Microsoft Copilot) harness failed, in lab testing, to invoke a tool at runtime ([Setup 10.6](00-Setup.md#106-a-third-agent-for-use-case-13s-audit-pass)) | Use case 13 (audit pass) |
| **AGT03-FDY** | An agent developed in Microsoft Foundry | |
| **AGT04-FDY** | A second agent developed in Microsoft Foundry | |
| **AGT05-VTX** | An agent developed in Google Vertex AI | |
| **AGT06-BDR** | An agent developed in Amazon Bedrock | |
| **AGT07-DBG** | An agent developed in Databricks Genie | |
| **Policy-Digest-Agent** | An autonomous agent: it runs without a signed-in user and has its own Microsoft Entra agent identity. It is created from **Contoso-Autonomous-Blueprint**. | Use cases 01 (Scenario 2), 02 to 08 |
| **Approval-Demo-Agent** | A temporary Copilot Studio agent, submitted for administrator approval | Use case 07 |

## 5. How to use this guide

| Step | Open | What you do |
|---|---|---|
| 1 | This page | Understand the project and choose the use cases to run from [Section 6](#6-available-use-cases) |
| 2 | [Setup](00-Setup.md) | Prepare the Contoso Labs tenant: licences, people and their roles, the evidence folder, test documents, the agents and every product setting the use cases need |
| 3 | [Tracker](98-Tracker.md) | Record every real value as you work, such as account names, setting values and results. |
| 4 | Individual use cases | Run the use cases in number order: 01, 02, 03 and so on. Setup has already prepared everything they need. |
| 5 | [Results and handoff](99-Results-and-Handoff.md) | Check the evidence, write the conclusion and record what you decide to adopt |

## 6. Available use cases

The number is the order in which you run them. A use case that builds on another always has a higher number. For example, a Defender use case that relies on an Entra setting comes after that Entra use case.

| No. | Use case | Business question | What you do |
|---|---|---|---|
| 01 | [Entra - Time-bound agent access](01-Entra001-Time-Bound-Agent-Access.md) | Can an agent be given access for a limited time, with a recorded reason? | Give the Copilot Studio agent group membership through an access package; then the sponsor requests time-limited Graph access for the autonomous agent, and its Graph request works only during the approved period |
| 02 | [Entra - Conditional Access for an autonomous agent](02-Entra002-Autonomous-Agent-Conditional-Access.md) | Can a Conditional Access policy check and block an agent that runs without a user? | Run the agent's token test with the policy off, in Report-only, on and off again, and read the result in its sign-ins |
| 03 | [Entra - Pause one agent capability](03-Entra003-Pause-One-Agent-Capability.md) | Can we block one thing an agent does while it keeps doing the rest? | Block the agent's Azure management access while its Microsoft Graph access keeps working; then restore it |
| 04 | [Entra - Sponsor stops an agent](04-Entra004-Sponsor-Stops-an-Agent.md) | Can the business sponsor stop an agent without IT, while only IT can restart it? | Allan disables the agent and cannot re-enable it; Alex restores it |
| 05 | [Entra - Sponsor continuity](05-Entra005-Sponsor-Continuity.md) | When a sponsor leaves, does someone else become accountable for the agent? | A sponsor-only workflow makes Megan the sponsor; Allan's account and the agent are untouched. The template's manager-email task is optional and disabled by default in this lab, so an email to Megan is not guaranteed unless you enable it (see the use case's Section 4.1) |
| 06 | [Entra - Contain a high-risk agent](06-Entra006-Contain-a-High-Risk-Agent.md) | Is an agent blocked automatically when its identity is high risk, and allowed again when the risk is cleared? | Lee declares the agent compromised: a standing policy blocks it. He dismisses the risk: it works again, with no policy change. |
| 07 | [Entra - Approval-gated agent access](07-Entra007-Approval-Gated-Agent-Attribute-and-Conditional-Access.md) | Can administrator approval tag an agent, and can a policy identify requests without that tag? | Approve a new agent through the tested publishing route with a template that tags it; on a separate autonomous agent, Report-only flags requests without the tag and shows the policy does not apply once tagged |
| 08 | [A365 - Apply a standard to existing agents](08-A365001-Apply-a-Standard-to-Existing-Agents.md) | Can a new control standard be applied to agents that already exist? | Tag an existing agent in one action with an Apply template rule; an attribute-scoped policy then applies to any agent carrying the tag, and stops when the tag is removed |
| 09 | [Purview - Existing labels and DLP still apply](09-Purview001-Existing-Labels-and-DLP-Still-Apply.md) | Do our existing sensitivity labels and DLP rules still protect content when an agent uses it? | The agent cannot use a labelled file but still answers from unlabelled files |
| 10 | [Purview - Insider risk for agents](10-Purview002-Insider-Risk-for-Agents.md) | Can insider risk investigation cover agents, not just people? | Monitor the agent's activity and investigate an alert. An optional custom policy and automated run (Sections 4B/5B) make the input workload repeatable for evaluating alert generation, not a particular alert guaranteed on demand |
| 11 | [Defender - Agent security posture](11-Defender001-Agent-Security-Posture.md) | Which agents should security review first, and which could not be assessed? | Review the lab agents; fix a real recommendation if one exists |
| 12 | [Defender - Show what the agent did](12-Defender002-Show-What-the-Agent-Did.md) | Can we trace what an agent actually did, beyond the prompt and answer? | Trace a normal interaction, and one blocked by the DLP rule from use case 09 |
| 13 | [Defender - Stop an unsafe agent action](13-Defender003-Stop-an-Unsafe-Agent-Action.md) | Can we stop a risky agent action before it runs? | Record, then block, a test action using a Microsoft test link and an email to an untrusted domain; normal email still works |
| 14 | [A365 - Withdraw a shared tool](14-A365002-Withdraw-a-Shared-Tool.md) | Can we apply a tenant-wide tool block without deleting agents, and observe withdrawal and recovery on a test agent? | Block and unblock one tool in the tenant-wide registry; verify the effect on AGT01-CPS while its knowledge still works |

## 7. What each use case page contains

- **Why it matters:** the concern, what an organization can already do and what Agent 365 adds.
- **People and assets:** who takes part, which agent and which documents or policies are used.
- **Workflow at a glance:** a short diagram of who does what and what you should see.
- **Before you start and set up the control:** the checks and settings needed before you run it.
- **Steps and expected results:** what to do, step by step, and what you should see.
- **Troubleshooting and undo:** what to do if a step fails, and how to put everything back.
- **Outcome:** what the evidence supports, and what it does not.
- **Sources:** the Microsoft documentation behind each step.

No results are filled in. Record your own results in the tracker.

## 8. Terms used in this guide

| Term | Meaning |
|---|---|
| **Microsoft Agent 365** | Microsoft's capability for managing, governing and securing AI agents across Microsoft 365, Entra, Purview and Defender |
| **Agent identity** | An identity in Microsoft Entra that belongs to an agent, so the agent can be governed like a user or app |
| **Autonomous agent** | An agent that acts on its own, without a signed-in user |
| **Blueprint** | The template in Microsoft Entra from which agent identities are created |
| **Sponsor** | The business person accountable for an agent |
| **Conditional Access** | Microsoft Entra policies that decide whether a sign-in or access request is allowed |
| **Report-only mode** | A policy setting that records what a policy would have done, without blocking anything |
| **Access package** | A Microsoft Entra bundle of access that can be assigned with an end date and a recorded reason |
| **Sensitivity label** | A Microsoft Purview tag that marks how confidential a document is |
| **DLP (Data Loss Prevention)** | Microsoft Purview policies that stop sensitive content being used or shared in the wrong way |
| **Tracker** | The private table where you record real accounts, settings and results |
| **Evidence folder** | The restricted SharePoint library **Contoso-Agent365-Evidence**, created in [Setup 5.3](00-Setup.md#53-create-the-evidence-library) |

## 9. Safe working rules

- Use only the fictional Contoso documents and the approved scope. Never weaken existing protection to force a result.
- Keep passwords, secrets, access tokens, real tenant IDs and screenshots out of this repository. Once the tracker holds real values, keep it private and do not publish it.
- Store evidence only in the evidence folder.
- Put every setting back as it was at the end of each use case. Each page explains how.

This is an independent, personal-project guide. It is not Microsoft intellectual property, it is not official Microsoft documentation, and it does not claim Microsoft endorsement. See the disclaimer at the top of this page. Check the linked Microsoft documentation for current licensing and preview status before you start.

**License:** the original content of this repository is released under the [MIT License](LICENSE). It is shared for education and awareness of the products it describes, not for sale.
