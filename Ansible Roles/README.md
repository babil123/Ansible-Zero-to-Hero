# 📘 Ansible Roles
<img src="https://github.com/bhuvan-raj/Ansible-Zero-to-Hero/blob/main/assets/roles.webp" alt="Banner" />


## 1. What is an Ansible Role?

* A **Role** in Ansible is a way to **organize playbooks** into a structured, reusable, and maintainable format.
* Instead of writing long playbooks with many tasks, variables, handlers, templates, etc. all in one file, Roles break them into **self-contained directories**.
* Each Role performs a **specific function** (like installing Nginx, configuring users, setting up databases) and can be easily shared and reused across projects.

👉 Think of it as a **modular building block** for automation.

---

## 2. Why Use Roles?

* **Organization:** Cleaner code, easier to manage large playbooks.
* **Reusability:** Write once, use multiple times across projects.
* **Collaboration:** Standard directory structure makes it easier for teams to work together.
* **Best Practices:** Encouraged by Ansible Galaxy and community standards.
* **Scalability:** Easier to scale automation as infrastructure grows.

---

## 3. Role Directory Structure

When you create a role (usually with `ansible-galaxy init <role_name>`), it generates the following directory structure:

```
roles/
└── myrole/
    ├── defaults/      # Default variables (lowest priority)
    │   └── main.yml
    ├── vars/          # Variables with higher priority
    │   └── main.yml
    ├── tasks/         # List of tasks executed (main entry point)
    │   └── main.yml
    ├── handlers/      # Handlers (used with notify)
    │   └── main.yml
    ├── templates/     # Jinja2 templates (.j2 files)
    ├── files/         # Static files (copied as-is)
    ├── meta/          # Role metadata (dependencies, author info)
    │   └── main.yml
    ├── tests/         # Test playbooks and inventory
    │   ├── test.yml
    │   └── inventory
    └── README.md      # Documentation for the role
```

### Key components:

* **tasks/main.yml** → entry point, defines what the role does.
* **handlers/main.yml** → for restart/reload services.
* **defaults/main.yml** → default variables (lowest priority).
* **vars/main.yml** → role-specific variables (higher priority than defaults).
* **files/** → contains files to copy to hosts.
* **templates/** → dynamic config files using Jinja2.
* **meta/main.yml** → dependencies, role author, supported platforms.

---

## 4. Using a Role in a Playbook

Example `site.yml`:

```yaml
- hosts: webservers
  roles:
    - nginx
    - { role: mysql, db_name: myapp, db_user: admin }
```

* Simply include the role name under `roles:`.
* You can also pass variables inline.

---

## 5. Creating a Role (Step by Step)

### Step 1: Create Role Skeleton

```bash
ansible-galaxy init nginx
```

### Step 2: Define Tasks

Edit `roles/nginx/tasks/main.yml`:

```yaml
- name: Install Nginx
  apt:
    name: nginx
    state: present
    update_cache: yes

- name: Start Nginx
  service:
    name: nginx
    state: started
    enabled: yes
```

### Step 3: Add Handlers

Edit `roles/nginx/handlers/main.yml`:

```yaml
- name: restart nginx
  service:
    name: nginx
    state: restarted
```

### Step 4: Add Templates (if needed)

`roles/nginx/templates/nginx.conf.j2`

```nginx
server {
    listen 80;
    server_name {{ ansible_hostname }};
    root /var/www/html;
}
```

---

## 6. Role Dependencies

* Define in `meta/main.yml`.
* Example: If your **wordpress** role depends on **mysql** and **php** roles:

```yaml
dependencies:
  - role: mysql
  - role: php
```

When you run `wordpress`, it will automatically run `mysql` and `php` first.

---

## 7. Variables in Roles

* **defaults/main.yml** → variables with **lowest precedence** (can be overridden easily).
* **vars/main.yml** → variables with **higher precedence**.
* Example:

`defaults/main.yml`:

```yaml
nginx_port: 80
```

`vars/main.yml`:

```yaml
nginx_service_name: nginx
```

Playbook can override:

```yaml
- hosts: webservers
  roles:
    - { role: nginx, nginx_port: 8080 }
```

---

## 8. Testing Roles

* Roles include a `tests/` directory.
* You can run test playbooks against local or staging environments.
* Example:

```bash
ansible-playbook roles/nginx/tests/test.yml -i roles/nginx/tests/inventory
```

---

## 9. Sharing Roles

* **Ansible Galaxy** is the central hub for sharing roles.
* You can publish roles and download community roles:

```bash
ansible-galaxy install geerlingguy.nginx
```

* This makes reusing community best practices easier.

---

## 10. Best Practices

✅ Use `ansible-galaxy init` to create roles (ensures standard structure).
✅ Put reusable code in roles instead of long playbooks.
✅ Document your role in `README.md`.
✅ Use `defaults/` for tunable variables, `vars/` only when necessary.
✅ Use handlers for restarting services.
✅ Keep templates and files organized.
✅ Use role dependencies wisely to avoid repetition.

---

## 11. Real-World Example: Nginx Role in a Playbook

`site.yml`:

```yaml
- hosts: webservers
  become: yes
  roles:
    - nginx
```

`roles/nginx/tasks/main.yml`:

```yaml
- name: Install Nginx
  apt:
    name: nginx
    state: present
    update_cache: yes

- name: Deploy nginx.conf
  template:
    src: nginx.conf.j2
    dest: /etc/nginx/sites-available/default
  notify:
    - restart nginx
```

`roles/nginx/handlers/main.yml`:

```yaml
- name: restart nginx
  service:
    name: nginx
    state: restarted
```
