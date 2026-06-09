# Securing Microsoft 365 Copilot with Microsoft Purview

A self-directed lab built in a personal Microsoft 365 tenant to answer one question: how do you make Copilot safe to deploy in an enterprise?

Copilot respects existing permissions, but most organizations have years of oversharing baked into SharePoint. Copilot surfaces that problem instantly. This lab builds the full Purview data security stack to find, classify, protect, and govern sensitive data before and after Copilot rollout, then validates every control with a live Copilot license.

## Environment

- Microsoft 365 tenant with full security and compliance licensing
- 7 SharePoint sites simulating a real org: HR, Finance, Legal, Executive, IT, Operations, Research
- Realistic Office format test files containing synthetic Canadian PII (SIN numbers, bank accounts)
- 7 deliberate oversharing and data exposure scenarios reproduced across the sites
- Azure OpenAI deployment (GPT-4.1 mini) for the custom labeling automation
- Copilot license assigned to a test user for live validation

## What Was Built

### Sensitivity Label Taxonomy

A complete label hierarchy from Personal up to Highly Confidential, including encrypted sublabels with real rights management:

| Label | Protection |
|---|---|
| Personal, Public, General | Classification only |
| Confidential / Finance | Encryption, Editor rights for Finance group |
| Highly Confidential / HR Only | Encryption, Viewer rights restricted to HR |
| Highly Confidential / Executive Restricted | Encryption, Viewer rights for one named user |

### Auto-Labeling

Two auto-labeling policies running in enabled mode, targeting Canadian Social Insurance Number and bank account Sensitive Information Types. Validated against the test corpus.

### Data Loss Prevention

Five DLP policies covering the full surface area:

1. SharePoint and OneDrive (Canadian PII)
2. Exchange Online (PII in email)
3. Endpoint DLP with seven blocked actions: print, copy/paste, screen capture, removable media, network share, cloud egress, remote desktop
4. Financial data policy
5. **DLP for Microsoft 365 Copilot**, blocking Copilot from processing or summarizing labeled content

### DSPM for AI

Data Security Posture Management for AI configured with risky AI usage detection and an oversharing remediation policy. SharePoint Advanced Management oversharing assessment enabled.

### Data Lifecycle Management

Retention policies mapped to realistic regulatory requirements:

- HR records: 7 years
- Finance records: 5 years
- Legal records: 10 years with record labels and disposition review

### eDiscovery

A full litigation scenario: eDiscovery case created, legal hold applied to the HR and Legal sites, content search built and executed against the held locations.

### Compliance Manager

Four assessments configured and mapped: PIPEDA, PHIPA (Ontario), Microsoft AI Baseline, and Data Protection Baseline.

## The Differentiator: AI Label Advisor

Purview auto-labeling only catches files matching predefined SIT patterns. Files that are sensitive by context rather than by pattern (a board strategy document, a reorg plan) slip through.

I built a Power Automate flow that closes this gap:

1. Runs daily and queries every SharePoint site via the Graph API for unlabeled Office files
2. Sends file metadata to Azure OpenAI, which recommends a sensitivity label from the full taxonomy
3. Collects Confidential and Highly Confidential recommendations into a batch
4. Sends one approval email to the file owner with Approve All / Reject All
5. On approval, applies labels automatically via the Graph API assignSensitivityLabel endpoint

This required registering the Microsoft.GraphServices resource provider and linking a Graph metered billing account to the Azure subscription, since assignSensitivityLabel is a metered API. App registration secrets are stored in Azure Key Vault. The flow was validated end to end on live files.



## Live Copilot Validation

The part most labs skip. A Copilot license was assigned to a test user and every control was tested against real Copilot prompts:

- Copilot blocked from summarizing files covered by the Copilot DLP policy (staff directory with PII, clinical trial participant data)
- Copilot blocked from accessing encrypted content labeled with restricted rights, even though the test user could see the file existed
- DSPM for AI capturing the interaction telemetry



## Key Takeaways

- Oversharing remediation has to come before Copilot deployment, not after. DSPM for AI and SharePoint Advanced Management give you the visibility, but fixing permissions is the real work.
- Auto-labeling based on SITs covers pattern-matchable data only. Contextually sensitive content needs either trainable classifiers or custom automation like the Label Advisor.
- Encryption with rights management is the only control that follows the file. DLP policies protect locations and channels; labels with encryption protect the data itself.
- The Graph API labeling endpoint is metered and requires deliberate billing setup. Worth knowing before promising automated labeling at scale.

## Documentation

Full project write-up with configuration detail and screenshots: [Purview_Copilot_Project.pdf]

