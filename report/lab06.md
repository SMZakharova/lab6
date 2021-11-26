---
# Front matter
title: "Отчёт по лабораторной работе №6"
subtitle: "Мандатное разграничение прав в Linux"
author: "Захарова Софья Михайловна"

# Generic otions
lang: ru-RU

# Bibliography
bibliography: bib/cite.bib
csl: pandoc/csl/gost-r-7-0-5-2008-numeric.csl

# Pdf output format
toc: true # Table of contents
toc_depth: 2
lof: true # List of figures
lot: true # List of tables
fontsize: 12pt
linestretch: 1.5
papersize: a4
documentclass: scrreprt
## I18n
polyglossia-lang:
  name: russian
  options:
	- spelling=modern
	- babelshorthands=true
polyglossia-otherlangs:
  name: english
### Fonts
mainfont: PT Serif
romanfont: PT Serif
sansfont: PT Sans
monofont: PT Mono
mainfontoptions: Ligatures=TeX
romanfontoptions: Ligatures=TeX
sansfontoptions: Ligatures=TeX,Scale=MatchLowercase
monofontoptions: Scale=MatchLowercase,Scale=0.9
## Biblatex
biblatex: true
biblio-style: "gost-numeric"
biblatexoptions:
  - parentracker=true
  - backend=biber
  - hyperref=auto
  - language=auto
  - autolang=other*
  - citestyle=gost-numeric
## Misc options
indent: true
header-includes:
  - \linepenalty=10 # the penalty added to the badness of each line within a paragraph (no associated penalty node) Increasing the value makes tex try to have fewer lines in the paragraph.
  - \interlinepenalty=0 # value of the penalty (node) added after each line of a paragraph.
  - \hyphenpenalty=50 # the penalty for line breaking at an automatically inserted hyphen
  - \exhyphenpenalty=50 # the penalty for line breaking at an explicit hyphen
  - \binoppenalty=700 # the penalty for breaking a line at a binary operator
  - \relpenalty=500 # the penalty for breaking a line at a relation
  - \clubpenalty=150 # extra penalty for breaking after first line of a paragraph
  - \widowpenalty=150 # extra penalty for breaking before last line of a paragraph
  - \displaywidowpenalty=50 # extra penalty for breaking before last line before a display math
  - \brokenpenalty=100 # extra penalty for page breaking after a hyphenated line
  - \predisplaypenalty=10000 # penalty for breaking before a display
  - \postdisplaypenalty=0 # penalty for breaking after a display
  - \floatingpenalty = 20000 # penalty for splitting an insertion (can only be split footnote in standard LaTeX)
  - \raggedbottom # or \flushbottom
  - \usepackage{float} # keep figures where there are in the text
  - \floatplacement{figure}{H} # keep figures where there are in the text
---

# Цель работы

Развить навыки администрирования ОС Linux. Получить первое практическое знакомство с технологией SELinux. Проверить работу SELinux на практике совместно с веб-сервером Apache.


# Задание

Лабораторная работа подразумевает работу с виртуальной машиной VirtualBox, операционной системой Linux, дистрибутивом Centos и закрепление теоретических основ получения практических навыков работы в консоли с атрибутами файлов.


# Выполнение лабораторной работы

Установим/обновим (за суперпользователя) веб-сервер Apache с помощью команды yum install httpd (рис.1).

![Рис.1. Установка веб-сервера.](images/1.jpg){ #fig:001 width=70% }

В конфигурационном файле /etc/httpd/httpd.conf зададим параметр ServerName: ServerName test.ru чтобы при запуске веб-сервера не выдавались лишние сообщения об ошибках, не относящихся к лабораторной работе. (рис.2).

![Рис.2. Изменение параметра.](images/2.jpg){ #fig:001 width=70% }

Также необходимо проследить, чтобы пакетный фильтр был отключен или в своей рабочей конфигурации позволял подключаться к 80-му и 81- му портам протокола tcp. Добавим разрешающие правила с помощью команд (рис.3). 

![Рис.3. Проверка.](images/3.jpg){ #fig:001 width=70% }

Войдем в систему с полученными учётными данными и убедимся, что SELinux работает в режиме enforcing политики targeted с помощью команд getenforce и sestatus (рис.4): 

![Рис.4. Получение данных.](images/4.jpg){ #fig:001 width=70% }

Обратимся к веб-серверу, запущенному на нашем компьютере, и убедимся, что последний работает: service httpd status (рис.5):

![Рис.5. Обращение к веб-серверу.](images/5.jpg){ #fig:001 width=70% }

Найдем веб-сервер Apache в списке процессов, определим его контекст безопасности, используем команду ps auxZ | grep httpd (рис. 6).
В нашем случае контекст безопасности unconfined_u:system_r:httpd_t

![Рис.6. Определение контекста.](images/6.jpg){ #fig:001 width=70% }

Посмотрим текущее состояние переключателей SELinux для Apache с помощью команды sestatus –b | grep httpd (рис.7).
Многие из переключателей находятся в положении «off».

![Рис.7. Текущее состояние переключателей.](images/7.jpg){ #fig:001 width=70% }

Посмотрим статистику по политике с помощью команды seinfo, также определим множество пользователей, ролей и типов. (рис. 8).

![Рис.8. Вывод статистики.](images/8.jpg){ #fig:001 width=70% }

Определим тип файлов и поддиректорий, находящихся в директории /var/www с помощью команды ls -lZ /var/www (рис. 9).

![Рис.9. Определени типа файлов.](images/9.jpg){ #fig:001 width=70% }

Определим тип файлов, находящихся в директории /var/www/html с помощью команды ls –lZ /var/www/html (рис. 10).

![Рис.10. Определение типа файлов во второй директории.](images/10.jpg){ #fig:001 width=70% }

Определим круг пользователей, которым разрешено создание файлов в директории /var/www/html. (рис.11)
Видно, что только суперпользователь может создать файл в данной директории.

![Рис.11. Определение прав.](images/11.jpg){ #fig:001 width=70% }

В следствие этого создадим от имени суперпользователя html-файл /var/www/html/test.html (рис.12). 

![Рис.12. Создание файла.](images/12.jpg){ #fig:001 width=70% }

Проверим контекст созданного файла. (рис.13).
Контекст, присваиваемый по умолчанию вновь созданным файлам в директории /var/www/html: unconfined_u:object_r:httpd_sys_content_t

![Рис.13. Проверка.](images/13.jpg){ #fig:001 width=70% }

Обратимся к файлу через веб-сервер, введя в браузере firefox адрес (рис.14).

![Рис.14. Ввод адресса.](images/14.jpg){ #fig:001 width=70% }

Изучим справку man httpd_selinux и выясним, какие контексты файлов определены для httpd и сопоставим их с типом файла test.html. Проверим контекст файла командой ls –Z /var/www/html/test.html (рис.15).
Т.к. по умолчанию пользователи CentOS являются свободными (unconfined) от типа, созданному нами файлу test.html был сопоставлен SELinux, пользователь unconfined_u. Это первая часть контекста. Далее политика ролевого разделения доступа RBAC используется процессами, но не
файлами, поэтому роли не имеют никакого значения для файлов. Роль object_r используется по умолчанию для файлов на «постоянных» носителях и на сетевых файловых системах. Тип httpd_sys_content_t позволяет процессу httpd получить доступ к файлу. Благодаря наличию последнего типа мы получили доступ к файлу при обращении к нему через браузер.

![Рис.15. Справка.](images/15.jpg){ #fig:001 width=70% }

Изменим контекст файла /var/www/html/test.html с httpd_sys_content_t на другой, к которому процесс httpd не должен иметь доступа, в нашем случае, на samba_share_t: (рис.16).

![Рис.16. Изменение.](images/16.jpg){ #fig:001 width=70% }

Попробуем еще раз получить доступ к файлу через веб-сервер, введя в браузере firefox адрес http://127.0.0.1/test.html (рис.17).
Мы получили сообщение об ошибке.

![Рис.17. Получение доступа.](images/17.jpg){ #fig:001 width=70% }

Проанализируем ситуацию, просмотрев log-файлы веб-сервера Apache, системный log-файл и audit.log при условии уже запущенных процессов setroubleshootd и audtd. (рис.18).
Исходя из log-файлов, мы можем заметить, что проблема в измененном контексте на шаге 13, т.к. процесс httpd не имеет доступа на samba_share_t. В системе оказались запущены процессы setroubleshootd и audtd, поэтому ошибки, связанные с измененным контекстом, также есть в файле /var/log/audit/audit.log.

![Рис.18. Просмотр log-файлов.](images/18.jpg){ #fig:001 width=70% }

Попробуем запустить веб-сервер Apache на прослушивание TCP-порта 81 (а не 80, как рекомендует IANA и прописано в /etc/services), заменив в файле /etc/httpd/conf/httpd.conf строчку Listen 80 на Listen 81. (рис.19).

![Рис.19. Замена строки.](images/19.jpg){ #fig:001 width=70% }

Перезапустим веб-сервер Apache и попробуем обратиться к файлу через веб-сервер, введя в браузере firefox адрес http://127.0.0.1/test.html (рис.20).
Из того, что при запуске файла через браузер появилась ошибка, можно сделать предположение, что в списках портов, работающих с веб-сервером Apache, отсутствует порт 81.

![Рис.20. Перезапуск.](images/20.jpg){ #fig:001 width=70% }

Подтвердим свои догадки, проанализировав log-файлы: tail –n1 /var/log/messages и просмотрев файлы /var/log/http/error_log, /var/log/http/access_log и /var/log/audit/audit.log (рис.21).
Во всех log-файлах появились записи, кроме /var/log/messages.

![Рис.21. Чтелие log-файов.](images/21.jpg){ #fig:001 width=70% }

Выполним команду semanage port –a –t http_port_t –p tcp 81 После этого проверим список портов командой semanage port –l | grep http_port_t (рис.22).
Убедились, что порт 81 присутствует в списке.

![Рис.22. Выполнение команд.](images/22.jpg){ #fig:001 width=70% }

Попробуем теперь запустить веб-сервер Apache еще раз. (рис.23).

![Рис.23. Попытка запуска.](images/23.jpg){ #fig:001 width=70% }

Вернем контекст httpd_sys_content_t к файлу /var/www/html/test.html: chcon –t httpd_sys_content_t /var/www/html/test.html (рис.24).
После этого вновь попробуем получить доступ к файлу через веб-сервер, введя в браузере firefox адрес http://127.0.0.1:81/test.html
Увидели слово содержимое файла - слово «test».

![Рис.24. Ввод команд.](images/24.jpg){ #fig:001 width=70% }

Исправим обратно конфигурационный файл apache, вернув Listen 80. (рис.25).

![Рис.25. Исправление файла.](images/25.jpg){ #fig:001 width=70% }

Удалим привязку http_port_t к 81 порту: semanage port –d –t http_port_t –p tcp 81. Данную команду выполнить невозможно на моей версии CentOS, поэтому получаем ошибку. (рис.26).

![Рис.26. Попытка удаления привязки.](images/26.jpg){ #fig:001 width=70% }

Удалим файл /var/www/html/test.html: rm /var/www/html/test.html (рис.27).

![Рис.27. Удаление файла.](images/27.jpg){ #fig:001 width=70% }



# Выводы

Я развил навыки администрирования ОС Linux. Получил первое практическое знакомство с технологией SELinux. Проверил работу SELinux на практике совместно с веб-сервером Apache.
