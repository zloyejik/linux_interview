# Описание действий для работы

  

## Install Python

  

### Install dependences

```

sudo apt update

sudo apt install python3 python3-venv python-is-python3 wget pkg-config  -y

sudo apt install libbz2-dev  libncurses-dev libgdbm-dev libdb-dev  liblzma-dev tk-dev uuid-dev libreadline-dev libsqlite3-dev libffi-dev -y

sudo apt install gcc g++ zlib1g-dev libssl-dev libxml2-dev libffi-dev qemu-utils

(sudo apt install build-essential zlib1g-dev libssl-dev libxml2-dev libffi-dev qemu-utils)

```

  

### Download python source

```

sudo mkdir -p /opt/python/3.13.9

sudo mkdir -p ~/distr/

cd ~/distr/

wget https://www.python.org/ftp/python/3.13.9/Python-3.13.9.tgz

tar -xvzf ~/distr/Python-3.13.9.tgz -C ~/distr/

```

  

### Install python from sources

```

export OPENSSL_LIBS=/usr/lib/x86_64-linux-gnu/libssl.so

  

cd ~/distr/Python-3.13.9

  

make clean

rm -f config.cache

  

sudo ./configure --prefix=/opt/python/3.13.9 --enable-optimizations --with-openssl=/usr --with-openssl-rpath=auto

sudo make -j$(nproc)

sudo make altinstall

```

  

## Создание виртуального окружения

в папке проекта выполнить команды

  

```

cd ansible

/opt/python/3.13.9/bin/python3.13 -m venv .venv

```

### 1. Активация

```

source .venv/bin/activate

```

  

### 2. Проверка, что используется нужный Python

```

which python3

```

Должно вывести: /путь/к/проекту/.venv/bin/python3

  

```

python3 --version

```

Должно вывести: Python 3.13.9

  

### 3. Обновление pip (рекомендуется)

```

pip3 install --upgrade pip

```

  

## установить Ansible

```

pip3 install ansible ansible-compat ansible-core ansible-lint passlib ovirt-engine-sdk-python

```

  

***Если у вас нет доступа в интернет***

скачайте пакеты с помощью команды:

```

mkdir .whlrepo

cd .whlrepo

python3 -m pip3 download passlib pip

python3 -m pip3 download ansible-core

python3 -m pip3 download ansible

python3 -m pip3 download ansible-base

python3 -m pip3 download ansible-lint

python3 -m pip3 download ansible-compat

python3 -m pip3 download passlib

python3 -m pip3 download ovirt-engine-sdk-python

```

перенесите папку на целевой компьютер в папку .whlrepo и запустите установку с помощью команды:

```

python3 -m pip3 --no-index --find-links=.whlrepo ansible ansible-compat ansible-core ansible-lint passlib ovirt-engine-sdk-python

```

для обновления используйте команду:

```

python3 -m pip3 install --upgrade --no-index --find-links=/where/its/downloaded ansible ansible-compat ansible-core ansible-lint passlib ovirt-engine-sdk-python  

```

  


  

## Создание ключей vault

  

echo "myvaultpass" > ./vaultpass

  

в файле:

  

*./inventory/group_vars/all/all.yml*

  

для переменной local_admin_user_pass

  

генерируем пароль:

  

`ansible-vault encrypt_string --stdin-name 'local_admin_user_pass'`

  

и добавляем в переменную значение в переменную local_admin_user_pass

  

в файле:

  

*./roles/local-sudoers/vars/main.yml*

  

для переменной localusers.password

  

генерируем пароль:

  

ansible-vault encrypt_string --stdin-name 'password'

  

и добавляем в переменную localusers.password, где перенос строки заменяется на \r\n

  

## Создание ключей ssh

  

`ssh-keygen -t rsa -b 4096 -C "ta-ansible@localhost.localdomain" -f ./.ssh/ansible-key`

  

`ssh-keygen -t rsa -b 4096 -C "ta-admin@localhost.localdomain" -f ./files/ssh/ta-admin`

  

# Collections

  

ansible-galaxy collection install -r requirements.yml

  

или

  

ansible-galaxy collection install collections/astra-ald_pro-3.1.2.tar.gz -p collections/

  

ansible-galaxy collection install collections/freeipa-ansible_freeipa-1.14.5.tar.gz -p collections/

  

# Vault file

  

in ansible folder

```

ansible-vault encrypt ./inventory/group_vars/all/vault.yml

ansible-vault view ./inventory/group_vars/all/vault.yml

  

```

  

