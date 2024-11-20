# Linux Server Backup Script

This script backs up critical files and directories from a Linux server, excluding optional files. It creates a compressed archive (`.tar.gz`) and retains backups for up to 7 days.

## Features
- Backs up essential directories:
  - `/etc` - System configuration files
  - `/home` - User-specific data
  - `/root` - Root user data
  - `/var/www` - Web server files
  - `/opt` - Third-party applications
  - `/var/lib/mysql` - MySQL database data
  - `/var/lib/postgresql` - PostgreSQL database data
- Automatically cleans up backups older than 7 days.
- Excludes unnecessary directories like `/lost+found`.

---

## Script: `backup.sh`

```bash
#!/bin/bash

# Backup destination directory (change as needed)
BACKUP_DIR="/backup"
DATE=$(date +"%Y-%m-%d_%H-%M-%S")
BACKUP_FILE="$BACKUP_DIR/backup_$DATE.tar.gz"

# Directories to include in the backup
INCLUDE_DIRS=(
    "/etc"         # System configuration
    "/home"        # User data
    "/root"        # Root user data
    "/var/www"     # Web server files
    "/opt"         # Third-party applications
    "/var/lib/mysql" # MySQL data (ensure database is stopped or use a dump)
    "/var/lib/postgresql" # PostgreSQL data
)

# Ensure the backup directory exists
mkdir -p "$BACKUP_DIR"

# Perform the backup
echo "Starting backup..."
tar -czf "$BACKUP_FILE" "${INCLUDE_DIRS[@]}" --exclude='/lost+found'

# Verify the backup
if [ $? -eq 0 ]; then
    echo "Backup completed successfully: $BACKUP_FILE"
else
    echo "Backup failed!"
    exit 1
fi

# Optionally, clean up old backups (keep last 7 days)
find "$BACKUP_DIR" -type f -mtime +7 -name "*.tar.gz" -exec rm {} \;

echo "Old backups cleaned up (if any)."
```

---

## Instructions

### 1. **Create the Script File**
- Save the above script as `backup.sh`.
- Place it in a directory of your choice, such as `/usr/local/bin`.

### 2. **Set Permissions**
Make the script executable:
```bash
chmod +x backup.sh
```

### 3. **Run the Script**
Execute the script manually:
```bash
sudo ./backup.sh
```

### 4. **Automate Backups**
To schedule the script to run regularly, use a cron job:
1. Edit the crontab file:
   ```bash
   sudo crontab -e
   ```
2. Add the following line for daily backups at 2 AM:
   ```bash
   0 2 * * * /path/to/backup.sh
   ```

---

## Notes

### Database Backups
For active databases, use dump commands instead of raw directory backups:
- **MySQL**:
  ```bash
  mysqldump --all-databases > /backup/mysql_backup_$DATE.sql
  ```
- **PostgreSQL**:
  ```bash
  pg_dumpall > /backup/postgres_backup_$DATE.sql
  ```

### Backup Storage
- Ensure the `BACKUP_DIR` is on a separate disk or remote storage.
- Use tools like `rsync` or `scp` to move backups off-site if needed.

---

## Additional Considerations
- Modify `INCLUDE_DIRS` to match your server's specific needs.
- Ensure the script has sufficient permissions to read all directories.
- Test the script on a non-critical system before deploying it in production.
