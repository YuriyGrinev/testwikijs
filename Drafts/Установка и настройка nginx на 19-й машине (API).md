Для связи ЭДО, ФС с базой данных, перенесенной в ГКУ, потребуется установить и настроить nginx на 19-й машине в МФЦ.

Установите nginx по официальной инструкции
[https://nginx.org/ru/linux_packages.html#RHEL-CentOS](https://nginx.org/ru/linux_packages.html#RHEL-CentOS "https://nginx.org/ru/linux_packages.html#RHEL-CentOS")

затем добавьте постоянное правило для firewall, чтобы разрешить использовать порт для внешних соединений и перезапустите firewall:

`firewall-cmd --permanent --add-port=5011/tcp && firewall-cmd --reload`

где номер порта 5011 используется по принципу 50ХХ (где ХХ - номер МФЦ)
Затем меняем конфигурацию nginx. В файле `/etc/nginx/conf.d/default.conf` меняем порт с `80` на `82` (так как 80 порт занят httpd):
```
server {
    listen       82;
    server_name  localhost;

    #charset koi8-r;
    #access_log  /var/log/nginx/host.access.log  main;

# more code
```
Далее вставляем в самый конец файла `/etc/nginx/nginx.conf` следующий фрагмент:

```
stream {
	upstream db11 {
	server 192.168.254.141:5432;
    }

server {
	listen 5011;
	proxy_pass db11;
	proxy_connect_timeout 2s;
    }
}
```
Разберем подробнее. В блоке stream мы указываем имя-переменную сервера, на который будет пересылаться весь трафик, принимаемый nginx на порт 5011 (директива listen), и, непосредственно адрес и порт сервера с postgresql. В блоке server указывается порт, который будет прослушивать nginx (`listen 5011;`), и путь проксирования, куда трафик будет перенаправляться (`proxy_pass db11;`)
Проверяем правильность конфигов nginx:
`[root@api11 ~]# nginx -t`  
`nginx: the configuration file /etc/nginx/nginx.conf syntax is ok`  
`nginx: configuration file /etc/nginx/nginx.conf test is successful`

теперь можно создать службу для запуска nginx вместе с системой и запустить ее:
`systemctl --now enable nginx`
Проверим что nginx начал слушать порт:
```
# netstat -tulpn | grep 5011
tcp        0      0 0.0.0.0:5011            0.0.0.0:*               LISTEN      26577/nginx: master
```

 