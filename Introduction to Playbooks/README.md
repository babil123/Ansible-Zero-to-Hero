# Ansible Playbooks

<img src="https://github.com/bhuvan-raj/Ansible-Zero-to-Hero/blob/main/assets/playbook.jpg" alt="Banner" />

## 1\. Playbooks: The Blueprint of Automation

Ansible Playbooks are the cornerstone of automation, serving as a declarative, human-readable description of your system's desired state. They are the core mechanism for **configuration management**, **application deployment**, and **orchestration** across your infrastructure.

### Core Philosophy: Declarative and Idempotent

| Concept | Explanation |
| :--- | :--- |
| **Declarative** | You define **what** the target hosts should look like (e.g., "Nginx must be installed and running") rather than listing the manual, procedural steps of **how** to achieve it. |
| **Idempotency** | Playbooks ensure safety and consistency. If a change is needed to reach the desired state, Ansible executes the task (`changed`). If the host is already compliant, it skips the action (`ok`). This allows playbooks to be run repeatedly without causing disruption. |
| **Orchestration** | A single playbook can manage tasks across multiple tiers (web, application, database) in a specific, coordinated order. |

-----

## 2\. Anatomy of a Playbook (YAML Structure)

A Playbook is written in **YAML** (YAML Ain't Markup Language) and is structured as a list of one or more **Plays**.

### A. The Basic Hierarchy

```yaml
---                            # YAML document start
- name: (Play Name)            # The Play: Targets a group of hosts
  hosts: <target_group_name>   # Which hosts from the inventory to affect
  become: yes                  # Use privilege escalation (e.g., sudo)
  gather_facts: yes            # Auto-collect host system data (facts)

  tasks:                       # List of tasks to execute sequentially
    - name: (Task 1 Name)      # The Task: Calls a specific module
      ansible.builtin.module_name:
        key: value             # Module arguments (defines the desired state)
        key2: value2
```

### B. Key Playbook Keywords

| Keyword | Level | Purpose |
| :--- | :--- | :--- |
| **`hosts`** | Play | Defines the target from the inventory (e.g., `webservers`, `all`, `host1`). |
| **`become`** | Play/Task | If set to `yes`, runs the subsequent tasks with privilege escalation (like `sudo`). |
| **`gather_facts`** | Play | Set to `no` to skip the default information gathering phase (used to speed up simple playbooks). |
| **`tasks`** | Play | The ordered list of actions (each action is a task running a module). |
| **`handlers`** | Play | A list of tasks that only run when explicitly notified by a preceding task. |
| **`vars`** | Play | Defines variables specific to this play (e.g., `version: 1.25`). |
| **`when`** | Task | Applies a conditional statement to control execution (e.g., `when: ansible_os_family == "Debian"`). |
| **`loop`** | Task | Repeats the task for every item in a specified list. |

-----

## 3\. Execution Commands and Options

Playbooks are executed using the dedicated `ansible-playbook` command.

| Command | Purpose |
| :--- | :--- |
| **`ansible-playbook site.yml`** | The standard command to execute the playbook. |
| **`-i /path/to/inventory`** | Specifies a non-default inventory file. |
| **`--limit <host_pattern>`** | Executes the playbook only on a subset of the targeted hosts (e.g., `--limit host01`). |
| **`--tags <tag_name>`** | Executes only those tasks that have been explicitly tagged (for selective runs). |
| **`--skip-tags <tag_name>`** | Skips tasks that match the specified tag. |
| **`-e "key=value"`** or **`--extra-vars`** | Passes variables to the playbook directly from the command line, often overriding variables defined elsewhere. |
| **`-K`** or **`--ask-become-pass`** | Prompts for the `sudo` password needed for privilege escalation (if not using passwordless `sudo`). |


### Basic Execution (The Standard Command)

This is the simplest form and runs the entire playbook against the hosts defined within the playbook and the default inventory file (`/etc/ansible/hosts`).

```bash
ansible-playbook site.yml
```

### Execution with Essential Flags

This command is the most frequently used in real-world scenarios, as it explicitly defines the inventory and uses privilege escalation.

| Command | Purpose |
| :--- | :--- |
| `ansible-playbook site.yml` | Specifies the playbook file to run. |
| `-i inventory.ini` | **(Inventory)** Tells Ansible to use a specific inventory file named `inventory.ini`. |
| `-K` | **(Ask Become Pass)** Prompts the user to enter the `sudo` (become) password for the remote user to run root-level tasks. |
| `-u jenkins_user` | **(User)** Specifies the remote SSH user to connect as (e.g., `jenkins_user`). |

**Example:**

```bash
ansible-playbook site.yml -i inventory.ini -K -u jenkins_user
```

-----


###  Controlling Execution Flow

These flags are used to focus the playbook on a specific area, saving time during development or maintenance.

| Command | Purpose | Example |
| :--- | :--- | :--- |
| **Limit Hosts** | Run against only a subset of the hosts defined in the playbook. | `ansible-playbook site.yml --limit webservers` |
| **List Tasks** | Show the full list of tasks that will be executed (useful for complex playbooks). | `ansible-playbook site.yml --list-tasks` |
| **Start at Task**| Skip all tasks before a specified task name. | `ansible-playbook site.yml --start-at-task "Install Nginx"` |
| **Use Tags** | Run only tasks that have a specific tag defined in the playbook. | `ansible-playbook site.yml --tags "setup, config"` |
| **Pass Variables** | Override playbook variables directly. (Always wins precedence). | `ansible-playbook site.yml -e "web_port=8080"` |




-----

## 4\. Running and Testing Playbooks

Ansible provides crucial safety options to verify code before making actual changes.

| Command/Option | Purpose |
| :--- | :--- |
| **`ansible-playbook --check <file.yml>`** | **Check Mode (Dry Run):** Connects to hosts and reports what **changes it would make** without actually executing them. This is the primary safety check. |
| **`ansible-playbook --diff <file.yml>`** | Shows the "diff" (difference) of files that would be changed, added, or removed. Best used alongside `--check`. |
| **`ansible-playbook --syntax-check <file.yml>`** | Checks the YAML and Ansible syntax for errors. This is the fastest way to catch typos before connecting to any remote host. |
| **`ansible-playbook -v <file.yml>`** | **Verbose Mode:** Increases the output detail. Use `-vvv` to show debugging information, which is essential when troubleshooting connection issues or task failures. |
| **`ansible-playbook --start-at-task "Task Name"`** | Starts the playbook execution from a specific named task, skipping all previous plays and tasks. |
