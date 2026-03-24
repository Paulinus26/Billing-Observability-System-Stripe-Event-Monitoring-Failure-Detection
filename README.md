# Billing-Observability-System-Stripe-Event-Monitoring-Failure-Detection
1. Overview

In a SaaS environment, billing failures directly affect revenue and customer access. This project focuses on building a system that provides visibility into payment events and supports structured investigation of billing issues.

The Billing Observability System captures real-time events from Stripe and stores them in Supabase for analysis. It also delivers real-time alerts via Slack to support faster response.

Using SQL queries, the system enables support engineers to detect failed payments, trace customer billing activity, and identify root causes.

This project simulates a real-world SaaS support environment by combining event ingestion, alerting, and data-driven investigation workflows.

2. System Architecture

The system follows an event-driven architecture designed to capture and operationalize billing events.

Flow

Stripe → Webhook (Node.js) → Supabase → SQL Analysis
                     ↓
                   Slack (Alerts)

Component Breakdown
Stripe
Generates billing events such as successful payments and failures.
Node.js Webhook
Receives incoming events, extracts relevant billing data, stores them in the database, and triggers alerts.
Supabase
Stores structured billing events for querying and investigation.
Slack
Provides real-time alerts for failed payments, enabling faster support response.
3. Technology Stack
Stripe — billing event source
Supabase — database and analysis
Node.js (Express) — webhook handler
Slack Webhooks — alerting layer
Stripe CLI — local event forwarding
4. System Design

The system is built around event-driven ingestion.

Billing systems operate asynchronously. Payment outcomes such as failures, retries, or successful charges occur independently of user actions. This system captures those events in real time and persists them for analysis.

Each event is stored with:

event type
customer identifier
status
error message
timestamp

This structure allows reconstruction of billing timelines and supports detailed investigation.

Alerting Layer

In addition to storing events, the system sends real-time notifications to Slack when critical billing failures occur.

This enables:

Immediate visibility into failed payments
Faster support response
Reduced dependency on manual log inspection
5. Implementation
Phase 1 — Setup
Created a Stripe account in Test Mode
Retrieved API keys (publishable and secret)
Created a Supabase project
Defined the billing_events table
CREATE TABLE billing_events (
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
Phase 2 — Webhook System
Initialized Node.js project
Installed dependencies (express, stripe, supabase client, body-parser)
Created a webhook endpoint

The webhook:

Receives Stripe events
Extracts relevant billing data
Inserts records into Supabase
Sends alerts to Slack for failed payments
Phase 3 — Stripe Integration
Installed Stripe CLI
Authenticated locally
Forwarded events to the webhook endpoint
stripe listen --forward-to localhost:3000/webhook

This enables real-time event ingestion during development.

Phase 4 — Event Simulation
Created product and pricing in Stripe
Created test customer
Created subscription

Used test cards:

Success: 4242 4242 4242 4242
Failure: 4000 0000 0000 0002

Generated events:

invoice.paid
invoice.payment_failed
Phase 5 — System Validation

Verified system functionality by:

Checking the billing_events table in Supabase
Confirming both successful and failed events were stored
Validating that Slack alerts were triggered for failed payments
Phase 6 — Investigation & Analysis

Used SQL queries to analyze billing events.

Failed payments
SELECT * FROM billing_events
WHERE event_type = 'invoice.payment_failed';
Revenue at risk
SELECT SUM(amount)
FROM billing_events
WHERE status = 'failed';
Customer timeline
SELECT *
FROM billing_events
WHERE customer_id = 'cus_xxx'
ORDER BY created_at;

These queries support root cause analysis and customer-level debugging.

6. Debugging & Investigation Workflow

The system supports a structured investigation process:

Detect failed payment event (via Slack or logs)
Query billing_events for failure details
Review error_message field
Trace customer billing history
Identify failure pattern
Determine action:
retry payment
notify customer
escalate issue

This workflow reflects real SaaS support operations.

7. Observations & Insights
Card declines were the most common failure type
Repeated failures were observed for the same customer
Failed transactions represent potential revenue loss
8. Business Impact
Improves visibility into billing failures
Enables faster incident detection through Slack alerts
Reduces investigation time using SQL-based workflows
Supports revenue recovery efforts
Strengthens collaboration between support and engineering
9. Key Features
Real-time webhook ingestion
Centralized billing event storage
SQL-based investigation
Failure detection and analysis
Slack alerts for failed payment events
10. Conclusion

This project demonstrates the design and implementation of a billing observability system in a SaaS environment.

It highlights:

Support engineering capability
Debugging and investigation skills
Practical understanding of event-driven systems
Ability to resolve real billing issues using data
