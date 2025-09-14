***

# Implementing an Ansible Playbook

This exercise demonstrates how to write and run an **Ansible Playbook** to install, configure, and verify the status of **web** and **database** services on a managed host (`serverb.lab.example.com`) and test them from the control node (`workstation.lab.example.com`).  

***

## Table of Contents
- [Outcomes](#outcomes)  
- [Environment Preparation](#environment-preparation)  
- [Project Setup](#project-setup)  
- [Playbook Creation](#playbook-creation)  
  - [First Play - Enable Internet Services](#first-play---enable-internet-services)  
  - [Second Play - Test Internet Web Server](#second-play---test-internet-web-server)  
  - [Complete Playbook](#complete-playbook)  
- [Validate Syntax](#validate-syntax)  
- [Run the Playbook](#run-the-playbook)  
- [Clean Up](#clean-up)  

***

## Outcomes
You should be able to:
- Construct and run an Ansible Playbook with **multiple plays**.  
- Install and configure **firewalld**, **httpd**, **MariaDB**, and **PHP**.  
- Validate that the services run correctly and serve PHP content.  

***

## Environment Preparation
As the `student` user, start the lab environment:

```bash
[student@workstation ~]$ lab start playbook-review
```

***

## Project Setup
Change to the working directory:

```bash
[student@workstation ~]$ cd ~/playbook-review
[student@workstation playbook-review]$
```

This directory already includes:
- `ansible.cfg`
- `inventory` file with `serverb.lab.example.com`  

***

## Playbook Creation
Create the `internet.yml` playbook.

```bash
[student@workstation playbook-review]$ vim internet.yml
```

***

### First Play - Enable Internet Services
The first play configures `serverb.lab.example.com` with privilege escalation.  

```yaml
---
- name: Enable internet services
  hosts: serverb.lab.example.com
  become: true
  tasks:
    - name: Latest version of all required packages installed
      ansible.builtin.dnf:
        name:
          - firewalld
          - httpd
          - mariadb-server
          - php
          - php-mysqlnd
        state: latest

    - name: firewalld enabled and running
      ansible.builtin.service:
        name: firewalld
        enabled: true
        state: started

    - name: firewalld permits http service
      ansible.posix.firewalld:
        service: http
        permanent: true
        state: enabled
        immediate: true

    - name: httpd enabled and running
      ansible.builtin.service:
        name: httpd
        enabled: true
        state: started

    - name: mariadb enabled and running
      ansible.builtin.service:
        name: mariadb
        enabled: true
        state: started

    - name: Test php page is installed
      ansible.builtin.copy:
        src: index.php
        dest: /var/www/html/index.php
        mode: 0644
```

***

### Second Play - Test Internet Web Server
This play tests the web service from the control node without privilege escalation.  

```yaml
- name: Test internet web server
  hosts: workstation.lab.example.com
  become: false
  tasks:
    - name: Connect to internet web server
      ansible.builtin.uri:
        url: http://serverb.lab.example.com
        status_code: 200
```

***

### Complete Playbook
Final content of `internet.yml`:

```yaml
---
- name: Enable internet services
  hosts: serverb.lab.example.com
  become: true
  tasks:
    - name: Latest version of all required packages installed
      ansible.builtin.dnf:
        name:
          - firewalld
          - httpd
          - mariadb-server
          - php
          - php-mysqlnd
        state: latest

    - name: firewalld enabled and running
      ansible.builtin.service:
        name: firewalld
        enabled: true
        state: started

    - name: firewalld permits http service
      ansible.posix.firewalld:
        service: http
        permanent: true
        state: enabled
        immediate: true

    - name: httpd enabled and running
      ansible.builtin.service:
        name: httpd
        enabled: true
        state: started

    - name: mariadb enabled and running
      ansible.builtin.service:
        name: mariadb
        enabled: true
        state: started

    - name: Test php page is installed
      ansible.builtin.copy:
        src: index.php
        dest: /var/www/html/index.php
        mode: 0644

- name: Test internet web server
  hosts: workstation.lab.example.com
  become: false
  tasks:
    - name: Connect to internet web server
      ansible.builtin.uri:
        url: http://serverb.lab.example.com
        status_code: 200
```

***

## Validate Syntax
Run syntax check on the playbook:

```bash
[student@workstation playbook-review]$ ansible-navigator run \
> -m stdout internet.yml --syntax-check
```

Output:
```
playbook: /home/student/playbook-review/internet.yml
```

***

## Run the Playbook
Execute the playbook:

```bash
[student@workstation playbook-review]$ ansible-navigator run \
> -m stdout internet.yml
```

Sample output:

```
PLAY [Enable internet services] ************************************************

TASK [Gathering Facts] *********************************************************
ok: [serverb.lab.example.com]

TASK [Latest version of all required packages installed] ***********************
changed: [serverb.lab.example.com]

TASK [firewalld enabled and running] *******************************************
ok: [serverb.lab.example.com]

TASK [firewalld permits http service] ******************************************
changed: [serverb.lab.example.com]

TASK [httpd enabled and running] ***********************************************
changed: [serverb.lab.example.com]

TASK [mariadb enabled and running] *********************************************
changed: [serverb.lab.example.com]

TASK [Test php page is installed] **********************************************
changed: [serverb.lab.example.com]

PLAY [Test internet web server] ************************************************

TASK [Gathering Facts] *********************************************************
ok: [workstation.lab.example.com]

TASK [Connect to internet web server] ******************************************
ok: [workstation.lab.example.com]

PLAY RECAP *********************************************************************
serverb.lab.example.com    : ok=7    changed=5    unreachable=0    failed=0 ...
workstation.lab.example.com : ok=2    changed=0    unreachable=0    failed=0 ...
```

***

## Clean Up
Finish the lab to avoid conflicts with later exercises:

```bash
[student@workstation ~]$ lab finish playbook-review
```

***
