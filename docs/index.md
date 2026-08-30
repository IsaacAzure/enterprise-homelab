# Enterprise Windows Infrastructure Lab

> A virtualised enterprise environment built to simulate real-world Windows infrastructure, Active Directory administration, Group Policy, delegated permissions, and remote management. 

!!! warning "Work in progress"
    This lab is under active development. Documentation and validation results will be updated as additional infrastructure components and administrative roles are implemented and tested.

<div class="grid cards" markdown>

* :material-domain:{ .lg .middle } **Active Directory**

  ---

  Domain configuration, DNS, DHCP, Windows clients, OU design, users, and security groups.

  [:octicons-arrow-right-24: Explore Active Directory](active-directory/domain_setup.md)

* :material-account-lock:{ .lg .middle } **Delegated Administration**

  ---

  Role-based permissions for Help Desk and SysAdmin accounts, including delegation testing and security boundaries.

  [:octicons-arrow-right-24: View Delegation](delegated-administration/overview.md)

* :material-file-cog:{ .lg .middle } **Group Policy**

  ---

  GPO configuration, WinRM, firewall rules, RSoP, Group Policy Modeling, and policy troubleshooting.

  [:octicons-arrow-right-24: Explore Group Policy](group-policy/overview.md)

* :material-console-line:{ .lg .middle } **Remote Administration**

  ---

  PowerShell Remoting, WinRM configuration, RSAT deployment, and remote Active Directory administration.

  [:octicons-arrow-right-24: View Remote Administration](remote-administration/overview.md)

</div>

---

## Lab Environment

<div class="grid cards" markdown>

* **Domain**

  `earth.local`

* **Domain Controller**

  Windows Server
  Active Directory · DNS · Group Policy

* **Endpoints**

  `WS_01`
  `WS_02`

* **Virtualisation**

  Fedora Linux
  KVM / libvirt

</div>

---

## Lab Architecture

```text
                         ┌──────────────────────┐
                         │     Fedora Linux     │
                         │                      │
                         │    KVM / libvirt     │
                         └──────────┬───────────┘
                                    │
                         192.168.150.0/24
                                    │
              ┌─────────────────────┼─────────────────────┐
              │                     │                     │
      ┌───────▼────────┐    ┌───────▼────────┐    ┌───────▼────────┐
      │ Domain         │    │ WS_01          │    │ WS_02          │
      │ Controller     │    │                │    │                │
      │                │    │ Windows Client │    │ Windows Client │
      │ Active Dir.    │    │                │    │                │
      │ DNS            │    │ Domain Joined  │    │ Domain Joined  │
      │ Group Policy   │    │                │    │                │
      └────────────────┘    └────────────────┘    └────────────────┘
```

[:material-sitemap: View Full Architecture](environment/architecture_network.md)

---

## Role-Based Administration

The environment includes separate administrative roles designed around delegated permissions and least-privilege principles.

| Capability | Help Desk | SysAdmin | Administrative SysAdmin |
| --- | :---: | :---: | :---: |
| View user information | ✓ | ✓ | — |
| Reset passwords | ✓ | ✓ | — |
| Create users | ✗ | ✗ | ✓ |
| Delete users | ✗ | ✗ | ✓ |
| Modify general user properties | ✗ | ✗ | ✓ |
| Modify group membership | ✗ | ✗ | ✓ |
| View GPO inheritance | ✗ | ✓ | — |
| Manage GPO links | ✗ | ✗ | — |
| Run RSoP | ✗ | ✓ | — |

`—` means that the Administrative SysAdmin test has not yet been completed.

[:material-account-group: Explore Administrative Roles](delegated-administration/overview.md)

---

## Testing and Validation

Each administrative role is tested independently to confirm both permitted and restricted actions.

<div class="grid cards" markdown>

* **Help Desk Testing**

  Password administration, account support and validation of restricted actions.

  [:octicons-arrow-right-24: View Tests](delegated-administration/permission-testing/helpdesk.md)

* **SysAdmin Testing**

  Password administration, permission boundaries, GPO visibility and RSoP diagnostics.

  [:octicons-arrow-right-24: View Tests](delegated-administration/permission-testing/sysadmin.md)

* **Administrative SysAdmin Testing**

  Privileged user-object and group-membership administration.

  [:octicons-arrow-right-24: View Tests](delegated-administration/permission-testing/administrative_sysadmin.md)

</div>

---

## Featured Troubleshooting

### PowerShell Remoting, WinRM and RSAT

A layered investigation across DNS, firewall rules, WinRM listeners, Kerberos authentication, remote RSAT installation, Active Directory Web Services and the PowerShell second-hop problem.

[:octicons-arrow-right-24: Read the Investigation](remote-administration/overview.md)

### RSoP Planning, DCOM and WMI

Troubleshooting Group Policy Modeling across Active Directory delegation, GPO targeting, DCOM security, WMI namespace permissions and remote activation.

[:octicons-arrow-right-24: Read the Investigation](delegated-administration/permission-testing/sysadmin.md#step-9-test-rsop-planning)

---

## Key Principles

This lab focuses on applying and testing enterprise administration concepts:

* **Least privilege**
* **Role-based access control**
* **Delegated administration**
* **Defence in depth**
* **Layered troubleshooting**
* **Configuration validation**
* **Understanding permission boundaries**

---

## Explore the Documentation

<div class="grid cards" markdown>

* [Lab Environment](environment/overview.md)
* [Active Directory](active-directory/domain_setup.md)
* [Group Policy](group-policy/overview.md)
* [Delegated Administration](delegated-administration/overview.md)
* [Permission Testing](delegated-administration/permission-testing/overview.md)
* [Remote Administration](remote-administration/overview.md)

</div>
