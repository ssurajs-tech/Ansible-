***

# Implementing Multiple Plays with Ansible

This exercise demonstrates how to write and execute an **Ansible Playbook** containing multiple plays to both configure and administer a managed host (`servera.lab.example.com`) and then test the configuration from another host (`workstation.lab.example.com`).

***

## Table of Contents
- [Outcomes](#outcomes)  
- [Environment Preparation](#environment-preparation)  
- [Project Setup](#project-setup)  
- [Playbook Creation](#playbook-creation)  
  - [First Play (Enable Intranet Services)](#first-play-enable-intranet-services)  
  - [Second Play (Test Intranet Web Server)](#second-play-test-intranet-web-server)  
  - [Complete Playbook](#complete-playbook)  
- [Validate Syntax](#validate-syntax)  
- [Run the Playbook](#run-the-playbook)  
- [Verify HTTPD Intranet](#verify-httpd-intranet)  
- [Clean Up](#clean-up)  

***

## Outcomes
You should be able to:  
- Construct an Ansible playbook with **multiple plays**.  
- Manage host configuration and services.  
- Verify web server accessibility through **firewalld** and the **URI check**.  

***

## Environment Preparation
As the `student` user on the `workstation` machine, prepare the environment:

```bash
[student@workstation ~]$ lab start playbook-multi
```

***

## Project Setup
Change into the project directory:

```bash
[student@workstation ~]$ cd ~/playbook-multi
[student@workstation playbook-multi]$
```

This directory already includes:
- `ansible.cfg` (configuration file)  
- `inventory` file with `servera.lab.example.com` defined  

***

## Playbook Creation
Create the `intranet.yml` playbook.

```bash
[student@workstation playbook-multi]$ vim intranet.yml
```

### First Play (Enable Intranet Services)
This play runs on `servera.lab.example.com` with privilege escalation.  

```yaml
- name: Enable intranet services
  hosts: servera.lab.example.com
  become: true
  tasks:
    - name: Latest version of httpd and firewalld installed
      ansible.builtin.dnf:
        name:
          - httpd
          - firewalld
        state: latest

    - name: Test html page is installed
      ansible.builtin.copy:
        content: "Welcome to the example.com intranet!\n"
        dest: /var/www/html/index.html

    - name: Firewall enabled and running
      ansible.builtin.service:
        name: firewalld
        enabled: true
        state: started

    - name: Firewall permits access to httpd service
      ansible.posix.firewalld:
        service: http
        permanent: true
        state: enabled
        immediate: true

    - name: Web server enabled and running
      ansible.builtin.service:
        name: httpd
        enabled: true
        state: started
```

***

### Second Play (Test Intranet Web Server)
This play tests access from `workstation.lab.example.com` using the **URI module**.

```yaml
- name: Test intranet web server
  hosts: workstation.lab.example.com
  become: false
  tasks:
    - name: Connect to intranet web server
      ansible.builtin.uri:
        url: http://servera.lab.example.com
        return_content: true
        status_code: 200
```

***

### Complete Playbook
Final content of `intranet.yml`:

```yaml
---
- name: Enable intranet services
  hosts: servera.lab.example.com
  become: true
  tasks:
    - name: Latest version of httpd and firewalld installed
      ansible.builtin.dnf:
        name:
          - httpd
          - firewalld
        state: latest

    - name: Test html page is installed
      ansible.builtin.copy:
        content: "Welcome to the example.com intranet!\n"
        dest: /var/www/html/index.html

    - name: Firewall enabled and running
      ansible.builtin.service:
        name: firewalld
        enabled: true
        state: started

    - name: Firewall permits access to httpd service
      ansible.posix.firewalld:
        service: http
        permanent: true
        state: enabled
        immediate: true

    - name: Web server enabled and running
      ansible.builtin.service:
        name: httpd
        enabled: true
        state: started

- name: Test intranet web server
  hosts: workstation.lab.example.com
  become: false
  tasks:
    - name: Connect to intranet web server
      ansible.builtin.uri:
        url: http://servera.lab.example.com
        return_content: true
        status_code: 200
```

***

## Validate Syntax
Run syntax check before execution:

```bash
[student@workstation playbook-multi]$ ansible-navigator run \
> -m stdout intranet.yml --syntax-check
```

Output:
```
playbook: /home/student/playbook-multi/intranet.yml
```

***

## Run the Playbook
Execute the playbook:

```bash
[student@workstation playbook-multi]$ ansible-navigator run \
> -m stdout intranet.yml
```

Sample output:

```
PLAY [Enable intranet services] ************************************************

TASK [Gathering Facts] *********************************************************
ok: [servera.lab.example.com]

TASK [Latest version of httpd and firewalld installed] *************************
changed: [servera.lab.example.com]

TASK [Test html page is installed] *********************************************
changed: [servera.lab.example.com]

TASK [Firewall enabled and running] ********************************************
changed: [servera.lab.example.com]

TASK [Firewall permits access to httpd service] ********************************
changed: [servera.lab.example.com]

TASK [Web server enabled and running] ******************************************
changed: [servera.lab.example.com]

PLAY [Test intranet web server] ************************************************

TASK [Gathering Facts] *********************************************************
ok: [workstation.lab.example.com]

TASK [Connect to intranet web server] ******************************************
ok: [workstation.lab.example.com]

PLAY RECAP *********************************************************************
servera.lab.example.com    : ok=6    changed=5    unreachable=0    failed=0
workstation.lab.example.com : ok=2    changed=0    unreachable=0    failed=0
```

***

## Verify HTTPD Intranet
Check the response of `curl` from the workstation:

```bash
[student@workstation playbook-multi]$ curl http://servera.lab.example.com
Welcome to the example.com intranet!
```

***

## Clean Up
When finished, return to your home directory and close the lab:

```bash
[student@workstation ~]$ lab finish playbook-multi
```

***
