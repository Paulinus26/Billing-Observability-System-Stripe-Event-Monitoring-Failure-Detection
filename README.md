# Billing-Observability-System-Stripe-Event-Monitoring-Failure-Detection
1. Overview

In a SaaS environment, billing failures directly impact revenue and customer access. This project implements a billing observability system that provides visibility into payment events and supports structured investigation of failures.

The system captures asynchronous events from Stripe, stores them in Supabase, and sends real-time alerts via Slack. Using SQL queries, support engineers can investigate failures, trace customer activity, and determine root causes.

This project reflects a real-world support engineering workflow: detection, investigation, and resolution of billing issues.


2. System Architecture

Operational Flow

Stripe → Webhook (Node.js) → Supabase → SQL Analysis
             
             ↓  
        
        Slack (Alerts)
                   

Architecture Diagram
![figure 1](https://github.com/user-attachments/assets/85d2ebee-29fe-49db-ab71-5dd2ff5d0366) 
Figure 1: Event-driven architecture showing Stripe events flowing through a Node.js webhook into Supabase, with Slack alerts for failed payments

Component Breakdown

• Stripe
Generates billing events such as invoice.paid and invoice.payment_failed.

• Node.js Webhook
Receives events, extracts relevant data, stores them in the database, and triggers alerts.

• Supabase
Stores structured billing events for querying and investigation.

• Slack
Provides real-time alerts for failed payment events


3. Technology Stack

• Stripe — billing event source

• Supabase — database and analysis

• Node.js (Express) — webhook handler

• Slack Webhooks — alerting layer

• Stripe CLI — local event forwarding


4. System Design

The system is built around event-driven ingestion.

Billing systems operate asynchronously. Events such as payment failures or retries occur independently of user interaction. This system captures those events in real time and stores them for analysis.

Data Model

Each event includes:

• event type

• customer ID

• status

• amount

• currency

• error message

• timestamp

```CREATE TABLE billing_events (
  id uuid DEFAULT gen_random_uuid() PRIMARY KEY,
  event_type text,
  customer_id text,
  status text,
  amount numeric,
  currency text,
  error_message text,
  payment_intent_id text,
  invoice_id text,
  created_at timestamp
);
```
![Figure 1](https://github.com/user-attachments/assets/e67d710c-c548-4c4a-a4da-f86e89707129)
Figure 2: Supabase table capturing Stripe billing events.


5. Implementation

Phase 1 — Setup

• Created Stripe account (Test Mode)

• Retrieved API keys

• Created Supabase project

• Created billing_events table

Phase 2 — Webhook System

• Initialized Node.js project

• Installed dependencies

• Built webhook endpoint

The webhook:

• receives Stripe events

• extracts relevant fields

• stores data in Supabase

• triggers Slack alerts

![Figure 2](https://github.com/user-attachments/assets/36a85553-328f-4ef9-9481-1f8e62fec1bf)
Figure 3: Webhook processing flow for event ingestion and alerting.

Phase 3 — Stripe Integration

• Installed Stripe CLI

• Authenticated locally

• Forwarded events to local server
```
stripe listen --forward-to localhost:3000/webhook
```
![figure 4](https://github.com/user-attachments/assets/27f5ac65-969b-4d55-90c5-5f041e829d55)
Figure 4: System Ingestion Pipeline Verification

Phase 4 — Event Simulation

Used test cards to simulate real billing outcomes:

Success: 4242 4242 4242 4242
Failure: 4000 0000 0000 0002

Generated events:

invoice.paid
invoice.payment_failed
![figure ](https://github.com/user-attachments/assets/8aff07a9-eb6b-401f-ad8a-625bd8c75fe8)
Figure 5: Simulating SaaS Billing Events

Phase 5 — System Validation

Confirmed system functionality by:

• confirming Slack alerts for failed payments

• verifying records in Supabase

![figure 5](https://github.com/user-attachments/assets/fd678ac3-43f9-4f5a-8818-f5e98e2e1eb9)
![figure b](https://github.com/user-attachments/assets/0e0ea120-239c-41c9-bcd2-4bd323b13ced)
Figure 6: Billing events stored and alerts triggered.


6. Investigation & Analysis

SQL queries are used to investigate billing issues.

Revenue at Risk
```
SELECT SUM(amount)
FROM billing_events
WHERE event_type = 'invoice.payment_failed';
```
![figure 6](https://github.com/user-attachments/assets/96c76b40-c21c-4aff-b568-320b5bad8a59)  
Figure 7: Query result showing failed payment totals.


7. Debugging & Investigation Workflow

This system supports a structured support workflow:

1. Detection
Slack alert signals a failed payment

2. Query
Retrieve event data from Supabase

3. Identification
Review error message and payment status

4. Analysis
Trace customer billing history

5. Resolution

• retry payment

• notify customer

• escalate if needed
![figure 7](https://github.com/user-attachments/assets/48296d54-aa0f-43aa-a1e3-05168c8850b6) 
Figure 8: Deep-Dive Investigation


9. Business Impact

• Enables real-time detection of billing failures

• Reduces response time through Slack alerts

• Supports independent investigation using SQL

• Improves revenue recovery workflows

• Provides audit trail for billing issues


10. Conclusion

This project demonstrates the ability to build a billing observability system using real SaaS tools.

It highlights:

• event-driven system design

• webhook integration

• debugging and investigation workflows

• data-driven problem solving
