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
<p style="line-height: 108%; margin-bottom: 0.28cm;" align="justify"><span style="font-family: Roboto, serif;">TEXT HERE</span></p>
<p style="line-height: 108%; margin-bottom: 0.28cm;" align="justify"><span style="font-family: Roboto, serif;">TEXT HERE</span></p>
<p style="line-height: 108%; margin-bottom: 0.28cm;" align="justify"><span style="font-family: Roboto, serif;">TEXT HERE</span></p>
<p style="line-height: 108%; margin-bottom: 0.28cm;" align="justify"><span style="font-family: Roboto, serif;">TEXT HERE</span></p>
<p style="line-height: 108%; margin-bottom: 0.28cm;" align="justify"><span style="font-family: Roboto, serif;">TEXT HERE</span></p>
<p style="line-height: 108%; margin-bottom: 0.28cm;" align="justify"><span style="font-family: Roboto, serif;">TEXT HERE</span></p>
<p style="line-height: 100%; margin-bottom: 0cm;">&nbsp;</p>
