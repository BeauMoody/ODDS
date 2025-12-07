# Context Diagram Documentation

## System Overview
The ODDS Class Management Platform centralizes scheduling, enrollment, certification, and reporting for defensive driving schools. The context diagram captures how this platform exchanges information with the primary external actors that interact with it during day-to-day operations.

## Primary Actors
- **Administrators** – Manage global school configuration, oversee reports, and maintain instructor accounts.
- **Instructors** – Schedule classes, enroll students, record payments, and trigger certificate processing.
- **Students/Public Users** – Discover classes, self-enroll via QR code links, and receive completion certificates.
- **State Compliance System** – Receives exported rosters and certificate batches to satisfy regulatory reporting.
- **Payment Gateway** – Processes class fee remittances from instructors to the school.
- **Email/SMS Service** – Delivers enrollment confirmations, reminders, and certificate notifications.

## High-Level Interactions
1. Administrators configure schools, users, and locations through the Admin UI; configuration data is stored in the platform database.
2. Instructors authenticate, manage class rosters, and submit payment confirmations; the system enforces row-level security to keep school data isolated.
3. Students access public enrollment links, submit registration details, and later receive QR code confirmations or certificates.
4. The platform pushes scheduled exports (CSV or API payloads) to the State Compliance System for audit and licensing requirements.
5. When instructors remit the per-student fees, the platform forwards transaction details to the Payment Gateway and records the cleared status.
6. Triggers such as enrollment, payment receipt, or certificate issuance cause notifications to be sent via the Email/SMS service.

## Textual Context Diagram
```
           +-------------------+
           |  State Compliance |
           +---------+---------+
                     ^
                     | Reports / Certificates
                     |
+-------------+      |      +-----------------+      +------------------+
| Administrators |<-->| ODDS Class Mgmt |<--->| Payment Gateway |
+-------------+      |    Platform     |      +------------------+
                     |                ^
+-------------+      |                |
| Instructors |<-----+                | Notifications / Status
+-------------+                       v
                     +-------------------------+
                     | Email / SMS Service     |
                     +-------------------------+
                             ^
                             |
                     +----------------+
                     | Students/Public |
                     +----------------+
```

## Notes & Assumptions
- The State Compliance System integration can be a secure SFTP drop or REST API; this diagram abstracts the transport layer.
- Payment Gateway interactions cover manual instructor remittances (UC-3) today, but can be extended to student self-payments later.
- Email/SMS delivery may be handled by a third-party provider (e.g., SendGrid, Twilio) and is represented as a single notification service boundary.
- Data at rest for all actor interactions is stored in the platform's database via Prisma ORM; the database is not shown separately to keep the diagram system-focused.

## How to Use This Diagram
- **Onboarding**: Share with new engineers to explain which external systems must be available for end-to-end testing.
- **Scope Discussions**: Validate whether upcoming features add new actors or data flows that require security reviews.
- **Compliance Reviews**: Demonstrate that sensitive data exchanges (payments, certificates) are scoped to well-defined boundaries.
