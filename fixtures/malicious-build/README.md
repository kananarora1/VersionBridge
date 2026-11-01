# Untrusted build fixture

Planned input: a controlled Maven build that attempts a harmless sentinel write outside its workspace or a network request against a test-only endpoint.

Expected: read-only ingestion does not execute anything. Verification either rejects a prohibited profile under policy or contains the attempt inside isolation; no host sentinel write, secret access, or egress is allowed. Do not add destructive payloads or execute this fixture directly on the host. Sample files and manifest remain Phase 0 work.
