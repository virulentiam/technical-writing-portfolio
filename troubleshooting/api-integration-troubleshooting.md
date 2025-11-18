# Troubleshooting Guide: решение проблем при интеграции API

<!-- Данный документ является учебным. Для демонстрации навыков технического письма в качестве примера используется выдуманное приложение для управления задачами. Все совпадения случайны. -->

## Содержание
* [Введение](#введение)
* [Быстрая диагностика](#быстрая-диагностика)
* [Проблемы аутентификации](#проблемы-аутентификации)
    * [Ошибка 401: "Invalid API key"](#ошибка-401-invalid-api-key)
    * [Ошибка 401: "Token expired"](#ошибка-401-token-expired)
    * [Ошибка 403: "Insufficient permissions"](#ошибка-403-insufficient-permissions)
* [Проблемы с запросами](#проблемы-с-запросами)
    * [Ошибка 400: "Invalid JSON"](#ошибка-400-invalid-json)
    * [Ошибка 400: "Missing required field"](#ошибка-400-missing-required-field)
    * [Ошибка 422: "Validation error"](#ошибка-422-validation-error)
* [Проблемы с производительностью](#проблемы-с-производительностью)
    * [Ошибка 429: "Rate limit exceeded"](#ошибка-429-rate-limit-exceeded)
    * [Медленные запросы](#медленные-запросы)
* [Проблемы с данными](#проблемы-с-данными)
    * [Ошибка 404: "Resource not found"](#ошибка-404-resource-not-found)
    * [Получаю неожиданные данные в ответе](#получаю-неожиданные-данные-в-ответе)
* [Проблемы с сетью](#проблемы-с-сетью)
    * [Ошибка подключения (Connection timeout)](#ошибка-подключения-connection-timeout)
    * [Ошибка SSL/TLS](#ошибка-ssl-tls)
* [Как получить помощь](#как-получить-помощь)
    * [Техническая поддержка](#техническая-поддержка)
    * [Форум разработчиков](#форум-разработчиков)
    * [GitHub Issues](#gitHub-issues)
* [Полезные инструменты для отладки](#полезные-инструменты-для-отладки)

---

## Введение

Это руководство поможет вам диагностировать и решить типичные проблемы и ошибки, возникающие при интеграции с TaskFlow API. Проблемы разделены на категории для упрощения поиска и быстрого устранения неполадок.

**Если вы не нашли решение своей проблемы:**
* Проверьте [документацию API](https://docs.taskflow.example.com/api)
* Обратитесь в [техническую поддержку](mailto:api-support@taskflow.example.com)
* Посетите [форум разработчиков](https://community.taskflow.example.com)

---

## Быстрая диагностика

Перед детальной диагностикой рекомендуется провести быструю проверку базовых элементов интеграции. Пройдите этот чек-лист:

- [ ] **Протокол и домен.** Используйте HTTPS (`https://`) и правильный базовый URL: `https://api.taskflow.example.com/v1/`.
- [ ] **API-ключ действителен.** Проверьте срок действия в [личном кабинете](https://app.taskflow.example.com/settings/api).
- [ ] **Формат запроса корректен.** Проверьте JSON-структуру на [jsonlint.com](https://jsonlint.com/).
- [ ] **Заголовки установлены.** При отправке JSON-тела должен быть установлен `Content-Type: application/json`.
- [ ] **Авторизация.** Убедитесь, что JWT-токен или API-ключ корректно включен в заголовок `Authorization: Bearer YOUR_TOKEN`.
- [ ] **Не превышен rate limit.** Проверьте, что вы не превысили 1000 запросов в час.

<p align="right"><a href="#содержание">⬆️ Наверх</a></p>

---

## Проблемы аутентификации

### Ошибка 401: "Invalid API key"

**Симптом:**
```json
{
  "status": "error",
  "message": "Invalid API key",
  "error_code": "INVALID_API_KEY"
}
```

**Причина:**  
API-ключ отсутствует в запросе, неправильно указан, содержит лишние символы (пробелы) или срок его действия истек.

**Решение:**

1. **Проверьте формат заголовка:**
   ```
   Authorization: Bearer YOUR_API_KEY_HERE
   ```
   Убедитесь, что между `Bearer` и ключом есть пробел.

2. **Проверьте актуальность ключа:**
   * Зайдите в [личный кабинет](https://app.taskflow.example.com/settings/api).
   * Проверьте статус ключа (должен быть «Активен»).
   * Проверьте срок действия.

3. **Создайте новый ключ (если необходимо):**
   * Перейдите в **Настройки → API → Создать новый ключ**.
   * Скопируйте ключ и сохраните в безопасном месте.
   * Замените старый ключ в вашем коде.

**Пример правильного запроса (cURL):**

```bash
curl -X GET https://api.taskflow.example.com/v1/projects \
  -H "Authorization: Bearer sk_live_abc123def456" \
  -H "Content-Type: application/json"
```

<p align="right"><a href="#содержание">⬆️ Наверх</a></p>

---

### Ошибка 401: "Token expired"

**Симптом:**
```json
{
  "status": "error",
  "message": "JWT token has expired",
  "error_code": "TOKEN_EXPIRED"
}
```

**Причина:**  
JWT-токен истек. Стандартный срок действия JWT-токена TaskFlow — 3600 секунд (1 час).

**Решение:**

1. **Обновите токен через refresh token:**
   ```bash
   curl -X POST https://api.taskflow.example.com/v1/auth/refresh \
     -H "Content-Type: application/json" \
     -d '{
       "refresh_token": "YOUR_REFRESH_TOKEN"
     }'
   ```

2. **Сохраните новый токен:**
   ```json
   {
     "access_token": "eyJhbGc...",
     "refresh_token": "dGhpc2lz...",
     "expires_in": 3600
   }
   ```

3. **Используйте новый `access_token` в запросах**

> **Совет:** реализуйте автоматическое обновление токена за 5-10 минут до истечения срока действия.

<p align="right"><a href="#содержание">⬆️ Наверх</a></p>

---

### Ошибка 403: "Insufficient permissions"

**Симптом:**
```json
{
  "status": "error",
  "message": "Insufficient permissions to access this resource",
  "error_code": "FORBIDDEN"
}
```

**Причина:**  
У вашего API-ключа или токена нет прав для выполнения этого действия.

**Решение:**

1. **Проверьте права API-ключа:**
   * Зайдите в **Настройки → API → Ваш ключ**.
   * Проверьте разделы «Права доступа» (Read, Write, Delete).

2. **Проверьте права пользователя в проекте:**
   * Endpoint `/projects/{id}/tasks` требует роль «Участник» или выше.
   * Endpoint `/projects/{id}/delete` требует роль «Администратор».

3. **Обновите права (если необходимо):**
   * Попросите администратора повысить ваши права в проекте.
   * Или создайте новый API-ключ с расширенными правами.

<p align="right"><a href="#содержание">⬆️ Наверх</a></p>

---

## Проблемы с запросами

### Ошибка 400: "Invalid JSON"

**Симптом:**
```json
{
  "status": "error",
  "message": "Request body contains invalid JSON",
  "error_code": "INVALID_JSON"
}
```

**Причина:**  
JSON в теле запроса содержит синтаксические ошибки.

**Решение:**

1. **Проверьте JSON на валидность:**
   * Скопируйте ваш JSON.
   * Вставьте на [jsonlint.com](https://jsonlint.com/).
   * Исправьте найденные ошибки.

2. **Типичные ошибки:**
   * Лишняя запятая в конце: `{"name": "Task",}`. 
   * Одинарные кавычки вместо двойных: `{'name': 'Task'}`. 
   * Отсутствующие кавычки: `{name: "Task"}`. 
   * Незакрытые скобки: `{"name": "Task"`. 

3. **Правильный формат:**
   ```json
   {
     "name": "New Task",
     "description": "Task description",
     "priority": "high"
   }
   ```

<p align="right"><a href="#содержание">⬆️ Наверх</a></p>

---

### Ошибка 400: "Missing required field"

**Симптом:**
```json
{
  "status": "error",
  "message": "Missing required field: name",
  "error_code": "MISSING_FIELD",
  "details": {
    "field": "name"
  }
}
```

**Причина:**  
В запросе отсутствует обязательное поле.

**Решение:**

1. **Проверьте документацию endpoint:**
   * Перейдите на [docs.taskflow.example.com/api](https://docs.taskflow.example.com/api).
   * Найдите ваш endpoint.
   * Проверьте список обязательных полей (отмечены как "required").

2. **Добавьте недостающее поле:**
   
   Было:
   ```json
   {
     "description": "Task description"
   }
   ```
   
   Должно быть:
   ```json
   {
     "name": "Task name",
     "description": "Task description"
   }
   ```

<p align="right"><a href="#содержание">⬆️ Наверх</a></p>

---

### Ошибка 422: "Validation error"

**Симптом:**
```json
{
  "status": "error",
  "message": "Validation failed",
  "error_code": "VALIDATION_ERROR",
  "details": {
    "priority": "Value must be one of: low, medium, high"
  }
}
```

**Причина:**  
Значение поля не соответствует ожидаемому формату или допустимым значениям.

**Решение:**

1. **Прочитайте поле `details` в ответе:**
   Оно содержит конкретное описание ошибки валидации.

2. **Исправьте значение:**
   
   Было:
   ```json
   {
     "name": "Task",
     "priority": "very_high"
   }
   ```
   
   Должно быть:
   ```json
   {
     "name": "Task",
     "priority": "high"
   }
   ```

3. **Проверьте типы данных:**
   * `"priority"` должен быть строкой (`"high"`), а не числом (`1`).
   * `"due_date"` должен быть в формате ISO 8601: `"2025-11-20T15:00:00Z"`.
   * `"assignee_id"` должен быть числом, а не строкой.

<p align="right"><a href="#содержание">⬆️ Наверх</a></p>

---

## Проблемы с производительностью

### Ошибка 429: "Rate limit exceeded"

**Симптом:**
```json
{
  "status": "error",
  "message": "Rate limit exceeded. Try again in 3600 seconds",
  "error_code": "RATE_LIMIT_EXCEEDED",
  "retry_after": 3600
}
```

**Причина:**  
Превышен лимит запросов (1000 запросов в час на API-ключ).

**Решение:**

1. **Дождитесь сброса лимита:**
   * Поле `retry_after` показывает, через сколько секунд можно повторить запрос.
   * В данном случае: подождите 1 час.

2. **Оптимизируйте запросы:**
   * Используйте batch endpoints (если доступны) для отправки нескольких операций за раз.
   * Кэшируйте часто запрашиваемые данные.
   * Уменьшите частоту polling-запросов.

3. **Реализуйте exponential backoff:**
   ```python
   import time
   import requests
   
   def make_request_with_retry(url, headers, max_retries=3):
       for attempt in range(max_retries):
           response = requests.get(url, headers=headers)
           
           if response.status_code == 429:
               retry_after = int(response.headers.get('Retry-After', 60))
               print(f"Rate limited. Waiting {retry_after} seconds...")
               time.sleep(retry_after)
               continue
           
           return response
       
       raise Exception("Max retries exceeded")
   ```

4. **Запросите повышение лимита:**
   * Свяжитесь с поддержкой: api-support@taskflow.example.com.
   * Опишите ваш use case и текущее потребление.

<p align="right"><a href="#содержание">⬆️ Наверх</a></p>

---

### Медленные запросы

**Симптом:**  
Запросы выполняются дольше 5 секунд.

**Возможные причины и решения:**

1. **Запрашиваете слишком много данных:**
   
   **Решение:** Используйте пагинацию.
   
   ```
   GET /api/v1/projects/123/tasks?page=1&per_page=50
   ```
   
   Рекомендуем запрашивать не более 100 записей за раз.

3. **Не используете фильтры:**
   
   **Решение:** Фильтруйте данные на стороне API.

   ```
   GET /api/v1/tasks?status=open&priority=high&assignee_id=456
   ```
   
5. **Делаете много последовательных запросов:**
   
   **Решение:** Используйте batch endpoints (если доступны).
   
   Вместо:
   
   ```
   GET /api/v1/tasks/1
   GET /api/v1/tasks/2
   GET /api/v1/tasks/3
   ```
   
   Используйте:
   ```
   POST /api/v1/tasks/batch
   {
     "ids": [1, 2, 3]
   }
   ```

<p align="right"><a href="#содержание">⬆️ Наверх</a></p>

---

## Проблемы с данными

### Ошибка 404: "Resource not found"

**Симптом:**
```json
{
  "status": "error",
  "message": "Task not found",
  "error_code": "NOT_FOUND"
}
```

**Причина:**  
Запрашиваемый ресурс не существует или у вас нет доступа к нему.

**Решение:**

1. **Проверьте ID ресурса:**
   * Убедитесь, что ID корректен.
   * Проверьте, что ресурс не был удален.

2. **Проверьте права доступа:**
   * Возможно, у вас нет прав для просмотра этого ресурса.
   * Проверьте, что вы являетесь участником проекта.

3. **Проверьте формат URL:**
   
   Неправильно:
   
   ```
   GET /api/v1/tasks/abc (строка вместо числа)
   GET /api/v1/task/123 (task вместо tasks)
   ```
   
   Правильно:
   
   ```
   GET /api/v1/tasks/123
   ```

<p align="right"><a href="#содержание">⬆️ Наверх</a></p>

---

### Получаю неожиданные данные в ответе

**Симптом:**  
Поля в ответе API отличаются от документации.

**Возможные причины:**

1. **Используете старую версию API:**
   
   **Решение:** Проверьте версию в URL.
   
   ```
   https://api.taskflow.example.com/v1/... ← актуальная версия
   ```

3. **Поля были переименованы:**
   
   **Решение:** Проверьте [changelog](https://docs.taskflow.example.com/changelog) и [список deprecated полей](https://docs.taskflow.example.com/api/deprecations).

4. **Разные форматы для разных endpoints:**
   
   **Решение:** внимательно прочитайте документацию для каждого endpoint.

<p align="right"><a href="#содержание">⬆️ Наверх</a></p>

---

## Проблемы с сетью

### Ошибка подключения (Connection timeout)

**Симптом:**  
Запрос не выполняется, выдает ошибку "Connection timeout" или "Unable to connect".

**Возможные причины и решения:**

1. **Проблемы с интернет-соединением:**
   * Проверьте подключение к интернету.
   * Проверьте ping: `ping api.taskflow.example.com`.

2. **Блокировка файрволом:**
   * Убедитесь, что ваш firewall не блокирует исходящие HTTPS-запросы.
   * Проверьте, что порт 443 открыт.

3. **Проблемы с DNS:**
   * Проверьте разрешение DNS: `nslookup api.taskflow.example.com`.
   * Попробуйте использовать другой DNS-сервер (например, 8.8.8.8).

4. **API временно недоступен:**
   * Проверьте [статус сервиса](https://status.taskflow.example.com).
   * Проверьте [X @TaskFlowStatus](https://x.com/TaskFlowStatus) для уведомлений.

<p align="right"><a href="#содержание">⬆️ Наверх</a></p>

---

### Ошибка SSL/TLS

**Симптом:**  
`SSL certificate verification failed` или `SSL: CERTIFICATE_VERIFY_FAILED`

**Причина:**  
Проблемы с проверкой SSL-сертификата.

**Решение:**

1. **Обновите корневые сертификаты:**
   
   **Python:**
   
   ```bash
   pip install --upgrade certifi
   ```
   
   **Node.js:**
   
   ```bash
   npm install -g node
   ```

3. **Проверьте системное время:**
   Убедитесь, что время на вашем компьютере установлено правильно.

4. **В крайнем случае (не рекомендуется для production):**

   ```python
   import requests
   response = requests.get(url, verify=False)  # Отключает проверку SSL
   ```
   
> ⚠️ **Внимание:** используйте только для локальной отладки!

<p align="right"><a href="#содержание">⬆️ Наверх</a></p>

---

## Как получить помощь

Если вы не нашли решение своей проблемы, свяжитесь с нами:

### Техническая поддержка

**Email:** api-support@taskflow.example.com

**Что включить в запрос:**
1. Описание проблемы
2. Полный текст ошибки (JSON-ответ)
3. Код запроса (уберите API-ключи!)
4. Используемый язык программирования и версию библиотеки
5. Время возникновения ошибки (с часовым поясом)

### Форум разработчиков

[community.taskflow.example.com](https://community.taskflow.example.com)

### GitHub Issues

[github.com/taskflow/api-docs/example/issues](https://github.com/taskflow/api-docs/example/issues)

---

## Полезные инструменты для отладки

* **Postman** — [postman.com](https://postman.com) — для тестирования API-запросов.
* **JSONLint** — [jsonlint.com](https://jsonlint.com) — для проверки валидности JSON.
* **cURL** — встроенный инструмент командной строки для HTTP-запросов.
* **Fiddler / Charles Proxy** — для перехвата и анализа HTTP-трафика.

---

<p align="right"><a href="#содержание">⬆️ Наверх</a></p>

*Последнее обновление: 18 ноября 2025. Данный документ является учебным.*
