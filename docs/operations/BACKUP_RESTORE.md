# Backup and Restore

HomeMaker provides local application-level backup and restore for the file-backed SQLite configuration used by the desktop/local application.

## User workflow

Backup and restore are available from **Settings → Data & Backup**.

The Data & Backup screen lets a user:

- create a complete local backup;
- see existing backups with their date/time and size; and
- restore an earlier backup.

Restoring a backup replaces the current HomeMaker database, media, and configuration with the selected snapshot. HomeMaker opens an in-app confirmation dialog that identifies the selected backup and requires the user to type the exact confirmation value `RESTORE` before the restore request is sent. After a successful restore, HomeMaker reloads so the restored state is shown.

## What a backup contains

A `.hmbackup` archive contains:

- a consistent SQLite snapshot created with SQLite's backup API;
- files under the durable `media/` directory, including uploaded Recipe images;
- files under the durable `config/` directory; and
- a manifest containing the backup format version and SHA-256 hashes for every captured file.

The `backups/` directory itself is not included, so backups do not recursively contain earlier backups.

Recipe image metadata is stored in SQLite while the image bytes live under `media/recipes/<recipe-id>/`. Because backup/restore captures the database and media tree as one application snapshot, restoring a backup restores both the Recipe-to-image association and the corresponding file contents.

## Default backup location

Backups created through the application are stored under the durable application data directory:

```text
<application-data>/backups/
```

On Windows the normal application-data root is `%LOCALAPPDATA%\HomeMaker` unless `HOMEMAKER_DATA_DIR` overrides it.

## API workflow

The Data & Backup UI uses the system backup API. With the backend running, the same operations are available directly for development/troubleshooting:

List backups:

```text
GET /api/system/backups
```

Create a backup:

```text
POST /api/system/backups
```

The response includes the archive name, size, and modification timestamp.

Restore an existing backup:

```text
POST /api/system/backups/{backup_name}/restore
```

Request body:

```json
{
  "confirmation": "RESTORE"
}
```

The exact confirmation value is required. Other values fail request validation.

## Restore safety behavior

Before any live data is replaced, the application:

1. validates the archive structure, member metadata, and format version;
2. rejects archives that exceed the configured restore resource limits;
3. streams each staged file in bounded chunks while validating its SHA-256 hash;
4. validates the staged SQLite database with `PRAGMA integrity_check`;
5. creates a rollback snapshot of the current live SQLite database and copies current media/configuration;
6. replaces the live data from staging;
7. runs the normal Alembic upgrade path against the restored database; and
8. validates SQLite integrity again.

If replacement, migration, or final validation fails, the previous database, media, and configuration are restored from the rollback snapshot.

A corrupt, incomplete, oversized, or otherwise invalid archive is rejected before live application data is modified. Failed member extraction removes partial staged output. Recipe media follows the same rollback boundary: a failed restore must not leave a restored Recipe image record paired with the previous live file tree, or vice versa.

### Restore resource limits

Restore limits are defined in `backend/app/services/backup_restore.py` and validated from ZIP metadata before member extraction begins:

- maximum archive members: **10,000**;
- maximum uncompressed size of one archive member: **512 MiB**;
- maximum aggregate uncompressed archive size: **4 GiB**;
- maximum uncompressed manifest size: **8 MiB**; and
- extraction/hash-validation chunk size: **1 MiB**.

These limits are intended to prevent a small compressed archive from expanding without bound in memory or on local disk. Normal file members are never read into memory as one complete buffer during restore; SHA-256 validation is computed incrementally while the member is streamed to staging.

## Supported database configuration

Application backup/restore requires a file-backed SQLite database.

It is rejected when the configured database is:

- SQLite `:memory:`; or
- a non-SQLite SQLAlchemy database URL.

When `HOMEMAKER_DATABASE_URL` points at another file-backed SQLite database, backup/restore uses that actual configured database rather than assuming `mealplanner.db` under the data directory.

## Manual round-trip test

Use disposable or deterministic test data, not irreplaceable user data.

1. Start HomeMaker normally.
2. Open **Settings → Data & Backup**.
3. Select **Create backup** and confirm a new `homemaker-*.hmbackup` entry appears.
4. Make a visible data change in the test application.
5. Select **Restore** on the backup created in step 3.
6. In the in-app restore confirmation, verify the selected backup and replacement warning, then type `RESTORE`.
7. Confirm HomeMaker reloads and the pre-change data returns.

When Recipe media is involved, also upload an image before creating the backup, replace or remove it after backup creation, restore the backup, and verify the original image and Recipe association return together.

The generated API documentation at `http://127.0.0.1:8000/docs` remains available for developer/troubleshooting use, but it is not the normal user workflow.

## Legacy data migration

The durable-data transition preserves earlier application data during the HomeMaker rename. On first normal startup, when the HomeMaker durable data directory does not already exist, the previous product data directory is copied into the HomeMaker directory through a staging directory and atomic final rename.

The old source is intentionally left untouched for recovery. Migration is not repeated after the HomeMaker durable directory exists. Legacy backup archives are normalized to the HomeMaker backup extension during this one-time migration so they remain visible in Settings.

The durable-data transition also preserves the older default `./data` development layout under the same first-run rules.

See [Configuration](CONFIGURATION.md) for exact path rules and development overrides.

## Recovery guidance

If a restore is rejected as corrupt, oversized, or otherwise invalid, do not modify the live database or delete the existing backup set. Try another known-good `.hmbackup` archive.

If application startup fails after restoring an older backup, the restore operation should already have rolled back when its Alembic upgrade failed. Run the repository migration recovery tests before attempting manual database repair.
