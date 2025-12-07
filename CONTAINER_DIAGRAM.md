# Container Diagram Documentation

## Purpose
The container view highlights the major runtime building blocks of the ODDS Class Management Platform and how they collaborate to deliver core features such as scheduling, enrollment, certification, and reporting.

## Containers
1. **Browser Client (Quasar/Vue 3 SPA)**
   - Provides the administrator, instructor, and public user experiences.
   - Implements routing, state management (Pinia), and API service wrappers for backend communication.
   - Hosted as a static bundle served by any CDN or edge platform.

2. **API Server (Node.js + Express)**
   - Central orchestration layer exposing REST endpoints described in `api/openapi.yaml`.
   - Handles authentication (JWT), authorization (role + school scoping), validation, reporting, and business workflows (e.g., certificate sequencing, payment tracking).
   - Integrates with Prisma to read/write domain data, and brokers calls to external services.

3. **Database (SQLite via Prisma ORM)**
   - Stores schools, users, classes, students, payments, and certificate state.
   - Prisma schema mirrors the legacy MySQL design, enabling future migration to PostgreSQL/MySQL with minimal code change.
   - Implements soft deletes, referential integrity, and indexes for high-volume searches (UC-2).

4. **Notification Service (Email/SMS provider)**
   - Abstracts outbound communications such as enrollment confirmations, reminders, and certificate notifications.
   - Currently a stub that can log messages; architecture assumes SendGrid/Twilio-style providers.

5. **Payment Gateway (Instructor Remittance)**
   - Processes $10 per student payments when instructors settle class fees (UC-3).
   - Today represented as an integration stub; future versions can plug in Authorize.net or another PSP.

6. **State Compliance Interface**
   - Exports CSV or API payloads for state reporting, supporting certificate batches and class rosters.
   - Modeled as a file/API boundary to keep regulatory interactions isolated.

## High-Level Responsibilities
- The Browser Client renders the UI and calls the API Server over HTTPS using JWT bearer tokens stored in secure storage.
- The API Server enforces access control, orchestrates workflows, and persists data changes via Prisma.
- The Database is the single source of truth for operational data and audit trails.
- External services are invoked asynchronously or synchronously depending on the workflow (e.g., immediate email after enrollment, scheduled compliance exports).

## Textual Container Diagram
```
+---------------------------+        HTTPS         +---------------------------+
|   Browser Client (SPA)    | <-------------------> |   API Server (Node/Express)|
| - Quasar/Vue 3            |                      | - Auth, business logic     |
| - Pinia, Axios            |                      | - REST endpoints           |
+---------------------------+                      +-------------+-------------+
                                                                  |
                                                                  | Prisma ORM
                                                                  v
                                                     +---------------------------+
                                                     |   Database (SQLite)       |
                                                     | - Domain entities         |
                                                     | - Audit + soft deletes    |
                                                     +---------------------------+
                                                                  |
        +---------------------------+        +--------------------+------------------+
        | Notification Service      |<-------| External Integrations (via API Server)|
        | - Email/SMS provider      |        | - Payment Gateway                     |
        +---------------------------+        | - State Compliance Interface           |
                                             +---------------------------------------+
```

## Deployment Considerations
- **Browser Client**: Built artefacts can be deployed to Netlify, Vercel, S3, or embedded within the Quasar SSR server.
- **API Server**: Runs on Node.js 18+ (Express). Expose port 5001, secure with HTTPS, configure environment variables (`JWT_SECRET`, database URL, external API keys).
- **Database**: SQLite for development; production should switch to PostgreSQL/MySQL with the same Prisma schema.
- **Integrations**: Use environment-specific credentials and secrets for payment and notification providers. State exports may require VPN or SFTP endpoints.

## Usage
- Share during architecture reviews to explain deployment slices.
- Use as a reference when onboarding new engineers or aligning DevOps on infrastructure needs.
- Extend with additional containers (e.g., worker queues, monitoring) as the platform evolves.
