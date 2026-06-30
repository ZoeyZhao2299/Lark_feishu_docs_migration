---
name: lark-to-feishu-migration
description: Migrate selected Lark cloud documents, sheets, and Base/Bitable resources into Feishu, then place them in Feishu My Document Library.
---

# Lark to Feishu Cloud Docs Migration Skill

Use this skill when migrating Lark cloud documents, sheets, and Base/Bitable resources into Feishu.

The recommended final destination is Feishu `My Document Library`, which is represented by the Wiki personal library alias `my_library`.

## Safety Rules

- Do not migrate every search result by default.
- Filter migration scope explicitly, for example by owner, folder, wiki space, or a reviewed allowlist.
- Do not migrate shared, external, cross-tenant, or other-owner resources unless the user explicitly approves that scope.
- Never commit or print app secrets, access tokens, device codes, OAuth tokens, or raw credential files.
- Use `--as user` for personal cloud documents. Bot identity cannot reliably access a user's personal Drive, Wiki, sheets, or Base resources.
- Treat `my_library` as Wiki personal library, not Drive root.
- Before deleting temporary Drive shortcuts or folders, verify they only contain migration-created shortcuts or are empty.

## Required Setup

Create one source Lark app and one target Feishu app.

Enable the relevant scopes in both developer consoles:

- Drive metadata read
- File download
- File upload
- Docs read/export/import
- Docx create/read/write
- Sheets read/create/write
- Base/Bitable read/create/write
- Wiki space read
- Wiki node read/move/create
- Search docs read

Configure two `lark-cli` profiles:

```bash
lark-cli --profile lark doctor
lark-cli --profile feishu doctor
```

Recommended profile names:

- `lark`: source Lark tenant
- `feishu`: destination Feishu tenant

If a proxy is needed, use environment variables appropriate for the local machine:

```bash
HTTP_PROXY=http://<proxy-host>:<proxy-port>
HTTPS_PROXY=http://<proxy-host>:<proxy-port>
ALL_PROXY=http://<proxy-host>:<proxy-port>
NO_PROXY=127.0.0.1,localhost
```

## User Authorization

Start OAuth device flow:

```bash
lark-cli --profile lark auth login --domain all --no-wait --json
lark-cli --profile feishu auth login --domain all --no-wait --json
```

After the user authorizes, finish with:

```bash
lark-cli --profile lark auth login --device-code '<device_code>'
lark-cli --profile feishu auth login --device-code '<device_code>'
```

Verify:

```bash
lark-cli --profile lark auth status
lark-cli --profile feishu auth status
```

Expected state:

```text
User identity: ready
tokenStatus: valid
```

## Inventory Source Resources

Search source documents:

```bash
lark-cli --profile lark drive +search \
  --as user \
  --query '' \
  --doc-types doc,docx,sheet,bitable,wiki,slides \
  --page-size 20 \
  --format json
```

Review the results and filter the migration scope.

Common filters:

- `owner_name == <expected owner name>`
- only resources in a reviewed CSV allowlist
- only resources under a specific folder or wiki space

## Type Mapping

| Source Type | Export Format | Feishu Import Type |
|---|---|---|
| `docx` | `.docx` | `docx` |
| `sheet` | `.xlsx` | `sheet` |
| `bitable` | `.base` | `bitable` |

For Wiki URLs, unwrap first:

```bash
lark-cli --profile lark drive +inspect \
  --as user \
  --url '<wiki_url>' \
  --format json
```

Use the resolved `type` and `token` for export.

## Export From Lark

Docx:

```bash
lark-cli --profile lark drive +export \
  --as user \
  --token '<docx_token>' \
  --doc-type docx \
  --file-extension docx \
  --file-name 'title.docx' \
  --output-dir work/exports \
  --overwrite \
  --format json
```

Bitable/Base:

```bash
lark-cli --profile lark drive +export \
  --as user \
  --token '<bitable_token>' \
  --doc-type bitable \
  --file-extension base \
  --file-name 'title.base' \
  --output-dir work/exports \
  --overwrite \
  --format json
```

Sheet:

```bash
lark-cli --profile lark drive +export \
  --as user \
  --token '<sheet_token>' \
  --doc-type sheet \
  --file-extension xlsx \
  --file-name 'title.xlsx' \
  --output-dir work/exports \
  --overwrite \
  --format json
```

If the export artifact is ready but download fails, retry with:

```bash
lark-cli --profile lark drive +export-download \
  --as user \
  --file-token '<export_file_token>' \
  --file-name 'title.docx' \
  --output-dir work/exports \
  --overwrite \
  --format json
```

## Import Into Feishu

You can first import into a temporary Feishu Drive folder:

```bash
lark-cli --profile feishu drive +create-folder \
  --as user \
  --name 'Lark Migration YYYY-MM-DD' \
  --format json
```

Import docx:

```bash
lark-cli --profile feishu drive +import \
  --as user \
  --file 'work/exports/title.docx' \
  --type docx \
  --name 'title' \
  --folder-token '<folder_token>' \
  --format json
```

Import bitable:

```bash
lark-cli --profile feishu drive +import \
  --as user \
  --file 'work/exports/title.base' \
  --type bitable \
  --name 'title' \
  --folder-token '<folder_token>' \
  --format json
```

Import sheet:

```bash
lark-cli --profile feishu drive +import \
  --as user \
  --file 'work/exports/title.xlsx' \
  --type sheet \
  --name 'title' \
  --folder-token '<folder_token>' \
  --format json
```

## Move Into Feishu My Document Library

Resolve `my_library`:

```bash
lark-cli --profile feishu wiki spaces get \
  --as user \
  --params '{"space_id":"my_library"}' \
  --format json
```

Move imported Drive documents into it:

```bash
lark-cli --profile feishu wiki +move \
  --as user \
  --obj-type docx \
  --obj-token '<feishu_doc_token>' \
  --target-space-id '<my_library_space_id>' \
  --format json
```

Use `--obj-type bitable` for Base files and `--obj-type sheet` for sheets.

## Cleanup

After moving documents into `my_library`, Feishu may leave shortcuts in the temporary Drive folder.

List the folder:

```bash
lark-cli --profile feishu drive files list \
  --as user \
  --params '{"folder_token":"<folder_token>","page_size":200}' \
  --format json
```

If only `shortcut` items remain, delete them:

```bash
lark-cli --profile feishu drive +delete \
  --as user \
  --type shortcut \
  --file-token '<shortcut_token>' \
  --yes \
  --format json
```

Then delete the empty temporary folder:

```bash
lark-cli --profile feishu drive +delete \
  --as user \
  --type folder \
  --file-token '<folder_token>' \
  --yes \
  --format json
```

## Final Verification

Verify personal library nodes:

```bash
lark-cli --profile feishu wiki +node-list \
  --as user \
  --space-id my_library \
  --format json
```

Verify the temporary Drive folder is empty or deleted.

## Lessons Learned

- Use user identity for personal cloud documents.
- Filter by owner or reviewed scope before bulk migration.
- Treat `my_library` as Wiki personal library, not Drive root.
- Export bitable as `.base`, then import as `bitable`.
- Clean up temporary Drive shortcuts after moving files into personal library.
