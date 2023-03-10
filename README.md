## Задание 1. Виды резервного копирования
### Full BackUP
Делает полное бэкапирование всего
- Самый долгий по продолжительности создания
- Дает нагрузку на диски и сеть, если бэкап сетевой
- Самый надежный и быстрый с точки зрения восстановления данных.
![full-backup](https://github.com/RSafin12/Screenshots/blob/main/full_backup.png)

### Диффернцированное резервное копирование
1. Сначала делаем Full BackUp
2. Затем при каждом запуске идет сохранение только измененых данных. Но точкой отсчета считает именно первый full-бэкап
Создание копии идет быстрее чем при full-бэкапе, но медленнее чем инкрементное. Восстановление наоборот.

В данной схеме хорошей практикой считает иногда делать фулл-бэкап, чтобы после фулл-бэкапа не было супер много дифф-бэкапов.
![diff-backup](https://github.com/RSafin12/Screenshots/blob/main/diff_backup.png)

### Инкрементное бэкапирование
Работает также как и дифференцильное, но точкой отсчета является последний слепок, а не первичный фулл-бэкап.

![inc-backup](https://github.com/RSafin12/Screenshots/blob/main/inc_backup.png)

## Задание 2
Установить Bacula дело несложное, комментировать не буду, сервисы после этого сразу стартанули без проблем со своими дефолтными конфигами.
Однако бэкапить сервер на сам сервер это не путь джедая.
![mem](https://github.com/RSafin12/Screenshots/blob/main/mem.png)

Поэтому я задумал создать хотя бы 2 VM, чтобы протестировать бэкапирование по сети. План был такой
- Нода brave основная, она бэкапить сама себя и является стораджем для второй
- Нода strong должна бэкапиться по сети на ноду brave

Пока удалось настроить простейшее бэкапирование основной ноды brave саму на себя
Все конфигуарционные файлы размещены в директории node-brave
Вывод команды bconsole -> status -> 6 я положил в файле output. Там у меня не сразу получились бэкапы, но последние 2 со статусом ОК. И сами бэкапы появились в заданной директории

![result](https://github.com/RSafin12/Screenshots/blob/main/result_backup.png)


Добавил файл theory с моими теоретическими заметками по теме.

### Дополнение

#### Настройка второго удаленного клиента

Создал в одной локальной сети вторую VM - strong
Настроить удаленный клиент в целом дело несложное, конфиг bacula-fd:
https://github.com/RSafin12/10.4-Backup-Bacula/blob/main/node-strong/bacula-fd.conf

Единственно важно помнить, что если удаленный FD не коннектится к SD, то нужно для SD указать внешний IP, в моем случае 172.16.16.16
По итогу бэкапирование также работает и для удаленного клиента
![2nd_result](https://github.com/RSafin12/Screenshots/blob/main/result_strong.png)

#### Настройка восстановления

Я воспользовался дефолтным Job для отката из бэкапа.

```
Job {
  Name = "RestoreFiles"
  Type = Restore
  Client=brave-fd
  Storage = stor-1
  FileSet="Full Set"
  Pool = def-pool
  Messages = Standard
  Where = /backup/restore
  Write Bootstrap = "/var/lib/bacula/%c.bsr"
}
```
RestoreJob по итогу работает корректно
![jobs_status](https://github.com/RSafin12/Screenshots/blob/main/status_restore.png)
В оригинальный файл я добавил строку
*So, let's have a look *
В бэкапе ее уже нет.
![jobs_status2](https://github.com/RSafin12/Screenshots/blob/main/restore_result.png)

## Задание 3

Результат выполнения скрипта из урока, конфиг rsync по факту дефолтный
![rsync](https://github.com/RSafin12/Screenshots/blob/main/rsync_res.png)
