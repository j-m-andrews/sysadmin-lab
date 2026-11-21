[200~# Day 3 - systemd Service Management

## Overview
systemd is the init system and service manager for RHEL. Every service on the system is managed through systemd. Starting, stopping, enabling, disabling, and monitoring services are core daily tasks for a systems administrator. This module covers service management, reading logs, attack surface reduction, a custom unit file, and troubleshooting a failed service.

## Environment
- Host: CachyOS Linux
- Hypervisor: VMware Workstation Pro 25H2
- VM: Red Hat Enterprise Linux 10.1
- Kernel: 6.12.0-124.52.1.el10_1.x86_64

## Service Management Fundamentals

### Viewing Running Services
```
systemctl list-units --type=service --state=running
```
Lists all currently active services on the system.

### Checking Service Status
```
systemctl status sshd
```
Returns the current state of a service, including load status, active state, main PID, memory and CPU usage, CGroup processes, and recent journal entries.

Key fields to read:
- **Loaded** - whether the unit file was found and whether the service is enabled or disabled at boot
- **Active** - current runtime state (running, inactive, failed)
- **Main PID** - process ID of the service, useful for deeper troubleshooting
- **CGroup** - all processes running under this service, tracked by systemd

### Starting and Stopping
```
sudo systemctl stop sshd
sudo systemctl start sshd
```
Controls whether a service is running right now. Does not affect boot behavior.

### Enabling and Disabling
```
sudo systemctl enable sshd
sudo systemctl disable sshd
systemctl is-enabled sshd
sudo systemctl enable --now httpd
```
Controls whether a service starts automatically at boot by creating or removing symlinks in systemd target directories (for example /etc/systemd/system/multi-user.target.wants/). Enabling does not start a service immediately, and disabling does not stop it. `enable --now` does both in one command.

Confirmed in the lab: a service can be disabled and still running (it will not survive a reboot), and a freshly installed service is typically inactive and disabled until explicitly started and enabled.

### Restart vs Reload
```
sudo systemctl restart sshd
sudo systemctl reload httpd
```
- **restart** - stops and starts the service fresh and drops active connections
- **reload** - signals the running process to re-read its configuration without stopping, preserving active connections. Prefer reload in production when applying config changes.

## Reading Logs with journalctl
```
journalctl -u sshd
journalctl -u sshd --since "1 hour ago"
```
Pulls journal entries for a specific service. The `--since` flag filters by time window. This is the primary tool for diagnosing service failures: look for error messages, PID changes, and timestamps to correlate with incidents. After connecting to the VM over SSH, the journal showed the accepted login, source IP, and session open.

## SSH Access
- Stopped, started, enabled, and disabled sshd
- Configured SSH access from the CachyOS host to the RHEL VM
- Verified the connection and confirmed the login entry in the systemd journal

## Attack Surface Reduction - Disabling cups
cups (Common Unix Printing System) was stopped and disabled. No server role on this system requires a print spooler, and cups listens on port 631 by default, which is unnecessary network exposure. Disabling services that a system does not need is a baseline hardening practice in DoD environments.

```
sudo systemctl stop cups
sudo systemctl disable cups
```
Output confirmed removal of four symlinks from the sockets, multi-user, and printer targets.

## Custom systemd Unit File
Created a custom service unit from scratch at /etc/systemd/system/labmonitor.service to demonstrate unit file structure.

```
[Unit]
Description=Lab Monitor Service
After=network.target

[Service]
Type=simple
ExecStart=/bin/bash -c "while true; do echo Lab monitor running >> /var/log/labmonitor.log; sleep 60; done"
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

- **[Unit]** - metadata and dependencies. `After=network.target` ensures the service starts only after networking is up.
- **[Service]** - defines how the service runs. `Restart=on-failure` tells systemd to restart the process automatically if it crashes.
- **[Install]** - defines boot behavior. `WantedBy=multi-user.target` adds the service to the standard multi-user boot target.

```
sudo systemctl daemon-reload
```
Must be run after creating or modifying any unit file so systemd re-reads the unit files on disk. The service was then started and verified running with two tracked CGroup processes (the bash loop and its sleep child). Log output was confirmed at /var/log/labmonitor.log, and the service was enabled for boot persistence.

## Apache HTTP Server
- Installed httpd with dnf
- Started and enabled Apache with `systemctl enable --now httpd`
- Configured firewalld to allow HTTP on port 80 (`firewall-cmd --permanent --add-service=http`, then `--reload`)
- Verified access from the host machine
- Fixed the ServerName warning by editing /etc/httpd/conf/httpd.conf
- Used reload instead of restart to apply the change without dropping connections

## Troubleshooting a Failed Service
Deliberately broke Apache with an invalid DocumentRoot, then diagnosed and fixed it from the logs.

- `systemctl status httpd` showed the service in a failed state
- The journal output identified the exact error: the DocumentRoot on line 124 of httpd.conf pointed to a directory that did not exist
- Corrected the config, then ran `apachectl configtest` to verify before restarting
- Restarted the service and confirmed active status

### Troubleshooting Workflow
1. `systemctl status <service>` - read the error
2. Identify the problem from the log output
3. Fix the configuration
4. Run configtest or the equivalent to verify before restarting
5. Restart the service
6. Confirm active status

## Key Concepts
- **Started/stopped vs enabled/disabled** - two independent states. A service can be running but disabled, or stopped but enabled. Confirm both when troubleshooting.
- **Symlinks control boot behavior** - enable and disable work entirely through symlinks in the target.wants directories, which helps when diagnosing a service that will not start at boot.
- **Signal 15 (SIGTERM)** - the clean shutdown signal sent by `systemctl stop` and `restart`, visible in the journal as "Received signal 15; terminating."
- **Always configtest before restarting** - catches configuration errors without taking the service down.

## Key Commands
- systemctl start / stop / restart / reload / enable / disable / status
- systemctl is-enabled, systemctl enable --now, systemctl daemon-reload
- systemctl list-units --type=service --state=running
- journalctl -u <service> --since
- firewall-cmd --permanent --add-service, --reload, --list-all
- apachectl configtest
- dnf install
~
