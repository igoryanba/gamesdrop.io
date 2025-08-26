# GamesDrop.io API - Руководство для мерчантов

## Оглавление
1. [Авторизация](#авторизация)
   - [Получение токена](#получение-токена)
   - [Безопасность](#безопасность-токена)
2. [Основные концепции](#основные-концепции)
   - [Статусы заказов](#статусы-заказов)
   - [Обработка ошибок](#возможные-ошибки)
3. [API Endpoints](#api-endpoints)
   - [Управление балансом](#проверка-баланса)
     - [Проверка баланса](#проверка-баланса)
     - [История транзакций](#история-транзакций-баланса)
   - [Управление товарами](#получение-информации-о-товаре)
   - [Управление заказами](#создание-заказа)
   - [Валидация игрока](#проверка-игрока)
   - [Telegram Stars](#telegram-stars)
4. [Тестирование API](#тестирование-api)
   - [Тестовый товар](#тестовый-товар)
5. [Рекомендации по интеграции](#рекомендации-по-использованию)
6. [Поддержка](#поддержка)

## Авторизация

### Получение токена

#### Процесс получения
1. Войдите в личный кабинет мерчанта
2. Создайте новый магазин
3. После создания магазина вы получите уникальный токен
4. Формат токена: `abcdef1234567890abcdef1234567890`

### Безопасность токена
- 🔒 Токен является конфиденциальной информацией
- ⚠️ Не передавайте токен третьим лицам
- 📝 Токен нельзя восстановить после создания
- 🔄 При необходимости можно сгенерировать новый токен (старый станет недействительным)

## Основные концепции

### Статусы заказов

| Статус | Описание | Действия |
|--------|----------|----------|
| `SUBMITTED` | Заказ создан и ожидает обработки | Ожидать перехода в PROCESSING |
| `PROCESSING` | Заказ в процессе обработки | Ожидать завершения |
| `COMPLETED` | Заказ успешно выполнен | Получить ключ/товар |
| `CANCELED` | Заказ отменен из-за ошибки | Создать новый заказ |
| `REFUND` | Заказ возвращен | Ожидать возврат средств |

### Возможные ошибки

| Код ошибки | Описание | Решение |
|------------|----------|----------|
| `INVALID_TOKEN` | Неверный токен авторизации | Проверить токен или получить новый |
| `OFFER_NOT_FOUND` | Товар не найден или нет доступа | Проверить ID товара и права доступа |
| `TRANSACTION_DUPLICATE` | Дубликат транзакции | Использовать новый transaction_id |
| `WRONG_PRICE` | Неверная цена товара | Обновить информацию о цене |
| `ORDER_NOT_FOUND` | Заказ не найден | Проверить ID заказа |
| `ORDER_NOT_PROCESSING` | Заказ еще не в обработке | Дождаться перехода в статус PROCESSING |
| `ORDER_NOT_COMPLETED` | Заказ еще не завершен | Дождаться завершения заказа |
| `ORDER_ALREADY_CANCELED` | Заказ уже отменен | Создать новый заказ |
| `ORDER_ALREADY_REFUNDED` | Заказ уже возвращен | Создать новый заказ |
| `SERVICE_UNAVAILABLE` | Сервис временно недоступен | Повторить попытку позже |
| `INVALID_REQUEST_BODY` | Неверный формат запроса | Проверить структуру запроса |
| `INSUFFICIENT_BALANCE` | Недостаточно средств на балансе | Пополнить баланс или уменьшить сумму покупки |
| `BALANCE_UNAVAILABLE` | Баланс временно недоступен | Повторить запрос позже |

## API Endpoints

### Проверка баланса

```http
GET /api/v1/balance
Authorization: {{token}}
```

**Response:**
```json
{
  "balance": 500.00,
  "draftBalance": 50.00,
  "currency": {
    "id": 3,
    "code": "USD"
  }
}
```

**Описание полей ответа:**
| Ключ | Значение | Дополнение |
|------|----------|------------|
| `balance` | `number` | Основной баланс партнера |
| `draftBalance` | `number` | Черновой баланс (средства в обработке) |
| `currency` | `object` | Информация о валюте |
| `currency.id` | `number` | ID валюты в системе |
| `currency.code` | `string` | Код валюты (всегда `USD`) |

### История транзакций баланса

```http
GET /api/v1/balance/transactions?page=1&limit=20
Authorization: {{token}}
```

**Response:**
```json
{
  "transactions": [
    {
      "id": 12,
      "amount": -75.00,
      "type": "PURCHASE",
      "description": "API Purchase - Virtual credits",
      "balanceAfter": 425.00,
      "createdAt": "2025-07-28T05:21:46.121Z"
    },
    {
      "id": 11,
      "amount": -50.00,
      "type": "PURCHASE", 
      "description": "Test Purchase - Mobile game credits",
      "balanceAfter": 500.00,
      "createdAt": "2025-07-28T05:16:23.718Z"
    }
  ],
  "total": 25
}
```

**Описание полей запроса:**
| Ключ | Значение | Дополнение |
|------|----------|------------|
| `page` | `number` | Номер страницы (по умолчанию: 1) |
| `limit` | `number` | Количество записей на странице (по умолчанию: 20) |

**Описание полей ответа:**
| Ключ | Значение | Дополнение |
|------|----------|------------|
| `transactions` | `array` | Список транзакций |
| `transactions[].id` | `number` | Уникальный ID транзакции |
| `transactions[].amount` | `number` | Сумма операции (отрицательная для списаний) |
| `transactions[].type` | `string` | Тип операции (`DEPOSIT`, `PURCHASE`, `REFUND`) |
| `transactions[].description` | `string` | Описание операции |
| `transactions[].balanceAfter` | `number` | Баланс после операции |
| `transactions[].createdAt` | `string` | Дата и время операции (UTC) |
| `total` | `number` | Общее количество транзакций |

### Получение информации о товаре

```http
POST /api/v1/offers/find-one
Authorization: {{token}}

{
  "offerId": 1001
}
```

**Response:**
```json
{
  "offerId": 1001,
  "productName": "PUBG MOBILE GIFT",
  "offerName": "60 UC",
  "count": 1,
  "price": 560.10,
  "currency": "RUB",
  "isReturnDataForCustomer": true
}
```

**Описание полей запроса:**
| Ключ | Значение | Дополнение |
|------|----------|------------|
| `offerId` | `number` | Важно: должно быть числом, не строкой |

**Описание полей ответа:**
| Ключ | Значение | Дополнение |
|------|----------|------------|
| `offerId` | `number` | - |
| `productName` | `string` | - |
| `offerName` | `string` | - |
| `count` | `number` | Количество единиц товара |
| `price` | `number` | - |
| `currency` | `KZT` \| `USD` \| `EUR` \| `RUB` | - |
| `isReturnDataForCustomer` | `boolean` | Возвращается ли ключ для активации |

### Создание заказа

```http
POST /api/v1/offers/create-order
Authorization: {{token}}

{
  "offerId": 1001,
  "price": 560.10,
  "transactionId": "test112321124214",
  "customer": {
    "email": "user@gmail.com",
    "gameUserId": "52357322414"
  }
}
```

**Описание полей запроса:**
| Ключ | Значение | Дополнение |
|------|----------|------------|
| `offerId` | `number` | Важно: должно быть числом, не строкой |
| `price` | `number` | - |
| `transactionId` | `string` | Уникальный идентификатор транзакции в вашей системе |
| `customer` | `undefined` \| `object` | - |
| `email` | `undefined` \| `string` | Необязательное поле для отслеживания |
| `gameUserId` | `undefined` \| `string` | Обязательно для некоторых типов товаров |

**Response:**
```json
{
  "orderId": 10222502,
  "count": 1,
  "price": 560.10,
  "currency": "RUB",
  "offerId": 1001,
  "productName": "PUBG MOBILE GIFT",
  "offerName": "60 UC",
  "status": "COMPLETED",
  "isReturnDataForCustomer": true,
  "key": "001434249936",
  "createdAt": "2024-05-28 10:08:04.296+00"
}
```

**Описание полей ответа:**
| Ключ | Значение | Дополнение |
|------|----------|------------|
| `orderId` | `number` | - |
| `count` | `number` | Количество единиц товара |
| `price` | `number` | - |
| `currency` | `KZT` \| `USD` \| `EUR` \| `RUB` | - |
| `offerId` | `number` | - |
| `productName` | `string` | - |
| `offerName` | `string` | - |
| `status` | `string` | Для некоторых номиналов обработка может занять время |
| `isReturnDataForCustomer` | `boolean` | - |
| `key` | `undefined` \| `string` | Возвращается только при COMPLETED и isReturnDataForCustomer=true |
| `createdAt` | `string` | UTC +0 |

### Проверка статуса заказа

```http
POST /api/v1/offers/order-status
Authorization: {{token}}

{
  "orderId": 10222502
}
```

**Описание полей запроса:**
| Ключ | Значение | Дополнение |
|------|----------|------------|
| `orderId` | `number` | - |

**Response:**
```json
{
  "orderId": 10222502,
  "count": 1,
  "price": 560.10,
  "currency": "RUB",
  "offerId": 1001,
  "productName": "PUBG MOBILE GIFT",
  "offerName": "60 UC",
  "status": "COMPLETED",
  "isReturnDataForCustomer": true,
  "key": "001434249936",
  "createdAt": "2024-05-28 10:08:04.296+00"
}
```

**Описание полей ответа:**
| Ключ | Значение | Дополнение |
|------|----------|------------|
| `orderId` | `number` | - |
| `count` | `number` | Количество единиц товара |
| `price` | `number` | - |
| `currency` | `KZT` \| `USD` \| `EUR` \| `RUB` | - |
| `offerId` | `number` | - |
| `productName` | `string` | - |
| `offerName` | `string` | - |
| `status` | `string` | Текущий статус заказа |
| `isReturnDataForCustomer` | `boolean` | - |
| `key` | `undefined` \| `string` | Возвращается только при COMPLETED и isReturnDataForCustomer=true |
| `createdAt` | `string` | UTC +0 |

### Проверка игрока

```http
POST /api/v1/offers/check-game-data
Authorization: {{token}}

{
  "offerId": 1001,
  "price": 560.10,
  "transactionId": "112321124214",
  "customer": {
    "email": "user@gmail.com",
    "gameUserId": "52357322414"
  }
}
```

**Описание полей запроса:**
| Ключ | Значение | Дополнение |
|------|----------|------------|
| `offerId` | `number` | Важно: должно быть числом, не строкой |
| `price` | `number` | - |
| `transactionId` | `string` | Уникальный идентификатор транзакции |
| `customer` | `object` | - |
| `email` | `undefined` \| `string` | Необязательное поле |
| `gameUserId` | `string` | Идентификатор игрока |
| `gameServerId` | `undefined` \| `string` | Идентификатор сервера (если требуется) |

**Успешный ответ:**
```json
{
  "status": "VALID",
  "gameUserLogin": "JJJ"
}
```

**Описание полей успешного ответа:**
| Ключ | Значение | Дополнение |
|------|----------|------------|
| `status` | `"VALID"` | Игрок валиден |
| `gameUserLogin` | `string` | Логин игрока |

**Ответ при ошибке:**
```json
{
  "status": "INVALID"
}
```

**Описание полей ответа с ошибкой:**
| Ключ | Значение | Дополнение |
|------|----------|------------|
| `status` | `"INVALID"` | Игрок не валиден |

## Тестирование API

### Тестовый товар

Для тестирования интеграции с API доступен специальный тестовый товар:

- **ID товара:** 999
- **Название:** Steam US
- **Наименование:** TEST OFFER GROUP
- **Цена:** 23.09 KZT (важно использовать точное значение)
- **Возвращает ключ:** Да

**Пример запроса информации о тестовом товаре:**
```http
POST /api/v1/offers/find-one
Authorization: {{token}}

{
  "offerId": 999
}
```

**Пример создания тестового заказа:**
```http
POST /api/v1/offers/create-order
Authorization: {{token}}

{
  "offerId": 999,
  "price": 23.09,
  "transactionId": "test_123456",
  "customer": {
    "email": "test@example.com",
    "gameUserId": "123456789"
  }
}
```

**Особенности тестового товара:**
- Всегда возвращает успешный ответ при правильных параметрах
- Генерирует тестовый ключ активации
- Создаёт заказ со статусом COMPLETED
- Проверка игрока всегда возвращает VALID для тестового товара
- Работает в тестовом режиме без реального списания средств

### Примеры работы с балансом

**Проверка баланса перед покупкой:**
```javascript
// 1. Проверяем текущий баланс
const balanceResponse = await fetch('/api/v1/balance', {
  headers: { 'Authorization': 'your-token-here' }
});
const { balance } = await balanceResponse.json();

// 2. Получаем информацию о товаре
const offerResponse = await fetch('/api/v1/offers/find-one', {
  method: 'POST',
  headers: { 
    'Authorization': 'your-token-here',
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({ offerId: 1001 })
});
const { price } = await offerResponse.json();

// 3. Проверяем достаточность средств
if (balance >= price) {
  // Создаем заказ
  console.log('Sufficient balance, creating order...');
} else {
  console.log('Insufficient balance, need to top up');
}
```

**Получение истории транзакций:**
```javascript
const transactionsResponse = await fetch('/api/v1/balance/transactions', {
  headers: { 'Authorization': 'your-token-here' }
});
const { transactions, total } = await transactionsResponse.json();

console.log(`Total transactions: ${total}`);
transactions.forEach(tx => {
  console.log(`${tx.createdAt}: ${tx.type} ${tx.amount} (Balance: ${tx.balanceAfter})`);
});
```

## Telegram Stars

🌟 **GamesDrop интегрировал поддержку Telegram Stars!** Теперь вы можете отправлять Telegram Stars пользователям через те же стандартные API эндпоинты.

### Доступные продукты

| Offer ID | Product Name | Количество Stars | Примерная цена |
|----------|--------------|------------------|----------------|
| `telegram_stars_50` | 50 Telegram Stars | 50 | ~$0.39 |
| `telegram_stars_100` | 100 Telegram Stars | 100 | ~$0.78 |
| `telegram_stars_500` | 500 Telegram Stars | 500 | ~$3.88 |
| `telegram_stars_1000` | 1000 Telegram Stars | 1000 | ~$7.77 |

*Цены динамические на основе реального TON курса (обновляются ежедневно)*

### 🎁 Premium подписки и подарки

| Offer ID | Product Name | Длительность | Функции |
|----------|--------------|--------------|---------|
| `telegram_premium_1m` | Telegram Premium | 1 месяц | Без рекламы, расширенные лимиты |
| `telegram_premium_3m` | Telegram Premium | 3 месяца | + Уникальные реакции |
| `telegram_premium_6m` | Telegram Premium | 6 месяцев | + Голос в текст |
| `telegram_premium_12m` | Telegram Premium | 12 месяцев | Все возможности Premium |
| `telegram_premium_gift_1m` | Premium подарок | 1 месяц | Подарок Premium + буст слот |
| `telegram_premium_gift_3m` | Premium подарок | 3 месяца | Подарок Premium + буст слоты |

### Использование тех же эндпоинтов

**1. Получение информации о Telegram Stars:**
```http
POST /api/v1/offers/find-one
Authorization: {{token}}

{
  "offerId": "telegram_stars_100"
}
```

**Response:**
```json
{
  "offerId": "telegram_stars_100",
  "productName": "Telegram Stars",
  "offerName": "100 Stars",
  "count": 1,
  "price": 0.78,
  "currency": "USD",
  "isReturnDataForCustomer": true
}
```

**2. Создание заказа на Telegram Stars:**
```http
POST /api/v1/offers/create-order
Authorization: {{token}}

{
  "offerId": "telegram_stars_100",
  "price": 0.78,
  "transactionId": "tg_stars_12345",
  "customer": {
    "email": "user@example.com",
    "gameUserId": "143594291"
  }
}
```

**Response при успешной доставке:**
```json
{
  "orderId": 10228901,
  "count": 1,
  "price": 0.78,
  "currency": "USD",
  "offerId": "telegram_stars_100",
  "productName": "Telegram Stars",
  "offerName": "100 Stars",
  "status": "COMPLETED",
  "isReturnDataForCustomer": true,
  "fulfillmentData": {
    "telegram_user_id": 143594291,
    "stars_amount": 100,
    "transaction_id": "tg_tx_abc123",
    "delivery_status": "completed"
  },
  "createdAt": "2025-08-26T12:30:15.234Z"
}
```

**Response при ошибке доставки:**
```json
{
  "orderId": 10228902,
  "status": "CANCELED",
  "message": "Failed to deliver Stars: user not found",
  "fulfillmentData": {
    "telegram_user_id": 123456789,
    "stars_amount": 100,
    "delivery_status": "failed",
    "error_message": "user not found"
  },
  "createdAt": "2025-08-26T12:30:15.234Z"
}
```

### 🔑 Важные особенности Telegram Stars

#### gameUserId требования:
- **gameUserId должен быть Telegram User ID** (число)
- Получить можно через @userinfobot в Telegram
- Пример: `"gameUserId": "143594291"`
- ❌ НЕ username (@username) - это не работает

#### Статусы доставки:
- `COMPLETED` - Stars успешно доставлены пользователю
- `CANCELED` - Не удалось доставить (неверный user ID, заблокирован бот, etc.)

#### Типичные ошибки доставки:
- `"user not found"` - неверный Telegram User ID
- `"STARGIFT_INVALID"` - пользователь не может получить Stars (restrictions)
- `"bot was blocked by user"` - пользователь заблокировал бота

### 📱 Интеграция для разработчиков

**Пример полного flow на JavaScript:**
```javascript
// 1. Получить актуальную цену
const getStarsPrice = async (starsAmount) => {
  const response = await fetch('/api/v1/offers/find-one', {
    method: 'POST',
    headers: {
      'Authorization': 'your-token-here',
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({ 
      offerId: `telegram_stars_${starsAmount}` 
    })
  });
  return response.json();
};

// 2. Отправить Stars пользователю
const sendStars = async (userTelegramId, starsAmount) => {
  const { price } = await getStarsPrice(starsAmount);
  
  const response = await fetch('/api/v1/offers/create-order', {
    method: 'POST',
    headers: {
      'Authorization': 'your-token-here',
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      offerId: `telegram_stars_${starsAmount}`,
      price: price,
      transactionId: `stars_${Date.now()}`,
      customer: {
        email: "optional@example.com",
        gameUserId: userTelegramId.toString()
      }
    })
  });
  
  const result = await response.json();
  
  if (result.status === 'COMPLETED') {
    console.log(`✅ Successfully sent ${starsAmount} Stars to ${userTelegramId}`);
    return result;
  } else {
    console.error(`❌ Failed to send Stars: ${result.message}`);
    throw new Error(result.message);
  }
};

// Использование:
sendStars(143594291, 100)
  .then(order => console.log('Order created:', order.orderId))
  .catch(error => console.error('Delivery failed:', error));
```

### 💼 B2B кейсы использования

**1. Награды в играх:**
```javascript
// Наградить игрока за достижение
await sendStars(userTelegramId, 50);
```

**2. Промо-кампании:**
```javascript
// Отправить Stars всем участникам конкурса
const winners = [143594291, 987654321, 456789123];
for (const userId of winners) {
  await sendStars(userId, 100);
}
```

**3. Кэшбек программы:**
```javascript
// Вернуть часть покупки в виде Stars
const cashbackAmount = Math.floor(purchaseAmount * 0.05); // 5% кэшбек
const starsAmount = Math.min(cashbackAmount * 100, 1000); // конвертируем в Stars
await sendStars(userTelegramId, starsAmount);
```

### ⚡ Performance и лимиты

- **Rate limiting**: 30 запросов в секунду на Telegram API
- **Minimum amount**: 1 Star
- **Maximum amount**: 2500 Stars за одну транзакцию
- **Retry logic**: Автоматические повторы при временных ошибках
- **Delivery time**: Мгновенная доставка (< 3 секунды)

### 🛡️ Безопасность

- Валидация Telegram User ID перед отправкой
- Проверка баланса GamesDrop перед созданием заказа  
- Логирование всех операций для аудита
- Защита от дублированных транзакций через `transactionId`

## Рекомендации по использованию

### 💰 Работа с балансом
- Регулярно проверяйте баланс перед крупными покупками
- Ведите учет транзакций для сверки с вашей системой
- При недостатке средств уведомляйте пользователей о необходимости пополнения
- Используйте пагинацию при запросе истории транзакций

### 🔍 Перед созданием заказа
- Получите актуальную информацию о товаре
- Проверьте достаточность баланса для покупки
- Проверьте валидность игрока (в случае прямого пополнения)

### 📝 При создании заказа
- Используйте уникальный transactionId
- Указывайте актуальную цену
- Указывайте offerId как число, не как строку
- Заполняйте все необходимые поля для данного типа товара

### ✅ После создания заказа
- Сохраните orderId
- Проверяйте статус заказа
- При статусе COMPLETED получите ключ/товар

### ⚠️ При возникновении ошибок
- Проверьте токен
- Убедитесь в корректности данных и формате запроса
- Создайте новый заказ при необходимости

## Поддержка

При возникновении вопросов обращайтесь в техническую поддержку:
- 📧 Email: support@gamesdrop.io
- 💬 Telegram: @gamesdrop_support

### Рекомендации по обработке ошибок

#### 🔍 Проверка перед запросом
- Валидация токена
- Проверка ID товара
- Актуальность цены
- Уникальность transaction_id
- Корректность формата данных (особенно offerId как число)

#### 🛠 Обработка ответов
- Обработка всех кодов ошибок
- Логирование ошибок
- Механизм повторных попыток

#### 📊 Работа с заказами
- Сохранение ID заказов
- Мониторинг статусов
- Актуализация данных


#### 📊 Работа с заказами
- Сохранение ID заказов
- Мониторинг статусов
- Актуализация данных
