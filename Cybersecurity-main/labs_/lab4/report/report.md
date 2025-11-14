---
## Front matter
title: "Лабораторная работа №4"
subtitle: "Кибербезопасность предприятия"
author: "Пронякова Ольга Максимовна"

## Generic otions
lang: ru-RU
toc-title: "Содержание"

## Bibliography
bibliography: bib/cite.bib
csl: pandoc/csl/gost-r-7-0-5-2008-numeric.csl

## Pdf output format
toc: true # Table of contents
toc-depth: 2
lof: true # List of figures
lot: true # List of tables
fontsize: 12pt
linestretch: 1.5
papersize: a4
documentclass: scrreprt
## I18n polyglossia
polyglossia-lang:
  name: russian
  options:
	- spelling=modern
	- babelshorthands=true
polyglossia-otherlangs:
  name: english
## I18n babel
babel-lang: russian
babel-otherlangs: english
## Fonts
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
## Pandoc-crossref LaTeX customization
figureTitle: "Рис."
tableTitle: "Таблица"
listingTitle: "Листинг"
lofTitle: "Список иллюстраций"
lotTitle: "Список таблиц"
lolTitle: "Листинги"
## Misc options
indent: true
header-includes:
  - \usepackage{indentfirst}
  - \usepackage{float} # keep figures where there are in the text
  - \floatplacement{figure}{H} # keep figures where there are in the text
---

# Цель работы

Захватить контроллер домена с помощью флага путем получения доступа во внутреннюю сеть. 

# Выполнение лабораторной работы

## 1. Получение meterpreter-сессии с корпоративным сайтом

С помощью модуля wp_wpdiscuz_unauthenticated_file_upload и команды options получили параметры модуля (рис. 1).

![Параметры модуля](image/1.JPG)

Настроили и запустили meterpreter-сессию с корпоративным сайтом с помощью того же модуля wp_wpdiscuz_unauthenticated_file_upload (рис. 2).

![Запуск сессии](image/2.JPG)

Свернули активную сессию с помощью команды background и просмотрели список активных сессий с помощью команды sessions. (рис. 3).

![Список активных сессий](image/3.JPG)

Мы получили сессию с корпоративным сайтом (модуль wordpress). Для успешного выполнения дальнейших операций с атакуемой машиной нам необходимо повысить текущую сессию.   Для повышения сессии мы : свернули активную сессию с помощью команды background, прописали команду sessions -u 1; зашли в новую сессию sessions 2. (рис. 4).

![Повышение сессии](image/4.JPG)

Узнали, какие интерфейсы имеются на машине во внутренней сети, поиск выполнили в shell-оболочке с помощью команды ip a  (рис. 5).

![Вход в оболочку](image/5.JPG)

Для продолжения атаки необходимо просканировать все доступные хосты во внутренней сети с помощью модуля Multi Gather Ping Sweep. Произошло сканирование внутренней сети организации и мы нашли все доступные хосты. Посмотрели  ARP-таблицу на атакуемой машине с помощью команды arp в meterpreter-сессии. Поскольку целевой адрес атакуемого узла находится во внутренней подсети организации, то мы прописали маршрут до активной meterpreter-сессии. Далее выполнили проброс портов во внутреннюю сеть для дальнейшего выполнения команд через технику proxychains. Инструмент proxychains создает туннель через цепочку прокси-серверов и передает по данному туннелю пакет до адреса назначения. Для проброса портов во внутреннюю сеть использовали команду run autoroute -s 10.10.10.0/24 (рис. 6).

![Подброс портов во внутреннюю сеть](image/6.JPG)

Далее необходимо просканировать доступные хосты во внутренней подсети на наличие открытых портов с использованием модуля nmap. Так как сканируемые машины находятся во внутренней сети, то в первую очередь необходимо настроить прокси, через который будут проходить все запросы при сканировании. Для этого нужно применить и настроить модуль metasploit auxiliary/server/socks_proxy. Мы выбрали, настроили и запустили модуль socks_proxy командами use auxiliary/server/socks_proxy , 
set srvhost 127.0.0.1 
set srvport 1080
set version 5
run   (рис. 7).

![Модуль socks_proxy](image/7.JPG)

Настройка и запуск модуля (рис. 8).

![Jobs](image/8.JPG)

Запустили сканирование 100 самых часто используемых портов с помощью команды proxychains nmap –n –sT –Pn --top-ports 100 (рис. 9).

![Сканирование портов](image/9.JPG)

Для атаки на контроллер домена мы использовали Zerologon.  Для проверки подверженности узла данной уязвимости можно использовать утилиту crackmapexec. В результате выполнения команды proxychains crackmapexec smb 10.10.10.20 -M zerologon можно узнать NetBIOS name атакуемой машины, в данном случае – это AD  (рис. 10).

![NetBIOS name атакуемой машины](image/10.JPG)

Сбросили пароль от системной учетной записи администратора контроллера домена (рис. 20).

![Пароль сброшен](image/20.JPG)

Получили дамп хешей учетных записей контроллера домена с помощью команды proxychains impacket-secretdump 'AD$@10.10.10.20 ' -no-pass  (рис. 21).

![Дамп хешей](image/21.JPG)

Получили сессию с контроллером домена, указав обязательные параметры модуля (рис. 22).

![Сессия с контроллером](image/22.JPG)

В активной meterpreter-сессии  перешли в shell-оболочку (рис. 23).

![Переход в оболочку](image/23.JPG)

С помощью команды net user /domain вывели список всех доменных пользователей, далее вывели полную информацию о пользователе «Flag». В результате получили флаг в поле описания пользователя (рис. 24).

![Получение флага](image/24.JPG)

Результат (рис. 25).

![флаг правильный](image/25.JPG)

# Выводы

В ходе данной лабораторной работы мы смогли получить доступ во внутреннюю сеть через узел и получить флаг.

::: {#refs}
:::
