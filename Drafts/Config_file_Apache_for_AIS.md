Создать файл /etc/httpd/conf.d/ais**XX**.conf со следующим содержимым, заменив номер МФЦ на актуальный:
```
<VirtualHost *:80>
  ServerName ais07.mfc07.umfc26
  ServerAlias ais07.mfc07
  DocumentRoot /var/www/ais07/ais07
  <Directory /var/www/ais07/ais07>
    Options FollowSymLinks
    AllowOverride All
    Require all granted
  </Directory>
  <Directory "/var/www/ais07/ais07/includes/overload0">
       php_admin_value mbstring.func_overload 0
</Directory>
  ErrorLog /var/www/ais07/logs/ais07.error.log
  CustomLog /var/www/ais07/logs/ais07.asses.log common
  php_admin_value date.timezone 'Europe/Moscow'
  php_admin_value short_open_tag    On
  php_admin_value error_reporting   "E_ALL & ~E_DEPRECATED & ~E_STRICT & ~E_NOTICE"
  php_admin_value max_execution_time  360
  php_admin_value memory_limit   -1
  php_admin_value upload_max_filesize  2048M
  php_admin_value post_max_size   2048M
  php_admin_value log_error_max_len  1204
  php_admin_value default_charset   UTF-8
  php_admin_value display_errors   On
  php_admin_value max_file_upload   20
  php_admin_value max_input_time   60
  php_admin_value mbstring.detect_order  auto
  php_admin_value mbstring.encoding_translation Off
  php_admin_value mbstring.func_overload  7
  php_admin_value mbstring.http_input  auto
  php_admin_value mbstring.http_output  UTF-8
  php_admin_value mbstring.internal_encoding UTF-8
  php_admin_value mbstring.language  Russian
</VirtualHost>


```
