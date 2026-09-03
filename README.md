# max2tg

**UPD. Форк https://github.com/Aist/max2tg, в который добавил "MAX_EXCLUDE_CHAT_IDS", позволяющий исключать определенные чаты из пересылки. В остальном без изменений.**

Пересылка сообщений из мессенджера **Max** (max.ru) в **Telegram** в реальном времени — с возможностью отвечать обратно.

> **Отказ от ответсвенности:** 
1. Этот проект является независимым, неофициальным и не связан с разработчиками мессенджера Max (или любой другой сторонней организацией). Авторы Max не одобряют, не поддерживают и не несут ответственности за этот код.

2. Программа предоставляется "как есть" (AS IS), без каких-либо гарантий — явных или подразумеваемых, включая, но не ограничиваясь гарантиями товарности, пригодности для конкретной цели или отсутствия ошибок.

3. Авторы не несут ответственности за любые прямые, косвенные, случайные, специальные или последствия ущерба, возникшие в связи с использованием этого ПО, включая потерю данных, доходов или другие убытки, даже если автор был уведомлён о возможности такого ущерба.

4. Использование этого ПО осуществляется исключительно на ваш страх и риск. Рекомендуется самостоятельно проверить код на безопасность и соответствие местному законодательству перед использованием.

5. Этот проект создан в образовательных и исследовательских целях. Авторы не поощряют и не рекомендуют использование для обхода требований государственных органов или нарушения пользовательских соглашений третьих сторон.

---

## Возможности

- Пересылка текстовых сообщений, фото, видео, файлов, аудио, стикеров, контактов, геолокаций и ссылок
- Поддержка пересланных и цитируемых сообщений (forward / reply)
- Разное оформление для личных и групповых чатов
- Ответ из Telegram обратно в Max (опционально, через inline-кнопку)
- Уведомления о статусе соединения с Max — при запуске, потере связи и восстановлении (с троттлингом, чтобы не спамить)
- Поддержка SOCKS5-прокси для подключения к Telegram
- Работает как userbot — подключается к вашему аккаунту Max через WebSocket
- Docker-ready: разворачивается одной командой

## Требования

- Python 3.12+
- Аккаунт в Max (web.max.ru)
- Telegram-бот (создаётся через [@BotFather](https://t.me/BotFather))

## Получение credentials

### Max: токен и device ID

1. Откройте [web.max.ru](https://web.max.ru) в Chrome/Firefox и войдите в свой аккаунт
2. Откройте DevTools: `F12` (или `Cmd+Option+I` на macOS)
3. Перейдите во вкладку **Application** (Chrome) или **Storage** (Firefox)
4. В левой панели: **Local Storage → https://web.max.ru**
5. Найдите и скопируйте значения:
   - `__oneme_auth` → JSON-объект авторизации; для `MAX_TOKEN` возьмите значение поля `token`, а не весь объект
   - `__oneme_device_id` → это ваш `MAX_DEVICE_ID`

> **Важно:** не делитесь этими значениями — они дают полный доступ к вашему аккаунту Max.

### Telegram: токен бота и chat ID

1. Напишите [@BotFather](https://t.me/BotFather) в Telegram → `/newbot` → следуйте инструкциям
2. Скопируйте полученный токен → это ваш `TG_BOT_TOKEN`
3. Узнайте свой chat ID: напишите [@userinfobot](https://t.me/userinfobot) → он ответит вашим ID → это `TG_CHAT_ID`
4. **Важно:** напишите вашему боту `/start`, чтобы он мог вам отправлять сообщения

## Настройка

Скопируйте пример конфигурации и заполните значения:

```bash
cp .env.example .env
```

Содержимое `.env`:

| Переменная      | Обязательная | Описание                                       |
|-----------------|--------------|------------------------------------------------|
| `MAX_TOKEN`     | да           | Токен авторизации Max                          |
| `MAX_DEVICE_ID` | да           | ID устройства Max                              |
| `MAX_CHAT_IDS`  | нет          | список ID чатов Max, разделенных запятой       |
| `MAX_EXCLUDE_CHAT_IDS`  | нет          | список ИСКЛЮЧЕННЫХ из пересылки ID чатов Max, разделенных запятой       |
| `MAX_PROXY`     | нет          | SOCKS5-прокси для подключения к Max (`socks5://host:port`) |
| `TG_BOT_TOKEN`  | да           | Токен Telegram-бота                            |
| `TG_CHAT_ID`    | да           | ID чата, куда пересылать сообщения             |
| `DEBUG`         | нет          | `true` — подробные логи + дамп JSON в `debug/` |
| `REPLY_ENABLED` | нет          | `true` — разрешить ответы из Telegram в Max    |
| `LOG_DIR`       | нет          | Путь к директории логов (по умолчанию `logs`)  |
| `TG_PROXY`      | нет          | SOCKS5-прокси для Telegram (`socks5://host:port`) |
| `TG_READ_TIMEOUT` | нет        | Таймаут чтения HTTP-ответа от Telegram, в секундах |
| `TG_WRITE_TIMEOUT` | нет       | Таймаут отправки обычного запроса к Telegram, в секундах |
| `TG_MEDIA_WRITE_TIMEOUT` | нет | Таймаут загрузки медиафайлов в Telegram, в секундах. Увеличьте, если файлы отправляются повторно из-за медленного прокси |
| `TG_BASE_URL`   | нет          | Адрес своего сервера Telegram Bot API вместо `api.telegram.org` (например `http://localhost:8081`), полезно вместе с telegram-bot-api  |

## Запуск

### Docker (рекомендуется для сервера)

```bash
git clone git@github.com/iam-k0kos/max2tg.git max2tg
cd max2tg
cp .env.example .env
# отредактируйте .env

docker-compose up -d
```

Логи на диске доступны на хосте в директории `./logs/` — файл `max2tg.log` с ротацией по 10 МБ (хранится 5 файлов):

```bash
tail -f logs/max2tg.log
```

### Локальный запуск

#### Linux / macOS

```bash
git clone <repo-url> max2tg
cd max2tg

python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt

cp .env.example .env
# отредактируйте .env

python -m app.main
```

#### Windows (PowerShell)

```powershell
git clone <repo-url> max2tg
cd max2tg

python -m venv .venv
.venv\Scripts\Activate.ps1
pip install -r requirements.txt

copy .env.example .env
# отредактируйте .env

python -m app.main
```

#### Windows (CMD)

```cmd
git clone <repo-url> max2tg
cd max2tg

python -m venv .venv
.venv\Scripts\activate.bat
pip install -r requirements.txt

copy .env.example .env
# отредактируйте .env

python -m app.main
```

### Запуск как systemd-сервис (Linux)

Создайте файл `/etc/systemd/system/max2tg.service`:

```ini
[Unit]
Description=Max to Telegram forwarder
After=network.target

[Service]
Type=simple
WorkingDirectory=/opt/max2tg
ExecStart=/opt/max2tg/.venv/bin/python -m app.main
Restart=always
RestartSec=10
EnvironmentFile=/opt/max2tg/.env

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now max2tg
sudo journalctl -u max2tg -f
```

## Как это работает

```
Max (WebSocket) ──→ max2tg ──→ [SOCKS5 proxy] ──→ Telegram Bot ──→ Ваш чат
                       ↑                                              │
                       └────────── (если REPLY_ENABLED) ──────────────┘
```

1. Приложение подключается к Max через WebSocket как ваш аккаунт
2. Новые входящие сообщения пересылаются в указанный Telegram-чат
3. Если `REPLY_ENABLED=true`, под каждым сообщением появляется кнопка «Ответить» — нажав её, можно написать текст, который отправится обратно в соответствующий чат Max

## Структура проекта

```
max2tg/
├── app/
│   ├── main.py          # точка входа
│   ├── config.py         # загрузка настроек из .env
│   ├── max_client.py     # WebSocket-клиент Max
│   ├── max_listener.py   # обработка и форматирование сообщений
│   ├── resolver.py       # кеш и резолвинг имён контактов/чатов
│   ├── tg_sender.py      # отправка сообщений в Telegram
│   └── tg_handler.py     # обработка ответов из Telegram
├── tests/
│   ├── test_config.py             # тесты загрузки настроек
│   ├── test_max_client.py         # тесты клиента Max (опкоды, парсинг)
│   ├── test_max_listener.py       # тесты форматирования сообщений
│   ├── test_resolver.py           # тесты резолвинга имён контактов
│   ├── test_tg_handler.py         # тесты обработки ответов из Telegram
│   └── test_disconnect_notify.py  # тесты уведомлений о статусе соединения
├── logs/                # логи (создаётся автоматически)
├── .env.example
├── Dockerfile
├── docker-compose.yml
├── pytest.ini
└── requirements.txt
```

---
