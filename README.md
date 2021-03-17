# Bacula Community Collection for Ansible - bacula.bacula_community

An Ansible Collection of roles to install Bacula Community - PostgreSQL Catalog, Bacula File Daemon only and BWeb in RHEL/CentOS/Debian/Ubuntu/OracleLinux/SLES.

## Ansible version compatibility

This collection has bcen tested against following Ansible versions: **>=2.9,<2.10**.

## Installing the Collection

Before using the Bacula Community collection, you need to install it with the Ansible Galaxy CLI:

    ansible-galaxy collection install bacula.bacula_community

## Roles

Role | Description
-------- | ---------------------
bce | install Bacula Community - PostgreSQL Catalog. This role may install PostgreSQL required packages if not present in the host.
bce_fdonly | install Bacula File Daemon only. This is a Bacula Client.
bce_sdonly | Install a Bacula Storage. This role follows the recommended procedure to install Bacula Community and stop/disable the Director.
bce_fdplugin | install a Bacula Community Plugin in a Bacula Client host. Please note it is required to have a Bacula File Daemon installed and running before installing the File Daemon Plugin.
bce_sdplugin | install a Bacula Community Plugin in a Bacula Storage host. Please note it is required to have a Bacula Storage Daemon installed and running before installing the Storage Daemon Plugin.
bce_plugindependencies | install Plugins required dependencies. This role is required and used by both the *bce_fdplugin* and the *bce_sdplugin* roles.

### Roles Variables

The roles have most of the variables defined in the `vars` directory or in the `default/main.yml` file.

Please set the repository URL for Bacula Community packages, `dl_area` variable, in the `default/main.yml` file for each role previously to use them.

You can use the following command to modify all `dl_area` values:

> find /root/.ansible/collections/ansible_collections/bacula/bacula_community/roles/ -name main.yml -ls -exec sed -i 's/@@customer@/<your_customer_download_area_here>/g' {} \\;

The Bacula Client and the Bacula Storage will use the password used by the Director {} resource present in the bacula-fd.conf and bacula-sd.conf files, respectively, to communicate with the remote Director provided in the deployment.

The following variables must be set in the playbook, inventory file or by using `--extra-vars`, depending on the role used: `bce_version, director_hostname, client_hostname, client_name`, `storage_hostname`, `storage_name`, `volumes_directory` and `dedup_directories`.

Variable | Description
-------- | ---------------------
bce_version | the Bacula Community version, for example, `12.6.1`.
director_hostname | the FQDN of the Director host, for example, `baculadir.domain.com`. The name of the Director is, by default, the hostname + "-dir". In this example, the Director name will be `baculadir-dir`.
client_hostname | the FQDN of the Client host, for example, `client.domain.com`.
storage_hostname | the FQDN of the Storage host, for example, `baculasd.domain.com`.
storage_name | the name of the Storage, for example, `mybacula-sd`. If not specified, the `storage_name` will be the hostname + `-sd`.
volumes_directory | this variable will be required in both the `bce_sdonly` and `bce_sdplugin` roles. This is where the volumes will be stored, the value used in the Device {} `archive_device` directive.
dedup_directories | these are the Dedup Index and the Dedup Containers directories required when using the `bce_sdplugin` role to install the GED plugin. If not defined, the default values are `/opt/bacula/dedup/index` and `/opt/bacula/dedup/containers`. They can be modified using, for example, --extra-vars='{"dedup_directories": [/mnt/dedup/index, /mnt/dedup/containers]}'. Please note the first directory will be used for the Dedup Index and the second one, for the Dedup Containers.

### Usage and Examples

The following examples use the playbooks in the `tests` directory of the Bacula Community collection.

1) To deploy Bacula Community (Director, Storage Daemon and File Daemon) `12.6.1` in the `baculadir.domain.com` host:

> ansible-playbook -i inventory tests/bce.yml --extra-vars "director_hostname=baculadir.domain.com bce_version=12.6.1"

2) To install a remote Bacula Community File Daemon only version `12.6.1` in the `client.domain.com` host and deploy the client configuration in the `baculadir.domain.com-dir` Director in the `baculadir.domain.com` host:

> ansible-playbook -i inventory tests/bce_fdonly.yml --extra-vars "client_hostname=client.domain.com client_name=bacula-fd bce_version=12.6.1 director_hostname=baculadir.domain.com"

3) To install a Bacula Community File Daemon Plugin `12.6.1` in the remote client in the `client.domain.com` host and deploy the plugin configuration (basic FileSet, not including plugin options in most of the cases) in the `baculadir.domain.com-dir` Director in the `baculadir.domain.com` host:

> ansible-playbook -i inventory tests/bce_fdplugin.yml --extra-vars "client_hostname=client.domain.com client_name=bacula-fd bce_version=12.6.1 fdplugin=mysql director_hostname=baculadir.domain.com"

4) To install a Bacula Community Storage Daemon Plugin `12.6.1` in the `baculadir.domain.com` (the same host as Director above):

> ansible-playbook -i inventory tests/bce_sdplugin.yml --extra-vars "storage_hostname=baculadir.domain.com storage_name=bacula-dedup-sd bce_version=12.6.1 sdplugin=dedup volumes_directory=/mnt/dedup/volumes director_hostname=baculadir.domain.com"

> ansible-playbook -i inventory tests/bce_sdplugin.yml --extra-vars '{"storage_hostname":"baculadir.domain.com",  "storage_name":"bacula-dedup-sd",  "bce_version":"12.6.1", "sdplugin":"dedup",  "volumes_directory":"/mnt/dedup/data/volumes",  "dedup_directories":"["/mnt/dedup/index", "/mnt/dedup/data/containers"],  "director_hostname":"baculadir.domain.com"}'

5) To install a remote Bacula Community Storage Daemon only `12.6.1` in the `bacula_sd.domain.com` host (Bacula Community will be installed and Director and File Daemon services must be manually stopped and disabled):

> ansible-playbook -i inventory tests/bce_sdonly.yml --extra-vars "storage_hostname=bacula_sd.domain.com storage_name=bacula-remote-sd bce_version=12.6.1 director_hostname=baculadir.domain.com director_name=baculadir.domain.com-dir"

## Requirements

`jmespath` library needs to be installed on the host running the playbook. This is needed for the `json_query` filter.

To avoid `[WARNING]: Module remote_tmp ... did not exist and was created with a mode of 0700, this may cause issues when running as another user. To avoid this, create the remote_tmp dir with the correct permissions
manually`, we do recommend to set the 'remote_tmp', for example, in the ansible.cfg:

remote_tmp = ~/.ansible/tmp

## Important Notes

All the roles will trigger the operating system update/upgrade as this is strongly recommended by Bacula Systems before installing Bacula Community.

## Licensing
License: BSD 2-Clause; see file LICENSE
