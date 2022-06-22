# Домашнее задание к занятию "08.01 Введение в Ansible"

## Подготовка к выполнению
1. Установите ansible версии 2.10 или выше.
2. Создайте свой собственный публичный репозиторий на github с произвольным именем.
3. Скачайте [playbook](./playbook/) из репозитория с домашним заданием и перенесите его в свой репозиторий.

## Основная часть
1. Попробуйте запустить playbook на окружении из `test.yml`, зафиксируйте какое значение имеет факт `some_fact` для указанного хоста при выполнении playbook'a.

```
vagrant@vagrant:/vagrant/8.1$ ansible-playbook -i inventory/test.yml site.yml
ok: [localhost]

TASK [Print OS] *****************************************************************************************
ok: [localhost] => {
    "msg": "Ubuntu"
}

TASK [Print fact] ***************************************************************************************
ok: [localhost] => {
    "msg": 12
}

PLAY RECAP **********************************************************************************************
localhost                  : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

2. Найдите файл с переменными (group_vars) в котором задаётся найденное в первом пункте значение и поменяйте его на 'all default fact'.

```
vagrant@vagrant:/vagrant/8.1$ cat group_vars/all/examp.yml
---
  some_fact: "all default fact"
```

3. Воспользуйтесь подготовленным (используется `docker`) или создайте собственное окружение для проведения дальнейших испытаний.

```
vagrant@vagrant:/vagrant/8.1$ docker ps
CONTAINER ID   IMAGE          COMMAND       CREATED             STATUS         PORTS     NAMES
6efd56f88aac   ubuntu:20.04   "/bin/bash"   About an hour ago   Up 2 minutes             ubuntu
cdde1c40d110   centos:7       "/bin/bash"   About an hour ago   Up 2 minutes             centos7
```

4. Проведите запуск playbook на окружении из `prod.yml`. Зафиксируйте полученные значения `some_fact` для каждого из `managed host`.

```
vagrant@vagrant:/vagrant/8.1$ ansible-playbook -i inventory/prod.yml site.yml 

PLAY [Print os facts] **************************************************************************************************************************

TASK [Gathering Facts] *************************************************************************************************************************
ok: [centos7]
ok: [ubuntu]

TASK [Print OS] ********************************************************************************************************************************
ok: [centos7] => {
    "msg": "CentOS"
}
ok: [ubuntu] => {
    "msg": "Ubuntu"
}

TASK [Print fact] ******************************************************************************************************************************
ok: [centos7] => {
    "msg": "el"
}
ok: [ubuntu] => {
    "msg": "deb"
}

PLAY RECAP *************************************************************************************************************************************
centos7                    : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
ubuntu                     : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
```

5. Добавьте факты в `group_vars` каждой из групп хостов так, чтобы для `some_fact` получились следующие значения: для `deb` - 'deb default fact', для `el` - 'el default fact'.

```
vagrant@vagrant:/vagrant/8.1$ cat group_vars/deb/examp.yml
---
  some_fact: "deb default fact"

vagrant@vagrant:/vagrant/8.1$ cat group_vars/el/examp.yml
---
  some_fact: "el default fact"
```

6.  Повторите запуск playbook на окружении `prod.yml`. Убедитесь, что выдаются корректные значения для всех хостов.

```
vagrant@vagrant:/vagrant/8.1$ ansible-playbook -i inventory/prod.yml site.yml 

PLAY [Print os facts] **************************************************************************************************************************

TASK [Gathering Facts] *************************************************************************************************************************
ok: [centos7]
ok: [ubuntu]

TASK [Print OS] ********************************************************************************************************************************
ok: [centos7] => {
    "msg": "CentOS"
}
ok: [ubuntu] => {
    "msg": "Ubuntu"
}

TASK [Print fact] ******************************************************************************************************************************
ok: [centos7] => {
    "msg": "el default fact"
}
ok: [ubuntu] => {
    "msg": "deb default fact"
}

PLAY RECAP *************************************************************************************************************************************
centos7                    : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
ubuntu                     : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

7. При помощи `ansible-vault` зашифруйте факты в `group_vars/deb` и `group_vars/el` с паролем `netology`.

```
vagrant@vagrant:/vagrant/8.1$ ansible-vault encrypt group_vars/deb/examp.yml 
New Vault password: 
Confirm New Vault password: 
Encryption successful

vagrant@vagrant:/vagrant/8.1$ ansible-vault encrypt group_vars/el/examp.yml 
New Vault password: 
Confirm New Vault password: 
Encryption successful
```

8. Запустите playbook на окружении `prod.yml`. При запуске `ansible` должен запросить у вас пароль. Убедитесь в работоспособности.

```
vagrant@vagrant:/vagrant/8.1$ ansible-playbook -i inventory/prod.yml site.yml --ask-vault-pass
Vault password: 

PLAY [Print os facts] **************************************************************************************************************************

TASK [Gathering Facts] *************************************************************************************************************************
ok: [ubuntu]
ok: [centos7]

TASK [Print OS] ********************************************************************************************************************************
ok: [centos7] => {
    "msg": "CentOS"
}
ok: [ubuntu] => {
    "msg": "Ubuntu"
}

TASK [Print fact] ******************************************************************************************************************************
ok: [centos7] => {
    "msg": "el default fact"
}
ok: [ubuntu] => {
    "msg": "deb default fact"
}

PLAY RECAP *************************************************************************************************************************************
centos7                    : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
ubuntu                     : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

9. Посмотрите при помощи `ansible-doc` список плагинов для подключения. Выберите подходящий для работы на `control node`.

```
local
```

10. В `prod.yml` добавьте новую группу хостов с именем  `local`, в ней разместите localhost с необходимым типом подключения.

```
vagrant@vagrant:/vagrant/8.1$ cat inventory/prod.yml ; echo ""
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
vagrant@vagrant:/vagrant/8.1$ ansible-playbook -i inventory/prod.yml site.yml --ask-vault-pass
Vault password: 

PLAY [Print os facts] **************************************************************************************************************************

TASK [Gathering Facts] *************************************************************************************************************************
ok: [localhost]
ok: [centos7]
ok: [ubuntu]

TASK [Print OS] ********************************************************************************************************************************
ok: [localhost] => {
    "msg": "Ubuntu"
}
ok: [centos7] => {
    "msg": "CentOS"
}
ok: [ubuntu] => {
    "msg": "Ubuntu"
}

TASK [Print fact] ******************************************************************************************************************************
ok: [localhost] => {
    "msg": "all default fact"
}
ok: [centos7] => {
    "msg": "el default fact"
}
ok: [ubuntu] => {
    "msg": "deb default fact"
}

PLAY RECAP *************************************************************************************************************************************
centos7                    : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
localhost                  : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
ubuntu                     : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```
 
12. Заполните `README.md` ответами на вопросы. Сделайте `git push` в ветку `master`. В ответе отправьте ссылку на ваш открытый репозиторий с изменённым `playbook` и заполненным `README.md`.

# Самоконтроль выполненения задания

1. Где расположен файл с `some_fact` из второго пункта задания?

```
\group_vars\all\examp.yml
```

2. Какая команда нужна для запуска вашего `playbook` на окружении `test.yml`?

```
ansible-playbook -i inventory/test.yml site.yml
```

3. Какой командой можно зашифровать файл?

```
ansible-vault encrypt
```

4. Какой командой можно расшифровать файл?

```
ansible-vault decrypt
```

5. Можно ли посмотреть содержимое зашифрованного файла без команды расшифровки файла? Если можно, то как?

```
ansible-vault view
```

6. Как выглядит команда запуска `playbook`, если переменные зашифрованы?

```
ansible-playbook  --ask-vault-pass
```

7. Как называется модуль подключения к host на windows?

```
Windows Remote Management
```

8. Приведите полный текст команды для поиска информации в документации ansible для модуля подключений ssh

```
ansible-doc -l
/ssh
```

9. Какой параметр из модуля подключения `ssh` необходим для того, чтобы определить пользователя, под которым необходимо совершать подключение?

```
-user
```
