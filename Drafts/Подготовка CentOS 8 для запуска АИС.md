# Обновляемся

`sudo dnf -y update && sudo dnf -y upgrade`
`reboot`

# vim & wget

`sudo dnf -y install vim wget curl tmux nano unzip`

# python
```
sudo dnf -y install python3
sudo alternatives --set python /usr/bin/python3
```
# инструментарий для 

`sudo dnf install -y yum-utils bash-completion policycoreutils-python-utils mlocate bzip2 setroubleshoot`

# добавляем EPEL репо и ставим htop и bashtop

`sudo dnf -y install epel-release && sudo dnf -y update && dnf config-manager --set-enabled powertools && sudo dnf -y install htop bashtop`
# Apache install

`sudo dnf install -y httpd`

# добавляем REMI репо, настраиваем. Ставим PHP

```
sudo dnf install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
sudo dnf install -y https://rpms.remirepo.net/enterprise/remi-release-8.rpm
sudo dnf install -y yum-utils
sudo dnf module reset php
sudo dnf module install -y php:remi-7.3
sudo dnf update -y
sudo dnf config-manager --set-enabled PowerTools
sudo dnf install -y php php-fpm php-bcmath php-curl php-xml php-pgsql php-mbstring php-mcrypt php-gd php-zip php-json php-dba php-soap php-pear php-devel php-dbase php-opcache
```
# Добавляем репозиторий nginx 
[https://nginx.org/ru/linux_packages.html#RHEL-CentOS](https://nginx.org/ru/linux_packages.html#RHEL-CentOS "https://nginx.org/ru/linux_packages.html#RHEL-CentOS")

# устанавливаем nginx

`sudo dnf install -y nginx`

# Первоначальная установка пакетов закончена

# включаем apache

`sudo systemctl enable httpd && sudo systemctl start httpd`

# ставим утилиту для мониторинга, проверяем, запустился ли apache и на каком порту
```
sudo yum -y install net-tools
netstat -tulpn | grep httpd
```
# добавляем правила в firewall

`sudo firewall-cmd --permanent --add-port=80/tcp ; sudo firewall-cmd --reload`
# Настраиваем авторотацию логов

открываем файл `vim /etc/logrotate.d/httpd`и добавляем следующую секцию:
```
/var/www/logs/*log {
    weekly
    missingok
    rotate 7
    size 100M
    compress
    delaycompress
    notifempty
    create 644 apache apache
    sharedscripts
    postrotate
        /bin/systemctl reload httpd.service > /dev/null 2>/dev/null || true
    endscript
}
```
vim в режиме вставки: esc затем a, сохранить - esc, затем :wq
Думаю сюда также стоит добавить секцию `/var/www/aisxx/logs`
Создать каталоги

```
mkdir -p /var/www/ais03/ais03
mkdir -p /var/www/ais03/rr
mkdir -p /var/www/ais03/logs
mkdir -p /var/www/logs
```
# SELinux

```
semanage fcontext -a -t httpd_sys_rw_content_t "/var/www/logs/"
restorecon '/var/www/logs/'
semanage fcontext -a -t httpd_sys_rw_content_t "/var/www/ais03/logs"
restorecon '/var/www/ais03/logs/'
```
# restart apache

`sudo systemctl restart httpd`
есть мягкий перезапуск apache

Создать архив из живой аис
```
tar --exclude='./www_stv/rosreestr/in/*' --exclude='./www_stv/rosreestr/out/*'  --exclude='./www_stv/kladr/*' --exclude='./www_stv/tmp/*' --exclude='./www_stv/tmpRosreestr/*' -cvzf ais07.tar.gz ./www_stv/

tar --exclude='./www_stv/rosreestr/in/*' --exclude='./www_stv/rosreestr/out/*'  --exclude='./www_stv/kladr/*' --exclude='./www_stv/tmp/*' --exclude='./www_stv/tmpRosreestr/*' --exclude='./www_stv/logs/debugbar/*' -cvzf ais07.tar.gz ./www_stv/


 tar --exclude='./www_stv/rosreestr/in/*' --exclude='./www_stv/rosreestr/out/*'  --exclude='./www_stv/kladr/*' --exclude='./www_stv/tmp/*' --exclude='./www_stv/tmpRosreestr/*' --exclude='./www_stv/logs/*/*.log' --exclude='./www_stv/logs/*/*.json' --exclude='./www_stv/logs/*.log' --exclude='./www_stv/videos/*' -cvzf ais.tar.gz ./www_stv/

```

# vim /etc/hosts

```
192.168.254.78 ais03.mfc03.umfc26 ais03.mfc03
192.168.254.79 db03.mfc03.umfc26 db03.mfc03
192.168.220.19 edo03.mfc03.umfc26
192.168.254.40 repository.umfc26.ru
```

# vim /etc/systemd/system/rr.service

```
[Unit]
Description=RR transport FTP Python script
After=httpd.service

[Service]
Type=simple
AmbientCapabilities=CAP_SYS_RAWIO
User=root
ExecStart=/usr/bin/python /var/www/ais03/rr/transportftp.py
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```

вылечить 502 ошибку nginx  | connect() to 58080 failed (13 permission denied) while connecting to upstream |

`setsebool -P httpd_can_network_connect 1`