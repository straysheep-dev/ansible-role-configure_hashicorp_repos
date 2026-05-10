configure_hashicorp_repos
=========================

![molecule workflow](https://github.com/straysheep-dev/ansible-role-configure_hashicorp_repos/actions/workflows/molecule.yml/badge.svg) ![ansible-lint workflow](https://github.com/straysheep-dev/ansible-role-configure_hashicorp_repos/actions/workflows/ansible-lint.yml/badge.svg)

Configures the [HashiCorp](https://www.hashicorp.com/trust/security) software repository on Debian and RedHat family systems.

GPG key fingerprints are verified against known values before any package manager action is taken. The reusable verification task lives at `tasks/verify-signing-keys.yml` and accepts `verify_key_path` and `verify_key_fingerprints` as parameters via the include's `vars:`.

Tested on Debian family (Debian, Ubuntu) and RedHat family (Fedora, Rocky) distributions.

- https://www.hashicorp.com/trust/security
- https://developer.hashicorp.com/packer/install

Requirements
------------

- `gpg` must be available on the target host for key fingerprint verification (used by `verify-signing-keys.yml`).
- **Debian family**: `ansible.builtin.deb822_repository` requires `python3-debian` on the target.
- **RedHat family**: `ansible.builtin.yum_repository` and `ansible.builtin.rpm_key` are used; no additional dependencies beyond a standard DNF system.

Role Variables
--------------

All variables are in `defaults/main.yml`. Key download and install paths are consolidated where possible; OS-specific paths are resolved via Jinja2 conditionals in the defaults so both task files share the same variable names.

`hashicorp_signing_key_fingerprints`, list of expected GPG fingerprints. Spaces are optional; `verify-signing-keys.yml` normalizes them automatically before comparison. Should only be updated when HashiCorp publishes new signing keys and the new fingerprints have been confirmed against the official documentation.

```yaml
hashicorp_signing_key_fingerprints:
  - "798A EC65 4E5C 1542 8C8E 42EE AA16 FCBC A621 E701"
```

`hashicorp_keyring_url`, ASCII-armored signing key download URL. Used by both Debian and RedHat paths. The same key bytes are served from both `apt.releases.hashicorp.com/gpg` and `rpm.releases.hashicorp.com/gpg`.

```yaml
hashicorp_keyring_url: "https://apt.releases.hashicorp.com/gpg"
```

`hashicorp_keyring_tmp`, temporary download path for the signing key before verification and installation.

```yaml
hashicorp_keyring_tmp: "/tmp/hashicorp-archive-keyring.asc"
```

`hashicorp_keyring_path`, final install path for the signing key. Mirrors the APT keyring convention on Debian and the RPM PKI path convention on RedHat, so the repo configuration can reference a local file instead of a remote URI.

```yaml
hashicorp_keyring_path: "{{ '/etc/apt/keyrings/hashicorp-archive-keyring.asc' if ansible_facts['os_family'] == 'Debian'
                            else '/etc/pki/rpm-gpg/RPM-GPG-KEY-hashicorp' if ansible_facts['os_family'] == 'RedHat' }}"
```

`hashicorp_repo_url`, repository base URL. Used as `uris` in the apt `deb822_repository` task and as `baseurl` in the dnf `yum_repository` task. The yum macros (`$releasever`, `$basearch`) are intentional and resolved by the dnf layer at runtime.

```yaml
hashicorp_repo_url: "{{ 'https://apt.releases.hashicorp.com' if ansible_facts['os_family'] == 'Debian'
                        else 'https://rpm.releases.hashicorp.com/fedora/$releasever/$basearch/stable' if ansible_facts['distribution'] == 'Fedora'
                        else 'https://rpm.releases.hashicorp.com/RHEL/$releasever/$basearch/stable' if ansible_facts['os_family'] == 'RedHat' }}"
```

`hashicorp_pinned_packages`, list of apt packages pinned to the HashiCorp origin at priority 1001 so they win over the distro's archive. Ubuntu universe ships several of these (`packer`, `terraform`, `vagrant`); without a pin you may end up with the older distro version.

```yaml
hashicorp_pinned_packages:
  - boundary
  - consul
  - nomad
  - packer
  - terraform
  - vagrant
  - vault
  - waypoint
```

Dependencies
------------

None.

Example Playbook
----------------

```yml
- name: "Default Playbook"
  hosts: all
  roles:
    - role: configure_hashicorp_repos
```

License
-------

[MIT](./LICENSE)

Author Information
------------------

[straysheep-dev/ansible-configs](https://github.com/straysheep-dev/ansible-configs)
