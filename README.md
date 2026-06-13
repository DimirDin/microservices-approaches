# Домашнее задание: Микросервисы — подходы


---

## Содержание

- [Задача 1: Обеспечить разработку (CI/CD)](#задача-1-обеспечить-разработку-cicd)
- [Задача 2: Логи](#задача-2-логи)
- [Задача 3: Мониторинг](#задача-3-мониторинг)

---

## Задача 1: Обеспечить разработку (CI/CD)

### Требования

- Облачная система
- Система контроля версий Git
- Репозиторий на каждый сервис
- Запуск сборки по событию из системы контроля версий
- Запуск сборки по кнопке с указанием параметров
- Возможность привязать настройки к каждой сборке
- Возможность создания шаблонов для различных конфигураций сборок
- Возможность безопасного хранения секретных данных (пароли, ключи доступа)
- Несколько конфигураций для сборки из одного репозитория
- Кастомные шаги при сборке
- Собственные Docker-образы для сборки проектов
- Возможность развернуть агентов сборки на собственных серверах
- Возможность параллельного запуска нескольких сборок
- Возможность параллельного запуска тестов

### Сравнительная таблица решений

| Параметр | **GitLab CI/CD** | **GitHub Actions** | **Jenkins** | **Azure DevOps** | **Bitbucket Pipelines** |
|----------|------------------|-------------------|-------------|------------------|------------------------|
| **Облачная система** | ✅ GitLab SaaS / Self-hosted | ✅ GitHub SaaS / Self-hosted runners | ⚠️ Self-hosted / CloudBees | ✅ Azure DevOps Services | ✅ Bitbucket Cloud |
| **Git-репозиторий** | ✅ Встроен (1 репо = 1 сервис) | ✅ Встроен | ❌ Внешний (требует интеграции) | ✅ Встроен (Azure Repos) | ✅ Встроен |
| **Сборка по событию Git** | ✅ Push, MR, Tag, Schedule, Webhook | ✅ Push, PR, Release, Schedule, Webhook | ✅ Требует webhook/plugin | ✅ Push, PR, CI trigger | ✅ Push, PR, Tag |
| **Сборка по кнопке (manual)** | ✅ Pipeline с `when: manual` + параметры | ✅ `workflow_dispatch` + inputs | ✅ Parameterized builds | ✅ Manual queue + variables | ✅ Manual pipelines + variables |
| **Настройки на сборку** | ✅ CI/CD Variables (project/group/instance) | ✅ Repository secrets + env vars | ✅ Credentials plugin | ✅ Pipeline variables | ✅ Repository variables |
| **Шаблоны сборок** | ✅ `.gitlab-ci.yml` templates, `include` | ✅ Reusable workflows, composite actions | ✅ Shared Libraries (Groovy) | ✅ Task groups, templates | ✅ Pipelines as templates |
| **Безопасное хранение секретов** | ✅ Masked variables, Vault integration, CI/CD Secrets | ✅ Encrypted secrets, OIDC, Vault | ✅ Credentials plugin, HashiCorp Vault | ✅ Variable groups, Key Vault | ✅ Repository variables (encrypted) |
| **Несколько конфигураций из одного репо** | ✅ Multi-pipeline, `rules`, `only/except` | ✅ Matrix builds, multiple workflow files | ✅ Multi-branch pipelines | ✅ Multiple build definitions | ✅ Branch-based pipelines |
| **Кастомные шаги** | ✅ `script` blocks, custom images, `before_script` | ✅ `run` steps, custom actions | ✅ Groovy DSL, shell scripts | ✅ Custom tasks, PowerShell/Bash | ✅ Custom script steps |
| **Собственные Docker-образы** | ✅ `image:` в `.gitlab-ci.yml`, registry built-in | ✅ `container:` jobs, GitHub Packages | ✅ Docker plugin, custom agents | ✅ Container jobs, ACR | ✅ Custom images, Docker support |
| **Собственные агенты (runners)** | ✅ GitLab Runner (Shell, Docker, Kubernetes, VM) | ✅ Self-hosted runners (VM, K8s, container) | ✅ Agents (Master + Slaves) | ✅ Self-hosted agents (VM, container) | ✅ Self-hosted runners (limited) |
| **Параллельные сборки** | ✅ `parallel` keyword, matrix builds, autoscaling runners | ✅ `strategy: matrix`, concurrency groups | ✅ Parallel stages, multiple executors | ✅ Parallel jobs, agent pools | ✅ Parallel steps (limited) |
| **Параллельные тесты** | ✅ `parallel` + `pytest-xdist`, test splitting | ✅ Matrix strategy, sharding | ✅ Parallel test execution | ✅ Parallel test agents | ⚠️ Limited |
| **Кривая обучения** | Низкая (YAML) | Низкая (YAML) | Высокая (Groovy) | Средняя (YAML/GUI) | Низкая (YAML) |
| **Vendor lock-in** | Средний (GitLab ecosystem) | Высокий (GitHub ecosystem) | Низкий (open source) | Высокий (Azure ecosystem) | Высокий (Atlassian) |
| **Open source** | ✅ CE (Community Edition) | ❌ SaaS only | ✅ Полностью | ❌ SaaS only | ❌ SaaS only |
| **Enterprise-функции** | Ultimate (SAST/DAST/compliance) | Enterprise (advanced security) | CloudBees | Azure DevOps Server | Premium |

### Выбор: **GitLab CI/CD**

**Обоснование:**

1. **Полное покрытие всех 14 требований:** GitLab CI/CD из коробки поддерживает каждый пункт — от облачного развёртывания до параллельных тестов. Ни один другой продукт не покрывает спектр так полно без дополнительных интеграций.

2. **All-in-one платформа:** GitLab объединяет Git-репозиторий, CI/CD, Container Registry, Security Scanning, Monitoring и Project Management в одном инструменте. Для микросервисной архитектуры, где каждый сервис — отдельный репозиторий, это критично: не нужно настраивать 5–7 интеграций между разными системами.

3. **GitLab Runner — максимальная гибкость агентов:**
   - Docker executor — изолированные сборки в контейнерах
   - Kubernetes executor — autoscaling от 0 до 100+ подов по требованию
   - Shell executor — быстрые сборки на bare metal
   - VM executor — полная изоляция для security-критичных задач
   - Возможность развернуть runners на собственных серверах (on-prem, private cloud) при сохранении управления через GitLab SaaS.

4. **Шаблоны и переиспользование:** Система `include` позволяет создать центральный репозиторий с `.gitlab-ci.yml` шаблонами и подключать их в каждый микросервис. Изменение шаблона = автоматическое обновление CI/CD во всех 50+ сервисах.

5. **Безопасность секретов:** Masked variables скрывают значения в логах, Vault integration обеспечивает динамическое получение секретов, а CI/CD Secrets (Ultimate) — полный lifecycle management.

6. **Параллелизм:** `parallel` keyword позволяет разбивать тестовый набор на N частей, запускаемых одновременно на разных runners. Для микросервисов с тысячами тестов это сокращает время CI в 5–10 раз.

7. **DevSecOps из коробки:** SAST, DAST, dependency scanning, container scanning — встроены в pipeline без дополнительных лицензий (Ultimate tier). Для крупной компании с требованиями compliance это экономит $50K–200K/год на отдельных security-инструментах.

8. **Масштабируемость:** GitLab.com обрабатывает CI/CD для Linux kernel, GNOME, KDE — доказанная способность масштабироваться до 1000+ разработчиков и 100K+ репозиториев.

**Архитектура решения:**

┌─────────────────────────────────────────────────────────────┐
│                    GitLab SaaS (gitlab.com)                  │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐  │
│  │  Service A  │  │  Service B  │  │  Service C ...      │  │
│  │  Repo + CI  │  │  Repo + CI  │  │  Repo + CI          │  │
│  └──────┬──────┘  └──────┬──────┘  └──────────┬──────────┘  │
│         │                │                      │             │
│  ┌──────┴────────────────┴────────────────────┴──────────┐  │
│  │           Central Templates Repo (.gitlab-ci.yml)       │  │
│  │           include: '/templates/microservice-ci.yml'     │  │
│  └────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
│
▼
┌─────────────────────────────────────────────────────────────┐
│              GitLab Runners (Kubernetes Cluster)             │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────────────┐  │
│  │ Runner 1│  │ Runner 2│  │ Runner 3│  │ Runner N (auto) │  │
│  │ (Build) │  │ (Test)  │  │ (Deploy)│  │ (Scale 0→100)  │  │
│  └─────────┘  └─────────┘  └─────────┘  └─────────────────┘  │
│  ┌────────────────────────────────────────────────────────┐  │
│  │  Docker Images: custom build-env, custom test-env       │  │
│  │  Secrets: HashiCorp Vault / GitLab CI/CD Variables      │  │
│  └────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘

**Альтернатива:** Если компания уже использует GitHub и миграция невозможна, **GitHub Actions** — хороший выбор, но потребует отдельных инструментов для security scanning (Snyk, SonarQube) и container registry (GitHub Packages), что увеличивает сложность интеграций.

---

## Задача 2: Логи

### Требования

- Сбор логов в центральное хранилище со всех хостов, обслуживающих систему
- Минимальные требования к приложениям, сбор логов из stdout
- Гарантированная доставка логов до центрального хранилища
- Обеспечение поиска и фильтрации по записям логов
- Обеспечение пользовательского интерфейса с возможностью предоставления доступа разработчикам для поиска по записям логов
- Возможность дать ссылку на сохранённый поиск по записям логов

### Сравнительная таблица решений

| Параметр | **ELK Stack** | **Grafana Loki** | **Datadog Logs** | **Splunk** | **OpenObserve** |
|----------|---------------|-------------------|------------------|------------|-----------------|
| **Сбор со всех хостов** | ✅ Beats/Fluentd/Logstash agents | ✅ Promtail / Fluent Bit / Vector | ✅ Datadog Agent | ✅ Universal Forwarder | ✅ OpenObserve Agent |
| **Сбор из stdout** | ✅ Filebeat reads container logs | ✅ Promtail reads stdout via Kubernetes API | ✅ Agent captures stdout | ✅ Forwarder reads stdout | ✅ Agent reads stdout |
| **Гарантированная доставка** | ✅ At-least-once (persistent queues) | ✅ WAL + retries | ✅ At-least-once | ✅ At-least-once | ✅ At-least-once |
| **Полнотекстовый поиск** | ✅ Elasticsearch (Lucene) — лучший в классе | ✅ LogQL (label-based + regex) | ✅ Full-text + facets | ✅ SPL — enterprise-grade | ✅ Full-text + SQL-like |
| **Фильтрация** | ✅ KQL, Lucene query syntax | ✅ LogQL (labels + regex) | ✅ Faceted search, patterns | ✅ SPL, pivot tables | ✅ SQL-like queries |
| **Web UI** | ✅ Kibana (rich dashboards) | ✅ Grafana (explore + dashboards) | ✅ Datadog UI | ✅ Splunk Web | ✅ OpenObserve UI |
| **RBAC / доступ разработчикам** | ✅ Spaces, roles, document-level security | ✅ Grafana orgs, teams, folders | ✅ Role-based access, teams | ✅ RBAC, apps, roles | ✅ Organizations, roles |
| **Ссылка на сохранённый поиск** | ✅ Saved searches, shareable URLs | ✅ Explore → Share link | ✅ Saved views, shareable | ✅ Saved searches, dashboards | ✅ Saved queries, shareable |
| **Масштабируемость** | ✅ Horizontal (shards, replicas) | ✅ Object storage (S3/GCS) backend | ✅ SaaS (auto-scaling) | ✅ Indexers + Search Heads | ✅ Single binary / cluster |
| **Стоимость** | $$ (инфраструктура + обслуживание) | $ (легковесный, S3 storage) | $$$$ (per GB ingested) | $$$$$ (per GB/day) | $ (self-hosted free / cloud $0.3/GB) |
| **Resource footprint** | Высокий (Elasticsearch: 16–64GB RAM) | Низкий (Loki: 1–4GB RAM) | N/A (SaaS) | Высокий | Низкий |
| **Kubernetes-native** | ⚠️ Требует настройки | ✅ Designed for K8s | ✅ Operator available | ⚠️ Требует настройки | ✅ Helm chart |
| **Open source** | ✅ Apache 2.0 | ✅ AGPL v3 | ❌ | ❌ | ✅ Apache 2.0 |

### Выбор: **Grafana Loki + Vector**

**Обоснование:**

1. **Полное покрытие требований:**
   - ✅ **Сбор со всех хостов:** Vector (или Fluent Bit) собирает логи с каждого хоста/пода и отправляет в Loki.
   - ✅ **stdout:** В Kubernetes контейнеры пишут в stdout → Vector читает через `/var/log/pods` или Kubernetes API. Приложения не требуют модификации.
   - ✅ **Гарантированная доставка:** Vector поддерживает persistent buffer (WAL) и retry-политики. При недоступности Loki логи накапливаются локально и отправляются при восстановлении.
   - ✅ **Поиск и фильтрация:** LogQL позволяет искать по labels (pod, service, namespace) и фильтровать по содержимому через regex. Для микросервисов label-based подход естественен — каждый сервис = отдельный label.
   - ✅ **Web UI:** Grafana Explore — единый интерфейс для логов, метрик и трейсов. Разработчики получают привычный инструмент.
   - ✅ **RBAC:** Grafana organizations и teams позволяют создать отдельные dashboard-ы и datasource-ы для каждой команды.
   - ✅ **Ссылка на поиск:** Grafana поддерживает shareable links на любой explore-запрос — можно вставить в Slack/Jira и коллега увидит те же данные.

2. **Оптимизация ресурсов:** Loki хранит только индексы labels (не полный текст), а сами логи — в object storage (S3, GCS, MinIO). Это снижает требования к RAM в 10–20 раз по сравнению с Elasticsearch. Для крупной компании с 100+ микросервисов это экономия $10K–50K/мес на инфраструктуре.

3. **Единый стек наблюдаемости:** Loki + Grafana + Prometheus + Tempo (трейсы) = полный observability stack от одного вендора. Разработчики работают в одном UI, а не переключаются между Kibana, Datadog и Jaeger.

4. **Kubernetes-native:** Loki разработан специально для K8s. Promtail (или Vector) автоматически обнаруживает новые поды и начинает сбор логов без ручной конфигурации.

5. **Гибкость хранения:** Retention policies, compaction, tiered storage (hot/warm/cold) — позволяют хранить недельные логи на SSD для быстрого поиска, а месячные — на S3 для экономии.

**Архитектура решения:**
┌─────────────────────────────────────────────────────────────┐
│                  Kubernetes Cluster (Services)               │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────────────┐  │
│  │ Service │  │ Service │  │ Service │  │ Service N       │  │
│  │ Pod A   │  │ Pod B   │  │ Pod C   │  │ Pod N           │  │
│  │ stdout  │  │ stdout  │  │ stdout  │  │ stdout          │  │
│  └────┬────┘  └────┬────┘  └────┬────┘  └────────┬────────┘  │
│       │            │            │                 │           │
│  ┌────┴────────────┴────────────┴─────────────────┴────────┐  │
│  │  Vector DaemonSet (1 агент на ноду)                  │  │
│  │  - Сбор stdout/stderr из /var/log/pods               │  │
│  │  - Enrichment: pod labels, node labels, K8s metadata │  │
│  │  - Buffering: WAL на локальном диске                 │  │
│  │  - Output: Loki (HTTP/gRPC)                          │  │
│  └────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
│
▼
┌─────────────────────────────────────────────────────────────┐
│                      Loki Stack                              │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐  │
│  │  Distributor│  │  Ingester   │  │  Querier            │  │
│  │  (приём)    │  │  (WAL +     │  │  (поиск по labels + │  │
│  │             │  │  index)     │  │  chunk storage)     │  │
│  └─────────────┘  └─────────────┘  └─────────────────────┘  │
│  ┌────────────────────────────────────────────────────────┐  │
│  │  Object Storage (S3 / MinIO / GCS) — долгосрочное        │  │
│  │  хранение chunk-ов логов                                 │  │
│  └────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
│
▼
┌─────────────────────────────────────────────────────────────┐
│                      Grafana                                 │
│  ┌────────────────────────────────────────────────────────┐  │
│  │  Explore → LogQL: {service="payment-service"} |= "error"│  │
│  │  Dashboards: логи + метрики + трейсы в одном месте      │  │
│  │  RBAC: команды, организации, view-only доступ           │  │
│  └────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘

**Альтернатива:** Если требуется мощный полнотекстовый поиск (например, для security-аудита с поиском по IP-адресам внутри логов) и бюджет позволяет, **ELK Stack** — классический выбор, но с существенно более высокими требованиями к ресурсам и обслуживанию.

---

## Задача 3: Мониторинг

### Требования

- Сбор метрик со всех хостов, обслуживающих систему
- Сбор метрик состояния ресурсов хостов: CPU, RAM, HDD, Network
- Сбор метрик потребляемых ресурсов для каждого сервиса: CPU, RAM, HDD, Network
- Сбор метрик, специфичных для каждого сервиса
- Пользовательский интерфейс с возможностью делать запросы и агрегировать информацию
- Пользовательский интерфейс с возможностью настраивать различные панели для отслеживания состояния системы

### Сравнительная таблица решений

| Параметр | **Prometheus + Grafana** | **Datadog** | **New Relic** | **Zabbix** | **InfluxDB + Grafana** |
|----------|--------------------------|-------------|---------------|------------|------------------------|
| **Сбор метрик со всех хостов** | ✅ Node Exporter (CPU, RAM, disk, net) | ✅ Datadog Agent | ✅ Infrastructure Agent | ✅ Zabbix Agent | ✅ Telegraf |
| **Метрики хоста (CPU/RAM/HDD/Net)** | ✅ Node Exporter — полный набор | ✅ Встроенно | ✅ Встроенно | ✅ Встроенно | ✅ Telegraf plugins |
| **Метрики сервиса (CPU/RAM/HDD/Net)** | ✅ cAdvisor / kubelet (K8s) | ✅ APM + container monitoring | ✅ APM + container | ⚠️ Требует настройки | ✅ cAdvisor + Telegraf |
| **Специфичные метрики сервиса** | ✅ Custom metrics (exposition format) | ✅ Custom metrics, APM | ✅ Custom events, NRQL | ✅ User parameters | ✅ Line protocol |
| **Язык запросов** | ✅ PromQL (мощный, функциональный) | ✅ Metrics Explorer | ✅ NRQL (SQL-like) | ✅ Zabbix API | ✅ Flux / InfluxQL |
| **Web UI + запросы** | ✅ Grafana (Explore + Dashboards) | ✅ Datadog UI | ✅ New Relic One | ✅ Zabbix Frontend | ✅ Grafana |
| **Настраиваемые панели** | ✅ Grafana Dashboards (JSON/templating) | ✅ Dashboards + Notebooks | ✅ Dashboards | ✅ Screens, maps | ✅ Grafana Dashboards |
| **Alerting** | ✅ Alertmanager (multi-channel) | ✅ Monitors + SLO tracking | ✅ Alerts + AI | ✅ Triggers | ✅ Grafana Alerting |
| **Масштабируемость** | ✅ Thanos / Cortex / Mimir (federated) | ✅ SaaS (auto) | ✅ SaaS (auto) | ⚠️ Вертикальная | ⚠️ Кластеризация сложна |
| **Kubernetes-native** | ✅ De-facto стандарт K8s | ✅ Operator | ✅ Operator | ⚠️ Требует настройки | ✅ Helm chart |
| **Open source** | ✅ Apache 2.0 | ❌ | ❌ | ✅ GPL v2 | ✅ MIT |
| **Стоимость** | $ (инфраструктура) | $$$$ (per host/metric) | $$$ (per host) | $ (self-hosted) | $$ (инфраструктура) |
| **Community / экосистема** | ✅ Огромная (CNCF graduated) | ✅ Большая | ✅ Большая | ✅ Средняя | ✅ Средняя |

### Выбор: **Prometheus + Grafana**

**Обоснование:**

1. **Полное покрытие требований:**
   - ✅ **Сбор со всех хостов:** Node Exporter собирает метрики CPU, RAM, диска, сети с каждого хоста. Prometheus server скрейпит (pull) метрики по HTTP.
   - ✅ **Метрики хоста:** Node Exporter предоставляет 100+ метрик: `node_cpu_seconds_total`, `node_memory_MemAvailable_bytes`, `node_filesystem_avail_bytes`, `node_network_receive_bytes_total`.
   - ✅ **Метрики сервиса:** cAdvisor (или kubelet в K8s) собирает container-level метрики: `container_cpu_usage_seconds_total`, `container_memory_working_set_bytes`. Для non-K8s — process exporter.
   - ✅ **Специфичные метрики:** Каждый сервис экспонирует custom metrics в Prometheus exposition format (`/metrics` endpoint). Например, `http_requests_total{service="payment",status="200"}`.
   - ✅ **Web UI + запросы:** Grafana Explore позволяет писать PromQL-запросы ad-hoc с автодополнением и подсказками.
   - ✅ **Настраиваемые панели:** Grafana Dashboards с templating variables — одна панель для всех микросервисов, переключение через dropdown.

2. **De-facto стандарт для микросервисов:** Prometheus — graduated проект CNCF, используется в Kubernetes, Docker, Envoy, Istio. Интеграция с любыми современными инструментами из коробки.

3. **Pull-модель — преимущество для микросервисов:** Prometheus сам опрашивает endpoints (`/metrics`), что упрощает discovery новых сервисов через Kubernetes Service Discovery. Не нужно настраивать push-агенты на каждом поде.

4. **Мощный язык запросов (PromQL):** Поддержка rate, histogram_quantile, aggregation operators, recording rules. Позволяет строить сложные запросы: "95-й перцентиль latency за последние 5 минут для сервиса payment".

5. **Alertmanager:** Многоуровневая система алертинга — grouping, inhibition, routing. Алерты можно направлять в Slack, PagerDuty, OpsGenie, webhook.

6. **Масштабируемость:** Thanos или Cortex позволяют федерацию нескольких Prometheus-серверов и долгосрочное хранение в object storage. Для крупной компании — horizontal scaling без vendor lock-in.

7. **Единый стек с логами:** Prometheus + Grafana + Loki + Tempo = полный observability. Разработчики работают в одном UI, а не переключаются между инструментами.

**Архитектура решения:**
┌─────────────────────────────────────────────────────────────┐
│                  Kubernetes Cluster                          │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────────────┐  │
│  │ Service │  │ Service │  │ Service │  │ Service N       │  │
│  │ Pod A   │  │ Pod B   │  │ Pod C   │  │ Pod N           │  │
│  │ /metrics│  │ /metrics│  │ /metrics│  │ /metrics        │  │
│  └────┬────┘  └────┬────┘  └────┬────┘  └────────┬────────┘  │
│       │            │            │                 │           │
│  ┌────┴────────────┴────────────┴─────────────────┴────────┐  │
│  │  Prometheus Server (scrape every 15s)                    │  │
│  │  - Service Discovery (K8s endpoints)                   │  │
│  │  - Relabeling: service labels, namespace, pod        │  │
│  │  - Retention: 15 days local, long-term → Thanos       │  │
│  └────────────────────────────────────────────────────────┘  │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐  │
│  │ Node Exporter│  │ cAdvisor    │  │ kube-state-metrics  │  │
│  │ (host CPU/   │  │ (container  │  │ (K8s objects:        │  │
│  │  RAM/disk)   │  │  CPU/RAM)   │  │  pods, deployments) │  │
│  └─────────────┘  └─────────────┘  └─────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
│
▼
┌─────────────────────────────────────────────────────────────┐
│                      Grafana                                 │
│  ┌────────────────────────────────────────────────────────┐  │
│  │  Dashboard "Microservices Overview":                    │  │
│  │  ┌────────────┐  ┌────────────┐  ┌────────────────────┐ │  │
│  │  │ CPU Usage  │  │ Memory     │  │ Request Rate       │ │  │
│  │  │ by Service │  │ by Service  │  │ by Service         │ │  │
│  │  │ (PromQL)   │  │ (PromQL)    │  │ (PromQL)           │ │  │
│  │  └────────────┘  └────────────┘  └────────────────────┘ │  │
│  │  Variables: service, namespace, $node                 │  │
│  │  Alerts: CPU > 80%, Memory > 90%, Error rate > 1%      │  │
│  └────────────────────────────────────────────────────────┘  │
│  ┌────────────────────────────────────────────────────────┐  │
│  │  Alertmanager → Slack / PagerDuty / Webhook             │  │
│  └────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘

**Пример PromQL-запросов для микросервисов:**

```promql
# CPU usage по сервису
rate(container_cpu_usage_seconds_total{pod=~"payment-.*"}[5m])

# Memory usage по сервису
container_memory_working_set_bytes{pod=~"payment-.*"}

# Request rate по сервису
rate(http_requests_total{service="payment"}[5m])

# Error rate (4xx + 5xx)
sum(rate(http_requests_total{service="payment",status=~"4..|5.."}[5m])) 
  / sum(rate(http_requests_total{service="payment"}[5m]))

# 95-й перцентиль latency
histogram_quantile(0.95, 
  sum(rate(http_request_duration_seconds_bucket{service="payment"}[5m])) by (le)
)
```

Альтернатива: Если компания предпочитает SaaS-решение и бюджет не критичен, Datadog предоставляет APM + инфраструктурный мониторинг + логи в одном UI, но стоимость быстро растёт с масштабом ($50–500/хост/мес). Для крупной компании с 500+ хостами это $25K–250K/мес.
Итоговая архитектура инфраструктуры
┌─────────────────────────────────────────────────────────────────────┐
│                    Инфраструктура микросервисов                      │
│                                                                      │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐  │
│  │   CI/CD         │  │   Логи          │  │   Мониторинг    │  │
│  │   GitLab CI/CD  │  │   Grafana Loki  │  │   Prometheus    │  │
│  │   + GitLab      │  │   + Vector      │  │   + Grafana     │  │
│  │   Runners (K8s) │  │   + Grafana     │  │   + Alertmanager│  │
│  └────────┬────────┘  └────────┬────────┘  └────────┬────────┘  │
│           │                    │                    │           │
│  ┌────────┴────────────────────┴────────────────────┴────────┐  │
│  │              Kubernetes Cluster (Services)                  │  │
│  │  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────────┐  │
│  │  │ Service │  │ Service │  │ Service │  │ Service N   │  │
│  │  │ A       │  │ B       │  │ C       │  │             │  │
│  │  │ stdout  │  │ stdout  │  │ stdout  │  │ stdout      │  │
│  │  │ /metrics│  │ /metrics│  │ /metrics│  │ /metrics    │  │
│  │  └─────────┘  └─────────┘  └─────────┘  └─────────────┘  │
│  └─────────────────────────────────────────────────────────────┘  │
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐  │
│  │  Единый вход: Grafana (логи + метрики + трейсы)             │  │
│  │  GitLab (код + CI/CD + registry + security)                 │  │
│  └─────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────┘
