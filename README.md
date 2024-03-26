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
    - `redis/`: Redis role directory created using the `ansible-galaxy init redis` command.

## Usage
1. Ensure you have Ansible installed on your system.
2. Clone or download this repository.
3. Navigate to the `redis_ansible_role` directory.
4. Edit the `hosts` file to include the correct IP addresses of your Redis master and slave nodes.
5. Execute the `lampstack.yml` playbook using the command:
ansible-playbook -i hosts lampstack.yml

## Directory Structure
- `redis_ansible_role/`
  - `roles/`: Directory containing Ansible roles.
    - `redis/`: Redis role directory created using the `ansible-galaxy` command.
      - `vars/`: Directory containing variable files.
        - `main.yml`: Defines variables for Redis role.

## Explanation of Variables in `vars/main.yml`

### Proxy Variables
- **PROXY_ADDRESS**: Specifies the proxy server address to be used for internet access. In this example, `http://172.168.10.28:3128` is provided as the proxy server address.

### Redis Hosts
- **redis_master**: Defines the IP address of the Redis master node.
- **redis_slave1**: Defines the IP address of the first Redis slave node.
- **redis_slave2**: Defines the IP address of the second Redis slave node.

### Redis Configuration
- **redis_master_password**: Sets the default password for accessing the Redis master. In this example, `"redis@123"` is used as the default password.
- **redis_port**: Specifies the port number on which Redis will listen for connections. The default port `6379` is provided here.

These variables are used within the Ansible role to configure and manage the Redis instances across the specified hosts. By defining these variables in the `vars/main.yml` file, it becomes easier to maintain and modify the configuration as needed without directly modifying the playbook files.

