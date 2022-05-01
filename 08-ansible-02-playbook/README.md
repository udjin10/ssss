# Домашняя работа к занятию 8.2 «Работа с Playbook»

## Подготовка к выполнению

1. Создайте свой собственный (или используйте старый) публичный репозиторий на github с произвольным именем.
2. Скачайте [playbook](./playbook/) из репозитория с домашним заданием и перенесите его в свой репозиторий.
3. Подготовьте хосты в соответствии с группами из предподготовленного playbook.

Хост поднял в `docker`: [docker-compose.yml](./playbook/ssh_env/docker-compose.yml)

## Основная часть

1. Приготовьте свой собственный inventory файл `prod.yml`.
```
---
clickhouse:
  hosts:
    clickhouse-01:
      ansible_host: 172.19.0.2
      ansible_connection: ssh
      ansible_user: app-admin
      ansible_ssh_private_key_file: ssh_env/id_rsa_pub
```
2. Допишите playbook: нужно сделать ещё один play, который устанавливает и настраивает [vector](https://vector.dev).
3. При создании tasks рекомендую использовать модули: `get_url`, `template`, `unarchive`, `file`.
4. Tasks должны: скачать нужной версии дистрибутив, выполнить распаковку в выбранную директорию, установить vector.

`group_vars/clickhouse/vars.yml`:
```
---
clickhouse_version: "22.3.3.44"
clickhouse_packages:
  - clickhouse-client
  - clickhouse-server
  - clickhouse-common-static
vector_version: "0.21.1"
vector_home: "/opt/vector/{{ vector_version }}"
vector_url: https://packages.timber.io/vector/{{ vector_version }}/vector-{{ vector_version }}-x86_64-unknown-linux-gnu.tar.gz
```
`site.yml`:
```
...
- name: Install Vector
  hosts: clickhouse
  tasks:
    - name: Get Vector
      ansible.builtin.get_url:
        url: "{{ vector_url }}"
        dest: "/tmp/vector-{{ vector_version }}-x86_64-unknown-linux-gnu.tar.gz"
        mode: 0644
        timeout: 60
        force: true
        validate_certs: false
      register: get_vector
      until: get_vector is succeeded
      tags: vector
    - name: Create directrory for Vector
      become: true
      file:
        path: "{{ vector_home }}"
        state: directory
        owner: app-admin
        group: app-admin
        mode: 0755
      tags: vector
    - name: Extract Vector in the installation directory
      become: true
      unarchive:
        copy: false
        src: "/tmp/vector-{{ vector_version }}-x86_64-unknown-linux-gnu.tar.gz"
        dest: "{{ vector_home }}"
        extra_opts: [--strip-components=2]
        creates: "{{ vector_home }}/bin/vector"
      tags: vector
    - name: Set environment Vector
      become: true
      template:
        src: templates/vector.sh.j2
        dest: /etc/profile.d/vector.sh
        mode: 0755
      tags: vector
```
`templates/vector.sh.j2`:
```
# Warning: This file is Ansible Managed, manual changes will be overwritten on next playbook run.
#!/usr/bin/env bash

export V_HOME={{ vector_home }}
export PATH=$PATH:$V_HOME/bin
```

5. Запустите `ansible-lint site.yml` и исправьте ошибки, если они есть.

Ошибка была только эта:
```
dmitry@Lenovo-B50:~/netology/mnt/08-ansible-02-playbook/playbook$ ansible-lint site.yml
[201] Trailing whitespace
site.yml:76
```
поправил отступы

7. Попробуйте запустить playbook на этом окружении с флагом `--check`.
```
dmitry@Lenovo-B50:~/netology/mnt/08-ansible-02-playbook/playbook$ ansible-playbook site.yml -i inventory/prod.yml --check

PLAY [Install Clickhouse] *******************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] **********************************************************************************************************************************************************************************************************************
The authenticity of host '172.19.0.2 (172.19.0.2)' can't be established.
ECDSA key fingerprint is SHA256:4TuEEVYXr10euE/FxjFGHGDsmYdMzkoCkmZq+S0tsDU.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
ok: [clickhouse-01]

TASK [Get clickhouse distrib] ***************************************************************************************************************************************************************************************************************
changed: [clickhouse-01] => (item=clickhouse-client)
changed: [clickhouse-01] => (item=clickhouse-server)
failed: [clickhouse-01] (item=clickhouse-common-static) => {"ansible_loop_var": "item", "changed": false, "dest": "./clickhouse-common-static-22.3.3.44.rpm", "elapsed": 0, "item": "clickhouse-common-static", "msg": "Request failed", "response": "HTTP Error 404: Not Found", "status_code": 404, "url": "https://packages.clickhouse.com/rpm/stable/clickhouse-common-static-22.3.3.44.noarch.rpm"}

TASK [Get clickhouse distrib] ***************************************************************************************************************************************************************************************************************
changed: [clickhouse-01]

TASK [Install clickhouse packages] **********************************************************************************************************************************************************************************************************
fatal: [clickhouse-01]: FAILED! => {"changed": false, "msg": "No RPM file matching 'clickhouse-common-static-22.3.3.44.rpm' found on system", "rc": 127, "results": ["No RPM file matching 'clickhouse-common-static-22.3.3.44.rpm' found on system"]}

PLAY RECAP **********************************************************************************************************************************************************************************************************************************
clickhouse-01              : ok=2    changed=1    unreachable=0    failed=1    skipped=0    rescued=1    ignored=0
```
8. Запустите playbook на `prod.yml` окружении с флагом `--diff`. Убедитесь, что изменения на системе произведены.
```
dmitry@Lenovo-B50:~/netology/mnt/08-ansible-02-playbook/playbook$ ansible-playbook site.yml -i inventory/prod.yml --diff

PLAY [Install Clickhouse] *******************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] **********************************************************************************************************************************************************************************************************************
ok: [clickhouse-01]

TASK [Get clickhouse distrib] ***************************************************************************************************************************************************************************************************************
changed: [clickhouse-01] => (item=clickhouse-client)
changed: [clickhouse-01] => (item=clickhouse-server)
failed: [clickhouse-01] (item=clickhouse-common-static) => {"ansible_loop_var": "item", "changed": false, "dest": "./clickhouse-common-static-22.3.3.44.rpm", "elapsed": 0, "item": "clickhouse-common-static", "msg": "Request failed", "response": "HTTP Error 404: Not Found", "status_code": 404, "url": "https://packages.clickhouse.com/rpm/stable/clickhouse-common-static-22.3.3.44.noarch.rpm"}

TASK [Get clickhouse distrib] ***************************************************************************************************************************************************************************************************************
changed: [clickhouse-01]

TASK [Install clickhouse packages] **********************************************************************************************************************************************************************************************************
changed: [clickhouse-01]

TASK [Flush handlers] ***********************************************************************************************************************************************************************************************************************

RUNNING HANDLER [Start clickhouse service] **************************************************************************************************************************************************************************************************
changed: [clickhouse-01]

TASK [Create database] **********************************************************************************************************************************************************************************************************************
changed: [clickhouse-01]

PLAY [Install Vector] ***********************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] **********************************************************************************************************************************************************************************************************************
ok: [clickhouse-01]

TASK [Get Vector] ***************************************************************************************************************************************************************************************************************************
changed: [clickhouse-01]

TASK [Create directrory for Vector] *********************************************************************************************************************************************************************************************************
--- before
+++ after
@@ -1,6 +1,6 @@
 {
     "path": "/opt/vector/0.21.1",
-    "state": "absent"
+    "state": "directory"
 }

changed: [clickhouse-01]

TASK [Extract Vector in the installation directory] *****************************************************************************************************************************************************************************************
changed: [clickhouse-01]

TASK [Set environment Vector] ***************************************************************************************************************************************************************************************************************
--- before
+++ after: /home/dmitry/.ansible/tmp/ansible-local-3887048zgx7pvgd/tmp_1pde29b/vector.sh.j2
@@ -0,0 +1,5 @@
+# Warning: This file is Ansible Managed, manual changes will be overwritten on next playbook run.
+#!/usr/bin/env bash
+
+export V_HOME=/opt/vector/0.21.1
+export PATH=$PATH:$V_HOME/bin

changed: [clickhouse-01]

PLAY RECAP **********************************************************************************************************************************************************************************************************************************
clickhouse-01              : ok=10   changed=8    unreachable=0    failed=0    skipped=0    rescued=1    ignored=0
```
9. Повторно запустите playbook с флагом `--diff` и убедитесь, что playbook идемпотентен.
```
dmitry@Lenovo-B50:~/netology/mnt/08-ansible-02-playbook/playbook$ ansible-playbook site.yml -i inventory/prod.yml --diff

PLAY [Install Clickhouse] *******************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] **********************************************************************************************************************************************************************************************************************
ok: [clickhouse-01]

TASK [Get clickhouse distrib] ***************************************************************************************************************************************************************************************************************
ok: [clickhouse-01] => (item=clickhouse-client)
ok: [clickhouse-01] => (item=clickhouse-server)
failed: [clickhouse-01] (item=clickhouse-common-static) => {"ansible_loop_var": "item", "changed": false, "dest": "./clickhouse-common-static-22.3.3.44.rpm", "elapsed": 0, "gid": 500, "group": "app-admin", "item": "clickhouse-common-static", "mode": "0664", "msg": "Request failed", "owner": "app-admin", "response": "HTTP Error 404: Not Found", "size": 246310036, "state": "file", "status_code": 404, "uid": 500, "url": "https://packages.clickhouse.com/rpm/stable/clickhouse-common-static-22.3.3.44.noarch.rpm"}

TASK [Get clickhouse distrib] ***************************************************************************************************************************************************************************************************************
ok: [clickhouse-01]

TASK [Install clickhouse packages] **********************************************************************************************************************************************************************************************************
ok: [clickhouse-01]

TASK [Flush handlers] ***********************************************************************************************************************************************************************************************************************

TASK [Create database] **********************************************************************************************************************************************************************************************************************
ok: [clickhouse-01]

PLAY [Install Vector] ***********************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] **********************************************************************************************************************************************************************************************************************
ok: [clickhouse-01]

TASK [Get Vector] ***************************************************************************************************************************************************************************************************************************
ok: [clickhouse-01]

TASK [Create directrory for Vector] *********************************************************************************************************************************************************************************************************
ok: [clickhouse-01]

TASK [Extract Vector in the installation directory] *****************************************************************************************************************************************************************************************
skipping: [clickhouse-01]

TASK [Set environment Vector] ***************************************************************************************************************************************************************************************************************
ok: [clickhouse-01]

PLAY RECAP **********************************************************************************************************************************************************************************************************************************
clickhouse-01              : ok=8    changed=0    unreachable=0    failed=0    skipped=1    rescued=1    ignored=0
```
10. Подготовьте README.md файл по своему playbook. В нём должно быть описано: что делает playbook, какие у него есть параметры и теги.

https://github.com/olkhovik/ansible-8.2/blob/main/README.md

11. Готовый playbook выложите в свой репозиторий, поставьте тег `08-ansible-02-playbook` на фиксирующий коммит, в ответ предоставьте ссылку на него.

https://github.com/olkhovik/ansible-8.2
