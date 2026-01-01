# Trellis Digital Ocean Space Object Storage Backup

This Ansible role provides ability to save trellis backup to Digital Ocean Space Object Storage.

## Features

- Daily database backups (mysqldump + gzip compression)
- Optional uploads/shared folder sync
- S3-compatible API via AWS CLI
- Preserves existing files (no deletion from bucket)
- Portal notification after successful backup

## Installation

Add to `trellis/server.yml`:

```yaml
- { role: trellis-do-backup, tags: [ trellis-do-backup ] }
```

## Configuration

### Required Variables (in `group_vars/{env}/main.yml`):

```yaml
do_spaces_bucket_name: "your-bucket-name"
do_backup_db_name: "your_database_name"
do_backup_folder_path: "/srv/www/example.com/shared"
do_spaces_region: "fra1"  # or nyc3, sfo3, ams3, sgp1
do_include_web_folder: "false"  # set to "true" to sync uploads
```

### Required Vault Variables (in `group_vars/{env}/vault.yml`):

```yaml
# Use either uppercase OR lowercase (both supported):
DO_ACCESS_KEY: "your-spaces-access-key"      # or: do_access_key
DO_SECRET_KEY: "your-spaces-secret-key"      # or: do_secret_key

# Portal API project IDs (already exist in your vault):
levcharity_portal_api_client_id_staging: 27
levcharity_portal_api_client_id_production: 27
```

## Backup Structure

Uses server hostname as folder name (e.g., `adi-il.org`, `adi.levcharity.dev`):

```
s3://bucket_name/
  adi-il.org/
    databases/
      2024-01-15-04-30.tar.gz
      2024-01-16-04-22.tar.gz
      ...
    web-shared/
      uploads/
        ...
```

## Cron Schedule

Runs daily at 4 AM (randomized minute) as root user.

## Portal Notification

After successful backup, sends POST to `https://portal.levcharity.com/api/v1/backup/{project_id}` with:
- `include_media`: boolean
- `storage_size`: string (GB)
- `created_at`: ISO 8601 timestamp
- `updated_at`: ISO 8601 timestamp

## Author

[Tomek](https://github.com/tomektomczuk)
