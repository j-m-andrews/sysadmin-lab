# Day 3 - systemd Service Management

## Overview
systemd is the init system and service manager for RHEL. Every service on the 
system is managed through systemd — starting, stopping, enabling, disabling, 
and monitoring services are core daily tasks for a systems administrator.

## Commands Covered

### Viewing Running Services
```bash
systemctl list-units --type=service --state=running
```
Lists all currently active services on the system.

### Checking Service Status
```bash
systemctl status sshd
```
Returns the current state of a service including load status, active state, 
main PID, memory/CPU usage, CGroup processes, and recent journal log entries.

Key fields to read:
- **Loaded** — whether the unit file was found and if the service is enabled/disabled at boot
- **Active** — current runtime state (running, inactive, failed)
- **Main PID** — process ID of the service, useful for deeper troubleshooting
- **CGroup** — all processes running under this service tracked by systemd

### Starting and Stopping Services
```bash
sudo systemctl stop sshd
sudo systemctl start sshd
```
Controls whether a service is running right now. Does not affect boot behavior.

### Enabling and Disabling Services
```bash
sudo systemctl enable labmonitor
sudo systemctl disable cups
```
Controls whether a service starts automatically at boot. Works by creating or 
removing symlinks in systemd target directories (e.g. 
/etc/systemd/system/multi-user.target.wants/).

Enabling a service does not start it immediately. Disabling does not stop it 
immediately. Start/stop and enable/disable are independent operations.

### Checking Enable State
```bash
systemctl is-enabled sshd
```
Returns enabled or disabled without the full status output.

### Restarting vs Reloading
```bash
sudo systemctl restart sshd
sudo systemctl reload firewalld
```
- **restart** — stops and starts the service fresh. Drops all active connections.
- **reload** — signals the running process to re-read its configuration without 
stopping. Active connections are preserved. Always prefer reload in production 
when applying config changes to services like sshd or firewalld.

### Reading Logs with journalctl
```bash
journalctl -u sshd
journalctl -u sshd --since "1 hour ago"
```
Pulls all journal log entries for a specific service. The --since flag filters 
by time window. Primary tool for diagnosing service failures — look for error 
messages, PID changes, and timestamps to correlate with incidents.

## Attack Surface Reduction — Disabling cups

cups (Common Unix Printing System) was stopped and disabled on this system. 
No server role requires a print spooler. cups listens on port 631 by default, 
representing unnecessary network exposure.

STIG V-257942 requires cups to be disabled on RHEL systems not designated as 
print servers. Disabling unnecessary services is a baseline hardening 
requirement in DoD environments.

```bash
sudo systemctl stop cups
sudo systemctl disable cups
```

Output confirmed removal of four symlinks from sockets, multi-user, and 
printer targets.

## Custom systemd Unit File

Created a custom service unit from scratch at 
/etc/systemd/system/labmonitor.service to demonstrate unit file structure.

```ini
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

### Unit File Sections
- **[Unit]** — metadata and dependencies. After=network.target ensures the 
service starts only after networking is up.
- **[Service]** — defines how the service runs. Restart=on-failure tells 
systemd to automatically restart the process if it crashes.
- **[Install]** — defines boot behavior. WantedBy=multi-user.target adds this 
service to the standard multi-user boot target.

### Activating a New Unit File
```bash
sudo systemctl daemon-reload
```
Must be run after creating or modifying any unit file. Tells systemd to 
re-read all unit files on disk before attempting to start the service.

Service was started, verified running with two tracked CGroup processes (bash 
loop + sleep child), log output confirmed at /var/log/labmonitor.log, then 
enabled for boot persistence.

## Key Concepts

**stopped/started vs enabled/disabled** — Two independent states. A service 
can be running but disabled (won't survive reboot) or stopped but enabled 
(will start on next boot). Always confirm both states when troubleshooting.

**Symlinks = boot behavior** — systemd enable/disable works entirely through 
symlinks in target.wants directories. Understanding this helps when 
troubleshooting services that won't start at boot.

**Signal 15 (SIGTERM)** — Clean shutdown signal sent by systemctl stop and 
restart. Visible in journal logs as "Received signal 15; terminating."
