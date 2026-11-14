# Маркетплейс

## Frontend

## API коллекция

Необходимо создать API коллекцию с помощью Bruno и добавить её в репозиторий (папка с коллекцией должна называться `bruno-collection`)

## Backend

_Апи развернуто по адресу `https://api.kitek-pg.ru/api/marketplace`_

Этот проект представляет собой REST API для маркетплейса, где пользователи могут создавать карточки товаров на продажу и делать ставки на товары других пользователей.

## Описание API

### Аутентификация

API использует JWT токены для аутентификации. После регистрации или входа вы получите токен, который нужно передавать в заголовке `Authorization: Bearer <token>` для защищенных эндпоинтов.

---

## Публичные эндпоинты (не требуют аутентификации)

### 1. Регистрация пользователя
**POST** `/api/auth/register`

Создает новую учетную запись пользователя.

**Тело запроса:**
```json
{
  "username": "testuser",
  "password": "password123",
  "email": "test@example.com"
}
```

**Ответ (успех):**
```json
{
  "success": true,
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": {
    "id": 1,
    "username": "testuser",
    "email": "test@example.com"
  }
}
```

**Возможные ошибки:**
- `400` - Неверные данные (имя пользователя < 3 символов, пароль < 6 символов)
- `409` - Пользователь с таким именем уже существует

### 2. Вход в систему
**POST** `/api/auth/login`

Авторизация существующего пользователя.

**Тело запроса:**
```json
{
  "username": "testuser",
  "password": "password123"
}
```

**Ответ (успех):**
```json
{
  "success": true,
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": {
    "id": 1,
    "username": "testuser",
    "email": "test@example.com"
  }
}
```

**Возможные ошибки:**
- `400` - Не указано имя пользователя или пароль
- `401` - Неверное имя пользователя или пароль

### 3. Получение всех товаров
**GET** `/api/items`

Возвращает список всех товаров, отсортированных по дате создания (новые первые).

**Ответ:**
```json
[
  {
    "id": 1,
    "title": "Ноутбук Dell",
    "description": "Игровой ноутбук в отличном состоянии",
    "price": 50000,
    "userId": 1,
    "username": "testuser",
    "status": "active",
    "imageUrl": "https://example.com/laptop.jpg",
    "createdAt": "2025-10-07T16:45:52.058Z",
    "highestBid": 55000,
    "bidCount": 3
  }
]
```

### 4. Получение ставок на товар
**GET** `/api/items/:id/bids`

Возвращает все ставки на конкретный товар, отсортированные по дате (новые первые).

**Ответ:**
```json
[
  {
    "id": 1,
    "amount": 55000,
    "userId": 2,
    "username": "buyer1",
    "createdAt": "2025-10-07T17:00:00.000Z"
  },
  {
    "id": 2,
    "amount": 52000,
    "userId": 3,
    "username": "buyer2",
    "createdAt": "2025-10-07T16:55:00.000Z"
  }
]
```

### 5. Статистика платформы
**GET** `/api/stats`

Возвращает общую статистику платформы.

**Ответ:**
```json
{
  "totalItems": 15,
  "totalUsers": 8,
  "totalBids": 42,
  "activeItems": 12,
  "totalValue": 750000,
  "averageItemPrice": 25000
}
```

---

## Защищенные эндпоинты (требуют JWT токен)

### 1. Получение информации о текущем пользователе
**GET** `/api/auth/me`

**Заголовки:**
```
Authorization: Bearer <your-jwt-token>
```

**Ответ:**
```json
{
  "id": 1,
  "username": "testuser",
  "email": "test@example.com",
  "createdAt": "2025-10-07T16:22:34.688Z"
}
```

### 2. Создание товара
**POST** `/api/items`

**Заголовки:**
```
Authorization: Bearer <your-jwt-token>
```

**Тело запроса:**
```json
{
  "title": "iPhone 14",
  "description": "Новый iPhone 14 в идеальном состоянии, все аксессуары в комплекте",
  "price": 80000,
  "imageUrl": "https://example.com/iphone.jpg"
}
```

**Ответ (успех):**
```json
{
  "id": 5,
  "title": "iPhone 14",
  "description": "Новый iPhone 14 в идеальном состоянии, все аксессуары в комплекте",
  "price": 80000,
  "imageUrl": "https://example.com/iphone.jpg",
  "userId": 1,
  "username": "testuser",
  "status": "active",
  "highestBid": null,
  "bidCount": 0,
  "createdAt": "2025-10-07T18:00:00.000Z"
}
```

**Возможные ошибки:**
- `400` - Не указано название товара или описание
- `400` - Не указана цена или цена <= 0
- `400` - Название слишком длинное (> 100 символов)
- `400` - Описание слишком длинное (> 1000 символов)
- `401` - Не указан токен аутентификации
- `403` - Недействительный токен

### 3. Удаление товара
**DELETE** `/api/items/:id`

Удаляет товар. Можно удалить только свой товар и только если на него нет ставок.

**Заголовки:**
```
Authorization: Bearer <your-jwt-token>
```

**Ответ (успех):**
```json
{
  "success": true,
  "message": "Item deleted"
}
```

**Возможные ошибки:**
- `403` - Нет прав на удаление этого товара
- `404` - Товар не найден
- `400` - Нельзя удалить товар, на который уже есть ставки

### 4. Создание ставки на товар
**POST** `/api/items/:id/bids`

Создает ставку на товар. Ставка должна быть выше текущей максимальной ставки или начальной цены.

**Заголовки:**
```
Authorization: Bearer <your-jwt-token>
```

**Тело запроса:**
```json
{
  "amount": 85000
}
```

**Ответ (успех):**
```json
{
  "id": 10,
  "itemId": 5,
  "userId": 2,
  "username": "buyer1",
  "amount": 85000,
  "createdAt": "2025-10-07T18:30:00.000Z"
}
```

**Возможные ошибки:**
- `400` - Не указана сумма ставки или сумма <= 0
- `400` - Ставка должна быть выше текущей максимальной ставки
- `400` - Нельзя делать ставку на свой товар
- `400` - Товар не доступен для торгов
- `404` - Товар не найден

### 5. Получение моих ставок
**GET** `/api/bids/my`

Возвращает все ставки текущего пользователя.

**Заголовки:**
```
Authorization: Bearer <your-jwt-token>
```

**Ответ:**
```json
[
  {
    "id": 10,
    "amount": 85000,
    "itemId": 5,
    "itemTitle": "iPhone 14",
    "createdAt": "2025-10-07T18:30:00.000Z",
    "isWinning": true
  },
  {
    "id": 8,
    "amount": 45000,
    "itemId": 3,
    "itemTitle": "Macbook Air",
    "createdAt": "2025-10-07T17:15:00.000Z",
    "isWinning": false
  }
]
```

---

## Структура данных

### Пользователь (User)
```json
{
  "id": 1,
  "username": "testuser",
  "email": "test@example.com",
  "password": "hashed_password",
  "createdAt": "2025-10-07T16:22:34.688Z"
}
```

### Товар (Item)
```json
{
  "id": 1,
  "title": "Название товара",
  "description": "Описание товара",
  "price": 50000,
  "imageUrl": "https://example.com/image.jpg",
  "userId": 1,
  "username": "seller",
  "status": "active",
  "highestBid": 55000,
  "bidCount": 3,
  "createdAt": "2025-10-07T16:45:52.058Z"
}
```

### Ставка (Bid)
```json
{
  "id": 1,
  "itemId": 1,
  "userId": 2,
  "username": "buyer",
  "amount": 55000,
  "createdAt": "2025-10-07T17:00:00.000Z"
}
```

---

## Коды ошибок

- `200` - Успешный запрос
- `201` - Ресурс создан
- `400` - Неверный запрос (некорректные данные)
- `401` - Не авторизован (требуется токен)
- `403` - Доступ запрещен (недействительный токен или нет прав)
- `404` - Ресурс не найден
- `409` - Конфликт (например, пользователь уже существует)
- `500` - Внутренняя ошибка сервера

---

## Примеры использования

### Полный цикл работы с API:

1. **Регистрация:**
```bash
curl -X POST http://localhost:3001/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{"username": "seller1", "password": "password123", "email": "seller1@test.com"}'
```

2. **Создание товара:**
```bash
curl -X POST http://localhost:3001/api/items \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_TOKEN_HERE" \
  -d '{
    "title": "Игровая клавиатура",
    "description": "Механическая клавиатура с RGB подсветкой",
    "price": 5000,
    "imageUrl": "https://example.com/keyboard.jpg"
  }'
```

3. **Создание ставки:**
```bash
curl -X POST http://localhost:3001/api/items/1/bids \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer BUYER_TOKEN_HERE" \
  -d '{"amount": 5500}'
```

4. **Получение всех товаров:**
```bash
curl -X GET http://localhost:3001/api/items
```

# Как сдавать

- Создайте форк репозитория в вашей организации с названием-этого-репозитория-вашафамилия
- Используя ветку wip сделайте задание
- Зафиксируйте изменения в вашем репозитории
- Когда документ будет готов - создайте пул реквест из ветки wip (вашей) на ветку main (тоже вашу) и укажите меня (ktkv419) как reviewer

Не мержите сами коммит, это сделаю я после проверки задания
