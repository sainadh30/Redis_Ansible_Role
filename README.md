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

## Configuring Proxy Settings
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

## Setting Up Redis Repository and Installing Redis

This Ansible playbook automates the process of setting up the Redis repository, installing Redis, and ensuring its service is running. Below are the steps involved:

### 1. Check if Redis Repository GPG Key file exists:
   - This step ensures that the Redis Repository GPG Key file exists in the designated path (`/usr/share/keyrings/redis-archive-keyring.gpg`). The GPG key is essential for verifying the authenticity of packages from the Redis repository.

### 2. Add Redis Repository GPG Key:
   - If the Redis Repository GPG Key file doesn't exist, this step downloads the key from `https://packages.redis.io/gpg` and saves it to `/usr/share/keyrings/redis-archive-keyring.gpg`. Having the GPG key ensures secure installation and updates from the Redis repository.

### 3. Check if Redis Repository file exists:
   - This step verifies whether the Redis Repository file exists in `/etc/apt/sources.list.d/redis.list`. This file contains information about the Redis repository, including its URL and other configuration details.

### 4. Add Redis Repository to System:
   - If the Redis Repository file doesn't exist, this step adds the Redis repository to the system's apt sources list (`/etc/apt/sources.list.d/redis.list`). It includes the URL of the repository along with the GPG key for package verification.

### 5. Update Redis Repository:
   - After adding the Redis repository, this step updates the package lists from the repositories using the `apt update` command. It ensures that the system has the latest information about available packages from the Redis repository.

### 6. Install Redis:
   - Once the repository is configured and updated, this step installs the Redis server (`redis-server`) package using the `apt` package manager. Redis server is the core component required for running Redis.

### 7. Start and Enable Redis service:
   - After installation, this step ensures that the Redis service (`redis-server`) is started and enabled. Starting the service ensures that Redis is up and running, while enabling it ensures that Redis starts automatically on system boot.

### 8. Check Status of Redis Service:
   - This step checks the status of the Redis service using the `systemctl status redis-server.service` command. It provides information about whether the Redis service is active, its PID, and any errors if encountered.

### 9. Display Redis service status:
   - Following the status check, this step displays the status of the Redis service fetched from the previous step. It provides visibility into the current state of the Redis service.

### 10. Set Redis Package to Remain Fixed at v7.2.4 (Disable auto-update):
    - Finally, this step ensures that the Redis package remains fixed at version 7.2.4 by setting it to "hold," which disables automatic updates for the Redis package. This ensures stability and prevents unexpected changes to the Redis version.

## Installing and Configuring Redis Sentinel

This Ansible playbook automates the installation and configuration of Redis Sentinel, a distributed system designed to monitor and manage Redis instances. Below are the steps involved:

### 1. Install Redis Sentinel:
   - This step uses the `ansible.builtin.apt` module to install the Redis Sentinel package (`redis-sentinel`). Redis Sentinel is a high availability solution for Redis, providing monitoring, automatic failover, and configuration management.

### 2. Enable Redis Sentinel service to start on boot:
   - After installation, this step ensures that the Redis Sentinel service (`redis-sentinel`) is enabled to start automatically on system boot. Enabling the service ensures continuous monitoring and management of Redis instances even after system restarts.

### 3. Start Redis Sentinel service:
   - Once enabled, this step starts the Redis Sentinel service using the `ansible.builtin.service` module. Starting the service is essential to initiate the monitoring and failover mechanisms provided by Redis Sentinel.

### 4. Check Redis Sentinel service status:
   - This step verifies the status of the Redis Sentinel service using the `ansible.builtin.systemd` module. It ensures that the Redis Sentinel service is successfully started and running as expected.

### 5. Update Redis package cache:
   - Finally, this step updates the package cache for Redis using the `ansible.builtin.apt` module with the `update_cache` option set to `yes`. Updating the package cache ensures that the latest information about available Redis packages is fetched from the repositories.

## Configuring Redis Master Instance

This Ansible playbook automates the configuration of a Redis master instance by modifying its configuration file (`redis.conf`). Below are the steps involved:

### 1. Check if redis.conf file exists:
   - This step ensures the existence of the Redis configuration file (`/etc/redis/redis.conf`) on the designated Redis master server. The file is essential for configuring Redis server settings.

### 2. Store current block content of /etc/redis/redis.conf:
   - If the Redis configuration file exists, this step retrieves its current content using the `ansible.builtin.shell` module. The content is stored for reference and potential modification.

### 3. Uncomment and edit Redis configurations:
   - This step utilizes the `ansible.builtin.lineinfile` module to uncomment and modify specific configurations within the Redis configuration file (`/etc/redis/redis.conf`). It ensures that essential configurations like binding IP, password requirements, and master authentication are appropriately set.

### 4. Store the new block content of /etc/redis/redis.conf:
   - After modifying the Redis configuration file, this step retrieves the updated content using the `ansible.builtin.shell` module. The updated content is stored for verification and backup purposes.

### 5. Backup Redis configuration file:
   - If changes are made to the Redis configuration file, this step creates a backup of the original configuration file (`/etc/redis/redis.conf`) with a timestamp appended to its filename. The backup ensures the ability to revert to the previous configuration if needed.

### 6. Restart Redis service:
   - Finally, this step restarts the Redis service (`redis-server`) on the Redis master server using the `ansible.builtin.service` module. Restarting the service applies the new configuration settings.


## Configuring Redis Slave 01

This Ansible playbook automates the configuration of Redis Slave 01 by modifying its Redis configuration file (`redis.conf`). Below are the steps involved:

### 1. Check if redis.conf file exists in Redis Slave 01:
   - This step ensures the existence of the Redis configuration file (`/etc/redis/redis.conf`) on Redis Slave 01, delegated to the designated server (`redis_slave1`). The file is essential for configuring Redis server settings on the slave node.

### 2. Store current block content of /etc/redis/redis.conf in Redis Slave 01:
   - If the Redis configuration file exists on Redis Slave 01, this step retrieves its current content using the `ansible.builtin.shell` module. The content is stored for reference and potential modification.

### 3. Update Redis configuration in redis.conf for slave nodes in Redis Slave 01:
   - This step utilizes the `ansible.builtin.lineinfile` module to update specific configurations within the Redis configuration file (`/etc/redis/redis.conf`) on Redis Slave 01. It ensures essential configurations like binding IP, password requirements, and replication settings are appropriately set.

### 4. Store the new block content of /etc/redis/redis.conf in Redis Slave 01:
   - After modifying the Redis configuration file on Redis Slave 01, this step retrieves the updated content using the `ansible.builtin.shell` module. The updated content is stored for verification and backup purposes.

### 5. Backup Redis configuration file in Redis Slave 01:
   - If changes are made to the Redis configuration file on Redis Slave 01, this step creates a backup of the original configuration file (`/etc/redis/redis.conf`) with a timestamp appended to its filename. The backup ensures the ability to revert to the previous configuration if needed.

### 6. Restart Redis service in Redis Slave 01:
   - Finally, this step restarts the Redis service (`redis-server`) on Redis Slave 01 using the `ansible.builtin.service` module. Restarting the service applies the new configuration settings and ensures proper replication from the master node.

## Configuring Redis Slave 02

This Ansible playbook automates the configuration of Redis Slave 02 by modifying its Redis configuration file (`redis.conf`). Below are the steps involved:

### 1. Check if redis.conf file exists in Redis Slave 02:
   - This step ensures the existence of the Redis configuration file (`/etc/redis/redis.conf`) on Redis Slave 02, delegated to the designated server (`redis_slave1`). The file is essential for configuring Redis server settings on the slave node.

### 2. Store current block content of /etc/redis/redis.conf in Redis Slave 02:
   - If the Redis configuration file exists on Redis Slave 02, this step retrieves its current content using the `ansible.builtin.shell` module. The content is stored for reference and potential modification.

### 3. Update Redis configuration in redis.conf for slave nodes in Redis Slave 02:
   - This step utilizes the `ansible.builtin.lineinfile` module to update specific configurations within the Redis configuration file (`/etc/redis/redis.conf`) on Redis Slave 02. It ensures essential configurations like binding IP, password requirements, and replication settings are appropriately set.

### 4. Store the new block content of /etc/redis/redis.conf in Redis Slave 02:
   - After modifying the Redis configuration file on Redis Slave 02, this step retrieves the updated content using the `ansible.builtin.shell` module. The updated content is stored for verification and backup purposes.

### 5. Backup Redis configuration file in Redis Slave 02:
   - If changes are made to the Redis configuration file on Redis Slave 02, this step creates a backup of the original configuration file (`/etc/redis/redis.conf`) with a timestamp appended to its filename. The backup ensures the ability to revert to the previous configuration if needed.

### 6. Restart Redis service in Redis Slave 02:
   - Finally, this step restarts the Redis service (`redis-server`) on Redis Slave 02 using the `ansible.builtin.service` module. Restarting the service applies the new configuration settings and ensures proper replication from the master node.

## Configuring Redis Sentinel

This Ansible playbook automates the configuration of Redis Sentinel by modifying its configuration file (`sentinel.conf`). Below are the steps involved:

### 1. Check if sentinel.conf file exists:
   - This step ensures the existence of the Redis Sentinel configuration file (`/etc/redis/sentinel.conf`). The file is essential for configuring Redis Sentinel settings.

### 2. Store current block content of /etc/redis/sentinel.conf:
   - If the Redis Sentinel configuration file exists, this step retrieves its current content using the `ansible.builtin.shell` module. The content is stored for reference and potential modification.

### 3. Edit sentinel.conf file:
   - This step utilizes the `ansible.builtin.lineinfile` module to edit specific configurations within the Redis Sentinel configuration file (`/etc/redis/sentinel.conf`). It ensures essential configurations like monitoring master, authentication, failover settings, and protected mode are appropriately set.

### 4. Store the new block content of /etc/redis/sentinel.conf:
   - After modifying the Redis Sentinel configuration file, this step retrieves the updated content using the `ansible.builtin.shell` module. The updated content is stored for verification and backup purposes.

### 5. Backup Redis configuration file:
   - If changes are made to the Redis Sentinel configuration file, this step creates a backup of the original configuration file (`/etc/redis/sentinel.conf`) with a timestamp appended to its filename. The backup ensures the ability to revert to the previous configuration if needed.

### 6. Restart Redis service:
   - Finally, this step restarts the Redis service (`redis-server`) to apply the new configuration settings.

# Validation

## 1.Run Redis CLI to connect to the Redis server.
```
redis-cli
```
## 2.Run AUTH command to authenticate. Change <YOUR_PASSWORD> to the same password on master node.
```
AUTH <YOUR_PASSWORD>
```

## 3.Run INFO command to check the Redis replication information.
```
INFO REPLICATION
```

## 4.Run SET command to set a key hello to world on the master node.
```
SET hello world
```

## 5.Connect to your redis replica server.
```
redis-cli
```

## 6.Run AUTH command to authenticate. Change <YOUR_PASSWORD> to the same password on the master node.
```
AUTH <YOUR_PASSWORD>
```

## 7.Run GET command to get the value of key hello. The result is world shows that the replication from master to replica works successfully.
```
GET hello
```

## 8.Run Redis CLI to connect to the Redis Sentinel server.
```
redis-cli -p 26379
```

## 9.Run INFO command to check the Redis replication information.
```
INFO SENTINEL
```

## 10.Check the log of Redis Sentinel.
```
tail -f /var/log/redis/redis-sentinel.log
```













