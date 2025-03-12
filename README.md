 Cкрипт на bash для мониторинга процесса test в среде linux. Скрипт должен отвечать следующим требованиям:
    1. Запускаться при запуске системы (предпочтительно написать юнит systemd в дополнение к скрипту)
    2.  Отрабатывать каждую минуту
    3.  Если процесс запущен, то стучаться(по https) на https://test.com/monitoring/test/api
    4.  Если процесс был перезапущен, писать в лог /var/log/monitoring.log (если процесс не запущен, то ничего не делать) 
    5.  Если сервер мониторинга не доступен, так же писать в лог.

###

monitor_test.sh
```
#!/bin/bash

# Лог файл
LOG_FILE="/var/log/monitoring.log"

# URL для мониторинга
MONITORING_URL="https://test.com/monitoring/test/api"

# Имя процесса
PROCESS_NAME="test"

# Файл для хранения предыдущего PID
PID_FILE="/var/run/monitor_test.pid"

# Получаем текущий PID процесса
CURRENT_PID=$(pgrep -x "$PROCESS_NAME")

# Проверяем, запущен ли процесс
if [ -n "$CURRENT_PID" ]; then
    # Процесс запущен
    if [ -f "$PID_FILE" ]; then
        # Читаем предыдущий PID из файла
        PREVIOUS_PID=$(cat "$PID_FILE")
        if [ "$CURRENT_PID" != "$PREVIOUS_PID" ]; then
            # PID изменился, процесс был перезапущен
            echo "$(date): Процесс $PROCESS_NAME был перезапущен. Старый PID: $PREVIOUS_PID, новый PID: $CURRENT_PID" >> "$LOG_FILE"
        fi
    else
        # Файл с PID не существует, это первый запуск
        echo "$(date): Процесс $PROCESS_NAME запущен. PID: $CURRENT_PID" >> "$LOG_FILE"
    fi

    # Сохраняем текущий PID в файл
    echo "$CURRENT_PID" > "$PID_FILE"

    # Отправляем запрос на сервер мониторинга
    if curl -s -o /dev/null -w "%{http_code}" "$MONITORING_URL" | grep -q "200"; then
        echo "$(date): Процесс $PROCESS_NAME запущен, сервер мониторинга доступен" >> "$LOG_FILE"
    else
        echo "$(date): Процесс $PROCESS_NAME запущен, но сервер мониторинга недоступен" >> "$LOG_FILE"
    fi
else
    # Процесс не запущен
    if [ -f "$PID_FILE" ]; then
        # Удаляем файл с PID, так как процесс не запущен
        rm "$PID_FILE"
    fi
    echo "$(date): Процесс $PROCESS_NAME не запущен" >> "$LOG_FILE"
fi


Обновленный systemd юнит /etc/systemd/system/monitor_te
```

###

```
[Unit]
Description=Мониторинг процесса test

[Service]
Type=oneshot
ExecStart=/usr/local/bin/monitor_test.sh
```
###

monitor_test.timer

```
[Unit]
Description=Таймер для мониторинга процесса test

[Timer]
OnCalendar=*:0/1
Persistent=true

[Install]
WantedBy=timers.target
```
