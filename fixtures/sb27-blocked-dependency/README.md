# Blocked dependency fixture

Planned input: a Maven application requiring an artifact absent from the controlled dependency cache.

Expected: offline verification records a blocked dependency outcome with a useful explanation; it must not report test success or silently enable network access. Sample application and manifest remain Phase 0 work.
