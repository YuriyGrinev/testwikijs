
Редактируем файл 00-mpm.conf
`vim /etc/httpd/conf.modules.d/00-mpm.conf`
```
LoadModule mpm_prefork_module modules/mod_mpm_prefork.so
#LoadModule mpm_event_module modules/mod_mpm_event.so
```

`sudo systemctl restart httpd`

Добавляем конфиг для аис

`vim /etc/httpd/conf.d/ais04.conf`

```
<VirtualHost *:80>
 ServerName ais04.mfc04.umfc26
 ServerAlias  ais04.mfc04
 DocumentRoot /var/www/ais04/ais04
 <Directory /var/www/ais04/ais04>
    Options FollowSymLinks
    AllowOverride All
    Require all granted
 </Directory>
 
 <Directory "/var/www/ais04/ais04/includes/overload0">
    php_admin_value mbstring.func_overload 0
 </Directory>

 <Directory "/var/www/ais04/ais04/rosreestr">
    Options +Indexes
 </Directory>

  ErrorLog /var/www/ais04/logs/ais04.error.log
  CustomLog /var/www/ais04/logs/ais04.access.log common

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
</VirtualHost>

```


`httpd -t`
Должно быть ОК

`sudo systemctl restart httpd`
