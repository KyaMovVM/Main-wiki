# Docker и Gitea Actions для test-node.js

## Обзор

Этот документ описывает процесс контейнеризации Node.js сервера (`test-node.js`) и автоматизации его сборки и развертывания через Gitea Actions. Документ предназначен для начинающих разработчиков, которые хотят понять основы Docker и CI/CD.

## Структура проекта

```text
main-project/
├── src/
│   └── test-node.js          # Node.js HTTP сервер
├── Dockerfile                 # Docker образ для test-node.js
└── .gitea/
    └── workflows/
        └── docker-deploy.yml  # Gitea Actions workflow
```

## 1. Исходный код: `src/test-node.js`

Простой HTTP сервер на Node.js, который слушает порт 3010:

```javascript
const http = require("http");

// Используем 0.0.0.0 для доступности из контейнера
const hostname = "0.0.0.0";
const port = 3010;

const server = http.createServer((req, res) => {
  res.writeHead(200, { "Content-Type": "text/plain" });
  res.end("Hello World\n");
});

server.listen(port, hostname, () => {
  console.log(`Server running at http://${hostname}:${port}/`);
});
```

**Важно:** Использование `0.0.0.0` вместо `localhost` необходимо для того, чтобы сервер был доступен извне контейнера.

## 2. Dockerfile: Создание образа Docker

Файл `Dockerfile` содержит инструкции для создания Docker образа нашего Node.js сервера.

### Полный Dockerfile

```dockerfile
FROM node:20-alpine

WORKDIR /app

COPY src/test-node.js /app/test-node.js

EXPOSE 3010

CMD ["node", "test-node.js"]
```

### Пошаговое объяснение

| Инструкция | Описание |
|------------|----------|
| `FROM node:20-alpine` | Используем официальный образ Node.js версии 20 на базе Alpine Linux. Alpine — минимальный дистрибутив (~30 МБ), что делает образ легким и быстрым. |
| `WORKDIR /app` | Устанавливает рабочую директорию `/app` внутри контейнера. Все последующие команды выполняются относительно этой директории. |
| `COPY src/test-node.js /app/test-node.js` | Копирует файл `src/test-node.js` из локальной файловой системы в контейнер по пути `/app/test-node.js`. |
| `EXPOSE 3010` | Объявляет, что контейнер будет слушать порт 3010. Это метаданные для документации; фактическая публикация порта происходит при запуске контейнера. |
| `CMD ["node", "test-node.js"]` | Команда по умолчанию, которая выполняется при запуске контейнера. Запускает Node.js сервер. |

### Локальная сборка и запуск

```bash
# Перейти в директорию main-project
cd main-project

# Собрать Docker образ
docker build -t test-node .

# Запустить контейнер в фоновом режиме
docker run -d -p 3010:3010 --name test-node-container test-node

# Проверить, что сервер работает
curl http://localhost:3010

# Остановить и удалить контейнер
docker stop test-node-container
docker rm test-node-container
```

## 3. Gitea Actions Workflow: Автоматизация сборки и развертывания

Gitea Actions позволяет автоматизировать сборку Docker образа и запуск контейнера при каждом push в репозиторий.

### Создание workflow файла

Создайте директорию `.gitea/workflows` в репозитории `main-project` и добавьте файл `docker-deploy.yml`:

```yaml
name: Build & Deploy test-node

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build-and-run:
    runs-on: self-hosted
    timeout-minutes: 15

    steps:
      - name: Checkout repository
        uses: actions/checkout@v3

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v2

      - name: Build Docker image
        run: |
          docker build -t test-node:${{ github.sha }} .

      - name: Stop existing container (if any)
        run: |
          docker rm -f test-node-${{ github.run_id }} || true

      - name: Run container
        run: |
          docker run -d --rm \
            -p 3010:3010 \
            --name test-node-${{ github.run_id }} \
            test-node:${{ github.sha }}

      - name: Verify service is reachable
        run: |
          sleep 5
          curl -f http://localhost:3010 || exit 1

      - name: Stop container after job
        if: always()
        run: |
          docker rm -f test-node-${{ github.run_id }} || true
```

### Объяснение ключевых элементов

#### Триггеры

```yaml
on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]
```

Workflow запускается при:

- Push в ветку `main`
- Создании Pull Request в ветку `main`

#### Конфигурация job

```yaml
runs-on: self-hosted
timeout-minutes: 15
```

- **`runs-on: self-hosted`** — workflow выполняется на вашем собственном Gitea runner. На машине с runner должен быть установлен Docker.
- **`timeout-minutes: 15`** — ограничивает время выполнения workflow до 15 минут. После этого job автоматически завершится.

#### Шаги выполнения

1. **Checkout** — получает код из репозитория
2. **Docker Buildx** — настраивает Docker для сборки (поддержка multi-arch)
3. **Build** — собирает Docker образ с тегом, включающим SHA коммита
4. **Stop existing** — останавливает предыдущий контейнер (если есть)
5. **Run** — запускает новый контейнер в фоновом режиме
6. **Verify** — проверяет доступность сервера через curl
7. **Cleanup** — останавливает контейнер после завершения workflow

### Настройка Gitea Runner

Для работы workflow необходим настроенный Gitea runner:

1. **Установите Gitea Actions Runner** на машине с Docker
2. **Зарегистрируйте runner** в настройках репозитория Gitea
3. Убедитесь, что Docker доступен из runner'а

Подробнее: [Документация Gitea Actions](https://docs.gitea.com/usage/actions/overview)

## 4. Использование переменных окружения

Если нужно передать переменные окружения в контейнер:

```yaml
- name: Run container
  run: |
    docker run -d --rm \
      -p 3010:3010 \
      -e NODE_ENV=production \
      -e PORT=3010 \
      --name test-node-${{ github.run_id }} \
      test-node:${{ github.sha }}
```

Или использовать секреты Gitea:

```yaml
- name: Run container
  env:
    API_KEY: ${{ secrets.API_KEY }}
  run: |
    docker run -d --rm \
      -p 3010:3010 \
      -e API_KEY="$API_KEY" \
      --name test-node-${{ github.run_id }} \
      test-node:${{ github.sha }}
```

## 5. Публикация образа в реестр

Если нужно сохранить Docker образ в реестре (например, для последующего использования):

```yaml
- name: Log in to Docker registry
  uses: docker/login-action@v2
  with:
    registry: ghcr.io
    username: ${{ secrets.REGISTRY_USER }}
    password: ${{ secrets.REGISTRY_PASSWORD }}

- name: Build and push Docker image
  run: |
    docker build -t ghcr.io/your-org/test-node:${{ github.sha }} .
    docker push ghcr.io/your-org/test-node:${{ github.sha }}
```

## Troubleshooting

### Контейнер не запускается

1. Проверьте логи: `docker logs test-node-container`
2. Убедитесь, что порт 3010 не занят: `netstat -an | grep 3010`
3. Проверьте, что сервер слушает на `0.0.0.0`, а не `localhost`

### Workflow падает на этапе Verify

1. Увеличьте время ожидания перед проверкой (больше `sleep`)
2. Проверьте, что контейнер действительно запустился: `docker ps`
3. Убедитесь, что порт правильно проброшен: `-p 3010:3010`

### Runner не запускает workflow

1. Проверьте статус runner'а в настройках Gitea
2. Убедитесь, что runner имеет доступ к Docker: `docker ps`
3. Проверьте логи runner'а

## Полезные команды Docker

```bash
# Просмотр запущенных контейнеров
docker ps

# Просмотр всех контейнеров (включая остановленные)
docker ps -a

# Просмотр логов контейнера
docker logs test-node-container

# Вход в запущенный контейнер
docker exec -it test-node-container sh

# Остановка контейнера
docker stop test-node-container

# Удаление контейнера
docker rm test-node-container

# Удаление образа
docker rmi test-node

# Просмотр использования ресурсов
docker stats
```

## Дополнительные улучшения

### Multi-stage build (для оптимизации)

```dockerfile
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

FROM node:20-alpine
WORKDIR /app
COPY --from=builder /app/node_modules ./node_modules
COPY src/test-node.js ./test-node.js
EXPOSE 3010
CMD ["node", "test-node.js"]
```

### Health check

Добавьте health check в Dockerfile:

```dockerfile
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD node -e "require('http').get('http://localhost:3010', (r) => {process.exit(r.statusCode === 200 ? 0 : 1)})"
```

### Использование .dockerignore

Создайте файл `.dockerignore` для исключения ненужных файлов:

```text
node_modules
.git
.gitignore
README.md
*.md
.env
```

## Заключение

Теперь у вас есть:

- ✅ Docker образ для Node.js сервера
- ✅ Автоматизированная сборка и развертывание через Gitea Actions
- ✅ Ограничение времени выполнения workflow (15 минут)
- ✅ Автоматическая очистка контейнеров после завершения

После каждого push в ветку `main` workflow автоматически соберёт образ, запустит контейнер, проверит его работоспособность и остановит через 15 минут.
