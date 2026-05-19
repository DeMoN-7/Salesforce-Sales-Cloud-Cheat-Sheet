# Salesforce-Sales-Cloud-Cheat-Sheet
Salesforce Lead-to-Opportunity lifecycle explained end-to-end with detailed trigger execution, flows, order of execution, lead conversion internals, object relationships, automation flow, and opportunity closure process.

# Salesforce Lead to Opportunity Lifecycle – Complete Detailed Roadmap

This document explains the complete lifecycle in Salesforce from:

**Lead Creation → Qualification → Conversion → Account/Contact/Opportunity Creation → Sales Process → Opportunity Closure**

It also explains:

- Which records are created at each stage
- How records are linked together
- Which triggers, flows, workflow automations, validation rules, assignment rules, and processes usually execute
- Internal Salesforce behavior during conversion
- Data relationships between standard objects
- Automation execution order
- What happens technically in the backend

---

# 1. High-Level Salesforce Sales Lifecycle

The complete business flow generally looks like this:

```text
Lead Created
   ↓
Lead Qualification
   ↓
Lead Converted
   ↓
Account Created/Matched
   ↓
Contact Created
   ↓
Opportunity Created
   ↓
Sales Stages Progress
   ↓
Quote / Products / Activities
   ↓
Opportunity Closed Won or Closed Lost
```

---

# 2. Understanding Core Salesforce Objects

Before understanding the flow, you must understand the main standard objects.

| Object | Purpose |
|---|---|
| Lead | Unqualified prospect/person |
| Account | Company/organization/customer |
| Contact | Person associated with Account |
| Opportunity | Potential revenue/deal |
| OpportunityLineItem | Products added in opportunity |
| Task/Event | Activities/calls/meetings |
| Campaign | Marketing initiative |
| Quote | Sales quotation |
| Case | Customer support issue |

---

# 3. LEAD CREATION – Complete Internal Flow

# 3.1 What is a Lead?

A Lead represents:

- Someone who showed interest
- Unknown qualification status
- Not yet a customer
- Not yet associated with a proper Account

Examples:

- Website form submission
- Marketing event attendee
- Purchased email list
- Manual sales entry
- API integration record

---

# 3.2 Ways Lead Can Be Created

| Method | Description |
|---|---|
| Manual UI | User creates lead manually |
| Web-to-Lead | Salesforce auto creates lead from website |
| API Integration | External system inserts lead |
| Data Loader | Bulk import |
| Apex | Custom code inserts lead |
| Flow | Record-triggered flow creates lead |
| Einstein/Marketing Cloud | Marketing automation |

---

# 3.3 Database Record Creation

When Lead is created:

```text
Lead Record Inserted
Table/Object: Lead
Primary Key: Lead.Id
```

Example:

```text
Lead
 ├── Id = 00Qxx000000123A
 ├── FirstName = John
 ├── LastName = Doe
 ├── Company = ABC Corp
 ├── Status = Open
 └── OwnerId = User/Queue
```

---

# 3.4 Automation Execution During Lead Insert

Salesforce executes automation in a specific order.

# ORDER OF EXECUTION (IMPORTANT)

When a Lead is inserted:

```text
1. System Validations
2. Before-Save Record Triggered Flows
3. Before Insert Apex Trigger
4. Custom Validation Rules
5. Duplicate Rules
6. Database Save (Not committed yet)
7. After Insert Apex Trigger
8. Assignment Rules
9. Auto-response Rules
10. Workflow Rules
11. Process Builder
12. After-Save Flows
13. Escalation Rules
14. Roll-up calculations
15. Commit to database
16. Post-commit logic (emails/events/platform events)
```

---

# 3.5 Typical Lead Triggers

Example:

```apex
trigger LeadTrigger on Lead (
    before insert,
    before update,
    after insert,
    after update
) {

}
```

Typical work inside trigger:

| Trigger Type | Purpose |
|---|---|
| before insert | Normalize fields |
| before insert | Generate custom lead score |
| after insert | Create tasks |
| after update | Notify sales reps |
| after update | Sync external systems |

---

# 3.6 Common Lead Flows

## Before Save Flow

Used for:

- Updating field values
- Fast field calculations
- Formatting phone/email

Runs before database save.

---

## After Save Flow

Used for:

- Creating related records
- Sending emails
- Calling Apex
- Posting Slack notifications
- Creating tasks

---

# 3.7 Lead Assignment Rule

After insert, Salesforce may assign lead automatically.

Example:

```text
IF Country = India
THEN assign to India Sales Queue
```

This updates:

```text
Lead.OwnerId
```

---

# 3.8 Lead Status Lifecycle

Typical statuses:

```text
Open
Working
Qualified
Nurturing
Converted
Disqualified
```

---

# 4. LEAD QUALIFICATION STAGE

# 4.1 What Happens During Qualification?

Sales rep verifies:

- Budget
- Authority
- Need
- Timeline
- Valid customer
- Genuine interest

This is business logic, but Salesforce automation usually supports it.

---

# 4.2 Activities Created During Qualification

Common related records:

| Record | Linked Through |
|---|---|
| Task | WhoId = Lead.Id |
| Event | WhoId = Lead.Id |
| CampaignMember | LeadId |
| Notes | ParentId |
| EmailMessage | RelatedToId |

---

# 4.3 Relationship Example

```text
Lead (00Q)
   ├── Tasks
   ├── Events
   ├── Campaign Memberships
   ├── Notes
   └── Attachments
```

---

# 5. LEAD CONVERSION – MOST IMPORTANT PART

# 5.1 What is Lead Conversion?

Lead conversion transforms an unqualified Lead into:

- Account
- Contact
- Opportunity (optional)

Lead itself is NOT deleted.

Instead:

```text
Lead.IsConverted = true
```

---

# 5.2 What Records Are Created?

During conversion:

| Object | Created? |
|---|---|
| Account | Yes or Existing Account used |
| Contact | Yes |
| Opportunity | Optional |
| OpportunityContactRole | Often created |

---

# 5.3 Internal Salesforce Conversion Process

When user clicks:

```text
Convert Lead
```

Salesforce internally executes:

```apex
Database.convertLead()
```

or internal equivalent.

---

# 5.4 Internal Mapping Logic

Lead fields are mapped into:

| Lead Field | Target Object |
|---|---|
| Company | Account.Name |
| FirstName | Contact.FirstName |
| LastName | Contact.LastName |
| Email | Contact.Email |
| Phone | Contact.Phone |

Custom fields use Lead Field Mapping settings.

---

# 5.5 Detailed Record Linking After Conversion

Suppose:

```text
Lead Id = 00Q100
```

Conversion creates:

```text
Account Id = 001100
Contact Id = 003100
Opportunity Id = 006100
```

Lead now stores:

```text
Lead.ConvertedAccountId = 001100
Lead.ConvertedContactId = 003100
Lead.ConvertedOpportunityId = 006100
Lead.IsConverted = true
```

---

# 5.6 Important Relationship Structure

```text
Account
   ├── Contacts
   │      └── Opportunities (through roles)
   ├── Opportunities
   ├── Cases
   ├── Contracts
   └── Assets
```

---

# 5.7 Automation During Lead Conversion

This is VERY important.

During conversion, Salesforce performs inserts/updates on multiple objects.

# Actual Internal Sequence

```text
1. Validate Lead
2. Create/Match Account
3. Create Contact
4. Create Opportunity
5. Update Lead (IsConverted=true)
6. Re-parent activities
7. Commit transaction
```

---

# 5.8 Which Triggers Fire During Conversion?

# Lead Triggers

```text
before update
after update
```

because Lead gets updated.

---

# Account Triggers

If new account created:

```text
before insert
after insert
```

If existing account used:

```text
before update
after update
```

---

# Contact Triggers

```text
before insert
after insert
```

---

# Opportunity Triggers

```text
before insert
after insert
```

---

# Task/Event Triggers

Activities may get updated/re-parented.

---

# 5.9 Flows Triggered During Conversion

Any record-triggered flow on:

- Lead
- Account
- Contact
- Opportunity
- Task

may execute.

This can create cascading automation.

---

# 5.10 Activity Re-parenting

Before conversion:

```text
Task.WhoId = Lead.Id
```

After conversion:

```text
Task.WhoId = Contact.Id
```

This is automatic.

---

# 5.11 Campaign Member Conversion

Campaign history is preserved.

Lead campaign memberships become Contact campaign memberships.

---

# 6. ACCOUNT CREATION STAGE

# 6.1 What is an Account?

Account represents:

- Company
- Customer organization
- Business entity

---

# 6.2 Record Relationships

```text
Account
   ├── Contacts
   ├── Opportunities
   ├── Cases
   ├── Orders
   ├── Assets
   └── Contracts
```

---

# 6.3 Account Record Example

```text
Account
 ├── Id = 001xx
 ├── Name = ABC Corp
 ├── OwnerId
 ├── Industry
 └── BillingAddress
```

---

# 6.4 Typical Account Automations

| Automation | Purpose |
|---|---|
| Before Trigger | Clean account names |
| After Trigger | Create onboarding tasks |
| Flow | Enrichment APIs |
| Validation Rule | Prevent bad data |
| Duplicate Rule | Avoid duplicate companies |

---

# 7. CONTACT CREATION STAGE

# 7.1 What is a Contact?

Contact represents:

- Individual person
- Linked to Account

---

# 7.2 Relationship Structure

```text
Contact
   ├── AccountId → Account
   ├── Cases
   ├── Activities
   └── Opportunities (through OCR)
```

---

# 7.3 Contact Example

```text
Contact
 ├── Id = 003xx
 ├── FirstName
 ├── LastName
 ├── Email
 └── AccountId = 001xx
```

---

# 8. OPPORTUNITY CREATION STAGE

# 8.1 What is an Opportunity?

Opportunity represents:

- Revenue deal
- Sales pipeline record
- Potential business

---

# 8.2 Opportunity Relationships

```text
Opportunity
   ├── AccountId
   ├── OpportunityLineItems
   ├── Quotes
   ├── Tasks
   ├── Events
   └── Contact Roles
```

---

# 8.3 Opportunity Example

```text
Opportunity
 ├── Id = 006xx
 ├── Name = ABC Software Deal
 ├── StageName = Prospecting
 ├── CloseDate
 ├── Amount
 └── AccountId = 001xx
```

---

# 8.4 Opportunity Creation Sources

| Source | Description |
|---|---|
| Lead Conversion | Most common |
| Manual Creation | Sales rep creates |
| API Integration | ERP/CPQ |
| Flow | Automation |
| Apex | Custom logic |

---

# 8.5 Opportunity Triggers

Typical logic:

| Trigger | Use Case |
|---|---|
| before insert | Default stage/amount |
| after insert | Create quote |
| before update | Validate stage movement |
| after update | Sync ERP |

---

# 8.6 Common Opportunity Flows

| Flow Type | Example |
|---|---|
| Before Save | Set probability |
| After Save | Create tasks |
| Scheduled Flow | Follow-up reminders |
| Approval Flow | Discount approvals |

---

# 9. OPPORTUNITY SALES STAGES

Typical pipeline:

```text
Prospecting
Qualification
Needs Analysis
Proposal
Negotiation
Closed Won
Closed Lost
```

Each stage may trigger:

- Validation rules
- Approval processes
- Flows
- Apex triggers
- Integrations

---

# 10. OPPORTUNITY PRODUCTS

# 10.1 OpportunityLineItem Creation

Products added to Opportunity create:

```text
OpportunityLineItem
```

Relationship:

```text
Opportunity
   └── OpportunityLineItems
```

---

# 10.2 Required Records

To create Opportunity Product:

```text
Product2
Pricebook2
PricebookEntry
OpportunityLineItem
```

---

# 10.3 Product Relationship Chain

```text
Product2
   ↓
PricebookEntry
   ↓
OpportunityLineItem
   ↓
Opportunity
```

---

# 11. QUOTES AND CPQ

Sometimes:

```text
Opportunity
   └── Quote
```

Quote may generate:

- PDF proposal
- Pricing details
- Discount approvals

---

# 12. APPROVAL PROCESSES

Often triggered during:

- Discount approval
- Contract approval
- High-value deal approval

Approval process creates:

```text
ProcessInstance
ProcessInstanceStep
```

---

# 13. INTEGRATIONS DURING SALES PROCESS

Typical integrations:

| System | Purpose |
|---|---|
| ERP | Order sync |
| Marketing Cloud | Campaign sync |
| SAP | Financial data |
| DocuSign | Contract signing |
| Slack | Notifications |

Usually called via:

- Apex callouts
- Platform Events
- Flows
- MuleSoft
- CDC events

---

# 14. OPPORTUNITY CLOSE PROCESS

# 14.1 Closed Won

When:

```text
Opportunity.StageName = Closed Won
```

Typical automations:

| Automation | Purpose |
|---|---|
| Create Contract | Legal agreement |
| Create Order | Fulfillment |
| Create Project | Delivery |
| Send Welcome Email | Customer onboarding |
| ERP Sync | Revenue booking |

---

# 14.2 Closed Lost

Common actions:

- Capture loss reason
- Notify manager
- Add to nurture campaign
- Create follow-up reminder

---

# 15. ORDER OF EXECUTION FOR OPPORTUNITY UPDATE

Example:

```text
User changes Stage = Closed Won
```

Salesforce executes:

```text
1. System Validation
2. Before Save Flows
3. Before Update Trigger
4. Validation Rules
5. Duplicate Rules
6. Save Record
7. After Update Trigger
8. Assignment Rules (if applicable)
9. Auto-response
10. Workflow Rules
11. Process Builder
12. After Save Flow
13. Rollups
14. Commit
15. Async jobs/future/queueable/platform events
```

---

# 16. IMPORTANT RECORD LINKING SUMMARY

# Lead to Account

```text
Lead.ConvertedAccountId → Account.Id
```

---

# Lead to Contact

```text
Lead.ConvertedContactId → Contact.Id
```

---

# Lead to Opportunity

```text
Lead.ConvertedOpportunityId → Opportunity.Id
```

---

# Contact to Account

```text
Contact.AccountId → Account.Id
```

---

# Opportunity to Account

```text
Opportunity.AccountId → Account.Id
```

---

# Opportunity to Contact

Indirect via:

```text
OpportunityContactRole
```

---

# Tasks Relationship

```text
Task.WhoId = Contact/Lead
Task.WhatId = Opportunity/Account
```

---

# 17. REAL END-TO-END EXAMPLE

# Step 1 – Lead Created

```text
Lead:
John Doe
ABC Corp
```

Triggers:

- Lead before insert
- Lead after insert
- Lead flows
- Assignment rules

---

# Step 2 – Sales Rep Qualifies

Sales rep updates:

```text
Lead.Status = Qualified
```

Triggers:

- before update
- after update
- flows

---

# Step 3 – Convert Lead

Salesforce creates:

```text
Account = ABC Corp
Contact = John Doe
Opportunity = ABC Corp Deal
```

Triggers fired:

```text
Lead update triggers
Account insert triggers
Contact insert triggers
Opportunity insert triggers
```

---

# Step 4 – Opportunity Progress

Stages:

```text
Prospecting → Proposal → Negotiation
```

Automations:

- Tasks
- Approvals
- Notifications
- Product additions

---

# Step 5 – Closed Won

Automations:

```text
Contract Created
Order Created
ERP Synced
Welcome Email Sent
```

---

# 18. ASYNCHRONOUS OPERATIONS

Many operations happen asynchronously.

Examples:

| Async Type | Purpose |
|---|---|
| Future Method | API callouts |
| Queueable | Complex processing |
| Batch Apex | Large data updates |
| Scheduled Apex | Timed operations |
| Platform Events | Event-driven integration |

---

# 19. COMMON INTERVIEW QUESTIONS

# Q1. Which triggers fire during Lead Conversion?

Answer:

```text
Lead update
Account insert/update
Contact insert
Opportunity insert
Task/Event update
```

---

# Q2. Is Lead deleted after conversion?

No.

```text
IsConverted = true
```

---

# Q3. How are activities transferred?

Automatically re-parented from Lead to Contact/Account/Opportunity.

---

# Q4. Can Opportunity creation be skipped?

Yes.

During conversion user may choose:

```text
Do not create opportunity
```

---

# Q5. Which object links Contact and Opportunity?

```text
OpportunityContactRole
```

---

# 20. COMPLETE TECHNICAL FLOW DIAGRAM

```text
Lead Created
   ↓
Lead Before Save Flow
   ↓
Lead Before Trigger
   ↓
Validation Rules
   ↓
Lead Saved
   ↓
Lead After Trigger
   ↓
Assignment Rules
   ↓
After Save Flow
   ↓
Lead Qualified
   ↓
Lead Conversion
   ├── Account Created/Matched
   ├── Contact Created
   ├── Opportunity Created
   ├── Activities Re-parented
   └── Lead Updated
            ↓
Opportunity Pipeline
   ├── Products
   ├── Quotes
   ├── Approvals
   ├── Integrations
   └── Tasks
            ↓
Closed Won/Lost
            ↓
Contract / Order / ERP / Delivery
```

---

# 21. MOST IMPORTANT SALESFORCE CONCEPTS TO REMEMBER

# Lead

Unqualified person/company.

---

# Conversion

Transforms Lead into:

- Account
- Contact
- Opportunity

---

# Account

Company/customer.

---

# Contact

Person associated with Account.

---

# Opportunity

Revenue deal.

---

# Trigger Timing

- Before Trigger → modify values before save
- After Trigger → create related records/integrations

---

# Flow Timing

- Before Save Flow → fast field updates
- After Save Flow → actions and related records

---

# Key Relationship Fields

| Relationship | Field |
|---|---|
| Contact → Account | AccountId |
| Opportunity → Account | AccountId |
| Lead → Converted Account | ConvertedAccountId |
| Lead → Converted Contact | ConvertedContactId |
| Lead → Converted Opportunity | ConvertedOpportunityId |

---

# 22. ADVANCED ENTERPRISE FLOW (REAL PROJECTS)

Large organizations often add:

```text
Lead
 ↓
Lead Scoring Engine
 ↓
Marketing Qualification
 ↓
Sales Qualification
 ↓
Territory Assignment
 ↓
Duplicate Management
 ↓
Account Matching
 ↓
Lead Conversion
 ↓
CPQ
 ↓
Approvals
 ↓
Contract Lifecycle
 ↓
ERP Sync
 ↓
Invoice
 ↓
Renewal Opportunity
```

This can involve:

- Apex
- Platform Events
- CDC
- Integration middleware
- OmniStudio
- Einstein
- Flow Orchestrator
- External services

---

# 23. IMPORTANT DEVELOPMENT BEST PRACTICES

# Avoid Trigger Recursion

Use static variables.

---

# Keep Logic Out of Triggers

Use:

- Handler classes
- Service layer
- Selector layer

---

# Prefer Flows for Simple Automation

Use Apex only when needed.

---

# Bulkification

Always process:

```apex
List<Lead>
```

not single records.

---

# Handle Conversion Carefully

Lead conversion fires many automations simultaneously.

Avoid:

- SOQL in loops
- DML in loops
- recursive updates

---

# 24. FINAL COMPLETE OBJECT RELATIONSHIP MAP

```text
Lead
 ├── Tasks
 ├── Events
 ├── CampaignMembers
 └── Converts Into
        ↓
     Account
        ├── Contacts
        │      ├── Tasks
        │      └── OpportunityContactRoles
        ├── Opportunities
        │      ├── Products
        │      ├── Quotes
        │      ├── Tasks
        │      ├── Orders
        │      └── Contracts
        ├── Cases
        └── Assets
```

---

# 25. FINAL SUMMARY

The complete lifecycle is:

```text
1. Lead Created
2. Automation Executes
3. Lead Qualified
4. Lead Converted
5. Account Created/Matched
6. Contact Created
7. Opportunity Created
8. Products/Quotes Added
9. Approvals Executed
10. Opportunity Closed Won/Lost
11. Contracts/Orders/ERP Sync
```

During this process Salesforce executes:

- Flows
- Triggers
- Validation Rules
- Assignment Rules
- Workflow Rules
- Approval Processes
- Async Jobs
- Integrations

All records remain connected using:

- Lookup relationships
- Master-detail relationships
- Standard relationship fields
- Conversion IDs

