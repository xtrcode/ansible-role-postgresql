Role Name
=========

A customizable PostgreSQL role for Debian 12-13. Other distributions are not tested.

Requirements
------------

`gather_facts` must be enabled to calculate the default memory configuration for PostgreSQL (see Role Variables).

Role Variables
--------------

Most-used variables are listed here. See all available in `defaults/main.yml`.

```yaml
postgres_packages:
  - postgresql
  - postgresql-common
  - python3-psycopg2
  - libpq-dev
```
Default packages needed for operation. Keep in mind that these are the Debian 12 packages.

```yaml
postgres_daemon: "postgresql"
```

Systemd unit file name.

```yaml
postgres_user: "postgres"
postgres_group: "postgres"
```
Default user and group, usually created automatically during installation.

```yaml
postgres_data_dir: "{{ postgres_data_dir_default }}"
postgres_data_dir_move: false
```

Controls weather the data directory should be moved upon first-run. Leave as is, to keep the default location.

_When the data dir is moved, a lock-file is placed in the default location (src) to prevent additional moves on consecutive runs_

```yaml
postgres_auth_method: "{{ ansible_fips | ternary('scram-sha-256', 'md5') }}"
```
Default authentication method used for `pg_hba.conf` entries.

```yaml
postgres_locales:
  - "en_US.UTF-8"
  - "de_DE.UTF-8"
```
Default locales. See all available locales by running `$ locale -a`.

```yaml
postgres_users: []
```
 List of users that should be `present` or `absent` in the `pg_hba.conf`. This directive **DOES NOT CREATE USERS**. Leave blank (`[]`) to keep distribution-defaults.

 ```yaml
 postgres_users: 
  - {
      contype: host,
      users: admin1,
      source: IP_OF_YOUR_WORKSTATION,
      databases: all,
      method: "scram-sha-256",
    }
  - {
      contype: local,
      users: all,
      source: all,
      databases: replication,
      method: peer,
      state: absent,
    }
  - {
      contype: host,
      users: all,
      source: 127.0.0.1/32,
      databases: replication,
      method: scram-sha-256,
      state: absent,
    }
  - {
      contype: host,
      users: all,
      source: "::1/128",
      databases: replication,
      method: scram-sha-256,
      state: absent,
    }
  - {
      contype: host,
      users: all,
      source: 127.0.0.1/32,
      databases: all,
      method: scram-sha-256,
      state: absent,
    }
  - {
      contype: host,
      users: all,
      source: "::1/128",
      databases: all,
      method: scram-sha-256,
      state: absent,
    }

 ```
 To remove the Debian 12 defaults and add a new user `admin1` to  `pg_hba.conf` use the following example:

```yaml
postgres_conf_port: 5432
postgres_conf_listen_addresses: "0.0.0.0"
postgres_conf_timezone: "Europe/Berlin"
```
Default port, listening address and timezone.

**DANGER ZONE**
 ```yaml
postgres_conf_max_connections: 100
postgres_conf_shared_buffers: "{{ (ansible_memtotal_mb * 0.15) | int | abs }}MB"
postgres_conf_work_mem: "{{ (ansible_memtotal_mb * 0.25) / (postgres_conf_max_connections | int) | int | abs }}MB"
postgres_conf_maintenance_work_mem: "{{ (ansible_memtotal_mb * 0.05) | int | abs }}MB"
postgres_conf_effective_cache_size: "{{ (ansible_memtotal_mb * 0.5) | int | abs }}MB"
postgres_conf_log_autovacuum_min_duration: 0
postgres_conf_log_checkpoints: "on"
postgres_conf_log_connections: "on"
postgres_conf_log_disconnections: "on"
postgres_conf_log_lock_waits: "on"
postgres_conf_log_temp_files: 0
 ```

 These are _my_ suitable defaults. Feel free to change to your requirements.

Dependencies
------------

None.

Example Playbook
----------------

### Install locally
```yaml
# requirements.yml
- src: git+ssh://git@github.com/xtrcode/ansible-role-postgresql.git
  scm: git
```

```bash
$ ansible-galaxy install -r requirements.yml -p ./roles
```

### Playbook
```yaml
- hosts: deploy
  roles:
    - { role: ansible-role-postgresql }
```

License
-------

MIT 

Author Information
------------------

Website: [blog.xtracode.ws](https://blog.xtracode.ws)
