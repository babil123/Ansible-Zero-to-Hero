# 🧩 **Secret Management in Ansible with Ansible Vault**

## 📘 1. What is Secret Management in Ansible?

Secret management is the process of **securely storing and accessing sensitive information** (like passwords, API keys, SSH keys, tokens, certificates, etc.) that Ansible uses in playbooks, roles, or inventory files.

If these secrets are written in plain text (like YAML files or inventories), they can easily be exposed in:

* Git repositories
* Logs
* CI/CD pipelines

To prevent that, **Ansible Vault** is used — it encrypts sensitive data and integrates seamlessly into Ansible workflows.

---

## 🔐 2. What is Ansible Vault?

**Ansible Vault** is a **built-in encryption feature** of Ansible that allows you to:

* Encrypt entire files (like variable files, inventories, etc.)
* Encrypt only specific variables or strings
* Decrypt them automatically during playbook execution (if you provide the password)

👉 Think of Vault as a **password-protected encryption layer** around your Ansible data.

---

## 🧱 3. Why Use Ansible Vault?

| Problem                          | Solution (with Vault)                                |
| -------------------------------- | ---------------------------------------------------- |
| Credentials stored in plain text | Encrypted using AES256                               |
| Git repo has secrets             | Encrypted files are safe to commit                   |
| Team collaboration               | Each team member can share the vault password        |
| Automated pipelines              | Vault password can be stored securely in CI/CD tools |

---

## ⚙️ 4. How Ansible Vault Works

* Vault uses **AES256 symmetric encryption**.
* A **Vault password** (or a password file) is required to encrypt/decrypt content.
* Ansible automatically decrypts the vault content **at runtime** during playbook execution.

---

## 🧩 5. Ways to Use Ansible Vault

You can use Ansible Vault in three main ways:

### (a) Encrypt Entire Files

Used for variable files, inventory files, or any YAML file.

```bash
ansible-vault encrypt vars.yml
```

🔹 Output:

```
Encryption successful
```

Now `vars.yml` is fully encrypted — you can safely commit it to Git.

---

### (b) Encrypt Specific Variables (Inline Encryption)

You can encrypt only the sensitive part instead of the whole file.

Example:

```bash
ansible-vault encrypt_string 'SuperSecret123!' --name 'db_password'
```

🔹 Output:

```yaml
db_password: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          6332613261356138...
```

You can place that variable directly inside any playbook or variable file.

---

### (c) Create Encrypted Files Directly

You can create a new file that’s automatically encrypted from the start.

```bash
ansible-vault create secrets.yml
```

This command will:

* Open your default editor (like vim or nano)
* Let you enter your secrets
* Save and encrypt the file

---

## 🛠️ 6. Common Ansible Vault Commands

| Command                        | Description                                        |
| ------------------------------ | -------------------------------------------------- |
| `ansible-vault create <file>`  | Create a new encrypted file                        |
| `ansible-vault encrypt <file>` | Encrypt an existing file                           |
| `ansible-vault decrypt <file>` | Decrypt a file                                     |
| `ansible-vault edit <file>`    | Edit an encrypted file                             |
| `ansible-vault rekey <file>`   | Change the encryption password                     |
| `ansible-vault view <file>`    | View encrypted file contents                       |
| `ansible-vault encrypt_string` | Encrypt only a string (inline variable encryption) |

---

## 🔧 7. Using Vault in a Playbook

### Example: `vars/secrets.yml` (encrypted file)

```yaml
db_user: admin
db_password: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          6332613261356138...
```

### Example: `site.yml`

```yaml
---
- name: Deploy Application
  hosts: web
  vars_files:
    - vars/secrets.yml
  tasks:
    - name: Print database credentials
      debug:
        msg: "DB User: {{ db_user }}, Password: {{ db_password }}"
```

To run this playbook:

```bash
ansible-playbook site.yml --ask-vault-pass
```

Ansible will prompt for the vault password and decrypt the variables automatically.

---

## ⚙️ 8. Using a Vault Password File (Automation-Friendly)

Instead of typing the password every time, you can store it in a **secure local file**.

```bash
echo "MyVaultPassword" > ~/.vault_pass.txt
chmod 600 ~/.vault_pass.txt
```

Then run:

```bash
ansible-playbook site.yml --vault-password-file ~/.vault_pass.txt
```

⚠️ **Never commit the password file to Git!**

---

## 💡 9. Using Multiple Vaults (Vault IDs)

In large environments, you might use **different vaults for different environments** — for example, *dev*, *staging*, and *prod*.

You can assign **vault IDs**:

```bash
ansible-vault encrypt --vault-id dev@prompt dev_vars.yml
ansible-vault encrypt --vault-id prod@prompt prod_vars.yml
```

Run your playbook with multiple vaults:

```bash
ansible-playbook site.yml --vault-id dev@prompt --vault-id prod@prompt
```

---

## 🔍 10. Best Practices for Secret Management

✅ **Do**

* Use `ansible-vault` for all sensitive data.
* Store non-sensitive variables separately.
* Use environment-based vaults (`dev`, `prod`).
* Restrict vault password access using permissions.
* Automate with a vault password file in CI/CD (but never commit it).

❌ **Don’t**

* Store plaintext secrets in Git.
* Share vault passwords in chat or email.
* Hardcode secrets inside playbooks.

---

## 🧠 11. Example Real-World Scenario

Let’s say you have a MySQL password you don’t want exposed.

1. Encrypt it:

   ```bash
   ansible-vault encrypt_string 'MyStrongP@ssw0rd' --name 'mysql_root_password'
   ```

2. Paste it into your variable file:

   ```yaml
   # group_vars/database.yml
   mysql_root_password: !vault |
             $ANSIBLE_VAULT;1.1;AES256
             623730333862336...
   ```

3. Use it in a task:

   ```yaml
   - name: Set MySQL root password
     mysql_user:
       name: root
       password: "{{ mysql_root_password }}"
       host_all: yes
       login_user: root
       login_password: "{{ mysql_root_password }}"
   ```

When you run the playbook, Ansible will decrypt the password automatically.

---

## 🧩 12. Integrating Vault with Git and CI/CD

### Git:

* Encrypted files (`*.yml` with vault) can be safely versioned.
* Password file must be kept out of Git.

### CI/CD:

In Jenkins, GitHub Actions, or GitLab CI:

* Store the vault password in the CI secret manager.
* Pass it as an environment variable:

  ```bash
  ansible-playbook site.yml --vault-password-file <(echo "$ANSIBLE_VAULT_PASS")
  ```

---

## 🧭 13. Limitations of Ansible Vault

* Vaults are **static** — no dynamic rotation like AWS Secrets Manager.
* Vault passwords must be manually managed.
* No centralized access control — access = whoever has the password.

To overcome these, many teams integrate **Ansible Vault** with:

* **HashiCorp Vault**
* **AWS Secrets Manager**
* **Azure Key Vault**
  (using community modules)

---

## 🧾 14. Summary

| Concept                    | Description                                                    |
| -------------------------- | -------------------------------------------------------------- |
| **Ansible Vault**          | Encrypts sensitive data for Ansible                            |
| **Encryption type**        | AES256                                                         |
| **Encrypt entire files**   | `ansible-vault encrypt file.yml`                               |
| **Encrypt variables only** | `ansible-vault encrypt_string`                                 |
| **Run with vault**         | `ansible-playbook play.yml --ask-vault-pass`                   |
| **Automate decryption**    | Use `--vault-password-file`                                    |
| **Multiple vaults**        | Use `--vault-id` for different environments                    |
| **Best practice**          | Separate secrets, protect password file, and automate securely |

---
