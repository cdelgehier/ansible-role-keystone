[![Build Status](https://travis-ci.org/open-io/ansible-role-openio-keystone.svg?branch=master)](https://travis-ci.org/open-io/ansible-role-openio-keystone)
# Ansible role `keystone`

An Ansible role for keystone. Specifically, the responsibilities of this role are to:

- Install a keystone
- Configure a keystone
- Support SQLite or MySQL engine
- Add user, project and role to keystone

## Requirements

- Ansible 2.4

## Role Variables


| Variable   | Default | Comments (type)  |
| :---       | :---    | :---             |
| `openio_keystone_bind_interface` | `"{{ ansible_default_ipv4.alias }}"` | Name of the NIC to use |
| `openio_keystone_bindir` | `/usr/bin` | OpenStack's bin folde |
| `openio_keystone_bootstrap_all_nodes` | `false` | Bootstrap the first endpoint on all nodes |
| `openio_keystone_cloudname` | `openio` | The cloud name in the /etc/openstack/clouds.yaml |
| `openio_keystone_config_cache_backend` | `dogpile.cache.memcached` | Cache in memcached |
| `openio_keystone_config_cache_debug_cache_backend` | `""` | No debug cache |
| `openio_keystone_config_cache_enabled` | `true` | Cache in memcached |
| `openio_keystone_config_cache_memcache_dead_retry` | `""` |  |
| `openio_keystone_config_cache_memcache_servers` | `` | `IP`:`PORT` for memcahed |
| `openio_keystone_config_default_admin_token` | `""` |  |
| `openio_keystone_config_default_debug` | `false` |  |
| `openio_keystone_config_default_log_dir` | `"/var/log/keystone"` | Logs folder |
| `openio_keystone_config_dir` | `/etc/keystone` | Keystone's config folder |
| `openio_keystone_credentials_tokens_key_repository` | `/etc/keystone/credential-keys/` | Credential folder |
| `openio_keystone_database_connection` | `sqlite:///{{ openio_keystone_database_sqlite_directory }}/{{ openio_keystone_database_sqlite_file }}` | Connection string |
| `openio_keystone_database_mysql_connection_address` | `` | Database's `IP`  |
| `openio_keystone_database_mysql_connection_database` | `keystone` | DB name |
| `openio_keystone_database_mysql_connection_password` | `` | Password for keystone user |
| `openio_keystone_database_mysql_connection_user` | `keystone` | DB user |
| `openio_keystone_database_sqlite_directory` | `/var/lib/keystone` | SQLite folder |
| `openio_keystone_database_sqlite_file` | `keystone.db` | SQLite file |
| `openio_keystone_fernet_tokens_key_repository` | `/etc/keystone/fernet-keys/` | Fernet keys folder |
| `openio_keystone_gridinit_dir` | `"/etc/gridinit.d/{{ openio_keystone_namespace }}"` | Path to copy the gridinit conf |
| `openio_keystone_gridinit_file_prefix` | `""` | Maybe set it to `{{ openio_keystone_namespace }}-` for old gridinit's style |
| `openio_keystone_namespace` | `"OPENIO"` | Namespace |
| `openio_keystone_nodes_group` | `openio_keystone` | Groupname in your inventory |
| `openio_keystone_openstack_distribution` | `pike` | Release of OpenStack |
| `openio_keystone_projects` | `` | `dict` of projects to declare |
| `openio_keystone_repo_managed` | `false` | Manage repository |
| `openio_keystone_serviceid` | `"0"` | ID in gridinit |
| `openio_keystone_services` | `` | `dict` of services to declare |
| `openio_keystone_services_to_bootstrap` | `` | `dict` of services to bootstrap |
| `openio_keystone_system_group_name` | `keystone` | Process group  |
| `openio_keystone_system_user_name` | `keystone` | Process user |
| `openio_keystone_tmp_keys_dir` | `/tmp` | temporary folder |
| `openio_keystone_token_provider` | `fernet` |   |
| `openio_keystone_users` | `` | `dict` of services to declare |
| `openio_keystone_uwsgi_bind` | `` | `dict` of uwsgi endpoint |
| `openio_keystone_wsgi_admin_program_name` | ` keystone-wsgi-admin` |   |
| `openio_keystone_wsgi_processes` | `ansible_processor_vcpus * 2` | `ansible_processor_vcpus * 2 ` limited to `openio_keystone_wsgi_processes_max` |
| `openio_keystone_wsgi_processes_max` | `16` | Max of `openio_keystone_wsgi_processes` |
| `openio_keystone_wsgi_program_names` | `` | WSGI endpoint to run |
| `openio_keystone_wsgi_public_program_name` | `keystone-wsgi-public` |  |
| `openio_keystone_wsgi_threads` | `1` | |


### User & project

After a bootstrap, it s possible to declare users, services, and projects into keystone

Like this:

- USERS
```yaml
openio_keystone_users:
  - name: demo
    password: DEMO_PASS
  - name: swift
    password: SWIFT_PASS
```

- SERVICES
```yaml
openio_keystone_services:
  - name: openio-swift
    type: object-store
    description: OpenIO SDS swift proxy
    endpoint:
      - interface: admin
        url: "http://foo.com:6007/v1/AUTH_%(tenant_id)s"
      - interface: internal
        url: "http://foo.com:6007/v1/AUTH_%(tenant_id)s"
      - interface: public
        url: "http://foo.com:6007/v1/AUTH_%(tenant_id)s"
```

- PROJECTS
```yaml
openio_keystone_projects:
  - name: service
    domain_id: default
    roles:
      - user: swift
        role: admin
  - name: demo
    description: OpenIO demo project
    domain_id: default
    roles:
      - user: demo
        role: _member_
      - user: demo
        role: admin
```
## Dependencies

- Epel for RedHat familly
- This role don't install a MariaDB server (in case of mysql engine). You have to install it


## Example Playbook

```yaml
- hosts: my_keystones
  gather_facts: true
  become: true
  roles:
    - role: repository
    - role: gridinit
    - role: memcached
      openio_memcached_namespace: "{{ namespace }}"
      openio_memcached_serviceid: "0"
      openio_memcached_bind_address: "{{ ansible_default_ipv4.address }}"


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
    openio_keystone_bind_interface: "{{ ansible_default_ipv4.alias }}"
    openio_keystone_namespace: "{{ namespace }}"
    openio_keystone_nodes_group: "my_keystones"
    openio_keystone_config_cache_memcache_servers: "127.0.01:6019"
    openio_keystone_database_engine: mysql
    #openio_keystone_database_mysql_connection_user: keystone
    openio_keystone_database_mysql_connection_password: "{{ keystone_mysql_keystoneuser_password }}"
    openio_keystone_database_mysql_connection_address: "{{ groups['db'] | map('extract', hostvars, ['ansible_default_ipv4', 'address']) | list | first }}"
    #openio_keystone_database_mysql_connection_database: keystone
    openio_keystone_bind_interface: "{{ ansible_default_ipv4.alias }}"
    #openio_keystone_config_cache_memcache_servers: "127.0.01:6019"
    openio_keystone_services_to_bootstrap:
      - name: keystone
        user: admin
        password: "{{ keystone_mysql_keystoneuser_password }}"
        project: admin
        role: admin
        regionid: RegionOne
        # eventually a VIP
        adminurl: "http://{{ VIP.address }}:35357"
        publicurl: "http://{{ VIP.address }}:5000"
        internalurl: "http://{{ VIP.address }}:5000"
    openio_keystone_services:
      - name: openio-swift
        type: object-store
        description: OpenIO SDS swift proxy
        endpoint:
          # eventually a VIP
          - interface: admin
            url: "http://{{ VIP.address }}:6007/v1/AUTH_%(tenant_id)s"
          - interface: internal
            url: "http://{{ VIP.address }}:6007/v1/AUTH_%(tenant_id)s"
          - interface: public
            url: "http://{{ VIP.address }}:6007/v1/AUTH_%(tenant_id)s"



    #shorter way with defaults and a local database
    #- role: keystone
    #  openio_keystone_namespace: "{{ namespace }}"
    #  openio_keystone_bind_interface: eth0
    #  openio_keystone_database_engine: mysql
    #  openio_keystone_database_mysql_connection_user: keystone
    #  openio_keystone_database_mysql_connection_password: keystonepass


```


```ini
[my_keystones]
node1 ansible_host=192.168.1.173
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
