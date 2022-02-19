# RCE-Windows
New Modified RCE Windows
-------------------------------------------------
All scripts require python 3.8

Starting the scanner:
===============

python3 scanner.py [-in <in_file_path>] [-out <out_file_path>] [-threads <max_cnt>] [-proxy <host>:<port>] [-timeout <secs>] [-deep <num_deep_scan_iters>]

All parameters are optional:

<in_file_path> - path to the input file (scanner_input.txt by default). Each non-empty line is in one of 5 options:

  1.a.b.c.d
     Specifies 1 IP to check
  2. a.b.c.d - x
     Specifies the IP range from a.b.c.d to a.b.c.x
  3. a.b.c.d - x.y
     Specifies the IP range from a.b.c.d to a.b.x.y
  4. a.b.c.d - x.y.z
     Specifies the IP range from a.b.c.d to a.x.y.z
  5. a.b.c.d/n
     Specifies the range of all IP addresses with n high bits fixed

<out_file_path> - path to the output file where IPs of vulnerable machines will be written (scanner_output.txt by default)

<max_cnt> - the maximum number of threads used, by default 800, this corresponds to the maximum number of threads per
    one process in Windows. For Linux, you can specify at least 3000 threads. The main thing is not to overload your proxy.

<host>:<port> is an optional SOCKS5 proxy through which the check will be passed.

<secs> - IP connection timeout and for read operations. Too low a value may cause the machine to skip,
     too high - to increase check time. Use the optimal value based on the "slowness" of your
     proxy and number of threads.

<num_deep_scan_iters> - if not 0, then the scanner will more thoroughly check machines for vulnerabilities, thus discarding
     false detections. Use if the scan reveals a lot of vulnerable machines, about which, when launched,
     exp it turns out that they are not vulnerable. Usually 5 iterations are enough, i.e. -deep 5 (default)
     in most cases will filter out all false detections.

When checking, each valid, in addition to writing to a file, is accompanied by a log to the console of the form:

[+] 192.168.0.141 is vulnerable (2/161 checked, 14 left)

Run exp:
=============

python3 exploit.py -ip <target_ip> [-port <taget_port>] [-file] [-proxy <host>:<port>] [-timeout <secs>] -payload <exe_or_dll_path> [-system]

<target_ip> and <taget_port> (default standard 445) specify the target machine.

If the '-file' option is specified, then <target_ip> is the path to a file containing a list of IP addresses to attack, one IP address
per line. The result of the scanner operation (scanner_output.txt) is suitable as a file.

<host>:<port> is an optional SOCKS5 proxy that the machine will ping through.

<secs> - IP connection timeout and for read operations

<exe_or_dll_path> is the path to the 64-bit EXE or DLL file. The payload will be executed fileless in one of the user processes,
    whose search rules are as follows:

    1. If a process with admin privileges is found, it will be used
    2. Otherwise, if a process explorer.exe with normal rights is found, it will be used
    3. Otherwise, if another process with normal rights is found, it will be used
    4. Otherwise, any of the remaining Jurassic processes will be used
    5. If no user processes are found (you never know), then the exp will end with an error.

If the '-system' option is specified, then the payload will be executed in the context of the system process (lsass.exe).

In this case, if the payload is not an executable (MZ-) file, it will be executed as a shellcode.

Executing an exp:
=================

The exp reports each step to the console and keeps in touch with the nuclear shellcode until the moment it transfers control to
user mode. If the shellcode detected some kind of error (for example, it could not allocate memory, create a thread, etc.), then
he will report this error to the exp and he will print it to the console. So if the exp has successfully completed, then you can be
we are at least sure that the usermode loader has received control and started the process of fileless launch of the payload.

EXP Parameters:
================

The lib\cfg.py file contains parameters that may need to be adjusted for a particular environment or machine, although
they are already set to average optimal values:

- NUM_THREADS
  This is the number of exploit threads. If the number of cores on the server is much greater than this value, then occasionally
  errors like [-] failed to <...> appear. In this case, the value of NUM_THREADS must be increased. Or vice versa, his
  can be reduced if a smaller value is enough not to overload the proxy.

- SRV2BASE_LEAK_TIMEOUT and NUM_SRV2BASE_LEAK_ATTEMPTS
  Several cycles are used to determine the base address of srv2.sys, at the end of each of which a flush occurs
  states - to improve the probability of penetration. SRV2BASE_LEAK_TIMEOUT indicates the length of each cycle, and
  NUM_SRV2BASE_LEAK_ATTEMPTS - their maximum number.

- READ_TIMEOUT and NUM_READ_ATTEMPTS
  Sometimes read operations fail, these values ​​indicate the timeout of one read operation and the maximum number
  retries for the read operation.

- RESTORE_CRITICAL_STRUCTS
  Indicates restored
  ------------------------------------------------------------------------------------
  Для работы всех скриптов требуется питон 3.8

Запуск сканера:
===============

python3 scanner.py [-in <in_file_path>] [-out <out_file_path>] [-threads <max_cnt>] [-proxy <host>:<port>] [-timeout <secs>] [-deep <num_deep_scan_iters>]

Все параметры опциональны:

<in_file_path> - путь ко входному файлу (по умолчанию scanner_input.txt). Каждая непустая строка - в одном из 5-и вариантов:

  1. a.b.c.d
     Указывает 1 IP для провеки
  2. a.b.c.d - x
     Указывает диапазон IP от a.b.c.d до a.b.c.x
  3. a.b.c.d - x.y
     Указывает диапазон IP от a.b.c.d до a.b.x.y
  4. a.b.c.d - x.y.z
     Указывает диапазон IP от a.b.c.d до a.x.y.z
  5. a.b.c.d/n
     Указывает диапазон всех IP адрксов с n фиксированными старшими битами

<out_file_path> - путь к выходному файлу, куда будут писаться IP уязвимых машин (по умолчанию scanner_output.txt)

<max_cnt> - максимальное число используемых потоков, по умолчанию 800, это соответствуеют максимальному число потоков на
    один процесс в винде. Для линукса можно указывать хоть 3000 потоков. Главное не перегрузите ваш прокси.

<host>:<port> - это опциональный SOCKS5 прокси, через который будет прокидываться чек.

<secs> - таймаут подключения к IP и для операций чтения. Слишком низкое значение может привести к пропуску машины,
     слишком высокое - к увеличению времени чека. Используйте оптимальное значение исходя из "тормознутости" вашего
     прокси и числа потоков.

<num_deep_scan_iters> - если не 0, то сканер будет более тщательно провреять машины на уязвимость, отбрасывая т.о.
     ложные детекты. Используйте, если при сканировании обраруживается много уязвимых машин, про которые при запуска
     экспа выясняется, что они не являются уязвимыми. Обычно хватает 5-и итераций, т.е. -deep 5 (значение по умолчанию)
     в большинстве случаев отсеит все ложные детекты.

При проверке каждый валид, помимо записи в файл, сопровождается логом в консоль вида:

[+] 192.168.0.141 is vulnerable (2/161 checked, 14 left)

Запуск экспа:
=============

python3 exploit.py -ip <target_ip> [-port <taget_port>] [-file] [-proxy <host>:<port>] [-timeout <secs>] -payload <exe_or_dll_path> [-system]

<target_ip> и <taget_port> (по умолчанию стандартный 445) задают целевую машину.

Если указана опция '-file', то <target_ip> - это путь к файлу, содержащему список IP адресов для атаки, по одному IP адресу
на строчку. В качестве файла подходит результат работы сканера (scanner_output.txt).

<host>:<port> - это опциональный SOCKS5 прокси, через который машина будет пробиваться.

<secs> - таймаут подключения к IP и для операций чтения

<exe_or_dll_path> - это путь к 64-битному EXE или DLL-файлу. Пейлоад будет выполнен бесфайлово в одном из юзерских процессов,
    правила поиска которого следующие:

    1. Если найден процесс с админсиким привилегиями, то будет использоваться он
    2. Иначе если найден процесс explorer.exe с обычными правами, о будет использоваться он
    3. Иначе если найден другой процесс с обычными правами, то будет использоваться он
    4. Иначе будет использоваться любой из оставшихся юреских процессов
    5. Если юзерских процессов не найдено (мало ли), то эксп завершится с ошибкой.

Если указана опция '-system', то пейлоад будет исполнен в контексте системного процесса (lsass.exe).

При этом если пейлоад не является исполняемым (MZ-) файлом, то будет исполнен как шеллкод.

Выполнение экспа:
=================

Эксп репортит каждый свой щаг в консоль и держит связь с ядерным шеллкодом вплоть до момента передачи им управления в
пользовательский режим. Если шеллкод задетектил какую-то ошибку (например, не смог выделить память, создать поток и т.д.), то
он отрепортит эту ошибку экспу и тот выведет ее в консоль. Так что если эксп успешно доработал до конца, то вы можете быть
уверены как минимум в том, что юзермодный загрузчик получил управление и начал процесс бесфайлового запуска пейлоада.

Параметры экспа:
================

В файле lib\cfg.py находятся параметры, которые может потребоваться поднастроить под конкретную среду или машину, хотя
они уже установлены в средние оптимальные значения:

- NUM_THREADS
  Это число потоков эксплойта. Если на сервере кол-во ядер сильно больше данного значения, то могут изредка
  появляться ошибки вида [-] failed to <...>. В этом случае значение NUM_THREADS необходимо увеличить. Либо наоборот, его
  можно уменьшить, если меньшего значения хватает, чтобы не перенагружать прокси.

- SRV2BASE_LEAK_TIMEOUT и NUM_SRV2BASE_LEAK_ATTEMPTS
  Для определения базового адреса srv2.sys используется несколько циклов, в конце каждого из которых происходит сброс
  состояния - для улучшения вероятности пробива. SRV2BASE_LEAK_TIMEOUT указывает продолжительность каждого цикла, а
  NUM_SRV2BASE_LEAK_ATTEMPTS - их максимальное количество.

- READ_TIMEOUT и NUM_READ_ATTEMPTS
  Иногда операции чтения фейлятся, данные значения указывают таймаут одной операции чтения и максимальное количество
  повторов для операции чтения.

- RESTORE_CRITICAL_STRUCTS
  Указывает восстанавливать ядерные структуры, которые портит эксп, в их исходное состояние. При установке этой опции
  эксп работает стабильнее, но иногда заблюдаются ошибка [-] memory read failure. При ее стабильном повторении рекомендуется
  установить опцию RESTORE_CRITICAL_STRUCTS в False.

Другие параметры относятся к advanced настройкам и в большинстве случаев их менять нет необходимости.

Известные проблемы:
===================

1. Наблюдались проблемы с пробивом экспа машины, на которой поднят BC. В этом случае используйте прямое подключение к машине.
2. Возможен BSOD целевой машине, если выполнение экспа было прервано и он не сумел произвести нужную очистку. Потому старайтесь
   выполнение экспа не прерывать.
3. Если через прокси машина упорно не пробивается и есть возможность подключиться по RDP к сетке, то рекомендуется на подключенную
   машину установить python 3.8 и пробить целевую машину без прокси. Вообще, рекомендуется как можно больше сократить расстояние
   от атакующего и атакуемого компьютера, это напрямую влияет на качество пробива.
4. На некоторых машинах эксп не может пройти стадию определения базы SRV2.SYS несмотрия на то, что целевая машина уязвима. Вероятная
   причина такой проблемы - это настройки/конфигурация или тип сетевого интерфейса, в результате чего в транпортных заголовках пакетов
   не попадается нужный шаблон.

Тестовые пейлоады:
------------------

1. hello.exe - выведет message box и создаст процесс notepad.exe
2. rdp64.dll - создаст локального админа с логопассом Support452:123$Qwerty (требует админиских прав, запускать с ключом -system 1)

  
  ----------------------------------------------------------------------------------------
  
  
  
