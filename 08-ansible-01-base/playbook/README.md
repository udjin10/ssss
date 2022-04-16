# Самоконтроль выполненения задания

1. Где расположен файл с `some_fact` из второго пункта задания?

Файл `group_vars/all/examp.yml`

2. Какая команда нужна для запуска вашего `playbook` на окружении `test.yml`?
```
ansible-playbook site.yml -i inventory/test.yml
```
3. Какой командой можно зашифровать файл?

`ansible-vault encrypt <file>`, например:
```
ansible-vault encrypt group_vars/deb/examp.yml
```
4. Какой командой можно расшифровать файл?

`ansible-vault decrypt <file>`, например:
```
ansible-vault decrypt group_vars/deb/examp.yml
```
5. Можно ли посмотреть содержимое зашифрованного файла без команды расшифровки файла? Если можно, то как?

`ansible-vault view <file>`, например:
```
ansible-vault view group_vars/deb/examp.yml
```
6. Как выглядит команда запуска `playbook`, если переменные зашифрованы?

Например, так:

`ansible-playbook site.yml -i inventory/prod.yml --ask-vault-pass`

Или можно положить пароль в файл и тогда так:

`ansible-playbook site.yml -i inventory/prod.yml --vault-password-file passwords.txt`

7. Как называется модуль подключения к host на windows?

- [Windows Remote Management](https://docs.ansible.com/ansible/latest/user_guide/windows_winrm.html)
- [psrp](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/psrp_connection.html)

8. Приведите полный текст команды для поиска информации в документации ansible для модуля подключений ssh

```
ansible-doc -t connection ssh
```

9. Какой параметр из модуля подключения `ssh` необходим для того, чтобы определить пользователя, под которым необходимо совершать подключение?

`remote_user`
