# Задача 3.1. Описание REST API на фронте при регистрации

## Запрос на регистрацию

- **Метод:** `POST`
- **URL:** `api/v1/register`
- **Пример запроса:** `https://{домен}/api/v1/register`

## Входные параметры

- `firstName` — обязательный, `string`, макс. 50
- `lastName` — обязательный, `string`, макс. 50
- `userName` — обязательный, `string`, мин. 3, макс. 30, `[a-z, 0-9]`
- `password` — обязательный, `string`, мин. 8, макс. 128, хотя бы 1 буква в верхнем и нижнем регистре и 1 цифра

## Тело запроса

```json
{
  "firstName": "Petya",
  "lastName": "Petrov",
  "userName": "Petya007",
  "password": "Parol_paroley!@"
}
```

## Тело ответа

- **Код:** `200/201`

```json
{
  "userId": "b01195fa-795f-4b46-bf55-93731ce4d480",
  "userName": "Petya007",
  "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9",
  "refreshToken": "def50200a1b2c3v14ya85h",
  "createdAt": "2026-03-31T10:30:00.123Z"
}
```

## Ошибки

- **Клиентские:** `400`, `409`, `422`
- **Серверные:** `500`, `503`

### Сценарии с ошибкой

#### 400 — неправильный JSON, тип данных, отсутствие тела запроса

```json
{
  "errorCode": "BAD_REQUEST",
  "message": "Malformed JSON in request body"
}
```

#### 409 — конфликт запроса с текущим состоянием (например, `userName` уже занят)

```json
{
  "errorCode": "EMAIL_ALREADY_EXISTS",
  "message": "User with this email already exists"
}
```

#### 422 — синтаксических ошибок нет, но данные не соответствуют правилам

```json
{
  "errorCode": "VALIDATION_ERROR",
  "message": "Invalid input data. Password is too weak."
}
```

#### 500 — внутренняя неожиданная ошибка сервера

```json
{
  "errorCode": "INTERNAL_ERROR",
  "message": "Something went wrong. Please try again later"
}
```

#### 503 — временная недоступность сервиса

```json
{
  "errorCode": "SERVICE_UNAVAILABLE",
  "message": "Service temporarily unavailable. Please try later"
}
```

# Задача 3.2. Алгоритм регистрации пользователя на бэке

1. Получение запроса, проверка `Content-Type` и наличия тела запроса. Если нет — ошибка `400`.
2. Парсинг JSON, проверка структуры и валидация данных. При ошибке — `400`.
3. Бизнес-валидация. При ошибке — `422`.
4. Поиск пользователя в БД. Есть совпадения — `409`.
5. Хэширование пароля, генерация `userId`, `createdAt`, нормализация `userName`.
6. Создание пользователя в БД. Ошибка БД — `500`.
7. Генерация `accessToken` и `refreshToken`.
8. Формирование ответа (`userId`, `userName`, `accessToken`, `refreshToken`, `createdAt`) и возврат `201`.
