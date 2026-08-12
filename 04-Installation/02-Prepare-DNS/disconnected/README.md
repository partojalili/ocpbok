# Prepare DNS — Disconnected / Air-Gapped

## Same as Connected

DNS requirements are identical in disconnected environments. The same A records, wildcard records, and PTR records are required.

## Additional Record

Add a DNS record for the mirror registry if it doesn't already have one:

| Record | Type | Value | Purpose |
|--------|------|-------|---------|
| `registry.example.com` | A | Mirror registry IP | Local image registry |
