***

# Managing Ansible Configuration Files

This section covers where Ansible configuration files are located, how Ansible selects them, and how to edit them to apply changes to default settings for both `ansible.cfg` and `ansible-navigator.yml`.

***

## Table of Contents
- [Objectives](#objectives)  
- [Configuring Ansible](#configuring-ansible)  
- [Managing Ansible Settings](#managing-ansible-settings)  
  - [Sample ansible.cfg](#sample-ansiblecfg)  
  - [ansible.cfg Parameters Explained](#ansiblecfg-parameters-explained)  
- [Privilege Escalation](#privilege-escalation)  
- [Connection Settings](#connection-settings)  
- [Deploying SSH Keys](#deploying-ssh-keys)  
- [Determining Current Configuration Settings](#determining-current-configuration-settings)  
- [Managing Settings for Automation Content Navigator](#managing-settings-for-automation-content-navigator)  
  - [Sample ansible-navigator.yml](#sample-ansible-navigatoryml)  
- [Configuration File Comments](#configuration-file-comments)  

***

## Objectives
- Describe where Ansible configuration files are located.  
- Learn how Ansible selects them.  
- Edit them to apply changes to default settings.  

***

## Configuring Ansible
You can create and edit **two files** in each of your Ansible project directories that configure the behavior of Ansible and the `ansible-navigator` command.  

- `ansible.cfg`: Configures the behavior of several Ansible tools.  
- `ansible-navigator.yml`: Changes default options for the `ansible-navigator` command.  

An *Ansible project directory* is a directory from which you run your playbooks using the `ansible-navigator` command.  

***

## Managing Ansible Settings
Create an **ansible.cfg** configuration file in each of your Ansible project directories.  

It consists of sections (in brackets) with key-value pairs.  

At minimum, include:  
- `[defaults]` → sets defaults for Ansible  
- `[privilege_escalation]` → privilege escalation on managed hosts  

***

### Sample ansible.cfg
```ini
[defaults]
inventory = ./inventory 
remote_user = user 
ask_pass = false 

[privilege_escalation]
become = true 
become_method = sudo 
become_user = root 
become_ask_pass = false 
```

***

### ansible.cfg Parameters Explained
- **inventory** → Path to static file, directory of files, or dynamic inventory scripts.  
- **remote_user** → User Ansible uses to connect to hosts. Defaults to `root` when using `ansible-navigator`.  
- **ask_pass** → Whether to prompt for SSH password. `false` = key-based auth, `true` = password.  
- **become** → Whether to switch (usually to root). Defaults `false`.  
- **become_method** → Method used to escalate privileges. Default `sudo`.  
- **become_user** → Target user after escalation (default: root).  
- **become_ask_pass** → Whether to prompt for sudo password (default: false).  

**Important**:  
Local files `~/.ansible.cfg` or `/etc/ansible/ansible.cfg` can be used by `ansible-playbook`, but **not** by `ansible-navigator`.  

***

## Privilege Escalation
Some tasks (e.g., installing software) need elevated privileges.  

On RHEL 8/9:  
- `/etc/sudoers` grants members of the **wheel** group sudo rights.  
- Example `/etc/sudoers` contents:  

```conf
## Allows people in group wheel to run all commands
%wheel  ALL=(ALL)       ALL

## Same thing without a password
# %wheel        ALL=(ALL)       NOPASSWD: ALL
```

If sudo needs a password, set in `ansible.cfg`:  

```ini
[privilege_escalation]
become_ask_pass = true
```

Security tradeoffs vary by organization.  

***

## Connection Settings
By default, Ansible uses **SSH** to connect to managed hosts.  

- Set `remote_user` to choose the connection user.  
- If you don’t, `ansible-navigator` uses `root` by default.  
- Public/private SSH keys are auto-used if available.  
- `ask_pass` enables password-based SSH login.  

***

## Deploying SSH Keys
If you don’t have keys:  

```bash
[user@controlnode ~]$ ssh-keygen
...output omitted...
```

Copy the public key:  

```bash
[user@controlnode ~]$ ssh-copy-id devops@host.example.com
...output omitted...
```

Or automate for **all hosts** with a playbook:  

```yaml
---
- name: Deploy public key to all hosts
  hosts: all
  tasks:
    - name: Copy public SSH key
      ansible.posix.authorized_key:
        user: devops
        key: "{{ lookup('ansible.builtin.file', '~/.ssh/id_rsa.pub') }}"
        state: present
```

Run with:  

```bash
ansible-navigator run --ask-pass --pae false -m stdout
```

**Important**: If you don’t set `--pae false` and `-m stdout`, the command may hang.  

***

## Determining Current Configuration Settings
Run:  

```bash
ansible-navigator config
```

Sample output:  

```
   Name                        Default Source                            Current
...output omitted...
44│Default ask pass            True    default                           False
45│Default ask vault pass      True    default                           False
46│Default become              False   /home/student/project/ansible.cfg True
47│Default become ask pass     False   /home/student/project/ansible.cfg True
 ...output omitted...
50│Default become method       False   /home/student/project/ansible.cfg sudo
51│Default become user         False   /home/student/project/ansible.cfg root
 ...output omitted...
```

- `Default` → default value.  
- `Source` → config file or env var.  
- `Current` → actual in-use value.  

Exit interactive mode with **Esc** or `:q`.  

***

## Managing Settings for Automation Content Navigator
The **ansible-navigator** command has its own optional YAML/JSON settings file.  

### It looks for files in this order:
1. `ANSIBLE_NAVIGATOR_CONFIG` env variable  
2. `ansible-navigator.yml` (project directory)  
3. `~/.ansible-navigator.yml`  

***

### Sample ansible-navigator.yml
```yaml
---
ansible-navigator:
  execution-environment: 
    image: utility.lab.example.com/ee-supported-rhel8:latest 
    pull:
      policy: missing 
  playbook-artifact: 
    enable: false 
  mode: stdout 
```

#### Explanation
- **execution-environment.image** → Container image for automation.  
- **execution-environment.pull.policy** → `missing` = pull only if not present locally.  
- **playbook-artifact.enable** → Disable JSON artifact generation. Required for password-prompts.  
- **mode** → `interactive` (default) or `stdout`.  

More docs: [Ansible Navigator Settings](https://ansible.readthedocs.io/projects/navigator/settings/)  

***

## Configuration File Comments
- Both `ansible.cfg` and `ansible-navigator.yml` use `#` for full-line comments.  
- `ansible.cfg` also supports `;` as inline comments.  

Example:  
```ini
[defaults]
inventory = ./inventory  # Relative path to inventory
remote_user = user ; This specifies the SSH user
```

***
