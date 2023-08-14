Добавить в файл /etc/hosts на сервере АИС:
```
192.168.254.78 ais03.mfc03.umfc26 ais03.mfc03
192.168.254.79 db03.mfc03.umfc26 db03.mfc03
192.168.220.19 edo03.mfc03.umfc26 edo03.mfc03
192.168.254.40 repository.umfc26.ru
192.168.220.19 fs03.mfc03.umfc26 fs03.mfc03
192.168.220.19 signer.mfc03.umfc26
192.168.254.54 smev3.mfc03.umfc26
```

Не забываем добавить маршрут

`vim /etc/sysconfig/network-scripts/route-ens18`

В нем
`192.168.220.0/24 via 192.168.254.3`

Применяем параметры

`reboot`