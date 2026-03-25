# Billing-Observability-System-Stripe-Event-Monitoring-Failure-Detection
1. Overview

In a modern SaaS environment, billing is the heartbeat of the business. When payments fail, it’s not just a technical error,it’s a disruption in service and a direct loss of revenue. This Billing Observability System was engineered to provide real-time visibility into the subscription lifecycle.

By capturing asynchronous events from Stripe and centralizing them into a queryable database while simultaneously alerting the team via Slack, this system transforms raw payment data into actionable intelligence. It allows support engineers to detect failures instantly, investigate root causes via SQL, and mitigate revenue leakage before it impacts the customer experience.

2. System Architecture

The architecture is built on an event-driven model, ensuring that every state change in a customer's subscription is recorded and processed.

The Operational Flow:

Stripe (Event Source) → Webhook (Node.js Handler) → Supabase & Slack → SQL Analysis

• Stripe: Acts as the primary source of truth for all financial transactions.

• Node.js Webhook: A lightweight middleware that ingests real-time POST requests, extracts metadata, and routes them to storage and communication channels.

• Slack Integration: Provides immediate, human-readable alerts for incident response.

• Supabase (PostgreSQL): A relational database that stores high-fidelity logs for long-term audit.

3. Technology Stack

• Stripe API: Managed billing infrastructure and event generation.

• Node.js & Express: Environment for handling concurrent webhook requests.

• Slack Webhook API: Instant notification layer for support teams.

• Supabase/PostgreSQL: Scalable relational storage for event querying.

• Stripe CLI: Developer tooling for local event tunneling.

4. System Design

This system utilizes Webhook-based Ingestion to handle the asynchronous nature of SaaS payments. Unlike a synchronous API call, billing events (like a card decline) happen outside the user's immediate session.

The design ensures Multi-Channel Delivery:

• Immediate Action: Slack alerts for the Support team to handle urgent customer outreach.

• Historical Record: Database entries for the Engineering and Finance teams to track trends.

5. Implementation

Phase 1 — Infrastructure Setup

The foundation requires a synchronized environment between the payment processor and the database.

• Stripe Configuration: Initialized in Test Mode to simulate transactions.

• Database Schema: Created the billing_events table to capture context including error messages and status.
![Figure 1](https://github.com/user-attachments/assets/e67d710c-c548-4c4a-a4da-f86e89707129)

Figure 1: Database Schema Initialization

```SQL
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
```
Image Placement: [Screenshot of the Supabase SQL Editor showing successful execution of the billing_events table creation script.]

Phase 2 — Webhook & Notification Logic
The Node.js handler distills dense JSON payloads into clean data for both Slack and Supabase.

Core Webhook Handler:

```JavaScript
app.post('/webhook', async (req, res) => {
  const event = req.body;
  const data = event.data.object;

  // Data Normalization (Cents to Dollars + Fallback Naming)
  const amount = (data.amount_due || data.amount || 0) / 100;
  const customerName = data.customer_name || "Harry Joe (Client)";

  // 1. Log to Supabase
  await supabase.from('billing_events').insert([{
    event_type: event.type,
    customer_id: customerName,
    amount: amount,
    error_message: data.last_payment_error?.message || null,
    created_at: new Date(event.created * 1000)
  }]);

  // 2. Alert Slack
  await axios.post(SLACK_WEBHOOK_URL, {
    text: `🚨 *Billing Failure Alert*\n*Customer:* ${customerName}\n*Amount:* $${amount}\n*Status:* Failed`
  });

  res.sendStatus(200);
});
```
Figure 2: Data Normalization and Multi-Channel Logic

Image Placement: [Screenshot of the index.js code in Notepad showing the logic that fixes the amount decimal error and sets the fallback customer name.]

Phase 3 — Stripe Integration & Tunneling
Using the Stripe CLI to forward cloud events to the local server mirrors a production load balancer routing traffic to microservices.

Figure 3: System Ingestion Pipeline Verification

Image Placement: [Screenshot of the terminal showing the Stripe CLI "Ready" message and successfully forwarding events to localhost:3000/webhook.]

Phase 4 — Operational Simulation

Validated the system using standardized test cards to trigger specific response paths.

Success: Used card 4242...

Failure: Used card 4000...0002

Figure 4: Simulating SaaS Billing Events

Image Placement: [Screenshot of the Stripe Dashboard showing the Customers list with Harry Joe and Paul Chidi profiles.]

Phase 5 — System Validation
Verification is confirmed when the Stripe event is "materialized" in both the database and the Slack channel simultaneously.

Figure 5: Multi-Channel Success Confirmation

Image Placement: [Screenshot showing a side-by-side view: The Slack notification for Harry Joe and the corresponding new row in the Supabase Table Editor.]

6. Investigation & Analysis

In a professional support scenario, we use SQL to move from "something is wrong" to "here is exactly why."

Detecting Revenue at Risk: Identifying the total volume of failed payments in a specific period.

Example Query:

```SQL
SELECT SUM(amount) FROM billing_events WHERE event_type = 'invoice.payment_failed';
```

7. Debugging & Investigation Workflow

When a billing incident occurs, the following SOP (Standard Operating Procedure) is followed:

Detection: A Slack alert provides immediate situational awareness.

Querying: Access billing_events in Supabase to find the specific payment_intent_id.

Identification: Read the error_message provided by the downstream bank to identify the root cause (e.g., Insufficient Funds).

Resolution: Determine if the issue is a technical timeout or a financial decline and take action.

8. Business Impact

Revenue Recovery: Instant detection allows for faster outreach, reducing churn.

Operational Efficiency: Support teams investigate issues independently without engineering intervention.

System Transparency: Provides a clear audit trail for financial troubleshooting and compliance.

9. Conclusion

This project demonstrates the ability to build scalable monitoring tools, handle complex API integrations, and use data-driven methodologies to solve critical SaaS business problems.

Figure 6: Final Production Alert View

Image Placement: [Screenshot of the Slack app showing the final, professional notification including the $20.00 amount and the customer name Harry Joe (Client).]
