В статье рассматривается инсталляция и настройка ''Reductor'' для фильтрации трафика в режиме приёма зеркала с коммутатора. Рассматриваемая в статье связка может использоваться у малых и средних провайдеров доступа к сети Интернет для фильтрации трафика по новомодным и не очень законам (ФЗ-139, ФЗ-149, ФЗ-187), как бы неприятно это не было. Схема опробована в более чем ста инсталляциях, от 200 до 130 000 пользователей.

# Введение

Цель статьи — осветить как базовые нюансы настройки фильтрации, так и "фишечки" продукта, показать как проходит настройка с момента установки CentOS и настройки сети до получения результата в виде направления абонентов на страницу с уведомлением о блокировке при открытии запрещённых сайтов.

# Лицензии
Для ознакомления существует полнофункциональная демоверсия, скачать можно на [http://carbonsoft.ru/products/reductor/carbon-reductor-4-0-2 официальном сайте]. Демопериод ограничен 60-тью днями, в случае если вы с самого начала обратились в поддержку и интеграция задерживается по какой-либо причине (редкость, обычно 1-10 дней) - может быть продлён. Этого более чем достаточно, чтобы определиться с покупкой.

# Базовая схема сети. Краткое описание
На сервер фильтрации устанавливается дистрибутив CentOS 6 (пока только x86_64), на нём настраивается доступ в интернет, устанавливается RPM-пакет Carbon Reductor, проводится его настройка. На коммутаторе, ведущем к фронт-роутеру настраивается дублирование трафика на порт сервера фильтрации. Carbon Reductor анализирует этот трафик и на обращения к сайтам указанным в списке запрещённых формирует HTTP-пакет (или tcp-reset, если модуль редиректа отключен), отправляющийся к пользователю от имени запрашиваемого ресурса и перенаправляет пользователя на страницу с уведомлением о том, что данный сайт заблокирован.

Профиты данной схемы — отсутствие необходимости вносить изменения в уже существующую сеть, за исключением настройки зеркала трафика (затрагивает только центральный коммутатор), а также то, что Carbon Reductor абсолютно не влияет на качество доступа абонентов в сеть Интернет, и тем более ''не становится точкой отказа''.
[[Файл:generic_scheme.png]]

# Базовые настройки Carbon Reductor
## Установка CentOS 6.5
Для установки рекомендуется использовать CentOS 6.5 Minimal x86_64, добыть который можно с любого из [[http://www.centos.org/modules/tinycontent/index.php?id=30 зеркал]]. 

Внимание - CentOS 7, по крайней мере на октябрь 2014 не поддерживается!

## Настройка доступа в Интернет
Для того чтобы настроить доступ к сети, необходимо поправить конфигурационный файл:

	/etc/sysconfig/network-scripts/ifcfg-eth0

или другой, в зависимости от того, через какую сетевую карту вы планируете ходить в интернет.

Сделать это можно с помощью текстового редактора vi:

	vi /etc/sysconfig/network-scripts/ifcfg-eth0

Для редактирования используем клавишу i - после нажатия в открытом файле появится возможность редактирования.

В файле необходимо изменить и добавить следующие поля:

	DEVICE=eth0
	IPADDR=здесь.ваш.ip.адрес
	NETMASK=здесь.ваша.сетевая.маска
	GATEWAY=здесь.укажите.ip.шлюза
	DEFROUTE=yes
	DNS1=здесь.укажите.ip.dns-сервера
	DNS2=здесь.укажите.ip.dns-сервера2
	ONBOOT=yes
	TYPE=Ethernet
	NM_CONTROLLED=no
	BOOTPROTO=static

Например:

	DEVICE=eth0
	IPADDR=192.168.0.105
	NETMASK=255.255.255.0
	GATEWAY=192.168.0.1
	DEFROUTE=yes
	DNS1=8.8.8.8
	DNS2=8.8.4.4
	ONBOOT=yes
	TYPE=Ethernet
	NM_CONTROLLED=no
	BOOTPROTO=static

## Установка нужных пакетов и самого reductor
	sudo -s
	wget "http://download5.carbonsoft.ru/reductor/reductor_install"
	bash reductor_install

Стартуем и настраиваем до тех пор пока не будет ни одного FAIL'а.

	/etc/init.d/reductor restart

## Консольное меню
Запускается после установки Carbon Reductor командой:

	menu

Дефолтная кодировка, если вы выбрали русский язык - UTF-8, поэтому при подключении с помощью PuTTY в настройках (Change Preference -> Translation) вместо KOI8-R нужно выбирать UTF-8. 

[[Файл:carbon_reductor_local_menu.png]]

### Обновление Carbon Reductor
Внутри четыре пункта:

Запуск обновления. Все настройки и сертификаты сохраняются, старая версия копируется в папку /usr/local/old_Reductor.$version/.

Включить автоматическое обновление на все версии. В два часа ночи Carbon Reductor проверит, вышла ли новая версия и обновится на неё.

Включить обновление на критические исправления. В два часа ночи Carbon Reductor проверит не изменилась ли версия Reductor с последним критическим исправлением, и если установленная версия старее - обновится.

Обновить веб-интерфейс Reductor. Здесь всё итак ясно. Ориентировочно следующая master-версия будет выпущена в начале ноября 2014.

### Регистрация и активация Carbon Reductor
Данный пункт readonly, выдача регномера и кода активации полностью автоматические, происходят при старте Reductor и каждом обновлении списков URL.

Активационный код меняется раз в 10 дней, поэтому код активации приходит вместе со списками сигнатур.

Состояние активации можно наблюдать в верхней строчке локального меню.

### Запуск обновления списков
Необходимо только после длительного простоя сервера, либо в целях отладки. Инициализирует загрузку актуального реестра РосКомНадзора.

При повторном запуске все предыдущие процессы обновления завершается.

### Статистика работы
В этом пункте можно узнать:

- актуальность списков
- данные о сертификате
- количество пройденных через модуль пакетов, 
- количество фактов блокировки сайтов, 
- состояние регистрации и активации
- версию и время обновления carbon reductor и его веб-интерфейса

### Настройка сканирования трафика
Вызывает мастер настройки сети, который конфигурирует сеть так, чтобы трафик приходящий с коммутатора анализировался модулем фильтрации.

## Настройка приёма зеркала трафика
'''Внимание''' - ничего делать руками не нужно, всё делается автоматически через мастер настройки сети.

Информация ниже - просто информация для понимания того как всё работает.

Трафик с зеркального порта коммутатора ловится в iptables с помощью хитрой комбинации хаков:

- Включается ip_forward = 1
- Отключается rp_filter
- Добавляется правило в ebtables broute BROUTING
- Интерфейс в который приходит зеркало трафика добавляется в bridge
- Включается nf_bridge_call_iptables
- На slave-device в bridge вешается любой IP адрес (для чего приходится патчить /etc/sysconfig/network-scripts/ifup-eth, Carbon Reductor патчит его автоматически при каждом старте).

Важный момент - чтобы зеркало трафика действительно не влияло на нормальный трафик пользователей POLICY для filter FORWARD должен оставаться DROP, иначе - возможны петли в сети.

В общем после настройки сетка обычно должна выглядеть следующим образом:

	ip link
	
	1. lo device
	2. eth0 <нормальный ip с которым сервер уходит в инет>
	3. br0 <бридж с интерфейсом с зеркалом трафика, левый ip, например 169.255.255.243>
	4. eth1 <Интерфейс с зеркалом трафика, левый ip, например 169.255.255.245>

В случае, если не удаётся растэгировать трафик на коммутаторе, придётся добавлять vlan-интерфейсы для каждого тэга.

## Настройка обновления списков РосКомНадзора
Товарищи, как и предполагалось изначально на своём сервере проверяют обращения провайдеров к их серверу с реестром. Тех, кто не скачивают реестр сперва предупреждают о том, что скачивать и фильтровать-то надо, дают на решение этой задачи срок от одного-двух дней до недели, через неделю если ничего не решено - выписывают штраф на 30 000 рублей, ещё через неделю на 60 000 рублей, а потом начинается разговор о лишении лицензии на провайдерскую деятельность.

Для настройки обновления достаточно в локальном меню вписать следующие поля:

	[x] Включить автообновление списков
	Email для связи    [   ]
	ИНН                [   ]
	ОГРН               [   ]
	Название компании  [   ]

Подложить экспортированный закрытый ключ, сохранить настройки, после чего запустить обновление списков.

### Как собственно оно работает
Автоматическое обновление списков работает следующим образом.

- Из данных вводимых в меню и шаблона request.xml.tmplt генерируется файл request.xml в кодировке cp1251
- request.xml с помощью закрытого ключа и openssl с поддержкой libgost подписывается, получаем request.xml.sign. 
- Данный файл используется PHP скриптом send.php, для запроса реестра запрещённых сайтов с сервера РосКомНадзора и в итоге возвращающим архив register.zip
- register.zip распаковывается, получаем из него dump.xml
- dump.xml парсится скриптом dumper.sh и возвращает список конкретных страниц, либо доменов, если конкретной страницы не указано.
- Если включена фильтрация HTTPS то дополнительно скриптом dumper_https.sh извлекаются ip адреса ресурсов доступных по протоколку https.
- Если включена опция использовать nslookup для получения ip-адресов, то nslookup "натравливается" на домен ресурса доступного по протоколу https и для фильтрации используются все ip адреса которые он возвращает.
- из всех списков URL экранируется и генерируется POST-запрос к серверу сигнатур
- сервер сигнатур возвращает ответ в формате "<сигнатура> <URL>" который загружается через procfs в модуль фильтрации в ядре Linux.

## Настройка перенаправления на страницу о блокировке
Перенаправление на страницу о блокировке сайта доступно только на уровне подписки SLA2-сопровождение и SLA3-аутсорсинг, на SLA1-стандарт - пользователю посылается tcp-reset, разрывающий соединение. Реализует его target-модуль netfilter ipt_FORBIDDEN также разработанный Carbon Soft и являющийся частью RPM-пакета.

По сути он реализует внутриядерный TCP-session-hijack для пакетов попавшихся в условие фильтрующего правила.

- Пользователю посылается TCP пакет с флагами PSH/ACK и HTTP 302 Found в data, направляющий на страницу о блокировке. 
- Вслед за ним для надёжности RST-пакет.
- И для максимальной надёжности от имени пользователя в сторону ресурса также посылается RST-пакет.

Для включения необходимо указать следующие опции:

	[x] Включить модуль перенаправления на страницу
	URL для редиректа:  [http://.....]

## Настройка специфичных опций фильтрации трафика
### Блокировка доменов, даже если указана страница
Carbon Reductor будет обрезать GET запрос и блокировать весь домен. Данная опция сделана по запросу одного провайдера, лучше её не использовать, особенно при подключении списков Минюста - будет заблокирован весь vk.com и многие другие сайты.
### Блокировка HTTPS по IP из реестра
При обработке dump.xml из реестра РосКомНадзора будут извлекаться IP адресов https ресурсов, добавляться в ipset, и если dst ip соответствует одному из этих адресов - абоненту запросившему его будет послан tcp-reset.
### Блокировка HTTPS по IP из вывода nslookup по домену из реестра
При обработке dump.xml из реестра РосКомНадзора будут извлекаться домены https ресурсов, для каждого домена будет выполняться nslookup, из его вывода будут извлечены адреса, которые в свою очередь будут добавляться в ipset, и если dst ip соответствует одному из этих адресов - абоненту запросившему его будет послан tcp-reset.
### Фильтрация всего трафика
Увеличивает нагрузку на модуль, так как придётся обрабатывать пакеты в том числе и торрентов, игр и многого другого. Тем не менее в проверке модуля они будут отброшены практически в самом начале, так как не будут иметь заголовков HTTP.

## Настройки значительно ускоряющие интеграцию и упрощающие сопровождение
Очень полезно автоматическое обновление на новые версии при их наличии раз в сутки. Процедура обновления уже обкатана и безопасна, а баги порой исправляются по 10 штук за день. Когда у продукта было 10-20 пользователей - было достаточно легко обновить каждого руками, а вот когда стало значительно больше - за всеми уследить сложно, и заметить людей сидящих со старой версией и не замечающих найденных и исправленных более месяца назад проблем становится ещё сложнее.

Также просто безумно облегчает жизнь и технической поддержке и администратору включение автоматической диагностики с отправкой отчётов об ошибках разработчикам и самому администратору. Ещё больше облегчает опция "пытаться исправлять найденные диагностикой ошибки", которая:

- форсированно пытается обновить списки, если они модифицировались более чем 6 часов назад
- правит параметры ядра для корректной работы, если их изменило стороннее ПО
- включает подробный вывод и советы по решению проблем в диагностике

## Веб-интерфейс
[[Файл:rkn_report.png|300px|thumb|Отчёт о выгрузках списков РКН]]
[[Файл:Carbon_Reductor_diagnostic_web.png|300px|thumb|Диагностика сервера Carbon Reductor]]
[[Файл:carbon_reductor_rkn_update_failure.png|300px|thumb|На нижнем левом графике видно что выгрузки не производились уже 13 часов]]
Веб-интерфейс обычно устанавливается отдельно, по желанию администратора сервера. Для этого в меню "Прочие настройки" есть пункт "Установить веб-интерфейс Carbon Reductor".

Веб-интерфейс разработан с использованием python, flask, jinja, sqlite и metro ui. Крутится на nginx + uwsgi.

На данный момент находится на стадии бета-версии.

Включает в себя:

- графики нагрузки
- просмотр списков сайтов
- просмотр логов: целиком, обновления списков, диагностики
- диагностику
- статистику и информацию о работе carbon reductor
- отчёт о выгрузках списков

# Альтернативные настройки Carbon Reductor
## Для скачивания списков без фильтрации трафика
Некоторые провайдеры с целью "отмазаться" от РосКомНадзора ставят Carbon Reductor с одной целью - скачивать список, чтобы РосКомНадзор не придирался к ним с штрафами. Скажем так, пока что это более менее прокатывает, но скорее всего в скором времени РосКомНадзор начнёт заниматься реальными проверками с помощью "засланных казачков".

Но если необходима кратковременная "затычка", то можно отключить некоторые проверки, чтобы редуктор работал и без зеркала трафика, служа просто утилитой для простого скачивания ссылок.

Официальной опцией это вряд-ли станет.

## Добавление собственных списков
Собственные списки добавить проще некуда: достаточно записать эти URL в файл:

	/usr/local/Reductor/userinfo/our.list

Который сохраняется при обновлении.

Формат файла:

	http://example.ru/directory/file
	http://subdomain1.example.ru/directory/file2
	http://subdomain2.example.ru

## Для работы с зеркалом трафика с маршрутизатора
В принципе ничто не мешает делать зеркало трафика только с 80го порта на маршрутизаторе.

### Linux

Пример как это сделать в статье "Организация съема трафика с Linux сервера для последующего анализа"   [http://habrahabr.ru/post/55256/] {{habr}}

В общем - поставить модуль ipt_TEE или ipt_ROUTE для iptables и добавить соответствующее правило в FORWARD.

Например так:

	iptables -t mangle -A FORWARD -j ROUTE --tee --gw 172.16.1.2

## White-листы
### Для пользователей
	 {{todo-here|text=Как отключить фильтрацию у конкретных пользователей}}
### Для сайтов
Достаточно прописать сайты, которые необходимо блокировать в файле:

	/usr/local/Reductor/list/our.whitelist

Есть и более гибкая настройка белых списков, подробнее о ней можно смотреть в подсказках локального меню.

## Установка дополнительного ПО и модификация файрвола
Правила стандартного файрвола переопределяются Carbon Reductor.

При установке дополнительного ПО может возникнуть проблема с тем, что дополнительному ПО ограничивается доступ в сеть.
Для этого в скрипте файрвола /usr/local/Reductor/bin/firewall.sh предусмотрена строчка:

	[ -x $HOOKDIR/firewall.sh ] && $HOOKDIR/firewall.sh start || true

Поэтому в /usr/local/Reductor/userinfo/hooks/firewall.sh (изначально отсутствует) - пропишите нужные вам правила.

Основной скрипт файрвола править не рекомендуется - при обновлении будет затёрт (хуки при обновлении сохраняются).
А в целом - можно спокойно делать yum install необходимого софта.

# Подбор железа
Минимум две сетёвки - одна для доступа в интернет и отсылки HTTP-302/RST, другая для

'''50-200 Мбит/с.''' Любой сервер на базе Intel (кроме Intel Atom) c двумя сетевыми картами по 1Гбит/с.

'''200-1000 Мбит/с.''' Потребуется процессор с частотой 2 ГГц+, две сетевые карты по 1Гбит/с

'''1-4 Гбит/c.''' 2-4х ядерный процессор, 2.5 ГГц+ на ядро, сетевая карта на 10Гбит/с

'''5-10 Гбит/с.''' 4х ядерный процессор, 3ГГц+ на ядро, сетевая карта на 10Гбит/с

'''10-50 Гбит/с.''' 8х ядерный процессор, 3ГГц+ на ядро, сетевые карты 10Гбит на каждые 10гбит трафика

'''Более 50 Гбит/с.''' Как правило требуется установка нескольких серверов фильтрации, аналогичных описанным в пункте 10-50Гбит/с. Масштабирование не ограничено. Лучше посоветоваться с суппортом.

# Настройка зеркала трафика на коммутаторах
Как уже говорилось [[http://xgu.ru/wiki/Carbon_Reductor#.D0.9D.D0.B0.D1.81.D1.82.D1.80.D0.BE.D0.B9.D0.BA.D0.B0_.D1.81.D0.BA.D0.B0.D0.BD.D0.B8.D1.80.D0.BE.D0.B2.D0.B0.D0.BD.D0.B8.D1.8F_.D1.82.D1.80.D0.B0.D1.84.D0.B8.D0.BA.D0.B0 выше]], Carbon Reductor умеет сканировать трафик направляемый с коммутатора.

Делается это следующим образом.

## Настройка зеркалирования трафика на Сisco:

### Настройка Remote SPAN
Необходимо в случае, если коммутатор-источник зеркалированного трафика находится через несколько коммутаторов от коммутатора-приемника.

На коммутаторе-источнике создаем VLAN:

	vlan 900
	name RSPAN
	remote-span
	Затем создаем monitor session и указываем с какого порта забирать трафик:

	Monitor session 1 source interface FastEthernet 0/2 rx 

Указываем назначение (в нашем случае это RSPAN – VLAN 902)

	Monitor session 1 destination remote vlan 902

Номера session на одном коммутаторе должны совпадать, т.е. указываем откуда берем и куда отправляем наш трафик.

Теперь на всех коммутаторах по пути от нашего коммутатора-источника до коммутатора-приемника создаем vlan 902 и помечаем его как remote-span 

На коммутаторе, к которому подключен Carbon Reductor (Например, к порту Fa0/15) создаем monitor session:

	monitor session 1 source remote vlan 902
	monitor session 1 destination interface Fa0/15

### Настройка обычного SPAN
В случае, когда берем трафик на одном коммутаторе и на нем же его будем зеркалировать в другой порт, VLAN).

	monitor session 1 source interface FastEthernet 0/2 rx 
	monitor session 1 destination interface Fa0/15

Сервер Reductor подключен к Fa0/15.

“Зеркалить” достаточно трафик исходящий от клиентов – ключевое слово “rx”.

После настройки проверяем наличие трафика в iptables, цепочка FORWARD:

	iptables -t filter -nvL FORWARD

## Зеркалирование трафика на D-Link DES 3200

	config mirror port 1 add source ports 2-7 both
	enable mirror

Лучше уточнить direction ingress и тп

	config mirror port 1 add source ports 2-7 tx



# Процесс отладки фильтрации
Перед обращением в поддержку с заявлением о том, что сайты не фильтруются стоит выполнить диагностику, убедиться что всё в порядке, после чего сделать дополнительные [[http://docs.carbonsoft.ru/pages/viewpage.action?pageId=48693470 проверки]].

## Проверить FORWARD

	iptables -nvL FORWARD

Там проверяем счётчики пакетов на правиле с условием reductor и таргетом FORBIDDEN/REJECT, а также на Policy.

	Chain FORWARD (policy DROP 0 packets, 0 bytes)
	 pkts bytes target     prot opt in     out     source               destination         
	  0     0 REJECT     tcp  --  *      *       0.0.0.0/0            0.0.0.0/0           tcp spt:443 match-set https_reductor src reject-with tcp-reset 
	  0     0 REJECT     tcp  --  *      *       0.0.0.0/0            0.0.0.0/0           tcp dpt:443 match-set https_reductor dst reject-with tcp-reset 
	  0     0 FORBIDDEN  tcp  --  *      *       0.0.0.0/0            0.0.0.0/0           tcp dpt:80 UNKNOWN match `reductor' [8 bytes of unknown target data]

Если нули в Policy - значит в FORWARD ещё не попадал трафик.


## Проверить mangle PREROUTING

Также важной информацией является наличие трафика в mangle PREROUTING:

	iptables -t mangle -I PREROUTING -p tcp --dport 80
	iptables -t mangle -nvL PREROUTING

Если счётчики нового правила не растут - значит трафик не попадает в iptables вообще.

## Счётчики увеличиваются но фильтрация не работает
Наиболее вероятная причина - не уходит RST/HTTP 302 пакет от редуктора к пользователю.

На тестовом клиенте и на всех промежуточных устройствах нужно запустить tcpdump или аналог.

	tcpdump -nvi any tcp port 80

и пробовать открыть URL из списка, наблюдая за пакетами (лучше сохранять в файл, потом можно будет прислать их в поддержку для анализа).

## Не работает редирект на страницу

Нужно запустить на тестовом клиенте 

	tshark -c 10 -x -n -V -i any tcp port 80

И смотреть подробное сообщение об ошибке во входящих пакетах. Возможно в настройках в URL на который перенаправляются абоненты не указан протокол (http/https).

# FAQ от разработчиков
## Почему только под CentOS 6 x86_64?
Потому что не хочется иметь заморочек с разношёрстным зоопарком поддерживаемых систем и окружений, а вместо этого улучшать функциальность продукта.

А версии для 32 бит нет, поскольку с 90% вероятностью это означает, что в качестве сервера будет использовано железо, устаревшее настолько, что обычный современный десктоп с хорошими Intel'овскими сетёвками будет работать производительнее и доставит меньше проблем.

## Можно ли с помощью Reductor реализовать услуги в духе детский интернет?
В текущем состоянии нет, но в планах имеется, более того, поддержка раздельных списков уже имеется в master версии.

Можно связаться с компанией разработчиком, сейчас проводится набор провайдеров для пилотных проектов со скидками в будущем.

## Какова производительность фильтрации?
На данный момент - около 900 000 пакетов на 1 ядро процессора в секунду. Потолок с учётом того, что пакет помимо Reductor идёт через сетевой стэк Linux и остальной netfilter - около 60Гбит/сек. Пока не было ни одного сервера упёршегося в потолок производительности, но на этот случай имеется схема с масштабированием - точки съёма зеркала трафика можно сместить чуть подальше от центра сети и разделить таким образом трафик на несколько серверов фильтрации.

Замер проводился на Core i5 и двумя 10Gbit сетёвками.

Проблемы обычно возникают с нагрузкой на сетёвки, что решаемо, при использовании хорошего оборудования.

# Ссылки
[http://docs.carbonsoft.ru/display/reductor Документация]

[http://docs.carbonsoft.ru/pages/viewpage.action?pageId=48037971 Подробное описание алгоритма фильтрации URL] 

[http://www.carbonsoft.ru Оифицальный сайт компании]

[http://www.carbonsoft.ru/products/reductor/carbon-reductor?xgu Страница проекта]

[https://twitter.com/carbon_reductor Twitter проекта]
