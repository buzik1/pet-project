# Monitoring with CI/CD

![CI/CD](https://img.shields.io/badge/CI/CD-Jenkins-blue?logo=jenkins)
![Automation](https://img.shields.io/badge/Automation-Ansible-black?logo=ansible)
![Infra](https://img.shields.io/badge/Infrastructure-Docker-blue?logo=docker)
![Monitoring](https://img.shields.io/badge/Monitoring-Prometheus-orange?logo=prometheus)
![Alerting](https://img.shields.io/badge/Alerting-Alertmanager-red?logo=prometheus)
![Chat](https://img.shields.io/badge/Chat-Slack-4A154B?logo=slack)
![Loki](https://img.shields.io/badge/Logs-Loki-orange?logo=grafana)
![Visualization](https://img.shields.io/badge/Visualization-Grafana-orange?logo=grafana)
![Secrets](https://img.shields.io/badge/Secrets-Ansible_Vault-black?logo=ansible)

> **Ключевая особенность:** Вся конфигурация инфраструктуры и пайплайн деплоя хранятся в GitHub.  
> Изменение настроек мониторинга или добавление новых инструментов, целей, экспортеров происходит автоматически при пуше в репозиторий.

Проект демонстрирует развертывание полноценного стека мониторинга с использованием подхода **CI/CD** и принципов **Infrastructure as Code (IaC)**.

---

##  Архитектура

На данный момент проект развёрнут на двух виртуальных машинах:

| Сервер | Роль | Компоненты |
|--------|------|------------|
| **`vm-ansible`** | Control Plane | Jenkins (CI/CD), Ansible, Ansible-vault, Git, yamllint |
| **`vm-mon1`** | Monitoring Stack | Prometheus, Grafana, Alertmanager, Node Exporter, Process Exporter, cAdvisor, Loki, Promtail |

**Схема работы CI/CD:**

1. Инженер вносит изменения в конфигурации на локальном компьютере и выполняет `git push`.
2. GitHub отправляет Webhook в Jenkins.
3. Jenkins выгружает актуальный код из репозитория.
4. Ansible Playbook доставляет файлы на целевые серверы и перезапускает Docker-сервисы.
5. Изменения применяются автоматически, без ручного вмешательства.


<img width="778" height="1026" alt="арх схема" src="https://github.com/user-attachments/assets/1917b1e3-23c3-4ed6-8520-7daa89fac99b" />






---

##  Стек технологий

| Компонент | Назначение |
|-----------|------------|
| **Git / GitHub** | Версионирование конфигураций, хранение кода, триггер сборок |
| **Jenkins** | Пайплайн деплоя (`Jenkinsfile`), обработка Webhook'ов |
| **Ansible** | Управление конфигурацией (копирование файлов, запуск `docker-compose`) |
| **Docker Compose** | Контейнеризация и оркестрация сервисов мониторинга |
| **Prometheus** | Сбор и хранение метрик |
| **Alertmanager** | Обработка и маршрутизация алертов, отправка уведомлений в Slack |
| **Loki** | Инструмент агрегации и хранения логов|
| **Promtail** | Агент для сбора логов, их парсинга и отправки в Loki |
| **Grafana** | Визуализация данных (дашборды) |
| **Node Exporter** | Метрики операционной системы (CPU, RAM, диск, сеть) |
| **Process Exporter** | Детальные метрики по процессам |
| **cAdvisor** | Мониторинг контейнеров |
| **Ansible Vault** | Шифрование и безопасное хранение секретов (токены, пароли) прямо в Git |
| **yamllint** | Автоматическая проверка синтаксиса YAML-конфигураций в пайплайне Jenkins |


---

##  Текущий функционал

- [x] Развертывание полного стека мониторинга одной командой (`docker-compose up -d`)
- [x] Автоматический деплой изменений через CI-CD (Jenkins + Ansible)
- [x] Мониторинг серверов
- [x] Сбор метрик с помощью экспортеров  Prometheus
- [x] Визуализация данных в Grafana
- [x] Настроена отправка нотификаций из Prometheus в Alertmanager
- [x] Интеграция Alertmanager со Slack через Incoming Webhook (channel #alertmanager)
- [x] Интеграция Grafana с Alertmanager для отправки нотификаций на основе логов, метрик, трейсов (UI Grafana)
- [x] Централизованный сбор логов контейнеров с помощью Promtail
- [x] Хранение логов в Loki
- [x] Интеграция Loki с Alertmanager для отправки нотификаций на основе логов (loki-alerts.yml)
- [x] Безопасное хранение и деплой секретов с помощью Ansible Vault
- [x] Уведомления о статусе сборок Jenkins в Slack (channel #builds)
- [x] Автоматическая проверка YAML-файлов (`yamllint`) перед деплоем
- [x] Интеграция дашбордов и источников данных Grafana в пайплайн CI/CD: версионирование и автоматический деплой через Provisioning

---

##  Оптимизация проекта

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

### 3. Авторезолв алертов Grafana и переход на Loki Ruler

**Проблема:** Алерты, созданные в интерфейсе Grafana на основе логов Loki, автоматически разрешались (auto-resolved) через 5 минут после срабатывания, даже если проблема сохранялась. 

**Решение:**
- Алерты по логам были перенесены из UI Grafana Alerting во встроенный компонент **Ruler** в Loki.
- Создан файл `loki-alerts.yml`, содержащий правила алертов в формате Prometheus.
- Loki самостоятельно оценивает правила и напрямую отправляет нотификации в Alertmanager, минуя Grafana.
Таким образом, алерты по логам обрабатываются нативно в Loki и не зависят от особенностей Grafana Alerting, что исключило ложные авторезолвы и повысило надёжность системы оповещения.

### 4. Безопасное управление секретами

**Проблема:** Секреты (Slack Webhook URL, пароли) хранились в открытых файлах на сервере, что делало проект уязвимым. При смене инфраструктуры приходилось вручную переносить секретные данные на новое окружение.

**Решение:**
- **Внедрен Ansible Vault:** Секреты зашифрованы в `vault.yml` и хранятся прямо в Git-репозитории. Ansible автоматически расшифровывает при деплое.

### 5. Автоматическая проверка YAML-конфигураций

**Проблема:** Ошибки в синтаксисе YAML-файлов не отлавливались до деплоя. Ansible проверяет только свои плейбуки, а конфигурации сервисов копируются, как статические файлы без валидации. В результате битый конфиг мог попасть на сервер и сломать сервис.

**Решение:**
- В пайплайн Jenkins добавлена отдельная стадия `Validate YAML configs`, которая выполняется перед запуском Ansible.
- Для проверки используется утилита `yamllint` с настроенным файлом `.yamllint`, который отключает некритичные предупреждения (длина строк) и оставляет только важные ошибки.
- Если хотя бы в одном YAML-файле найдена синтаксическая ошибка, сборка немедленно останавливается, и деплой не выполняется.


## 📈 Планы по развитию

- [x] **Grafana Provisioning**  
  Автоматическая загрузка дашбордов и источников данных из Git, чтобы настройки Grafana не терялись при пересоздании контейнера.
- [ ] **Динамический инвентарь Ansible**  
  Переход от статического списка хостов к динамическому (скрипт или плагин), чтобы автоматически подхватывать новые серверы.
- [ ] **Healthcheck для контейнеров**  
  Добавление `healthcheck` в `docker-compose.yml` для автоматического перезапуска зависших сервисов.
