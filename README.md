# Pet Project: Мониторинг с CI/CD

![CI/CD](https://img.shields.io/badge/CI/CD-Jenkins-blue?logo=jenkins)
![Automation](https://img.shields.io/badge/Automation-Ansible-black?logo=ansible)
![Infra](https://img.shields.io/badge/Infrastructure-Docker-blue?logo=docker)
![Monitoring](https://img.shields.io/badge/Monitoring-Prometheus-orange?logo=prometheus)

> **Ключевая особенность:** Вся конфигурация инфраструктуры и пайплайн деплоя хранятся в Git.  
> Изменение настроек мониторинга или добавление новых экспортеров, целей происходит автоматически при пуше в репозиторий.

Проект демонстрирует развертывание полноценного стека мониторинга с использованием современного подхода **CI/CD** и принципов **Infrastructure as Code (IaC)**.

---

##  Архитектура

На данный момент проект развёрнут на двух виртуальных машинах:

| Сервер | Роль | Компоненты |
|--------|------|------------|
| **`vm-ansible`** | Control Plane | Jenkins (CI/CD), Ansible, Node Exporter, Process Exporter |
| **`vm-1`** | Monitoring Stack | Prometheus, Grafana, Node Exporter, Process Exporter |

**Схема работы CI/CD:**

1. Разработчик вносит изменения в конфигурации на локальном компьютере и выполняет `git push`.
2. GitHub отправляет Webhook в Jenkins.
3. Jenkins выгружает актуальный код из репозитория.
4. Ansible Playbook доставляет файлы на целевые серверы и перезапускает Docker-сервисы.
5. Изменения применяются автоматически, без ручного вмешательства.

---

##  Стек технологий

| Компонент | Назначение |
|-----------|------------|
| **Git / GitHub** | Версионирование конфигураций, хранение кода, триггер сборок |
| **Jenkins** | Пайплайн деплоя (`Jenkinsfile`), обработка Webhook'ов |
| **Ansible** | Управление конфигурацией (копирование файлов, запуск `docker-compose`) |
| **Docker Compose** | Контейнеризация и оркестрация сервисов мониторинга |
| **Prometheus** | Сбор и хранение метрик с экспортеров |
| **Grafana** | Визуализация данных (дашборды) |
| **Node Exporter** | Метрики операционной системы (CPU, RAM, диск, сеть) |
| **Process Exporter** | Детальные метрики по процессам |


---

##  Текущий функционал

- [x] Развертывание полного стека мониторинга одной командой (`docker-compose up -d`)
- [x] Автоматический деплой изменений через Jenkins + Ansible
- [x] Мониторинг серверов
- [x] Сбор метрик ОС через Node Exporter
- [x] Сбор метрик процессов через Process Exporter
- [x] Визуализация данных в Grafana

---

##  Трудности и их решения

### 1. Конфликт веток `master` и `main`

**Проблема:** Jenkins был настроен на ветку `main`, а GitHub отправлял Webhook на `master`. Сборка падала с ошибкой `couldn't find remote ref refs/heads/master`.

**Решение:**
- Переименовал локальную ветку: `git branch -m master main`
- Обновил настройки Jenkins Pipeline (Branches to build: `*/main`)
- Удалил старую ветку на GitHub и запушлил новую

### 2. Process Exporter создавал директорию вместо файла

**Проблема:** При деплое через Ansible на `vm-1` появлялась директория `/opt/pet-project/process-exporter.yml` вместо файла. Контейнер падал с ошибкой `is a directory`.

**Причина:** Модуль `copy` в Ansible при первом запуске создал директорию (файл отсутствовал в репозитории), и последующие копирования не могли перезаписать её файлом.

**Решение:**
- В плейбук добавлена предварительная очистка:
  ```yaml
  - name: Ensure no file or directory exists at process-exporter.yml path
    file:
      path: /opt/pet-project/process-exporter.yml
      state: absent



## 📈 Планы по развитию
- [ ] **Alertmanager**  
  Настройка алертов при падении сервисов, высокой нагрузке или нехватке дискового пространства. Уведомления в Telegram.
- [ ] **Grafana Provisioning**  
  Автоматическая загрузка дашбордов и источников данных из Git, чтобы настройки Grafana не терялись при пересоздании контейнера.
- [ ] **Сбор логов (Loki + Promtail)**  
  Централизованный сбор и просмотр логов контейнеров прямо в интерфейсе Grafana.
- [ ] **Динамический инвентарь Ansible**  
  Переход от статического списка хостов к динамическому (скрипт или плагин), чтобы автоматически подхватывать новые серверы.
- [ ] **Healthcheck для контейнеров**  
  Добавление `healthcheck` в `docker-compose.yml` для автоматического перезапуска зависших сервисов.
- [ ] **Нотификации Jenkins в Telegram**  
  Получение уведомлений о статусе сборки прямо в мессенджер.
