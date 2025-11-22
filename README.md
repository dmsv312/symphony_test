# Shopware 6 Docker dev stack

Готовая docker-compose инфраструктура для локальной разработки плагина под Shopware 6. После копирования `.env.example` и запуска контейнеров вы устанавливаете чистую Shopware в папку `shopware/` и можете сразу писать плагин, запускать консоль и собирать ассеты.

## Структура проекта
```
.
├── docker-compose.yml
├── .env.example
├── .gitignore
├── README.md
├── docker
│   ├── nginx
│   │   ├── default.conf
│   │   └── Dockerfile
│   ├── node
│   │   └── Dockerfile
│   └── php
│       └── Dockerfile
└── shopware/            # код Shopware будет установлен сюда (bind mount)
```

## Предварительные требования
- Docker Engine + Docker Compose v2 (Linux/macOS/Windows с Docker Desktop)
- Свободные порты: по умолчанию 8080 (frontend/admin), 8081 (Adminer), 3306 (MySQL)

## Быстрый старт
1. Скопируйте файл окружения и при необходимости обновите пароли/порты:
   ```bash
   cp .env.example .env
   ```
2. Запустите контейнеры в фоне:
   ```bash
   docker compose up -d --build
   ```
3. Зайдите в контейнер PHP:
   ```bash
   docker compose exec php bash
   ```
4. Внутри контейнера установите Shopware 6 (в пустую папку `/var/www/shopware`):
   ```bash
   composer create-project shopware/production .
   ```
5. Скопируйте `.env` Shopware из примера и задайте параметры БД/URL под поднятые сервисы (можно через редактор или командой):
   ```bash
   cat <<'EOT' > .env.local
   APP_ENV=${APP_ENV:-dev}
   APP_URL=${APP_URL:-http://localhost:8080}
   DATABASE_URL="mysql://${DB_USER:-shopware}:${DB_PASSWORD:-shopware}@${DB_HOST:-db}:${DB_PORT:-3306}/${DB_NAME:-shopware}"
   MAILER_DSN=null://null
   EOT
   ```
6. Выполните установку и базовую настройку:
   ```bash
   bin/console system:install --basic-setup --create-database --force
   ```
7. Создайте первого администратора (если мастер установки не сделал это автоматически):
   ```bash
   bin/console user:create admin --admin --password=Admin123 --email=admin@example.com
   ```
8. Соберите темы/ассеты (можно и позже при разработке):
   ```bash
   bin/console theme:compile
   ```
9. Откройте витрину: http://localhost:8080, админка: http://localhost:8080/admin

## Работа с окружением
- Остановка контейнеров: `docker compose down`
- Перезапуск после изменений Dockerfile: `docker compose up -d --build`
- Консоль Shopware: `docker compose exec php bin/console ...`
- Установка зависимостей PHP: `docker compose exec php composer install`
- Сборка ассетов (admin/storefront) через Node:
  ```bash
  docker compose run --rm node npm install
  docker compose run --rm node npm run build
  ```
- БД через Adminer: http://localhost:8081 (сервер: `db`, пользователь/пароль берите из `.env`).

## Примечания по разработке плагина
- Каталог проекта монтирован во все нужные контейнеры, поэтому код плагина (`shopware/custom/plugins/...`) доступен сразу и PHP, и Node.
- Права на файлы наследуются от хоста; при необходимости можно выполнить внутри контейнера `chown -R www-data:www-data /var/www/shopware` после установки.
- Для создания каркаса плагина используйте: `docker compose exec php bin/console plugin:create YourPluginName`.
