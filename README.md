# Bacula Community Collection for Ansible - bacula.bacula_community

An Ansible Collection of roles to install Bacula Community - PostgreSQL/MySQL/MariaDB Catalog, Bacula File Daemon only and Bacula Storage Daemon only in RHEL/CentOS/Debian/Ubuntu/OracleLinux.

## Ansible version compatibility

This collection has been tested against following Ansible versions: **>=2.9,<2.10**.

## Installing the Collection

Before using the Bacula Community collection, you need to install it with the Ansible Galaxy CLI:

    ansible-galaxy collection install bacula.bacula_community

## Roles

Role | Description
-------- | ---------------------
bce | install Bacula Community - PostgreSQL Catalog. This role may install PostgreSQL required packages if not present in the host.
bce_mysql | install Bacula Community - MySQL Catalog. This role may install MySQL required packages if not present in the host.
bce_mariadb | install Bacula Community - MariaDB Catalog. This role may install MariaDB required packages if not present in the host.
bce_fdonly | install Bacula File Daemon only. This is a Bacula Client.
bce_sdonly | Install a Bacula Storage. This role follows the recommended procedure to install Bacula Community and stop/disable the Director.

### Roles Variables

The roles have most of the variables defined in the `vars` directory or in the `default/main.yml` file.

Please set the repository URL for Bacula Community packages, `dl_area` variable, in the `default/main.yml` file for each role previously to use them.

You can use the following command to modify all `dl_area` values:

> find /root/.ansible/collections/ansible_collections/bacula/bacula_community/roles/ -name main.yml -ls -exec sed -i 's/@@repository_code@/<your_user_repository_code_here>/g' {} \\;

The Bacula Client and the Bacula Storage will use the password used by the Director {} resource present in the bacula-fd.conf and bacula-sd.conf files, respectively, to communicate with the remote Director provided in the deployment.

The following variables must be set in the playbook, inventory file or by using `--extra-vars`, depending on the role used: `bce_version, director_hostname, client_hostname, client_name`, `storage_hostname`, `storage_name`, `volumes_directory` and `dedup_directories`.

Variable | Description
-------- | ---------------------
bce_version | the Bacula Community version, for example, `11.0.1`.
director_hostname | the FQDN of the Director host, for example, `baculadir.domain.com`. The name of the Director is, by default, the hostname + "-dir". In this example, the Director name will be `baculadir-dir`.
client_hostname | the FQDN of the Client host, for example, `client.domain.com`.
client_name | the name of the File Daemon/Client, for example, `client1-fd`. If not specified, the `client_name` will be the `client_hostname` + `-fd`.
storage_hostname | the FQDN of the Storage host, for example, `baculasd.domain.com`.
storage_name | the name of the Storage, for example, `storage1-sd`. If not specified, the `storage_name` will be the `storage_hostname + `-sd`.
volumes_directory | this variable will be required by the `bce_sdonly` role. This is where the volumes will be stored, the value used in the Device {} `archive_device` directive.

### Usage and Examples

The following examples use the playbooks in the `tests` directory of the Bacula Community collection.

1) To deploy Bacula Community Version (Director, Storage Daemon and File Daemon) `11.0.1` in the `baculadir.domain.com` host, using an inventory file:

> ansible-playbook -i tests/directors.yaml tests/bce.yml"

The `directors.yaml` file:

```
---
directors:
  hosts:
    baculadir.domain.com:
      director_hostname: baculadir.domain.com
      bce_version: 11.0.1
```

or using --extra-vars (please modify `- hosts: directors` to `- hosts: {{ director_hostname }}` in the `tests/bce.yml playbook):

> ansible-playbook tests/bce.yml --extra-vars "director_hostname=baculadir.domain.com bce_version=11.0.1"

2) To install two remote Bacula Community Version File Daemon only version `11.0.1` in the `client1.domain.com` and the `client2.domain.com` hosts and deploy the clients configuration in the `baculadir.domain.com-dir` Director in the `baculadir.domain.com` host:

> ansible-playbook -i tests\clients.yaml tests/bce_fdonly.yml

The `clients.yaml` file:

```
---
clients:
  hosts:
    client1.domain.com:
      client_hostname: client1.domain.com
    client2.domain.com:
      client_hostname: client2.domain.com
  vars:
    bce_version: 11.0.1
    director_hostname: baculadir.domain.com
```

3) To install two remote Bacula Community Version Storage Daemon only `11.0.1` in the `storage1.domain.com` and the `storage2.domain.com` hosts (Bacula Community Version will be installed and Director service will be manually stopped and disabled):

> ansible-playbook -i tests/storages.yaml tests/bce_sdonly.yml

The `storages.yaml` file:

```
---
storages:
  hosts:
    storage1.domain.com
      storage_hostname: storage1.domain.com
    storage2.domain.com
      storage_hostname: storage2.domain.com
  vars:
    director_hostname: baculadir.domain.com
    bce_version: 11.0.1
    volumes_directory: /opt/bacula/volumes
```

**Please note all the examples above can use a inventory file or --extra-vars or you may modify the playbooks to use any variable assignment allowed in Ansible.**

## Requirements

`jmespath` library needs to be installed on the host running the playbook. This is needed for the `json_query` filter.

To avoid `[WARNING]: Module remote_tmp ... did not exist and was created with a mode of 0700, this may cause issues when running as another user. To avoid this, create the remote_tmp dir with the correct permissions
manually`, we do recommend to set the 'remote_tmp', for example, in the ansible.cfg:

remote_tmp = ~/.ansible/tmp

## Important Notes

All the roles will trigger the operating system update/upgrade as this is strongly recommended by Bacula Systems before installing Bacula Community Version.

## Licensing
License: BSD 2-Clause; see file LICENSE
