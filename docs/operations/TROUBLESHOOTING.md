# Troubleshooting

This page documents failures that follow from the current development, database, migration, durable-data, backup/restore, frontend-proxy, and seeded-data behavior.

Use the format **Symptom → Cause → Resolution** when adding new entries.

## Frontend cannot reach the backend

**Symptom**

The UI loads, but API requests fail, return network errors, or Vite reports proxy failures.

**Cause**

The frontend proxies `/api` and `/health` to `127.0.0.1:8000`, but the backend is not listening there.

**Resolution**

From `backend/`:

```bash
python run_test.py
```

Then verify:

```text
http://127.0.0.1:8000/health
```

Expected response:

```json
{"status":"ok"}
```

## Port 8000 is already in use

**Symptom**

The backend fails to start because it cannot bind to `127.0.0.1:8000`.

**Cause**

Another process is already using the port.

**Resolution**

Stop the process using port `8000`, then restart:

```bash
python run_test.py
```

The normal frontend development proxy expects the backend on port `8000`, so changing only the backend port also requires changing the Vite proxy configuration.

## Port 5173 is already in use

**Symptom**

Vite selects another port or fails to start on the expected development URL.

**Cause**

Another process is using port `5173`.

**Resolution**

Stop the conflicting process and rerun:

```bash
npm run dev
```

Using the configured port avoids confusion with documented URLs and proxy behavior.

## `pytest` is not recognized or found

**Symptom**

Running backend tests reports that `pytest` is unavailable.

**Cause**

The intended virtual environment is not active or backend development dependencies were not installed.

**Resolution**

From `backend/`, activate the virtual environment and run:

```bash
python -m pip install -e ".[dev]"
pytest -q
```

## Import/module errors after backend setup

**Symptom**

Backend commands fail with missing application modules or dependencies.

**Cause**

The editable backend package was not installed into the active Python environment, or a different Python interpreter is being used.

**Resolution**

Check the interpreter:

```bash
python --version
python -m pip --version
```

Then reinstall from `backend/`:

```bash
python -m pip install -e ".[dev]"
```

The backend requires Python 3.12 or later.

## Seeded data does not reset when the backend restarts

**Symptom**

Changes made during local testing remain after stopping and restarting `run_test.py`.

**Cause**

`run_test.py` intentionally refreshes deterministic data without resetting the existing seeded/UAT state.

**Resolution**

Explicitly reset:

```bash
python testdata/seed_test_db.py --reset
```

Then restart:

```bash
python run_test.py
```

## Database startup fails after a schema change

**Symptom**

FastAPI fails during startup after model/migration changes.

**Cause**

Application startup runs Alembic migrations before normal request handling. A broken migration, incompatible populated database, or recovery problem can therefore prevent startup.

**Resolution**

From `backend/`:

```bash
pytest -q tests/test_migration_recovery.py
python testdata/validate_populated_upgrade.py
```

Then run the full backend suite:

```bash
pytest -q --cov=app --cov-report=term-missing
```

Do not delete an important database as the first recovery action.

## Existing `./data` did not migrate to the durable location

**Symptom**

The application starts with an empty/new durable data directory even though an older `./data` directory exists.

**Cause**

Automatic legacy migration only runs when the durable target does not already exist, `HOMEMAKER_DATA_DIR` is not explicitly set, and the previous working-directory-relative `./data` source exists. The migration intentionally never overwrites an already initialized durable directory.

**Resolution**

Do not delete either location. Verify the configured data directory and process working directory first. The legacy source is intentionally retained for recovery. If both locations already contain user data, reconcile them deliberately rather than forcing one to overwrite the other.

See [Configuration](CONFIGURATION.md) for exact path rules.

## Backup creation returns a conflict

**Symptom**

`POST /api/system/backups` returns HTTP `409`.

**Cause**

Application backup requires an existing file-backed SQLite database. In-memory SQLite, a non-SQLite database URL, or a database that cannot be snapshotted consistently is rejected.

**Resolution**

Verify `HOMEMAKER_DATABASE_URL` points to the expected local SQLite file and that the file is readable. Do not copy an active WAL database manually; use the application backup service so SQLite creates a consistent snapshot.

## Restore is rejected or fails

**Symptom**

A restore request returns `400`, `404`, or `422`, or restore-time migration fails.

**Cause**

Common causes are a missing backup (`404`), missing exact `{"confirmation":"RESTORE"}` request value (`422`), corrupt archive/hash/SQLite validation (`400`), or failure to upgrade the restored database to the current Alembic head.

**Resolution**

Do not modify the live database or delete other backups. Try another known-good archive when corruption is reported. Restore-time replacement/migration failures are rollback-protected; the previous database, media, and configuration are restored automatically when rollback succeeds.

Run:

```bash
pytest -q tests/test_storage.py tests/test_backup_restore.py tests/test_system_backup_api.py
pytest -q tests/test_migration_recovery.py
python testdata/validate_populated_upgrade.py
```

See [Backup and Restore](BACKUP_RESTORE.md) for the complete workflow.

## Foreign-key errors during database/test-data changes

**Symptom**

A database mutation or fixture reset fails with a foreign-key constraint error.

**Cause**

SQLite foreign-key enforcement is enabled. New child/history tables may require cleanup in dependency order.

**Resolution**

If the failure occurs in deterministic test data, update/use the repository's fixture/reset logic rather than manually deleting arbitrary parent rows.

Run:

```bash
python testdata/seed_test_db.py --reset
```

For schema changes, inspect the model relationships and migration ordering.

## Frontend TypeScript errors

**Symptom**

Frontend CI fails before the production build or local code reports TypeScript errors.

**Cause**

The frontend type-check step found an invalid contract, import, or TypeScript expression.

**Resolution**

From `frontend/`:

```bash
npm run typecheck
```

Fix all type errors before rerunning:

```bash
npm run build
```

## Frontend selector/API-helper test fails

**Symptom**

CI fails on one of the Node-based `.mjs` test files.

**Cause**

A selector, state transformation, typed JSON helper, or API helper no longer matches its expected behavior.

**Resolution**

Run the failing test directly, for example:

```bash
node --experimental-strip-types src/dashboardSelectors.test.mjs
```

Then run the complete frontend check set documented in [Testing](../developer/TESTING.md).

## Meal Cycle activation returns a conflict

**Symptom**

Activating a Meal Cycle returns HTTP `409`.

**Cause**

Activation requires a valid `DRAFT` cycle with:

- a start date,
- serving times for slot definitions,
- no other active cycle for the household, and
- no blocking validation errors.

**Resolution**

Correct the cycle schedule and validation issues, or complete/cancel the currently active cycle as appropriate, then retry activation.

Do not bypass activation checks by editing database fields directly.

## Meal Cycle completion returns a conflict

**Symptom**

Completing an active cycle returns HTTP `409`.

**Cause**

One or more PlannedMeals have not reached finalized completion.

**Resolution**

Finish/finalize all required meal occurrences in the cycle, then retry completion.

Cycle completion intentionally releases active inventory and production-coverage reservations only after the completion requirement is satisfied.

## Cannot edit a completed/finalized occurrence

**Symptom**

A workflow rejects edits to a finalized occurrence.

**Cause**

Finalized completion/history records are intentionally immutable.

**Resolution**

Treat the finalized record as historical truth. Do not modify it directly in SQLite. If correction support is needed, it should be implemented as an explicit product workflow with audit/history semantics.

## Shopping demand looks different after an active-cycle edit

**Symptom**

Shopping requirements change after adding, replacing, moving, removing, or resizing an occurrence in an active cycle.

**Cause**

Active-cycle reconciliation recalculates derived operational state, including reservations, produced-stock coverage, gather selections, shopping demand, validation, and prep-related views.

**Resolution**

Verify the plan change first. If the changed demand is unexpected, inspect active-cycle reconciliation tests and the shopping baseline/plan-delta behavior before modifying generated shopping rows manually.

Completed purchase history should remain preserved through regeneration.

## Manual shopping purchase does not create inventory

**Symptom**

Completing a manual shopping item does not create an InventoryLot.

**Cause**

Manual shopping items remain separate from generated ingredient demand. Inventory creation requires the manual purchase to be explicitly linked to an Ingredient and to contain the required intake data.

**Resolution**

Verify the manual item has been resolved to an Ingredient and that purchase/storage data required by the workflow is present.

## Where to investigate next

For implementation failures, use this order:

1. reproduce with the smallest relevant test,
2. inspect the API route,
3. inspect service/engine logic,
4. inspect models/migrations/storage for persistence behavior,
5. inspect frontend API/helper state only after confirming the backend contract.

Related documentation:

- [Development Setup](../getting-started/DEVELOPMENT_SETUP.md)
- [Testing](../developer/TESTING.md)
- [API Reference](../developer/API.md)
- [Data Model](../database/DATA_MODEL.md)
- [Backup and Restore](BACKUP_RESTORE.md)
