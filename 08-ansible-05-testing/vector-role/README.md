Ansible-Role Vector for CentOS 7
=========

Role installs Vector on CentOS 7. 

Requirements
------------

None.

Role Variables
--------------

Available variables are listed below, along with default values (see `defaults/main.yml` and `vars/main.yml`):

`vector_version` - версия Vector

`vector_rpm` - Файл установки Vector

`vector_config_dir` - Папка конфигураций Vector

`vector_config` - Конфигурация Vector


* The version vector to install.
```yml
  vector_version: 0.21.2
```
* Change server name clickhouse ['clickhouse-01']
```yml
  vector_conf_endpoint: http://{{ hostvars['clickhouse-01'].ansible_default_ipv4.address }}:8123
```
Dependencies
------------

None.

Example Playbook
----------------

The simpliest example:
```yaml
- name: Install Vector
  hosts: vector
  roles:
    - vector-role
```

License
-------

MIT

Author Information
------------------

The role was created by [Dmitry Olkhovik](https://github.com/olkhovik) in June 2022 as a homework during a DevOps cource on Netology online educational platform.

