---
title: First Update – Partner-Ready eKYC API
description: Launching secure profile creation, status tracking, and doc uploads in our PocketBase-backed integration service.
authors: likweitan
tags:
  - release-notes
  - api
  - integrations
---

The first milestone for the eKYC Express project is live! This release brings an end-to-end onboarding flow for partners who need to trigger KYC verification, route applicants to our hosted experience, and track progress from their own dashboards.

## Highlights

- **PocketBase integration** authenticates automatically on startup, so you can begin creating profiles without manual prep.
- **Idempotent profile creation** lets you reuse an existing verification record when a document number already exists.
- **Status polling endpoint** now surfaces the latest step from the status history collection, which keeps partner systems in sync.
- **Multi-part update route** accepts document images and facial capture uploads for manual review.

## Endpoint recap

| Endpoint                  | Method  | Purpose                                                             |
| ------------------------- | ------- | ------------------------------------------------------------------- |
| `/api/profiles/create`    | `POST`  | Submit applicant info, receive redirect URL to the hosted KYC flow. |
| `/api/profiles/:id`       | `GET`   | Look up current verification status and metadata.                   |
| `/api/profiles/:id`       | `PATCH` | (Internal) Upload documents, facial photo, and video proofs.        |
| `/api/status_history/:id` | `POST`  | Log manual status transitions with context notes.                   |
| `/`                       | `GET`   | Health check for uptime monitors.                                   |

Check out `getting-started.md` for full payload details and authentication requirements.

## What’s next

We are exploring webhook notifications for status changes and expanding document validation to cover more global ID formats. If you have integration feedback, open an issue in the repo or reach out to the integrations team directly.
