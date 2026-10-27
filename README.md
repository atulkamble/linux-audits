# Linux Audit Control Commands (`auditctl`)

Linux auditing is used to monitor user activities, track file changes, record system events, and investigate security incidents using the Linux Audit Framework (`auditd`).

## 1. Install and start Audit Service

```
# RHEL / CentOS / Amazon Linux
sudo dnf install audit -y

# Ubuntu / Debian
sudo apt update
sudo apt install auditd audispd-plugins -y

# Enable audit service
sudo systemctl enable auditd

# Start audit service
sudo systemctl start auditd

# Check status
sudo systemctl status auditd
```

## 2. Important Linux Audit Commands

| Command             | Purpose                               |
| ------------------- | ------------------------------------- |
| `auditctl -s`       | Check auditing status                 |
| `auditctl -l`       | List active audit rules               |
| `auditctl -w`       | Monitor a file or directory           |
| `auditctl -W`       | Remove a file watch                   |
| `auditctl -D`       | Delete all audit rules                |
| `auditctl -e 1`     | Enable auditing                       |
| `auditctl -e 0`     | Disable auditing                      |
| `auditctl -e 2`     | Lock audit configuration until reboot |
| `auditctl -b 8192`  | Set audit backlog limit               |
| `ausearch`          | Search audit events                   |
| `aureport`          | Generate audit reports                |
| `augenrules --load` | Load persistent audit rules           |

## 3. Practical Audit Commands

### Monitor important files

```
# Monitor user account changes
sudo auditctl -w /etc/passwd -p wa -k user_changes

# Monitor password changes
sudo auditctl -w /etc/shadow -p wa -k password_changes

# Monitor sudo configuration
sudo auditctl -w /etc/sudoers -p wa -k sudo_changes

# Monitor SSH configuration
sudo auditctl -w /etc/ssh/sshd_config -p wa -k ssh_changes

# Monitor a directory
sudo auditctl -w /home -p wa -k home_changes
```

Audit permission flags:

| Flag | Meaning           |
| ---- | ----------------- |
| `r`  | Read              |
| `w`  | Write             |
| `x`  | Execute           |
| `a`  | Attribute changes |

### Monitor command execution

```
# Monitor execution of a specific command
sudo auditctl -w /usr/bin/sudo -p x -k sudo_execution

# Monitor user account creation command
sudo auditctl -w /usr/sbin/useradd -p x -k user_creation
```

## 4. Search Audit Logs

```
# View all audit events
sudo ausearch -i

# Search by audit rule key
sudo ausearch -k user_changes -i

# Search failed events
sudo ausearch --success no -i

# Search events for a user ID
sudo ausearch -ui 1000 -i

# Search today's events
sudo ausearch -ts today -i

# Search command execution
sudo ausearch -k sudo_execution -i

# View audit log file
sudo tail -f /var/log/audit/audit.log
```

## 5. Generate Audit Reports

```
# Summary report
sudo aureport

# Authentication report
sudo aureport -au

# Login report
sudo aureport -l

# Failed events
sudo aureport --failed

# Executable report
sudo aureport -x

# File access report
sudo aureport -f

# User report
sudo aureport -u
```

## 6. Make Audit Rules Permanent

Rules added using `auditctl` are generally temporary and disappear after reboot.

```
# Create persistent rules
sudo vi /etc/audit/rules.d/custom.rules
```

Add:

```
-w /etc/passwd -p wa -k user_changes
-w /etc/shadow -p wa -k password_changes
-w /etc/sudoers -p wa -k sudo_changes
-w /etc/ssh/sshd_config -p wa -k ssh_changes
```

Load and verify:

```
sudo augenrules --load
sudo auditctl -l
```

## 7. Linux Auditing Architecture

Users / Applications / System Calls

Linux Kernel Audit Subsystem

auditd — Audit Daemon

/var/log/audit/audit.log

ausearch

Search events

aureport

Generate reports

## 8. Hands-on Lab: Audit File Changes

```
# Step 1: Create test file
sudo touch /tmp/audit-test.txt

# Step 2: Enable file monitoring
sudo auditctl -w /tmp/audit-test.txt -p wa -k test_audit

# Step 3: Modify file
echo "Linux Audit Testing" | sudo tee -a /tmp/audit-test.txt

# Step 4: Search audit events
sudo ausearch -k test_audit -i

# Step 5: View active rules
sudo auditctl -l

# Step 6: Remove test rule
sudo auditctl -W /tmp/audit-test.txt -p wa -k test_audit
```

Points to remember: `auditctl` manages kernel audit rules, `auditd` collects audit events, `ausearch` searches logs, and `aureport` generates reports. Use `auditctl -e 2` carefully because it locks audit configuration until reboot.
