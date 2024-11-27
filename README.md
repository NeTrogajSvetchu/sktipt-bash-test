###

monitor_test.sh
```
#!/bin/bash

PROCESS_NAME="test"
LOG_FILE="/var/log/monitoring.log"
URL="https://test.com/monitoring/test/api"

if pgrep -x "$PROCESS_NAME" > /dev/null; then

    RESPONSE=$(curl -s -o /dev/null -w "%{http_code}" "$URL")

    if [ "$RESPONSE" -ne 200 ]; then
        echo "$(date): Сервер мониторинга недоступен, код ответа: $RESPONSE" >> "$LOG_FILE"
    fi
else

    exit 0
fi


if [ ! -f /tmp/test_pid ]; then
    
    pgrep -x "$PROCESS_NAME" > /tmp/test_pid
elif ! pgrep -x "$PROCESS_NAME" | grep -f /tmp/test_pid > /dev/null; then
   
    echo "$(date): Процесс '$PROCESS_NAME' был перезапущен." >> "$LOG_FILE"
   
    pgrep -x "$PROCESS_NAME" > /tmp/test_pid
fi
```

###

monitor_test.service 

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
