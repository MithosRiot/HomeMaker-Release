# Installing HomeMaker on Windows

HomeMaker's normal Windows release is distributed as `HomeMaker-Setup.exe`.

## Requirements

Supported release target:

- 64-bit Windows 10 or Windows 11
- a normal desktop user account
- Microsoft Edge WebView2 runtime, which is included with current Windows 10/11 installations and serviced by Windows/Microsoft Edge updates

You do **not** need to install Python, Node.js, npm, SQLite, Docker, Git, or a separate database server.

## Install

1. Download `HomeMaker-Setup.exe` from the HomeMaker GitHub release for the version you want to install.
2. Run the installer.
3. Keep the default install location unless you have a reason to change it.
4. Optionally select the desktop-shortcut task.
5. Finish installation and launch HomeMaker.

The installer is currently unsigned. Windows SmartScreen can therefore show an unknown-publisher warning on locally built or unsigned release candidates. Verify that the installer came from the HomeMaker release/build you intended to use before continuing.

## What the installer installs

The installer places the self-contained application under the current user's local application-program directory, normally:

```text
%LOCALAPPDATA%\Programs\HomeMaker
```

That installation contains:

- the compiled React frontend;
- the FastAPI backend;
- the bundled Python runtime and application dependencies;
- Alembic migration files;
- SQLite support used by the Python runtime;
- the HomeMaker desktop launcher.

Persistent user data is **not** stored in the installation directory.

## Launch behavior

Start **HomeMaker** from the Start menu, desktop shortcut if selected, or the installed executable.

The launcher:

1. initializes the durable HomeMaker data directory;
2. starts the backend on an automatically selected local port;
3. binds that backend only to `127.0.0.1`;
4. runs database migrations before the application becomes usable;
5. opens the HomeMaker UI in the desktop webview window.

HomeMaker does not create an inbound Windows Firewall exception for its local API.

Launching HomeMaker a second time while it is already running does not start another backend instance. Close the existing HomeMaker window before launching it again.

Closing the HomeMaker desktop window requests a clean backend shutdown.

## Data and logs

Persistent data is stored under:

```text
%LOCALAPPDATA%\HomeMaker
```

Important locations:

```text
%LOCALAPPDATA%\HomeMaker\mealplanner.db
%LOCALAPPDATA%\HomeMaker\media
%LOCALAPPDATA%\HomeMaker\backups
%LOCALAPPDATA%\HomeMaker\config
%LOCALAPPDATA%\HomeMaker\logs\homemaker.log
```

The application directory and persistent-data directory are intentionally separate so upgrades and normal uninstall do not replace or remove your database, media, backups, or configuration.

## First startup and migration

On startup, HomeMaker applies all pending Alembic database migrations before serving the UI.

If the HomeMaker durable data directory does not yet exist, the existing one-time legacy migration behavior may copy data from the previous product data directory or former development `./data` location. The migration source is left intact for recovery.

If startup or database migration fails, HomeMaker shows a startup-failed message and records details in:

```text
%LOCALAPPDATA%\HomeMaker\logs\homemaker.log
```

Do not delete the database to work around a migration failure. Preserve the data directory and use the log when troubleshooting.

## Notifications

HomeMaker's prep-reminder UI remains available in the packaged application. Browser-style operating-system notifications depend on notification support exposed by the embedded Windows webview. If that notification API is unavailable or permission is denied, HomeMaker continues to use the documented in-app reminder list as the fallback.

## Uninstall

Uninstall HomeMaker from **Settings → Apps → Installed apps** or the Start-menu uninstall entry supplied by Windows/installer registration.

The normal uninstaller removes the installed application files and shortcuts. It does **not** remove:

```text
%LOCALAPPDATA%\HomeMaker
```

Delete that directory manually only when you intentionally want to remove the database, Recipe media, backups, configuration, and logs.

See [Upgrades](../operations/UPGRADES.md) before installing a newer build over an existing installation.
