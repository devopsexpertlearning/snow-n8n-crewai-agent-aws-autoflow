# AWS WorkSpaces Request Automation

This repository contains the **n8n workflows** for automating AWS WorkSpaces/EC2/Other resources requests using **ServiceNow**, **CrewAI**, and **AWS WorkSpaces API**, including approval workflow, reminders, escalations, and notifications.  

---

## Table of Contents

- [Overview](#overview)  
- [Modules](#modules)  
- [Architecture](#architecture)  
- [Setup Instructions](#setup-instructions)  
- [Credentials](#credentials)  
- [Workflow Execution](#workflow-execution)  
- [Notifications](#notifications)  
- [Audit & Tracking](#audit--tracking)  
- [License](#license)  

---

## Overview

This automation handles **workspace requests** and other AWS resource requests from ServiceNow tickets.  

Key features:

- Automatic polling of ServiceNow tickets  
- Classification using CrewAI  
- Separate handling for:
  - Workspace-only requests → approval workflow  
  - Mixed requests → partial approval + requester notification  
  - Non-workspace requests → manual handling by team  
- Approval workflow with:
  - 3 reminders  
  - 2 escalations  
  - Auto-close after 6 days if no approval  
- AWS WorkSpaces provisioning via IAM credentials  
- Email notifications to requester, approver, and escalation manager  
- Complete audit logging in ServiceNow  

---

## Modules

The workflow is split into **6 modules** for easier maintenance and testing:

1. **Module 1 – ServiceNow Ticket Polling**  
   Polls ServiceNow twice daily for new tickets.

2. **Module 2 – CrewAI Classification**  
   Sends ticket description & metadata to CrewAI API for classification.

3. **Module 3 – Routing Logic**  
   Routes tickets based on classification: workspace-only, mixed, non-workspace.

4. **Module 4 – Approval Workflow**  
   Handles workspace approval:
   - Day 0: Initial email  
   - Days 1–3: Reminders  
   - Days 4–5: Escalations  
   - Day 6: Auto-close

5. **Module 5 – AWS WorkSpaces Provisioning**  
   Provisions WorkSpaces using AWS API upon approval.

6. **Module 6 – Notifications**  
   Sends emails to requester, approver, and escalation manager.

7. **Integration

Module 1 (ServiceNow Poll) → Module 2 (CrewAI Classification)
       ↓
Module 3 (Routing Logic) → Module 4 (Approval Workflow)
       ↓
Module 5 (Provision Workspace) → Module 6 (Notifications)


---

## Architecture

ServiceNow Ticket → n8n Poll → CrewAI Classify → Switch (Routing) ↓                        ↓                         ↓ Workspace Only           Workspace+Others           Non-Workspace ↓                        ↓                         ↓ Approval Workflow → Provision WorkSpace  → Notify Requester ↑ 3 Reminders → 2 Escalations → Auto-close


---

## Setup Instructions

1. **Install n8n**  
   - Docker: `docker run -it --rm -p 5678:5678 -v ~/.n8n:/home/node/.n8n n8nio/n8n`  
   - Or install natively: `npm install -g n8n`

2. **Import Workflows**  
   - Import JSON files for Modules 1–6 into n8n.

3. **Configure Nodes**  
   - Each node references **n8n Credentials** (ServiceNow, SMTP, AWS, CrewAI).  

4. **Activate Workflows**  
   - Enable Cron triggers for polling and Wait nodes for approval workflow.

---

## Credentials

| Credential Type | Name in n8n | Notes |
|-----------------|------------|------|
| ServiceNow      | ServiceNow Dev | Instance URL, User, Password |
| SMTP            | SMTP AWS SES | Host, User, Password |
| AWS             | AWS WorkSpaces | IAM Access Key, Secret Key, Region |
| CrewAI          | CrewAI API | Endpoint URL / API Key |

**All credentials are encrypted in n8n. No secrets are stored in workflow JSON.**

---

## Workflow Execution

- **Ticket Polling**: Runs twice daily.  
- **CrewAI Classification**: Classifies tickets as `workspace_only`, `workspace_and_others`, or `non_workspace`.  
- **Approval Workflow**:
  - Emails sent to approver (3 reminders)  
  - Escalation emails to manager (2 reminders)  
  - Auto-close ticket after 6 days if no response  
- **AWS WorkSpaces Provisioning**: Triggered automatically after approval.  
- **Notifications**: Emails sent to requester with workspace details or closure notes.

---

## Notifications

- **Requester**:
  - Workspace approved → Workspace details  
  - Workspace + other resources → instructions for separate tickets  
  - Auto-close → notified after 6 days  

- **Approver**:
  - Initial approval email + 3 reminders  

- **Manager**:
  - 2 escalation emails  

- **Team Member**:
  - Non-workspace tickets assigned for manual handling

---

## Audit & Tracking

All events logged in ServiceNow:

- Classification result  
- Approval state changes  
- 3 reminders sent  
- 2 escalations sent  
- Auto-close details  
- AWS WorkSpaces provisioning info  

---

## License

This project is **open-source** under the **MIT License**.

---

## Author

**DevOps Expert**  
Email: info@devopsexpert.work.gd

---