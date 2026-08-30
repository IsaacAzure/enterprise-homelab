# Lab Overview

## Background

My primary operating system is **Fedora Linux**, currently running Fedora 44.

I wanted to build a virtual home lab on this device to develop and document practical infrastructure and systems administration skills.

Previously, when using Windows as my primary operating system, I used VirtualBox to build a simple Active Directory home lab. Since moving to Fedora, I have also used GNOME Boxes to create and experiment with several virtual machines, including:

* Fedora
* Ubuntu
* Rocky Linux
* Kali Linux
* Open Suse

For this project, I wanted to move beyond individual virtual machines and build a more structured home lab environment that could be expanded over time.

## Initial Lab Architecture

The initial design consisted of a Fedora host running virtual machines through **KVM/QEMU and virt-manager**.

```text
Fedora Host
KVM/QEMU via virt-manager
│
├── Windows Server
│   ├── Active Directory Domain Services
│   ├── DNS
│   ├── DHCP
│   │
│   ├── Windows 11 Client #1
│   └── Windows 11 Client #2
│
├── Ubuntu Server
│   ├── SSH
│   ├── File Sharing
│   └── Web Services
│
└── Rocky Linux
    └── RHCSA Practice Environment
```

This initial environment was designed as the foundation of a larger infrastructure blueprint. Rather than attempting to build the entire environment at once, I decided to start with a smaller, functional deployment and expand it as the project developed.

## Initial Objectives

The first stage of the project focused on building the Windows infrastructure environment.

The initial tasks were:

1. Install and configure **KVM/QEMU and virt-manager** on Fedora.
2. Build a **Windows Server virtual machine**.
3. Configure the server as a **Domain Controller**.
4. Install and configure the **DNS and DHCP roles**.
5. Build two **Windows client virtual machines**.
6. Join both clients to the Active Directory domain.

Once the core environment was operational, the project expanded into areas including Active Directory organisation, delegated administration, Group Policy, PowerShell Remoting, RSAT, testing, and troubleshooting.

## Project Approach

The lab is being developed incrementally.

Each major component is:

1. **Designed**
2. **Configured**
3. **Tested**
4. **Validated against the intended design**
5. **Documented**

The goal is not simply to build a working environment, but to understand how the different systems interact and to document both successful configurations and troubleshooting processes encountered along the way.
