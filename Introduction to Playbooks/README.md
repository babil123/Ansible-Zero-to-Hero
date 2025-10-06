# Ansible Playbooks

<img src="https://github.com/bhuvan-raj/Ansible-Zero-to-Hero/blob/main/assets/playbook.jpg" alt="Banner" />


## $\text{1. Ansible Galaxy In-depth and Commands}$

**Ansible Galaxy** is the official hub for community-contributed Ansible content, primarily focusing on **Roles** and **Collections** to promote modularity and reusability. It is essential for jumpstarting automation projects.

### $\text{What It Is}$

| Concept | Explanation |
| :--- | :--- |
| **Role** | A standardized, portable directory structure that logically groups related tasks, handlers, variables, files, and templates for a single function (e.g., configuring Nginx, setting up PostgreSQL). |
| **Collection** | The newer format; a complete package of automation content that can include multiple roles, playbooks, modules, and plugins. Collections are versioned and signed. |
| **Repository** | Ansible Galaxy acts as a central repository (like PyPI for Python or Maven for Java). |

### $\text{Ansible-Galaxy Commands}$

The `ansible-galaxy` command-line tool is used to manage this content:

| Command | Purpose | Example |
| :--- | :--- | :--- |
| `ansible-galaxy init` | Creates the standard skeleton directory structure for a new Role. | `ansible-galaxy init my_new_role` |
| `ansible-galaxy install` | Downloads a Role or Collection from the Galaxy repository. | `ansible-galaxy install geerlingguy.nginx` |
| `ansible-galaxy list` | Shows all installed Roles and Collections on your local control node. | `ansible-galaxy list` |
| `ansible-galaxy remove` | Uninstalls a specific Role or Collection. | `ansible-galaxy remove geerlingguy.nginx` |

-----

## $\text{2. Core Philosophy}$

The design of Ansible Playbooks is rooted in three key principles:

| Concept | Description |
| :--- | :--- |
| **Declarative** | You define the **final desired state** of the system ("Nginx must be running") rather than specifying the step-by-step procedure ("1. Run apt update, 2. Run apt install nginx, 3. Run service nginx start"). |
| **Idempotency** | Playbooks can be run repeatedly without causing unintended side effects or disruption. Ansible modules check the current state: if the desired state is met, the task reports **`ok`** (no change); if a change is needed, it reports **`changed`**. |
| **Orchestration** | A single playbook can define coordinated automation across many different types of hosts (e.g., run tasks on Web Servers, then Database Servers, then Load Balancers, all in one sequence). |

-----

## $\text{3. Hierarchy of a Playbook (YAML Structure)}$

A playbook is a list of **Plays**, which contain **Tasks**. This order is executed sequentially.

| Hierarchy Level | YAML Marker/Keyword | Purpose |
| :--- | :--- | :--- |
| **Playbook** | `.yml` file | The entire automation file. |
| **Play** | `- name: ...`, `hosts: ...` | Targets a specific group of hosts from the Inventory (e.g., `webservers`) and defines global settings like privilege escalation. |
| **Task** | `- name: ...` | A single action on the remote host. A task executes exactly one **Module**. The order of tasks is critical. |
| **Module** | `ansible.builtin.module_name: ...` | The actual tool (code) that performs a function (e.g., installing a package, copying a file, starting a service). |

```yaml
---
# ----------------------------------------
# PLAY: Targets all machines in the 'webservers' group
# ----------------------------------------
- name: 1. Ensure essential packages are present
  hosts: webservers
  become: yes          # Run all subsequent tasks with elevated privileges

  tasks:
    # ----------------------------------------
    # TASK 1: Installs a package
    # ----------------------------------------
    - name: Install Git
      ansible.builtin.package:
        name: git
        state: present

    # ----------------------------------------
    # TASK 2: Creates a file
    # ----------------------------------------
    - name: Create configuration file
      ansible.builtin.file:
        path: /etc/myconfig.conf
        state: touch
```

-----

## $\text{4. What Are Ansible Modules}$

Ansible Modules are the **tools** or **functional units** that perform the specific work on remote systems.

  * **Execution:** Modules are executed by the control node (where you run the playbook) and briefly push their code to the remote host, execute, and then retrieve the result before being removed.
  * **Idempotency:** Most modules are designed to be idempotent (they check if a change is needed before performing it).
  * **Module Naming:** Modules are categorized (e.g., `ansible.builtin.package`, `ansible.builtin.service`).

| Module | Purpose | Example |
| :--- | :--- | :--- |
| `package` | Installs, removes, or manages software packages (`apt`, `yum`, `pacman`). | `name: nginx, state: latest` |
| `service` | Starts, stops, restarts, or enables services (like systemd units). | `name: apache2, state: started` |
| `file` | Creates, deletes, or changes permissions/ownership of files and directories. | `path: /tmp/data, state: directory` |
| `copy` | Copies files from the local control node to the remote host. | `src: local.conf, dest: /etc/remote.conf` |
| `template` | Creates configuration files dynamically using **Jinja2** variables. | `src: config.j2, dest: /etc/config.ini` |

-----

## $\text{5. Commands for Ansible Playbook Execution}$

The primary execution tool is `ansible-playbook`.

| Command | Purpose |
| :--- | :--- |
| **Standard Run** | `ansible-playbook site.yml` |
| **Dry Run (Safety Check)** | `ansible-playbook site.yml --check --diff` |
| **Specify Inventory** | `ansible-playbook site.yml -i inventory.ini` |
| **Supply SUDO Password** | `ansible-playbook site.yml -K` (Prompts for become password) |
| **Specify User** | `ansible-playbook site.yml -u remote_user` |
| **Limit to Specific Hosts** | `ansible-playbook site.yml --limit host01,host02` |
| **Use Tags (Selective Run)** | `ansible-playbook site.yml --tags "apache_setup"` |
| **Skip Tasks** | `ansible-playbook site.yml --skip-tags "debug"` |
| **Override Variables** | `ansible-playbook site.yml -e "port=8080"` (Highest precedence) |
| **Start Execution Later** | `ansible-playbook site.yml --start-at-task "Start Web Service"` |

-----

## $\text{6. Example Playbook: Installing Nginx and Apache}$

This playbook demonstrates installing both Apache (`httpd` for RHEL/CentOS systems) and Nginx, showing how to use the `when` conditional and `handlers`.

**File: `web_servers.yml`**

```yaml
---
- name: 1. Install and Start Apache
  hosts: webservers
  become: yes
  vars:
    # Define package names based on OS family
    apache_package: "{{ 'apache2' if ansible_os_family == 'Debian' else 'httpd' }}"
    nginx_package: "nginx"

  tasks:
    - name: Ensure Apache package is installed
      ansible.builtin.package:
        name: "{{ apache_package }}"
        state: present

    - name: Copy the index.html test page
      ansible.builtin.copy:
        content: "<h1>Server {{ inventory_hostname }} is running Apache!</h1>"
        dest: /var/www/html/index.html
        mode: '0644'
      notify: Restart Apache Handler # If this task *changes* the file, notify the handler

    - name: Ensure Apache service is running and enabled
      ansible.builtin.service:
        name: "{{ apache_package }}"
        state: started
        enabled: yes
      tags: apache_start

  handlers:
    - name: Restart Apache Handler
      ansible.builtin.service:
        name: "{{ apache_package }}"
        state: restarted

- name: 2. Install and Start Nginx (Conditional Play)
  hosts: webservers
  become: yes
  # Only run this play on Debian-based systems to avoid conflicts
  when: ansible_os_family == "Debian"

  tasks:
    - name: Ensure Nginx package is installed
      ansible.builtin.package:
        name: nginx
        state: present

    - name: Ensure Nginx service is running and enabled
      ansible.builtin.service:
        name: nginx
        state: started
        enabled: true
      tags: nginx_start
```
