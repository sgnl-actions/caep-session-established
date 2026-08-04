# CAEP Session Established Event Transmitter

This action transmits CAEP (Continuous Access Evaluation Protocol) Session Established events as Security Event Tokens (SET) to specified receivers. It implements the [OpenID CAEP 1.0 Final specification](https://openid.net/specs/openid-caep-1_0-final.html) for session lifecycle notifications.

## Overview

The CAEP Session Established event communicates that a new session has been created for a subject. This is used for:
- Notifying downstream systems when users authenticate
- Propagating session metadata (authentication methods, context class)
- Correlating federated sessions across identity providers
- Session-aware access policy enforcement

## Prerequisites

- Node.js 22 runtime environment
- Configured authentication (Bearer, Basic, or OAuth2)
- Target receiver endpoint that accepts Security Event Tokens

## Configuration

### Input Parameters

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `subject` | text | Yes | Subject identifier JSON (e.g., `{"format":"email","email":"user@example.com"}`) |
| `audience` | text | Yes | Intended recipient of the SET |
| `fp_ua` | text | No | User agent fingerprint for the session |
| `acr` | text | No | Authentication Context Class Reference (e.g., `urn:mace:incommon:iap:silver`) |
| `amr` | text | No | Authentication Methods References as JSON array (e.g., `["pwd","otp","mfa"]`) |
| `ext_id` | text | No | External session identifier (e.g., federated SAML session ID) |
| `event_timestamp` | text | No | Unix timestamp (seconds) when the session was established (defaults to now) |
| `initiating_entity` | text | No | What initiated the session: `admin`, `user`, `policy`, `system` |
| `reason_admin` | text | No | Admin-readable reason (JSON i18n object or plain string) |
| `reason_user` | text | No | User-readable reason (JSON i18n object or plain string) |
| `address` | text | No | Destination URL override (defaults to ADDRESS env var) |

### Outputs

| Name | Type | Description |
|------|------|-------------|
| `status` | text | Operation result: `success` or `failed` |
| `statusCode` | number | HTTP status code from the SET receiver |
| `body` | text | Response body from the SET receiver |
| `retryable` | boolean | Whether the error is retryable |

## Usage Examples

### Basic Session Established

```json
{
  "subject": "{\"format\":\"email\",\"email\":\"user@example.com\"}",
  "audience": "https://receiver.example.com"
}
```

### Session with Authentication Context

```json
{
  "subject": "{\"format\":\"email\",\"email\":\"user@example.com\"}",
  "audience": "https://receiver.example.com",
  "fp_ua": "Mozilla/5.0 (Windows NT 10.0; Win64; x64)",
  "acr": "urn:mace:incommon:iap:silver",
  "amr": "[\"pwd\",\"otp\",\"mfa\"]",
  "ext_id": "saml-session-abc123",
  "initiating_entity": "user"
}
```

## Event Structure

```json
{
  "iss": "https://sgnl.ai/",
  "aud": "https://receiver.example.com",
  "iat": 1234567890,
  "jti": "unique-event-id",
  "sub_id": {
    "format": "email",
    "email": "user@example.com"
  },
  "events": {
    "https://schemas.openid.net/secevent/caep/event-type/session-established": {
      "event_timestamp": 1234567890,
      "fp_ua": "Mozilla/5.0 ...",
      "acr": "urn:mace:incommon:iap:silver",
      "amr": ["pwd", "otp", "mfa"],
      "ext_id": "saml-session-abc123",
      "initiating_entity": "user"
    }
  }
}
```

## Error Handling

Retryable errors (automatic retry): 429, 502, 503, 504
Non-retryable errors (permanent failure): 400, 401, 403, 404

## Development

```bash
npm install
npm test
npm run build
npm run lint
npm run validate
```

## References

- [OpenID CAEP 1.0 Final Specification](https://openid.net/specs/openid-caep-1_0-final.html)
- [RFC 8417 - Security Event Token (SET)](https://datatracker.ietf.org/doc/html/rfc8417)
