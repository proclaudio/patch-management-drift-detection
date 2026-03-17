# Patch Management + Drift Detection

Automated patch management and configuration drift detection across a
Rocky Linux lab environment. Implements enterprise-grade patching workflow
with pre-flight checks, controlled reboots, and post-patch verification.
Drift detection runs on a daily AWX schedule to ensure all servers remain
compliant with the configured baseline.

---

## Architecture
```mermaid
flowchart TD
    A[AWX Scheduled Job] --> B{Workflow Type}
    B --> C[Patch Management]
    B --> D[Drift Detection]
    C --> E[Check service health]
    E --> F[dnf update]
    F --> G{Reboot required?}
    G -->|Yes| H[Controlled reboot]
    G -->|No| I[Verify services]
    H --> I
    I --> J[Patch complete]
    D --> K[Scan all 5 servers]
    K --> L{Changes detected?}
    L -->|Yes| M[AWX reports drift]
    L -->|No| N[Servers compliant]
```

---

## Technologies

| Technology       | Purpose                              |
| ---------------- | ------------------------------------ |
| Ansible          | Patch automation and drift detection |
| AWX              | Scheduling and orchestration         |
| dnf              | Package management                   |
| needs-restarting | Reboot requirement detection         |
| Rocky Linux 9    | Target servers                       |

---

## Patch Targets

Production servers are excluded from automated patching.
Only development and staging servers are patched automatically.

| Host         | Environment | Patched          |
| ------------ | ----------- | ---------------- |
| dev-web-01   | Development | Yes              |
| stage-web-01 | Staging     | Yes              |
| prod-web-01  | Production  | No - manual only |

---

## Patching Workflow
```
Pre-flight check (verify all services running)
       |
       v
dnf check-update
       |
       v
dnf update (if updates available)
       |
       v
needs-restarting -r (check if reboot required)
       |
       v
Controlled reboot (if required, 300s timeout)
       |
       v
Post-flight check (verify all services running)
       |
       v
Structured patch report (host, times, updates, reboot, status)
```

Key design decisions:
- serial: 1 patches one server at a time (rolling update)
- Maintenance window guard aborts patching outside 01:00-05:00
- Pre-flight check aborts if any service is down before patching
- Post-flight check alerts if any service fails after patching
- Conditional reboot only when needs-restarting confirms it
- dnf-utils installed automatically as pre-task

---

## Drift Detection

Scans all 5 servers daily for configuration drift from the baseline:

| Check | Expected State |
| ----- | -------------- |
| node_exporter binary | Present at /usr/local/bin/node_exporter |
| node_exporter service | active (running) |
| SSH PermitRootLogin | no |
| SSH PasswordAuthentication | no |
| auditd service | active (running) |

Any deviation produces a DRIFT DETECTED message in AWX output,
identifying exactly which server and which control has drifted.

---

## AWX Schedule

Drift detection runs automatically every 24 hours:

- Job: Drift Detection
- Schedule: Daily at 02:00 AM
- Targets: All 5 lab servers

---

## Screenshots

### AWX Patch Job Template
![Patch Template](screenshots/awx-patch-template.png)

### Patch Job Running
![Patch Job](screenshots/awx-patch-job.png)

### Patch Complete with Summary
![Patch Complete](screenshots/awx-patch-complete.png)

### Daily Drift Check Schedule
![Drift Schedule](screenshots/awx-drift-schedule.png)

### Drift Detection Output
![Drift Output](screenshots/awx-drift-output.png)

---

## DevOps Skills Demonstrated

- Automated patch management with safety checks
- Rolling update strategy (serial: 1)
- Maintenance window enforcement
- Pre-flight and post-flight service verification
- Conditional reboot logic (only when required)
- Structured patch reporting with human-readable output
- Configuration drift detection across 5 servers
- AWX job scheduling for automated compliance
- Environment-aware automation (dev/stage vs prod)
- SRE reliability engineering practices

---

## Part of DevOps Portfolio

- [Project 1 - Enterprise Infrastructure Automation Lab](https://github.com/proclaudio/enterprise-infrastructure-automation-lab)
- [Project 2 - CI/CD Push-to-Deploy Pipeline](https://github.com/proclaudio/cicd-push-to-deploy-pipeline)
- [Project 3 - Infrastructure Monitoring Stack](https://github.com/proclaudio/infrastructure-monitoring-stack)
- [Project 4 - Automated Security Hardening](https://github.com/proclaudio/automated-security-hardening)
- [Project 5 - Centralized Log Management](https://github.com/proclaudio/centralized-log-management)
- **Project 6 - Patch Management + Drift Detection** (this repo)
- Project 7 - AWX RBAC + Team Management (coming soon)
- Project 8 - Kubernetes Platform Lab (coming soon)
