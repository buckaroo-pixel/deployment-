Сервис мониторинга
Описание функционала проекта
- Авторизация: пользователи хранятся в базе данных.
- Admin API: добавление сервисов для мониторинга (передается health endpoint, возвращающий JSON с полем {"status":"UP|DOWN"}).
- Cron-запуск: сбор метрик доступности каждый интервал (по умолчанию — каждую минуту).
- API (JSON): получение информации о периодах недоступности за указанный интервал и расчет процента доступности.
- Обработка различных причин недоступности: включая timeout, DNS, SSL, ошибки HTTP (4xx/5xx), неверный JSON и прочее.
Стек технологий
- Java 17
- Spring Boot 3
- Maven
- PostgreSQL
- Flyway миграции
- Swagger UI (OpenAPI)
Быстрый старт (без Docker)
1) Подготовка базы данных PostgreSQL
Создайте новую БД (например):
```sql
CREATE DATABASE service_monitor;
```

По умолчанию приложение ожидает:
- URL: jdbc:postgresql://localhost:5432/service_monitor
- Пользователь: postgres
- Пароль: postgres

Если необходимо, измените эти настройки в файле src/main/resources/application.yml.
2) Запуск приложения
В корне проекта выполните команду:

```bash
mvn clean spring-boot:run
```

После запуска:
- Swagger UI будет доступен по адресу: http://localhost:8080/swagger-ui/index.html
3) Тестовые пользователи
Пользователи создаются через миграции Flyway:

- ADMIN  
  - Логин: admin  
  - Пароль: admin123

- USER  
  - Логин: user  
  - Пароль: user123
Примеры работы с API
Добавление сервиса на мониторинг (доступно только для ADMIN)
```bash
curl -u admin:admin123 -X POST "http://localhost:8080/api/admin/apps" ^
  -H "Content-Type: application/json" ^
  -d "{"name":"demo","healthUrl":"http://localhost:8081/actuator/health","active":true}"
```

Просмотр метрик доступности (доступно для USER и ADMIN)
```bash
curl -u user:user123 "http://localhost:8080/api/metrics/apps/1/availability?from=2025-01-01T00:00:00Z&to=2025-12-31T23:59:59Z"
```

Получение периодов недоступности
```bash
curl -u user:user123 "http://localhost:8080/api/metrics/apps/1/outages?from=2025-01-01T00:00:00Z&to=2025-12-31T23:59:59Z"
```

Примечания
- Мониторинг выполняется по cron (по умолчанию **каждую минуту**).
- Формат ответа от health endpoint ориентирован на Spring Actuator: используется верхний уровень поля status.
- Вся ошибка запроса или парсинга трактуется как **DOWN**, и добавляется в запись с указанием типа ошибки (errorType).
