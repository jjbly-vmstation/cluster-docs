Overview: Kerberos-Based Single Sign-On for Windows Automation
This project utilizes Kerberos (GSSAPI) to facilitate passwordless authentication between the Linux Ansible Control Node and the Windows Server 2025 / Windows 11 Guest environment. This eliminates the need for static credentials in inventory files and leverages the vmstation.local Domain Controller (R430) as the central Authentication Service.

1. Prerequisites
Domain Controller: Windows Server 2025 (R430) at 192.168.4.20.

Realm: VMSTATION.LOCAL (Must be uppercase).

Ansible Node Dependencies: krb5-user, libkrb5-dev, python3-pip.

2. Control Node Configuration
Install the Kerberos transport for WinRM:

Bash
pip install pywinrm[kerberos]
Configure /etc/krb5.conf to point to the R430:

Plaintext
[libdefaults]
    default_realm = VMSTATION.LOCAL

[realms]
    VMSTATION.LOCAL = {
        kdc = 192.168.4.20
        admin_server = 192.168.4.20
    }
3. Operational Workflow
Before running any playbooks, the administrator must initialize a Ticket Granting Ticket (TGT):

Bash
kinit Administrator@VMSTATION.LOCAL
This ticket is stored in memory and allows Ansible to authenticate against all domain-joined Windows hosts without a password for the duration of the ticket's lifetime (default 10 hours).

File: vmstation/cluster-setup/inventory.yml (Segment)
Add this group to your existing inventory to handle the Kerberos-enabled hosts.

YAML
    # --- KERBEROS MANAGED WINDOWS NODES ---
    windows_sso_nodes:
      hosts:
        r430_host:
          ansible_host: 192.168.4.20
        dad_vm:
          ansible_host: 192.168.4.50
      vars:
        ansible_user: Administrator@VMSTATION.LOCAL
        ansible_connection: winrm
        ansible_winrm_transport: kerberos
        ansible_winrm_server_cert_validation: ignore
        ansible_winrm_kerberos_delegation: true # Allows gMSA password retrieval