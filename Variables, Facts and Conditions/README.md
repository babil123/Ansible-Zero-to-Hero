# Variables, Facts and Conditions


##  What Are Ansible Variables?

Ansible variables are dynamic values used within playbooks, roles, and inventory to enable **customization, flexibility, and reusability** of configurations. Instead of hard-coding specific values (like usernames, package names, or port numbers) directly into tasks, you use variables as placeholders.

  * **Purpose:** Variables allow the same playbook or role to be applied across different environments, systems, or contexts by simply changing the variable values.
  * **Syntax:** Variables are referenced using the Jinja2 templating syntax, enclosed in double curly braces:  `{{ variable_name }}`.
  * **Naming Rules:** Variable names must:
      * Contain only letters, numbers, and underscores.
      * Start with a letter or an underscore.
      * Avoid reserved Python and Ansible playbook keywords.

## Different Ways to Define Variables (In-Depth with Examples)

Ansible provides numerous ways to define variables, each with a specific scope and purpose.

## Defining Variables in the Playbook (`vars`)

Variables can be defined directly within a playbook using the `vars:` section at the play level. These variables are available to all tasks within that play.

**Example: `my_playbook.yml`**

```yaml
---
- name: Example Playbook with Play Variables
  hosts: webservers
  vars:
    http_port: 8080
    app_name: "MyWebApp"
  tasks:
    - name: Ensure web package is installed
      ansible.builtin.package:
        name: httpd
        state: present

    - name: Start web service and enable on boot
      ansible.builtin.service:
        name: httpd
        state: started
        enabled: yes

    - name: Print the application port
      ansible.builtin.debug:
        msg: "The application {{ app_name }} is running on port {{ http_port }}"
```

## Defining Variables in the Inventory

Variables can be set in the inventory file (`hosts` or `inventory.ini`/`.yml`) to be specific to a host or a group of hosts.

##### A. Host Variables (Specific to a single host)

Defined on the same line as the host in the INI format, or nested under the host in YAML.

**Example: `inventory.ini`**

```ini
[webservers]
web1 ansible_host=192.168.1.10 max_clients=50
web2 ansible_host=192.168.1.11 max_clients=100
```

In a playbook, `max_clients` would resolve to `50` on `web1` and `100` on `web2`.

##### B. Group Variables (Specific to a group of hosts)

Defined in a `:[group_name]:vars` section in INI, or within the group definition in YAML.

**Example: `inventory.ini`**

```ini
[webservers]
web1
web2

[webservers:vars]
package_state=present
```

The variable `package_state` is available to both `web1` and `web2`.

## Using Separate Host and Group Variable Files (`host_vars/` and `group_vars/`)

This is a best practice for managing large variable sets outside of the inventory file, improving organization and readability. Ansible automatically loads variable files located in the `host_vars/` and `group_vars/` directories, which are typically placed next to the inventory file or the playbook.

  * **Directory Structure:**
    ```
    inventory/
    ├── hosts
    ├── group_vars/
    │   ├── webservers.yml  # Vars for the 'webservers' group
    │   └── databases.yml   # Vars for the 'databases' group
    └── host_vars/
        ├── web1.yml        # Vars for the 'web1' host
        └── db1.yml         # Vars for the 'db1' host
    ```

**Example: `group_vars/webservers.yml`**

```yaml
app_user: www-data
log_dir: /var/log/httpd
```

**Example: `host_vars/web1.yml`**

```yaml
# Overrides the group default for web1
app_user: web1_user
```

#### 2.4. Defining Variables from External Files (`vars_files`)

The `vars_files` directive allows you to load variables from any arbitrary external YAML file into a play.

**Example: `secrets.yml`**

```yaml
db_password: "super_secret_password"
```

**Example: `my_playbook.yml`**

```yaml
---
- name: Use external variables
  hosts: databases
  vars_files:
    - secrets.yml # Loads variables from this file
  tasks:
    - name: Configure database with secret
      ansible.builtin.mysql_user:
        name: app_db_user
        password: "{{ db_password }}" # Using the variable
```


#### 2.6. Using the Command Line (`--extra-vars` or `-e`)

Variables can be passed at runtime using the `--extra-vars` (or `-e`) command-line option. These variables have the **highest precedence** and are commonly used for last-minute overrides or temporary changes.

**Example (Key-Value Pair):**

```bash
ansible-playbook my_playbook.yml -e "environment=production"
```

**Example (JSON/YAML String):**

```bash
ansible-playbook my_playbook.yml -e '{"user":"admin", "port":22}'
```

##  Registering Variables (Dynamic Variables)

The `register` keyword captures the output of a task (e.g., a shell command's stdout, a module's return value) and stores it as a variable for later use. Registered variables are always *host-level* variables.

**Example: `my_playbook.yml`**

```yaml
---
- name: Registering a Variable Example
  hosts: localhost
  tasks:

 - name: Run a shell command and save the output
      ansible.builtin.shell: date +%Y%m%d
      register: today_date_output  # The entire output structure is stored here

    - name: Use the registered variable in a debug message
      ansible.builtin.debug:
        msg: "The deployment date is: {{ today_date_output.stdout }}"
```


###  Variable Precedence

When variables are defined in multiple locations, Ansible determines which value to use based on a strict order of precedence (from *lowest* to *highest*):

| Precedence | Location | Notes |
| :--- | :--- | :--- |
| **Lowest** | Role Defaults (`roles/NAME/defaults/main.yml`) | Easily overridden, good for setting baseline values. |
| | Inventory file/script Group Vars | Variables defined in the inventory for groups. |
| | Inventory Group Vars (`group_vars/all`, `group_vars/GROUP`) | Group-specific variables from files. |
| | Inventory file/script Host Vars | Variables defined in the inventory for specific hosts. |
| | Inventory Host Vars (`host_vars/HOST`) | Host-specific variables from files. |
| | **Facts** (`ansible_facts`) | Information gathered from the target hosts. |
| | Play Variables (`vars:`) | Variables defined directly in the play. |
| | Registered Variables (`register:`) | Variables set dynamically from task output. |
| | Role Parameters (`include_role`/`import_role` params) | Variables passed directly when calling a role. |
| **Highest** | **Extra Vars** (`--extra-vars` or `-e`) | Variables passed on the command line; they override everything else. |

Understanding this precedence is critical for debugging and ensuring your automation uses the correct values in complex environments.
