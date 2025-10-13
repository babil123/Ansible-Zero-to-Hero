#  Ansible Collections 
<img src="https://github.com/bhuvan-raj/Ansible-Zero-to-Hero/blob/main/assets/collection.png" alt="Banner" width=800 height= 400 />


## 1. What is an Ansible Collection?

Ansible **Collections** are a **packaging and distribution format** for Ansible content — introduced in **Ansible 2.9**.

They bundle together **modules**, **plugins**, **roles**, and **playbooks** into a single, reusable, and distributable unit.

In short:

> A collection = a portable package that organizes and shares automation content.

---

## 🔷 2. Why Collections?

Before Ansible 2.9, all modules and plugins were part of the **Ansible core repository**, making it:

* Hard to maintain and update.
* Difficult for contributors to add new modules.
* Tightly coupled with Ansible release cycles.

With **collections**, Ansible separated content from the core engine, making it **modular**, **independent**, and **versioned** individually.

---

## 🔷 3. Structure of an Ansible Collection

A typical Ansible Collection follows this folder structure:

```
collection/
└── ansible_collections/
    └── namespace/
        └── collection_name/
            ├── docs/                 # Documentation for the collection
            ├── plugins/              # Custom plugins (modules, lookup, inventory, etc.)
            │   ├── modules/
            │   ├── lookup/
            │   └── inventory/
            ├── roles/                # Roles included in the collection
            ├── playbooks/            # Example playbooks or reusable playbooks
            ├── tests/                # Integration/unit tests
            ├── files/                # Supporting files
            ├── README.md             # Documentation overview
            └── galaxy.yml            # Metadata file (namespace, name, version, dependencies)
```

---

## 🔷 4. The `galaxy.yml` File (Metadata)

This file defines the **metadata** for your collection — similar to `package.json` or `setup.py`.

**Example:**

```yaml
namespace: my_namespace
name: my_collection
version: 1.0.0
readme: README.md
authors:
  - Bubu <bubu@example.com>
description: A collection for webserver automation
license: MIT
tags: ['webserver', 'nginx']
dependencies:
  community.general: ">=3.0.0"
repository: https://github.com/my_namespace/my_collection
```

---

## 🔷 5. Installing a Collection

You can install a collection from **Ansible Galaxy** or a **private repository**.

### 🔹 From Galaxy

```bash
ansible-galaxy collection install community.general
```

### 🔹 From a specific version

```bash
ansible-galaxy collection install community.general:==3.8.0
```

### 🔹 From Git or URL

```bash
ansible-galaxy collection install git+https://github.com/my_namespace/my_collection.git
```

### 🔹 From a tar.gz file

```bash
ansible-galaxy collection install my_namespace-my_collection-1.0.0.tar.gz
```

---

## 🔷 6. Verifying Installed Collections

To list all installed collections:

```bash
ansible-galaxy collection list
```

To see their installation path:

```bash
ansible-galaxy collection list | grep my_namespace
```

---

## 🔷 7. Using Collections in Playbooks

Once installed, you can use modules, roles, and plugins from a collection using the **fully qualified collection name (FQCN)** format:

```
<namespace>.<collection_name>.<module_name>
```

**Example:**

```yaml
- name: Install packages using yum module from community.general
  hosts: all
  tasks:
    - name: Install latest nginx
      community.general.yum:
        name: nginx
        state: latest
```

Or you can **set a collection globally** for all tasks:

```yaml
- name: Example Playbook
  hosts: webservers
  collections:
    - community.general
  tasks:
    - name: Use yum module directly
      yum:
        name: httpd
        state: present
```

---

## 🔷 8. Creating Your Own Collection

You can easily create a new collection using:

```bash
ansible-galaxy collection init my_namespace.my_collection
```

This command will generate the full collection structure with default directories and a `galaxy.yml` file.

---

## 🔷 9. Building a Collection Package

After developing your collection, you can **build it into a distributable artifact**:

```bash
ansible-galaxy collection build
```

This creates a `.tar.gz` file in your current directory, for example:

```
my_namespace-my_collection-1.0.0.tar.gz
```

---

## 🔷 10. Publishing a Collection to Galaxy

Once built, you can **upload your collection** to [Ansible Galaxy](https://galaxy.ansible.com):

1. Create an account on Ansible Galaxy and link it with your **GitHub**.
2. Push your collection to a GitHub repository.
3. Import the collection:

   * Go to your Galaxy profile → **My Content** → your **namespace**.
   * Click on **Import Collection** → Choose your repo.
   * Once imported, it’s available for others to install.

Alternatively, you can publish using the CLI:

```bash
ansible-galaxy collection publish my_namespace-my_collection-1.0.0.tar.gz --token <GALAXY_TOKEN>
```

---

## 🔷 11. Managing Collection Paths

Collections are installed in one of the following paths (depending on configuration):

| Path Type     | Default Location                                              |
| ------------- | ------------------------------------------------------------- |
| System-wide   | `/usr/share/ansible/collections/`                             |
| User-specific | `~/.ansible/collections/`                                     |
| Custom path   | Defined in `ansible.cfg` under `[defaults] collections_paths` |

**Example:**

```ini
[defaults]
collections_paths = ./collections:/usr/share/ansible/collections
```

---

## 🔷 12. Dependencies Between Collections

Collections can depend on other collections (defined in `galaxy.yml`):

```yaml
dependencies:
  community.docker: ">=1.8.0"
  ansible.posix: ">=1.3.0"
```

When installing, Ansible automatically fetches those dependencies.

---

## 🔷 13. Advantages of Collections

✅ **Modularity** – Organize content cleanly.
✅ **Reusability** – Share across teams and projects.
✅ **Version Control** – Update without breaking other playbooks.
✅ **Faster Development** – Developers can maintain their own content independently.
✅ **Decoupled from Core** – Collections evolve without waiting for Ansible releases.

---

## 🔷 14. Commonly Used Official Collections

| Collection          | Description                                          |
| ------------------- | ---------------------------------------------------- |
| `ansible.builtin`   | Default Ansible modules and plugins.                 |
| `community.general` | Large community collection with hundreds of modules. |
| `community.docker`  | Modules for Docker management.                       |
| `community.aws`     | Modules for AWS automation.                          |
| `ansible.posix`     | Linux/Unix management utilities.                     |
| `kubernetes.core`   | Kubernetes automation modules and plugins.           |

---

## 🔷 15. Best Practices

* Use **FQCN** (Fully Qualified Collection Names) to avoid ambiguity.
* Keep your collection **modular and small** (one purpose per collection).
* Always include:

  * `README.md`
  * `galaxy.yml`
  * `tests/` folder
* Use **semantic versioning** (`1.0.0`, `1.1.0`, etc.).
* Document your modules and roles properly.
* Automate build and publish using CI/CD pipelines.

---

## 🔷 16. Summary

| Concept             | Description                                                       |
| ------------------- | ----------------------------------------------------------------- |
| **Collection**      | A bundle of Ansible content (roles, modules, plugins, playbooks). |
| **Namespace**       | Top-level identifier for organization or user.                    |
| **Collection Name** | Logical name of the collection within a namespace.                |
| **FQCN**            | Format to access content: `<namespace>.<collection>.<module>`.    |
| **Galaxy.yml**      | Metadata for building and publishing.                             |
| **Install/Publish** | Managed with `ansible-galaxy` CLI commands.                       |

---

## 🔷 17. Key Commands Summary

| Command                                           | Purpose                               |
| ------------------------------------------------- | ------------------------------------- |
| `ansible-galaxy collection init ns.name`          | Create a new collection skeleton.     |
| `ansible-galaxy collection build`                 | Build a collection archive (.tar.gz). |
| `ansible-galaxy collection publish <file>.tar.gz` | Publish collection to Galaxy.         |
| `ansible-galaxy collection install <ns.name>`     | Install a collection.                 |
| `ansible-galaxy collection list`                  | List installed collections.           |

---
