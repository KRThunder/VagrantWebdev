## Виртуальная машина для веб-разработки

Виртуальная машина основата на утилите Vagrant, для работы нужно следующее:
 - [Vagrant](http://www.vagrantup.com/downloads.html)
 - [VirtualBox](https://www.virtualbox.org/wiki/Downloads)

В качестве гостевой ОС используется Debian 7 Wheezy.

После развертывания, запускается из консоли командой `vagrant up` либо командным файлом `bin/*/vagrant/up`.
После этого начинается процесс инициализации, который длится достаточно долго, инициализация производится
только при первом запуске либо при обновлении.

Во время инициализации из интернета скачивается некоторое количество мегабайт, так что нужен неплохой интернет.
Подробнее о работе с утилитой можно почитать [здесь](http://habrahabr.ru/post/113354/). Описание установки
и настройки можно пропустить.

После окончания работы машину нужно корректно выключить командой `vagrant halt`, доступ к виртуальному серверу можно
получить с помощью `vagrant ssh` или напрямую - адрес машины `192.168.2.10`.

Сервер предоставляет следующие сервисы:
 - Веб-сервер (Apache 2 + PHP 5.4),
 - Xdebug ([его конфигурация](https://github.com/Andre-487/VagrantWebdev/blob/master/provision/data/php/xdebug.ini))
 - MySQL
 - PostgreSQL
 - Redis
 - Memcached
 - Sphinx
 
Так же внутри настроены SQLite, Vim, Python PIP, PHP Pear, PHPUnit (с дополнением DbUnit), Xdebug
и другие приятные вещи.

Для пользователя `root` в MySQL и пользователя `postgres` в PostgreSQL установлен пароль `password`.

В переменных окружения Apache есть значение `DEBUG_SERVER = 1` для облегчения обнаружения отладочного сервера

Для Sphinx реализован препроцессинг файла конфигурации, файлы отдельных приложений можно складывать в каталог
`/etc/sphinxsearch/conf.d/`.
Для запуска демона Sphinx необходимо в файле `/etc/default/sphinxsearch` установить опцию `START=yes`.


## Параметры, утилиты и командные файлы
### Настройки
Параметры хранятся в файле `params.ini`, эти параметры распространяются как на виртуальную машину, так и на утилиты.

По умолчанию каталогом приложений считается `D:\htdocs` для Windows и `~/htdocs` - для Unix-like систем,
изменить каталог можно в файле `params.ini`.

IP-адрес виртуальной машины в закрытой виртуальной сети по умолчанию `192.168.2.10`, его можно изменить в файле
`params.ini`, после его изменения требуется повторить `PatchHosts`.

### Доставка почты
По умолчанию почтовый сервер Exim4 работает в режиме local, при котором почта не доставляется на удаленные серверы.
Чтобы изменить это поведение, установите в файле `params.ini` опцию `use_smtp=1`, а так же настройте параметры
SMTP-сервера, через который будет пересылаться почта. После этого необходимо повторить `vagrant provision`

### Утилиты и командные файлы
В каталоге `bin` находятся утилиты и командные файлы. Они обеспечивают быстрый доступ к базовым командам утилиты Vagrant,
а так же дополнительные возможности, описанные ниже.

Утилита `mysqldump` предназначена для быстрого создания дампа всех баз MySQL,
дамп записывается в подкаталог `runtime`.
Аналогичные возможности есть и для других СУБД: `pg_dumpall` - для PostgreSQL и `redis_dump` - для Redis.

Утилита `sphinx_rotate` служит для обновления индекса сервера Sphinx.

### Виртуальные хосты, PatchHosts и UpdateApacheVHosts
Веб-сервер виртуальной машины обеспечивает доступ к веб-приложениям в виртуальной зоне имен loc.
Домены формируются следующим образом: `(?<dir_name>[\w\.]+)\.loc`. То есть, к примеру, если приложение находится
в каталоге `/var/www/test`, оно будет доступно по домену `test.loc` (а так же `www.test.loc`).

Для обеспечения этой функции доменные имена зоны loc должны быть добавлены в файл `hosts` рабочей машины,
а в гостевой ОС, в конфигурации Apache должны присутствовать соответствующие виртуальные хосты. Для этих целей
созданы утилиты PatchHosts для рабочей машины и UpdateApacheVHosts - для гостевой.

Утилита PatchHosts, основываясь на содержимим каталога `/var/www` дописывает домены в зоне loc в файл `hosts`.
В случае, если утилита не сможет записать файл самостоятельно (в Windows 8.1 это всегда так, даже с правами администратора),
она покажет нужный контент в блокноте, в этом случае нужно скопировать значения вручную в файл `hosts`.

В случае, если имеется *dnsmasq*, для ресолвинга доменов в зоне loc можно добавить в его ".d" каталог файл
`./etc/dnsmasq/vagrant_webdev.conf`. *На Ubuntu* актуальных версий и, возможно, некоторых других системах dnsmasq установлен
по умолчанию, в данной системе рекомундуется использовать именно этот способ, файл нужно копировать в каталог
`/etc/NetworkManager/dnsmasq.d`.
Так же следует обратить внимание на факт, что на Ubuntu часто проявляется проблема с игнорированием файла `/etc/hosts`.

При добавлении в `/var/www` нового приложения необходимо обновить виртуальные хосты Apache.
Для этого служит утилита UpdateApacheVHosts. Эта утилита запускается автоматически при запуске, перезагрузке
или продолжении работы машины с помощью командных файлов из каталога `bin/*/vagrant`, в ином случае должна запускаться
вручную после изменений в каталоге приложений.

### Обновление
Для обновления нужно скачать новую версию, например, с помощью `git pull` или в виде архива,
и повторить `vagrant provision`.
