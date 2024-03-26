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
      - `tasks/`: Directory containing task files.
        - `main.yml`: Main task file for Redis role.
        - `masterconf.yml`: Task file for configuring Redis on the master node.
        - `proxy.yml`: Task file for configuring proxy settings.
        - `redis.yml`: Task file for installing Redis and managing its services.
        - `sentinel.yml`: Task file for configuring Redis Sentinel.
        - `sentinelconf.yml`: Task file for configuring Redis Sentinel configuration file.
        - `slave1conf.yml`: Task file for configuring Redis on slave node 1.
        - `slave2conf.yml`: Task file for configuring Redis on slave node 2.
      - `vars/`: Directory containing variable files.
        - `main.yml`: Defines variables for Redis role.

## Usage
1. Ensure you have Ansible installed on your system.
2. Clone or download this repository.
3. Navigate to the `redis_ansible_role` directory.
4. Edit the `hosts` file to include the correct IP addresses of your Redis master and slave nodes.
5. Execute the `lampstack.yml` playbook using the command:
   ```bash
   ansible-playbook -i hosts lampstack.yml

## Explanation of Variables in `vars/main.yml`

### Proxy Variables
- **PROXY_ADDRESS**: Specifies the proxy server address to be used for internet access. In this, `http://172.168.10.28:3128` is provided as the proxy server address.

### Redis Hosts
- **redis_master**: Defines the IP address of the Redis master node.
- **redis_slave1**: Defines the IP address of the first Redis slave node.
- **redis_slave2**: Defines the IP address of the second Redis slave node.

### Redis Configuration
- **redis_master_password**: Sets the default password for accessing the Redis master. In this example, `"redis@123"` is used as the default password.
- **redis_port**: Specifies the port number on which Redis will listen for connections. The default port `6379` is provided here.

These variables are used within the Ansible role to configure and manage the Redis instances across the specified hosts. By defining these variables in the `vars/main.yml` file, it becomes easier to maintain and modify the configuration as needed without directly modifying the playbook files.

## Configuring Proxy Settings with Ansible

To configure proxy settings using Ansible, follow these steps:

### Step 1: Create the `tasks/proxy.yml` File

Create a YAML file named `proxy.yml` within your Ansible playbook directory.

### Step 2: Add Proxy Configuration Tasks

Add the following tasks to the `proxy.yml` file to configure proxy settings:

1. **Check if Proxy.sh file exists**:
   - Use Ansible's `stat` module to check if the `proxy.sh` file exists in the directory `/etc/profile.d/`.
   - Register the result in the variable `proxy_file_stat`.

2. **Create proxy.sh file if does not exist**:
   - Use Ansible's `file` module to create the `proxy.sh` file using the `touch` state if it doesn't exist.
   - Conditionally execute this task when `proxy_file_stat.stat.exists` is false.

3. **Store current block content of /etc/profile.d/proxy.sh**:
   - Use Ansible's `shell` module to fetch the current content of `proxy.sh` using the `cat` command.
   - Register the result in the variable `proxy_block_content`.
   - Execute this task when `proxy_file_stat.stat.exists` is true.

4. **Setting Proxy environmental variables**:
   - Use Ansible's `blockinfile` module to insert a block of content into the `proxy.sh` file, setting proxy environmental variables.
   - Export `no_proxy`, `https_proxy`, and `http_proxy` variables.
   - Mark the block with a comment for management tracking.

5. **Store the new block content of /etc/profile.d/proxy.sh**:
   - Fetch the current content of `proxy.sh` after modification and register it in `proxy_blockinfile_result`.

6. **Back Up /etc/profile.d/proxy.sh file if content changes**:
   - Use Ansible's `copy` module to create a backup of the `proxy.sh` file if its content has changed.
   - Backup file names include the date and time of the backup.
   - Conditionally execute this task when the content of `proxy.sh` has changed.

7. **Setting APT Proxy variables**:
   - Similar to previous steps, configure APT proxy variables in the `80proxy` file located in `/etc/apt/apt.conf.d/`.

8. **Setting WGET Proxy variables**:
   - Set up proxy variables for wget by configuring the `wgetrc` file.

These tasks collectively ensure the proper configuration of proxy settings across relevant files and directories.


Copy code
# Installing Redis with Ansible

To install Redis using Ansible, follow these steps:

### Step 1: Check Redis Repository GPG Key

Check if the Redis Repository GPG Key file exists:

```yaml
- name: Check if Redis Repository GPG Key file exists
  ansible.builtin.stat:
    path: /usr/share/keyrings/redis-archive-keyring.gpg
  register: redis_key_stat
Step 2: Add Redis Repository GPG Key
If the Redis Repository GPG Key does not exist, add it:

- name: Add Redis Repository GPG Key
  ansible.builtin.shell:
    cmd: curl -fsSL https://packages.redis.io/gpg | sudo gpg --dearmor -o /usr/share/keyrings/redis-archive-keyring.gpg
  when: not redis_key_stat.stat.exists
Step 3: Check Redis Repository File
Check if the Redis Repository file exists:

- name: Check if Redis Repository file exists
  ansible.builtin.stat:
    path: /etc/apt/sources.list.d/redis.list
  register: redis_repo_stat
Step 4: Add Redis Repository to System
If the Redis Repository file does not exist, add it to the system:

- name: Add Redis Repository to System
  ansible.builtin.shell:
    cmd: echo "deb [signed-by=/usr/share/keyrings/redis-archive-keyring.gpg] https://packages.redis.io/deb $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/redis.list
  when: not redis_repo_stat.stat.exists
Step 5: Update Redis Repository
Update the Redis repository:

- name: Update Redis Repository
  ansible.builtin.apt:
    update_cache: yes
Step 6: Install Redis
Install Redis using APT:

- name: Install Redis
  ansible.builtin.apt:
    name: redis-server
    state: present
Step 7: Start and Enable Redis Service
Start and enable the Redis service:

- name: Start and Enable Redis service
  ansible.builtin.systemd:
    name: redis-server
    state: started
    enabled: yes
Step 8: Check Status of Redis Service
Check the status of the Redis service:

- name: Check Status of Redis Service
  ansible.builtin.shell:
    cmd: systemctl status redis-server.service
  register: redis_service_status
Step 9: Display Redis Service Status
Display the status of the Redis service:

- name: Display Redis service status
  ansible.builtin.debug:
    msg: "{{ redis_service_status.stdout }}"
Step 10: Set Redis Package to Remain Fixed
Set the Redis package to remain fixed at a specific version (e.g., v7.2.4) to disable auto-update:


- name: Set Redis Package to Remain Fixed at v7.2.4 (Disable auto-update)
  ansible.builtin.command:
    cmd: echo "redis-server hold" | sudo dpkg --set-selections
These tasks collectively ensure the installation of Redis on your system using Ansible.

## Configuring Redis Sentinel with Ansible

To configure Redis Sentinel using Ansible, follow these steps:

### Step 1: Create the `tasks/sentinel.yml` File

Create a YAML file named `sentinel.yml` within your Ansible playbook directory.

### Step 2: Add Redis Sentinel Configuration Tasks

Add the following tasks to the `sentinel.yml` file to configure Redis Sentinel:

1. **Install Redis Sentinel**:
   - Use Ansible's `apt` module to install the `redis-sentinel` package.
   - Set the state to `present` to ensure the package is installed.

2. **Enable Redis Sentinel service to start on boot**:
   - Use Ansible's `service` module to enable the `redis-sentinel` service to start automatically on boot.
   - Set `enabled: yes` to enable the service.

3. **Start Redis Sentinel service**:
   - Use Ansible's `service` module to start the `redis-sentinel` service.
   - Set the state to `started` to ensure the service is running.

4. **Check Redis Sentinel service status**:
   - Use Ansible's `systemd` module to check the status of the `redis-sentinel` service.
   - Set the state to `started` to retrieve the service status.

5. **Update Redis package cache**:
   - Use Ansible's `apt` module to update the Redis package cache.

These tasks collectively ensure the proper installation and configuration of Redis Sentinel, along with updating the Redis package cache when necessary.

## Configuring Redis Configuration File with Ansible

To configure the Redis configuration file using Ansible, follow these steps:

### Step 1: Check if `redis.conf` File Exists

Use the `stat` module to check if the `redis.conf` file exists in the directory `/etc/redis/`.
Register the result in the variable `redis_file_stat`.
Delegate the task to the Redis master server.

### Step 2: Store Current Block Content of `redis.conf`

Use the `shell` module to fetch the current content of `redis.conf` using the `cat` command.
Register the result in the variable `redis_block_content`.
Execute this task when `redis_file_stat.stat.exists` is true.
Delegate the task to the Redis master server.

### Step 3: Uncomment and Edit Redis Configurations

Use the `lineinfile` module to uncomment and edit specific Redis configurations in the `redis.conf` file.
Loop through the configurations to be edited, such as `bind`, `requirepass`, and `masterauth`.
Delegate the task to the Redis master server.

### Step 4: Store the New Block Content of `redis.conf`

Fetch the current content of `redis.conf` after modification and register it in `redis_blockinfile_result`.
Delegate the task to the Redis master server.

### Step 5: Backup Redis Configuration File

Create a backup of the `redis.conf` file if its content has changed.
Backup file names include the date and time of the backup.
Conditionally execute this task when the content of `redis.conf` has changed.
Delegate the task to the Redis master server.

### Step 6: Restart Redis Service

Use the `service` module to restart the `redis-server` service.
Delegate the task to the Redis master server.

These tasks collectively ensure the proper configuration of the Redis configuration file, including uncommenting and editing specific configurations as required.

## Configuring Redis Configuration File on Redis Slave 01 with Ansible

To configure the Redis configuration file on Redis Slave 01 using Ansible, follow these steps:

### Step 1: Check if `redis.conf` File Exists on Redis Slave 01

Use the `stat` module to check if the `redis.conf` file exists in the directory `/etc/redis/` on Redis Slave 01.
Register the result in the variable `redis_file_stat`.
Delegate the task to Redis Slave 01.

### Step 2: Store Current Block Content of `redis.conf` on Redis Slave 01

Use the `shell` module to fetch the current content of `redis.conf` using the `cat` command on Redis Slave 01.
Register the result in the variable `redis_block_content`.
Execute this task when `redis_file_stat.stat.exists` is true.
Delegate the task to Redis Slave 01.

### Step 3: Update Redis Configuration in `redis.conf` for Slave Nodes on Redis Slave 01

Use the `lineinfile` module to update specific Redis configurations in the `redis.conf` file for slave nodes on Redis Slave 01.
Loop through the configurations to be edited, such as `bind`, `requirepass`, `masterauth`, and `replicaof`.
Delegate the task to Redis Slave 01.

### Step 4: Store the New Block Content of `redis.conf` on Redis Slave 01

Fetch the current content of `redis.conf` after modification and register it in `redis_blockinfile_result` on Redis Slave 01.
Delegate the task to Redis Slave 01.

### Step 5: Backup Redis Configuration File on Redis Slave 01

Create a backup of the `redis.conf` file if its content has changed on Redis Slave 01.
Backup file names include the date and time of the backup.
Conditionally execute this task when the content of `redis.conf` has changed.
Delegate the task to Redis Slave 01.

### Step 6: Restart Redis Service on Redis Slave 01

Use the `service` module to restart the `redis-server` service on Redis Slave 01.
Delegate the task to Redis Slave 01.

These tasks collectively ensure the proper configuration of the Redis configuration file on Redis Slave 01, including updating specific configurations for slave nodes.

## Configuring Redis Configuration File on Redis Slave 02 with Ansible

To configure the Redis configuration file on Redis Slave 02 using Ansible, follow these steps:

### Step 1: Check if `redis.conf` File Exists on Redis Slave 02

Use the `stat` module to check if the `redis.conf` file exists in the directory `/etc/redis/` on Redis Slave 02.
Register the result in the variable `redis_file_stat`.
Delegate the task to Redis Slave 01.

### Step 2: Store Current Block Content of `redis.conf` on Redis Slave 02

Use the `shell` module to fetch the current content of `redis.conf` using the `cat` command on Redis Slave 02.
Register the result in the variable `redis_block_content`.
Execute this task when `redis_file_stat.stat.exists` is true.
Delegate the task to Redis Slave 01.

### Step 3: Update Redis Configuration in `redis.conf` for Slave Nodes on Redis Slave 02

Use the `lineinfile` module to update specific Redis configurations in the `redis.conf` file for slave nodes on Redis Slave 02.
Loop through the configurations to be edited, such as `bind`, `requirepass`, `masterauth`, and `replicaof`.
Delegate the task to Redis Slave 01.

### Step 4: Store the New Block Content of `redis.conf` on Redis Slave 02

Fetch the current content of `redis.conf` after modification and register it in `redis_blockinfile_result` on Redis Slave 01.
Delegate the task to Redis Slave 01.

### Step 5: Backup Redis Configuration File on Redis Slave 02

Create a backup of the `redis.conf` file if its content has changed on Redis Slave 02.
Backup file names include the date and time of the backup.
Conditionally execute this task when the content of `redis.conf` has changed.
Delegate the task to Redis Slave 01.

### Step 6: Restart Redis Service on Redis Slave 02

Use the `service` module to restart the `redis-server` service on Redis Slave 02.
Delegate the task to Redis Slave 01.

These tasks collectively ensure the proper configuration of the Redis configuration file on Redis Slave 02, including updating specific configurations for slave nodes.

## Configuring Sentinel Configuration File for Redis with Ansible

To configure the Sentinel configuration file for Redis using Ansible, follow these steps:

### Step 1: Check if `sentinel.conf` File Exists

Use the `stat` module to check if the `sentinel.conf` file exists in the directory `/etc/redis/`.
Register the result in the variable `sentinel_file_stat`.

### Step 2: Store Current Block Content of `sentinel.conf`

Use the `shell` module to fetch the current content of `sentinel.conf` using the `cat` command.
Register the result in the variable `sentinel_block_content`.
Execute this task when `sentinel_file_stat.stat.exists` is true.

### Step 3: Edit `sentinel.conf` File

Use the `lineinfile` module to edit specific configurations in the `sentinel.conf` file.
Loop through the configurations to be edited, such as `sentinel monitor`, `sentinel auth-pass`, `sentinel down-after-milliseconds`, `sentinel failover-timeout`, and `protected-mode`.

### Step 4: Store the New Block Content of `sentinel.conf`

Fetch the current content of `sentinel.conf` after modification and register it in `sentinel_blockinfile_result`.

### Step 5: Backup Redis Configuration File

Create a backup of the `sentinel.conf` file if its content has changed.
Backup file names include the date and time of the backup.
Conditionally execute this task when the content of `sentinel.conf` has changed.

### Step 6: Restart Redis Service

Use the `service` module to restart the `redis-server` service.

These tasks collectively ensure the proper configuration of the Sentinel configuration file for Redis.










