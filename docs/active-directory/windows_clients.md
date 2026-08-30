# Adding Windows Clients

With the Domain Controller, DNS, and DHCP services configured, I could begin adding Windows client systems to the lab.

The first Windows 11 workstation also exposed an important networking issue in the virtual environment, which led to further troubleshooting of DHCP and the creation of the new subnet documented previously.

---

## Preparing Active Directory

Before creating the client systems, I created an Organizational Unit (OU) in Active Directory called:

`Workstations`

This OU would be used to contain the Windows workstation computer objects as they were added to the domain.

---

## Creating WS_01

I created the first Windows 11 client using **virt-manager**.

The virtual machine was installed and configured as the first workstation in the lab:

`WS_01`

During the initial network configuration, I noticed that the IP address assigned to the client was **not within the DHCP scope configured on the Domain Controller**.

This indicated that another DHCP service was assigning an address before the Windows Server DHCP service could do so.

---

## Investigating the DHCP Conflict

The issue was eventually traced to **libvirt's DHCP service**.

The default libvirt virtual network had its own DHCP service, which was assigning an IP address to the Windows client before the DHCP service running on the Domain Controller could provide one.

To inspect the configuration of the default libvirt network, I used:

```bash
sudo virsh net-dumpxml default
```

This displays the XML configuration associated with the default libvirt network and helped identify the DHCP configuration being provided by libvirt.

!!! note 
	This troubleshooting led to the network and subnet changes documented in the previous networking section.

The lab was subsequently moved to the new subnet, with DHCP being provided by the Windows Server Domain Controller.

### Windows Client Network Path

After resolving the DHCP conflict, the intended client networking model was:
```mermaid
flowchart LR
    A[Windows Client] --> B[libvirt Network]
    B --> C[Windows Server DHCP]
    C --> D[Client Receives Lab IP Configuration]
    D --> E[Domain Services]
```
The Domain Controller could therefore provide the Windows clients with the network configuration required for the Active Directory environment.

## Joining the Clients to the Domain

The Windows clients were then added to the Active Directory domain.

The first workstation was:

WS_01

A second Windows workstation was later created:

WS_02

The same setup process used for WS_01 was followed when creating and joining WS_02.

!!! note
	WS_02 was added after the Group Policy Objects had been created.

The GPO configuration, user testing, wallpaper deployment, `gpupdate /force`, and `gpresult /r` verification are documented separately in the [Group Policy](../group-policy/overview.md) section so that the lab remains in chronological and logical order.

### Result

At this stage, the lab had Windows 11 client systems connected to the virtual network and able to participate in the Active Directory environment.

The DHCP issue encountered while creating the first client also highlighted an important aspect of virtualised networking: both libvirt and Windows Server can provide DHCP services, so care must be taken to ensure that clients receive their configuration from the intended DHCP server.

