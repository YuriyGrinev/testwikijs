Для сбора информации с хостов Linux в систему мониторинга Prometheus на требуемые хосты необходимо установить node_exporter.

Для сбора информации с Apache server-stat необходимо включить работу server-stat в Apache, установить и настроить apache_exporter

1. Установка и запуск как службы [**node_exporter**](https://github.com/prometheus/node_exporter "https://github.com/prometheus/node_exporter")


```bash
cd /tmp/
wget https://github.com/prometheus/node_exporter/releases/download/v1.1.2/node_exporter-1.1.2.linux-amd64.tar.gz
tar xvfz node_exporter-1.1.2.linux-amd64.tar.gz 
cd node_exporter-1.1.2.linux-amd64/
firewall-cmd --add-rich-rule='rule family=ipv4 source address=10.0.0.0/8 port port="9100" protocol="tcp" reject' --permanent
firewall-cmd --add-rich-rule='rule family=ipv4 source address=192.168.254.0/24 port port="9100" protocol="tcp" accept' --permanent
firewall-cmd --reload
sudo useradd -rs /bin/false nodeusr
sudo cp node_exporter /usr/local/bin/
sudo cat > /etc/systemd/system/node_exporter.service <<EOF
[Unit]
Description=Node Exporter
After=network.target

[Service]
User=nodeusr
Group=nodeusr
Type=simple
ExecStart=/usr/local/bin/node_exporter --collector.systemd  

[Install]
WantedBy=multi-user.target
EOF
systemctl daemon-reload 
systemctl enable --now node_exporter.service 

```

Вставьте все строки разом в терминал, для последовательного выполнения (скачивание архива, добавление правил в firewall, добавление пользователя, создание и запуск службы node_exporter

_//TODO: Добавить настройку апача для работы с `apache_exporter`_

Добавить метку