# HomeMaker Windows Upgrades

HomeMaker separates installed application files from persistent user data so a newer Windows build can be installed over an existing installation without replacing the database, Recipe media, backups, or configuration.

## Before upgrading

1. Close HomeMaker.
2. Create a current backup from **Settings → Backup & Restore**.
3. Keep the backup in the configured HomeMaker backup directory or copy it to another safe location.

Persistent data normally lives under:

```text
%LOCALAPPDATA%\HomeMaker
```

The installed application normally lives under:

```text
%LOCALAPPDATA%\Programs\HomeMaker
```

## Upgrade in place

Run the newer `HomeMaker-Setup.exe` while the previous version is installed.

The installer uses the same application identity and installation directory. It replaces installed program files but does not delete or overwrite the separate HomeMaker data directory.

On the first launch after upgrade, HomeMaker runs pending Alembic migrations against the existing database before opening the application UI.

The following persistent areas are preserved by the installer:

- `mealplanner.db`
- `media/`
- `backups/`
- `config/`
- `logs/`

## If startup fails after upgrade

Do not remove or recreate the database.

1. Close HomeMaker if a partial window remains open.
2. Preserve `%LOCALAPPDATA%\HomeMaker`.
3. Review:

```text
%LOCALAPPDATA%\HomeMaker\logs\homemaker.log
```

4. Keep the pre-upgrade backup available.
5. Report the failing version/build and log details.

Migration failures are surfaced as startup failures instead of silently replacing user data.

## Uninstall and reinstall

Normal uninstall removes the application files and shortcuts but intentionally leaves `%LOCALAPPDATA%\HomeMaker` in place. A later reinstall therefore reconnects to the preserved durable data and runs any required migrations.

To perform a genuinely clean-data reinstall, uninstall HomeMaker and then manually delete `%LOCALAPPDATA%\HomeMaker` only after confirming that no data or backups are needed.

## Release-candidate verification

Before promoting a Windows build, test upgrade-over-existing-data using a populated installation that contains:

- a non-empty SQLite database;
- Recipe media;
- at least one `.hmbackup` file;
- local configuration;
- representative Meal Cycle, Inventory, Shopping, completion, and History data.

After upgrade, verify that the application starts, migrations finish, all persistent data remains present, and another restart produces the same state.
