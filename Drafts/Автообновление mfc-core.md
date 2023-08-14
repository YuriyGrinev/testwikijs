Скрипт, размещаем `/var/www/ais24/mfc-core/update-core.sh`

```
#!/bin/bash
# Скрипт для обновления mfc-core
date
pause=$(( RANDOM % 1200 ))
min=$((pause / 60))
sec=$((pause % 60))
echo "Пауза $min мин. $sec сек."
sleep $pause
version=$(curl -s 'http://ais24.mfc24.umfc26:85/api/update/check' | python3 -c "import sys, json; print(json.load(sys.stdin)['current_version'])")
echo "Установленная версия: "$version
last_version=$(curl -s 'http://ais24.mfc24.umfc26:85/api/update/check' | python3 -c "import sys, json; print(json.load(sys.stdin)['update']['version'])")
zip=$(curl -s 'http://ais24.mfc24.umfc26:85/api/update/check' | python3 -c "import sys, json; print(json.load(sys.stdin)['update']['zip'])")
echo "Последняя версия: "$last_version
if [ "$version" == "$last_version" ]
then
        echo "Установлена последняя версия"
        exit
else
        echo "Обновляемся"
        echo "Переходим в каталог"; cd /var/www/ais24/mfc-core ;        pwd
        echo "Скачиваем файл"
        wget -O $zip "http://mfc-core.update.azovkomeks.ru/"$zip
        ls | grep $zip
        unzip -o $zip
        rm -f $zip
fi

```

`crontab -e`

Запускаем раз в сутки в час ночи

`0 1 * * * /var/www/ais24/mfc-core/update-core.sh &>/tmp/cron_update-core_log.log`

`cd /var/www/ais24/mfc-core/`

`chmod +x update-core.sh`