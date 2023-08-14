Добавляем дополнительные пакеты

`apt-get install postgresql-devel postgresql-devel-static gcc`

Скачиваем и распаковываем последнюю версию pg_stat_kcache

`wget https://github.com/powa-team/pg_stat_kcache/archive/REL2_2_1.tar.gz -O pg_stat_kcache-REL2_2_1.tar.gz tar zxvf pg_stat_kcache-REL2_2_1.tar.gz cd pg_stat_kcache-REL2_2_1`

Компилируем

`make`

Результат компиляции

![[make pg_stat_kcache.png]]


Устанавливаем
`make install`

Результат

![[make install pg_stat_kcache.png]]

Прописываем запуск расширений при старте сервера Postgres
`vim /pgdata/postgresql.conf`

`shared_preload_libraries = 'pg_stat_statements,pg_stat_kcache'`

Перегружаем сервер
`systemctl restart postgresql.service`

Создаем к базе данных
`su - postgres -s /bin/bash psql mfc_23`

И создаем расширения

`CREATE EXTENSION IF NOT EXISTS pg_stat_statements; CREATE EXTENSION IF NOT EXISTS pg_stat_kcache;`

Сброс статистики в начале дня:
`select pg_stat_statements_reset();`