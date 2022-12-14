Данный репозиторий содержит в себе Vagrantfile и Ansible роли для развертывания 3-х виртуальных машин:
* vm1-freeipa - FreeIPA Server
* vm2-foreman - Foreman
* vm3-zabbix  - Zabbix

Все работы выполняются на Fedora 36

Vagrant разворачивает 3 виртуальных машины  
Ansible выполняет provisioning после разворачивания виртуальной машины

# Зависимости

Для начала на хостовой машине необходимо установить следующее:
* Vagrant
* Ansible
* VirtualBox

## Установка Vagrant
https://www.vagrantup.com/downloads

```
sudo dnf install -y dnf-plugins-core
sudo dnf config-manager --add-repo https://rpm.releases.hashicorp.com/fedora/hashicorp.repo
sudo dnf -y install vagrant
```

## Установка Ansible
https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html
```
python3 -m pip -V
python3 -m pip install --user ansible
python3 -m pip install --upgrade --user ansible
```

## Установка VirtualBox
https://computingforgeeks.com/how-to-install-virtualbox-on-fedora-linux/

```
sudo dnf -y install @development-tools
sudo dnf -y install kernel-headers kernel-devel dkms elfutils-libelf-devel qt5-qtx11extras
```
```
cat <<EOF | sudo tee /etc/yum.repos.d/virtualbox.repo
[virtualbox]
name=Fedora $releasever - $basearch - VirtualBox
baseurl=http://download.virtualbox.org/virtualbox/rpm/fedora/36/\$basearch
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://www.virtualbox.org/download/oracle_vbox.asc
EOF
```
```
sudo dnf install VirtualBox-6.1
sudo usermod -a -G vboxusers $USER
```
С помощью `VBoxManage -v` удостовериться, что VirtualBox корректно установился и работает

При необходимости в настройках BIOS отключить SecureBoot и включить виртуализацию

# Создание виртуальных машин одной командой

Создать директорию для проекта и перейти в неё
```
mkdir VagrantVM
cd VagrantVM
```

Склонировать себе репозиторий
```
git clone https://github.com/AnMakar/vagrant-ansible.git
cd vagrant-ansible
```
Выполнить команду
```
vagrant up
```

# Настройки Vagrant

Настройки Vagrant выполняются в vagrantfile

**Замечания:**
* Для обхода блокировки ресурсов в России используется не прокси, а прямая ссылка на образ. На всякий случай оставлены закомментированными настройки для прокси
* Ни одна найденная в Ansible Galaxy роль не настраивала FreeIPA как надо, выдавала кучу ошибок, поэтому роли пришлось писать самому

### VB1:
* Name: vm1-freeipa
* IP: 192.168.56.100
* RAM: 4096
* CPU: 3
* Port Forwarding: 8080
* Provisioning: ./provisioning/install_freeipa.yaml - производит установку домена FreeIPA, создает 2-х пользователей user1 и user2

**Замечания:**  
- [x] Установлен домен FreeIPA "test.local"  
- [x] В домене FreeIPA есть пользователи user1 и user2  
- [ ] Web-интерфейс FreeIPA должен быть доступен с хостовой машины в браузере по порту 8080  
  *Web-интерфейс на данный момент с хостовой машины недоступен, но может работать в виртуальной машине*

### VB2:
* Name: vm2-foreman
* IP: 192.168.56.101
* RAM: 3072
* CPU: 2
* Port Forwarding: 8081
* Provisioning: ./provisioning/install_foreman.yaml - устанавливает FreeIPA Clinet и подключает vm2-foreman к домену FreeIPA на vm1-freeipa, устанавливает Foreman

**Замечания:**  
- [ ] Введена в качестве клиента в ВМ1
- [x] Должен быть установлен сервис Foreman
- [ ] Web-интерфейс Foreman должен быть доступен с хостовой машины в браузере по порту 8081

### VB3:
* Name: vm3-zabbix
* IP: 192.168.56.102
* RAM: 2048
* CPU: 2
* Port Forwarding: 8082
* Provisioning: ./provisioning/install_zabbix.yaml - устанавливает Docker, разворачивает Docker-контейнер с Zabbix

**Замечания:**  
- [ ] Dockerfile который создает Zabbix  
  *Используется готовый официальный Docker-образ - zabbix-appliance - https://hub.docker.com/r/zabbix/zabbix-appliance/*
- [x] 80 порт контейнера проброшен на 80 порт VM3  
- [ ] Web-интерфейс Foreman должен быть доступен с хостовой машины браузере по порту 8082  
  *Web-интерфейс Foreman **не доступен** с хостовой машины в браузере по порту 8082*

# Задание  
### Задание 1  
- [x] При выполнении команды vagrant up появляются в три ВМ с указанными в задании параметрами  
(Т.е. в репозитории есть Vagrantfile).
### Задание 2  
- [ ] Из браузера хостовой машины можно получить доступ к web оболочкам соответствующих
сервисов по соответствующим портам.
- [x] Неважно откуда и как, но можно получить доступ к web оболочкам соответствующих сервисов.
  - [x] Даже если они развернуты не ansible role
  - [ ] Даже если они уже содержаться в vagrant box
  - [x] Даже если они в виде контейнеров
  - [x] Даже если они где-то развернуты руками...
### Задание 3  
- [ ] Есть Dockerfile с сервисом zabbix
  - [ ] Есть кастомный Dockerfile который содержит в себе сервис Zabbix.
  - [ ] Есть кастомный Dockerfile который запускается.
  - [x] Есть Docker контейнер который содержит в себе сервис Zabbix (например из Dockerhub).
- [x] Из браузера хостовой машины можно получить доступ к web оболочке zabbix.
- [x] Неважно откуда и как но можно получить доступ к web оболочке zabbix.  

# Результат
* FreeIPA недоступен в браузере хостовой машины, но доступен внутри виртуальной машины  
![FreeIPA в браузере](https://github.com/AnMakar/vagrant-ansible/blob/main/images/freeipa_out_vm.png)
![FreeIPA в виртуальной машине](https://github.com/AnMakar/vagrant-ansible/blob/main/images/freeipa_in_vm.png)
* Foreman недоступен в браузере хостовой машины, но доступен внутри виртуальной машины
![Foreman в браузере](https://github.com/AnMakar/vagrant-ansible/blob/main/images/foreman_out_vm.png)
![Foreman в виртуальной машине](https://github.com/AnMakar/vagrant-ansible/blob/main/images/foreman_in_vm.png)
* Zabbix доступен в браузере хостовой машины
![Zabbix в браузере](https://github.com/AnMakar/vagrant-ansible/blob/main/images/zabbix_out_vm.png)
