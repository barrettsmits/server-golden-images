# Cloud_Init Ansible Role

This role downloads and configures the current Noble (version codename) Ubuntu cloud image and creates a Proxmox VM template with cloud-init support. It handles the download, customization, and template creation process automatically.

Requirements
------------

- Ansible 2.9 or higher
- Proxmox VE server
- libguestfs-tools (for virt-customize)
- Sufficient storage space in /tmp and target Proxmox storage
- Network connectivity to cloud-images.ubuntu.com

Role Variables
--------------

The following variables can be configured (defaults shown):

## Connection Settings:
  
  ### defaults/main.yml
```
---
# defaults file for cloud_init

# Connection settings
cloud_init_api_host: "proxmox.example.com"  # Your Proxmox host
cloud_init_api_user: "root@pam"             # Proxmox API user
cloud_init_api_token_id: ""                 # Your API token ID
cloud_init_api_token_secret: ""             # Your API token secret
cloud_init_force_download: false            # Force redownload of cloud image
cloud_init_public_keys: ""                  # Your SSH public key
cloud_init_ci_password: ""                  # Cloud-init user password
cloud_init_node: proxmox                    # Proxmox node name

# Image settings
cloud_init_img_target_path: /tmp
cloud_init_img_version: noble               # Ubuntu version codename
cloud_init_version_number: 24.04.1          # Ubuntu version number
cloud_init_img_target_name: "{{ cloud_init_img_version }}-server-cloudimg-amd64"
cloud_init_img_source_url: "https://cloud-images.ubuntu.com/{{ cloud_init_img_version }}/current/{{ cloud_init_img_target_name }}.img"
cloud_init_img_file_path: /tmp
cloud_init_img_file_name: "{{ cloud_init_img_version }}-ubuntu-{{ cloud_init_version_number }}"

# Remote settings
cloud_init_remote_user: root
cloud_init_img_remote_path: /tmp

# VM settings
cloud_init_user_data_file: vendor.yaml
cloud_init_storage_target: local             # Your storage pool name
cloud_init_disksize: 20G
cloud_init_nameservers:
  - 1.1.1.1                                 # Example nameserver
  - 8.8.8.8                                 # Example nameserver
  - 8.8.4.4                                 # Example nameserver
cloud_init_template_vmid: 9000              # Template VM ID
cloud_init_template_name: "ubuntu-{{ cloud_init_version_number }}-template"

# Cloud-init User
cloud_init_ciuser: ubuntu                   # Default cloud-init username
cloud_init_cipassword: ""                   # Set your cloud-init user password
cloud_init_sshkeys: ""                      # Your SSH public key
```

## Dependencies
------------

This role has no external dependencies, but requires:
- `community.general.proxmox_kvm` module
- Valid Proxmox API credentials
- Proper network connectivity to Proxmox host
- `libguestfs-tools` installed on the local machine running the playbook
- Access to ubuntu cloud images (https://cloud-images.ubuntu.com/)

## Example Playbook
----------------

```
- name: Download current Ubuntu Cloud Image
  hosts: localhost
  gather_facts: true
  become_user: user_name                                 # Change this to match the user running the playbook.
  vars_files:
    - ~/secrets/creds.yml                                # Ansible vault file containing secrets
  vars:
    ansible_become_pass: "{{ become_password }}"         # Become password from vault
    cloud_init_force_download: false                     # Force redownload of cloud image
    cloud_init_img_version: noble                        # Ubuntu version codename
    cloud_init_template_name: ubuntu-24.04.1-template    # Template name

  tasks:
    - include_role:
        name: cloud_init
        tasks_from: localhost-tasks.yml
  tags:
    - download

```
and

```
- name: Proxmox tasks
  hosts: proxmox_prd
  gather_facts: true
  become: true
  vars_files:
    - ~/secrets/creds.yml                                                   # Ansible Vault file
  vars:
    ansible_become_pass: "{{ become_password }}"                            # Become password from vault 
    var_api_host: proxmox.example.com                                       # Your Proxmox host FQDN
    var_api_user: root@pam                                                  # Proxmox API user
    var_api_token_id: "{{ vault_api_token_id }}"                            # Proxmox API token ID
    var_api_token_secret: "{{ vault_api_token_secret }}"                    # Proxmox API token secret
    cloud_init_template_name: ubuntu-24.04.1-template                       # Template name
    
    # Cloud-init User
    cloud_init_ciuser: ubuntu                                               # Default cloud-init username           
    
    # VM setttings   
    cloud_init_storage_target: local-lvm                                    # Your Proxmox storage target
    cloud_init_disksize: 32G                                                # Template disk size
    cloud_init_template_vmid: 9000                                          # Template VM ID
  tasks:
    - include_role:
        name: cloud_init
        tasks_from: proxmox-tasks.yml
  tags:
    - create
```

## Features
--------

- Downloads latest Ubuntu cloud image based on the codename provided in the role variables.
- Customizes image with qemu-guest-agent and Python 3.
- Creates Proxmox VM template with cloud-init support.
- Configures network, SSH keys, and basic system settings.
- Includes custom cloud-init configuration via vendor.yaml.

## License
-------

MIT

## Author Information
------------------
Created for homelab infrastructure management. For questions or issues, please open a GitHub issue.