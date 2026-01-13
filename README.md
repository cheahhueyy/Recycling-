# Recycling-
# GreenCycle Management System (GCMS)

Company: Escube Technology Sdn. Bhd  
Project: GreenCycle Management System (GCMS)  
Author: cheahhueyy  
Date: 2026-01-13

## Overview
GreenCycle Management System (GCMS) is a demo application for a recycling facility to manage waste intake, processing, inventory of processed outputs, sales, invoicing, suppliers, and operational reporting. GCMS focuses on traceability, real-time processing status, secure role-based access, and business reporting to support day-to-day operations.

This README captures the functional and non-functional requirements, suggested architecture, data model, API examples, UI flows, acceptance criteria, testing checklist, and deployment notes for a demo implementation.

---

## Key Features (at-a-glance)
- Record and track incoming waste with automatic unique Waste Intake IDs
- Track processing stages (sorting, shredding, compacting) with real-time status updates
- Store and view processed outputs (e.g., plastic pellets, metal scraps)
- Manage sales transactions and auto-generate invoices (PDF/Excel export)
- Maintain supplier profiles and view transaction history
- Dashboards for daily/weekly/monthly operational reports
- Role-based secure login and permissions
- Low-stock alerts for processed material thresholds
- Daily automated database backups

---

## Functional Requirements (mapped & acceptance criteria)

1. Record incoming waste
   - UI: form with category, weight, supplier, optional notes, received-by, timestamp
   - Acceptance: New intake persisted and visible in intake list within 2s.

2. Auto-generate Waste Intake ID
   - Format suggestion: GCMS-WI-{YYYYMMDD}-{6-digit seq} (e.g., GCMS-WI-20260113-000123)
   - Acceptance: ID generated server-side, unique, immutable.

3. Track waste processing stages
   - Stages: Received → Sorting → Shredding → Compacting → Processed
   - Acceptance: Stage history with timestamps stored and auditable.

4. Real-time processing status updates
   - Staff updates status via UI or mobile device; updates visible to all authorized users in <3s.
   - Acceptance: Status change event updates DB/log; optionally via websockets or polling.

5. Store/view processed output details
   - Track processed product type, quantity, yield rate (input → output), batch references.
   - Acceptance: Output records link to originating intake(s).

6. Manage sales transactions
   - Create sale records referencing processed outputs, prices, customer, payment status.
   - Acceptance: Sale persists and reduces stock.

7. Auto-generate invoices
   - Generate invoice PDF/Excel after sale completion; invoice number sequence maintained.
   - Acceptance: Downloadable invoice available within 5s after sale saved.

8. Manage supplier profiles
   - CRUD suppliers, contact details, transaction history view.
   - Acceptance: Supplier transaction list includes intakes and payments.

9. Dashboards (daily/weekly/monthly)
   - KPIs: total intake weight, processed output quantities, sales revenue, low-stock items.
   - Acceptance: Dashboard loads in <3s for typical dataset used by demo.

10. Secure login & role-based access
    - Roles: Admin, Manager, Operator, Viewer (customizable)
    - Acceptance: Access control enforced across API/UI.

11. Export reports (PDF/Excel)
    - One-click export for displayed reports and filtered datasets.
    - Acceptance: Files correctly formatted and downloadable.

12. Low-stock alerts
    - Configurable threshold per processed product; alert via UI and optional email.
    - Acceptance: Alert fires when stock < threshold and is visible in alerts list.

---

## Non-Functional Requirements (NFR) & Implementation Notes

1. Performance
   - Requirement: 200 concurrent users, response time < 3s
   - Implementation: Horizontally scalable app servers, connection pooling, DB indexing, caching (Redis)

2. Security
   - Requirement: HTTPS/TLS 1.2+
   - Implementation: Enforce TLS, use OAuth2/JWT for auth, secure secrets storage, input validation, role-based access

3. Usability
   - Requirement: Usable with < 2 hours training
   - Implementation: Consistent layout, contextual help, wizard for intake and sale flows

4. Scalability
   - Requirement: Support 2× increase in records
   - Implementation: Scale DB vertically or add read replicas, use pagination and background jobs for heavy tasks

5. Reliability
   - Requirement: 99% uptime/month
   - Implementation: Use managed infrastructure, health checks, auto-restart, simple failover

6. Backup
   - Requirement: Daily DB automatic backup
   - Implementation: Managed DB daily snapshots, retention policy, automated restore runbook

---

## Suggested Technology Stack (demo-friendly)
- Frontend: React (TypeScript) with component library (Material UI / Ant Design)
- Backend: Node.js (Express / NestJS) or Python (FastAPI) — REST + optional WebSocket
- Database: PostgreSQL (relational for traceability)
- Cache/Realtime: Redis (caching, pub/sub for status updates)
- Auth: Keycloak / Auth0 / Firebase Auth / JWT for demo
- Reporting & Exports: server-side PDF via wkhtmltopdf / Puppeteer, Excel via SheetJS or server libs
- Storage: S3-compatible for file storage (invoices, backups)
- CI/CD: GitHub Actions
- Deployment: Docker + Kubernetes (or Docker Compose for local demo)
- Monitoring: Prometheus + Grafana, Sentry for errors
- Email: SendGrid / SMTP for alerts

---

## Data Model (core tables / entities)
- users (id, name, email, password_hash, role_id, created_at)
- roles (id, name, permissions)
- suppliers (id, name, contact_person, phone, email, address, notes)
- waste_intakes (id, waste_intake_id, category, weight_kg, supplier_id, received_by, received_at, status)
- process_stages (id, waste_intake_id, stage_name, status, updated_by, updated_at, notes)
- processed_outputs (id, sku, name, quantity_kg, unit, batch_no, source_intake_ids, created_at)
- inventory (processed_output_id, quantity_kg, location)
- sales_transactions (id, transaction_no, buyer_name, items[], total_amount, created_by, status, created_at)
- invoices (id, invoice_no, sale_id, pdf_url, issued_at, due_date, paid_at)
- alerts (id, type, message, related_id, severity, created_at, acknowledged_by, acknowledged_at)
- backups (id, file_url, created_at, status)

Notes:
- Use audit columns (created_by, created_at, updated_by, updated_at) for traceability.

---

## Waste Intake ID & Invoice Numbering
- Waste Intake ID format: GCMS-WI-{YYYYMMDD}-{000001}
  - Implementation: Use DB sequence increment per day or global sequence, padded to 6 digits.
- Invoice number format: GCMS-INV-{YYYY}-{00001}

---

## Example REST API Endpoints (subset)
- POST /api/auth/login — authenticate, returns JWT
- GET /api/users/me — current user profile
- POST /api/waste-intakes — create intake (server generates Waste Intake ID)
- GET /api/waste-intakes — list/filter intakes
- GET /api/waste-intakes/{id} — intake details + stage history
- POST /api/waste-intakes/{id}/stages — add/update processing stage
- GET /api/processed-outputs — inventory list
- POST /api/sales — create sale (reduce inventory)
- POST /api/sales/{id}/invoice — generate invoice (PDF), return download URL
- GET /api/reports/summary?range=daily|weekly|monthly — KPIs
- POST /api/suppliers — create supplier
- GET /api/alerts — list alerts
- POST /api/alerts/{id}/acknowledge — acknowledge alert

Authentication: Bearer token (JWT). All endpoints validate permissions.

---

## UI Flow (high level)
1. Login → Dashboard
2. Intake → Fill form (category, weight, supplier) → Submit → system assigns Waste Intake ID → visible in intake list
3. Processing → Operator selects intake → updates stage/status → stage history recorded (optional photo)
4. Processed Output → Create product batch from processed intakes → update inventory
5. Sales → Create sale referencing inventory → generate invoice → download/send invoice
6. Alerts → System triggers when inventory < threshold → notify managers
7. Reporting → Filter date range → export PDF/Excel

---

## Roles & Permissions (example)
- Admin: full access (users, system config, backups)
- Manager: dashboards, sales, invoices, suppliers, reports
- Operator: create intakes, update process stages, view inventory
- Viewer: read-only access to dashboards and reports

Map endpoints to role checks in backend.

---

## Alerts & Thresholds
- Thresholds configurable per processed product.
- Alert types: low_stock, processing_delay, failed_backup.
- Delivery: in-app notifications + optional email.

---

## Testing & QA Checklist
- Unit tests for core business logic (ID generation, inventory updates, invoice generation)
- Integration tests for API endpoints (happy path & edge cases)
- End-to-end tests for main flows: intake → processing → output → sale → invoice
- Load test: simulate 200 concurrent users (locust / k6) and measure response times
- Security scan: dependency scanning, SAST tools, TLS verification
- Backup restore test: perform a periodic restore on staging

---

## Deployment & DevOps Notes
- Use environment variables for secrets (do not store in repo)
- DB migrations with a managed tool (Flyway / Alembic / TypeORM migrations)
- Daily backup schedule with retention policy (e.g., 30 days)
- Health checks and readiness/liveness endpoints
- Logging: structured logs (JSON) and central aggregation (ELK / Grafana Loki)
- Use CI to run tests and build artifacts; CD to push to staging and production

---

## Demo Roadmap (suggested milestones)
- Week 1: Core domain model, DB schema, authentication, intake creation with ID
- Week 2: Process stage updates, operator UI, inventory representation
- Week 3: Sales, invoice generation (PDF), supplier management
- Week 4: Dashboards, export (PDF/Excel), alerts, backups & deployment scripts
- Demo: Deploy to demo environment, run sample dataset, produce guided walkthrough

---

## Sample Acceptance Tests (example)
- Create intake → verify DB record and format of Waste Intake ID
- Update stage to Shredding → check stage history entry and status on intake page
- Create processed output from intakes → verify inventory increases and source tracking
- Create sale that consumes inventory → verify inventory decreases and invoice is generated
- Lower inventory below threshold → verify alert created and email notification sent (if configured)
- Run export → verify Excel and PDF open and contain expected data

---

## Security & Privacy Considerations
- Encrypt data in transit (TLS 1.2+). Consider encrypting sensitive fields at rest.
- Enforce strong passwords and optionally 2FA for Admin roles.
- Role-based access control for multi-tenant concerns (if applicable).
- Logging must avoid storing raw credentials or PII.

---

## Backup & Restore Runbook (demo)
1. Backup: automated daily snapshot of DB to S3 with timestamped names.
2. Retention: keep last 30 days (configurable).
3. Restore test monthly: restore backup to staging DB and run smoke tests.
4. Emergency restore: documented steps with contact person (Admin).

---

## Next Steps / How to Start
1. Scaffold repo with chosen stack (frontend, backend, migrations)
2. Implement authentication and user roles
3. Create DB schema and implement intake creation + ID generation
4. Build operator UI and process stage updates (websocket or polling)
5. Implement inventory and sales/invoice flows
6. Add reporting, exports, alerting, and backups
7. Run performance tests and tune indexes/caching

---

## Contacts
Escube Technology Sdn. Bhd — Project: GreenCycle Management System (GCMS)  
Owner / Requester: cheahhueyy

---

## Appendix
- Sample Waste Intake ID: GCMS-WI-20260113-000042  
- Sample Invoice No: GCMS-INV-2026-00012

If you'd like, I can:
- generate a starter project scaffold (backend + database migrations + sample data),
- provide an initial ER diagram and SQL schema,
- produce example API implementation (routes and controllers) in Node.js/Express or Python/FastAPI,
- or create a clickable demo UI prototype (Figma wireframes).

Which would you like to do next?
