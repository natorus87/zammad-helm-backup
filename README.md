# Zammad Backup CronJob

This Helm-based CronJob automates backups for your Zammad instance, ensuring data safety and recovery readiness. It creates consistent backups of your Zammad files and database, storing them in a predefined location for easy retrieval and restoration.

---

## Features

- Full backups of Zammad's files (attachments, configurations, etc.)
- Database dumps for PostgreSQL (or MySQL if configured)
- Retention policy with automated deletion of old backups
- Support for Kubernetes Persistent Volumes (PVCs)
- Debugging mode for troubleshooting

---

## Configuration

The CronJob supports environment variables and ConfigMaps for customization:

### Environment Variables

| Variable                   | Description                                         | Example                              |
|----------------------------|-----------------------------------------------------|--------------------------------------|
| `BACKUP_DIR`               | Target directory for backup storage                 | `/var/tmp/zammad_backup`             |
| `HOLD_DAYS`                | Number of days to retain backups                    | `10`                                 |
| `FULL_FS_DUMP`             | Backup all files (`yes`) or attachments only (`no`) | `yes`                                |
| `DATABASE_URL`             | Connection string for Zammad's database             | `postgres://user:pass@host:5432/db`  |
| `DEBUG`                    | Enable detailed output for debugging (`yes/no`)     | `yes`                                |

### Volume Mounts

Ensure PersistentVolumeClaims (PVCs) are configured for the backup directory and Zammad data:

```yaml
volumes:
  - name: zammad-backup-storage
    persistentVolumeClaim:
      claimName: zammad-backup-pvc
  - name: zammad-data
    persistentVolumeClaim:
      claimName: zammad-data-pvc
```

---

## Usage

1. Deploy the CronJob in your Kubernetes cluster:

   ```bash
   kubectl apply -f zammad-backup-cronjob.yaml
   ```

2. Monitor CronJob executions:

   ```bash
   kubectl get cronjobs
   kubectl get pods --selector=job-name=<job-name>
   ```

3. Verify backups in the specified directory (`BACKUP_DIR`).

---

## Example CronJob Configuration

Here’s an example CronJob definition:

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: zammad-backup
spec:
  schedule: "0 2 * * *" # Run daily at 2 AM
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: zammad-backup
            image: your-backup-image:latest
            env:
              - name: BACKUP_DIR
                value: "/var/tmp/zammad_backup"
              - name: HOLD_DAYS
                value: "10"
              - name: FULL_FS_DUMP
                value: "yes"
              - name: DEBUG
                value: "yes"
              - name: DATABASE_URL
                value: "postgres://zammad:password@zammad-db:5432/zammad_production"
            volumeMounts:
              - name: zammad-backup-storage
                mountPath: /var/tmp/zammad_backup
          restartPolicy: OnFailure
          volumes:
            - name: zammad-backup-storage
              persistentVolumeClaim:
                claimName: zammad-backup-pvc
```

---

## Requirements

- Kubernetes 1.19+
- Zammad instance (PostgreSQL or MySQL backend)
- PersistentVolume for backup storage
- Proper access to database credentials

---

## Backup Retention

Old backups are automatically deleted after the number of days specified in `HOLD_DAYS`.

---

## Debugging

Set `DEBUG=yes` to enable verbose logging:

```bash
kubectl logs <backup-pod-name>
```

---

## License

This project is licensed under the GNU General Public License v3.0. For more information, see the [LICENSE](LICENSE) file.
