# HomeMaker

HomeMaker is a self-contained Windows application for meal planning, recipes, inventory, shopping, prep, cooking, leftovers, history, and local backups.

## Download

Download the latest Windows installer from the **Releases** section of the HomeMaker-Release repository.

**Installer:** `HomeMaker-Setup.exe`

This distribution repository contains official HomeMaker releases and normal-user documentation. The application source code is maintained separately and is not published here.

## Requirements

- Windows 10 or Windows 11
- No separate Python, Node.js, SQLite, Docker, or database installation required

## Install

1. Open the desired release.
2. Download `HomeMaker-Setup.exe`.
3. Run the installer.
4. Launch HomeMaker from the Start menu or desktop shortcut.

Windows may display a SmartScreen warning because the installer is currently unsigned. Verify that the installer was downloaded from the official HomeMaker-Release repository before continuing.

## User documentation

User-facing installation, upgrade, backup/restore, troubleshooting, and workflow documentation is maintained alongside each promoted release in this repository.

## Data

HomeMaker stores durable user data under:

`%LOCALAPPDATA%\HomeMaker`

Normal upgrades and uninstall preserve application data unless it is explicitly removed.

## Updates

New versions are published as GitHub Releases. Download the newer installer and install it over the existing version to upgrade while preserving your data.

## Support

If you encounter a problem with an official release, open an issue in the HomeMaker-Release repository with:

- HomeMaker version
- Windows version
- what you were doing
- what happened
- any error message shown

Do not include private data, backup files, or personal meal/inventory information in public issues.
