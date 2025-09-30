## Part 1: Configuring Managed Nodes (Agentless Access)

Ansible's power comes from its **agentless** nature, relying on standard protocols already on the Managed Node.

### 1\. The SSH Connection

Ansible uses the **SSH connection plugin** (default) for communication. This requires:

1.  **Control Node:** SSH **client** (e.g., OpenSSH) and **Python** (for running Ansible).
2.  **Managed Node:** SSH **server** and a **Python interpreter** (for executing Ansible modules/tasks).

### 2\. Key-Based Authentication (The Gold Standard)

Password authentication is insecure and breaks automation. **Key-based authentication** is required for production environments.

#### Setup Steps:

1.  **Generate Key Pair (on Control Node):**
    ```bash
    ssh-keygen -t rsa -b 4096
    # Creates: ~/.ssh/id_rsa (Private) and ~/.ssh/id_rsa.pub (Public)
    ```
2.  **Distribute Public Key (to Managed Node):**
    The public key is copied to the target user's `~/.ssh/authorized_keys` file.
    ```bash
    ssh-copy-id user@managed_node_ip
    ```
3.  **Use `ssh-agent`:** Recommended for managing keys protected by a passphrase.

### 3\. Privilege Escalation (`become`)

Ansible needs root privileges for most administrative tasks, achieved via the `become` directives (usually `sudo`).

| Ansible Directive | Purpose | Inventory/Playbook Setting |
| :--- | :--- | :--- |
| `ansible_become: yes` | Enables privilege escalation. | Set as `yes` in inventory or play. |
| `ansible_become_method: sudo` | Specifies the tool (default is `sudo`). | |
| `ansible_become_user: root` | The user to become (defaults to `root`). | |

**Key Best Practice:** Configure the remote user for `NOPASSWD` sudo access via the `/etc/sudoers` file to eliminate password prompts during automation.

-----

## Part 2: Inventory Management

The **Inventory** is a file (or set of files/scripts) that lists the Managed Nodes and organizes them into logical groups, along with connection variables.

### 1\. Inventory Formats

Ansible supports INI and YAML formats for static inventories.

| Format | Structure | Pros | Cons |
| :--- | :--- | :--- | :--- |
| **INI-like** | Section headers (`[]`), simple `key=value`. | Easiest to read, simple syntax. | Poorly handles complex, nested variables. |
| **YAML** | Hierarchical, indentation-based (lists, dictionaries). | Excellent for complex variables, better readability/structure at scale. | Highly sensitive to indentation errors. |

### 2\. Static Inventory Basics (INI Example)

```ini
[webservers]
web1.prod.local ansible_host=10.0.1.10
web2.prod.local

[dbservers]
db_master
db_replica

[backend:children]
webservers
dbservers

[all:vars]
ansible_user=deployer
ansible_private_key_file=/home/user/.ssh/prod_key
```

### 3\. YAML Inventory Example

YAML inventories are structured around the top-level group `all`, which contains `children` (groups) and global `vars`. This structure closely mirrors the data hierarchy Ansible uses internally.

**(File: `inventory.yml`)**

```yaml
---
# The root of the entire inventory
all:
  # Global variables applied to every host
  vars:
    ansible_user: devops_user
    ansible_connection: ssh
    ansible_become: true

  # The children key defines all host groups
  children:

    webservers:
      # Hosts belonging to the 'webservers' group
      hosts:
        web01.example.com:
          # Host-specific variable overrides (inline)
          ansible_host: 192.168.1.10
          http_port: 80
        web02.example.com:
          ansible_host: 192.168.1.11

      # Group-level variables for 'webservers'
      vars:
        app_environment: production

    dbservers:
      hosts:
        db01.example.com:
          ansible_host: 192.168.1.20
          # Override global user for this host
          ansible_user: db_admin
        db02.example.com:
          ansible_host: 192.168.1.21

    # Nested Groups: The 'app_tier' group contains the 'webservers' and 'dbservers' groups
    app_tier:
      children:
        webservers:
        dbservers:
```

### 4\. Static vs. Dynamic Inventory

| Feature | Static Inventory | Dynamic Inventory |
| :--- | :--- | :--- |
| **Definition** | Manually written file (INI/YAML). | Generated at runtime by scripts/plugins. |
| **Source of Truth**| The local file system. | External system (Cloud API, CMDB, etc.). |
| **Ideal For** | Small, stable, or on-prem environments. | Large, elastic, or multi-cloud infrastructures. |

**Dynamic Inventory** is critical for cloud environments (AWS, Azure, GCP) where server IPs and host counts change frequently due to auto-scaling or ephemeral workloads. It guarantees that Ansible targets the correct, currently running machines.

### 5\. Inventory Verification

Use the `ansible-inventory` command to test your inventory structure and variable loading:

```bash
# List all hosts, groups, and variables in JSON format
ansible-inventory -i inventory.yml --list

# Show a simple graphical view of groups and membership
ansible-inventory -i inventory.yml --graph
```
