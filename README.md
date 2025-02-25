# ansible-auditd-laurel

Ansible role to deploy Auditd and Laurel plugin. By default the role will fetch the latest binary from [GitHub](https://github.com/threathunters-io/laurel/blob/master/INSTALL.md#or-use-one-of-the-provided-binaries) but has also the option to build it from source.

LAUREL is an event post-processing plugin for auditd(8) to improve its usability in modern security monitoring setups.

* https://github.com/threathunters-io/laurel

# Supported Operating Systems

- Debian 12
- Debian 11
- Debian 10
- RHEL / Rocky Linux 9
- RHEL / Rocky Linux 8

# Default Variables

The defaults variables are the following:

- `laurel_user` - This user user will run Laurel plugin
  - Default is `_laurel`
- `laurel_build_dir` - The directory where the build from sources takes place
  - Default is `/var/_install/laurel`
- `laurel_user_allowed_read` - Defines the user that will have the right to read the laurel's logs. The role will fail if the user does not exists
  - Default is `splunkfwd`
- `laurel_local_tmp` - Ansible fetches the latest release from GitHub on the Ansible Controller (locally)
  - Default is `/tmp`
- `laurel_version` - The version of the Laurel to be installed
  - Default is `latest`

# Usage

You can call this role with the following play:

```
---
- hosts: laurel
  become: yes
  roles:
    - role: ansible-auditd-laurel
```

You can override the following variable as well, with the following play:

```
---
- hosts: laurel
  become: yes
  roles:
    - role: ansible-auditd-laurel
      vars:
        laurel_user: _user53
        laurel_build_dir: /var/_install67/laurel
        laurel_user_allowed_read: filebeat
        laurel_local_tmp: /tmp
        laurel_version: latest
```

## Default: Retrieve last release from GitHub

By default, the role will fetch the latest Laurel release from GitHub. The Ansible Controller will fetch it once in the directory `laurel_local_tmp` and will copy the binaries in `laurel_build_dir`.

Please checkout the [network](#network) section to know more about the URLs reached.

Note: when using the default mode, there is no need to add any specific tag

## Build it from source

This role has the ability to build Laurel from source, to do so use the tags as the following:

```sh
ansible-playbook ... --tags build --skip-tags binary
```

# Network

- Default mode:
  - `https://api.github.com/`
  - `https://github.com/`

- Build mode:
  - `https://github.com/`
  - `https://sh.rustup.rs`

# Configuration Maintenance

This role allows you to push Auditd and/or Laurel configuration files and avoid to go through the whole installation process. It allows you to maintain the configuration files. The following steps describe how to proceed:

1. Modify the configuration file of Auditd (`templates/auditd.rules.j2`) or Laurel's configuration file (`templates/config.toml.j2`);
2. Launch your playbook with the tag `config` (`$ ansible-playbook laurel.yml -t config`).

# Auditd Configuration

We are providing an Auditd's configuration copy from Florian Roth's repo (https://github.com/Neo23x0/auditd).

You can replace this configuration with your own by editing the file [auditd.rules.j2](./files/auditd.rules).

# Laurel Configuration

The configuration provided for Laurel set the command line arguments concatenated into a single string by default.

You can edit the file [config.toml.j2](./templates/config.toml.j2) and replace the configuration with your own.

# Laurel Logs

Laurel's logs will be available in `/var/log/laurel/audit.log` in JSON format. Laurel will automatically rotate the logs as per the [settings](https://github.com/certeu/ansible-auditd-laurel/blob/main/templates/config.toml.j2#L14).

# Known Issues when building from source

Ansible will first try to install `cargo` (and `rustc` for the compilation) from the default distribution repositories. If the version of Rust is too old it won't reach the 2021 edition for the earliest update. If the build fails, Ansible will remove the `cargo` and `rustc` from the machine and will reach `sh.rustup.rs` and `static.rust-lang.org` to install the earliest version of Rust. This has been experienced using Debian 10 and 11.

# License

Licensed under the GPL.