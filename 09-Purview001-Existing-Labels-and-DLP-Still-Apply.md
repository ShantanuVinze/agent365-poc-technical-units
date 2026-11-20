# 09 - Purview001 - Existing labels and DLP still apply

This use case shows that the data protection Contoso already runs, **sensitivity labels** and **data loss prevention (DLP)**, keeps working when an agent uses the same content.

Allan asks **AGT01-CPS** about a document labelled confidential and gets an answer. Adele then turns on a DLP rule for that label. The agent stops using the document's content, while it still answers from the unlabelled general policy.

It shows that agents do not open a way around the protections Contoso already defines: the same labels and the same kind of DLP policy apply to the agent.

## Section 1 - Why it matters

**Contoso's concern:** "When we add agents, do our existing data protections still apply, or do agents create a way around them?"

**What Contoso can already do:** with Microsoft Purview, Contoso labels confidential documents and uses DLP policies. A DLP policy for the **Microsoft 365 Copilot and Copilot Chat** location can stop Copilot from processing files with a given label.

**What Agent 365 adds:** the same labels and DLP model extends to agents. When a Copilot Studio agent uses SharePoint as knowledge, a DLP policy for the Microsoft 365 Copilot location can stop it, in Teams, SharePoint and Microsoft 365 Copilot, from processing content with the chosen label, and the agent shows the label of the items it cites. No separate policy model is needed for agents.

**Licensing:** the [Agent 365 service description](https://learn.microsoft.com/en-us/office365/servicedescriptions/microsoft-agent-365/microsoft-agent-365) lists "Agents inherit and honor data sensitivity labels" and "Data loss prevention - block agents from accessing and sharing sensitive content" as requiring Microsoft 365 E7 or Agent 365.

| This use case shows, when observed | It does not show |
|---|---|
| A label-based DLP rule stops this agent using a labelled document, for this person | That a new kind of policy was needed. It is the same DLP location Contoso uses for Microsoft 365 Copilot. |
| Unlabelled knowledge still works under the same policy | That the policy covers agent platforms or channels other than those Microsoft documents |
| The agent shows the source's label in its citations | That a labelled item never appears. Microsoft notes such items can still be listed in citations; their content is not used. |

## Section 2 - People and assets

| Name | Role in this use case | Microsoft access needed |
|---|---|---|
| **Adele** | Creates, turns on and turns off the DLP policy; reviews the evidence | Purview Data Security AI Admin (least privilege), or Compliance Administrator from Setup |
| **Allan** | Asks the agent the fixed questions | Agent access and read access from Setup |
| **Lee** | Finds the interactions in the audit log | Audit Reader |

| Asset | Built in | Used for |
|---|---|---|
| **AGT01-CPS**, with the knowledge folder | [Setup Section 10](00-Setup.md#section-10---the-copilot-studio-agent) | The agent Allan asks |
| `Contoso-Board-Confidential.docx`, with a non-encrypting label | [Setup Section 6](00-Setup.md#section-6---test-documents) and [7.1](00-Setup.md#71-apply-a-sensitivity-label) | The labelled document |
| `Contoso-Meal-General-Policy.docx` | [Setup Section 6](00-Setup.md#section-6---test-documents) | The unlabelled document that keeps working |
| Audit logging | [Setup 7.3](00-Setup.md#73-confirm-audit-logging-is-on) | Evidence of each interaction |
| DLP policy `Contoso-DLP-Copilot-Confidential-Label` | Created in [Step 2](#step-2---turn-on-the-dlp-rule) using the settings in [4.1](#41-dlp-policy-settings) | The control demonstrated |
| Evidence folder | [Setup 5.3](00-Setup.md#53-create-the-evidence-library) | `09-Existing-Labels-and-DLP/<date>-<run>` |

A non-encrypting label keeps this test about DLP. With an encrypting label, the result would also depend on Allan's VIEW and EXTRACT rights, which is a different control.

### Workflow at a glance

```text
AGT01-CPS reads the knowledge folder, including:
   Contoso-Board-Confidential.docx  (labelled)
   Contoso-Meal-General-Policy.docx (not labelled)

Step 1  No DLP rule   Allan asks about both documents
                      -> labelled: WS03-CONFIDENTIAL-311, Lighthouse, label in citation*
                      -> general:  WS03-GENERAL-104, 80*
Step 2  Adele turns on the DLP rule for the label (wait up to 4 hours)
Step 3  DLP rule on   -> labelled: content not used (may still be cited)*
                      -> general:  still answered*
Step 4  DLP rule off  -> labelled: answered again*

Lee finds each interaction in the audit log.

* Expected results to confirm and record.
```

## Section 3 - Before you start

| Check | Who | Pass condition | If it fails |
|---|---|---|---|
| Check 1 - The agent answers | Allan | In Microsoft 365 Copilot, AGT01-CPS answers the general policy question with WS03-GENERAL-104, 80, 40 and a citation ([Setup 10.3](00-Setup.md#103-publish-and-share-with-allan)) | Fix Setup Section 10 first |
| Check 2 - The label is applied | Allan | `Contoso-Board-Confidential.docx` shows its label in Word ([Setup 7.1](00-Setup.md#71-apply-a-sensitivity-label)), and the label does not apply encryption | Fix Setup 7.1 |
| Check 3 - The DLP location exists | Adele | **Purview -> Data loss prevention -> Policies -> Create policy -> Custom** offers the **Microsoft 365 Copilot and Copilot Chat** location | Record and stop |
| Check 4 - No existing rule applies | Adele | No existing DLP policy for the Microsoft 365 Copilot and Copilot Chat location already excludes this label for Allan. Record any existing policies, including any default ones, with their mode. | Record it. Step 1 would already show the restricted behaviour. |
| Check 5 - Audit is on | Lee | [Setup 7.3](00-Setup.md#73-confirm-audit-logging-is-on) passed | Fix audit first, or record that audit evidence will be missing |

## Section 4 - Set up the control

### 4.1 DLP policy settings

Adele creates this policy at **Step 2**, after the "before" questions, not now.

| Setting | Value |
|---|---|
| Path | **Purview -> Data loss prevention -> Policies -> Create policy -> Custom -> Custom policy** (this location is only available in the Custom template) |
| Name | `Contoso-DLP-Copilot-Confidential-Label` |
| Description | `Lab: stop Copilot and agents processing items with the confidential label` |
| Locations | Only **Microsoft 365 Copilot and Copilot Chat**. Edit its scope to include **Allan** only, so no one else is affected. |
| Rule name | `Confidential label - no processing` |
| Condition | **Content contains -> Sensitivity labels ->** the label from Setup 7.1. Label and sensitive information type conditions cannot be in the same rule. |
| Action | **Prevent Copilot from processing content**, choosing the option for files and emails (not prompts or web searches). Record the exact labels shown. |
| Policy mode | **Turn the policy on** |

Changes to a DLP policy can take **up to four hours** to reach Microsoft 365 Copilot and Copilot Chat.

## Section 5 - Run it in Contoso Labs

Record every result in the [tracker](98-Tracker.md#09---purview001---existing-labels-and-dlp-still-apply).

Allan asks these two questions in Microsoft 365 Copilot, selecting **AGT01-CPS**. He starts a **new conversation** for each question, and records the answer, the citations and the time. Replace `<step>` with the step number, for example `PURVIEW001-STEP1`.

```text
Q-LABEL: For PURVIEW001-<step>, read Contoso-Board-Confidential.
Return its document marker and programme code name. Cite the source.
```

```text
Q-GENERAL: For PURVIEW001-<step>, read Contoso-Meal-General-Policy.
Return its document marker and meal allowance. Cite the source.
```

### Step 1 - Before the DLP rule

Allan asks both questions with `PURVIEW001-STEP1`.

| Question | Expected |
|---|---|
| Q-LABEL | **WS03-CONFIDENTIAL-311** and **Lighthouse**, with a citation that shows the document's label |
| Q-GENERAL | **WS03-GENERAL-104** and **80**, with a citation |

### Step 2 - Turn on the DLP rule

Adele creates `Contoso-DLP-Copilot-Confidential-Label` with the settings in [4.1](#41-dlp-policy-settings), turned on. She records the time and checks the policy's status in the DLP list. Wait up to four hours before Step 3.

### Step 3 - After the DLP rule

Allan asks both questions with `PURVIEW001-STEP3`.

| Question | Expected |
|---|---|
| Q-LABEL | The agent does **not** return WS03-CONFIDENTIAL-311 or Lighthouse from the document. The document may still be listed in the citations. |
| Q-GENERAL | Still answered: WS03-GENERAL-104 and 80 |

- If the agent gives a value without using the document, record it as a guess and check the citation.
- If Q-GENERAL also fails, the agent or channel has a problem; it is not a DLP result.
- If Q-LABEL is still answered, run again after 1, 2 and 4 hours with `-02`, `-03` and so on, and record the times.

### Step 4 - Turn the rule off

Adele turns the policy off. Wait up to four hours. Allan asks Q-LABEL again with `PURVIEW001-STEP4`. Expected: answered again.

### Evidence

**Lee** opens **Purview -> Audit -> Search**, filtered to Allan and the test window, and finds the interaction records for each step. Microsoft documents that these records can include the files accessed and their sensitivity labels. Record what the records actually show for Steps 1 and 3.

## Section 6 - What you should see

| Evidence | What it supports |
|---|---|
| Step 1: labelled content returned, with the label in the citation | The agent shows existing labels on what it cites |
| Step 3: labelled content not used; general question still answered | The existing DLP rule applies to the agent for this label, and only this label |
| Step 4: labelled content answered again | The change came from the DLP rule alone |
| Audit records for each interaction | The interactions are recorded, with the files accessed |

## Section 7 - If something goes wrong

| What you see | What to do |
|---|---|
| Step 1 is already restricted | An existing policy applies (Check 4), or the knowledge is not ready |
| No label in the Step 1 citation | Check the label was applied and saved; wait for the folder to be indexed |
| Step 3 unchanged | Wait the full four hours. Check the location, the label and that Allan is in scope. |
| Both questions fail in Step 3 | Agent or channel problem; recheck Check 1 |
| The agent seems to answer from memory | Use a new conversation; check the citation |
| The **Microsoft 365 Copilot and Copilot Chat** location is missing | You started from a template other than **Custom** |

## Section 8 - Put everything back

1. Adele deletes `Contoso-DLP-Copilot-Confidential-Label`, or keeps it off.
2. Leave the documents, label and agent knowledge as they are. Later use cases use them.

## Section 9 - Outcome

| Action | Result |
|---|---|
| Ask about the labelled document | "The agent shows your sensitivity labels on what it cites." |
| Turn on the label-based DLP rule | "When DLP said Copilot must not process confidential content, the agent stopped using that document." |
| Ask about the unlabelled document | "Everything else kept working." |
| Explain where this comes from | "Your existing labels and DLP approach now extends to agents. There is no separate policy model to learn." |

**Do not say:**

- that they cover every agent platform or channel;
- that a labelled item never appears in a citation.

Record the conclusion using the [results criteria](99-Results-and-Handoff.md#09---purview001---existing-labels-and-dlp-still-apply).

## Section 10 - Sources

| Topic | Source |
|---|---|
| Purview capabilities for Copilot Studio agents; labels shown in citations; encryption needs VIEW and EXTRACT; DLP for the Microsoft 365 Copilot location covers Copilot Studio agents with SharePoint knowledge; audit records include files accessed and their labels | [Purview for Copilot Studio agents](https://learn.microsoft.com/en-us/purview/ai-copilot-studio) |
| Label condition; items still appear in citations; label and sensitive information type conditions in separate rules; Custom template only; simulation mode supported; up to four hours to apply; roles including Purview Data Security AI Admin | [DLP for Microsoft 365 Copilot and Copilot Chat](https://learn.microsoft.com/en-us/purview/dlp-microsoft365-copilot-location-learn-about) |
| The agent retrieves only content the user can access | [SharePoint knowledge in Copilot Studio](https://learn.microsoft.com/en-us/microsoft-copilot-studio/knowledge-add-sharepoint) |
| Agent 365 licensing: agents honor sensitivity labels; DLP blocks agents | [Microsoft Agent 365 service description](https://learn.microsoft.com/en-us/office365/servicedescriptions/microsoft-agent-365/microsoft-agent-365) |
