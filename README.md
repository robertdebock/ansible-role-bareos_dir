# [bareos_dir](#bareos_dir)

Install and configure [BareOS](https://www.bareos.com/) Director.

|GitHub|GitLab|Quality|Downloads|Version|
|------|------|-------|---------|-------|
|[![github](https://github.com/robertdebock/ansible-role-bareos_dir/workflows/Ansible%20Molecule/badge.svg)](https://github.com/robertdebock/ansible-role-bareos_dir/actions)|[![gitlab](https://gitlab.com/robertdebock-iac/ansible-role-bareos_dir/badges/master/pipeline.svg)](https://gitlab.com/robertdebock-iac/ansible-role-bareos_dir)|[![quality](https://img.shields.io/ansible/quality/63105)](https://galaxy.ansible.com/robertdebock/bareos_dir)|[![downloads](https://img.shields.io/ansible/role/d/63105)](https://galaxy.ansible.com/robertdebock/bareos_dir)|[![Version](https://img.shields.io/github/release/robertdebock/ansible-role-bareos_dir.svg)](https://github.com/robertdebock/ansible-role-bareos_dir/releases/)|

## [Example Playbook](#example-playbook)

This example is taken from [`molecule/default/converge.yml`](https://github.com/robertdebock/ansible-role-bareos_dir/blob/master/molecule/default/converge.yml) and is tested on each push, pull request and release.

```yaml
---
- name: Converge
  hosts: all
  become: yes
  gather_facts: yes

  roles:
    - role: robertdebock.bareos_dir
      bareos_dir_clients:
        - name: client1
          address: 127.0.0.1
          password: "MySecretPassword"
        - name: "disabled-client"
          enabled: no
      bareos_dir_storages:
        - name: File-1
          address: dir-1
          password: "MySecretPassword"
          device: FileStorage
          media_type: File
          tls_enable: yes
          tls_verify_peer: no
        - name: "disabled-storage"
          enabled: no
      bareos_dir_messages:
        - name: "Standard"
          description: "Send relevant messages to the Director."
          director:
            server: bareos-dir
            messages:
              - all
              - "!skipped"
              - "!restored"
          append:
            file: "/var/log/bareos/bareos.log"
            messages:
              - all
              - "!skipped"
              - "!terminate"
          console:
            - all
            - "!skipped"
            - "!saved"
        - name: "disabled-message"
          enabled: no
      bareos_dir_profiles:
        - name: webui-admin
          jobacl:
            - "*all*"
          clientacl:
            - "*all*"
          storageacl:
            - "*all*"
          scheduleacl:
            - "*all*"
          poolacl:
            - "*all*"
          commandacl:
            - "!.bvfs_clear_cache"
            - "!.exit"
            - "!.sql"
            - "!configure"
            - "!create"
            - "!delete"
            - "!purge"
            - "!prune"
            - "!sqlquery"
            - "!umount"
            - "!unmount"
            - "*all*"
          filesetacl:
            - "*all*"
          catalogacl:
            - "*all*"
          whereacl:
            - "*all*"
          pluginoptionsacl:
            - "*all*"
        - name: "disabled-message"
          enabled: no
      bareos_dir_jobdefs:
        - name: DefaultJob-1
          type: Backup
          level: Incremental
          fileset: SelfTest
          schedule: WeeklyCycle
          storage: File-1
          messages: Standard
          pool: Full
          priority: 10
          write_bootstrap: "/var/lib/bareos/%c.bsr"
          full_backup_pool: Full
          differential_backup_pool: Differential
          incremental_backup_pool: Incremental
        - name: "disabled-jobdef"
          enabled: no
      bareos_dir_jobs:
        - name: my_job
          description: "My backup job"
          pool: Full
          type: Backup
          client: client1
          fileset: LinuxAll
          storage: File-1
          messages: Standard
        - name: disabled_job
          enabled: no
      bareos_dir_pools:
        - name: Full
          pool_type: Backup
          recycle: yes
          autoprune: yes
          volume_retention: 365 days
          maximum_volume_bytes: 50G
          maximum_volumes: 100
          label_format: "Full-"
        - name: "disabled-pool"
          enabled: no
      bareos_dir_filesets:
        - name: LinuxAll
          description: "Backup all regular filesystems, determined by filesystem type."
          include:
            files:
              - /
            options:
              signature: MD5
              one_fs: no
              fs_types:
                - btrfs
                - ext2
                - ext3
                - ext4
                - reiserfs
                - jfs
                - vfat
                - xfs
                - zfs
          exclude:
            files:
              - /var/lib/bareos
              - /var/lib/bareos/storage
              - /proc
              - /tmp
              - /var/tmp
              - /.journal
              - /.fsck
        - name: disabled-fileset
          enabled: no
```

The machine needs to be prepared. In CI this is done using [`molecule/default/prepare.yml`](https://github.com/robertdebock/ansible-role-bareos_dir/blob/master/molecule/default/prepare.yml):

```yaml
---
- name: Prepare
  hosts: all
  become: yes
  gather_facts: no

  roles:
    - role: robertdebock.bootstrap
    # The roles buildtools, python_pip and postgres are required.
    # bareos-dir needs to connect to a database.
    - role: robertdebock.buildtools
    - role: robertdebock.python_pip
    - role: robertdebock.postgres
    # The roles core_dependencies and postfix are  required for the `bareos_role`: "dir".
    # bareos-dir needs to send emails.
    # - role: robertdebock.core_dependencies
    # - role: robertdebock.postfix
    - role: robertdebock.bareos_repository
```

Also see a [full explanation and example](https://robertdebock.nl/how-to-use-these-roles.html) on how to use these roles.

## [Role Variables](#role-variables)

The default values for the variables are set in [`defaults/main.yml`](https://github.com/robertdebock/ansible-role-bareos_dir/blob/master/defaults/main.yml):

```yaml
---
# defaults file for bareos_dir

# The director has these configuration parameters.
bareos_dir_hostname: "{{ inventory_hostname }}"
bareos_dir_password: "secretpassword"
bareos_dir_queryfile: "/usr/lib/bareos/scripts/query.sql"
bareos_dir_max_concurrent_jobs: 10
bareos_dir_message: Daemon
bareos_dir_tls_enable: yes
bareos_dir_tls_verify_peer: no

bareos_dir_clients: []
bareos_dir_storages: []
bareos_dir_messages: []
bareos_dir_profiles: []
bareos_dir_pools: []
bareos_dir_filesets: []
bareos_dir_jobs: []
bareos_dir_jobdefs: []
```

## [Requirements](#requirements)

- pip packages listed in [requirements.txt](https://github.com/robertdebock/ansible-role-bareos_dir/blob/master/requirements.txt).

## [State of used roles](#state-of-used-roles)

The following roles are used to prepare a system. You can prepare your system in another way.

| Requirement | GitHub | GitLab |
|-------------|--------|--------|
|[robertdebock.bootstrap](https://galaxy.ansible.com/robertdebock/bootstrap)|[![Build Status GitHub](https://github.com/robertdebock/ansible-role-bootstrap/workflows/Ansible%20Molecule/badge.svg)](https://github.com/robertdebock/ansible-role-bootstrap/actions)|[![Build Status GitLab](https://gitlab.com/robertdebock-iac/ansible-role-bootstrap/badges/master/pipeline.svg)](https://gitlab.com/robertdebock-iac/ansible-role-bootstrap)|
|[robertdebock.bareos_repository](https://galaxy.ansible.com/robertdebock/bareos_repository)|[![Build Status GitHub](https://github.com/robertdebock/ansible-role-bareos_repository/workflows/Ansible%20Molecule/badge.svg)](https://github.com/robertdebock/ansible-role-bareos_repository/actions)|[![Build Status GitLab](https://gitlab.com/robertdebock-iac/ansible-role-bareos_repository/badges/master/pipeline.svg)](https://gitlab.com/robertdebock-iac/ansible-role-bareos_repository)|
|[robertdebock.buildtools](https://galaxy.ansible.com/robertdebock/buildtools)|[![Build Status GitHub](https://github.com/robertdebock/ansible-role-buildtools/workflows/Ansible%20Molecule/badge.svg)](https://github.com/robertdebock/ansible-role-buildtools/actions)|[![Build Status GitLab](https://gitlab.com/robertdebock-iac/ansible-role-buildtools/badges/master/pipeline.svg)](https://gitlab.com/robertdebock-iac/ansible-role-buildtools)|
|[robertdebock.python_pip](https://galaxy.ansible.com/robertdebock/python_pip)|[![Build Status GitHub](https://github.com/robertdebock/ansible-role-python_pip/workflows/Ansible%20Molecule/badge.svg)](https://github.com/robertdebock/ansible-role-python_pip/actions)|[![Build Status GitLab](https://gitlab.com/robertdebock-iac/ansible-role-python_pip/badges/master/pipeline.svg)](https://gitlab.com/robertdebock-iac/ansible-role-python_pip)|
|[robertdebock.postgres](https://galaxy.ansible.com/robertdebock/postgres)|[![Build Status GitHub](https://github.com/robertdebock/ansible-role-postgres/workflows/Ansible%20Molecule/badge.svg)](https://github.com/robertdebock/ansible-role-postgres/actions)|[![Build Status GitLab](https://gitlab.com/robertdebock-iac/ansible-role-postgres/badges/master/pipeline.svg)](https://gitlab.com/robertdebock-iac/ansible-role-postgres)|

## [Context](#context)

This role is a part of many compatible roles. Have a look at [the documentation of these roles](https://robertdebock.nl/) for further information.

Here is an overview of related roles:
![dependencies](https://raw.githubusercontent.com/robertdebock/ansible-role-bareos_dir/png/requirements.png "Dependencies")

## [Compatibility](#compatibility)

This role has been tested on these [container images](https://hub.docker.com/u/robertdebock):

|container|tags|
|---------|----|
|[Debian](https://hub.docker.com/repository/docker/robertdebock/debian/general)|all|
|[EL](https://hub.docker.com/repository/docker/robertdebock/enterpriselinux/general)|8, 9|
|[Fedora](https://hub.docker.com/repository/docker/robertdebock/fedora/general)|37|
|[opensuse](https://hub.docker.com/repository/docker/robertdebock/opensuse/general)|all|
|[Ubuntu](https://hub.docker.com/repository/docker/robertdebock/ubuntu/general)|jammy|

The minimum version of Ansible required is 2.12, tests have been done to:

- The previous version.
- The current version.
- The development version.

If you find issues, please register them in [GitHub](https://github.com/robertdebock/ansible-role-bareos_dir/issues)

## [License](#license)

[Apache-2.0](https://github.com/robertdebock/ansible-role-bareos_dir/blob/master/LICENSE).

## [Author Information](#author-information)

[robertdebock](https://robertdebock.nl/)

Please consider [sponsoring me](https://github.com/sponsors/robertdebock).
