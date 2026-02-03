# Networking troubleshooting quick notes

## Quick “can I reach the internet” checks
- `ip route` (default gateway present)
- `curl -I https://example.com` (HTTPS works)
- `dig example.com` (DNS resolves)

## Routing
**Command**
- `ip route`

**What good looks like**
- A `default via <gateway>` route exists.

**Common failure symptoms**
- No default route. Outbound traffic fails.
- Wrong gateway/interface.

## Listening ports
**Command**
- `ss -tulpn`

**What good looks like**
- The expected service is `LISTEN` on the expected port.

**Common failure symptoms**
- Service is running but not listening on the right port.
- Nothing listening.

## DNS
**Command**
- `dig example.com`

**What good looks like**
- `status: NOERROR` and an `ANSWER SECTION`.

**Common failure symptoms**
- `SERVFAIL`, `NXDOMAIN`, or timeouts.

## Traceroute
**Command**
- `traceroute example.com`

**What it tells you**
- Shows path/hops. Helps spot where traffic stops.

**Notes**
- `* * *` hops are common because routers may not reply.
