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
