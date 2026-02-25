# YugabyteDB SSL Cluster Ansible Playbook

This folder contains a complete Ansible setup to install YugabyteDB `2.23.0.0` on Ubuntu 22 and bootstrap a 3-node cluster with TLS enabled for:

- node-to-node encryption
- client-to-server encryption

## File layout

```text
ansible/
├── group_vars/
│   └── all.yml
├── install_yugabyte_ssl.yml
├── inventory.ini
└── README.md
```

## What each file does

- `inventory.ini`: Node list and SSH user values.
- `group_vars/all.yml`: Version, paths, cert settings, and Yugabyte flags.
- `install_yugabyte_ssl.yml`: Installs Yugabyte, creates certs, distributes certs, starts secure cluster, checks status.

## How to test/run

1. Update `inventory.ini` with real hosts and SSH user values.
2. Optionally adjust `group_vars/all.yml` (version, base paths, flags).
3. Validate syntax:

```bash
ansible-playbook --syntax-check -i ansible/inventory.ini ansible/install_yugabyte_ssl.yml
```

4. Optional dry run:

```bash
ansible-playbook --check -i ansible/inventory.ini ansible/install_yugabyte_ssl.yml
```

5. Run for real:

```bash
ansible-playbook -i ansible/inventory.ini ansible/install_yugabyte_ssl.yml
```

## Notes

- Certificates are generated on the master via `yugabyted cert generate_server_certs`.
- Certificate archive is copied to all nodes and extracted into `{{ yb_certs_dir }}`.
- Master is started first; replicas then join via `--join={{ master_ip }}`.
