# Verifier worker application

Separate process entry point that receives a verification job and delegates to the verifier module and isolated runner.

- `src/`: job dispatch, cancellation, lifecycle, worker configuration.
- `tests/`: worker retry, timeout, cancellation, and crash recovery.

The worker must run project code only inside the configured sandbox. Use the [verifier contract](../../modules/verifier/README.md); persist results through storage. No host build command is supplied by this scaffold.
