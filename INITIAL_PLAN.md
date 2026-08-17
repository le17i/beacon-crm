# Beacon CRM — Initial Product Spec & Roadmap



> **Slogan:** Clarity in Pipeline & Developer-First Automation
> 
> 
> **Positioning:** Open-source, ultra-fast, API-first CRM with a modern interface for freelancers, startups, and SMBs looking to manage opportunities without the complexity of legacy CRMs.
> 
> 

---

## 1. Product Overview & Objectives



The **Beacon CRM** was conceived with two main goals:

1. **Product (Value Proposition):** Act as a beacon in the sales flow, providing real-time visibility into leads, pipeline health, and frictionless automation.


2. **Portfolio (Engineering & Product):** Demonstrate mastery of modern architecture (API-first, webhooks, frontend reactivity, resilient data modeling) and strategic product thinking (*Build in Public*).



---

## 2. Personas & Use Cases



* **Persona 1: Developer / Solopreneur**

* *Need:* Capture leads from custom forms/landing pages directly into a Kanban board without setting up heavy CRMs.




* **Persona 2: Small Agency / Startup**

* *Need:* Visualize active deal values, track conversation history, and trigger automations when a contract closes.





---

## 3. Release Matrix (Initial Roadmap)



| Phase | Release Scope | Primary Focus | Success Metrics (KPIs) |
| --- | --- | --- | --- |
| **MVP (v0.1)** | Epics 1 & 2 | Kanban board, lead ingestion via public API, and stage management | • Onboarding < 3 minutes<br>

<br>• API ingestion success = 100% |
| **Release v1.0** | Epic 3 | Lead profile, interaction timeline, and custom fields | • Weekly dashboard retention |
| **Release v1.1** | Epics 4 & 5 | Analytics Dashboard and outgoing Webhooks engine (Triggers) | • Webhook delivery success > 99.5% |

---

## 4. Epics & User Stories Mapping



### Epic 1: Visual Pipeline Management (Core Pipeline)



* **Objective:** Provide immediate visual control over the sales flow with simple drag-and-drop movement and real-time calculations.


* **Features & User Stories:**

* **US1.1 — Kanban View:** As a user, I want to view my leads organized by columns (stages) to understand the status of each opportunity.


* **US1.2 — Drag & Drop:** As a user, I want to drag a lead card between columns to update its stage instantly.


* **US1.3 — Column Metrics:** As a user, I want to see the total lead count and total monetary value at the top of each column.


* **US1.4 — Stage Customization:** As a user, I want to create, reorder, edit, and delete pipeline columns.





### Epic 2: Lead Ingestion & API Open-First



* **Objective:** Allow automated lead entry from any external source (landing pages, forms, webhooks).


* **Features & User Stories:**

* **US2.1 — API Key Management:** As a user, I want to generate and revoke API keys with specific permissions to integrate my applications.


* **US2.2 — Capture Endpoint (`POST /api/v1/leads`):** As an external system, I want to send a JSON payload with lead data so it automatically lands in the first column of the pipeline.


* **US2.3 — Basic Deduplication:** As a system, I want the CRM to identify if a lead (by email or phone) already exists, updating the record or flagging duplicates.





### Epic 3: Context & Timeline (Lead Journey)



* **Objective:** Centralize all customer information and history into a single, context-rich page.


* **Features & User Stories:**

* **US3.1 — Detailed Lead Profile:** As a user, I want to open a lead's page and view contact details, estimated deal value, source, and status.


* **US3.2 — Unified Event Timeline:** As a user, I want to see in chronological order: stage changes, manual notes, and completed tasks.


* **US3.3 — Custom Fields:** As a user, I want to create custom fields (e.g., *Budget*, *Tech Stack*, *Job Title*) to adapt the CRM to my business.





### Epic 4: Commercial Visibility & Analytics



* **Objective:** Turn raw lead data into strategic intelligence for decision-making.


* **Features & User Stories:**

* **US4.1 — Key KPIs:** As a manager, I want to view in real time: Global Conversion Rate (%), Total Pipeline Value ($), and Average Deal Size ($).


* **US4.2 — Conversion Funnel & Bottlenecks:** As a manager, I want a drop-off chart showing transition rates between stages to spot bottlenecks.


* **US4.3 — Time & Source Filters:** As a manager, I want to filter the dashboard by timeframe (30 days, quarter, year) and acquisition channel.





### Epic 5: Automation Engine & Outgoing Webhooks



* **Objective:** React to CRM events and connect the system to external tools (Slack, n8n, email marketing, PostgreSQL).


* **Features & User Stories:**

* **US5.1 — Trigger Rules:** As a user, I want to set up rules like: *"When a lead is moved to 'Proposal Sent', send a webhook to URL X"*.


* **US5.2 — Asynchronous / Guaranteed Delivery:** As a system, I want to guarantee delivery of the JSON payload to the registered URL with automatic retries on failure.


* **US5.3 — Webhook Audit Logs:** As a user, I want to see a table of triggered events, HTTP status codes (200, 500, etc.), and a manual retry button.





---

## 5. Suggested Technical Architecture



* **Frontend:** React / Next.js, Tailwind CSS, `@dnd-kit` (for reactive, accessible Kanban), Lucide Icons.


* **Backend:** Node.js / TypeScript (Hono or Fastify) — lightweight and ultra-fast for API endpoints and webhooks.


* **Database:** PostgreSQL / Supabase (Tables: `organizations`, `pipelines`, `stages`, `leads`, `activity_logs`, `api_keys`, `webhooks`).


* **Jobs / Queues:** Redis / BullMQ or pg-boss for resilient processing of outgoing webhooks.



---

## 6. "Build in Public" & Marketing Strategy



1. **Post 1 — Announcing the Concept:**

* *Angle:* "Why I'm building Beacon CRM: solving a real problem with open-source architecture."


* *Content:* Concept preview, Kanban prototype, and GitHub repository link.




2. **Post 2 — Lead Ingestion Architecture:**

* *Angle:* "Designing a secure, resilient API endpoint to ingest thousands of form webhooks."


* *Content:* Hono/TypeScript code snippets, Zod schema validation, and DB design.




3. **Post 3 — Reactive Kanban UX:**

* *Angle:* "Building lag-free drag-and-drop in React using optimistic updates."


* *Content:* Video/GIF demonstrating smooth card movements and optimistic state updates.




4. **Post 4 — Live Demo & Practical Guide:**

* *Angle:* "Connecting an HTML contact form to Beacon CRM in under 3 minutes."


* *Content:* Step-by-step cURL tutorial and live lead capture demonstration.