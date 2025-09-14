***

# Writing and Running Ansible Playbooks

This guide walks through writing and running an **Ansible Playbook** to install and configure Apache HTTPD servers, test functionality, and verify results. The objective is to practice using YAML playbooks with `ansible-navigator`.

***

## Table of Contents
- [Outcomes](#outcomes)  
- [Environment Preparation](#environment-preparation)  
- [Project Directory](#project-directory)  
- [Playbook Creation](#playbook-creation)  
  - [Playbook Structure](#playbook-structure)  
  - [Tasks](#tasks)  
- [Validate Syntax](#validate-syntax)  
- [Run the Playbook](#run-the-playbook)  
  - [First Run](#first-run)  
  - [Idempotency Test](#idempotency-test-second-run)  
- [Verify HTTPD Setup](#verify-httpd-setup)  
- [Clean Up](#clean-up)  

***

## Outcomes
You should be able to:
- Write a playbook using YAML syntax and Ansible Playbook structure.  
- Run it with the `ansible-navigator run` command.  
- Deploy **Apache HTTPD** and serve test pages on multiple servers.  

***

## Environment Preparation
As the `student` user on the `workstation` machine, prepare the system for this exercise:

```bash
[student@workstation ~]$ lab start playbook-basic
```

***

## Project Directory
The `~/playbook-basic/` directory contains:
- `ansible.cfg` (pre-created)  
- `inventory` (with `web` group defined: `serverc.lab.example.com` and `serverd.lab.example.com`)  

Change into this directory:

```bash
[student@workstation ~]$ cd ~/playbook-basic
[student@workstation playbook-basic]$
```

***

## Playbook Creation

Create the playbook:  

```bash
[student@workstation playbook-basic]$ vim site.yml
```

### Playbook Structure
```yaml
---
- name: Install and start Apache HTTPD
  hosts: web
  tasks:
    - name: Ensure httpd package is present
      ansible.builtin.dnf:
        name: httpd
        state: present

    - name: Correct index.html is present
      ansible.builtin.copy:
        src: files/index.html
        dest: /var/www/html/index.html

    - name: Ensure httpd is started
      ansible.builtin.service:
        name: httpd
        state: started
        enabled: true
```

***

## Validate Syntax
Before running, validate the playbook:

```bash
[student@workstation playbook-basic]$ ansible-navigator run \
> -m stdout site.yml --syntax-check
```

Output:
```
playbook: /home/student/playbook-basic/site.yml
```

***

## Run the Playbook

### First Run
```bash
[student@workstation playbook-basic]$ ansible-navigator run \
> -m stdout site.yml
```

Output excerpt:
```
PLAY [Install and start Apache HTTPD] ******************************************

TASK [Gathering Facts] *********************************************************
ok: [serverd.lab.example.com]
ok: [serverc.lab.example.com]

TASK [Ensure httpd package is present] *****************************************
changed: [serverd.lab.example.com]
changed: [serverc.lab.example.com]

TASK [Correct index.html is present] *******************************************
changed: [serverd.lab.example.com]
changed: [serverc.lab.example.com]

TASK [Ensure httpd is started] *************************************************
changed: [serverd.lab.example.com]
changed: [serverc.lab.example.com]

PLAY RECAP *********************************************************************
serverc.lab.example.com    : ok=4    changed=3    unreachable=0    failed=0
serverd.lab.example.com    : ok=4    changed=3    unreachable=0    failed=0
```

***

### Idempotency Test (Second Run)
```bash
[student@workstation playbook-basic]$ ansible-navigator run \
> -m stdout site.yml
```

On the second run, no changes should occur:

```
TASK [Ensure httpd package is present] *****************************************
ok: [serverd.lab.example.com]
ok: [serverc.lab.example.com]

TASK [Correct index.html is present] *******************************************
ok: [serverc.lab.example.com]
ok: [serverd.lab.example.com]

TASK [Ensure httpd is started] *************************************************
ok: [serverd.lab.example.com]
ok: [serverc.lab.example.com]

PLAY RECAP *********************************************************************
serverc.lab.example.com    : ok=4    changed=0    unreachable=0    failed=0
serverd.lab.example.com    : ok=4    changed=0    unreachable=0    failed=0
```

***

## Verify HTTPD Setup
Use `curl` to confirm the test page is served by both hosts:

```bash
[student@workstation playbook-basic]$ curl serverc.lab.example.com
This is a test page.

[student@workstation playbook-basic]$ curl serverd.lab.example.com
This is a test page.
```

***

## Clean Up
Finish the lab to reset the environment:

```bash
[student@workstation ~]$ lab finish playbook-basic
```

***
