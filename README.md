
# 🛰️ XProxy

**XProxy** — сервер для централизованного управления агентами, написанными на C. Он обеспечивает безопасное и масштабируемое взаимодействие между удалёнными клиентами и администратором в режиме реального времени.  
Обмен данными реализован через HTTP short-poll и REST API. Сервер обрабатывает команды, собирает информацию об ОС, процессах, логах клавиатуры и скриншотах, сохраняя данные в базе и управляя очередями команд.

---

## ⚙️ Возможности

- 📡 Поддержка реального времени через HTTP short-poll
- 🖥 Управление агентами:
  - Выполнение и очереди shell-команд
  - Опрос и выдача управляющих команд (скриншоты, выключение, блокировка и др.)
- 🧠 Сбор информации:
  - Состояние ОС (аптайм, память)
  - Активные процессы
  - Скриншоты
  - Клавиатурный ввод
- 🔁 Очереди команд с потокобезопасной реализацией
- REST API для интеграции с интерфейсами администратора

---

## 🧭 REST API

### 🎯 Регистрация и жизненный цикл

| Метод | Путь | Назначение |
|-------|------|------------|
| `POST` | `/register_client` | Регистрация нового агента |
| `GET` | `/clients` | Получение списка клиентов |
| `POST` | `/keep_alive` | Поддержание активности агента |
| `POST` | `/disconnect` | Отключение клиента вручную |

---

### 🧩 Сбор данных

| Метод | Путь | Назначение |
|-------|------|------------|
| `POST` | `/update_computer/:client_id` | Обновление информации об ОС |
| `POST` | `/update_processes/:client_id` | Отправка процессов клиента |
| `GET` | `/get_processes/:client_addr` | Получение процессов |
| `GET` | `/get_osinfo/:client_addr` | Получение инфо об ОС |
| `POST` | `/post_screen` | Отправка скриншота |
| `GET` | `/get_and_remove_screen/:client_addr` | Получение скриншота |
| `POST` | `/post_kbdata` | Отправка логов клавиатуры |
| `GET` | `/get_kbdata/:client_addr` | Получение логов клавиатуры |

---

### 🧨 Командная система

| Метод | Путь | Назначение |
|-------|------|------------|
| `POST` | `/add_command/:cmd_type/:client_addr` | Добавление управляющей команды |
| `DELETE` | `/remove_command/:cmd_type/:client_addr` | Удаление команды |
| `GET` | `/poll_about_command/:cmd_type` | Получение очереди команд (клиент) |
| `GET` | `/poll_status/:cmd_type/:client_addr` | Получение статуса выполнения |
| `POST` | `/update_command/:cmd_type` | Обновление статуса |
| `HEAD` | `/check_failure/:cmd_type` | Проверка на сбой выполнения |

#### Поддерживаемые типы:

- `0` — Выключение компьютера
- `1` — Снятие скриншота
- `2` — Блокировка ввода
- `3` — Сбор данных клавиатуры

---

### 🐚 Shell-очередь

| Метод | Путь | Назначение |
|-------|------|------------|
| `POST` | `/shell/enque/:client_addr` | Поставить команду в очередь |
| `POST` | `/shell/deque/:client_addr` | Удалить выполненную |
| `POST` | `/shell/update_head` | Обновить текущую |
| `GET` | `/shell/get_head_admin/:client_addr` | Получить текущую (админ) |
| `GET` | `/shell/get_head_client` | Получить текущую (агент) |
| `POST` | `/shell/clear/:client_addr` | Очистить очередь клиента |

---

## 🗃️ Структура данных

- **Client**: ID, IP, имя машины, онлайн-статус
- **OSInfo**: доступная и используемая память, аптайм
- **Process**: PID, имя, путь, владелец, использование памяти
- **Command**: тип команды, статус, stdout/stderr, код возврата
- **ShellCommander**: потокобезопасная очередь команд на клиента

---

## 🛠 Технологии

- Go 1.20+
- Gin Web Framework
- PostgreSQL / SQLite
- Встроенные структуры данных (`sync.Map`, `container/list`)

---

## 🚀 Быстрый запуск (Docker Compose)

1. Убедитесь, что установлен Docker и Docker Compose
2. Склонируйте репозиторий

```bash
git clone https://github.com/mans5647/xproxy.git
cd xproxy
````

3. Запустите проект

```bash
docker-compose up --build
```

4. Сервер будет доступен по адресу:
   [http://localhost:10013](http://localhost:10013)

---

## 📦 Структура `docker-compose.yml`

```yaml
version: '3.8'

services:
  xproxy:
    build: .
    ports:
      - "10013:10013"
    depends_on:
      - db
    environment:
      - DB_HOST=db
      - DB_PORT=5432
      - DB_USER=postgres
      - DB_PASSWORD=postgres
      - DB_NAME=xproxy

  db:
    image: postgres:14
    restart: always
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: xproxy
    volumes:
      - pgdata:/var/lib/postgresql/data

volumes:
  pgdata:
```

---

## 📈 Планы на будущее

* Защита API через авторизацию и токены
* Настройка HTTPS и client‑ID проверки
* Метрики и логирование с Grafana/Prometheus
* Использование протокола WebSocket для скорости работы
---
