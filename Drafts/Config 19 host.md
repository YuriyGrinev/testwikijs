добавить в конфиг httpd:

`vi /etc/httpd/conf/httpd.conf`

```
<VirtualHost *:80>
ServerName edo03.mfc03.umfc26
ServerAlias edo03.mfc03
ProxyRequests On
ProxyPreserveHost On
ProxyVia full
    <proxy *>
      Order deny,allow
      Allow from all
    </proxy>
ProxyPass / http://192.168.220.12:80/
ProxyPassReverse / http://192.168.220.12:80/
</VirtualHost>


<VirtualHost *:80>
ServerName ais03.mfc03.umfc26
ServerAlias ais03.mfc03
ProxyRequests On
ProxyPreserveHost On
ProxyVia full
    <proxy *>
      Order deny,allow
      Allow from all
    </proxy>
ProxyPass / http://192.168.254.78:80/
ProxyPassReverse / http://192.168.254.78:80/
</VirtualHost>
```

Поменять адреса сервера БД в следующих файлах:

```
vim /home/glassfish/glassfish4/glassfish/domains/domain1/config/casestatusws-point-config.xml
vim /home/glassfish/glassfish4/glassfish/domains/domain1/config/pre-entry-point-config.xml
vim /home/ws/receprion-result-case-node/node-config.xml
vim /var/www/mfc-portal/web/gku/conect.php
vim /var/www/mfc-portal/config/dbMfc.php
```
