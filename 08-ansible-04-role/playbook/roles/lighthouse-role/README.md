Ansible-Role Lighthouse for CentOS 7
=========

Role installs Lighthouse on CentOS 7. 

Requirements
------------

None.

Role Variables
--------------

Available variables are listed below, along with default values (see `defaults/main.yml` and `vars/main.yml`):

* The git repo of lighthouse to install:
  ```yml
  lighthouse_vcs: https://github.com/VKCOM/lighthouse.git
  ```

Dependencies
------------

To install and work `ligthouse`, you will also need to install `Git` and `NGINX`.

Example Playbook
----------------

The simpliest example:
```yaml
- name: Install Lighthouse
  hosts: lighthouse
  roles:
    - lighthouse
```

License
-------

MIT

Author Information
------------------

The role was created by [Dmitry Olkhovik](https://github.com/olkhovik) in June 2022 as a homework during a DevOps cource on Netology online educational platform.

