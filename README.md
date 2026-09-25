# Task Tracker

Микросервисное приложение для управления задачами с JWT-аутентификацией, автоматической генерацией ежедневных отчетов и отправкой их по электронной почте.

## Функциональность

* Регистрация и аутентификация пользователей.
* JWT-аутентификация.
* Создание, просмотр, изменение и удаление задач.
* Управление статусами задач:

  * `TODO`
  * `IN_PROGRESS`
  * `COMPLETED`
* Автоматическая генерация ежедневных отчетов.
* Отправка отчетов пользователям по электронной почте.
* Асинхронное взаимодействие между микросервисами через Kafka.
* Transactional Outbox Pattern для надежной публикации событий.

## Архитектура

Приложение состоит из следующих сервисов:

* **Backend Service** — REST API, пользователи, задачи и аутентификация.
* **Scheduler Service** — запуск периодической генерации ежедневных отчетов.
* **Summarization Service** — генерация отчетов с использованием GigaChat.
* **Email Sender Service** — отправка электронной почты через SMTP по протоколу Brevo.
* **PostgreSQL** — хранение данных.
* **Apache Kafka** — обмен сообщениями между сервисами.

## Используемые технологии

* Java 21
* Spring Boot
* Spring Security
* Spring Data JPA
* JWT
* PostgreSQL
* Liquibase
* Apache Kafka
* GigaChat API
* JavaMailSender / SMTP
* Docker
* Docker Compose
* GitHub Actions
* Docker Hub
* JUnit
* MockMvc
* Testcontainers

## Тестирование

Backend Service содержит интеграционные тесты с использованием:

* JUnit
* Spring Boot Test
* MockMvc
* Testcontainers

Для тестов используются контейнеры PostgreSQL и Kafka, что позволяет запускать тесты в окружении, близком к реальному.

Тесты автоматически запускаются через GitHub Actions.

## CI/CD

При изменении исходного кода сервиса и push в его GitHub-репозиторий автоматически запускается GitHub Actions.

Для Backend Service сначала выполняются тесты. При успешном прохождении тестов создается новый Docker image и публикуется в Docker Hub.

Для остальных сервисов также автоматически собираются и публикуются новые Docker images.

Таким образом, Docker Hub содержит актуальные версии сервисов.

## Деплой

Для запуска приложения на сервере используется Docker Compose.

Получить актуальные версии Docker images:

```bash
docker compose pull
```

Создайте 5 файлов с переменными окружения рядом с `docker-compose.yml`.

1. Файл `.env`:

```bash
POSTGRES_URL=URL_FOR_POSTGRES_DB
POSTGRES_USERNAME=USERNAME_FOR_POSTGRES_DB
POSTGRES_PASSWORD=PASSWORD_FOR_POSTGRES_DB
```

2. Файл `.env.backend`:

```bash
POSTGRES_URL=URL_FOR_POSTGRES_DB_IN_DOCKER_NETWORK
POSTGRES_USERNAME=USERNAME_FOR_POSTGRES_DB_IN_DOCKER_NETWORK
POSTGRES_PASSWORD=PASSWORD_FOR_POSTGRES_DB_IN_DOCKER_NETWORK

KAFKA_SERVERS=SERVER_KAFKA_IN_DOCKER_NETWORK
KAFKA_SENDING_EMAIL_TOPIC=TOPIC_FOR_EMAIL_SENDER_SERVICE

JWT_SECRET=YOUR_JWT_SECRET
INTERNAL_SECRET=YOUR_INTERNAL_SECRET_FOR_INTERNAL_SERVICE
```

3. Файл `.env.scheduler`:

```bash
KAFKA_SERVERS=SERVER_KAFKA_IN_DOCKER_NETWORK
KAFKA_SENDING_EMAIL_TOPIC=TOPIC_FOR_EMAIL_SENDER_SERVICE

KAFKA_SUMMARIZATION_REQUEST_TOPIC=TOPIC_FOR_SUMMARIZATION_REQUEST
KAFKA_SUMMARIZATION_RESPONSE_TOPIC=TOPIC_FOR_SUMMARIZATION_RESPONSE

BACKEND_URL=URL_FOR_BACKEND_IN_DOCKER_NETWORK
BACKEND_SECRET=YOUR_INTERNAL_SECRET_FOR_INTERNAL_SERVICE
```

4. Файл `.env.summarization`:

```bash
KAFKA_SERVERS=SERVER_KAFKA_IN_DOCKER_NETWORK

KAFKA_SUMMARIZATION_REQUEST_TOPIC=TOPIC_FOR_SUMMARIZATION_REQUEST
KAFKA_SUMMARIZATION_RESPONSE_TOPIC=TOPIC_FOR_SUMMARIZATION_RESPONSE

GIGACHAT_AUTH_URL=URL_FOR_AUTH_IN_GIGACHAT
GIGACHAT_CHAT_URL=URL_FOR_CHAT_IN_GIGACHAT
GIGACHAT_AUTH_KEY=YOUR_SECRET_KEY_FOR_AUTH_IN_GIGACHAT
```

5. Файл `.env.email-sender`:

```bash
KAFKA_SERVERS=SERVER_KAFKA_IN_DOCKER_NETWORK
KAFKA_SENDING_EMAIL_TOPIC=TOPIC_FOR_EMAIL_SENDER_SERVICE

MAIL_HOST=MAIL_SERVICE_HOST_NAME
MAIL_PORT=MAIL_SERVICE_PORT
MAIL_USERNAME=MAIL_SERVICE_USERNAME
MAIL_PASSWORD=MAIL_SERVICE_PASSWORD
MAIL_FROM_EMAIL=YOUR_MAIL_FROM_EMAIL
```

Запустить приложение в папке с `docker-compose.yml`:

```bash
docker compose up -d
```

Проверить состояние сервисов:

```bash
docker compose ps
```

Посмотреть логи:

```bash
docker compose logs -f
```

Для обновления приложения после выхода новой версии:

```bash
docker compose pull
docker compose up -d
```

Конфиденциальные данные передаются через переменные окружения и не хранятся в Git-репозитории.
