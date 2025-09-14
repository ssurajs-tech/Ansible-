
# Ansible Inventory Management – Guided Exercise

This README demonstrates how to modify the default Ansible inventory file, create groups, verify hosts, and build a custom static inventory file with multiple groups and children in Ansible.

***

## Table of Contents
- [Modifying /etc/ansible/hosts](#modifying-etcansiblehosts)  
  - [Adding servera.lab.example.com](#adding-serveralabexamplecom)  
  - [Creating a webservers Group](#creating-a-webservers-group)  
  - [Verifying Managed Hosts](#verifying-managed-hosts)  
- [Creating a Custom Static Inventory](#creating-a-custom-static-inventory)  
  - [Server Inventory Specifications](#server-inventory-specifications)  
  - [Building the Custom Inventory File](#building-the-custom-inventory-file)  
  - [Verifying the Custom Inventory](#verifying-the-custom-inventory)  
- [Experimentation](#experimentation)  

***

## Modifying /etc/ansible/hosts

### Adding servera.lab.example.com
Edit the `/etc/ansible/hosts` file and append the following entry:

```bash
[student@workstation ~]$ sudo vim /etc/ansible/hosts
...output omitted...
## db-[99:101]-node.example.com

servera.lab.example.com
```

***

### Creating a webservers Group
Continue editing `/etc/ansible/hosts` and add the **[webservers]** group at the bottom:

```bash
## db-[99:101]-node.example.com

servera.lab.example.com

[webservers]
serverb.lab.example.com
```

***

### Verifying Managed Hosts
Check inventory with the `ansible --list-hosts` command.

#### List all managed hosts:
```bash
[student@workstation ~]$ ansible all --list-hosts
  hosts (2):
    servera.lab.example.com
    serverb.lab.example.com
```

#### List ungrouped hosts:
```bash
[student@workstation ~]$ ansible ungrouped --list-hosts
  hosts (1):
    servera.lab.example.com
```

#### List hosts in the webservers group:
```bash
[student@workstation ~]$ ansible webservers --list-hosts
  hosts (1):
    serverb.lab.example.com
```

***

## Creating a Custom Static Inventory

### Server Inventory Specifications
The following table defines the purpose, location, and environment for each managed host.

| Hostname                 | Purpose     | Location      | Environment |
|---------------------------|-------------|---------------|-------------|
| servera.lab.example.com  | Web server  | Raleigh       | Development |
| serverb.lab.example.com  | Web server  | Raleigh       | Testing     |
| serverc.lab.example.com  | Web server  | Mountain View | Production  |
| serverd.lab.example.com  | Web server  | London        | Production  |

Additionally:
- Groups for US cities (**Raleigh** and **Mountain View**) must be children of the **us** group.  

***

### Building the Custom Inventory File
1. Create the working directory:
```bash
[student@workstation ~]$ mkdir ~/deploy-inventory
[student@workstation ~]$ cd ~/deploy-inventory
[student@workstation deploy-inventory]$
```

2. Create and edit the `inventory` file using the specifications:

```ini
[webservers]
server[a:d].lab.example.com

[raleigh]
servera.lab.example.com
serverb.lab.example.com

[mountainview]
serverc.lab.example.com

[london]
serverd.lab.example.com

[development]
servera.lab.example.com

[testing]
serverb.lab.example.com

[production]
serverc.lab.example.com
serverd.lab.example.com

[us:children]
raleigh
mountainview
```

***

### Verifying the Custom Inventory
Use the `ansible` command with the `-i inventory` option to verify grouped hosts in your custom file.

#### List all managed hosts:
```bash
[student@workstation deploy-inventory]$ ansible all -i inventory --list-hosts
  hosts (4):
    servera.lab.example.com
    serverb.lab.example.com
    serverc.lab.example.com
    serverd.lab.example.com
```

#### List ungrouped hosts:
```bash
[student@workstation deploy-inventory]$ ansible ungrouped -i inventory \
> --list-hosts
 [WARNING]: No hosts matched, nothing to do

  hosts (0):
```

#### List hosts in the development group:
```bash
[student@workstation deploy-inventory]$ ansible development -i inventory \
> --list-hosts
  hosts (1):
    servera.lab.example.com
```

#### List hosts in the testing group:
```bash
[student@workstation deploy-inventory]$ ansible testing -i inventory \
> --list-hosts
  hosts (1):
    serverb.lab.example.com
```

#### List hosts in the production group:
```bash
[student@workstation deploy-inventory]$ ansible production -i inventory \
> --list-hosts
  hosts (2):
    serverc.lab.example.com
    serverd.lab.example.com
```

#### List hosts in the us group:
```bash
[student@workstation deploy-inventory]$ ansible us -i inventory --list-hosts
  hosts (3):
    servera.lab.example.com
    serverb.lab.example.com
    serverc.lab.example.com
```

***

## Experimentation
You are encouraged to experiment with other variations of:

```bash
ansible <host-or-group> -i inventory --list-hosts
```

