# Day 3 - systemd Service Management

## Overview
Practiced managing services with systemd on RHEL 10, including 
installation, configuration, troubleshooting, and firewall management.

## Environment
- Host: CachyOS Linux
- Hypervisor: VMware Workstation Pro 25H2
- VM: Red Hat Enterprise Linux 10.1
- Kernel: 6.12.0-124.52.1.el10_1.x86_64

## Tasks Completed

### systemd Fundamentals
- Understood difference between start/stop and enable/disable
- Practiced checking service status with systemctl status
- Used grep to filter specific fields from status output
- Confirmed enabled services survive reboot, started services do not

### SSH Service Management
- Stopped, started, enabled and disabled sshd
- Configured SSH access from CachyOS host to RHEL VM
- Verified SSH connection and confirmed log entries in systemd journal

### Apache HTTP Server
- Installed httpd package with dnf
- Started and enabled Apache with systemctl enable --now
- Configured firewalld to allow HTTP traffic on port 80
- Fixed ServerName warning by editing /etc/httpd/conf/httpd.conf
- Used reload vs restart to apply config changes without downtime

### Troubleshooting a Failed Service
- Deliberately broke Apache config with invalid DocumentRoot
- Diagnosed failure using systemctl status log output
- Identified error on specific line of httpd.conf
- Used apachectl configtest to verify fix before restarting
- Restored service successfully

## Troubleshooting Workflow
1. systemctl status <service> - read the error
2. Identify the problem from log output
3. Fix the config
4. Run configtest or equivalent to verify
5. Restart the service
6. Confirm active status

## Key Commands
- systemctl start/stop/restart/reload/enable/disable/status
- systemctl is-enabled
- systemctl enable --now
- firewall-cmd --permanent --add-service=http
- firewall-cmd --reload
- firewall-cmd --list-all
- apachectl configtest
- dnf install
