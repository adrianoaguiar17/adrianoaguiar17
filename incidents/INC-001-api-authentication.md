# `INC-001` — API Authentication Error

```text
┌─────────────────────────────────────────────────────────────┐
│ SUPPORT INCIDENT                                            │
├─────────────────────────────────────────────────────────────┤
│ ID          INC-001                                         │
│ STATUS      RESOLVED                                        │
│ SEVERITY    MEDIUM                                          │
│ CATEGORY    API / Authentication                            │
│ TYPE        LAB CASE                                        │
└─────────────────────────────────────────────────────────────┘
```

## `> issue`

A user can access the application normally, but an integration fails when requesting a protected API endpoint.

```http
GET /api/v1/customers

HTTP/1.1 401 Unauthorized
```

The integration had worked previously, and no application errors were visible to the user.

## `> investigation`

```text
[01] Reproduce the request                         [ OK ]
[02] Confirm endpoint and HTTP method              [ OK ]
[03] Inspect authentication header                 [ OK ]
[04] Validate token                                [ OK ]
[05] Compare application environments              [ FOUND ]
```

### `01 — Reproduce`

The request was reproduced outside the application to isolate the API from the client interface.

```bash
curl -X GET \
  https://api.example.com/v1/customers \
  -H "Authorization: Bearer ********"
```

Result:

```http
HTTP/1.1 401 Unauthorized
```

The issue was therefore not limited to the application UI.

### `02 — Endpoint`

The endpoint and HTTP method matched the API documentation.

```text
Method      GET
Endpoint    /api/v1/customers

Result      VALID
```

### `03 — Authentication`

The request contained the expected authentication header.

```text
Authorization: Bearer ********

Header      PRESENT
Format      VALID
```

This suggested that the problem was related to the credential itself rather than a missing authentication header.

### `04 — Token validation`

The token was valid, but its environment did not match the requested API environment.

```text
API environment      PRODUCTION
Token environment    DEVELOPMENT

                     ↑ MISMATCH
```

## `> root_cause`

```text
ROOT CAUSE FOUND

The production integration was using
a development API token.

The token itself was valid, but it was
not authorized for the production environment.
```

## `> resolution`

The development credential was replaced with the correct production credential and the request was tested again.

```http
GET /api/v1/customers

HTTP/1.1 200 OK
```

```text
Authentication      [ OK ]
API Request         [ OK ]
Response            [ 200 ]
Incident            [ RESOLVED ]
```

## `> prevention`

```text
✓ Keep development and production credentials separated
✓ Use environment variables for API credentials
✓ Avoid hardcoded tokens
✓ Verify environment before debugging application logic
✓ Never expose credentials in logs
```

## `> takeaway`

A `401 Unauthorized` does not necessarily mean authentication is missing.

The authentication mechanism may be correct while the credential is invalid for the environment being accessed.

Troubleshooting the request layer by layer helps isolate the problem before changing application logic.

---

### `> navigation`

[← Back to Support Console](../README.md) · **INC-001 / 004** · Next Incident →

---

<sub>LAB CASE — Created to demonstrate a structured technical support troubleshooting process. No real credentials, customer information or production systems are represented.</sub>
