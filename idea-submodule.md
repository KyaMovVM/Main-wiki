# Настройка Git Submodules для Workspace

## Обзор

Проект использует структуру с несколькими связанными репозиториями на GitHub:

- **Main** (`main-project/`) — основное приложение
- **Main-wiki** (`wiki/`) — документация проекта
- **kyaserver** — монорепозиторий, объединяющий оба репозитория через Git submodules

## Структура репозиториев

```text
kyaserver/                    # Главный репозиторий
├── main-project/             # Git submodule → Main
├── wiki/                     # Git submodule → Main.wiki
├── .gitmodules               # Конфигурация submodules
└── kyaserver.code-workspace  # VS Code workspace файл
```

## Текущая конфигурация

### Файл `.gitmodules`

```ini
[submodule "main-project"]
    path = main-project
    url = https://github.com/KyaMovVM/Main.git

[submodule "wiki"]
    path = wiki
    url = https://github.com/KyaMovVM/Main-wiki.git
```

### VS Code Workspace (`kyaserver.code-workspace`)

```json
{
    "folders": [
        {
            "path": "."
        },
        {
            "path": "main-project/Japanese game"
        }
    ],
    "settings": {
        "liveServer.settings.multiRootWorkspaceName": "Japanese game"
    }
}
```

## Быстрый старт

### 1. Клонирование репозитория с submodules

**Рекомендуемый способ** (автоматически инициализирует все submodules):

```bash
git clone --recurse-submodules https://github.com/KyaMovVM/kyaserver.git
cd kyaserver
```

### 2. Если репозиторий уже клонирован без submodules

```bash
cd kyaserver
git submodule update --init --recursive
```

### 3. Открытие Workspace в VS Code

После клонирования и инициализации submodules откройте файл `kyaserver.code-workspace` в VS Code:

```bash
code kyaserver.code-workspace
```

## Работа с Submodules

### Обновление submodules до последних версий

```bash
# Обновить все submodules до коммитов, указанных в главном репозитории
git submodule update --remote

# Или обновить конкретный submodule
git submodule update --remote main-project
git submodule update --remote wiki
```

### Добавление нового submodule

Если нужно добавить новый submodule:

```bash
git submodule add https://github.com/KyaMovVM/NewRepo.git path/to/new-repo
git add .gitmodules path/to/new-repo
git commit -m "Add NewRepo as submodule"
git push
```

### Обновление ссылки на submodule

Когда в submodule появились новые коммиты, нужно обновить ссылку в главном репозитории:

```bash
cd main-project  # или cd wiki
git checkout main  # или нужная ветка
git pull

cd ..  # вернуться в корень kyaserver
git add main-project  # или wiki
git commit -m "Update submodule to latest version"
git push
```

## Важные замечания

1. **Не переименовывайте папки submodules** — это сломает ссылки в `.gitmodules` и workspace файлах.

2. **При клонировании всегда используйте `--recurse-submodules`** или выполняйте `git submodule update --init --recursive` после обычного клона.

3. **Submodules указывают на конкретные коммиты**, а не на ветки. Чтобы обновить submodule до последней версии, используйте `git submodule update --remote`.

4. **VS Code workspace** автоматически отобразит все папки, указанные в `folders`, включая submodules.

## Альтернативный способ (без submodules)

Если по каким-то причинам вы не хотите использовать submodules, можно создать скрипт для клонирования:

**`scripts/clone-repos.sh`** (для Linux/Mac):

```bash
#!/usr/bin/env bash
# Клонирование репозиториев без использования submodules
set -e

ROOT_DIR="$(cd "$(dirname "$0")/.." && pwd)"
cd "$ROOT_DIR"

[ -d main-project ] || git clone https://github.com/KyaMovVM/Main.git main-project
[ -d wiki ] || git clone https://github.com/KyaMovVM/Main-wiki.git wiki

echo "Готово. Откройте kyaserver.code-workspace в VS Code."
```

**`scripts/clone-repos.ps1`** (для Windows):

```powershell
# Клонирование репозиториев без использования submodules
$repoUrl = "https://github.com/KyaMovVM"

if (-not (Test-Path "main-project")) {
    git clone "$repoUrl/Main.git" main-project
}

if (-not (Test-Path "wiki")) {
    git clone "$repoUrl/Main-wiki.git" wiki
}

Write-Host "Готово. Откройте kyaserver.code-workspace в VS Code."
```

## Полезные команды

```bash
# Проверить статус всех submodules
git submodule status

# Посмотреть информацию о submodules
git submodule

# Перейти в submodule и работать с ним как с обычным репозиторием
cd main-project
git status
git checkout -b feature-branch
# ... работа ...
git push origin feature-branch
cd ..
```

## Troubleshooting

### Submodule показывает как изменённый после клона

Это нормально, если submodule указывает на коммит, который не является HEAD ветки. Чтобы обновить до последней версии:

```bash
git submodule update --remote
```

### Ошибка "fatal: reference is not a tree"

Означает, что коммит, на который ссылается submodule, не существует. Обновите ссылку:

```bash
git submodule update --init --recursive --remote
```

### VS Code не видит файлы в submodule

Убедитесь, что:

1. Submodules инициализированы: `git submodule update --init --recursive`
2. Workspace файл содержит правильные пути к папкам submodules
3. Перезагрузите окно VS Code: `Ctrl+Shift+P` → "Reload Window"
