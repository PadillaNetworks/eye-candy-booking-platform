# Engineering Case Study: Eye Candy Beauty Lounge

## Problem

A multi-professional appointment business needed a branded booking website with a dependable staff dashboard and a way to collect deposits through the appropriate professional's Square connection. The system also needed staff-visible payment status, controlled customer communications, and recoverable application data.

## My contribution

Configured, deployed, integrated and validated the real production booking application through iterative releases. Work covered application behavior, integration testing, Render configuration, webhook diagnostics, Resend delivery, backup verification and automated source packaging. The project was developed with AI-assisted implementation and human-run tests and deployment decisions.

## System components

| Layer | Technology | Engineering purpose |
|---|---|---|
| Web application | Python, Flask, Gunicorn | Public customer booking and protected staff workflows |
| Hosting | Render | Deployment, environment configuration and worker processes |
| Database | SQLite | Appointment, status and integration state |
| Merchant payments | Square API | Professional account connections, deposit checkout and payment reconciliation |
| Notifications | Resend | Customer email delivery and test/live separation |
| Backups | Cloudflare R2, S3-compatible API, AES-256-GCM | Encrypted offsite snapshots and recovery verification |
| Source delivery | GitHub, GitHub Actions | Version control, tagged release points, and repeatable handoff artifacts |

## Notable technical challenges

### 1. Synchronizing booking state across two systems

A booking could be rescheduled or cancelled directly in Square rather than inside the custom app. The integration had to reconcile these updates and prevent newly occupied appointment times from remaining available on the customer side. The user verified sandbox scenarios for external reschedules, cancellations and schedule conflicts.

### 2. Trusting a payment only after verification

A generated payment link is **not** a completed payment. The production flow was exercised using a real $1 checkout: Square reported success, then the staff dashboard displayed a verified paid deposit for the linked booking. This verified the end-to-end reconciliation rather than merely accepting a webhook response.

### 3. Diagnosing external API failures

A Resend diagnostic POST initially returned HTTP 403 with Cloudflare 1010. The sending client was updated to identify itself with an application-specific User-Agent. Subsequent requests reached Resend authentication and authenticated delivery completed with HTTP 200; test email receipt was confirmed. The incident reinforced separating network-edge rejection from API authorization and app-level state.

### 4. Preventing unsafe bulk mail during launch

The payment and appointment email workers used separate feature flags and explicit production opt-ins. Existing pending/uncertain queues were checked before enabling customer-facing email delivery. Appointment email cutover used a UTC activation timestamp to avoid treating older test events as new production notifications. A final audit reported zero queued, uncertain or review-required messages; email gates reported READY after launch configuration.

### 5. Restoring data rather than merely backing it up

Daily SQLite snapshots were pushed into private Cloudflare R2 as authenticated encrypted bundles. Recovery testing downloaded a remote object, decrypted into a temporary location, checked database integrity and verified the Square connection key format without overwriting production data. A new encrypted launch backup was later generated and verified separately.

### 6. Making future maintenance reproducible

A GitHub Actions workflow produces a sanitized, timestamped source code ZIP from committed files after code pushes. The user verified downloading the artifact. The public portfolio intentionally documents engineering outcomes without exposing the production source bundle.

## Verification matrix

| Workflow | Test method | Verified outcome |
|---|---|---|
| Production payment events | Square dashboard synthetic event tests | Both payment event endpoints returned HTTP 200 |
| Payment-link creation | Staff action for a fictional booking | Square production link became Link Ready |
| Revocation | Revoke unpaid link | Staff-confirmed revoked state |
| Payment settlement | Authorized $1 transaction | Square Succeeded; booking showed Paid |
| Deposit email | Controlled inbox | Received and opened Square checkout link |
| General appointment email | Controlled inbox | Test email received; live config gate later Ready |
| Availability conflicts | Sandbox calendar tests | Occupied slots excluded from customer booking choices |
| Offsite recovery | Isolated decrypt + integrity validation | User reported test passed |
| Source automation | GitHub Actions run | Artifact downloaded successfully |

## Operational notes

Public availability, all staff schedules, charge refunds and full worker expiration behavior are separate checks from the evidence above. Other professionals configure their own services and schedule. The site should only expose a professional as bookable after their offerings and availability are set.

## Portfolio relevance

Demonstrates practical foundations applicable to network engineering and security engineering: HTTP/API troubleshooting, application-to-service integration, environment-based access control, log-driven incident response, production change management, secrets protection and recoverable infrastructure.
