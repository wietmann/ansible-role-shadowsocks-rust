Ansible Role: Shadowsocks-Rust
==============================

Ansible Role that installs and manages [shadowsocks-rust](https://github.com/shadowsocks/shadowsocks-rust) on Linux hosts.

Role downloads defined version of `shadowsocks-rust` from release page on GitHub, sets up process environment (create process user and group, add systemd unit files, add configuration files) and manages process instances via systemd. Currently `sslocal` and `ssserver` instanced service units are present.

By default, Musl build for x86_64 is installed.

Requirements
------------

Python, Ansible >= 2.9

Role Variables
--------------

|Variable|Deafule value|Description|
|-|-|-|
|`shadowsocks_rust_version`|`1.12.4`|Shadowsocks-rust version|
|`shadowsocks_rust_bin_path`|`/usr/local/bin`|Path to directory where binaries will reside|
|`shadowsocks_rust_linux_libc`|`musl`|Choose LibC version for Linux binary. See release page for specific version for available values|
|`shadowsocks_rust_config_path`|`/etc/shadowsocks-rust`|Path to Shadowsocks-rust configuration files directory|
|`shadowsocks_rust_user`|`shadowsocks-rust`|System user to run shadowsocks-rust process with|
|`shadowsocks_rust_group`|`shadowsocks-rust`|System group to run shadowsocks-rust process with|
|`shadowsocks_rust_config`|`[]`|List of Shadowsocks instances configurations|
|`shadowsocks_rust_config[*].name`|-|Instance name (will be used as name for in systemd service after `@`)|
|`shadowsocks_rust_config[*]state`|`present`|Whether to create or destroy instance. Available values: `present`, `absent`|
|`shadowsocks_rust_config[*].type`|`server`|Instance type: `server`, `local`|
|`shadowsocks_rust_config[*].config`|-|Dictionary representing Shadowsocks config file structure. See shadowsocks-rust documentation for available values.|

Dependencies
------------

None.

Supported platforms
-------------------

* RHEL 7, 8
* Debian: 9, 10, 11
* Ubuntu: 18.04, 20.04
* Amazon Linux 2

**NB1**: Role should work on non-LTS Ubuntu releases, although not tested specifically.

Example Playbook
----------------

Setup 2 different shadowsocks servers on one host:

```yaml
---
- hosts: all
  become: yes

  vars:
    shadowsocks_rust_config:
      - name: server_home
        state: present
        type: server
        config:
          server: "0.0.0.0"
          server_port: 12345
          password: "VerySecretPassword1"
          timeout: 60
          method: "aes-256-gcm"
          fast_open: true
      - name: server_friends
        state: present
        type: server
        config:
          server: "0.0.0.0"
          server_port: 12346
          password: "VerySecretPassword2"
          timeout: 60
          method: "aes-256-gcm"
          fast_open: true
    _packages:
      "Debian":
        - xz-utils
        - zip
        - tar
      "RedHat":
        - xz
        - zip
        - tar

  pre_tasks:
    - name: Install dependencies
      package:
        name: "{{ _packages[ansible_os_family] }}"
        state: present

  roles:
    - role: wietmann.shadowsocks_rust
```

Client (`sslocal`)
```yaml
---
- hosts: all
  become: yes

  vars:
    shadowsocks_rust_config:
      - name: nl_vps1_home
        state: present
        type: local
        config:
          server: "somewhere.tld"
          server_port: 12345
          password: "VerySecretPassword1"
          method: "aes-256-gcm"
          local_port: 1081
    _packages:
      "Debian":
        - xz-utils
        - zip
        - tar
      "RedHat":
        - xz
        - zip
        - tar

  pre_tasks:
    - name: Install dependencies
      package:
        name: "{{ _packages[ansible_os_family] }}"
        state: present

  roles:
    - role: wietmann.shadowsocks_rust
```

Testing
-------

This role uses [Molecule](https://molecule.readthedocs.io/en/stable/) for testing. Molecule is run against Docker containers with enabled systemd support (see `default` scenario). Docker Engine, `docker` library and `molecule-docker` driver Python packages should be installed on controller host.

Preferred way to test role for compatibility with different versions of Ansible is to use [Tox](https://tox.wiki/en/stable/). See `tox.ini` for the full list of supported Ansible versions.

Run tests for all environments:
```
tox
```

List available environments:
```
tox -l
```

Run tests only for specific environment (e.g. Ansible 2.9):
```
tox -e py3-ansible29
```

Run only specific Molecule subcommand (e.g. `molecule verify`):

```
# All test environments
tox -- molecule verify
# Specific test environments
tox -e py3-ansible29 -- molecule verify
```

License
-------

SPDX:MIT  
See full text in [LICENSE](LICENSE) file.

Author Information
------------------

This role was created in 2021 by Dmitry Danilov.
