## 1. Introduction to Configuration Management
<img src="https://github.com/bhuvan-raj/Ansible-Zero-to-Hero/blob/main/assets/conf.webp" alt="Banner" />


### 🔹 What is Configuration Management (CM)?

* Configuration Management is the **process of automating the setup, management, and maintenance of IT systems** (servers, networks, applications, and services).
* Instead of manually installing software, editing config files, and applying patches, CM tools **automate these tasks**.
* Key aspects:

  * Ensures **consistency** across environments (Dev → Test → Prod).
  * Reduces **manual errors**.
  * Helps in **scalability** (managing 100s of servers easily).
  * Supports **auditing and compliance**.

### 🔹 Example Without CM:

* Imagine setting up 10 servers.
* You manually install Apache, configure `httpd.conf`, set firewall rules.
* Repeating it for all 10 servers → **time-consuming + error-prone**.

### 🔹 Example With CM:

* You write a playbook (in Ansible) or a recipe (in Chef).
* Run it on all servers → **automated, consistent, repeatable**.

---

## 2. Configuration Management vs Infrastructure as Code (IaC)

| Feature      | Configuration Management (CM)                                                      | Infrastructure as Code (IaC)                                       |
| ------------ | ---------------------------------------------------------------------------------- | ------------------------------------------------------------------ |
| Purpose      | Manages and configures existing infrastructure (software setup, patching, updates) | Provisions and creates infrastructure (servers, networks, storage) |
| Tools        | Ansible, Puppet, Chef, SaltStack                                                   | Terraform, CloudFormation, Pulumi                                  |
| Example Task | Install Apache, configure Nginx, create users, manage cronjobs                     | Create EC2 instances, VPC, load balancers                          |
| Analogy      | **CM = Managing house interiors (furniture, paint, AC settings)**                  | **IaC = Building the house (walls, rooms, roof)**                  |

👉 In practice: **IaC (Terraform) provisions infra → CM (Ansible) configures apps/services on it.**

---

## 3. Difference Between Ansible and Terraform

| Aspect           | Ansible (CM tool)                               | Terraform (IaC tool)                  |
| ---------------- | ----------------------------------------------- | ------------------------------------- |
| Category         | Configuration Management                        | Infrastructure Provisioning           |
| Language         | YAML (Playbooks)                                | HCL (HashiCorp Config Language)       |
| Execution Model  | Push (agentless, via SSH)                       | Declarative, via provider APIs        |
| State Management | Doesn’t maintain state (executes each time)     | Maintains state (`terraform.tfstate`) |
| Use Case         | App deployment, software installation, patching | Create servers, networks, cloud infra |
| Learning Curve   | Easier (simple YAML)                            | Moderate (HCL, providers, states)     |

👉 **They complement each other**: Use Terraform to build infra, then Ansible to configure and deploy apps.

---

## 4. Popular Configuration Management Tools

* **Puppet**

  * Written in Ruby.
  * Declarative model.
  * Has its own DSL (domain-specific language).
* **Chef**

  * Ruby-based.
  * Uses recipes and cookbooks.
* **SaltStack**

  * Python-based.
  * Uses master/minion model.
* **Ansible**

  * Python-based.
  * **Agentless**, uses SSH.
  * Playbooks written in YAML.

### 🔹 Why Ansible is Popular?

1. **Agentless** → No need to install agents (uses SSH/WinRM).
2. **Simple YAML syntax** → Easy to learn & human-readable.
3. **Idempotent** → Running the same playbook multiple times won’t break things.
4. **Cross-platform** → Works on Linux, Windows, network devices.
5. **Large community & modules** → Thousands of built-in modules.
6. **Push model** → You control when and where configurations are applied.

---

# 🛠️ Ansible From Scratch

---

## 1. What is Ansible?

* Ansible is an **open-source configuration management and automation tool**.
* Written in **Python**.
* Uses **YAML playbooks** to define tasks.
* Works on **agentless architecture** → connects via **SSH (Linux)** or **WinRM (Windows)**.

---

## 2. Ansible Architecture

**Components:**

1. **Control Node**

   * Where Ansible is installed.
   * Executes playbooks and pushes configurations.

2. **Managed Nodes**

   * Target systems (servers, VMs, containers).
   * No need to install anything.

3. **Inventory**

   * A file that lists target systems (e.g., `hosts.ini`).

4. **Modules**

   * Predefined units of work (e.g., install package, copy file).

5. **Playbooks**

   * YAML files containing automation instructions.

👉 Flow: Control Node → Inventory → SSH/WinRM → Managed Nodes.

---

## 3. Ansible Installation

### On Control Node (Linux)

```bash
sudo apt update
sudo apt install ansible -y
ansible --version
```

---

## 4. Inventory File

Example `hosts.ini`:

```ini
[webservers]
192.168.1.10
192.168.1.11

[dbservers]
192.168.1.20
```

---

## 5. Ad-Hoc Commands

Run quick commands without playbooks:

```bash
ansible all -i hosts.ini -m ping
ansible webservers -i hosts.ini -m shell -a "uptime"
ansible dbservers -i hosts.ini -m yum -a "name=httpd state=present"
```

---

## 6. Ansible Playbooks

### Example Playbook: Install Apache

```yaml
---
- name: Install Apache on web servers
  hosts: webservers
  become: yes
  tasks:
    - name: Install Apache
      apt:
        name: apache2
        state: present

    - name: Start Apache
      service:
        name: apache2
        state: started
        enabled: true
```

Run:

```bash
ansible-playbook -i hosts.ini install-apache.yml
```

---

## 7. Key Concepts in Ansible

* **Tasks** → Single action (install package, copy file).
* **Handlers** → Run only when notified (e.g., restart service after config change).
* **Variables** → Store values (e.g., app version).
* **Templates** → Dynamic config files using Jinja2 (`.j2`).
* **Roles** → Structured way to organize playbooks.
* **Galaxy** → Public repo of community roles.

---

## 8. Ansible Use Cases

1. **Configuration Management** → Install software, configure services.
2. **Application Deployment** → Deploy Java, Python, Node.js apps.
3. **Provisioning (basic)** → Create users, manage cloud instances.
4. **Orchestration** → Multi-tier deployments (web + app + db).
5. **Security & Compliance** → Apply patches, enforce policies.

---
