# ansible-secure-infra
# Ansible Web Server Setup

This repository contains Ansible playbooks to automate the setup of an Apache web server. 

Common challenges encountered when running Ansible within a Windows Subsystem for Linux (WSL) environment.

---

## Getting Started

To get this playbook running, follow the steps, which also cover typical debugging scenarios.

### 1. Project Relocation & Permissions (World Writable Directory Warning)

Problem: Running Ansible from a Windows-mounted directory in WSL often triggers a "world writable directory" warning due as Linux permissions aren't properly handled.

Solution: Move your Ansible project to a native Linux filesystem within your WSL instance.

**Steps:**
1.  **Create your project directory in WSL and navigate into it:**
    ```bash
    mkdir -p /home/.......
    cd /home/..........
    ```
2.  **Copy all contents from your old Windows location:**
    ```bash
    
3.  **Set correct permissions:**
    ```bash
    chmod go-w . playbooks roles roles/apache
    chmod -R go-w roles/apache
    ```

### 2. Role Not Found Error (`apache` role)

Problem: Ansible couldn't find the `apache` role, resulting in an `ERROR! the role 'apache' was not found`.

**Solution:** Ensure the `apache` role adheres to Ansible's standard directory structure and is correctly placed within your project's `roles` directory.

**Steps:**
1.  **Create the standard role directories:**
    ```bash
    mkdir -p roles/apache/tasks roles/apache/handlers roles/apache/templates roles/apache/files roles/apache/defaults roles/apache/meta
    ```
2.  **Create and populate `roles/apache/tasks/main.yml`:**
    This file should contain tasks for installing Apache, copying configurations (like an `index.html`), and ensuring the service is running.
    *(Example content for `roles/apache/tasks/main.yml` is in the previous conversations if you need it).*
3.  **Verify `playbooks/webserver.yml`** correctly calls the `apache` role:
    ```yaml
    ---
    - name: Configure Webserver
      hosts: all
      become: yes
      roles:
        - apache
    ```

### 3. Sudo Password Prompt (`sudo: a password is required`)

**Problem:** Tasks requiring elevated privileges (`become: yes`) failed, prompting for a `sudo` password.

**Solution:** Tell Ansible to ask for the `sudo` password interactively.

**Step:**
1.  **Run the playbook with `--ask-become-pass` (or `-K`):**
    ```bash
    ansible-playbook playbooks/webserver.yml --ask-become-pass
    ```

### 4. APT Cache Update Failure (`Failed to update apt cache`)

**Problem:** The playbook failed due to the `apt` module being unable to update its package cache. This was traced to a specific third-party repository (`download.falcosecurity.org`) experiencing a temporary resolution failure.

**Solution:** Remove the problematic repository definition from your system's APT sources.

**Steps:**
1.  **Identify and remove repository files:**
    ```bash
    ls /etc/apt/sources.list.d/
    sudo rm /etc/apt/sources.list.d/falcosecurity.list /etc/apt/sources.list.d/falcosecurity.list.save
    ```
2.  **Manually update APT cache to confirm the fix:**
    ```bash
    sudo apt update
    ```
3.  **Re-run the Ansible playbook:**
    ```bash
    ansible-playbook playbooks/webserver.yml --ask-become-pass
    ```

---

## ✅ Verification

Once all issues are resolved, your Ansible playbook should run successfully.

**Example of Successful Playbook Output:**
✅ You can find a sample output of a successful Ansible playbook run in the [playbook_success_output.txt]
