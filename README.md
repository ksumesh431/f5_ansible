# f5_ansible — Sample automation for F5 BIG-IP (certs, pools, members)

Description
This repository contains sample Ansible code and reference playbooks to automate common F5 BIG-IP tasks such as importing/uploading certificates, creating pools, and adding pool members. Use this as a starting point / reference for building repeatable F5 automation workflows.

Highlights
- Example playbooks for:
  - Importing TLS certificates and keys
  - Creating pools and adding members
  - Creating virtual servers (example)
- Inventory and variable patterns (provider block / vaulted secrets)
- Notes on prerequisites, running playbooks, and troubleshooting

Maintainer
- ksumesh431 (they/them)

Prerequisites
- Ansible (recommended ansible-core 2.12+)
- f5networks Ansible collection (install via ansible-galaxy; example below)
- Network access to the BIG-IP management IP with an account that can make configuration changes
- Optionally: Ansible Vault for storing credentials

Recommended collection and install
- Install the F5 collection (adjust collection name/version to your environment):
  ansible-galaxy collection install f5networks.f5_collection

Repository layout (example)
- playbooks/
  - import_cert.yml
  - create_pool.yml
  - add_pool_members.yml
  - site.yml
- inventories/
  - hosts.yml
- group_vars/
  - bigip.yml
- roles/ (optional — expand into roles as needed)
- README.md

Provider / credentials pattern
Ansible F5 modules commonly accept a provider dict. Store sensitive values in vaulted files or environment variables.

Example group_vars/bigip.yml
f5_provider:
  server: "{{ f5_host }}"
  user: "{{ f5_user }}"
  password: "{{ f5_pass }}"
  server_port: 443
  validate_certs: false
f5_partition: Common

Example vaulted vars (vault.yml — DO NOT COMMIT)
f5_host: 10.0.0.10
f5_user: admin
f5_pass: !vault |
  $ANSIBLE_VAULT;1.1;AES256
  ...

Example playbooks

1) Import a certificate (PEM cert + key)
playbooks/import_cert.yml
- name: Import TLS certificate and key to BIG-IP
  hosts: bigip
  gather_facts: no
  collections:
    - f5networks.f5_collection
  vars_files:
    - ../group_vars/bigip.yml
  tasks:
    - name: Import certificate (PEM)
      # adjust module name to match installed collection; example using bigip_certificate*
      f5networks.f5_collection.bigip_certificate:
        provider: "{{ f5_provider }}"
        name: "example-cert"
        partition: "{{ f5_partition }}"
        certificate: "{{ lookup('file', 'files/example.crt') }}"
        private_key: "{{ lookup('file', 'files/example.key') }}"
        state: present

