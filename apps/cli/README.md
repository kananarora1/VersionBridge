# CLI application

Local entry point for creating runs, inspecting findings and evidence, selecting patches, requesting verification, and exporting artifacts.

- `src/`: commands, presentation, API/application-service client.
- `tests/`: command behavior, user selection, exit codes, and error reporting.

Use the same application services or API as the HTTP workflow. Keep analysis and patch logic in modules. There is no runnable CLI yet.
