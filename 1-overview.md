# Обзор проекта KyaServer

Obsidian view:
	графы
	флоу?
	Алгоритмы?
Figma view?
VR view
draw.io
	интерактивность
	псевдокод

Мозговой штурм
	Создать намерение
		Требование
			М

WSL - Windows
WSL - Server
WSL - Ubuntu
WSL - Docker?

Winderton, S0erdevs

https://www.dive.club/deep-dives/figma-prototyping

# AI Roadmap
#ai #roadmap 



**Свобода воли** [[Дивергентность]] , черный ящик СМ1, алгоритмы и СМ2

https://www.youtube.com/watch?v=v9ejT8FO-7I&list=PLrhzvIcii6GNjpARdnO4ueTUAVR9eMBpc

Декораторы
https://www.youtube.com/watch?v=3tyaO-OE0K0

https://www.youtube.com/watch?v=GWZf_B129zs

https://www.youtube.com/watch?v=KZpYtNtGxSU

На русском
https://www.youtube.com/watch?v=d_zUrg6ELyE

# Pydantic
https://www.youtube.com/watch?v=XIdQ6gO3Anc

GITEA LOCAL
WSL
NVIM?

Справочная информация о проекте?

KyaServer — это монорепозиторий для управления несколькими проектами, включая основное приложение, Docker-контейнеры и документацию.

Цель:
продумать все требования к системе.
Продумать архитектуру.
Система обязана быть отзывчивой?
Как мы это узнаем?

Разработка видётся основная на Python.
Бэкенд на Node.js.

## Основные компоненты

- **main-project/** — Git submodule с основным приложением (Main)
- **wiki/** — Git submodule с документацией (Main.wiki)
- **docker-projects/** — Коллекция Docker-контейнеров:
  - `bindmount-apps/` — Приложение с bind mount
  - `getting-started-app/` — Стартовое приложение с тестами
  - `multi-container-app/` — Мультиконтейнерное приложение
- **tests/** — Тесты корневого уровня
- **compose.yaml** — Docker Compose для локального разворачивания
- **Dockerfile** — Docker-образ для основного приложения

## Технологии

- **Node.js** — Основная платформа разработки
- **Docker** — Контейнеризация приложений
- **Jest** — Фреймворк для тестирования
- **Gitea** — Система управления версиями и CI/CD
- **Git Submodules** — Управление зависимостями между репозиториями

## Структура репозиториев

Проект использует несколько связанных репозиториев на Gitea:

- Main — основное приложение
- Main.wiki — документация
- kyaserver — текущий монорепозиторий
