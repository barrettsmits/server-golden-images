# Proxmox Cloud Image Template Creator

This Ansible playbook automates the creation of VM templates in Proxmox using official cloud images from various Linux distributions. It downloads the latest cloud images, configures them with cloud-init, and creates ready-to-use templates in your Proxmox environment.

## Supported Cloud Images

- Debian 12 (Bookworm)
- Ubuntu Noble (24.04)
- Rocky Linux 9.4
- Arch Linux

## Prerequisites

- Ansible installed on your control node
- Access to a Proxmox server
- Proxmox API token with appropriate permissions
- The following Ansible collections:
  - community.general
  - ansible.builtin

## Directory Structure

```
proxmox_kvm_ansible/
├── cloud-images.yml # Main playbook file
├── tasks/
│   └── main.yml # Task definitions
├── templates/
│   └── default_cloudconfig.yml.j2 # Cloud-init template
├── vars/
│   └── vars.yml # Variable definitions
└── secrets/
    └── secrets.yml # Sensitive variables (not in repo)

```

## Configuration

### Required Variables

Create a `secrets.yml` file in the `secrets/` directory with the following variables:

```
# Proxmox credentials
pve_iaac_name: "Your Proxmox service account username"
pve_iaac_api_token_id: "Your Proxmox API token ID"
pve_iaac_token_secret: "Your Proxmox API token secret"

# Cloud-init user credentials
ci_user_name: "Username for the default cloud-init user"
ci_user_password_hash: "SHA-512 hashed password for cloud-init user" 
ssh_public_key: "Your SSH public key for cloud-init user authentication"

# Proxmox root user credentials
pve_root_name: root
pve_root_password: "Root password for Proxmox host"

```
### Additional Variables

```
# Proxmox Storage Variables
pve_iso_storage: "Storage for ISO files"
pve_snippets_storage: "Storage for snippets"
pve_template_storage: "Storage for templates" 

``` 


## Features

- Automatically downloads the latest cloud images
- Verifies checksums for security
- Configures cloud-init with custom settings
- Creates optimized VM templates with:
  - QEMU guest agent
  - Common system utilities
  - SSH key authentication
  - Sudo access for admin user
  - Network configuration

## Default Template Configuration

Each template is created with:
- 2 CPU cores
- 2GB RAM
- virtio networking
- DHCP network configuration
- Cloud-init drive
- Common system utilities pre-installed

## Security Notes

- Store sensitive information in `secrets.yml` and ensure it's not committed to version control
- Use API tokens instead of password authentication when possible
- Regularly update cloud images to get the latest security patches

## Contributing

Feel free to submit issues and pull requests for improvements or bug fixes.

## License

[MIT License](LICENSE)