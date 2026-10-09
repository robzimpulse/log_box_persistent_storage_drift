# log_box_persistent_storage_drift Context

## Purpose:
This repository provides a persistent storage backend for the LogBox ecosystem using the Drift (formerly Moor) library. It implements the `PersistentDataStorage` interface from the core package, enabling long-term storage, complex querying, and reactive streaming of log entries using SQLite.

## Key Components:
- **lib/src/drift_persistent_storage.dart**: The primary adapter that connects the LogBox storage interface to the Drift database layer. It handles the serialization/deserialization of `EntryModel` objects to and from the database.
- **lib/src/database/database.dart**: Defines the Drift database schema, including tables for log data and logic for schema migrations.
- **lib/src/dao/data_dao.dart**: The Data Access Object (DAO) containing specific SQL queries for fetching, filtering, and watching log entries.
- **lib/src/table/data_table.dart**: Defines the underlying SQL table structure (e.g., `DataDrift`) used to store JSON-serialized log data and metadata.
- **lib/src/adapter/**: Contains logic for mapping between Drift-specific data types and the domain models used by LogBox.

## Dependencies:
- **drift / drift_flutter**: The primary database framework used for SQLite abstraction and reactive queries.
- **sqlite3 / sqlite3_flutter_libs**: The underlying SQL engine.
- **log_box**: The core internal module that defines the `PersistentDataStorage` contract and `EntryModel` base classes.
- **path_provider**: Used to locate the correct directory on the device for storing the database file.
- **json_annotation**: Used for serializing log entries into the JSON format stored in the database.

## Local Conventions:
- **Schema Decoupling**: Logs are stored as JSON blobs in a generic `DataDrift` table rather than individual columns for every property. This allows the storage layer to remain agnostic of specific log types (Dio, WebView, etc.).
- **Type-Based Decoding**: Uses a `MapObjectDecoder` registry to determine how to reconstruct specific `EntryModel` types from the stored JSON based on a type string.
- **Reactive Queries**: Leverages Drift's `watch` capabilities to provide real-time updates to the UI via the `fetchStream` and `getStream` methods.
- **DAO Pattern**: All SQL logic is encapsulated within DAOs to keep the `DriftPersistentStorage` adapter clean and focused on interface implementation.

## Development:
- **Commands**: Use the `Makefile` (`make` lists targets). `make analyze` and `make format-check` must pass — CI enforces both (infos are fatal).
- **Code Generation**: Run `make generate` after modifying `@JsonSerializable` models; commit the `.g.dart` files.
- **Drift Schema Migrations**: After changing Drift tables, run `make generate-migration` to update `drift_schemas/` and the generated migration code; commit both.
- **Releasing**: Bump `version:` in `pubspec.yaml` and add a matching `## <version>` section to `CHANGELOG.md` in the same PR; merging creates tag `v<version>` via `release.yaml`.

## Known Pitfalls:
- **Cross-Repo Dependency Bumps**: `log_box` is pinned by git tag (`ref: v<version>`), so CI never sees an unreleased core change. For a shared-constraint bump (e.g. rxdart), land and release it in [log_box](https://github.com/robzimpulse/log_box) first, then update `ref:` and the constraint here in its own PR. See core's `AGENTS.md` → "Cross-Repo Dependency Bumps".
- **Pushing Workflow Changes**: Pushing files under `.github/workflows/` requires a GitHub token with the `workflow` scope. If a push is rejected with "refusing to allow an OAuth App to create or update workflow", run `gh auth refresh -h github.com -s workflow` and make git use that token (`gh auth setup-git`) — a stale macOS Keychain token will otherwise keep failing.
