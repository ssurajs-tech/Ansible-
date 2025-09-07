
# Red Hat Ansible Automation Platform 2 – Overview & Lab Setup

This repository provides an overview of **Red Hat Ansible Automation Platform 2** and step-by-step instructions for installing and working with its key components such as **ansible-core**, **automation content navigator**, **execution environments**, and more.

---

## 🚀 Overview

**Red Hat Ansible Automation Platform 2** includes a number of distinct components that together provide a complete and integrated set of automation tools and resources.

### 🔹 Ansible Core

* Provides the **fundamental automation functionality** to run Playbooks.
* Defines the **automation language** (YAML-based).
* Includes loops, conditionals, modules, and basic CLI tools.
* In version **2.2**, Ansible Core **2.13** is provided via:

  * `ansible-core` RPM package
  * `ee-minimal-rhel8` and `ee-supported-rhel8` execution environments

---

### 🔹 Ansible Content Collections

* Previously, Ansible followed a **"batteries included"** model (large number of built-in modules).
* With rapid growth, modules were reorganized into **Content Collections**:

  * Related **modules, roles, and plugins** grouped together
  * Supported by specific developer groups
* **ansible.builtin** is always included in Ansible Core.
* Red Hat provides **120+ certified collections** with subscription access.
* Community collections available via [Ansible Galaxy](https://galaxy.ansible.com).

---

### 🔹 Automation Content Navigator (`ansible-navigator`)

* New top-level CLI tool for **developing and testing playbooks**.
* Replaces utilities like `ansible-playbook`, `ansible-inventory`, `ansible-config`.
* Runs playbooks in **containers**, separating:

  * **Control Node** (where navigator runs)
  * **Execution Environment** (container where automation runs)
* Ensures consistency between **development and production**.

---

### 🔹 Automation Execution Environments

* **Container images** that package everything needed:

  * Ansible Core
  * Content Collections
  * Python libraries, dependencies
* Playbooks executed in chosen environments with `ansible-navigator`.
* Ensures **portability and reproducibility**.

---

### 🔹 Automation Controller

* Formerly **Ansible Tower**.
* Provides:

  * **Web UI**
  * **REST API**
  * Central management of automation code execution.

---

### 🔹 Automation Hub

* Available at [console.redhat.com](https://console.redhat.com).
* Provides access to **certified collections**.
* Compatible with `ansible-galaxy` and automation controller.

---

## ⚙️ Installing Ansible

### 1. Install `ansible-navigator`

```bash
sudo dnf install ansible-navigator
```

This sets up your workstation as a **control node**.

> ℹ️ In production:
> Use `subscription-manager` to register your system and enable the required repo:
> `ansible-automation-platform-2.2-for-rhel-9-x86_64-rpms`

---

### 2. Verify Installation

```bash
ansible-navigator --version
```

---

### 3. Log in to Automation Hub Registry

```bash
podman login utility.lab.example.com -u admin -p redhat
```

---

### 4. Download Execution Environment Image

```bash
ansible-navigator images
```

* Downloads `ee-supported-rhel8:latest`
* Displays available images in interactive mode

---

## 🎯 Outcomes

By completing the setup, you will:

* Be able to install and use **ansible-navigator**.
* Run automation inside **execution environments**.
* Access and use **Red Hat Certified Collections**.
* Integrate automation into enterprise workflows with **automation controller**.

---

## 📌 Summary

* Automation reduces human error and ensures **consistent infrastructure state**.
* **Ansible**: open-source automation platform for **Linux, Windows, and network devices**.
* **Playbooks**: human-readable YAML defining desired infrastructure state.
* Agentless design → installed only on **control node** and in **execution environments**.
* **ansible-navigator**: key tool for developing, testing, and running Ansible automation.


