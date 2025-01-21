# Monitoring Deployment Guide using Ansible

## Overview
This document describes the process of deploying a containerized monitoring system using various Prometheus exporters (Node Exporter, MySQL Exporter, Blackbox Exporter, cAdvisor) using Ansible.

### Note on Architecture
By default, all roles and configurations in this repository are tailored for ARM-based Linux systems. This ensures compatibility with a wide range of modern servers, including those running on ARM architecture. 

If your environment uses a standard x86 architecture, you can easily adapt the deployment by adjusting the relevant architecture-specific variables in `group_vars/all.yml`. Refer to the **Configuration** section for more details.

## Project Structure
```
.
├── ansible.cfg
├── inventory.ini
├── group_vars
│   └── all.yml
├── roles
│   ├── blackbox_exporter
│   ├── cadvisor
│   ├── mysql_exporter
│   └── node_exporter
└── site.yml
```

## Prerequisites

### Local Environment
1. Ansible installed (recommended version 2.18 or higher)
2. SSH keys for hosts access
3. Configured network connection to target hosts

### Target Server Requirements
1. Operating System: ARM Linux
2. Open SSH access
3. User with sudo privileges
4. For MySQL Exporter: configured database access

## Configuration

### 1. Inventory File Setup (inventory.ini)
Ensure that the inventory.ini file contains correct:
- IP addresses or hostnames of servers
- SSH key paths
- Connection usernames

### 2. Variable Verification
In the `group_vars/all.yml` file, configure:
- Exporter versions
- MySQL connection parameters
- Target system architecture

## Deployment Execution

### Host Availability Check
Before running the playbook, verify server accessibility:
```bash
ansible all -i inventory.ini -m ping
```

### Deployment Options

1. **Deploy to All Servers**
```bash
ansible-playbook site.yml
```

2. **Environment-Specific Deployment**
```bash
# Deploy to dev environment
ansible-playbook site.yml --limit dev

# Deploy to stage environment
ansible-playbook site.yml --limit stage

# Deploy to prod environment
ansible-playbook site.yml --limit prod
```

3. **Cloud Provider-Specific Deployment**
```bash
# AWS servers only
ansible-playbook site.yml --limit aws

# Oracle servers only
ansible-playbook site.yml --limit oracle
```

4. **Single Server Deployment**
```bash
ansible-playbook site.yml --limit server-hostname
```

### Additional Deployment Parameters

- `--check`: Dry run without making changes
```bash
ansible-playbook site.yml --check
```

- `--diff`: Show file changes
```bash
ansible-playbook site.yml --diff
```

- `-v` or `-vv`: Verbose output for debugging
```bash
ansible-playbook site.yml -vv
```

## Installation Verification

After successful deployment, verify:

1. Service Status:
```bash
ansible all -i inventory.ini -m shell -a "systemctl status node_exporter"
ansible all -i inventory.ini -m shell -a "systemctl status mysqld_exporter"
ansible all -i inventory.ini -m shell -a "systemctl status blackbox_exporter"
```

2. Metrics Accessibility:
- Node Exporter: http://server:9100/metrics
- MySQL Exporter: http://server:9104/metrics
- Blackbox Exporter: http://server:9115/metrics
- cAdvisor: http://server:8080/metrics

## Troubleshooting

### Common Issues and Solutions

1. **SSH Connection Errors**
- Verify SSH key paths
- Check key permissions (600)
- Confirm server network accessibility

2. **Exporter Installation Errors**
- Verify exporter versions in group_vars/all.yml
- Confirm architecture (arch) settings
- Check network connectivity for package downloads

3. **MySQL Exporter Issues**
- Verify MySQL credentials
- Check database network accessibility
- Confirm database user permissions

### Logs and Debugging

For detailed problem analysis:
```bash
# View service logs
ansible all -i inventory.ini -m shell -a "journalctl -u node_exporter -n 50"

# Run playbook with verbose output
ansible-playbook site.yml -vvv
```

## Maintenance

### Exporter Updates

1. Modify versions in `group_vars/all.yml`
2. Re-run the playbook:
```bash
ansible-playbook site.yml
```

### Configuration Backup
It is recommended to maintain configuration files in a version control system.

## Security Considerations

1. Regularly update exporter versions
2. Use secure passwords for MySQL
3. Configure firewalls to restrict access to exporter ports
4. Store sensitive data in encrypted format (ansible-vault)

## Support

When encountering issues:
1. Check the "Troubleshooting" section in this documentation
2. Consult official exporter documentation
3. Review service logs on target servers
