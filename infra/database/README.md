# Database infrastructure

Place local service definitions, health checks, volume configuration, and non-secret example connection settings here after the database/index decision. Schema migrations belong in `modules/storage/migrations/`.

Use disposable fixture databases in tests. Credentials and persistent run artifacts must not be committed. No service definitions have been implemented yet.
