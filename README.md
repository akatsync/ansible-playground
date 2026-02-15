# Ansible Playground Notes

This repo is a small Ansible lab to install Docker on hosts in the `vm` group.

## Current repository structure

```text
ansible_playground/
├── .gitignore
├── .vault_pass                 # local vault password file (ignored by git)
├── README.md
├── ansible.cfg
├── inventory.ini
├── install_docker.yml
└── group_vars/
    └── vm/
        ├── vars.yml            # non-secret vars for group [vm]
        └── vault.yml           # encrypted secrets (Ansible Vault)
```

## What each file does

- `ansible.cfg`: project defaults (`inventory`, `become`, output format).
- `inventory.ini`: target hosts and connection vars.
- `install_docker.yml`: installs Docker and enables service.
- `group_vars/vm/vars.yml`: normal/shared variables for `vm`.
- `group_vars/vm/vault.yml`: secret variables for `vm` (encrypted).

## Quick start runbook

1. Update inventory:

```ini
[vm]
vm1 ansible_host=192.168.1.14 ansible_user=mio
```

2. Add/edit secrets in vault:

```bash
ansible-vault edit group_vars/vm/vault.yml
```

Example secrets inside vault:

```yaml
ansible_password: "YOUR_VM_PASSWORD"
ansible_become_password: "YOUR_SUDO_PASSWORD"
```

3. Run connectivity test:

```bash
ansible vm -m ping --ask-vault-pass
```

Or with a local vault password file:

```bash
printf '%s' 'your_vault_password_here' > .vault_pass
chmod 600 .vault_pass
ansible vm -m ping --vault-password-file .vault_pass
```

4. Run playbook:

```bash
ansible-playbook install_docker.yml --ask-vault-pass
```

Or with password file:

```bash
ansible-playbook install_docker.yml --vault-password-file .vault_pass
```

## Useful commands (daily)

```bash
# sanity checks
ansible --version
ansible-config dump --only-changed
ansible-inventory --graph

# inspect host facts
ansible vm -m setup --ask-vault-pass

# dry run + diff
ansible-playbook install_docker.yml --check --diff --ask-vault-pass

# verbose troubleshooting
ansible-playbook install_docker.yml -vvv --ask-vault-pass
```

## Variable concepts (practical)

- `group_vars/<group>/...` applies to hosts in that inventory group.
- For this repo, `group_vars/vm/*` loads automatically for `[vm]` hosts.
- Keep non-secrets in `vars.yml`; keep credentials in `vault.yml`.
- Same variable name in multiple places: higher precedence wins.
- Extra vars (`-e key=value`) override most other variable sources.

## Ansible Vault notes

### Create a new encrypted file

```bash
ansible-vault create group_vars/vm/vault.yml
```

### Edit encrypted file

```bash
ansible-vault edit group_vars/vm/vault.yml
```

### Encrypt existing plaintext file

```bash
ansible-vault encrypt group_vars/vm/vault.yml
```

### View/decrypt for reading

```bash
ansible-vault view group_vars/vm/vault.yml
```

### Decrypt back to plaintext (usually avoid in shared repos)

```bash
ansible-vault decrypt group_vars/vm/vault.yml
```

### Re-encrypt with a new vault password

```bash
ansible-vault rekey group_vars/vm/vault.yml
```

### Use password prompt vs password file

```bash
# prompt
ansible-playbook install_docker.yml --ask-vault-pass

# local password file
chmod 600 .vault_pass
ansible-playbook install_docker.yml --vault-password-file .vault_pass
```

## Git and secrets

- Encrypted vault files can be committed safely if vault password hygiene is good.
- `.vault_pass` must stay local only (already ignored in `.gitignore`).
- Never commit plaintext passwords to `inventory.ini`, `vars.yml`, or playbooks.

## Common issue

If you use password-based SSH and see:

`to use the 'ssh' connection type with passwords ... install the sshpass program`

Install `sshpass` on your Ansible control machine, or switch to SSH key auth.
