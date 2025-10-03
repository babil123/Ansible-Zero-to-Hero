# Ansible Ad-Hoc Commands

<img src="https://github.com/bhuvan-raj/Ansible-Zero-to-Hero/blob/main/assets/adhoc.png" alt="Banner" />

### **1. What are Ad-Hoc Commands?**

  * **Definition:** Ad-hoc commands are single-line commands executed from the Ansible **Control Node** to perform a **single, quick task** on one or more **Managed Nodes**.
  * **Contrast with Playbooks:**
      * **Ad-Hoc:** Used for one-time, simple tasks (e.g., checking uptime, restarting a service, quick file copy). They are **not reusable** or intended for complex configuration management.
      * **Playbooks:** Used for complex, multi-step, repeatable, and reusable automation (configuration, deployment, orchestration).
  * **Tool:** Ad-hoc commands use the `/usr/bin/ansible` command-line tool.

-----

### **2. The Basic Ad-Hoc Command Syntax**

The general structure of an ad-hoc command is:

```bash
ansible <pattern> -m <module> -a "<module_options>" [optional_flags]
```

| Component | Description | Example Value |
| :--- | :--- | :--- |
| **`ansible`** | The command-line tool. | `ansible` |
| **`<pattern>`** | The target host(s) or group(s) from your **Inventory** file. | `all`, `webservers`, `host1` |
| **`-m`** | Specifies the **Ansible Module** to use. | `-m ping`, `-m file`, `-m yum` |
| **`-a`** | Specifies the **Arguments** (options/parameters) for the module. | `-a "src=/local/file dest=/remote/file"` |
| **`[optional_flags]`** | Additional command-line options. | `-b` (become/sudo), `-K` (ask become password) |

-----

### **3. Essential Optional Flags**

| Flag | Full Name | Purpose |
| :--- | :--- | :--- |
| **`-b`** | `--become` | Run the task with **privilege escalation** (like `sudo`). |
| **`-K`** | `--ask-become-pass` | Prompt for the **become password** (if needed for `sudo`). |
| **`-u`** | `--user` | Specify the **remote user** to connect as (e.g., `-u ec2-user`). |
| **`-i`** | `--inventory` | Specify an alternate **inventory file**. |
| **`-f`** | `--forks` | Specify the number of **parallel processes** (default is 5). |

-----

### **4. Key Ad-Hoc Examples**

| Task | Module | Command Example | Explanation |
| :--- | :--- | :--- | :--- |
| **Check Connectivity** | **`ping`** | `ansible all -m ping` | Tests if Ansible can connect to and successfully execute a task on all hosts. |
| **Run Simple Command** | **`command`** (Default) | `ansible webservers -a "uptime"` | Runs the `uptime` command on all hosts in the `webservers` group. |
| **Run Shell Command** | **`shell`** | `ansible db -m shell -a 'echo $TERM \| wc -l'` | Runs a command that requires **shell features** (like pipes `|` or variables `$`). |
| **Copy a File** | **`copy`** | `ansible all -m copy -a "src=/etc/hosts dest=/tmp/hosts"` | Copies a file from the control node to the managed nodes. |
| **Manage Files/Dirs** | **`file`** | `ansible all -m file -a "path=/tmp/testdir state=directory mode=0755"` | Creates a directory with specific permissions. |
| **Manage Packages** | **`yum`** (or `apt`, `dnf`) | `ansible all -b -m yum -a "name=nginx state=present"` | Installs the latest version of Nginx (requires `become`). |
| **Manage Services** | **`service`** | `ansible webservers -b -m service -a "name=httpd state=restarted"` | Restarts the Apache service on all webservers (requires `become`). |
| **Gather Facts** | **`setup`** | `ansible all -m setup` | Gathers extensive system information (facts) about the managed nodes. |

### **Ad-Hoc Command to Install Nginx**

Since installing a package requires elevated permissions (root/sudo), you must use the `--become` or `-b` flag.

#### **If your managed nodes use `apt` (e.g., Debian, Ubuntu):**

```bash
ansible all -b -m apt -a "name=nginx state=present"
```

| Component | Description |
| :--- | :--- |
| **`ansible all`** | Targets all hosts defined in your inventory. |
| **`-b`** | **`--become`**: Executes the task with root privileges (required for installation). |
| **`-m apt`** | Specifies the **`apt`** module for Debian/Ubuntu systems. |
| **`-a "..."`** | Arguments for the `apt` module. |
| **`name=nginx`** | The package to install. |
| **`state=present`** | Ensures the package is installed (if it's missing, it will install it). |

-----

#### **If your managed nodes use `yum` or `dnf` (e.g., Red Hat, CentOS, Fedora):**

```bash
ansible all -b -m yum -a "name=nginx state=present"
```

| Component | Description |
| :--- | :--- |
| **`ansible all`** | Targets all hosts defined in your inventory. |
| **`-b`** | **`--become`**: Executes the task with root privileges. |
| **`-m yum`** | Specifies the **`yum`** module (or use `-m dnf` for modern Fedora/CentOS/RHEL systems). |
| **`-a "..."`** | Arguments for the module. |
| **`name=nginx`** | The package to install. |
| **`state=present`** | Ensures the package is installed. |

### **To Run and Verify (The Next Step)**

1.  **Run the Installation:**
    ```bash
    ansible webservers -b -m apt -a "name=nginx state=present"
    ```
2.  **Verify the Service is Running (Optional Next Ad-Hoc Command):**
    ```bash
    ansible webservers -b -m service -a "name=nginx state=started"
    ```
    *This command ensures that after installation, the service is also running.*
