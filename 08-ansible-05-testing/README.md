# Домашняя работа к занятию 8.5 «Тестирование Roles»

## Подготовка к выполнению
1. Установите molecule: `pip3 install "molecule==3.4.0"`
2. Выполните `docker pull aragast/netology:latest` -  это образ с podman, tox и несколькими пайтонами (3.7 и 3.9) внутри

## Основная часть

Наша основная цель - настроить тестирование наших ролей. Задача: сделать сценарии тестирования для vector. Ожидаемый результат: все сценарии успешно проходят тестирование ролей.

### Molecule

1. Запустите  `molecule test -s centos7` внутри корневой директории clickhouse-role, посмотрите на вывод команды.
<details><summary>log</summary>

```
dmitry@Lenovo-B50:~/netology/mnt/08-ansible-05-testing/clickhouse$ molecule test -s centos_8
INFO     centos_8 scenario test matrix: dependency, lint, cleanup, destroy, syntax, create, prepare, converge, idempotence, side_effect, verify, cleanup, destroy
INFO     Performing prerun...
INFO     Guessed /home/dmitry/netology/08-ansible-05-testing/clickhouse as project root directory
INFO     Using /home/dmitry/.cache/ansible-lint/44bbe3/roles/alexeysetevoi.clickhouse symlink to current repository in order to enable Ansible to find the role using its expected full name.
INFO     Added ANSIBLE_ROLES_PATH=~/.ansible/roles:/usr/share/ansible/roles:/etc/ansible/roles:/home/dmitry/.cache/ansible-lint/44bbe3/roles
INFO     Inventory /home/dmitry/netology/08-ansible-05-testing/clickhouse/molecule/centos_8/../resources/inventory/hosts.yml linked to /home/dmitry/.cache/molecule/clickhouse/centos_8/inventory/hosts
INFO     Inventory /home/dmitry/netology/08-ansible-05-testing/clickhouse/molecule/centos_8/../resources/inventory/group_vars/ linked to /home/dmitry/.cache/molecule/clickhouse/centos_8/inventory/group_vars
INFO     Inventory /home/dmitry/netology/08-ansible-05-testing/clickhouse/molecule/centos_8/../resources/inventory/host_vars/ linked to /home/dmitry/.cache/molecule/clickhouse/centos_8/inventory/host_vars
INFO     Running centos_8 > dependency
WARNING  Skipping, missing the requirements file.
WARNING  Skipping, missing the requirements file.
INFO     Inventory /home/dmitry/netology/08-ansible-05-testing/clickhouse/molecule/centos_8/../resources/inventory/hosts.yml linked to /home/dmitry/.cache/molecule/clickhouse/centos_8/inventory/hosts
INFO     Inventory /home/dmitry/netology/08-ansible-05-testing/clickhouse/molecule/centos_8/../resources/inventory/group_vars/ linked to /home/dmitry/.cache/molecule/clickhouse/centos_8/inventory/group_vars
INFO     Inventory /home/dmitry/netology/08-ansible-05-testing/clickhouse/molecule/centos_8/../resources/inventory/host_vars/ linked to /home/dmitry/.cache/molecule/clickhouse/centos_8/inventory/host_vars
INFO     Running centos_8 > lint
COMMAND: yamllint .
ansible-lint
flake8

./vector-role/molecule/resources/verify.yml
  44:1      error    too many blank lines (1 > 0)  (empty-lines)

WARNING: PATH altered to include /usr/bin
WARNING  Loading custom .yamllint config file, this extends our internal yamllint config.
WARNING  Listing 3 violation(s) that are fatal
risky-file-permissions: File permissions unset or incorrect
tasks/install/apt.yml:45 Task/Handler: Hold specified version during APT upgrade | Package installation

no-changed-when: Commands should not change things if nothing needs doing
vector-role/molecule/resources/verify.yml:11 Task/Handler: Validate Config

yaml: too many blank lines (1 > 0) (empty-lines)
vector-role/molecule/resources/verify.yml:44

You can skip specific rules or tags by adding them to your configuration file:
# .ansible-lint
warn_list:  # or 'skip_list' to silence them completely
  - experimental  # all rules tagged as experimental
  - no-changed-when  # Commands should not change things if nothing needs doing
  - yaml  # Violations reported by yamllint

Finished with 2 failure(s), 1 warning(s) on 72 files.
/bin/bash: line 2: flake8: command not found
CRITICAL Lint failed with error code 127
WARNING  An error occurred during the test sequence action: 'lint'. Cleaning up.
INFO     Inventory /home/dmitry/netology/08-ansible-05-testing/clickhouse/molecule/centos_8/../resources/inventory/hosts.yml linked to /home/dmitry/.cache/molecule/clickhouse/centos_8/inventory/hosts
INFO     Inventory /home/dmitry/netology/08-ansible-05-testing/clickhouse/molecule/centos_8/../resources/inventory/group_vars/ linked to /home/dmitry/.cache/molecule/clickhouse/centos_8/inventory/group_vars
INFO     Inventory /home/dmitry/netology/08-ansible-05-testing/clickhouse/molecule/centos_8/../resources/inventory/host_vars/ linked to /home/dmitry/.cache/molecule/clickhouse/centos_8/inventory/host_vars
INFO     Running centos_8 > cleanup
WARNING  Skipping, cleanup playbook not configured.
INFO     Inventory /home/dmitry/netology/08-ansible-05-testing/clickhouse/molecule/centos_8/../resources/inventory/hosts.yml linked to /home/dmitry/.cache/molecule/clickhouse/centos_8/inventory/hosts
INFO     Inventory /home/dmitry/netology/08-ansible-05-testing/clickhouse/molecule/centos_8/../resources/inventory/group_vars/ linked to /home/dmitry/.cache/molecule/clickhouse/centos_8/inventory/group_vars
INFO     Inventory /home/dmitry/netology/08-ansible-05-testing/clickhouse/molecule/centos_8/../resources/inventory/host_vars/ linked to /home/dmitry/.cache/molecule/clickhouse/centos_8/inventory/host_vars
INFO     Running centos_8 > destroy
INFO     Sanity checks: 'docker'

PLAY [Destroy] *****************************************************************

TASK [Destroy molecule instance(s)] ********************************************
changed: [localhost] => (item=centos_8)

TASK [Wait for instance(s) deletion to complete] *******************************
FAILED - RETRYING: [localhost]: Wait for instance(s) deletion to complete (300 retries left).
ok: [localhost] => (item=centos_8)

TASK [Delete docker networks(s)] ***********************************************

PLAY RECAP *********************************************************************
localhost                  : ok=2    changed=1    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0

INFO     Pruning extra files from scenario ephemeral directory
```
</details>

2. Перейдите в каталог с ролью vector-role и создайте сценарий тестирования по умолчанию при помощи `molecule init scenario --driver-name docker`.
```
dmitry@Lenovo-B50:~/netology/mnt/08-ansible-05-testing/vector-role$ molecule init scenario --driver-name docker
INFO     Initializing new scenario default...
INFO     Initialized scenario in /home/dmitry/netology/mnt/08-ansible-05-testing/vector-role/molecule/default successfully.
```
3. Добавьте несколько разных дистрибутивов (centos:8, ubuntu:latest) для инстансов и протестируйте роль, исправьте найденные ошибки, если они есть.
```
...
platforms:
  - name: centos7
    #image: docker.io/pycontribs/centos:7
    image: centos:7
    dockerfile: ../resources/Dockerfile.j2
    privileged: true
    #pre_build_image: true
    command: /sbin/init
  - name: centos8
    image: centos:8
    dockerfile: ../resources/Dockerfile.j2
    privileged: true
    command: /sbin/init
  - name: ubuntu
    image: ubuntu:focal
    dockerfile: ../resources/Dockerfile.j2
    privileged: true
    command: /sbin/init
```
<details><summary>log</summary>

```
dmitry@Lenovo-B50:~/netology/mnt/08-ansible-05-testing/vector-role$ molecule test
INFO     default scenario test matrix: dependency, lint, cleanup, destroy, syntax, create, prepare, converge, idempotence, side_effect, verify, cleanup, destroy
INFO     Performing prerun...
INFO     Guessed /home/dmitry/netology/08-ansible-05-testing/vector-role as project root directory
WARNING  Computed fully qualified role name of vector_role does not follow current galaxy requirements.
Please edit meta/main.yml and assure we can correctly determine full role name:

galaxy_info:
role_name: my_name  # if absent directory name hosting role is used instead
namespace: my_galaxy_namespace  # if absent, author is used instead

Namespace: https://galaxy.ansible.com/docs/contributing/namespaces.html#galaxy-namespace-limitations
Role: https://galaxy.ansible.com/docs/contributing/creating_role.html#role-names

As an alternative, you can add 'role-name' to either skip_list or warn_list.

INFO     Using /home/dmitry/.cache/ansible-lint/dd0813/roles/vector_role symlink to current repository in order to enable Ansible to find the role using its expected full name.
INFO     Added ANSIBLE_ROLES_PATH=~/.ansible/roles:/usr/share/ansible/roles:/etc/ansible/roles:/home/dmitry/.cache/ansible-lint/dd0813/roles
INFO     Running default > dependency
WARNING  Skipping, missing the requirements file.
WARNING  Skipping, missing the requirements file.
INFO     Running default > lint
INFO     Lint is disabled.
INFO     Running default > cleanup
WARNING  Skipping, cleanup playbook not configured.
INFO     Running default > destroy
INFO     Sanity checks: 'docker'

PLAY [Destroy] *****************************************************************

TASK [Destroy molecule instance(s)] ********************************************
changed: [localhost] => (item=centos7)
changed: [localhost] => (item=centos8)
changed: [localhost] => (item=ubuntu)

TASK [Wait for instance(s) deletion to complete] *******************************
ok: [localhost] => (item=centos7)
ok: [localhost] => (item=centos8)
ok: [localhost] => (item=ubuntu)

TASK [Delete docker networks(s)] ***********************************************

PLAY RECAP *********************************************************************
localhost                  : ok=2    changed=1    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0

INFO     Running default > syntax

playbook: /home/dmitry/netology/08-ansible-05-testing/vector-role/molecule/default/converge.yml
INFO     Running default > create

PLAY [Create] ******************************************************************

TASK [Log into a Docker registry] **********************************************
skipping: [localhost] => (item=None)
skipping: [localhost] => (item=None)
skipping: [localhost] => (item=None)
skipping: [localhost]

TASK [Check presence of custom Dockerfiles] ************************************
ok: [localhost] => (item={'command': '/sbin/init', 'dockerfile': '../resources/Dockerfile.j2', 'image': 'centos:7', 'name': 'centos7', 'privileged': True})
ok: [localhost] => (item={'command': '/sbin/init', 'dockerfile': '../resources/Dockerfile.j2', 'image': 'centos:8', 'name': 'centos8', 'privileged': True})
ok: [localhost] => (item={'capabilities': ['SYS_ADMIN'], 'command': '/sbin/init', 'dockerfile': '../resources/Dockerfile.j2', 'env': {'ANSIBLE_USER': 'ansible', 'DEPLOY_GROUP': 'deployer', 'SUDO_GROUP': 'sudo', 'container': 'docker'}, 'image': 'ubuntu:focal', 'name': 'ubuntu', 'privileged': True, 'tmpfs': ['/run', '/tmp'], 'volumes': ['/sys/fs/cgroup:/sys/fs/cgroup']})

TASK [Create Dockerfiles from image names] *************************************
changed: [localhost] => (item={'command': '/sbin/init', 'dockerfile': '../resources/Dockerfile.j2', 'image': 'centos:7', 'name': 'centos7', 'privileged': True})
changed: [localhost] => (item={'command': '/sbin/init', 'dockerfile': '../resources/Dockerfile.j2', 'image': 'centos:8', 'name': 'centos8', 'privileged': True})
changed: [localhost] => (item={'capabilities': ['SYS_ADMIN'], 'command': '/sbin/init', 'dockerfile': '../resources/Dockerfile.j2', 'env': {'ANSIBLE_USER': 'ansible', 'DEPLOY_GROUP': 'deployer', 'SUDO_GROUP': 'sudo', 'container': 'docker'}, 'image': 'ubuntu:focal', 'name': 'ubuntu', 'privileged': True, 'tmpfs': ['/run', '/tmp'], 'volumes': ['/sys/fs/cgroup:/sys/fs/cgroup']})

TASK [Discover local Docker images] ********************************************
ok: [localhost] => (item={'diff': [], 'dest': '/home/dmitry/.cache/molecule/vector-role/default/Dockerfile_centos_7', 'src': '/home/dmitry/.ansible/tmp/ansible-tmp-1656675044.5817084-191640-219749928376920/source', 'md5sum': 'fd341b6280f87a872bf8890df5c0bc92', 'checksum': '233a8b9aa85a4fe93b09e62ce108520490ad00e9', 'changed': True, 'uid': 1000, 'gid': 1000, 'owner': 'dmitry', 'group': 'dmitry', 'mode': '0600', 'state': 'file', 'size': 3648, 'invocation': {'module_args': {'src': '/home/dmitry/.ansible/tmp/ansible-tmp-1656675044.5817084-191640-219749928376920/source', 'dest': '/home/dmitry/.cache/molecule/vector-role/default/Dockerfile_centos_7', 'mode': '0600', 'follow': False, '_original_basename': 'Dockerfile.j2', 'checksum': '233a8b9aa85a4fe93b09e62ce108520490ad00e9', 'backup': False, 'force': True, 'unsafe_writes': False, 'content': None, 'validate': None, 'directory_mode': None, 'remote_src': None, 'local_follow': None, 'owner': None, 'group': None, 'seuser': None, 'serole': None, 'selevel': None, 'setype': None, 'attributes': None}}, 'failed': False, 'item': {'command': '/sbin/init', 'dockerfile': '../resources/Dockerfile.j2', 'image': 'centos:7', 'name': 'centos7', 'privileged': True}, 'ansible_loop_var': 'item', 'i': 0, 'ansible_index_var': 'i'})
ok: [localhost] => (item={'diff': [], 'dest': '/home/dmitry/.cache/molecule/vector-role/default/Dockerfile_centos_8', 'src': '/home/dmitry/.ansible/tmp/ansible-tmp-1656675045.9925463-191640-174953800790317/source', 'md5sum': '1971c75923921dda4956df46c7fc61ce', 'checksum': '68cb441e1c731de4a0b9f9c093eaa6faccd7a333', 'changed': True, 'uid': 1000, 'gid': 1000, 'owner': 'dmitry', 'group': 'dmitry', 'mode': '0600', 'state': 'file', 'size': 3648, 'invocation': {'module_args': {'src': '/home/dmitry/.ansible/tmp/ansible-tmp-1656675045.9925463-191640-174953800790317/source', 'dest': '/home/dmitry/.cache/molecule/vector-role/default/Dockerfile_centos_8', 'mode': '0600', 'follow': False, '_original_basename': 'Dockerfile.j2', 'checksum': '68cb441e1c731de4a0b9f9c093eaa6faccd7a333', 'backup': False, 'force': True, 'unsafe_writes': False, 'content': None, 'validate': None, 'directory_mode': None, 'remote_src': None, 'local_follow': None, 'owner': None, 'group': None, 'seuser': None, 'serole': None, 'selevel': None, 'setype': None, 'attributes': None}}, 'failed': False, 'item': {'command': '/sbin/init', 'dockerfile': '../resources/Dockerfile.j2', 'image': 'centos:8', 'name': 'centos8', 'privileged': True}, 'ansible_loop_var': 'item', 'i': 1, 'ansible_index_var': 'i'})
ok: [localhost] => (item={'diff': [], 'dest': '/home/dmitry/.cache/molecule/vector-role/default/Dockerfile_ubuntu_focal', 'src': '/home/dmitry/.ansible/tmp/ansible-tmp-1656675047.0945094-191640-65910684648996/source', 'md5sum': 'b4a918fa72ee5c0f06ad8874732cc36b', 'checksum': 'b4ca383a98d11113e5527d880ec6454d6048ca64', 'changed': True, 'uid': 1000, 'gid': 1000, 'owner': 'dmitry', 'group': 'dmitry', 'mode': '0600', 'state': 'file', 'size': 3744, 'invocation': {'module_args': {'src': '/home/dmitry/.ansible/tmp/ansible-tmp-1656675047.0945094-191640-65910684648996/source', 'dest': '/home/dmitry/.cache/molecule/vector-role/default/Dockerfile_ubuntu_focal', 'mode': '0600', 'follow': False, '_original_basename': 'Dockerfile.j2', 'checksum': 'b4ca383a98d11113e5527d880ec6454d6048ca64', 'backup': False, 'force': True, 'unsafe_writes': False, 'content': None, 'validate': None, 'directory_mode': None, 'remote_src': None, 'local_follow': None, 'owner': None, 'group': None, 'seuser': None, 'serole': None, 'selevel': None, 'setype': None, 'attributes': None}}, 'failed': False, 'item': {'capabilities': ['SYS_ADMIN'], 'command': '/sbin/init', 'dockerfile': '../resources/Dockerfile.j2', 'env': {'ANSIBLE_USER': 'ansible', 'DEPLOY_GROUP': 'deployer', 'SUDO_GROUP': 'sudo', 'container': 'docker'}, 'image': 'ubuntu:focal', 'name': 'ubuntu', 'privileged': True, 'tmpfs': ['/run', '/tmp'], 'volumes': ['/sys/fs/cgroup:/sys/fs/cgroup']}, 'ansible_loop_var': 'item', 'i': 2, 'ansible_index_var': 'i'})

TASK [Build an Ansible compatible image (new)] *********************************
ok: [localhost] => (item=molecule_local/centos:7)
ok: [localhost] => (item=molecule_local/centos:8)
ok: [localhost] => (item=molecule_local/ubuntu:focal)

TASK [Create docker network(s)] ************************************************

TASK [Determine the CMD directives] ********************************************
ok: [localhost] => (item={'command': '/sbin/init', 'dockerfile': '../resources/Dockerfile.j2', 'image': 'centos:7', 'name': 'centos7', 'privileged': True})
ok: [localhost] => (item={'command': '/sbin/init', 'dockerfile': '../resources/Dockerfile.j2', 'image': 'centos:8', 'name': 'centos8', 'privileged': True})
ok: [localhost] => (item={'capabilities': ['SYS_ADMIN'], 'command': '/sbin/init', 'dockerfile': '../resources/Dockerfile.j2', 'env': {'ANSIBLE_USER': 'ansible', 'DEPLOY_GROUP': 'deployer', 'SUDO_GROUP': 'sudo', 'container': 'docker'}, 'image': 'ubuntu:focal', 'name': 'ubuntu', 'privileged': True, 'tmpfs': ['/run', '/tmp'], 'volumes': ['/sys/fs/cgroup:/sys/fs/cgroup']})

TASK [Create molecule instance(s)] *********************************************
changed: [localhost] => (item=centos7)
changed: [localhost] => (item=centos8)
changed: [localhost] => (item=ubuntu)

TASK [Wait for instance(s) creation to complete] *******************************
changed: [localhost] => (item={'failed': 0, 'started': 1, 'finished': 0, 'ansible_job_id': '583808146411.192040', 'results_file': '/home/dmitry/.ansible_async/583808146411.192040', 'changed': True, 'item': {'command': '/sbin/init', 'dockerfile': '../resources/Dockerfile.j2', 'image': 'centos:7', 'name': 'centos7', 'privileged': True}, 'ansible_loop_var': 'item'})
changed: [localhost] => (item={'failed': 0, 'started': 1, 'finished': 0, 'ansible_job_id': '618961707710.192067', 'results_file': '/home/dmitry/.ansible_async/618961707710.192067', 'changed': True, 'item': {'command': '/sbin/init', 'dockerfile': '../resources/Dockerfile.j2', 'image': 'centos:8', 'name': 'centos8', 'privileged': True}, 'ansible_loop_var': 'item'})
changed: [localhost] => (item={'failed': 0, 'started': 1, 'finished': 0, 'ansible_job_id': '995610802968.192094', 'results_file': '/home/dmitry/.ansible_async/995610802968.192094', 'changed': True, 'item': {'capabilities': ['SYS_ADMIN'], 'command': '/sbin/init', 'dockerfile': '../resources/Dockerfile.j2', 'env': {'ANSIBLE_USER': 'ansible', 'DEPLOY_GROUP': 'deployer', 'SUDO_GROUP': 'sudo', 'container': 'docker'}, 'image': 'ubuntu:focal', 'name': 'ubuntu', 'privileged': True, 'tmpfs': ['/run', '/tmp'], 'volumes': ['/sys/fs/cgroup:/sys/fs/cgroup']}, 'ansible_loop_var': 'item'})

PLAY RECAP *********************************************************************
localhost                  : ok=7    changed=3    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0

INFO     Running default > prepare
WARNING  Skipping, prepare playbook not configured.
INFO     Running default > converge

PLAY [Converge] ****************************************************************

TASK [Gathering Facts] *********************************************************
ok: [ubuntu]
ok: [centos8]
ok: [centos7]

TASK [Include vector-role] *****************************************************

TASK [vector-role : Get Vector distrib] ****************************************
skipping: [ubuntu]
changed: [centos8]
changed: [centos7]

TASK [vector-role : Get Vector distrib] ****************************************
skipping: [centos7]
skipping: [centos8]
changed: [ubuntu]

TASK [vector-role : Install vector packages rpm] *******************************
skipping: [ubuntu]
changed: [centos7]
changed: [centos8]

TASK [vector-role : Install vector packages deb] *******************************
skipping: [centos7]
skipping: [centos8]
changed: [ubuntu]

TASK [vector-role : Copy config] ***********************************************
changed: [ubuntu]
changed: [centos7]
changed: [centos8]

TASK [vector-role : vector service] ********************************************
changed: [centos7]
changed: [ubuntu]
changed: [centos8]

TASK [vector-role : start vector] **********************************************
changed: [ubuntu]
changed: [centos7]
changed: [centos8]

TASK [vector-role : Make log dir] **********************************************
changed: [centos7]
changed: [ubuntu]
changed: [centos8]

TASK [vector-role : Check if file exists] **************************************
ok: [centos7]
ok: [centos8]
ok: [ubuntu]

TASK [vector-role : Make log file] *********************************************
changed: [centos7]
changed: [ubuntu]
changed: [centos8]

TASK [vector-role : copy rsyslog config] ***************************************
changed: [centos7]
changed: [ubuntu]
changed: [centos8]

RUNNING HANDLER [vector-role : start_rsyslog] **********************************
changed: [ubuntu]
changed: [centos8]
changed: [centos7]

PLAY RECAP *********************************************************************
centos7                    : ok=11   changed=9    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0
centos8                    : ok=11   changed=9    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0
ubuntu                     : ok=11   changed=9    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0

INFO     Running default > idempotence

PLAY [Converge] ****************************************************************

TASK [Gathering Facts] *********************************************************
ok: [ubuntu]
ok: [centos8]
ok: [centos7]

TASK [Include vector-role] *****************************************************

TASK [vector-role : Get Vector distrib] ****************************************
skipping: [ubuntu]
ok: [centos7]
ok: [centos8]

TASK [vector-role : Get Vector distrib] ****************************************
skipping: [centos7]
skipping: [centos8]
ok: [ubuntu]

TASK [vector-role : Install vector packages rpm] *******************************
skipping: [ubuntu]
ok: [centos7]
ok: [centos8]

TASK [vector-role : Install vector packages deb] *******************************
skipping: [centos7]
skipping: [centos8]
ok: [ubuntu]

TASK [vector-role : Copy config] ***********************************************
ok: [centos7]
ok: [ubuntu]
ok: [centos8]

TASK [vector-role : vector service] ********************************************
ok: [centos7]
ok: [ubuntu]
ok: [centos8]

TASK [vector-role : start vector] **********************************************
ok: [ubuntu]
ok: [centos8]
ok: [centos7]

TASK [vector-role : Make log dir] **********************************************
ok: [centos7]
ok: [centos8]
ok: [ubuntu]

TASK [vector-role : Check if file exists] **************************************
ok: [centos7]
ok: [ubuntu]
ok: [centos8]

TASK [vector-role : Make log file] *********************************************
skipping: [centos7]
skipping: [centos8]
skipping: [ubuntu]

TASK [vector-role : copy rsyslog config] ***************************************
ok: [centos7]
ok: [ubuntu]
ok: [centos8]

PLAY RECAP *********************************************************************
centos7                    : ok=9    changed=0    unreachable=0    failed=0    skipped=3    rescued=0    ignored=0
centos8                    : ok=9    changed=0    unreachable=0    failed=0    skipped=3    rescued=0    ignored=0
ubuntu                     : ok=9    changed=0    unreachable=0    failed=0    skipped=3    rescued=0    ignored=0

INFO     Idempotence completed successfully.
INFO     Running default > side_effect
WARNING  Skipping, side effect playbook not configured.
INFO     Running default > verify
INFO     Running Ansible Verifier

PLAY [Verify] ******************************************************************

TASK [Example assertion] *******************************************************
ok: [centos7] => {
    "changed": false,
    "msg": "All assertions passed"
}
ok: [centos8] => {
    "changed": false,
    "msg": "All assertions passed"
}
ok: [ubuntu] => {
    "changed": false,
    "msg": "All assertions passed"
}

PLAY RECAP *********************************************************************
centos7                    : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
centos8                    : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
ubuntu                     : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

INFO     Verifier completed successfully.
INFO     Running default > cleanup
WARNING  Skipping, cleanup playbook not configured.
INFO     Running default > destroy

PLAY [Destroy] *****************************************************************

TASK [Destroy molecule instance(s)] ********************************************
changed: [localhost] => (item=centos7)
changed: [localhost] => (item=centos8)
changed: [localhost] => (item=ubuntu)

TASK [Wait for instance(s) deletion to complete] *******************************
changed: [localhost] => (item=centos7)
changed: [localhost] => (item=centos8)
changed: [localhost] => (item=ubuntu)

TASK [Delete docker networks(s)] ***********************************************

PLAY RECAP *********************************************************************
localhost                  : ok=2    changed=2    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0

INFO     Pruning extra files from scenario ephemeral directory
```
</details>

4. Добавьте несколько assert'ов в verify.yml файл для  проверки работоспособности vector-role (проверка, что конфиг валидный, проверка успешности запуска, etc). Запустите тестирование роли повторно и проверьте, что оно прошло успешно.

`verify.yml`:
```
- name: Verify
  hosts: all
  gather_facts: false
  vars:
    vector_config: /etc/vector/vector.yaml
    vector_package: vector
  tasks:
  - name: Validate Config
    ansible.builtin.command: vector validate "{{ vector_config }}"
    register: validation_status
  - name: Assert Config Validation Status
    assert:
      that:
        - validation_status.rc == 0
      fail_msg: "Config is not valid"
  - name: Collect Facts About System Services
    ansible.builtin.service_facts:
    register: service_state
  - name: Assert Systemd Unit Status
    assert:
      that:
        - "'enabled' == service_state['ansible_facts']['services']['vector.service']['status']"
      fail_msg: "vector.service is not enabled"
  - name: Assert Systemd Unit State
    assert:
      that:
        - "'running' == service_state['ansible_facts']['services']['vector.service']['state']"
      fail_msg: "vector.service is not running"
```
<details><summary>log</summary>

```
dmitry@Lenovo-B50:~/netology/mnt/08-ansible-05-testing/vector-role$ molecule test
INFO     default scenario test matrix: dependency, lint, cleanup, destroy, syntax, create, prepare, converge, idempotence, side_effect, verify, cleanup, destroy
INFO     Performing prerun...
INFO     Guessed /home/dmitry/netology/08-ansible-05-testing/vector-role as project root directory
WARNING  Computed fully qualified role name of vector_role does not follow current galaxy requirements.
Please edit meta/main.yml and assure we can correctly determine full role name:

galaxy_info:
role_name: my_name  # if absent directory name hosting role is used instead
namespace: my_galaxy_namespace  # if absent, author is used instead

Namespace: https://galaxy.ansible.com/docs/contributing/namespaces.html#galaxy-namespace-limitations
Role: https://galaxy.ansible.com/docs/contributing/creating_role.html#role-names

As an alternative, you can add 'role-name' to either skip_list or warn_list.

INFO     Using /home/dmitry/.cache/ansible-lint/dd0813/roles/vector_role symlink to current repository in order to enable Ansible to find the role using its expected full name.
INFO     Added ANSIBLE_ROLES_PATH=~/.ansible/roles:/usr/share/ansible/roles:/etc/ansible/roles:/home/dmitry/.cache/ansible-lint/dd0813/roles
INFO     Running default > dependency
WARNING  Skipping, missing the requirements file.
WARNING  Skipping, missing the requirements file.
INFO     Running default > lint
INFO     Lint is disabled.
INFO     Running default > cleanup
WARNING  Skipping, cleanup playbook not configured.
INFO     Running default > destroy
INFO     Sanity checks: 'docker'

PLAY [Destroy] *****************************************************************

TASK [Destroy molecule instance(s)] ********************************************
changed: [localhost] => (item=centos7)
changed: [localhost] => (item=centos8)
changed: [localhost] => (item=ubuntu)

TASK [Wait for instance(s) deletion to complete] *******************************
ok: [localhost] => (item=centos7)
ok: [localhost] => (item=centos8)
ok: [localhost] => (item=ubuntu)

TASK [Delete docker networks(s)] ***********************************************

PLAY RECAP *********************************************************************
localhost                  : ok=2    changed=1    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0

INFO     Running default > syntax

playbook: /home/dmitry/netology/08-ansible-05-testing/vector-role/molecule/default/converge.yml
INFO     Running default > create

PLAY [Create] ******************************************************************

TASK [Log into a Docker registry] **********************************************
skipping: [localhost] => (item=None)
skipping: [localhost] => (item=None)
skipping: [localhost] => (item=None)
skipping: [localhost]

TASK [Check presence of custom Dockerfiles] ************************************
ok: [localhost] => (item={'command': '/sbin/init', 'dockerfile': '../resources/Dockerfile.j2', 'image': 'centos:7', 'name': 'centos7', 'privileged': True})
ok: [localhost] => (item={'command': '/sbin/init', 'dockerfile': '../resources/Dockerfile.j2', 'image': 'centos:8', 'name': 'centos8', 'privileged': True})
ok: [localhost] => (item={'capabilities': ['SYS_ADMIN'], 'command': '/sbin/init', 'dockerfile': '../resources/Dockerfile.j2', 'env': {'ANSIBLE_USER': 'ansible', 'DEPLOY_GROUP': 'deployer', 'SUDO_GROUP': 'sudo', 'container': 'docker'}, 'image': 'ubuntu:focal', 'name': 'ubuntu', 'privileged': True, 'tmpfs': ['/run', '/tmp'], 'volumes': ['/sys/fs/cgroup:/sys/fs/cgroup']})

TASK [Create Dockerfiles from image names] *************************************
changed: [localhost] => (item={'command': '/sbin/init', 'dockerfile': '../resources/Dockerfile.j2', 'image': 'centos:7', 'name': 'centos7', 'privileged': True})
changed: [localhost] => (item={'command': '/sbin/init', 'dockerfile': '../resources/Dockerfile.j2', 'image': 'centos:8', 'name': 'centos8', 'privileged': True})
changed: [localhost] => (item={'capabilities': ['SYS_ADMIN'], 'command': '/sbin/init', 'dockerfile': '../resources/Dockerfile.j2', 'env': {'ANSIBLE_USER': 'ansible', 'DEPLOY_GROUP': 'deployer', 'SUDO_GROUP': 'sudo', 'container': 'docker'}, 'image': 'ubuntu:focal', 'name': 'ubuntu', 'privileged': True, 'tmpfs': ['/run', '/tmp'], 'volumes': ['/sys/fs/cgroup:/sys/fs/cgroup']})

TASK [Discover local Docker images] ********************************************
ok: [localhost] => (item={'diff': [], 'dest': '/home/dmitry/.cache/molecule/vector-role/default/Dockerfile_centos_7', 'src': '/home/dmitry/.ansible/tmp/ansible-tmp-1656676169.9174726-205954-117978003699226/source', 'md5sum': 'fd341b6280f87a872bf8890df5c0bc92', 'checksum': '233a8b9aa85a4fe93b09e62ce108520490ad00e9', 'changed': True, 'uid': 1000, 'gid': 1000, 'owner': 'dmitry', 'group': 'dmitry', 'mode': '0600', 'state': 'file', 'size': 3648, 'invocation': {'module_args': {'src': '/home/dmitry/.ansible/tmp/ansible-tmp-1656676169.9174726-205954-117978003699226/source', 'dest': '/home/dmitry/.cache/molecule/vector-role/default/Dockerfile_centos_7', 'mode': '0600', 'follow': False, '_original_basename': 'Dockerfile.j2', 'checksum': '233a8b9aa85a4fe93b09e62ce108520490ad00e9', 'backup': False, 'force': True, 'unsafe_writes': False, 'content': None, 'validate': None, 'directory_mode': None, 'remote_src': None, 'local_follow': None, 'owner': None, 'group': None, 'seuser': None, 'serole': None, 'selevel': None, 'setype': None, 'attributes': None}}, 'failed': False, 'item': {'command': '/sbin/init', 'dockerfile': '../resources/Dockerfile.j2', 'image': 'centos:7', 'name': 'centos7', 'privileged': True}, 'ansible_loop_var': 'item', 'i': 0, 'ansible_index_var': 'i'})
ok: [localhost] => (item={'diff': [], 'dest': '/home/dmitry/.cache/molecule/vector-role/default/Dockerfile_centos_8', 'src': '/home/dmitry/.ansible/tmp/ansible-tmp-1656676171.345127-205954-218242098195063/source', 'md5sum': '1971c75923921dda4956df46c7fc61ce', 'checksum': '68cb441e1c731de4a0b9f9c093eaa6faccd7a333', 'changed': True, 'uid': 1000, 'gid': 1000, 'owner': 'dmitry', 'group': 'dmitry', 'mode': '0600', 'state': 'file', 'size': 3648, 'invocation': {'module_args': {'src': '/home/dmitry/.ansible/tmp/ansible-tmp-1656676171.345127-205954-218242098195063/source', 'dest': '/home/dmitry/.cache/molecule/vector-role/default/Dockerfile_centos_8', 'mode': '0600', 'follow': False, '_original_basename': 'Dockerfile.j2', 'checksum': '68cb441e1c731de4a0b9f9c093eaa6faccd7a333', 'backup': False, 'force': True, 'unsafe_writes': False, 'content': None, 'validate': None, 'directory_mode': None, 'remote_src': None, 'local_follow': None, 'owner': None, 'group': None, 'seuser': None, 'serole': None, 'selevel': None, 'setype': None, 'attributes': None}}, 'failed': False, 'item': {'command': '/sbin/init', 'dockerfile': '../resources/Dockerfile.j2', 'image': 'centos:8', 'name': 'centos8', 'privileged': True}, 'ansible_loop_var': 'item', 'i': 1, 'ansible_index_var': 'i'})
ok: [localhost] => (item={'diff': [], 'dest': '/home/dmitry/.cache/molecule/vector-role/default/Dockerfile_ubuntu_focal', 'src': '/home/dmitry/.ansible/tmp/ansible-tmp-1656676172.4521856-205954-171681238095535/source', 'md5sum': 'b4a918fa72ee5c0f06ad8874732cc36b', 'checksum': 'b4ca383a98d11113e5527d880ec6454d6048ca64', 'changed': True, 'uid': 1000, 'gid': 1000, 'owner': 'dmitry', 'group': 'dmitry', 'mode': '0600', 'state': 'file', 'size': 3744, 'invocation': {'module_args': {'src': '/home/dmitry/.ansible/tmp/ansible-tmp-1656676172.4521856-205954-171681238095535/source', 'dest': '/home/dmitry/.cache/molecule/vector-role/default/Dockerfile_ubuntu_focal', 'mode': '0600', 'follow': False, '_original_basename': 'Dockerfile.j2', 'checksum': 'b4ca383a98d11113e5527d880ec6454d6048ca64', 'backup': False, 'force': True, 'unsafe_writes': False, 'content': None, 'validate': None, 'directory_mode': None, 'remote_src': None, 'local_follow': None, 'owner': None, 'group': None, 'seuser': None, 'serole': None, 'selevel': None, 'setype': None, 'attributes': None}}, 'failed': False, 'item': {'capabilities': ['SYS_ADMIN'], 'command': '/sbin/init', 'dockerfile': '../resources/Dockerfile.j2', 'env': {'ANSIBLE_USER': 'ansible', 'DEPLOY_GROUP': 'deployer', 'SUDO_GROUP': 'sudo', 'container': 'docker'}, 'image': 'ubuntu:focal', 'name': 'ubuntu', 'privileged': True, 'tmpfs': ['/run', '/tmp'], 'volumes': ['/sys/fs/cgroup:/sys/fs/cgroup']}, 'ansible_loop_var': 'item', 'i': 2, 'ansible_index_var': 'i'})

TASK [Build an Ansible compatible image (new)] *********************************
ok: [localhost] => (item=molecule_local/centos:7)
ok: [localhost] => (item=molecule_local/centos:8)
ok: [localhost] => (item=molecule_local/ubuntu:focal)

TASK [Create docker network(s)] ************************************************

TASK [Determine the CMD directives] ********************************************
ok: [localhost] => (item={'command': '/sbin/init', 'dockerfile': '../resources/Dockerfile.j2', 'image': 'centos:7', 'name': 'centos7', 'privileged': True})
ok: [localhost] => (item={'command': '/sbin/init', 'dockerfile': '../resources/Dockerfile.j2', 'image': 'centos:8', 'name': 'centos8', 'privileged': True})
ok: [localhost] => (item={'capabilities': ['SYS_ADMIN'], 'command': '/sbin/init', 'dockerfile': '../resources/Dockerfile.j2', 'env': {'ANSIBLE_USER': 'ansible', 'DEPLOY_GROUP': 'deployer', 'SUDO_GROUP': 'sudo', 'container': 'docker'}, 'image': 'ubuntu:focal', 'name': 'ubuntu', 'privileged': True, 'tmpfs': ['/run', '/tmp'], 'volumes': ['/sys/fs/cgroup:/sys/fs/cgroup']})

TASK [Create molecule instance(s)] *********************************************
changed: [localhost] => (item=centos7)
changed: [localhost] => (item=centos8)
changed: [localhost] => (item=ubuntu)

TASK [Wait for instance(s) creation to complete] *******************************
changed: [localhost] => (item={'failed': 0, 'started': 1, 'finished': 0, 'ansible_job_id': '847686235113.206354', 'results_file': '/home/dmitry/.ansible_async/847686235113.206354', 'changed': True, 'item': {'command': '/sbin/init', 'dockerfile': '../resources/Dockerfile.j2', 'image': 'centos:7', 'name': 'centos7', 'privileged': True}, 'ansible_loop_var': 'item'})
changed: [localhost] => (item={'failed': 0, 'started': 1, 'finished': 0, 'ansible_job_id': '396706946638.206381', 'results_file': '/home/dmitry/.ansible_async/396706946638.206381', 'changed': True, 'item': {'command': '/sbin/init', 'dockerfile': '../resources/Dockerfile.j2', 'image': 'centos:8', 'name': 'centos8', 'privileged': True}, 'ansible_loop_var': 'item'})
changed: [localhost] => (item={'failed': 0, 'started': 1, 'finished': 0, 'ansible_job_id': '830796610996.206408', 'results_file': '/home/dmitry/.ansible_async/830796610996.206408', 'changed': True, 'item': {'capabilities': ['SYS_ADMIN'], 'command': '/sbin/init', 'dockerfile': '../resources/Dockerfile.j2', 'env': {'ANSIBLE_USER': 'ansible', 'DEPLOY_GROUP': 'deployer', 'SUDO_GROUP': 'sudo', 'container': 'docker'}, 'image': 'ubuntu:focal', 'name': 'ubuntu', 'privileged': True, 'tmpfs': ['/run', '/tmp'], 'volumes': ['/sys/fs/cgroup:/sys/fs/cgroup']}, 'ansible_loop_var': 'item'})

PLAY RECAP *********************************************************************
localhost                  : ok=7    changed=3    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0

INFO     Running default > prepare
WARNING  Skipping, prepare playbook not configured.
INFO     Running default > converge

PLAY [Converge] ****************************************************************

TASK [Gathering Facts] *********************************************************
ok: [ubuntu]
ok: [centos8]
ok: [centos7]

TASK [Include vector-role] *****************************************************

TASK [vector-role : Get Vector distrib] ****************************************
skipping: [ubuntu]
changed: [centos8]
changed: [centos7]

TASK [vector-role : Get Vector distrib] ****************************************
skipping: [centos7]
skipping: [centos8]
changed: [ubuntu]

TASK [vector-role : Install vector packages rpm] *******************************
skipping: [ubuntu]
changed: [centos7]
changed: [centos8]

TASK [vector-role : Install vector packages deb] *******************************
skipping: [centos7]
skipping: [centos8]
changed: [ubuntu]

TASK [vector-role : Copy config] ***********************************************
changed: [ubuntu]
changed: [centos7]
changed: [centos8]

TASK [vector-role : vector service] ********************************************
changed: [centos7]
changed: [ubuntu]
changed: [centos8]

TASK [vector-role : start vector] **********************************************
changed: [ubuntu]
changed: [centos7]
changed: [centos8]

TASK [vector-role : Make log dir] **********************************************
changed: [centos7]
changed: [centos8]
changed: [ubuntu]

TASK [vector-role : Check if file exists] **************************************
ok: [centos7]
ok: [centos8]
ok: [ubuntu]

TASK [vector-role : Make log file] *********************************************
changed: [centos7]
changed: [ubuntu]
changed: [centos8]

TASK [vector-role : copy rsyslog config] ***************************************
changed: [centos7]
changed: [ubuntu]
changed: [centos8]

RUNNING HANDLER [vector-role : start_rsyslog] **********************************
changed: [ubuntu]
changed: [centos8]
changed: [centos7]

PLAY RECAP *********************************************************************
centos7                    : ok=11   changed=9    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0
centos8                    : ok=11   changed=9    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0
ubuntu                     : ok=11   changed=9    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0

INFO     Running default > idempotence

PLAY [Converge] ****************************************************************

TASK [Gathering Facts] *********************************************************
ok: [ubuntu]
ok: [centos8]
ok: [centos7]

TASK [Include vector-role] *****************************************************

TASK [vector-role : Get Vector distrib] ****************************************
skipping: [ubuntu]
ok: [centos7]
ok: [centos8]

TASK [vector-role : Get Vector distrib] ****************************************
skipping: [centos7]
skipping: [centos8]
ok: [ubuntu]

TASK [vector-role : Install vector packages rpm] *******************************
skipping: [ubuntu]
ok: [centos7]
ok: [centos8]

TASK [vector-role : Install vector packages deb] *******************************
skipping: [centos7]
skipping: [centos8]
ok: [ubuntu]

TASK [vector-role : Copy config] ***********************************************
ok: [centos7]
ok: [ubuntu]
ok: [centos8]

TASK [vector-role : vector service] ********************************************
ok: [centos7]
ok: [ubuntu]
ok: [centos8]

TASK [vector-role : start vector] **********************************************
ok: [ubuntu]
ok: [centos8]
ok: [centos7]

TASK [vector-role : Make log dir] **********************************************
ok: [centos7]
ok: [centos8]
ok: [ubuntu]

TASK [vector-role : Check if file exists] **************************************
ok: [centos7]
ok: [ubuntu]
ok: [centos8]

TASK [vector-role : Make log file] *********************************************
skipping: [centos7]
skipping: [centos8]
skipping: [ubuntu]

TASK [vector-role : copy rsyslog config] ***************************************
ok: [centos7]
ok: [ubuntu]
ok: [centos8]

PLAY RECAP *********************************************************************
centos7                    : ok=9    changed=0    unreachable=0    failed=0    skipped=3    rescued=0    ignored=0
centos8                    : ok=9    changed=0    unreachable=0    failed=0    skipped=3    rescued=0    ignored=0
ubuntu                     : ok=9    changed=0    unreachable=0    failed=0    skipped=3    rescued=0    ignored=0

INFO     Idempotence completed successfully.
INFO     Running default > side_effect
WARNING  Skipping, side effect playbook not configured.
INFO     Running default > verify
INFO     Running Ansible Verifier

PLAY [Verify] ******************************************************************

TASK [Validate Config] *********************************************************
changed: [ubuntu]
changed: [centos7]
changed: [centos8]

TASK [Assert Config Validation Status] *****************************************
ok: [centos7] => {
    "changed": false,
    "msg": "All assertions passed"
}
ok: [centos8] => {
    "changed": false,
    "msg": "All assertions passed"
}
ok: [ubuntu] => {
    "changed": false,
    "msg": "All assertions passed"
}

TASK [Collect Facts About System Services] *************************************
ok: [ubuntu]
ok: [centos8]
ok: [centos7]

TASK [Assert Systemd Unit Status] **********************************************
ok: [centos7] => {
    "changed": false,
    "msg": "All assertions passed"
}
ok: [centos8] => {
    "changed": false,
    "msg": "All assertions passed"
}
ok: [ubuntu] => {
    "changed": false,
    "msg": "All assertions passed"
}

TASK [Assert Systemd Unit State] ***********************************************
ok: [centos7] => {
    "changed": false,
    "msg": "All assertions passed"
}
ok: [centos8] => {
    "changed": false,
    "msg": "All assertions passed"
}
ok: [ubuntu] => {
    "changed": false,
    "msg": "All assertions passed"
}

PLAY RECAP *********************************************************************
centos7                    : ok=5    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
centos8                    : ok=5    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
ubuntu                     : ok=5    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

INFO     Verifier completed successfully.
INFO     Running default > cleanup
WARNING  Skipping, cleanup playbook not configured.
INFO     Running default > destroy

PLAY [Destroy] *****************************************************************

TASK [Destroy molecule instance(s)] ********************************************
changed: [localhost] => (item=centos7)
changed: [localhost] => (item=centos8)
changed: [localhost] => (item=ubuntu)

TASK [Wait for instance(s) deletion to complete] *******************************
changed: [localhost] => (item=centos7)
changed: [localhost] => (item=centos8)
changed: [localhost] => (item=ubuntu)

TASK [Delete docker networks(s)] ***********************************************

PLAY RECAP *********************************************************************
localhost                  : ok=2    changed=2    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0

INFO     Pruning extra files from scenario ephemeral directory
```
</details>

5. Добавьте новый тег на коммит с рабочим сценарием в соответствии с семантическим версионированием.

https://github.com/olkhovik/vector-role/releases/tag/v0.2.1

### Tox

1. Добавьте в директорию с vector-role файлы из [директории](./example)
2. Запустите `docker run --privileged=True -v <path_to_repo>:/opt/vector-role -w /opt/vector-role -it aragast/netology:latest /bin/bash`, где path_to_repo - путь до корня репозитория с vector-role на вашей файловой системе.
3. Внутри контейнера выполните команду `tox`, посмотрите на вывод.
4. Создайте облегчённый сценарий для `molecule` с драйвером `molecule_podman`. Проверьте его на исполнимость.
5. Пропишите правильную команду в `tox.ini` для того чтобы запускался облегчённый сценарий.
6. Запустите команду `tox`. Убедитесь, что всё отработало успешно.
7. Добавьте новый тег на коммит с рабочим сценарием в соответствии с семантическим версионированием.

https://github.com/olkhovik/vector-role/releases/tag/v0.3.1

После выполнения у вас должен получится один сценарий molecule и один tox.ini файл в репозитории. Ссылка на репозиторий являются ответами на домашнее задание. Не забудьте указать в ответе теги решений Tox и Molecule заданий.

Роль основной части:

https://github.com/olkhovik/vector-role

## Необязательная часть

1. Проделайте схожие манипуляции для создания роли lighthouse.
2. Создайте сценарий внутри любой из своих ролей, который умеет поднимать весь стек при помощи всех ролей.
3. Убедитесь в работоспособности своего стека. Создайте отдельный verify.yml, который будет проверять работоспособность интеграции всех инструментов между ними.
4. Выложите свои roles в репозитории. В ответ приведите ссылки.

