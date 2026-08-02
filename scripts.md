## МИНИ-ГАЙД ПО СКРИПТАМ

---

### 1. Где хранить?

| Где | Доступ | Пример |
| :--- | :--- | :--- |
| `/usr/local/bin/` | Для всех пользователей (нужен `sudo`) | Главные команды |
| `~/.local/bin/` | Только для тебя | Личные скрипты |
| Любая папка (`~/scripts/`) | Только для тебя | Нужно добавить в `PATH` |

**Как добавить свою папку в PATH** (если выбрал `~/scripts`):
```bash
echo 'export PATH=$PATH:~/scripts' >> ~/.bashrc
source ~/.bashrc
```

---

### 2. Структура скрипта

```bash
#!/bin/bash
# Комментарий

# Переменные
NAME="Мой скрипт"

# Команды
echo "Привет, $NAME!"
```

**Обязательно:**
- Первая строка — `#!/bin/bash` (или `#!/usr/bin/env python3` для Python).
- Сделать исполняемым: `chmod +x /путь/к/скрипту`.

---

### 3. Спецсимволы (самые важные)

| Символ | Что делает |
| :--- | :--- |
| `$@` | Все аргументы командной строки |
| `$1`, `$2` | Первый, второй аргумент |
| `$?` | Код возврата последней команды (0 = успех) |
| `$(...)` | Подстановка результата команды |
| `&&` | Выполнить, если предыдущее успешно |
| `\|\|` | Выполнить, если предыдущее провалилось |

---

### 4. Как запускать программы из скрипта (важный нюанс!)

| Ситуация | Как правильно |
| :--- | :--- |
| Обычная программа | `firefox` или `/путь/к/программе` |
| **AppImage** | `exec /путь/к/файлу.AppImage "$@"` |
| **Программа со своим окружением** (Qt, библиотеки) | Сначала `export LD_LIBRARY_PATH=...`, потом `exec` |
| **Графическая программа, чтобы не занимала терминал** | Добавь `&` в конце: `firefox &` (или `nohup firefox &`) |
| **Скрипт, который принимает параметры** | Используй `"$@"` для передачи всех аргументов |

**Пример скрипта для QtCreator (со своим окружением):**
```bash
#!/bin/bash
export LD_LIBRARY_PATH=/opt/qt6.9/lib:$LD_LIBRARY_PATH
export QT_PLUGIN_PATH=/opt/qt6.9/plugins
exec /opt/qtcreator6.9/bin/qtcreator "$@"
```

**Пример для AppImage (с параметрами):**
```bash
#!/bin/bash
exec /home/r7bd/Applications/osu.AppImage --fullscreen "$@"
```

---

### 5. Красивый вывод (цвета)

```bash
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
NC='\033[0m' # Сброс цвета

echo -e "${GREEN}▶ Успех!${NC}"
echo -e "${RED}▶ Ошибка!${NC}"
```

---

### 6. Готовый пример: скрипт `sysinfo` (минимальный, но красивый)

```bash
#!/bin/bash

# Цвета
GREEN='\033[0;32m'
CYAN='\033[0;36m'
NC='\033[0m'

clear
echo -e "${CYAN}═══════════════════════════════════════════════════${NC}"
echo -e "         📊 СОСТОЯНИЕ СИСТЕМЫ "
echo -e "${CYAN}═══════════════════════════════════════════════════${NC}"

# Время работы
UPTIME=$(uptime -p | sed 's/up //')
echo -e "${GREEN}▶ Время работы:${NC} $UPTIME"

# Загрузка CPU
LOAD=$(uptime | awk -F'load average:' '{print $2}' | cut -d',' -f1 | xargs)
echo -e "${GREEN}▶ Загрузка CPU:${NC} $LOAD"

# Температура (подавляем ошибки)
if command -v sensors &> /dev/null; then
    TEMP=$(sensors 2>/dev/null | grep -i "Package id 0" | awk '{print $4}' | sed 's/+//')
    [ -n "$TEMP" ] && echo -e "${GREEN}▶ Температура CPU:${NC} $TEMP"
fi

# Оперативная память
MEM_TOTAL=$(free -h | grep "^Mem:" | awk '{print $2}')
MEM_USED=$(free -h | grep "^Mem:" | awk '{print $3}')
MEM_AVAIL=$(free -h | grep "^Mem:" | awk '{print $7}')
echo -e "${GREEN}▶ Оперативная память:${NC} $MEM_USED / $MEM_TOTAL (свободно: $MEM_AVAIL)"

# Место на дисках
echo -e "${GREEN}▶ Место на дисках:${NC}"
df -h | grep -E '^/dev/' | awk '{print "  " $6 ": " $3 "/" $2 " (" $5 " занято)"}'

# IP-адрес
IP=$(hostname -I | awk '{print $1}')
echo -e "${GREEN}▶ IP-адрес:${NC} $IP"

# Количество процессов
PROC=$(ps aux --no-heading | wc -l)
echo -e "${GREEN}▶ Процессов:${NC} $PROC"

echo -e "${CYAN}═══════════════════════════════════════════════════${NC}"
```

---

### 7. Как установить этот скрипт

```bash
sudo nano /usr/local/bin/sysinfo
# Вставь содержимое, сохрани (Ctrl+O, Enter, Ctrl+X)
sudo chmod +x /usr/local/bin/sysinfo
```

Теперь запускай командой:
```bash
sysinfo
```

---

### 8. Дополнительные нюансы

| Ситуация | Решение |
| :--- | :--- |
| Скрипт не запускается из меню (Alt+F2) | Убедись, что путь есть в `PATH` (`echo $PATH`). `/usr/local/bin/` там есть по умолчанию. |
| Программа запускается, но с ошибками | Проверь, что в скрипте прописаны все `export` с путями (как для Qt). |
| Хочешь, чтобы скрипт не закрывался сразу | Добавь в конце `read -p "Нажми Enter для выхода"` |
| Хочешь скрыть вывод ошибок | Добавь `2>/dev/null` к команде, например: `sensors 2>/dev/null` |

---

### 9. Пример для AppImage (osu!) с передачей параметров

```bash
#!/bin/bash
exec /home/r7bd/Applications/osu.AppImage --fullscreen "$@"
```

Сохрани как `/usr/local/bin/osu`, сделай исполняемым → запускай `osu` из любого места.

---

### 10. Готовые шаблоны для быстрого копирования

**Шаблон для запуска графической программы:**
```bash
#!/bin/bash
/путь/к/программе &
```

**Шаблон с окружением (для Qt):**
```bash
#!/bin/bash
export LD_LIBRARY_PATH=/opt/qt6.9/lib:$LD_LIBRARY_PATH
exec /opt/qtcreator6.9/bin/qtcreator "$@"
```

**Шаблон с проверкой аргументов:**
```bash
#!/bin/bash
if [ $# -eq 0 ]; then
    echo "Использование: $0 <аргумент>"
    exit 1
fi
echo "Аргумент: $1"
```

---
