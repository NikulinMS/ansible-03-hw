# Домашнее задание к занятию "`Использование Ansible`" - `Никулин Михаил Сергеевич`



---

## Основная часть

#### 1. Допишите playbook: нужно сделать ещё один play, который устанавливает и настраивает LightHouse.
#### 2. При создании tasks рекомендую использовать модули: `get_url`, `template`, `yum`, `apt`.
#### 3. Tasks должны: скачать статику LightHouse, установить Nginx или любой другой веб-сервер, настроить его конфиг для открытия LightHouse, запустить веб-сервер.

В [site.yml](playbook%2Fsite.yml) добавлен PLAY с `tags=lighthouse`, устанавливающий и настраивающий lighthouse. 

#### 4. Подготовьте свой inventory-файл `prod.yml`.

[prod.yml](playbook%2Finventory%2Fprod.yml)

#### 5. Запустите `ansible-lint site.yml` и исправьте ошибки, если они есть.

<details>
    <summary>ansible-lint site.yml</summary>

```shell
root@nikulin:/home/nikulinn/Documents/mnt-homeworks/08-ansible-03-yandex/playbook# ansible-lint site.yml 
WARNING  Listing 3 violation(s) that are fatal
no-changed-when: Commands should not change things if nothing needs doing.
site.yml:66 Task/Handler: Create Clickhouse table

yaml[line-length]: Line too long (191 > 160 characters)
site.yml:67

latest[git]: Result of the command may vary on subsequent runs.
site.yml:205 Task/Handler: Clone Lighthouse source code by Git

Read documentation for instructions on how to ignore specific rule violations.

                   Rule Violation Summary                   
 count tag               profile rule associated tags       
     1 yaml[line-length] basic   formatting, yaml           
     1 latest[git]       safety  idempotency                
     1 no-changed-when   shared  command-shell, idempotency 

Failed: 3 failure(s), 0 warning(s) on 1 files. Last profile that met the validation criteria was 'min'.
```
</details>

- ```latest[git]``` - ошибка, связанная с невозможностью обеспечения идемпотентности, т.к. не указана конкретная версия (коммит) гита, который бы подтвердил, что с каждым запуском код не будет меняться. - Внесена новая переменная, которая определяет скачиваемый коммит.
-  ```no-changed-when``` - ошибка, связанная с тем, что программа не может определить, произвел task изменения или нет. - Добавлено условие изменения.
-  ```yaml[line-length]``` - ошибка, показывающая, что слишком длинная строчка используется в команде. На текущий момент ее оставим без изменения.

Все ошибки были исправлены.
```
root@nikulin:/home/nikulinn/Documents/mnt-homeworks/08-ansible-03-yandex/playbook# ansible-lint site.yml 
WARNING  Listing 1 violation(s) that are fatal
yaml[line-length]: Line too long (191 > 160 characters)
site.yml:67

Read documentation for instructions on how to ignore specific rule violations.

                Rule Violation Summary                
 count tag               profile rule associated tags 
     1 yaml[line-length] basic   formatting, yaml     

Failed: 1 failure(s), 0 warning(s) on 1 files. Last profile that met the validation criteria was 'min'.
```

#### 6. Попробуйте запустить playbook на этом окружении с флагом `--check`.

<details>
    <summary>ansible-playbook -i inventory/prod.yml site.yml --check</summary>

```shell
root@nikulin:/home/nikulinn/Documents/mnt-homeworks/08-ansible-03-yandex/playbook# ansible-playbook -i inventory/prod.yml site.yml --check

PLAY [Install Clickhouse] **********************************************************************************************************************************************************************************************************

TASK [Get clickhouse distrib] ******************************************************************************************************************************************************************************************************
changed: [clickhouse-01] => (item=clickhouse-client)
changed: [clickhouse-01] => (item=clickhouse-server)
failed: [clickhouse-01] (item=clickhouse-common-static) => {"ansible_loop_var": "item", "changed": false, "dest": "./clickhouse-common-static-22.3.3.44.rpm", "elapsed": 0, "item": "clickhouse-common-static", "msg": "Request failed", "response": "HTTP Error 404: Not Found", "status_code": 404, "url": "https://packages.clickhouse.com/rpm/stable/clickhouse-common-static-22.3.3.44.noarch.rpm"}

TASK [Get clickhouse distrib] ******************************************************************************************************************************************************************************************************
changed: [clickhouse-01] => (item=clickhouse-client)
changed: [clickhouse-01] => (item=clickhouse-server)
changed: [clickhouse-01] => (item=clickhouse-common-static)

TASK [Install clickhouse packages] *************************************************************************************************************************************************************************************************
fatal: [clickhouse-01]: FAILED! => {"ansible_facts": {"pkg_mgr": "yum"}, "changed": false, "msg": "No RPM file matching 'clickhouse-common-static-22.3.3.44.rpm' found on system", "rc": 127, "results": ["No RPM file matching 'clickhouse-common-static-22.3.3.44.rpm' found on system"]}

PLAY RECAP *************************************************************************************************************************************************************************************************************************
clickhouse-01              : ok=1    changed=1    unreachable=0    failed=1    skipped=0    rescued=1    ignored=0 
```
</details>

Проверка останавливается на этапе установки пакетов Clickhouse, т.к. они не скачаны в систему
#### 7. Запустите playbook на `prod.yml` окружении с флагом `--diff`. Убедитесь, что изменения на системе произведены.

![img_1.png](img%2Fimg_1.png)
![img_2.png](img%2Fimg_2.png)

#### 8. Повторно запустите playbook с флагом `--diff` и убедитесь, что playbook идемпотентен.

<details>
    <summary>ansible-playbook -i inventory/prod.yml site.yml --diff</summary>

```shell
root@nikulin:/home/nikulinn/Documents/mnt-homeworks/08-ansible-03-yandex/playbook# ansible-playbook -i inventory/prod.yml site.yml --diff

PLAY [Install Clickhouse] **********************************************************************************************************************************************************************************************************

TASK [Get clickhouse distrib] ******************************************************************************************************************************************************************************************************
ok: [clickhouse-01] => (item=clickhouse-client)
ok: [clickhouse-01] => (item=clickhouse-server)
failed: [clickhouse-01] (item=clickhouse-common-static) => {"ansible_loop_var": "item", "changed": false, "dest": "./clickhouse-common-static-22.3.3.44.rpm", "elapsed": 0, "gid": 1000, "group": "centos", "item": "clickhouse-common-static", "mode": "0644", "msg": "Request failed", "owner": "centos", "response": "HTTP Error 404: Not Found", "secontext": "unconfined_u:object_r:user_home_t:s0", "size": 246310036, "state": "file", "status_code": 404, "uid": 1000, "url": "https://packages.clickhouse.com/rpm/stable/clickhouse-common-static-22.3.3.44.noarch.rpm"}

TASK [Get clickhouse distrib] ******************************************************************************************************************************************************************************************************
ok: [clickhouse-01] => (item=clickhouse-client)
ok: [clickhouse-01] => (item=clickhouse-server)
ok: [clickhouse-01] => (item=clickhouse-common-static)

TASK [Install clickhouse packages] *************************************************************************************************************************************************************************************************
ok: [clickhouse-01]

TASK [Modify Clickhouse config.xml] ************************************************************************************************************************************************************************************************
ok: [clickhouse-01]

TASK [Flush handlers] **************************************************************************************************************************************************************************************************************

TASK [Wait for clickhouse-server to become available] ******************************************************************************************************************************************************************************
Pausing for 30 seconds (output is hidden)
(ctrl+C then 'C' = continue early, ctrl+C then 'A' = abort)
ok: [clickhouse-01]

TASK [Create database] *************************************************************************************************************************************************************************************************************
ok: [clickhouse-01]

TASK [Create Clickhouse table] *****************************************************************************************************************************************************************************************************
ok: [clickhouse-01]

PLAY [Vector] **********************************************************************************************************************************************************************************************************************

TASK [Create vector work directory] ************************************************************************************************************************************************************************************************
ok: [vector-01]

TASK [Get Vector distrib] **********************************************************************************************************************************************************************************************************
ok: [vector-01]

TASK [Unzip Vector archive] ********************************************************************************************************************************************************************************************************
ok: [vector-01]

TASK [Install Vector binary] *******************************************************************************************************************************************************************************************************
ok: [vector-01]

TASK [Check Vector installation] ***************************************************************************************************************************************************************************************************
changed: [vector-01]

TASK [Create Vector etc directory] *************************************************************************************************************************************************************************************************
ok: [vector-01]

TASK [Create Vector config vector.yaml] ********************************************************************************************************************************************************************************************
ok: [vector-01]

TASK [Create vector.service daemon] ************************************************************************************************************************************************************************************************
changed: [vector-01]

TASK [Modify Vector.service file ExecStart] ****************************************************************************************************************************************************************************************
--- before: /lib/systemd/system/vector.service
+++ after: /lib/systemd/system/vector.service
@@ -7,7 +7,7 @@
 [Service]
 User=vector
 Group=vector
-ExecStartPre=/usr/bin/vector validate
+ExecStartPre=/usr/bin/vector validate --config-yaml /etc/vector/vector.yaml
 ExecStart=/usr/bin/vector
 ExecReload=/usr/bin/vector validate
 ExecReload=/bin/kill -HUP $MAINPID

changed: [vector-01]

TASK [Modify Vector.service file ExecStartPre] *************************************************************************************************************************************************************************************
--- before: /lib/systemd/system/vector.service
+++ after: /lib/systemd/system/vector.service
@@ -8,7 +8,7 @@
 User=vector
 Group=vector
 ExecStartPre=/usr/bin/vector validate --config-yaml /etc/vector/vector.yaml
-ExecStart=/usr/bin/vector
+ExecStart=/usr/bin/vector --config-yaml /etc/vector/vector.yaml
 ExecReload=/usr/bin/vector validate
 ExecReload=/bin/kill -HUP $MAINPID
 Restart=no

changed: [vector-01]

TASK [Create user vector] **********************************************************************************************************************************************************************************************************
ok: [vector-01]

TASK [Create Vector data_dir] ******************************************************************************************************************************************************************************************************
ok: [vector-01]

RUNNING HANDLER [Start Vector service] *********************************************************************************************************************************************************************************************
changed: [vector-01]

PLAY [Lighthouse] ******************************************************************************************************************************************************************************************************************

TASK [Pre-install Nginx & Git client] **********************************************************************************************************************************************************************************************
ok: [lighthouse-01]

TASK [Clone Lighthouse source code by Git] *****************************************************************************************************************************************************************************************
ok: [lighthouse-01]

TASK [Prepare nginx config] ********************************************************************************************************************************************************************************************************
ok: [lighthouse-01]

PLAY RECAP *************************************************************************************************************************************************************************************************************************
clickhouse-01              : ok=6    changed=0    unreachable=0    failed=0    skipped=0    rescued=1    ignored=0   
lighthouse-01              : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
vector-01                  : ok=13   changed=5    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
```
</details>

#### 9. Подготовьте README.md-файл по своему playbook. В нём должно быть описано: что делает playbook, какие у него есть параметры и теги.


В [site.yml](playbook%2Fsite.yml) добавлен PLAY с `tags=lighthouse`, устанавливающий lighthouse.  

Для работы с ВМ в Yandex Cloud подготовлен ```inventory``` [prod.yml](playbook%2Finventory%2Fprod.yml).  

Подготовленные ВМ:    

| Hostname          | Public IP      | Private IP      | Software           |
|-------------------|----------------|-----------------|--------------------|
| clickhouse-01     | 84.201.157.245 | 192.168.101.27  | Clickhouse         |
| vector-01         | 62.84.117.248  | 192.168.101.31  | Vector             |
| lighthouse-01     | 130.193.50.9   | 192.168.101.37  | Lighthouse & Nginx | 

Vector генерирует данные для отправки в БД Clickhouse с помощью источника данных `type: demo_logs`. 
Шаблон конфига [vector.yaml.j2](playbook%2Ftemplates%2Fvector%2Fvector.yaml.j2)

GUI Lighthouse с данными из БД Clickhouse:  
![img_1.png](img%2Fimg_1.png)

CLI clickhouse-client с данными из БД Clickhouse:
![img_2.png](img%2Fimg_2.png)

[site.yml](playbook%2Fsite.yml) содержит 3 play'я task'ов. Каждый Play содержит в себе task'и по установке 
Clickhouse, Vector и Lighthouse соответственно. Каждый play можно выполнить отдельно, используя тэги: `clickhouse`, 
`vector` и `lighthouse`.  
Плейбук использует 4 файла с переменными: 3 файла для каждой из групп хостов индивидуально:  
- [clickhouse_vars.yml](playbook%2Fgroup_vars%2Fclickhouse%2Fclickhouse_vars.yml)  
- [vector_vars.yml](playbook%2Fgroup_vars%2Fvector%2Fvector_vars.yml)
- [lighthouse_vars.yml](playbook%2Fgroup_vars%2Flighthouse%2Flighthouse_vars.yml) 

и один файл, применяемый для всез групп хостов:  
- [all_vars.yml](playbook%2Fgroup_vars%2Fall%2Fall_vars.yml)    

Для конфигурации Vector и Nginx используются шаблоны конфигов:  
- [vector.yaml.j2](playbook%2Ftemplates%2Fvector%2Fvector.yaml.j2)
- [lighthouse.conf.j2](playbook%2Ftemplates%2Fnginx%2Flighthouse.conf.j2)   

#### 10. Готовый playbook выложите в свой репозиторий, поставьте тег `08-ansible-03-yandex` на фиксирующий коммит, в ответ предоставьте ссылку на него.