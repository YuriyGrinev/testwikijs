0. заметки

/etc/sysctl.conf  
net.core.somaxconn = 2048  
net.ipv4.tcp_max_syn_backlog = 512

# 1. Делаем клон машины из шаблона
![Pasted image 20230809122012.png](Pasted%20image%2020230809122012.png)
Настройки клонирования
![clone_ais_VM_config.png](clone_ais_VM_config.png)
После создания клона запускаем машину.

При возникновении ошибки **Integrity failure, возможное нарушение целостности** для восстановления нормальной работы системы, администратору необходимо сформировать новую контрольную сумму объектов контроля, выполнив команду:

`osec --file /etc/osec/dirs.conf –D /var/lib/osec`

Перезагрузить систему.  
  

# 2. Подготавливаем машину
### 2.0 Меняем имя машины

`echo 'ais30.mfc30.umfc26' > /etc/hostname`

### 2.1 Удаляем лишние логи

```
cd /var/log/journal/822ea04c93654188b2798a6e63ca6e20/
rm -f system@*
rm -f user-500@*
```

### 2.2 меняем ip адрес и роут

```
vim /etc/net/ifaces/eth0/ipv4address
vim /etc/net/ifaces/eth0/ipv4route
ifdown eth0 && ifup eth0
```

192.168.254.190  
default via 192.168.254.1 dev ens18 proto static metric 100  
10.0.0.0/8 via 192.168.254.3 dev ens18 proto static metric 100  
10.0.3.0/24 via 192.168.254.3 dev ens18 proto static metric 100  
10.0.10.0/24 via 192.168.254.3 dev ens18 proto static metric 100  
10.111.0.0/16 via 192.168.254.3 dev ens18 proto static metric 100  
11.0.0.0/8 via 192.168.254.3 dev ens18 proto static metric 100  
12.0.0.0/8 via 192.168.254.3 dev ens18 proto static metric 100  
192.167.0.0/16 via 192.168.254.3 dev ens18 proto static metric 100  
192.168.192.0/24 via 192.168.254.3 dev ens18 proto static metric 100  
192.168.254.0/24 dev ens18 proto kernel scope link src 192.168.254.190 metric 100

### 2.3 пароль root'а и admin

`passwd root passwd admin`

### 2.4 Установим переменные

`export EDITOR=vim`

`export MFCid=30`

`export MFCip=192.168.254.190`

### 2.3 Переименовываем каталоги и удаляем лишний

```
mv /www/ais24/ /www/ais$MFCid/
mv /www/ais$MFCid/ais24/ /www/ais$MFCid/ais$MFCid/
rm -rf /www/zamena/
```

### 2.4 Заменяем содержимое файла hosts

```
echo '
127.0.0.1       localhost.localdomain localhost
83.221.222.162 azovkomeks.ru
89.22.182.55 update.altsp.su
83.221.222.162 mfc-core.update.azovkomeks.ru
192.168.254.190 ais30.mfc30.umfc26 ais30.mfc30
192.168.254.191 db30.mfc30.umfc26 db30.mfc30
192.168.254.43 edo30.mfc30.umfc26 edo30.mfc30
192.168.254.40 repository.umfc26.ru
192.168.254.194 fs30.mfc30.umfc26 fs30.mfc30
192.167.0.19 signer.mfc30.umfc26 signer2.mfc30.umfc26
192.168.254.54 smev3.mfc30.umfc26
172.21.254.254 hybrid.sberbank.ru
93.188.44.9 beeline.amega-inform.ru
77.88.21.158 smtp.yandex.ru
' > /etc/hosts
```

### 2.5 Фиксируем изменения

`integalert fix`

# 3. Редактируем конфиги

### 3.0 Конфиги Apache

В каталоге `/etc/httpd2/conf/sites-enabled/` создаем ссылку на `../sites-available/ais.conf`

`ln -s ../sites-available/ais.conf ais30.conf`

<details>
<summary>Текущий конфиг ais30.conf</summary>
```
<VirtualHost *:80>
 ServerName ais30.mfc30.umfc26
 ServerAlias  ais30.mfc30
 DocumentRoot /var/www/ais30/ais30
 <Directory /var/www/ais30/ais30>
    Options FollowSymLinks
    AllowOverride All
    Require all granted
 </Directory>
 
 <Directory "/var/www/ais30/ais30/includes/overload0">
    php_admin_value mbstring.func_overload 0
 </Directory>

 <Directory "/var/www/ais30/ais30/rosreestr">
    Options +Indexes
 </Directory>

  ErrorLog /var/www/ais30/logs/ais30.error.log
#  CustomLog /var/www/ais30/logs/ais30.access.log common

  php_admin_value date.timezone 'Europe/Moscow'
  php_admin_value short_open_tag                  On
  php_admin_value error_reporting                 "E_ALL & ~E_DEPRECATED"
  php_admin_value memory_limit                    -1
  php_admin_value upload_max_filesize             600000M
  php_admin_value post_max_size                   600000M
  php_admin_value log_error_max_len               1204
  php_admin_value default_charset                 UTF-8
  php_admin_value display_errors                  On
  php_admin_value max_execution_time              600
  php_admin_value max_file_upload                 180
  php_admin_value max_input_time                  600
  php_admin_value mbstring.detect_order           auto
  php_admin_value mbstring.encoding_translation   Off
  php_admin_value mbstring.func_overload          7
  php_admin_value mbstring.http_input             auto
  php_admin_value mbstring.http_output            UTF-8
  php_admin_value mbstring.internal_encoding      UTF-8
  php_admin_value mbstring.language               Russian
  KeepAlive                       On
  KeepAliveTimeout                30
  HostnameLookups                 Off

AddOutputFilterByType DEFLATE text/plain
AddOutputFilterByType DEFLATE text/html
AddOutputFilterByType DEFLATE text/xml
AddOutputFilterByType DEFLATE text/css
AddOutputFilterByType DEFLATE application/xml
AddOutputFilterByType DEFLATE application/xhtml+xml
AddOutputFilterByType DEFLATE application/rss+xml
AddOutputFilterByType DEFLATE application/javascript
AddOutputFilterByType DEFLATE application/x-javascript

</VirtualHost>

Listen 85
<VirtualHost *:85>
    DocumentRoot /var/www/ais30/mfc-core/public/
    Options +Indexes +FollowSymLinks
    <Directory "/var/www/ais30/mfc-core">
        AllowOverride All
        Require all granted
        php_admin_value mbstring.func_overload 0
        php_admin_value mbstring.internal_encoding UTF-8
    </Directory>
</VirtualHost>

```
</details>

Редактируем его

`sed -e 's/ais24/ais'"$MFCid"'/g' -e 's/mfc24/mfc'"$MFCid"'/' -i /etc/httpd2/conf/sites-enabled/ais$MFCid.conf`

Конфиг mfc-core

`sed -e 's/ais24/ais'"$MFCid"'/g' -e 's/mfc24/mfc'"$MFCid"'/' -i /etc/httpd2/conf/sites-enabled/mfc-core.conf`

Правим DocumentRoot

`sed -e 's/ais24/ais'"$MFCid"'/g' -i /etc/httpd2/conf/sites-enabled/000-default.conf`

Изменяем параметры mpm для работы с увеличенным количеством клиентов

`vim /etc/httpd2/conf/extra-available/httpd-mpm.conf`

```
<IfModule mpm_prefork_module>
        StartServers          8
        MinSpareServers       16
        MaxSpareServers      40
        MaxClients          300
        ServerLimit         300
        MaxRequestsPerChild   4000
</IfModule>
```
Включаем модуль статистики, добавляем конфиг и включаем его

```
/usr/sbin/a2enmod status
touch /etc/httpd2/conf/mods-available/status.conf
ln -s /etc/httpd2/conf/mods-available/status.conf /etc/httpd2/conf/mods-enabled/status.conf
echo '
ExtendedStatus On
<Location /server-status>
  SetHandler server-status
  Order deny,allow
  Deny from all
  Allow from 127.0.0.1
</Location>
' > /etc/httpd2/conf/mods-enabled/status.conf
```

Проверяем конфигурацию Apache и перезапускаем сервис
```
apachectl -t
service httpd2 restart
```

Устанавливаем и запускаем prometheus_node_exporter:

```sh
apt-get install -y prometheus-node_exporter && \
systemctl enable prometheus-node_exporter.service && \
sed -i 's/9092/9100/g'  /etc/sysconfig/prometheus-node_exporter && \
systemctl daemon-reload && \
systemctl enable --now prometheus-node_exporter.service && \
systemctl restart prometheus-node_exporter.service && \
systemctl status prometheus-node_exporter.service
```

Устанавливаем Apache_exporter

```
export VER="1.0.1"
wget https://github.com/Lusitaniae/apache_exporter/releases/download/v${VER}/apache_exporter-${VER}.linux-amd64.tar.gz
tar xvf apache_exporter-1.0.1.linux-amd64.tar.gz 
cp apache_exporter-1.0.1.linux-amd64/apache_exporter /usr/local/bin/
apache_exporter --version

```

`vim /etc/systemd/system/multi-user.target.wants/prometheus-apache_exporter.service`

```
[Unit]
Description=Prometheus Apache Exporter Service
Documentation=https://github.com/Lusitaniae/apache_exporter
Wants=network-online.target
After=network-online.target

[Service]
Type=simple
User=prometheus
Group=prometheus
ExecReload=/bin/kill -HUP $MAINPID
ExecStart=/usr/local/bin/apache_exporter \
  --insecure \
  --scrape_uri=http://127.0.0.1/server-status/?auto \
  --web.listen-address=0.0.0.0:9117 \
  --telemetry.endpoint=/metrics

SyslogIdentifier=apache_exporter
Restart=always


[Install]
WantedBy=multi-user.target
```

`systemctl daemon-reload `
### 3.1 Конфиги АИС

```
vim /www/ais$MFCid/ais$MFCid/server.ini
vim /www/ais$MFCid/ais$MFCid/config/db.php
```

выставляем права

```
chown -R apache:apache /www/ais$MFCid/ais$MFCid/*
chmod 777 /home/www/ais$MFCid/ais$MFCid/tmp
```

### 3.2 Сервера электронной очереди

`vim /www/ais$MFCid/elque/config.js`

Добавляем в секцию **module.exports.env_conf**

```
    timeUserGone            :   60, // Время отсутствия соединения с пользователем за которое считаем его ушедшим из программы
    timePriemAll            :   2,
    timeHeaderWait          :   3

```


### 3.3 mfc-core конфиг и службу

```
vim /www/ais$MFCid/mfc-core/.env
sed -e 's/ais24/ais'"$MFCid"'/g' -i /etc/systemd/system/elque.service
systemctl daemon-reload
systemctl restart elque.service
```
### 3.4 Crontab
`crontab -e`

### 3.5 **Обработчика очереди**

`sed -e 's/ais24/ais'"$MFCid"'/g' -i /etc/supervisord.d/queue-laravel.ini`
### 3.6 Правим скрипты автообновлений

```
sed -e 's/ais24/ais'"$MFCid"'/g' -e 's/mfc24/mfc'"$MFCid"'/' -i /www/ais$MFCid/elque/update-elque.sh
sed -e 's/ais24/ais'"$MFCid"'/g' -e 's/mfc24/mfc'"$MFCid"'/' -i /www/ais$MFCid/mfc-core/update-core.sh
```

Добавляем в файл `/www/ais$MFCid/mfc-core/update-core.sh`

`vim /www/ais$MFCid/mfc-core/update-core.sh`

```
chmod 777 -R /home/www/aisXX/mfc-core/
chown -R apache:apache /home/www/aisXX/mfc-core/
```

### 3.7 Добавляем ДНС

`vim /etc/resolv.conf`

```
nameserver 192.168.254.1
nameserver 100.126.1.2
update_chrooted -f all
```
# 4. Обновляем АИС

Удаляем 170 хост
`vim /root/.ssh/known_hosts`

Скачиваем файл
```
cd /www/ais$MFCid/ais$MFCid/
scp admin@192.168.254.170:/home/admin/Archive_2.2023.04.26.1618.zip .
unzip Archive_2.2023.04.26.1618.zip
rm Archive*
```

выставляем права

```
chown -R apache:apache /www/ais$MFCid/ais$MFCid/*
chmod 777 /home/www/ais$MFCid/ais$MFCid/tmp/*
chmod -R 777 /home/www/ais$MFCid/mfc-core/storage/framework/cache/*
```

# 5. Копируем видео

Переходим в каталог с плейлистом

`cd /www/ais$MFCid/ais$MFCid/templates/queue/4.0/`

Копируем в него плейлист со старой машины. Не забываем поменять IP-адрес машины.

`scp root@$MFCip:/var/www/ais$MFCid/ais$MFCid/templates/queue/4.0/playlist.xml .`

Переходим в каталог с видео на новой машине

`cd /www/ais$MFCid/ais$MFCid/videos/`

Удаляем все файлы в каталоге

`yes | rm *`

Создаем файл со скриптом

`vim getvideo.py`

Копируем в него следующий код

```Python
import os
import subprocess
import xml.etree.ElementTree as ET

def get_files (playlist, mfcid):
    #Парсим XML-файл
    tree = ET.parse(playlist)
    root = tree.getroot()

    # Получаем все элементы 'location' с помощью XPath
    locations = root.findall(".//{http://xspf.org/ns/0/}location")
    str = f'http://ais{mfcid}.mfc{mfcid}.umfc26//videos/'
    locations_list = [location.text.replace(str, "") for location in locations]

    # Выводим список значений
    return locations_list

def download(hostname, remote_files):
    for file_url in remote_files:
        subprocess.call(['curl', '-O', f'http://{hostname}/videos/{file_url}'])

mfcid = os.environ['MFCid']
hostname = os.environ['MFCip']
playlist = f'/www/ais{mfcid}/ais{mfcid}/templates/queue/4.0/playlist.xml'
remote_files = get_files(playlist, mfcid)
download(hostname, remote_files)
```

Запускаем

`python3 getvideo.py`

Удаляем файл

`rm getvideo.py*`

# 6. Обновляем ОС

```
apt-get update
apt-get dist-upgrade
integalert fix
```

# 7. Синхронизация времени

`vim /etc/ntpd.conf`

`servers ntp3.ntp-servers.net`

`ntpd -d ntp3.ntp-servers.net`

`systemctl start ntpd.service`

`systemctl enable ntpd.service`

`date +%T -s "09:41:30"`