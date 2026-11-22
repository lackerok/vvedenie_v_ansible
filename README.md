# Домашняя работа к занятию 1 «Введение в Ansible»

Выполнена обязательная часть задания.

## Подготовка

В качестве управляющего узла использована виртуальная машина
Ubuntu 24.04.3 LTS.

Установлены:
- Ansible Core 2.19.15;
- коллекция community.docker 5.4.0;
- Docker.

Ansible установлен в отдельное Python-окружение.
Исходный playbook скопирован из репозитория задания.

<img width="872" height="269" alt="1" src="https://github.com/user-attachments/assets/3f9e8147-0b4f-410a-aae4-c455b1bf9ce3" />


## 1. Первый запуск playbook

Выполнена команда:

```bash
ansible-playbook -i inventory/test.yml site.yml
```

Для хоста localhost получено значение `some_fact: 12`.
Playbook выполнен успешно.

<img width="944" height="363" alt="2" src="https://github.com/user-attachments/assets/75d4ef1e-c943-4cfa-b897-ac1340f7dc1d" />


## 2. Изменение переменной группы all

Исходное значение найдено в файле
`playbook/group_vars/all/examp.yml`.

Содержимое изменено на:

```yaml
---
some_fact: "all default fact"
```

Повторный запуск на test.yml подтвердил новое значение:
`all default fact`.

<img width="947" height="398" alt="3" src="https://github.com/user-attachments/assets/d89bd56b-5b4e-4ade-a915-f26e1db95b63" />


## 3. Подготовка окружения

Создано собственное окружение из двух Docker-контейнеров:

| Хост | ОС | Группа Ansible |
|---|---|---|
| ubuntu | Ubuntu 24.04 | deb |
| rocky9 | Rocky Linux 9 | el |

В рамках разрешённого собственного окружения хост centos7
из исходного inventory заменён на rocky9.

В оба образа установлен Python. Dockerfile находятся
в каталоге `playbook/docker`.

Команды сборки и запуска из каталога playbook:

```bash
docker build -t ansible-lab-ubuntu:24.04 -f docker/Dockerfile.ubuntu docker/
docker build -t ansible-lab-rocky:9 -f docker/Dockerfile.rocky docker/
docker run -d --name ubuntu ansible-lab-ubuntu:24.04
docker run -d --name rocky9 ansible-lab-rocky:9
```

Для подключения к контейнерам используется
`community.docker.docker`.

<img width="938" height="239" alt="4" src="https://github.com/user-attachments/assets/3acf404d-aba5-4bcc-81b3-a7e78c62f7ab" />


## 4. Первый запуск на prod.yml

Выполнена команда:

```bash
ansible-playbook -i inventory/prod.yml site.yml
```

Получены следующие значения:

| Хост | some_fact |
|---|---|
| ubuntu | deb |
| rocky9 | el |

Переменные конкретных групп deb и el переопределяют
общее значение из group_vars/all.

<img width="940" height="420" alt="5" src="https://github.com/user-attachments/assets/93444f14-c85a-4ab5-b86f-db201ca455c1" />


## 5. Изменение переменных групп

В файле `group_vars/deb/examp.yml` задано:

```yaml
---
some_fact: "deb default fact"
```

В файле `group_vars/el/examp.yml` задано:

```yaml
---
some_fact: "el default fact"
```

## 6. Проверка новых значений

Повторно выполнена команда:

```bash
ansible-playbook -i inventory/prod.yml site.yml
```

Получены корректные значения:

| Хост | some_fact |
|---|---|
| ubuntu | deb default fact |
| rocky9 | el default fact |

Для обоих хостов: `unreachable=0`, `failed=0`.

<img width="946" height="453" alt="6" src="https://github.com/user-attachments/assets/928e491b-f441-4af2-a4db-2148f7c66516" />


## 7. Шифрование через Ansible Vault

Выполнена команда:

```bash
ansible-vault encrypt group_vars/deb/examp.yml group_vars/el/examp.yml
```

Использован учебный пароль `netology`, указанный в задании.

Оба файла успешно зашифрованы и начинаются с заголовка:

```text
$ANSIBLE_VAULT;1.1;AES256
```

<img width="953" height="263" alt="7" src="https://github.com/user-attachments/assets/3158f758-f36c-412d-b07b-a28b92d58544" />


## 8. Запуск с зашифрованными переменными

Выполнена команда:

```bash
ansible-playbook -i inventory/prod.yml site.yml --ask-vault-pass
```

Ansible запросил пароль и успешно выполнил playbook.
Значения переменных сохранились:
- ubuntu — `deb default fact`;
- rocky9 — `el default fact`.

Файлы на диске остались зашифрованными.

<img width="943" height="484" alt="8" src="https://github.com/user-attachments/assets/1a3f27ca-1646-43d1-9031-30b844e8e9f7" />


## 9. Выбор плагина подключения для control node

Список плагинов просмотрен командой:

```bash
ansible-doc -t connection -l
```

Изучена документация:

```bash
ansible-doc -t connection ansible.builtin.local
```

Выбран плагин `ansible.builtin.local`, который выполняет
задачи непосредственно на управляющем узле.

<img width="929" height="697" alt="9" src="https://github.com/user-attachments/assets/8f81dad8-88e3-438f-94d2-83eb4d8a7d96" />


## 10. Добавление группы local

В файл `inventory/prod.yml` добавлена группа:

```yaml
local:
  hosts:
    localhost:
      ansible_connection: ansible.builtin.local
      ansible_python_interpreter: "{{ ansible_playbook_python }}"
```

Хост localhost — управляющая виртуальная машина Ubuntu.
Для него используется тот же Python, которым запущен Ansible.

## 11. Финальный запуск

Выполнена команда:

```bash
ansible-playbook -i inventory/prod.yml site.yml --ask-vault-pass
```

После ввода пароля получены значения:

| Хост | some_fact | Источник |
|---|---|---|
| ubuntu | deb default fact | group_vars/deb/examp.yml |
| rocky9 | el default fact | group_vars/el/examp.yml |
| localhost | all default fact | group_vars/all/examp.yml |

Для группы local отдельное значение some_fact не задано,
поэтому localhost использует переменную группы all.

Для каждого из трёх хостов:
`ok=3`, `changed=0`, `unreachable=0`, `failed=0`.

<img width="937" height="582" alt="10" src="https://github.com/user-attachments/assets/f3c75422-1094-4f5b-8a62-fa33bc08f8b6" />


## Итог

Playbook успешно выполняется на двух контейнерах
и управляющем узле. Значения переменных выбираются
из соответствующих group_vars. Файлы переменных
групп deb и el сохранены в зашифрованном виде.

Решение размещено в ветке master.
