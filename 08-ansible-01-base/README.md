# Домашняя работа к занятию 8.1 «Введение в Ansible»

## Подготовка к выполнению
1. Установите ansible версии 2.10 или выше.
2. Создайте свой собственный публичный репозиторий на github с произвольным именем.
3. Скачайте [playbook](./playbook/) из репозитория с домашним заданием и перенесите его в свой репозиторий.

```
dmitry@Lenovo-B50:~$ ansible --version
ansible [core 2.12.4]
  config file = /etc/ansible/ansible.cfg
  configured module search path = ['/home/dmitry/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python3/dist-packages/ansible
  ansible collection location = /home/dmitry/.ansible/collections:/usr/share/ansible/collections
  executable location = /usr/bin/ansible
  python version = 3.8.10 (default, Mar 15 2022, 12:22:08) [GCC 9.4.0]
  jinja version = 2.10.1
  libyaml = True
```


## Основная часть
1. Попробуйте запустить playbook на окружении из `test.yml`, зафиксируйте какое значение имеет факт `some_fact` для указанного хоста при выполнении playbook'a.
```
dmitry@Lenovo-B50:~/netology/mnt/08-ansible-01-base/playbook$ ansible-playbook site.yml -i inventory/test.yml

PLAY [Print os facts] ***********************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] **********************************************************************************************************************************************************************************************************************
ok: [localhost]

TASK [Print OS] *****************************************************************************************************************************************************************************************************************************
ok: [localhost] => {
    "msg": "Ubuntu"
}

TASK [Print fact] ***************************************************************************************************************************************************************************************************************************
ok: [localhost] => {
    "msg": 12
}

PLAY RECAP **********************************************************************************************************************************************************************************************************************************
localhost                  : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```
2. Найдите файл с переменными (group_vars) в котором задаётся найденное в первом пункте значение и поменяйте его на 'all default fact'.
```
dmitry@Lenovo-B50:~/netology/mnt/08-ansible-01-base/playbook$ cat group_vars/all/examp.yml
---
  some_fact: "all default fact"
```
3. Воспользуйтесь подготовленным (используется `docker`) или создайте собственное окружение для проведения дальнейших испытаний.
```
dmitry@Lenovo-B50:~/netology/mnt/08-ansible-01-base/playbook$ docker ps
CONTAINER ID   IMAGE                      COMMAND                  CREATED         STATUS         PORTS     NAMES
4bfae4206127   pycontribs/ubuntu:latest   "tail -F /var/log/sy…"   2 minutes ago   Up 7 seconds             ubuntu
3f8f159557f6   centos:7                   "tail -F /var/log/me…"   2 minutes ago   Up 7 seconds             centos7
```
4. Проведите запуск playbook на окружении из `prod.yml`. Зафиксируйте полученные значения `some_fact` для каждого из `managed host`.
```
dmitry@Lenovo-B50:~/netology/mnt/08-ansible-01-base/playbook$ ansible-playbook site.yml -i inventory/prod.yml

PLAY [Print os facts] ***********************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] **********************************************************************************************************************************************************************************************************************
ok: [ubuntu]
ok: [centos7]

TASK [Print OS] *****************************************************************************************************************************************************************************************************************************
ok: [centos7] => {
    "msg": "CentOS"
}
ok: [ubuntu] => {
    "msg": "Ubuntu"
}

TASK [Print fact] ***************************************************************************************************************************************************************************************************************************
ok: [centos7] => {
    "msg": "el"
}
ok: [ubuntu] => {
    "msg": "deb"
}

PLAY RECAP **********************************************************************************************************************************************************************************************************************************
centos7                    : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
ubuntu                     : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```
5. Добавьте факты в `group_vars` каждой из групп хостов так, чтобы для `some_fact` получились следующие значения: для `deb` - 'deb default fact', для `el` - 'el default fact'.
```
dmitry@Lenovo-B50:~/netology/mnt/08-ansible-01-base/playbook$ cat group_vars/{deb,el}/*
---
  some_fact: "deb default fact"
---
  some_fact: "el default fact"
```
6. Повторите запуск playbook на окружении `prod.yml`. Убедитесь, что выдаются корректные значения для всех хостов.
```
dmitry@Lenovo-B50:~/netology/mnt/08-ansible-01-base/playbook$ ansible-playbook site.yml -i inventory/prod.yml

PLAY [Print os facts] ***********************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] **********************************************************************************************************************************************************************************************************************
ok: [ubuntu]
ok: [centos7]

TASK [Print OS] *****************************************************************************************************************************************************************************************************************************
ok: [centos7] => {
    "msg": "CentOS"
}
ok: [ubuntu] => {
    "msg": "Ubuntu"
}

TASK [Print fact] ***************************************************************************************************************************************************************************************************************************
ok: [centos7] => {
    "msg": "el default fact"
}
ok: [ubuntu] => {
    "msg": "deb default fact"
}

PLAY RECAP **********************************************************************************************************************************************************************************************************************************
centos7                    : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
ubuntu                     : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```
7. При помощи `ansible-vault` зашифруйте факты в `group_vars/deb` и `group_vars/el` с паролем `netology`.
```
dmitry@Lenovo-B50:~/netology/mnt/08-ansible-01-base/playbook$ ansible-vault encrypt group_vars/deb/examp.yml
New Vault password:
Confirm New Vault password:
Encryption successful
dmitry@Lenovo-B50:~/netology/mnt/08-ansible-01-base/playbook$ ansible-vault encrypt group_vars/el/examp.yml
New Vault password:
Confirm New Vault password:
Encryption successful
dmitry@Lenovo-B50:~/netology/mnt/08-ansible-01-base/playbook$ cat group_vars/{deb,el}/*
$ANSIBLE_VAULT;1.1;AES256
63336437396461646464656136626432313334306561313166666437613734633962626634353066
3731343531623466326530653762306563363139376363640a313662346533313464623231376632
65303839323133333464373166326463613665353462366163666538326235343230613131376635
6166656539626233370a346435653038306263343164356234633331643637323635383765663764
35653366643837656236653663386262666266313865666630373064363462633035626236626463
3838653962656264373663616334656631376532383830633465
$ANSIBLE_VAULT;1.1;AES256
35333035323863653631323633663466363836373161613166383364386564636232376336653332
6337303264393935343837373731316333326633313431370a383763326563636131643061373532
30383761363862373662376637376639333435643232343335653533653535316361393838613338
6434313736313834350a366633363637366534656235303538353330333836326536613261393961
32336164356564623362373963663062383039663930346536626638626166313362646161356239
6463353361366535323932393631613437646531343230346363
```
8. Запустите playbook на окружении `prod.yml`. При запуске `ansible` должен запросить у вас пароль. Убедитесь в работоспособности.
```
dmitry@Lenovo-B50:~/netology/mnt/08-ansible-01-base/playbook$ ansible-playbook site.yml -i inventory/prod.yml --ask-vault-pass
Vault password:

PLAY [Print os facts] ***********************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] **********************************************************************************************************************************************************************************************************************
ok: [ubuntu]
ok: [centos7]

TASK [Print OS] *****************************************************************************************************************************************************************************************************************************
ok: [centos7] => {
    "msg": "CentOS"
}
ok: [ubuntu] => {
    "msg": "Ubuntu"
}

TASK [Print fact] ***************************************************************************************************************************************************************************************************************************
ok: [centos7] => {
    "msg": "el default fact"
}
ok: [ubuntu] => {
    "msg": "deb default fact"
}

PLAY RECAP **********************************************************************************************************************************************************************************************************************************
centos7                    : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
ubuntu                     : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```
9. Посмотрите при помощи `ansible-doc` список плагинов для подключения. Выберите подходящий для работы на `control node`.
```
dmitry@Lenovo-B50:~/netology/mnt/08-ansible-01-base/playbook$ ansible-doc -l -t connection
...
local                          execute on controller
```
10. В `prod.yml` добавьте новую группу хостов с именем  `local`, в ней разместите localhost с необходимым типом подключения.
```
dmitry@Lenovo-B50:~/netology/mnt/08-ansible-01-base/playbook$ cat inventory/prod.yml
---
  el:
    hosts:
      centos7:
        ansible_connection: docker
  deb:
    hosts:
      ubuntu:
        ansible_connection: docker
  local:
    hosts:
      localhost:
        ansible_connection: local
```
11. Запустите playbook на окружении `prod.yml`. При запуске `ansible` должен запросить у вас пароль. Убедитесь что факты `some_fact` для каждого из хостов определены из верных `group_vars`.
```
dmitry@Lenovo-B50:~/netology/mnt/08-ansible-01-base/playbook$ ansible-playbook site.yml -i inventory/prod.yml --ask-vault-pass
Vault password:

PLAY [Print os facts] ***********************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] **********************************************************************************************************************************************************************************************************************
ok: [localhost]
ok: [ubuntu]
ok: [centos7]

TASK [Print OS] *****************************************************************************************************************************************************************************************************************************
ok: [localhost] => {
    "msg": "Ubuntu"
}
ok: [centos7] => {
    "msg": "CentOS"
}
ok: [ubuntu] => {
    "msg": "Ubuntu"
}

TASK [Print fact] ***************************************************************************************************************************************************************************************************************************
ok: [localhost] => {
    "msg": "all default fact"
}
ok: [centos7] => {
    "msg": "el default fact"
}
ok: [ubuntu] => {
    "msg": "deb default fact"
}

PLAY RECAP **********************************************************************************************************************************************************************************************************************************
centos7                    : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
localhost                  : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
ubuntu                     : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```
12. Заполните `README.md` ответами на вопросы. Сделайте `git push` в ветку `master`. В ответе отправьте ссылку на ваш открытый репозиторий с изменённым `playbook` и заполненным `README.md`.

https://github.com/olkhovik/ansible-8.1

## Необязательная часть

1. При помощи `ansible-vault` расшифруйте все зашифрованные файлы с переменными.
```
dmitry@Lenovo-B50:~/netology/mnt/08-ansible-01-base/playbook$ ansible-vault decrypt group_vars/deb/examp.yml
Vault password:
Decryption successful
dmitry@Lenovo-B50:~/netology/mnt/08-ansible-01-base/playbook$ cat group_vars/deb/examp.yml
---
  some_fact: "deb default fact"
dmitry@Lenovo-B50:~/netology/mnt/08-ansible-01-base/playbook$ ansible-vault decrypt group_vars/el/examp.yml
Vault password:
Decryption successful
dmitry@Lenovo-B50:~/netology/mnt/08-ansible-01-base/playbook$ cat group_vars/el/examp.yml
---
  some_fact: "el default fact"
```
2. Зашифруйте отдельное значение `PaSSw0rd` для переменной `some_fact` паролем `netology`. Добавьте полученное значение в `group_vars/all/exmp.yml`.
```
dmitry@Lenovo-B50:~/netology/mnt/08-ansible-01-base/playbook$ ansible-vault encrypt_string PaSSw0rd
New Vault password:
Confirm New Vault password:
!vault |
          $ANSIBLE_VAULT;1.1;AES256
          36643735366465363738346630633237303061636662613163623262313163376237623939383333
          6430346262653161656430353530626466616661633965300a626334353665303332303036383535
          38303634323162303061356331336361653737646266626533323832616535373965633634306539
          3261643865643638630a303630646631646165353662633064306531303833316638386162333230
          3064
Encryption successful
```
```
dmitry@Lenovo-B50:~/netology/mnt/08-ansible-01-base/playbook$ cat group_vars/all/examp.yml
---
  some_fact: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          36643735366465363738346630633237303061636662613163623262313163376237623939383333
          6430346262653161656430353530626466616661633965300a626334353665303332303036383535
          38303634323162303061356331336361653737646266626533323832616535373965633634306539
          3261643865643638630a303630646631646165353662633064306531303833316638386162333230
          3064
```
3. Запустите `playbook`, убедитесь, что для нужных хостов применился новый `fact`.
```
dmitry@Lenovo-B50:~/netology/mnt/08-ansible-01-base/playbook$ ansible-playbook site.yml -i inventory/prod.yml --ask-vault-pass
Vault password:

PLAY [Print os facts] ***********************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] **********************************************************************************************************************************************************************************************************************
ok: [localhost]
ok: [ubuntu]
ok: [centos7]

TASK [Print OS] *****************************************************************************************************************************************************************************************************************************
ok: [localhost] => {
    "msg": "Ubuntu"
}
ok: [centos7] => {
    "msg": "CentOS"
}
ok: [ubuntu] => {
    "msg": "Ubuntu"
}

TASK [Print fact] ***************************************************************************************************************************************************************************************************************************
ok: [localhost] => {
    "msg": "PaSSw0rd"
}
ok: [centos7] => {
    "msg": "el default fact"
}
ok: [ubuntu] => {
    "msg": "deb default fact"
}

PLAY RECAP **********************************************************************************************************************************************************************************************************************************
centos7                    : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
localhost                  : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
ubuntu                     : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```
4. Добавьте новую группу хостов `fedora`, самостоятельно придумайте для неё переменную. В качестве образа можно использовать [этот](https://hub.docker.com/r/pycontribs/fedora).
```
dmitry@Lenovo-B50:~/netology/mnt/08-ansible-01-base/playbook$ cat inventory/prod.yml
---
...  
  fed:
    hosts:
      fedora:
        ansible_connection: docker
```
```
dmitry@Lenovo-B50:~/netology/mnt/08-ansible-01-base/playbook$ cat site.yml
---
  - name: Print os facts
    hosts: all
    tasks:
      ...
      - name: Print custom var
        debug:
          msg: "{{ custom_var }}"
        when:
          - ansible_distribution == "Fedora"
```
```
dmitry@Lenovo-B50:~/netology/mnt/08-ansible-01-base/playbook$ cat group_vars/fed/examp.yml
---
  custom_var: "Hi, Fedora!"
```
```
dmitry@Lenovo-B50:~/netology/mnt/08-ansible-01-base/playbook$ ansible-playbook site.yml -i inventory/prod.yml --ask-vault-pass
Vault password:

PLAY [Print os facts] ***********************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] **********************************************************************************************************************************************************************************************************************
ok: [localhost]
ok: [ubuntu]
ok: [fedora]
ok: [centos7]

TASK [Print OS] *****************************************************************************************************************************************************************************************************************************
ok: [localhost] => {
    "msg": "Ubuntu"
}
ok: [centos7] => {
    "msg": "CentOS"
}
ok: [ubuntu] => {
    "msg": "Ubuntu"
}
ok: [fedora] => {
    "msg": "Fedora"
}

TASK [Print fact] ***************************************************************************************************************************************************************************************************************************
ok: [localhost] => {
    "msg": "PaSSw0rd"
}
ok: [centos7] => {
    "msg": "el default fact"
}
ok: [ubuntu] => {
    "msg": "deb default fact"
}
ok: [fedora] => {
    "msg": "PaSSw0rd"
}

TASK [Print custom var] *********************************************************************************************************************************************************************************************************************
skipping: [centos7]
skipping: [ubuntu]
skipping: [localhost]
ok: [fedora] => {
    "msg": "Hi, Fedora!"
}

PLAY RECAP **********************************************************************************************************************************************************************************************************************************
centos7                    : ok=3    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0
fedora                     : ok=4    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
localhost                  : ok=3    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0
ubuntu                     : ok=3    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0
```
5. Напишите скрипт на bash: автоматизируйте поднятие необходимых контейнеров, запуск ansible-playbook и остановку контейнеров.
```
#!/usr/local/env bash
docker-compose up -d
ansible-playbook site.yml -i inventory/prod.yml --vault-password-file vault_pass.txt
docker-compose down
```
6. Все изменения должны быть зафиксированы и отправлены в вашей личный репозиторий.

[Коммит с изменениями по Необязательной части ДЗ](https://github.com/olkhovik/ansible-8.1/commit/719b6a1519c6930e42e32d9a330d05ed322a4bbe)
