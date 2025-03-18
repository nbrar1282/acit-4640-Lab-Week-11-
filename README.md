# Ansible Configuration Updates

## Overview
This update improves the Ansible setup by ensuring correct SSH user assignment, refactoring the playbook into roles, and configuring a dynamic inventory for EC2 instances running Rocky Linux and Ubuntu.

## Key Changes

### 1. Refactored Playbook (`play.yml`)
* Moved tasks into separate roles:
   * `redis_server` for Rocky Linux instances
   * `frontend_servers` for Ubuntu instances
* Set `ansible_user` dynamically using `vars` in `play.yml`:

```
vars:
  ansible_user: "{{ 'rocky' if 'server_role_redis_server' in group_names else 'ubuntu' }}"
```

* This ensures that Rocky Linux instances use the `rocky` user and Ubuntu instances use the `ubuntu` user.

### 2. Dynamic Inventory (`aws_ec2.yml`)
* Configured Ansible to retrieve EC2 instance details dynamically.
* Ensured hosts are grouped by their `tags.Role`:

```
keyed_groups:
  - key: tags.Role
    prefix: "server_role"
    separator: "_"
```

* Uses the `public_dns_name` for SSH connections:

```
compose:
  ansible_host: public_dns_name
```

### 3. SSH Key Configuration
* Updated `ansible.cfg` to ensure Ansible uses the correct SSH key:

```
[defaults]
private_key_file = ~/.ssh/aws
host_key_checking = False
```

* Verified SSH access manually before running Ansible:

```
ssh -i ~/.ssh/aws rocky@<Rocky_EC2_Public_IP>
ssh -i ~/.ssh/aws ubuntu@<Ubuntu_EC2_Public_IP>
```

### 4. Testing and Debugging
* Validated dynamic inventory using:

```
ansible-inventory -i inventory/aws_ec2.yml --list | grep ansible_user
```

* Ran Ansible with verbose logging to troubleshoot any issues:

```
ansible-playbook -i inventory/aws_ec2.yml play.yml -vvv
```

## Summary
* The playbook now dynamically assigns `ansible_user` based on the server role.
* The inventory is fully dynamic, retrieving EC2 details automatically.
* SSH access issues are resolved, and the correct key is used.
* The setup is modular, scalable, and works without manually specifying hostnames or IP addresses.
