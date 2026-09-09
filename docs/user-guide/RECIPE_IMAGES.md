# Recipe Images

Recipes can have one optional uploaded image. Images are presentation media only; removing or losing an image does not remove or invalidate the Recipe itself.

## Supported files

HomeMaker accepts:

- JPEG (`image/jpeg`)
- PNG (`image/png`)
- WebP (`image/webp`)

The maximum upload size is 5 MiB.

The application validates the declared image type and the file signature. Images are stored as uploaded; they are not resized or re-encoded.

## Add or replace an image

1. Create and save the Recipe first.
2. Open the Recipe editor.
3. Choose **Upload / replace image**.
4. Select a supported image up to 5 MiB.

A successful replacement changes the Recipe's active image. If validation or storage fails, the previously valid image remains in place.

## Remove an image

Open the Recipe editor and choose **Remove image**.

Removing an image removes the Recipe-to-image association and managed media file. The Recipe and its structured data remain unchanged.

## Missing media

If a Recipe image file is missing from disk, Recipe library/detail/editor pages show the normal Recipe image fallback instead of failing the page. Missing media does not cause the application to recreate or invent an image automatically.

## Storage and backup

Image bytes are stored in the durable application data area under:

```text
<application-data>/media/recipes/<recipe-id>/
```

SQLite stores only managed image metadata and the Recipe association; image bytes are not stored in the database.

Application backups include both the SQLite metadata and the durable media tree. Restoring a backup therefore restores the Recipe image association and image file together.

See [Backup and Restore](../operations/BACKUP_RESTORE.md) for backup behavior and [Configuration](../operations/CONFIGURATION.md) for the application-data location.
