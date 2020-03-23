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

-------------------------------

1. Версия Ansible  =>2.4 требует для своей работы Python 2.6 или выше
user@linux1:~/linux/homework-10-1$ sudo apt install python
user@linux1:~/linux/homework-10-1$ python -V
Python 2.7.17

2. Далее произведите установку для Вашей ОС по инструкции и убедитесь что Ansible установлен корректно
user@linux1:~/linux/homework-10-1$ sudo apt-get update
user@linux1:~/linux/homework-10-1$ sudo apt-get install software-properties-common
user@linux1:~/linux/homework-10-1$ sudo apt-add-repository --yes --update ppa:ansible/ansible
user@linux1:~/linux/homework-10-1$ sudo apt-get install ansible
user@linux1:~/linux/homework-10-1$ ansible --version
ansible 2.9.6
  config file = /etc/ansible/ansible.cfg
  configured module search path = [u'/home/user/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python2.7/dist-packages/ansible
  executable location = /usr/bin/ansible
  python version = 2.7.17 (default, Nov  7 2019, 10:07:09) [GCC 7.4.0]

3. Создаем папку ansible и кладем в нее Vagrantfile со следующим содержимым
# -*- mode: ruby -*-
# vim: set ft=ruby :

MACHINES = {
  :nginx => {
        :box_name => "centos/7",
        :ip_addr => '192.168.11.150'
  }
}

Vagrant.configure("2") do |config|

  MACHINES.each do |boxname, boxconfig|

      config.vm.define boxname do |box|

          box.vm.box = boxconfig[:box_name]
          box.vm.host_name = boxname.to_s

          box.vm.network "private_network", ip: boxconfig[:ip_addr]

          box.vm.provider :virtualbox do |vb|
            vb.customize ["modifyvm", :id, "--memory", "200"]
          end
          
          box.vm.provision "shell", inline: <<-SHELL
            mkdir -p ~root/.ssh; cp ~vagrant/.ssh/auth* ~root/.ssh
            sed -i '65s/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config
            systemctl restart sshd
          SHELL

      end
  end
end

4. Поднимите управляемый хост командой vagrant up и убедитесь что все прошло успешно и есть доступ по ssh
user@linux1:~/linux/homework-10-1/ansible$ vagrant ssh
[vagrant@nginx ~]$ 

5. Узнаем параметры для подключения к хосту nginx с помощью команды vagrant ssh-config 
user@linux1:~/linux/homework-10-1/ansible$ vagrant ssh-config
Host nginx
  HostName 127.0.0.1
  User vagrant
  Port 2222
  UserKnownHostsFile /dev/null
  StrictHostKeyChecking no
  PasswordAuthentication no
  IdentityFile /home/user/linux/homework-10-1/ansible/.vagrant/machines/nginx/virtualbox/private_key
  IdentitiesOnly yes
  LogLevel FATAL

6. Создадим в папке ansible папку staging и создадим в ней файл hosts со следующим содержимым 
user@linux1:~/linux/homework-10-1/ansible$ cat staging/hosts
[web]
nginx ansible_host=127.0.0.1 ansible_port=2222 ansible_user=vagrant ansible_private_key_file=.vagrant/machines/nginx/virtualbox/private_key

7. И наконец убедимся, что Ansible может управлять нашим хостом. Сделать это можно с помощью команды: ansible nginx -i staging/hosts -m ping
user@linux1:~/linux/homework-10-1/ansible$ ansible nginx -i staging/hosts -m ping
The authenticity of host '[127.0.0.1]:2222 ([127.0.0.1]:2222)' can't be established.
ECDSA key fingerprint is SHA256:D/Y28iBX2ky+QSxqLRI6rsXLvyCcxIK9DyImcqVvysY.
Are you sure you want to continue connecting (yes/no)? yes
nginx | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    }, 
    "changed": false, 
    "ping": "pong"
}

8. Для этого в текущем каталоге создадим файл ansible.cfg со следующим содержимым:
user@linux1:~/linux/homework-10-1/ansible$ cat ansible.cfg
[defaults]
inventory = staging/hosts
remote_user = vagrant
host_key_checking = False
retry_files_enabled = False

9. Теперь из инвентори можно убрать информацию о пользователе:
user@linux1:~/linux/homework-10-1/ansible$ cat staging/hosts
[web]
nginx ansible_host=127.0.0.1 ansible_port=2222 ansible_private_key_file=.vagrant/machines/nginx/virtualbox/private_key

10. Еще раз убедимся, что управляемый хост доступе, только теперь без явного указания inventory файла:
user@linux1:~/linux/homework-10-1/ansible$ ansible nginx -m ping
nginx | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    }, 
    "changed": false, 
    "ping": "pong"
}


