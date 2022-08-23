# Bacula Community Collection for Ansible - baculasystems.bacula_community

An Ansible Collection of roles to install one or more of the Bacula Community Director, (using PostgreSQL/MySQL/MariaDB catalog), a Bacula File Daemon (FD) only and a Bacula Storage Daemon (SD) only in RHEL/CentOS/Debian/Ubuntu/OracleLinux Linux distributions.

## Ansible version compatibility

This collection has been tested against following Ansible versions: **>=2.9,<2.12**.

## Requirements

- The `jmespath` library package must be installed on the Ansible host running the playbook (eg: `apt install python3-jmespath` on Debian 10). This is needed for the `json_query` filter. This requirement is included in the requirements.txt file available in the collection.

- The `remote_tmp` variable should be set in your Ansible configuration to avoid the warning:

`[WARNING]: Module remote_tmp ... did not exist and was created with a mode of 0700, this may cause issues when running as another user. To avoid this, create the remote_tmp dir with the correct permissions manually`

For example:

remote_tmp = ~/.ansible/tmp

## Important Notes

Bacula Systems strongly recommends having a fully patched system before installing any Bacula component. This being the case, all the roles are written to automatically trigger the Linux distribution's update/upgrade process (eg: apt update; apt upgrade in Debian/Ubuntu) prior to deploying any of the Bacula components.

Some roles have a `templates` directory with Clients and Autochangers that will be deployed when using the roles. We strongly recommend that you adapt these templates to your environment before deploying Clients and Storages.

## Installing the Collection

Before using the Bacula Community collection, it must be installed with the Ansible Galaxy command:

> ansible-galaxy collection install baculasystems.bacula_community

## Roles

Role | Description
-------- | ---------------------
bce | install Bacula Community version - PostgreSQL Catalog. This role may install PostgreSQL required packages if not present in the host. This role also installs the Bacula Director (DIR), a Storage Daemon (SD), and a File Daemon (SD).
bce_mysql | install Bacula Community version - MySQL Catalog. This role may install MySQL required packages if not present in the host (must set install_mysql=yes). If MySQL is already installed in the host, the mysql_root_password variable must be provided. This role also installs the Bacula Director (DIR), a Storage Daemon (SD), and a File Daemon (SD).
bce_mariadb | install Bacula Community version - MariaDB Catalog. This role may install MariaDB required packages if not present in the host. This role also installs the Bacula Director (DIR), a Storage Daemon (SD), and a File Daemon (SD).
bce-fdonly | install Bacula File Daemon only. This is a Bacula Client.
bce-sdonly | Install a Bacula Storage. This role follows the recommended procedure to install Bacula Community Director, Storage Daemon and File Daemon and stop/disable the Director.

### Roles Variables

The roles have most of the variables defined in the `vars` directories or in the `default/main.yml` files.

Please set your Bacula Community subscription's repository URL (download area), the `repository_code` variable, in the `default/main.yml` file for each role prior to using them.

You may use the following command to set all `repository_code` values at once:

> find /root/.ansible/collections/ansible_collections/baculasystems/bacula_community/roles/ -name main.yml -exec sed -i 's/@@repository_code@/<your_repository_string>/g' {} \\;

The Bacula Clients (File Daemons) and the Bacula Storage Daemons will use the password used by the Director {} resource present in the bacula-fd.conf and bacula-sd.conf files respectively to communicate with the remote Director provided in the deployment.

The variables in the following table must be set in a playbook, in an inventory file, or by passing them with the `--extra-vars` option on the command line. Depending on the role used, the following variables may be required: `bce_version, director_hostname, client_hostname, client_name`, `storage_hostname`, `storage_name` and `volumes_directory`


Variable | Description
-------- | ---------------------
bce_version | the Bacula Community version version, for example: `13.0.1`.
director_hostname | the FQDN of the Director host, for example: `baculadir.example.com`. The name of the Director is, by default, the hostname + "-dir". In this example, the Director name will be `baculadir-dir`.
client_hostname | the FQDN of the Client host, for example, `baculaclient.example.com`.
client_name | the name of the File Daemon/Client. For example, `baculaclient1-fd`. If not specified, the `client_name` will be automatically created using the `client_hostname` + `-fd`.
storage_hostname | the FQDN of the Storage host, for example, `baculasd1.example.com`.
storage_name | the name of the Storage. For example, `baculasd1-sd`. If not specified, the `storage_name` will be the `storage_hostname + `-sd`.
volumes_directory | this variable will be required when using the `bce-sdonly` role. This is where the Bacula volumes will be stored. It is the value used in the SD Device resource(s) `Archive Device = xxxx` directive(s).
install_mysql | if set to yes, the MySQL Server (either community version or packages version) will be installed. Default value is no.
mysql_root_password | if a MySQL Server is already installed, this variable is the current MySQL root password and must be provided.

### Usage and Examples

The following examples use the playbooks in the `tests` directory of the Bacula Community collection.

1) To deploy Bacula Community version (DIR, SD, and FD) `13.0.1`, using a PostgreSQL Catalog, on a host with the FQDN `baculadir.example.com` using the --extra_vars command line option, and the `bce.yml` playbook:

> ansible-playbook -i baculadir.example.com, tests/bce.yml --extra-vars "director_hostname=baculadir.example.com bce_version=13.0.1"

This is the `tests/bce.yml` file referenced in the command line above:
```
---
- hosts: "{{ director_hostname }}"
  remote_user: root
  tasks:
  - name: Install Bacula Community version - PostgreSQL Catalog
    import_role:
      name: baculasystems.bacula_community.bce
```

2) Using an inventory file to install two remote Bacula Community version version `13.0.1` File Daemon to the hosts `client1.example.com` and `client2.example.com`, and deploy the two clients' resource definitions in the Director's configuration on the `baculadir.example.com` host, first an inventory file needs to be created:

Note: In this example, please be sure to edit the `tests/clients.yaml` inventory file to match your environment.

The `clients.yaml` inventory file:
```
---
clients:
  hosts:
    client1.example.com:
      client_hostname: client1.example.com
      client_name: client1-fd
    client2.example.com:
      client_hostname: client2.example.com

  vars:
    bce_version: 13.0.1
    director_hostname: baculadir.example.com
```

Then, to deploy the two Bacula File Daemons to the two hosts using the `bce-fdonly.yml` playbook and the above `clients.yaml` inventory file:

> ansible-playbook -i tests/clients.yaml tests/bce-fdonly.yml

This is the `tests/bce-fdonly.yml` playbook referenced in the command line above:
```
---
- hosts: clients
  tasks:

  - name: Install and configure Bacula File Daemon
    import_role:
      name: baculasystems.bacula_community.bce_fdonly
```

3) To install two remote Bacula Community version version `13.0.1` Storage Daemons to the hosts `storage1.example.com` and `storage2.example.com`, using an inventory file. Bacula Community version DIR, SD, and FD compnents will be installed and the Bacula Director service will be automatically disabled. First an inventory file needs to be created:

Note: In this example, please be sure to edit the `tests/storages.yaml` inventory file to match your environment.

Example `tests/storages.yaml` inventory file:
```
---
storages:
  hosts:
    storage1.example.com:
      storage_hostname: storage1.example.com
    storage2.example.com:
      storage_hostname: storage2.example.com
  vars:
    director_hostname: baculadir.example.com
    bce_version: 13.0.1
    volumes_directory: /opt/bacula/volumes
```

Then, to deploy the two Bacula Storage Daemons to the two hosts using the `bce-sdonly.yml` playbook and the above `storages.yaml` inventory file:

> ansible-playbook -i tests/storages.yaml tests/bce-sdonly.yml

This is the `tests/bce-sdonly.yml` playbook referenced in the command line above:
```
---
- hosts: storages
  tasks:

  - name: Install and configure Bacula Storage Daemon only
    import_role:
      name: baculasystems.bacula_community.bce_sdonly
```


**Please note all the examples above can use an inventory file and/or --extra-vars or you may modify the playbooks to use any variable assignment allowed in Ansible.**


## Licensing

License: BSD 2-Clause; see file LICENSE
