## Установка SAMS на Debian Lenny 5.0 ##

Данная статья описывает процесс установки Sams 1.0.5 из официальных пакетов собранных для Debian Lenny.

### Установка зависимостей ###
**Основные пакеты:**

|Компонент         |Программа|Версия|Пакеты для установки|
|:--------------------------|:-----------------|:-----------|:-------------------------------------|
|Web-сервер        |Apache | 2.2.9  |apache2, apache2-mpm-prefork|
|PHP               | PHP   | 5.2.6  |php5, libapache2-mod-php5, php5-cli, php5-common, php5-mysql, php5-gd |
|БД                |MySQL  |5.0.51a | mysql-server-5.0, mysql-client-5.0, libmysqlclient15off|
|Прокси-сервер     |Squid  |2.7     | squid|
|Библиотеки        |pcre   |7.6     |libpcre3|

```
aptitude install apache2 apache2-mpm-prefork
aptitude install php5 libapache2-mod-php5 php5-cli php5-common php5-mysql php5-gd
aptitude install mysql-server-5.0 mysql-client-5.0 libmysqlclient15off
aptitude install squid
aptitude install libpcre3
```

**Дополнительные пакеты для работы SAMS:**
  * squidguard - Для использования редиректора squidguard
  * php5-ldap - Для авторизации пользователей в Active Directory | LDAP
  * php-fpdf - Для генерации pdf отчетов

### Получение и установка SAMS ###
Скачать последнюю версию SAMS собранную в пакеты для Debian Lenny вы можете по адресу:
```
  http://nixdev.net/release/sams/debian/lenny/
```
Из всех возможных вариантов вам нужно выбрать последнюю версию SAMS (на момент написания статьи это 1.0.3-2). Также вам необходимо знать какая архитектура процессора используется у вас (32х или 64х битная).


SAMS для Debian разделен на 3 независимых пакета:
  * **sams** - конфигурационный файл и все исполняемые файлы демонов
  * **sams-web** - web-интерфейс для управления работой sams и squid
  * **sams-doc** - документация к sams

Рекомендуется устанавливать все 3 пакета выбранной версии:
Для 32х битных систем:
```
  wget -c "http://nixdev.net/release/sams/debian/lenny/sams_1.0.3-2_i386.deb" 
  dpkg -i sams_1.0.3-2_i386.deb
```

Для 64х битных систем
```
  wget -c "http://nixdev.net/release/sams/debian/lenny/sams_1.0.3-2_amd64.deb"
  dpkg -i sams_1.0.3-2_amd64.deb
```

Для всех систем:
```
  wget -c "http://nixdev.net/release/sams/debian/lenny/sams-web_1.0.3-2_all.deb"
  wget -c "http://nixdev.net/release/sams/debian/lenny/sams-doc_1.0.3-2_all.deb"
  
  dpkg -i sams-web_1.0.3-2_all.deb sams-doc_1.0.3-2_all.deb
```

### Настройка SAMS ###
**Создание БД:**

Для работы SAMS необходимо создать пользователя sams в БД MySQL:
```
  mysql -u root -p
  GRANT ALL ON squidctrl.* TO sams@localhost IDENTIFIED BY "yourpassword";
  GRANT ALL ON squidlog.* TO sams@localhost IDENTIFIED BY "yourpassword";
```

Где: **yourpassword** - пароль пользователя sams в БД

Если вы только установили MySQL, то **пароль пользователя root - пустой**. Поэтому mysql-консоль нужно будет запускать без параметра -p:
```
  mysql -u root
```

После этого имя пользователя и пароль надо сохранить в файле конфигурации SAMS /etc/sams.conf:

  * MYSQLUSER=sams  - Имя пользователя MySQL, от имени которого будет работать SAMS
  * MYSQLPASSWORD=yourpasswd - Пароль пользователя в MySQL

Создаем базы SAMS в MySQL

Для этого перемещаемся в каталог /usr/share/sams/mysql и там выполняем команды:
```
  cd /usr/share/sams/mysql
  mysql -u root -p < sams_db.sql
  mysql -u root -p < squid_db.sql
```

**Настройка Apache:**
Пакет sams-web уже содержит конфигурационный файл для web-сервера apache для работы SAMS по адресу **http://<адрес сервера>/sams/**
Для настройки sams для работы на виртуальном домене - обратитесь к документации по серверу apache.

**Настройка PHP:**

С текущей версии SAMS научился работать с PHP в режиме safe\_mode=On. Но это требует дополнительной настройки конфигурации. Для этого редактируем файл конфигурации php /etc/php5/apache2/php.ini.

  1. включаем режим safe mode. Для этого выставляем параметр safe\_mode
```
  safe_mode = On
```
  1. SAMS для некоторых функций WEB интерфейса использует системные команды, например wbinfo. В режиме safe\_mode php блокирует доступ к системным командам. Php позволяет выполнять системные команды, расположенные в каталоге, заданном параметром safe\_mode\_exec\_dir.
Изменяем этот параметр:
```
  safe_mode_exec_dir = "/usr/share/sams/bin"
```

  1. Далее разрешаем исполнение системных скриптов из кода php. Ищем в файле конфигурации параметр и убираем из него запрет вызова функций phpinfo system shell\_exec exec:
```
  disable_functions = "chdir,dl,ini_get_all,popen,proc_open,passthru,pcntl_exec"
```
  1. Все. PHP готов к работе.

## Настройка SAMS: ##

  1. Для запуска демонов sams необходимо отредактировать init-скрипт sams. Для этого необходимо в файле /etc/init.d/sams изменить значение параметра **SAMS\_ENABLE** с **FALSE** на **TRUE**. **Только после этого sams будет учитывать трафик**
  1. Web-интерфейс SAMS доступен по адресу http://<адресс сервера>/sams/
  1. Для доступа к web-интерфейсу используйте следующие логин/пароль: admin/qwerty
  1. Т.к. пароль администратора и аудитора установлен по умолчанию - рекомендуется сразу сменить его через web-интерфейс
  1. После этого необходимо настроить параметры работы samsdaemon: тип авторизации пользователей, частота парсинга логов и реконфигурации squid, и прочее
  1. После этого можно создавать пользователей