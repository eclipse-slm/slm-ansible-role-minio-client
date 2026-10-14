# Ansible Role: MinIO Client

[![License: MIT-0](https://img.shields.io/badge/License-MIT--0-blue.svg)](https://spdx.org/licenses/MIT-0.html)

An Ansible role for installing, configuring, and managing MinIO Client (mc) and MinIO users.

## Features

- Install MinIO Client (mc) binary
- Configure MinIO server aliases
- Create MinIO users with service accounts
- Manage user policies and permissions
- Uninstall MinIO Client

## Requirements

- Ansible 2.2 or higher
- Access to MinIO server API endpoint
- MinIO root credentials (root user and password)

## Role Variables

### Default Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `mc_bin_path` | `/usr/local/bin/mc` | Path where the MinIO Client binary will be installed |
| `mc_version` | `RELEASE.2025-08-13T08-35-41Z` | mc release to install. The last open-source release; see the note below. |
| `mc_arch` | derived from `ansible_architecture` | `linux-amd64` or `linux-arm64`. Falls back to `linux-amd64` when facts are not gathered. |
| `mc_download_url` | GitHub release asset for `mc_version`/`mc_arch` | Full download URL, override to install from a mirror |
| `mc_sha256` | checksums for the pinned release | Map of `mc_arch` to the expected SHA256. Set to `{}` to skip verification when overriding `mc_version`. |

> **Note on the download source:** MinIO archived the open-source MinIO Client
> project and `dl.min.io` now returns HTTP 410 for every download path, so this
> role installs mc from the GitHub release assets of the last open-source
> release and verifies the download against a pinned checksum.
> Point `mc_download_url` at your own mirror if you do not want to depend on
> the availability of an archived GitHub repository.

### Required Variables

These variables must be provided when using the role:

| Variable | Description |
|----------|-------------|
| `minio.url.api.internal` | MinIO server API URL (e.g., `http://localhost:9000`) |
| `_minio_root_user` | MinIO root username |
| `_minio_root_password` | MinIO root password |

### Optional Variables

| Variable | Description |
|----------|-------------|
| `_alias` | Alias name for the MinIO server (default: `slm`) |

### Task-Specific Variables

#### add_user.yml

| Variable | Description |
|----------|-------------|
| `_user` | Username to create |
| `_password` | User password |
| `_access_key` | Service account access key |
| `_secret_key` | Service account secret key |

## Dependencies

This role has no external dependencies.

## Tasks

### install_mc.yml
Downloads and installs the MinIO Client binary from `mc_download_url`, verifying it against `mc_sha256`.

### uninstall_mc.yml
Removes the MinIO Client binary and configuration data.

### add_user.yml
Creates a new MinIO user with the following steps:
1. Installs MinIO Client if not present
2. Adds the user to MinIO
3. Attaches the `readwrite` policy (if not already attached)
4. Creates a service account for the user

### get_policies_attached_to_user.yml
Retrieves and sets facts about policies attached to a specific user.

## Example Playbook

### Basic Installation

```yaml
- hosts: servers
  become: true
  vars:
    minio:
      url:
        api:
          internal: "http://minio.example.com:9000"
    _minio_root_user: "admin"
    _minio_root_password: "your-root-password"
    
  roles:
    - role: eclipse-slm.minio_client
```

### Create a MinIO User

```yaml
- hosts: servers
  become: true
  vars:
    minio:
      url:
        api:
          internal: "http://minio.example.com:9000"
    _minio_root_user: "admin"
    _minio_root_password: "your-root-password"
    
  tasks:
    - name: Create MinIO user
      ansible.builtin.include_role:
        name: eclipse-slm.minio_client
        tasks_from: add_user
      vars:
        _alias: "slm"
        _user: "myuser"
        _password: "mypassword"
        _access_key: "myaccesskey"
        _secret_key: "mysecretkey"
```

### Uninstall MinIO Client

```yaml
- hosts: servers
  become: true
  vars:
    minio:
      url:
        api:
          internal: "http://minio.example.com:9000"
    _minio_root_user: "admin"
    _minio_root_password: "your-root-password"
    
  tasks:
    - name: Uninstall MinIO Client
      ansible.builtin.include_role:
        name: eclipse-slm.minio_client
        tasks_from: uninstall_mc
```

## Testing

This role uses Molecule for testing. To run tests:

```bash
cd molecule/default
molecule test
```

## License

MIT-0

## Author Information

- **Author**: Benjamin Gotz
- **Company**: Fraunhofer IPA
- **Namespace**: eclipse-slm
