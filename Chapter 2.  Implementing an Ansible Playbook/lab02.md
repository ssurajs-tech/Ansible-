***

# Ansible Playbook Manage Exercise

This repository documents the steps for configuring **Ansible Navigator** with persistent custom settings and testing inventory configurations using playbooks.  

The exercise ensures that the automation environment is properly configured and validated through a set of structured tasks.

***

## Table of Contents
- [Environment Preparation](#environment-preparation)  
- [Configure Automation Content Navigator](#configure-automation-content-navigator)  
- [Configure Ansible Defaults](#configure-ansible-defaults)  
- [Static Inventory Setup](#static-inventory-setup)  
- [Validate Inventory with Playbooks](#validate-inventory-with-playbooks)  
- [Privilege Escalation Setup](#privilege-escalation-setup)  
- [Final Step - Clean Up](#final-step---clean-up)  

***

## Environment Preparation
As the `student` user on the `workstation` machine, prepare the system for this exercise:

```bash
[student@workstation ~]$ lab start playbook-manage
```

Change into the **playbook-manage** directory:

```bash
[student@workstation ~]$ cd ~/playbook-manage
[student@workstation playbook-manage]$
```

***

## Configure Automation Content Navigator
Create the `ansible-navigator.yml` file with the following content:

```yaml
---
ansible-navigator:
  execution-environment:
    image: utility.lab.example.com/ee-supported-rhel8:latest
    pull:
      policy: missing
  playbook-artifact:
    enable: false
```

Run the following command to list available execution environment images:

```bash
[student@workstation playbook-manage]$ ansible-navigator images
```

Example output (truncated):

```
----------------------------------------------------------------------------------
Execution environment image and pull policy overview
----------------------------------------------------------------------------------
Execution environment image name:     utility.lab.example.com/ee-supported-rhel8:latest
Execution environment image tag:      latest
Execution environment pull arguments: None
Execution environment pull policy:    missing
Execution environment pull needed:    True
----------------------------------------------------------------------------------
Updating the execution environment
...output omitted...
Running the command: podman pull utility.lab.example.com/ee-supported-rhel8:latest
Trying to pull utility.lab.example.com/ee-supported-rhel8:latest...
...output omitted...
```

After pulling, the image should appear in the list:

```
  Image                 Tag     Execution environment    Created      Size
0│ee-supported-rhel8    latest  True                     3 weeks ago  1.34 GB
```

Press `Esc` to exit the image list.

***

## Configure Ansible Defaults
In the **playbook-manage** directory, create the `ansible.cfg` file:

```ini
[defaults]
inventory = ./inventory
```

***

## Static Inventory Setup
Create the `inventory` file with the following structure:

```ini
[myself]
workstation.lab.example.com

[intranetweb]
servera.lab.example.com

[internetweb]
serverb.lab.example.com

[web:children]
intranetweb
internetweb
```

***

## Validate Inventory with Playbooks

### Verify `myself` group
```bash
[student@workstation playbook-manage]$ ansible-navigator run \
> -m stdout ping-myself.yml
```

Output:

```
PLAY [Validate inventory hosts] **********************************************

TASK [Ping myself] ***********************************************************
ok: [workstation.lab.example.com]

PLAY RECAP *******************************************************************
workstation.lab.example.com : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

### Verify `intranetweb` group
```bash
[student@workstation playbook-manage]$ ansible-navigator run \
> -m stdout ping-intranetweb.yml
```

```
ok: [servera.lab.example.com]
```

### Verify `internetweb` group
```bash
[student@workstation playbook-manage]$ ansible-navigator run \
> -m stdout ping-internetweb.yml
```

```
ok: [serverb.lab.example.com]
```

### Verify `web` group (children of both intranetweb & internetweb)
```bash
[student@workstation playbook-manage]$ ansible-navigator run \
> -m stdout ping-web.yml
```

```
ok: [servera.lab.example.com]
ok: [serverb.lab.example.com]
```

### Verify all hosts
```bash
[student@workstation playbook-manage]$ ansible-navigator run \
> -m stdout ping-all.yml
```

```
ok: [serverb.lab.example.com]
ok: [servera.lab.example.com]
ok: [workstation.lab.example.com]
```

***

## Privilege Escalation Setup
Open the `ansible.cfg` file and add privilege escalation configuration:

```ini
[defaults]
inventory = ./inventory
remote_user = student

[privilege_escalation]
become = true
become_method = sudo
become_user = root
become_ask_pass = true
```

Run the playbook again:

```bash
[student@workstation playbook-manage]$ ansible-navigator run \
> -m stdout ping-intranetweb.yml
```

You’ll now be prompted for the sudo password:

```
BECOME password: student
PLAY [Validate inventory hosts] **********************************************
ok: [servera.lab.example.com]
```

***

## Final Step - Clean Up
After testing, return to the home directory and finish the lab:

```bash
[student@workstation ~]$ lab finish playbook-manage
```

***
