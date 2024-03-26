# Redis High Availability Ansible Role

## Overview
This Ansible role is designed to set up a Redis high availability configuration using Ansible. It automates the process of installing, configuring, and managing Redis services across multiple nodes.

## Directory Structure
- `redis_ansible_role/`
  - `hosts`: Contains details of the Redis hosts.
    ```
    [redis]
    redis_master ansible_host=172.31.81.82
    redis_slave1 ansible_host=172.31.28.179
    redis_slave2 ansible_host=172.31.34.174
    ```
  - `lampstack.yml`: Ansible playbook file to install, configure, and manage Redis services.
    ```yaml
    ---
    - name: Install Configure and Manage Redis Services
      hosts: all
      become: yes

      roles:
        - redis
    ```
  - `roles/`: Directory containing Ansible roles.
    - `redis/`: Redis role directory created using the `ansible-galaxy` command.

## Usage
1. Ensure you have Ansible installed on your system.
2. Clone or download this repository.
3. Navigate to the `redis_ansible_role` directory.
4. Edit the `hosts` file to include the correct IP addresses of your Redis master and slave nodes.
5. Execute the `lampstack.yml` playbook using the command:
ansible-playbook -i hosts lampstack.yml
