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

[database]

db1
db2
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


## Ansible Facts (The State of the Managed Node)

Ansible Facts are pieces of information automatically gathered about your managed hosts by the `setup` module. They provide a rich dictionary of the system's current state, allowing your automation to be intelligent and context-aware.

####  What are Facts?

  * **Definition:** Facts are system-specific variables collected by Ansible's `setup` module. They contain detailed information about the remote host.
  * **Examples of Data:** OS type, distribution, version, kernel version, network interfaces, IP addresses, CPU architecture, memory, disk space, and much more.
  * **Storage:** All gathered facts are stored in a special variable dictionary called `ansible_facts`.
  * **Function:** Facts are essential for making conditional decisions (e.g., "If OS is RedHat, use `yum`; if Debian, use `apt`") and for generating dynamic configuration files (Templating).

####  How Facts are Gathered

  * **Automatic Collection:** By default, every Ansible Playbook execution automatically runs the `setup` module at the beginning of each play (unless explicitly disabled).
  * **Module:** The module responsible for gathering facts is `ansible.builtin.setup`.
  * **Ad-Hoc Commands:** You can retrieve all facts for a host using an ad-hoc command:
    ```bash
    ansible webservers -m setup
    ```
  * **Filtering Facts (Performance):** Gathering all facts can be slow on a large number of hosts. You can filter the facts collected to improve performance:
      * **In Playbook:** Use the `gather_subset` keyword to gather only specific categories:
        ```yaml
        - hosts: all
          gather_facts: yes
          gather_subset:
            - network
            - os
            - distribution
          # ... rest of the play
        ```
  * **Disabling Facts:** If your playbook does not require any system-specific information, you can disable fact gathering entirely using the `gather_facts: no` keyword at the play level to significantly speed up execution time.

####  Common and Essential Facts

Facts are typically referenced using dot notation or bracket notation within the `ansible_facts` dictionary.

| Fact Variable | Description | Example Value | Usage Example |
| :--- | :--- | :--- | :--- |
| `ansible_facts['os_family']` | High-level OS family. | `Debian`, `RedHat`, `Darwin` | Conditional package management. |
| `ansible_facts['distribution']` | Specific OS distribution. | `Ubuntu`, `CentOS`, `Fedora` | Conditional package management. |
| `ansible_facts['default_ipv4']['address']` | The host's primary IPv4 address. | `192.168.1.100` | Dynamic template configuration. |
| `ansible_facts['hostname']` | The short hostname of the system. | `web01` | Naming configuration files. |
| `ansible_facts['memtotal_mb']` | Total physical memory in MB. | `4096` | Checking resource thresholds. |
| `ansible_facts['processor_count']` | Number of logical processors/cores. | `4` | Setting application thread limits. |


## Conditional Execution (`when` Keyword)

Conditional statements in Ansible allow you to control which tasks are executed on which hosts based on specific criteria. The primary mechanism for this is the `when` keyword.

#### 1\. The `when` Keyword

  * **Definition:** The `when` keyword takes a **Jinja2 expression** that must evaluate to **True** for the task to run. If the expression evaluates to **False**, the task is skipped.
  * **Syntax:** The expression is generally written without the double curly braces (`{{ }}`) that normally enclose variables in Jinja2, as the entire line is treated as a Jinja2 template.

#### 2\. Conditional Logic with Facts

The most common use of `when` is to conditionally execute tasks based on gathered facts:

| Condition Type | Playbook Example | Explanation |
| :--- | :--- | :--- |
| **Simple Equality** | `when: ansible_facts['os_family'] == "RedHat"` | Runs only on hosts belonging to the RedHat OS family (e.g., CentOS, RHEL). |
| **Logical OR** | `when: ansible_facts['distribution'] == "Ubuntu" or ansible_facts['distribution'] == "Debian"` | Runs on hosts running either Ubuntu **or** Debian. |
| **Logical AND** | `when: ansible_facts['os_family'] == "RedHat" and ansible_facts['distribution_major_version'] == "7"` | Runs only on RHEL/CentOS version 7 systems. |
| **Logical NOT (Exclusion)** | `when: ansible_facts['hostname'] != "db-master"` | Runs on all hosts **except** the one named `db-master`. |
| **List Format (Implicit AND)**| ` when:  `- ansible\_facts['os\_family'] == "RedHat"`<br>`- ansible\_facts['distribution\_major\_version'] == "7"\` | Alternative syntax for Logical AND. |

#### 3\. Conditional Logic with Variables

The `when` statement can also check for standard playbook variables or variables passed via the command line (`-e`).

```yaml
- name: Deploy a backup job
  ansible.builtin.shell: /usr/local/bin/backup.sh
  when: env_type == "production"

# Command line execution example:
# ansible-playbook site.yml -e "env_type=production"
```

#### 4\. Conditional Logic with Registered Variables

Tasks can capture their output using the `register` keyword. This output (a dictionary containing status, stdout, stderr, etc.) can then be used in subsequent `when` conditions.

| Condition Type | Playbook Example | Explanation |
| :--- | :--- | :--- |
| **Check for Failure** | `when: result is failed` | Runs the task only if the previously registered task (`result`) failed. |
| **Check for Change** | `when: package_install is changed` | Runs if the previous task (`package_install`) actually modified the host's state. |
| **Check String Content**| `when: motd_contents.stdout.find('Warning') != -1` | Runs if the string output of the previous task contains the word 'Warning'. |
| **Check File Existence**| `when: file_check.stat.exists` | Runs if the `stat` module check (`file_check`) confirmed the file exists. |

#### **Example: Combining Facts and Conditionals**

This example dynamically selects the correct package manager based on the OS family:

```yaml
- name: Conditional Package Management based on OS Family
  hosts: all
  become: yes
  tasks:
    # --- STEP 1: Use a FACT to set a Variable dynamically ---
    - name: Set OS-specific package and service names
      ansible.builtin.set_fact:
        # If OS Family is RedHat, package is 'httpd', otherwise (for Debian) it's 'apache2'
        web_package_name: "{{ 'httpd' if ansible_facts['os_family'] == 'RedHat' else 'apache2' }}"
        # Service name is the same as the package name we just determined
        web_service_name: "{{ 'httpd' if ansible_facts['os_family'] == 'RedHat' else 'apache2' }}"

    # --- STEP 2: Use the Variable for the installation task ---
    - name: Install web server package using the appropriate package manager
      ansible.builtin.package:
        name: "{{ web_package_name }}" # Uses the dynamically set variable
        state: present

    # --- STEP 3: Use a FACT in a Conditional (when) to start the service ---
    - name: Enable and Start service on RedHat family hosts
      ansible.builtin.service:
        name: "{{ web_service_name }}" # Uses the dynamically set service variable
        state: started
        enabled: yes
      when: ansible_facts['os_family'] == "RedHat"
      
    - name: Enable and Start service on Debian family hosts
      ansible.builtin.service:
        name: "{{ web_service_name }}" # Uses the dynamically set service variable
        state: started
        enabled: yes
      when: ansible_facts['os_family'] == "Debian"
```
