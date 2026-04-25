# Day 1 & 2 - User and Group Management

## Overview
Configured user accounts, group membership, file permissions, and sudo access on RHEL 10.

## Environment
- Host: CachyOS Linux
- Hypervisor: VMware Workstation Pro 25H2
- VM: Red Hat Enterprise Linux 10.1
- Kernel: 6.12.0-124.52.1.el10_1.x86_64

## Tasks Completed

### User Management
- Created regular user accounts with useradd (-m, -c, -s flags)
- Created locked service account with no login shell and no home directory
- Set passwords and verified yescrypt hashed entries in /etc/shadow

### Group Management
- Created sysadmins and developers groups with groupadd
- Assigned users to groups with usermod -aG
- Verified membership with id and getent commands

### File Permissions
- Reviewed octal notation (chmod) and ownership (chown)
- Created /labfiles directory owned by jsmith:sysadmins with 770 permissions
- Verified jdoe blocked from accessing restricted directory

### Sudo Configuration
- Edited /etc/sudoers safely with visudo
- Implemented least privilege sudo restricting jsmith to systemctl commands only
- Verified restriction works correctly after removing wheel group access

## Key Commands
- useradd, userdel, usermod
- groupadd, groupdel, gpasswd
- chmod, chown
- visudo, sudo
- id, getent, groups
