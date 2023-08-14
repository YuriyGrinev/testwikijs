## Шаг 1 - Установка сервера Postgres

В первую очередь просмотрите список доступных потоков для модуля postgresql с помощью команды dnf:

`dnf module list postgresql`

![../Files/Pasted image 20230814134133.png](../Files/Pasted%20image%2020230814134133.png)
В этом выводе вы можете видеть все три версии PostgreSQL из репозитория **AppStream**: `9.6`, `10` и `12`. Поток, который предоставляет Postgres версии 10, является потоком по умолчанию, о чем сигнализирует `[d]` в его конце. 

Для установки PostgreSQL версии 9.6 необходимо активировать поток модуля этой версии. При активации потока модуля вы переопределяете поток по умолчанию и делаете все пакеты, связанные с активированным потоком, доступными в системе. Обратите внимание, что только один поток заданного модуля можно активировать в системе в один момент времени.

Чтобы активировать поток модуля для Postgres версии 9.6, запустите следующую команду:

`dnf module enable postgresql:9.6`

![../Files/Pasted image 20230814134202.png](../Files/Pasted%20image%2020230814134202.png)
Устанавливаем сервер Postgres.

`dnf install postgresql-server`

Убедимся, что устанавливается нужная версия.
![../Files/Pasted image 20230814134216.png](../Files/Pasted%20image%2020230814134216.png)
Устанавливаем модули

`dnf install postgresql-contrib`
## Шаг 2 - Создание директории для базы данных

Создаем каталог

`mkdir /pgdata`

Далее аналогично инструкции [Добавление диска для файлов РР или базы данных](Добавление%20диска%20для%20файлов%20РР%20или%20базы%20данных.md) добавляем диск к ВМ. Объем диска предлагаю выбирать так: текущий объем базы данных * 2, но не менее 50 Гб.

```
fdisk -l
fdisk /dev/sdb
pvcreate /dev/sdb1
vgcreate vol_pgdata /dev/sdb1
lvcreate -l+100%FREE vol_pgdata
lvrename /dev/vol_pgdata/lvol0 /dev/vol_pgdata/pgdata
mkfs.xfs -f /dev/vol_pgdata/pgdata
vim /etc/fstab
```

В файле `/etc/fstab`

`/dev/vol_pgdata/pgdata /pgdata xfs defaults 0 0`

Подключаем диск

`mount -a`

Устанавливаем владельца и права
```
chown postgres:postgres /pgdata
chmod 700 /pgdata
```

## Шаг 3 — Создание нового кластера баз данных PostgreSQL
Нужно создать новый кластер баз данных PostgreSQL, прежде чем вы сможете создавать таблицы и загружать в них данные. Кластер баз данных — это набор баз данных, управляемых одним экземпляром сервера. Создание кластера баз данных включает в себя создание директорий, в которых будут размещаться данные базы данных, создание таблиц общего каталога и создание баз данных template1 и postgres.

База данных template1 представляет собой шаблон сортировок, который используется для создания новых баз данных, все, что хранится в template1, даже объекты, которые вы добавляете самостоятельно, будут размещаться в новых базах данных при их создании. База данных postgres — это база данных по умолчанию, предназначенная для пользователей, утилит и сторонних приложений.

Пакет Postgres, установленный на предыдущем шаге, содержит удобный скрипт с именем postgresql-setup, который помогает администрированию кластеров баз данных на низком уровне. Для создания кластера баз данных запустите скрипт с помощью sudo и опции --initdb:

`su - postgres`
```
initdb --locale=ru_RU.utf8 -D /pgdata/
exit
```
Изменяем каталог по-умолчанию

`systemctl edit postgresql.service`
Вписываем туда

```
[Service]
Environment=PGDATA=/pgdata/
```
![../Files/Pasted image 20230814134913.png](../Files/Pasted%20image%2020230814134913.png)
Сохраняем и выходим

```
Ctrl+X
Y
Enter
```
Перезагружаем systemd

`systemctl daemon-reload`
Настраиваем SElinux

```
semanage fcontext -a -t postgresql_db_t '/pgdata(/.*)?'
restorecon -R -v '/pgdata'
```
Запускаем сервер

`systemctl start postgresql`

Проверяем статус

`systemctl status postgresql`

![../Files/Pasted image 20230814135010.png](../Files/Pasted%20image%2020230814135010.png)
Активируем запуск службы при загрузке сервера:

`systemctl enable postgresql`

Редактируем доступ к серверу

`vim /pgdata/pg_hba.conf`

Добавляем доступ для ip машины АИС этого МФЦ.

`host    all             all             192.168.254.122/32       md5`

Редактируем адрес, на котором будет открыт порт

`vim /pgdata/postgresql.conf`

`listen_addresses = '*'`

Добавляем порт в firewall

`firewall-cmd --permanent --add-port=5432/tcp ; firewall-cmd --reload`

Перегружаем службу

`systemctl restart postgresql`

Проверяем

`netstat -tulpn`
![../Files/Pasted image 20230814135046.png](../Files/Pasted%20image%2020230814135046.png)