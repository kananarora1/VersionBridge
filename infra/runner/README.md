# Runner infrastructure

Place the verifier image, sandbox configuration, resource quotas, network controls, dependency-cache preparation, and cleanup policy here after choosing the isolation runtime.

Require a non-root process, no secrets or host socket, read-only original snapshot, bounded disposable workspace, and explicit timeout handling. Prove isolation with controlled fixtures before executing user builds. A container configuration alone is not evidence that a checkpoint passed.
