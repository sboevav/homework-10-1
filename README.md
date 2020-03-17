# Описание

Стенд для практики на уроке "Автоматизация с помощью Ansible"

## Quick Start

Для запуска стенда достаточно дать:

```
vagrant up
```

Пример однострочника:

```
ansible -i inventories/staging/all.yml -m ping
```

Пример запуска playbook:

```
ansible-playbook -i inventories/staging/all.yml playbooks/vars.yml
```
