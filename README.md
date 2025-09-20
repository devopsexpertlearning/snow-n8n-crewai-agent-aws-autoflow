# snow-n8n-crewai-agent-aws-autoflow
Automated ticket handling and approval workflow integrating ServiceNow, CrewAI, n8n, and AWS WorkSpaces. Classifies tickets, manages workspace approvals with reminders and escalations, provisions AWS WorkSpaces, and handles non-workspace requests automatically.

### 🔹 End-to-End Flow

#### 1. Ticket Collection

n8n polls ServiceNow twice per day for new tickets.


#### 2. CrewAI Classification

- CrewAI analyzes the ticket description & metadata.

- Output includes:

  - is_workspace_request
  -  resources_requested (e.g., ["workspace","ec2","rds"])
  -  action_type:
  -  "workspace_only"
  -  "workspace_and_others"
  -  "non_workspace"


#### 3. Handling Based on Classification

- ##### Case A: Workspace Only

  - n8n starts approval workflow (explained in Step 4).

- ##### Case B: Workspace + Other Resources

  - n8n handles in two parts:

    1. Workspace request → goes through approval workflow.

    2. Other resources (EC2, RDS, etc.) →

    3. Send mail to requester: “Your WorkSpace request is being processed. For EC2/RDS, please raise separate tickets.”

- ##### Case C: Non-Workspace

  - n8n sends notification to a configured team member.

  - Assigns ticket in ServiceNow for manual handling.
    
  - No email goes to requester.
  


#### 4. Approval Workflow for WorkSpace Requests

1. Day 0 → Initial approval email to approver.

2. If no response →

Day 1 → Reminder #1

Day 2 → Reminder #2

Day 3 → Reminder #3

##### 3. If still no approval → escalate to manager:

Day 4 → Escalation #1

Day 5 → Escalation #2

##### 4. If still no approval →

Day 6 → Auto-close ticket in ServiceNow

Add closure note: “Auto-closed after 3 reminders + 2 escalations without approval.”

Notify requester about closure.


#### 5. Approver Decision

##### Approve:

CrewAI updates approval_state = approved.

n8n provisions AWS WorkSpace via WorkSpaces API.

Workspace details emailed to requester.

ServiceNow ticket updated with details.

##### Reject:

CrewAI updates approval_state = rejected.

n8n closes ticket in ServiceNow with rejection note.

Notify requester of rejection.


#### 6. Notifications

##### Requester:

Workspace approved → receives workspace details.

Workspace + other resources → also receives instruction to raise new tickets for non-workspace resources.

Auto-close → notified after closure.


##### Approver: initial email + 3 reminders.

Manager: escalation emails (2 times).

Team Member: receives non-workspace tickets for manual handling.


#### 7. Audit & Tracking

All events logged in ServiceNow:

Classification result (workspace only / mixed / non-workspace)

Approval state changes

3 reminders sent

2 escalations sent

Auto-close details (if applicable)

AWS WorkSpace provisioning info


#### 8. Flow Summary

SNOW Ticket → n8n poll → CrewAI classify
      ↓
┌───────────────┬────────────────────────────┬─────────────────────┐
│ Workspace Only│ Workspace + Other Resources│ Non-Workspace       │
└───────┬───────┴───────────────┬────────────┴─────────────┘
        ↓                           ↓                        ↓
Approval Workflow                   Approve workflow +       Assign to team
3 Reminders → 2 Escalations         Notify requester         Notify member
→ Auto-Close if no approval         for other services
        ↓
Approve → Provision Workspace
Reject  → Close Ticket

✅ This now matches your requirement:

3 approval reminders

2 escalation reminders

Auto-close after 6 days if still no approval

Handles workspace-only, mixed (workspace + others), and non-workspace requests correctly.
