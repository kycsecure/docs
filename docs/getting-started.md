---
id: getting-started
title: Getting Started
sidebar_label: Getting Started
---

## Introduction

The eKYC Integration API lets partners trigger identity verification flows and keep track of their progress.
It accepts profile details, creates or reuses a PocketBase record, and returns a redirect URL that points to
your hosted onboarding experience. A separate status endpoint exposes the latest verification outcome so you
can keep internal systems in sync.

## Base URLs

| Environment | URL (example)                 |
| ----------- | ----------------------------- |
| Testing     | `http://localhost:3000`       |
| Production  | `https://your-domain.example` |

Use the correct base URL when constructing requests. Every example below assumes the local development host.

## Authentication

All partner-facing endpoints are protected with static API keys. Supply your key in the `x-api-key` header
for each authenticated request. If you cannot set custom headers, the API also checks the `apiKey` property
in the JSON payload of the create endpoint.

```
x-api-key: your-secret-api-key-1
```

If the key is missing or invalid, the API responds with `401 Invalid or missing API key`.

## Quickstart Flow

1. **Provision an API key** and share it securely with the calling system.
2. **Create a profile** by sending the applicant's basic details to `/api/profiles/create`.
3. **Redirect the applicant** to the `url` returned in the create response.
4. **Poll for status** by calling `GET /api/profiles/:id` to read the latest verification status.

### Sample Client Request

```bash
curl -X POST "http://localhost:3000/api/profiles/create" \
  -H "Content-Type: application/json" \
  -H "x-api-key: your-secret-api-key-1" \
  -d '{
    "name": "Jane Doe",
    "document_type": "passport",
    "document_number": "A1234567",
    "nationality": "SG"
  }'
```

When the profile already exists (matched on document type, number, and country), the API will return the
existing PocketBase record identifier so you can resume the flow without creating duplicates.

## API Reference

### Create / Reuse Profile

- **Method:** `POST`
- **Path:** `/api/profiles/create`
- **Authentication:** `x-api-key` header (required)
- **Body:** JSON

| Field             | Type     | Required | Description                                           |
| ----------------- | -------- | -------- | ----------------------------------------------------- |
| `name`            | `string` | yes      | Full name of the applicant                            |
| `document_type`   | `string` | yes      | Supported document type (e.g., `passport`, `id_card`) |
| `document_number` | `string` | yes      | Document identifier                                   |
| `nationality`     | `string` | yes      | ISO country code or plain country name                |

**Successful response (201 Created):**

```json
{
  "success": true,
  "url": "https://kyc-2-rho.vercel.app/RECORD_ID",
  "id": "RECORD_ID"
}
```

**Idempotent response (200 OK):** Returned when a matching record already exists.

```json
{
  "success": true,
  "url": "https://kyc-2-rho.vercel.app/RECORD_ID",
  "id": "RECORD_ID",
  "message": "Existing record found"
}
```

**Error responses:**

| Status | Body                                                        | When it happens                              |
| ------ | ----------------------------------------------------------- | -------------------------------------------- |
| `400`  | `{ "error": "Missing required fields", "required": [...] }` | One or more required fields omitted          |
| `401`  | `{ "error": "Invalid or missing API key" }`                 | No API key or key not in `VALID_API_KEYS`    |
| `500`  | `{ "error": "Internal server error" }`                      | PocketBase authentication or storage failure |

### Retrieve Profile Status

- **Method:** `GET`
- **Path:** `/api/profiles/:id`
- **Authentication:** None (public status lookup). The route returns JSON immediately; the duplicate
  internal route with API key middleware is not invoked because the public handler responds first.

**Response (200 OK):**

```json
{
  "success": true,
  "data": {
    "name": "Jane Doe",
    "document_type": "passport",
    "document_number": "A1234567",
    "nationality": "SG",
    "verification_status": "unverified",
    "status": "pending_review",
    "createdAt": "2025-10-23T12:34:56.789Z"
  }
}
```

- `verification_status` reflects the stored status on the profile record.
- `status` reports the most recent entry in the `status_history` collection if one exists; otherwise it
  falls back to the profile status.

**Error responses:**

| Status | Body                                                                      | When it happens           |
| ------ | ------------------------------------------------------------------------- | ------------------------- |
| `404`  | `{ "error": "KYC record not found", "message": "Invalid or expired ID" }` | Unknown PocketBase record |
| `500`  | `{ "error": "Internal server error" }`                                    | PocketBase query failure  |

### Health Check

- **Method:** `GET`
- **Path:** `/`
- **Response:**

```json
{
  "status": "OK",
  "service": "eKYC Integration API",
  "recordsStored": 0
}
```

This endpoint is useful for uptime monitors and confirms basic server health.

## Next Steps

- Build automated retries around the create endpoint to gracefully handle transient 500 errors.
- Subscribe to status updates (for example with webhooks) if you need near real-time changes instead of polling.
- When migrating this document into Docusaurus, update the sidebar labels and order as needed.
