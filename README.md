# adm_training_task8
<h1 align="center">Занятие 8. Systemd — создание unit-файла</h1>
<h3 class="western"><a name="_heading=h.h6i87lkp3f19"></a> <span style="font-family: Roboto, serif;"><span style="font-size: small;">Описание домашнего задания</span></h3>
<p style="line-height: 100%; margin-bottom: 0cm;"><span style="font-family: Roboto, serif;">1) Написать service, который будет раз в 30 секунд мониторить лог на предмет наличия ключевого слова (файл лога и ключевое слово должны задаваться в /etc/default).</span></p>
<p style="line-height: 100%; margin-bottom: 0cm;"><span style="font-family: Roboto, serif;">2) Установить spawn-fcgi и создать unit-файл (spawn-fcgi.service).</span></p>
<p style="line-height: 100%; margin-bottom: 0cm;"><span style="font-family: Roboto, serif;">3) Доработать unit-файл Nginx (nginx.service) для запуска нескольких инстансов сервера с разными конфигурационными файлами одновременно.</span></p>
<p style="line-height: 100%; margin-bottom: 0cm;">&nbsp;</p>
<h3 class="western"><a name="_heading=h.df570rpzx1qg"></a><span style="font-family: Roboto, serif;"><span style="font-size: small;">Используемые ОС</span></h3>
<p style="line-height: 108%; margin-bottom: 0.28cm;" align="justify"><span style="font-family: Roboto, serif;">Хостовая ОС Ubuntu 24.04 Desktop. Гостевая ОС Ubuntu 24.04 Server. VirtualBox версия 7.0.16_Ubuntu r162802</span></p>
<h3 class="western"><span style="font-family: Roboto, serif;"><span style="font-size: small;">Выполнение</span></h3>
<p style="line-height: 108%; margin-bottom: 0.28cm;" align="justify"><span style="font-family: Roboto, serif;"><b>1) Написать service, который будет раз в 30 секунд мониторить лог на предмет наличия ключевого слова</b></span></p>
<p style="line-height: 108%; margin-bottom: 0.28cm;" align="justify"><span style="font-family: Roboto, serif;">Создаём файл с конфигурацией для сервиса в директории /etc/default - из неё сервис будет брать необходимые переменные:</span></p>
<p>root@training:~# cat /etc/default/watchlog<br /># Configuration file for my watchlog service<br /># Place it to /etc/default</p>
<p># File and word in that file that will be monitored<br />WORD="ALERT"<br />LOG=/var/log/watchlog.log<br />root@training:~#</p>
<p style="line-height: 108%; margin-bottom: 0.28cm;" align="justify"><span style="font-family: Roboto, serif;">Затем создаём /var/log/watchlog.log и пишем туда строки на своё усмотрение,
плюс ключевое слово ‘ALERT’. Пусть так:</span></p>
<p>root@training:~# cat /var/log/watchlog.log<br />&rsquo;Twas brillig, and the slithy toves<br />Did gyre and gimble in the wabe:<br />ALERT<br />All mimsy were the borogoves,<br />And the mome raths outgrabe.<br />root@training:~#</p>
<p style="line-height: 108%; margin-bottom: 0.28cm;" align="justify"><span style="font-family: Roboto, serif;">Создадим скрипт /opt/watchlog.sh:</span></p>
<p>root@training:~# cat /opt/watchlog.sh<br />#!/bin/bash</p>
<p>WORD=$1<br />LOG=$2<br />DATE=`date`</p>
<p>if grep $WORD $LOG<br />then<br />logger "$DATE: I found word, Master!"<br />else<br />exit 0<br />fi<br />root@training:~#</p>
<p style="line-height: 108%; margin-bottom: 0.28cm;" align="justify"><span style="font-family: Roboto, serif;">Команда logger отправляет лог (с датой и временем) в системный журнал. Добавим права на запуск файла:</span></p>
<p style="line-height: 108%; margin-bottom: 0.28cm;" align="justify"><span style="font-family: Roboto, serif;">root@training:~# chmod +x /opt/watchlog.sh</span></p>
<p style="line-height: 108%; margin-bottom: 0.28cm;" align="justify"><span style="font-family: Roboto, serif;">Создадим юнит для сервиса /etc/systemd/system/watchlog.service:</span></p>
<p>root@training:~# cat /etc/systemd/system/watchlog.service<br />[Unit]<br />Description=My watchlog service</p>
<p>[Service]<br />Type=oneshot<br />EnvironmentFile=/etc/default/watchlog<br />ExecStart=/opt/watchlog.sh $WORD $LOG<br />root@training:~#</p>
<p style="line-height: 108%; margin-bottom: 0.28cm;" align="justify"><span style="font-family: Roboto, serif;">Перезагружаем сервисы для принятия изменений:</span></p>
<p style="line-height: 108%; margin-bottom: 0.28cm;" align="justify"><span style="font-family: Roboto, serif;">root@training:~# systemctl daemon-reload</span></p>
<p style="line-height: 108%; margin-bottom: 0.28cm;" align="justify"><span style="font-family: Roboto, serif;">Создадим юнит для таймера /etc/systemd/system/watchlog.timer:</span></p>
<p>root@training:~# cat /etc/systemd/system/watchlog.timer<br />[Unit]<br />Description=Run watchlog script every 30 second</p>
<p>[Timer]<br /># Run every 30 second<br />OnUnitActiveSec=30s<br />Unit=watchlog.service</p>
<p>[Install]<br />WantedBy=multi-user.target<br />root@training:~#</p>
<p style="line-height: 108%; margin-bottom: 0.28cm;" align="justify"><span style="font-family: Roboto, serif;">Включаем таймер:</span></p>
<p style="line-height: 108%; margin-bottom: 0.28cm;" align="justify"><span style="font-family: Roboto, serif;">root@training:~# systemctl enable watchlog.timer</span></p>
<p style="line-height: 108%; margin-bottom: 0.28cm;" align="justify"><span style="font-family: Roboto, serif;">root@training:~# systemctl start watchlog.timer</span></p>
<p style="line-height: 108%; margin-bottom: 0.28cm;" align="justify"><span style="font-family: Roboto, serif;">Проверим статус, подождем немного и проверим выполнение:</span></p>
<p style="line-height: 108%; margin-bottom: 0.28cm;" align="justify"><span style="font-family: Roboto, serif;">root@training:~# systemctl status watchlog.timer</span></p>
<p style="line-height: 108%; margin-bottom: 0.28cm;" align="justify"><span style="font-family: Roboto, serif;">root@training:~# tail -n 1000 /var/log/syslog  | grep Master</span></p>
<img width="999" height="715" alt="image" src="https://github.com/user-attachments/assets/3d99a8cf-e676-4778-b88d-2f42df3007a1" />
<p style="line-height: 100%; margin-bottom: 0cm;">&nbsp;</p>
<p style="line-height: 108%; margin-bottom: 0.28cm;" align="justify"><span style="font-family: Roboto, serif;">Таймер работает. Примерно 30 секунд. Для большей точности можно использовать параметр AccuracySec.</span></p>
<p style="line-height: 108%; margin-bottom: 0.28cm;" align="justify"><span style="font-family: Roboto, serif;"><b>2) Установить spawn-fcgi и создать unit-файл (spawn-fcgi.service)</b></span></p>
<p style="line-height: 108%; margin-bottom: 0.28cm;" align="justify"><span style="font-family: Roboto, serif;">Устанавливаем spawn-fcgi и необходимые для него пакеты:</span></p>
<p style="line-height: 108%; margin-bottom: 0.28cm;" align="justify"><span style="font-family: Roboto, serif;">root@training:~# apt install spawn-fcgi php php-cgi php-cli apache2 libapache2-mod-fcgid -y</span></p>
<p style="line-height: 108%; margin-bottom: 0.28cm;" align="justify"><span style="font-family: Roboto, serif;">Создаём файл с настройками для будущего сервиса в файле /etc/spawn-fcgi/fcgi.conf:</span></p>
<p>root@training:~# cat /etc/spawn-fcgi/fcgi.conf<br /># You must set some working options before the "spawn-fcgi" service will work.<br /># If SOCKET points to a file, then this file is cleaned up by the init script.<br />#<br /># See spawn-fcgi(1) for all possible options.<br />#<br /># Example :<br />SOCKET=/var/run/php-fcgi.sock<br />OPTIONS="-u www-data -g www-data -s $SOCKET -S -M 0600 -C 32 -F 1 -- /usr/bin/php-cgi"<br />root@training:~#</p>
<p style="line-height: 108%; margin-bottom: 0.28cm;" align="justify"><span style="font-family: Roboto, serif;">Создаём юнит-файл:</span></p>
<p>root@training:~# cat /etc/systemd/system/spawn-fcgi.service<br />[Unit]<br />Description=Spawn-fcgi startup service by Otus<br />After=network.target</p>
<p>[Service]<br />Type=simple<br />PIDFile=/var/run/spawn-fcgi.pid<br />EnvironmentFile=/etc/spawn-fcgi/fcgi.conf<br />ExecStart=/usr/bin/spawn-fcgi -n $OPTIONS<br />KillMode=process</p>
<p>[Install]<br />WantedBy=multi-user.target<br />root@training:~#</p>
<p style="line-height: 108%; margin-bottom: 0.28cm;" align="justify"><span style="font-family: Roboto, serif;">Убеждаемся, что все успешно работает:</span></p>
<p>root@training:~# systemctl daemon-reload<br />root@training:~# systemctl start spawn-fcgi<br />root@training:~# systemctl status spawn-fcgi</p>
<img width="999" height="799" alt="image" src="https://github.com/user-attachments/assets/9dcbb39b-07f0-4585-b205-6546251e8836" />
<p style="line-height: 100%; margin-bottom: 0cm;">&nbsp;</p>
<p style="line-height: 108%; margin-bottom: 0.28cm;" align="justify"><span style="font-family: Roboto, serif;"><b>3) Доработать unit-файл Nginx (nginx.service) для запуска нескольких инстансов сервера с разными конфигурационными файлами одновременно</b></span></p>
<p style="line-height: 108%; margin-bottom: 0.28cm;" align="justify"><span style="font-family: Roboto, serif;">Установим Nginx из стандартного репозитория:</span></p>
<p style="line-height: 108%; margin-bottom: 0.28cm;" align="justify"><span style="font-family: Roboto, serif;">root@training:~# apt install nginx -y</span></p>
<p style="line-height: 108%; margin-bottom: 0.28cm;" align="justify"><span style="font-family: Roboto, serif;">Для запуска нескольких экземпляров сервиса модифицируем исходный service для использования различных конфигураций, а также PID-файлов. Для этого создадим новый Unit для работы с шаблонами (/etc/systemd/system/nginx@.service):</span></p>
<p>root@training:~# cat /etc/systemd/system/nginx@.service<br /># Stop dance for nginx<br /># =======================<br />#<br /># ExecStop sends SIGSTOP (graceful stop) to the nginx process.<br /># If, after 5s (--retry QUIT/5) nginx is still running, systemd takes control<br /># and sends SIGTERM (fast shutdown) to the main process.<br /># After another 5s (TimeoutStopSec=5), and if nginx is alive, systemd sends<br /># SIGKILL to all the remaining processes in the process group (KillMode=mixed).<br />#<br /># nginx signals reference doc:<br /># http://nginx.org/en/docs/control.html<br />#<br />[Unit]<br />Description=A high performance web server and a reverse proxy server<br />Documentation=man:nginx(8)<br />After=network.target nss-lookup.target</p>
<p>[Service]<br />Type=forking<br />PIDFile=/run/nginx-%I.pid<br />ExecStartPre=/usr/sbin/nginx -t -c /etc/nginx/nginx-%I.conf -q -g 'daemon on; master_process on;'<br />ExecStart=/usr/sbin/nginx -c /etc/nginx/nginx-%I.conf -g 'daemon on; master_process on;'<br />ExecReload=/usr/sbin/nginx -c /etc/nginx/nginx-%I.conf -g 'daemon on; master_process on;' -s reload<br />ExecStop=-/sbin/start-stop-daemon --quiet --stop --retry QUIT/5 --pidfile /run/nginx-%I.pid<br />TimeoutStopSec=5<br />KillMode=mixed</p>
<p>[Install]<br />WantedBy=multi-user.target<br />root@training:~#</p>
<p style="line-height: 108%; margin-bottom: 0.28cm;" align="justify"><span style="font-family: Roboto, serif;">Далее необходимо создать два файла конфигурации (/etc/nginx/nginx-first.conf, /etc/nginx/nginx-second.conf). Их можно сформировать из стандартного конфига /etc/nginx/nginx.conf, с модификацией путей до PID-файлов и разделением по портам:</span></p>
<p>root@training:~# cat /etc/nginx/nginx-first.conf<br />user www-data;<br />worker_processes auto;<br />pid /run/nginx-first.pid;<br />error_log /var/log/nginx/error.log;<br />include /etc/nginx/modules-enabled/*.conf;</p>
<p>events {<br /> worker_connections 768;<br /> # multi_accept on;<br />}</p>
<p>http {</p>
<p>##<br /> # Basic Settings<br /> ##<br /> server {<br /> listen 9001;<br /> }</p>
<p>sendfile on;<br /> tcp_nopush on;<br /> types_hash_max_size 2048;<br /> # server_tokens off;</p>
<p># server_names_hash_bucket_size 64;<br /> # server_name_in_redirect off;</p>
<p>include /etc/nginx/mime.types;<br /> default_type application/octet-stream;</p>
<p>##<br /> # SSL Settings<br /> ##</p>
<p>ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3; # Dropping SSLv3, ref: POODLE<br /> ssl_prefer_server_ciphers on;</p>
<p>##<br /> # Logging Settings<br /> ##</p>
<p>access_log /var/log/nginx/access.log;</p>
<p>##<br /> # Gzip Settings<br /> ##</p>
<p>gzip on;</p>
<p># gzip_vary on;<br /> # gzip_proxied any;<br /> # gzip_comp_level 6;<br /> # gzip_buffers 16 8k;<br /> # gzip_http_version 1.1;<br /> # gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;</p>
<p>##<br /> # Virtual Host Configs<br /> ##</p>
<p>include /etc/nginx/conf.d/*.conf;<br /> #include /etc/nginx/sites-enabled/*;<br />}</p>
<p><br />#mail {<br /># # See sample authentication script at:<br /># # http://wiki.nginx.org/ImapAuthenticateWithApachePhpScript<br />#<br /># # auth_http localhost/auth.php;<br /># # pop3_capabilities "TOP" "USER";<br /># # imap_capabilities "IMAP4rev1" "UIDPLUS";<br />#<br /># server {<br /># listen localhost:110;<br /># protocol pop3;<br /># proxy on;<br /># }<br />#<br /># server {<br /># listen localhost:143;<br /># protocol imap;<br /># proxy on;<br /># }<br />#}<br />root@training:~# </p>
<p>root@training:~# cat /etc/nginx/nginx-second.conf<br />user www-data;<br />worker_processes auto;<br />pid /run/nginx-second.pid;<br />error_log /var/log/nginx/error.log;<br />include /etc/nginx/modules-enabled/*.conf;</p>
<p>events {<br /> worker_connections 768;<br /> # multi_accept on;<br />}</p>
<p>http {</p>
<p>##<br /> # Basic Settings<br /> ##<br /> server {<br /> listen 9002;<br /> }</p>
<p>sendfile on;<br /> tcp_nopush on;<br /> types_hash_max_size 2048;<br /> # server_tokens off;</p>
<p># server_names_hash_bucket_size 64;<br /> # server_name_in_redirect off;</p>
<p>include /etc/nginx/mime.types;<br /> default_type application/octet-stream;</p>
<p>##<br /> # SSL Settings<br /> ##</p>
<p>ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3; # Dropping SSLv3, ref: POODLE<br /> ssl_prefer_server_ciphers on;</p>
<p>##<br /> # Logging Settings<br /> ##</p>
<p>access_log /var/log/nginx/access.log;</p>
<p>##<br /> # Gzip Settings<br /> ##</p>
<p>gzip on;</p>
<p># gzip_vary on;<br /> # gzip_proxied any;<br /> # gzip_comp_level 6;<br /> # gzip_buffers 16 8k;<br /> # gzip_http_version 1.1;<br /> # gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;</p>
<p>##<br /> # Virtual Host Configs<br /> ##</p>
<p>include /etc/nginx/conf.d/*.conf;<br /> #include /etc/nginx/sites-enabled/*;<br />}</p>
<p><br />#mail {<br /># # See sample authentication script at:<br /># # http://wiki.nginx.org/ImapAuthenticateWithApachePhpScript<br />#<br /># # auth_http localhost/auth.php;<br /># # pop3_capabilities "TOP" "USER";<br /># # imap_capabilities "IMAP4rev1" "UIDPLUS";<br />#<br /># server {<br /># listen localhost:110;<br /># protocol pop3;<br /># proxy on;<br /># }<br />#<br /># server {<br /># listen localhost:143;<br /># protocol imap;<br /># proxy on;<br /># }<br />#}<br />root@training:~#</p>
<p style="line-height: 108%; margin-bottom: 0.28cm;" align="justify"><span style="font-family: Roboto, serif;">Этого достаточно для успешного запуска сервисов.</span></p>
<p style="line-height: 108%; margin-bottom: 0.28cm;" align="justify"><span style="font-family: Roboto, serif;">Проверим работу:</span></p>
<p>root@training:~# systemctl start nginx@first<br />root@training:~# systemctl start nginx@second<br />root@training:~# systemctl status nginx@first<br />root@training:~# systemctl status nginx@second</p>
<img width="1317" height="902" alt="image" src="https://github.com/user-attachments/assets/840fd56c-41e6-4bc3-b4d2-d29048c5f0af" />
<p style="line-height: 100%; margin-bottom: 0cm;">&nbsp;</p>
<p style="line-height: 108%; margin-bottom: 0.28cm;" align="justify"><span style="font-family: Roboto, serif;">Проверить можно несколькими способами, например, посмотреть, какие порты слушаются:</span></p>
<p style="line-height: 108%; margin-bottom: 0.28cm;" align="justify"><span style="font-family: Roboto, serif;">root@training:~# ss -tnulp | grep nginx</span></p>
<img width="1317" height="143" alt="image" src="https://github.com/user-attachments/assets/98e4566d-6f05-4792-8cf0-16c1ba679251" />
<p style="line-height: 100%; margin-bottom: 0cm;">&nbsp;</p>
<p style="line-height: 108%; margin-bottom: 0.28cm;" align="justify"><span style="font-family: Roboto, serif;">Или просмотреть список процессов:</span></p>
<p style="line-height: 108%; margin-bottom: 0.28cm;" align="justify"><span style="font-family: Roboto, serif;">root@training:~# ps afx | grep nginx</span></p>
<img width="1317" height="206" alt="image" src="https://github.com/user-attachments/assets/663744d9-8c4a-4e04-aaa6-868a04274556" />
<p style="line-height: 100%; margin-bottom: 0cm;">&nbsp;</p>
<p style="line-height: 108%; margin-bottom: 0.28cm;" align="justify"><span style="font-family: Roboto, serif;">Видим две группы процессов Nginx, всё в порядке.</span></p>
<p style="line-height: 100%; margin-bottom: 0cm;">&nbsp;</p>
<p style="line-height: 108%; margin-bottom: 0.28cm;" align="justify"><span style="font-family: Roboto, serif;">Задание завершено.</span></p>
