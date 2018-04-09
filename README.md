[![Build Status](https://travis-ci.org/open-io/ansible-role-openio-keystone.svg?branch=master)](https://travis-ci.org/open-io/ansible-role-openio-keystone)
# Ansible role `keystone`

An Ansible role for keystone. Specifically, the responsibilities of this role are to:

- Install a keystone
- Configure a keystone
- Support SQLite or MySQL engine

## Requirements

- Ansible 2.4+

## Role Variables


| Variable   | Default | Comments (type)  |
| :---       | :---    | :---             |
| `openio_keystone_bind_interface` | `ansible_default_ipv4.alias` | Network interface to use |
| `openio_keystone_bindir` | `/usr/bin` | OpenStack's bin folder |
| `openio_keystone_bootstrap_all_nodes` | `true` | Bootstrap the first endpoint on all nodes |
| `openio_keystone_config_cache_backend` | `""` |  |
| `openio_keystone_config_cache_debug_cache_backend` | `""` |  |
| `openio_keystone_config_cache_enabled` | `true` |  |
| `openio_keystone_config_cache_memcache_dead_retry` | `""` |  |
| `openio_keystone_config_cache_memcache_servers` | `"{{ hostvars[inventory_hostname]['ansible_' + openio_keystone_bind_interface]['ipv4']['address'] }}` | Local memcached address |
| `openio_keystone_config_default_admin_token` | `""` |  |
| `openio_keystone_config_default_debug` | `false` |  |
| `openio_keystone_config_default_log_dir` | `/var/log/keystone` | Log file |
| `openio_keystone_config_dir` | `/etc/keystone` |  |
| `openio_keystone_credentials_tokens_key_repository` | `/etc/keystone/credential-keys/` |  |
| `openio_keystone_database_connection` | `` |  |
| `openio_keystone_database_engine` | `sqlite` | SQL engine used to store data (sqlite or mysql is possible) |
| `openio_keystone_database_mysql_connection_address` | `openio_keystone_bind_interface.ipv4.address` | MySQL's address (in case of mysql engine) |
| `openio_keystone_database_mysql_connection_database` | `keystone` | MySQL's database for keystone (in case of mysql engine) |
| `openio_keystone_database_mysql_connection_password` | `KEYSTONE_PASS` | MySQL's password for previous user (in case of mysql engine) |
| `openio_keystone_database_mysql_connection_user` | `keystone` | MySQL's user for keystone (in case of mysql engine) |
| `openio_keystone_database_sqlite_directory` | `/var/lib/keystone` | Database's folder (in case of sqlite engine) |
| `openio_keystone_database_sqlite_file` | `keystone.db` | Database's file (in case of sqlite engine) |
| `openio_keystone_fernet_tokens_key_repository` | `/etc/keystone/fernet-keys/` | Folder for fernet keys |
| `openio_keystone_nodes_group` | `openio_keystone` | Group of inventory |
| `openio_keystone_openstack_distribution` | `pike` | Distribution of OpenStack |
| `openio_keystone_repo_managed` | `false` | Install repository OpenStack |
| `openio_keystone_services_to_bootstrap` | `` | List of service to declare |
| `openio_keystone_system_group_name` | `keystone` |  |
| `openio_keystone_system_user_name` | `keystone` |  |
| `openio_keystone_tmp_keys_dir` | `/tmp` | Local folder used to share keys and credentials |
| `openio_keystone_token_provider` | `fernet` |  |
| `openio_keystone_uwsgi_bind` | `` | Dict of service to serve via uwsgi |
| `openio_keystone_wsgi_admin_program_name` | `keystone-wsgi-admin` | WSGI Script Admin |
| `openio_keystone_wsgi_processes` | `{{ [[ansible_processor_vcpus|default(1), 1] | max * 2, openio_keystone_wsgi_processes_max] | min }}` |  |
| `openio_keystone_wsgi_program_names` | `` | Dict of WSGI Script to serve |
| `openio_keystone_wsgi_public_program_name` | `keystone-wsgi-public` | WSGI Script public |
| `openio_keystone_wsgi_threads` | `1` |  |
## Dependencies

- Epel for RedHat familly
- This role don't install a MariaDB server (in case of mysql engine). You have to install it
- This role don't manage the gridinit ressources. You have to configure them before.

## Example Playbook

```yaml
- hosts: keystones
  gather_facts: true
  become: true
  roles:
    - role: repository
    - role: gridinit
    - role: memcached
      openio_memcached_namespace: "{{ namespace }}"
      openio_memcached_serviceid: "0"
      openio_memcached_bind_address: "{{ ansible_default_ipv4.address }}"


    #sqlite
    #- role: keystone
    #  openio_keystone_nodes_group: keystones
    #  openio_keystone_config_cache_memcache_servers: "{{ ansible_default_ipv4.address }}:6019"
  
    #mysql
    - role: mariadb
      mariadb_bind_address: "{{ ansible_default_ipv4.address }}"
      mariadb_root_password: "{{ keystone_mysql_rootuser_password }}"
      mariadb_databases:
        - name: keystone
      mariadb_users:
        - name: keystone
          password: "{{ keystone_mysql_keystoneuser_password }}"
          priv: 'keystone.*:ALL'
          host: '%'

        - name: keystone
          password: "{{ keystone_mysql_keystoneuser_password }}"
          priv: '*.*:SUPER'
          append_privs: "yes"
          host: '%'


    - role: keystone
      openio_keystone_nodes_group: keystones
      openio_keystone_namespace: "{{ namespace }}"
      openio_keystone_config_cache_memcache_servers: "{{ ansible_default_ipv4.address }}:6019"
      openio_keystone_database_engine: mysql
      openio_keystone_database_mysql_connection_user: keystone
      openio_keystone_database_mysql_connection_password: keystonepass
      openio_keystone_database_mysql_connection_address: "{{ ansible_default_ipv4.address }}"
      openio_keystone_database_mysql_connection_database: keystone
      openio_keystone_services_to_bootstrap:
        - name: keystone
          user: admin
          password: ADMIN_PASS
          project: admin
          role: admin
          regionid: RegionOne
          adminurl: "http://{{ ansible_default_ipv4.address }}:35357"
          publicurl: "http://{{ ansible_default_ipv4.address }}:5000"
          internalurl: "http://{{ ansible_default_ipv4.address }}:5000"


    #shorter way with defaults and a local database
    #- role: keystone
    #  openio_keystone_namespace: "{{ namespace }}"
    #  openio_keystone_bind_interface: eth0
    #  openio_keystone_database_engine: mysql
    #  openio_keystone_database_mysql_connection_user: keystone
    #  openio_keystone_database_mysql_connection_password: keystonepass

      
```


```ini
[keystones]
node1 ansible_host=192.168.1.173
```

### Rights for keystone user (mysql)
```
mariadb_databases:
  - name: keystone
mariadb_users:
  - name: keystone
    password: "{{ keystone_mysql_keystoneuser_password }}"
    priv: 'keystone.*:ALL'
    host: '%'

  - name: keystone
    password: "{{ keystone_mysql_keystoneuser_password }}"
    priv: '*.*:SUPER'
    append_privs: yes
    host: '%'
```

## Contributing

Issues, feature requests, ideas are appreciated and can be posted in the Issues section.

Pull requests are also very welcome.
The best way to submit a PR is by first creating a fork of this Github project, then creating a topic branch for the suggested change and pushing that branch to your own fork.
Github can then easily create a PR based on that branch.

## License

Apache License, Version 2.0

## Contributors

- [Cedric DELGEHIER](https://github.com/cdelgehier) (maintainer)
- [Romain ACCIARI](https://github.com/racciari) (maintainer)
- [Pierre Mavro](https://github.com/deimosfr)
