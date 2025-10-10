# Ansible Galaxy**
<img src="https://github.com/bhuvan-raj/Ansible-Zero-to-Hero/blob/main/assets/galaxy.png" alt="Banner" />

---

## 🌍 **1. What is Ansible Galaxy?**

**Ansible Galaxy** is the **official hub for sharing and discovering Ansible content** — primarily **roles** and **collections**.

Think of it like:

> **“GitHub for Ansible automation content.”**

It allows DevOps engineers to:

* Find **pre-built roles** (e.g., install Nginx, configure Docker, manage AWS, etc.)
* **Reuse** community-developed automation
* **Publish** their own roles or collections for others

🔗 Website: [https://galaxy.ansible.com](https://galaxy.ansible.com)

---

## ⚙️ **2. Why Use Ansible Galaxy?**

| Feature                | Description                                                     |
| ---------------------- | --------------------------------------------------------------- |
| 🔁 Reusability         | Download ready-made roles instead of writing from scratch       |
| 🌐 Collaboration       | Share your automation publicly or within a team                 |
| 🧱 Structure           | Provides standardized role templates                            |
| 🛠️ CLI Integration    | Comes with the `ansible-galaxy` command-line tool               |
| 🧩 Collections Support | Supports both **roles** and **collections** for larger projects |

---

## 🧰 **3. Ansible Galaxy CLI Overview**

Ansible provides the **`ansible-galaxy`** command to interact with Galaxy.

You can use it to:

* Create roles
* Download roles
* Remove roles
* List installed roles
* Manage collections

### 🔹 Common Syntax:

```bash
ansible-galaxy [subcommand] [options]
```

---

## 🧭 **4. Important Ansible Galaxy Commands**

| Command                                          | Description                                     | Example                                        |
| ------------------------------------------------ | ----------------------------------------------- | ---------------------------------------------- |
| `ansible-galaxy init <role_name>`                | Create a new role structure                     | `ansible-galaxy init nginx_setup`              |
| `ansible-galaxy install <role_name>`             | Install a role from Galaxy                      | `ansible-galaxy install geerlingguy.nginx`     |
| `ansible-galaxy install -r requirements.yml`     | Install multiple roles from a requirements file | `ansible-galaxy install -r requirements.yml`   |
| `ansible-galaxy list`                            | List installed roles                            | `ansible-galaxy list`                          |
| `ansible-galaxy remove <role_name>`              | Remove an installed role                        | `ansible-galaxy remove geerlingguy.nginx`      |
| `ansible-galaxy search <keyword>`                | Search roles on Galaxy                          | `ansible-galaxy search nginx`                  |
| `ansible-galaxy collection install <collection>` | Install a collection                            | `ansible-galaxy collection install amazon.aws` |
| `ansible-galaxy collection list`                 | List installed collections                      | `ansible-galaxy collection list`               |

---

## 🧱 **5. Structure of an Ansible Role**

When you create a role using `ansible-galaxy init`, it generates a standard directory layout:

```
my_role/
├── defaults/
│   └── main.yml
├── files/
├── handlers/
│   └── main.yml
├── meta/
│   └── main.yml
├── tasks/
│   └── main.yml
├── templates/
├── tests/
├── vars/
│   └── main.yml
└── README.md
```

### 🔹 Important File: `meta/main.yml`

Contains role metadata required for publishing:

```yaml
galaxy_info:
  author: bubu
  description: Install and configure Nginx
  company: ""
  license: MIT
  min_ansible_version: 2.9
  platforms:
    - name: Ubuntu
      versions:
        - focal
        - jammy
  galaxy_tags:
    - nginx
    - web
dependencies: []
```

---

## 📦 **6. How to Download (Install) a Role from Ansible Galaxy**

### ✅ **Step-by-Step**

1. **Search for a role:**

   ```bash
   ansible-galaxy search nginx
   ```

2. **Choose a role** (example: `geerlingguy.nginx`)

3. **Install the role:**

   ```bash
   ansible-galaxy install geerlingguy.nginx
   ```

4. **Verify installation:**

   ```bash
   ansible-galaxy list
   ```

   Output example:

   ```
   - geerlingguy.nginx, 3.2.0 (installed in /home/user/.ansible/roles)
   ```

5. **Use in a playbook:**

   ```yaml
   - hosts: webservers
     roles:
       - geerlingguy.nginx
   ```

---

## 🧩 **7. How to Push (Publish) a Role to Ansible Galaxy**

Let’s go from **local → GitHub → Galaxy** step-by-step 👇

---

### 🪜 **Step 1: Create a Galaxy Account**

1. Go to [https://galaxy.ansible.com](https://galaxy.ansible.com)
2. Click **Sign In**
3. Choose **“Login with GitHub”**
4. Authorize the Ansible Galaxy app
5. You now have a **Galaxy namespace** (usually same as your GitHub username)

---

### 🧱 **Step 2: Create a Role Locally**

```bash
ansible-galaxy init nginx_setup
```

Fill in:

* `tasks/main.yml` → actual automation tasks
* `meta/main.yml` → metadata (author, description, etc.)
* `README.md` → role usage documentation

Example task:

```yaml
# tasks/main.yml
- name: Install Nginx
  apt:
    name: nginx
    state: present
```

---

### 💾 **Step 3: Push Role to GitHub**

1. Initialize git inside your role folder:

   ```bash
   cd nginx_setup
   git init
   git add .
   git commit -m "My first Ansible role"
   ```

2. Create a **public repo** on GitHub (name format must be):

   ```
   <github_username>.<role_name>
   ```

   Example: `bubu.nginx_setup`

3. Add remote and push:

   ```bash
   git remote add origin https://github.com/bubu/nginx_setup.git
   git branch -M main
   git push -u origin main
   ```

✅ Your role is now live on GitHub.

---

### 🌐 **Step 4: Import Role into Ansible Galaxy**

Now go to Galaxy:

1. Navigate to 👉 **Roles → Role Namespaces**
2. Click your namespace (e.g., `bubu`)
3. On the top-right, click the **three dots (⋮)** → **Import Role**
4. Fill in:

   * Repository Name → `bubu.nginx_setup`
   * Repository URL → `https://github.com/bubu/nginx_setup`
   * Branch → `main` (default)
5. Click **Import**

---

### 🔄 **Step 5: Verify Import**

Once imported successfully:

* Galaxy will show logs of the import
* You’ll get a success message ✅

Your role page will be available at:

```
https://galaxy.ansible.com/bubu/nginx_setup
```

---

### 💡 **Step 6: Test Installation from Galaxy**

From any system with Ansible:

```bash
ansible-galaxy install bubu.nginx_setup
```

Use it in a playbook:

```yaml
- hosts: web
  roles:
    - bubu.nginx_setup
```

---

## 🧠 **8. Common Issues and Fixes**

| Problem                  | Cause                              | Solution                                             |
| ------------------------ | ---------------------------------- | ---------------------------------------------------- |
| Import fails             | Invalid `meta/main.yml`            | Validate YAML syntax                                 |
| Repo not found           | Repo is private or naming mismatch | Make it **public** and follow `<user>.<role>` naming |
| “My Content” not visible | Using new UI                       | Go via `Roles → Role Namespaces → ⋮ → Import Role`   |
| Role missing metadata    | Incomplete `galaxy_info`           | Fill all required fields                             |

---

## 🧩 **9. Example `requirements.yml` (for Multiple Roles)**

You can install multiple roles together using a file:

```yaml
# requirements.yml
- src: geerlingguy.nginx
  version: 3.2.0
- src: bubu.nginx_setup
  version: 1.0.0
```

Then run:

```bash
ansible-galaxy install -r requirements.yml
```

---

## 🧠 **10. Summary**

| Concept                  | Description                                                         |
| ------------------------ | ------------------------------------------------------------------- |
| **Website**              | [https://galaxy.ansible.com](https://galaxy.ansible.com)            |
| **Purpose**              | To find, share, and reuse Ansible roles and collections             |
| **CLI Tool**             | `ansible-galaxy`                                                    |
| **Key Command Examples** | `init`, `install`, `list`, `remove`, `search`, `collection install` |
| **Publishing Steps**     | Create → Push to GitHub → Import via Galaxy                         |
| **Namespace**            | Created automatically after logging in with GitHub                  |
| **Role Format**          | `<github_username>.<role_name>`                                     |

---
