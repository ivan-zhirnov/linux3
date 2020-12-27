# linux3

## Задание
1 Часть:
1. Создать нескольких пользователей, задать им пароли, домашние директории и шеллы;
2. Создать группу admin;
3. Включить нескольких из ранее созданных пользователей, а также пользователя root, в группу admin;
4. Запретить всем пользователям, кроме группы admin, логин в систему по SSH в выходные дни (суббота и воскресенье, без учета праздников).

2 Часть. Установить docker; дать конкретному пользователю:
1. права работать с docker (выполнять команды docker ps и т.п.);
2. возможность перезапускать демон docker (systemctl restart docker) не выдавая прав более, чем для этого нужно;

## Выполнение
### 1 Часть
1. Создать нескольких пользователей, задать им пароли, домашние директории и шеллы
```
sudo useradd -p password -s /bin/bash user1
sudo useradd -p password -s /bin/bash user2
sudo useradd -p password -s /bin/bash user3 
```
2. Создать группу admin  
`sudo groupadd admin`
3. Включить нескольких из ранее созданных пользователей, а также пользователя root, в группу admin
```
sudo usermod -aG admin user1
sudo usermod -aG admin user2
sudo usermod -aG admin user3
sudo usermod -aG admin root
```
4. Запретить всем пользователям, кроме группы admin, логин в систему по SSH в выходные дни (суббота и воскресенье, без учета праздников).  
* Устанавливаем PAM  
`sudo apt-get install libpam-script`
* Создаем скрипт со следующим кодом: `sudo nano /usr/share/libpam-script/pam_script_acct`
```
#!bin/bash
script="$1"
shift

if groups $PAM_USER | grep admin > /dev/null
then
       exit 0
else
       if [[ $(date +%u) -lt 6 ]]
       then
               exit 0
       else
               exit 1
       fi
fi

if [ ! -e "$script" ]
then
       exit 0
fi
```
* Делаем файл исполняемым
`sudo chmod +x /usr/share/libpam-script/pam_script_acct`
* Добавлеяем записи в файл `sudo nano /etc/pam.d/sshd`
```
#account    required     pam_time.so
account    required     pam_script.so
```
* Проверяем вход через ssh: `ssh user1@localhost`  
![](https://sun9-48.userapi.com/impg/IJInnlj7ezdybtFMwMqQJxsy-lb211QZXbs9hw/AtfM7qC1kq4.jpg?size=721x415&quality=96&proxy=1&sign=9bd52e3b697be851de4962cf51b3fcea&type=album)


### 2 Часть
1. Установить докер (Debian)
```
sudo apt-get update
sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -    
sudo apt-key fingerprint 0EBFCD88
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/debian \
   $(lsb_release -cs) \
   stable"
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io
```
2. Даем пользователю права на работу с docker  
Для этого просто добавляем пользователя в группу  
`sudo usermod -aG docker user1`  
Проверяем:
```
ssh user1@localhost
docker --version
```
![](https://github.com/ivan-zhirnov/linux3/blob/main/screenshot.jpg)
