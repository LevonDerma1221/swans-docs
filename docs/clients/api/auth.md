# Authentication and keys

Two independent credential systems, mirroring the separation between trading and control.

## FIX sessions

- One `SenderCompID` per member connection; `Username` (553) and `Password` (554) on Logon, rotated at least every 90 days via the member portal.
- TLS 1.3 with client certificates issued by SWANS; certificates bound to the session and to the IP allow-list.
- Sessions are enabled per account and per instrument group. Disabling a session cancels its resting orders if `CancelOnDisconnect` is set for the session.

## API keys (REST and WebSocket)

| Key type | Used for | Issued by | Scopes |
|---|---|---|---|
| Read key | Reference data, market data, own-account read endpoints | Member admin in portal | `read:refdata`, `read:marketdata`, `read:account:{ids}` |
| Control key | Hedge uploads, margin simulation, key management | Member admin, with a second approver | `write:account:{ids}`, `admin:keys` |
| Clearer key | Clearer API | Clearing-member admin, mTLS required | `clearer:{clearer_id}` |

- Keys are OAuth2 client credentials; tokens expire after one hour; refresh with the client secret.
- Every key has a name, an expiry (max 365 days), an IP allow-list and scopes fixed at creation. Expired keys count against the per-member limit of 20 until revoked.
- Revoking a key invalidates its tokens immediately. Revoking a control key does not affect FIX sessions or resting orders; those are governed by FIX credentials.
- Member admins are two named individuals; key creation and revocation are logged and visible in the portal audit trail.

## Replay and staleness

Signed control-plane requests (Clearer API limits, kill, hedge uploads) carry `client_ts` and `recv_window` (max 60 s); requests outside the window are rejected. Idempotency keys are required on `PUT`/`POST` control calls; a repeated key returns the original result.

## Operator actions

Halt, resume, bust, settlement steps and parameter changes are performed by named SWANS staff with individual credentials and hardware second factor; every action is journaled with actor, time and reason, and appears in the regulatory audit extract.
