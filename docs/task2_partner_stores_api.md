# Задание 2: Проектирование REST API — экран "Выберите магазин"

## Что должен показывать экран (по макету)
- Заголовок: "Выберите магазин"
- Список карточек магазинов партнёров
- Для каждой карточки:
  - название
  - логотип
  - информация о доставке:
    - либо ближайшее окно времени "сегодня 21:00–23:00"
    - либо “Быстрая доставка от 20 до 60 минут”
  - при клике по карточке — переход по ссылке на внешний ресурс партнёра

## Допущения
- Список партнёров зависит от города/региона пользователя.
- Формирование “сегодня” и слотов делается с учётом таймзоны пользователя.

---

## Пример REST API запроса

### Endpoint
`GET /api/v1/partner-stores`

### Query parameters
- `cityId` (string, required) — идентификатор города/региона (пример: `msk`)
- `locale` (string, optional) — локаль интерфейса (пример: `ru-RU`)
- `tz` (string, optional) — таймзона (пример: `Europe/Moscow`)

### Request example
```http
GET /api/v1/partner-stores?cityId=msk&locale=ru-RU&tz=Europe%2FMoscow
Accept: application/json


{
  "screen": {
    "title": "Выберите магазин"
  },
  "items": [
    {
      "id": "metro",
      "name": "METRO",
      "logoUrl": "https://cdn.petrushka.example/partners/metro.png",
      "delivery": {
        "type": "TIME_WINDOW",
        "title": "Ближайшая доставка",
        "dayLabel": "сегодня",
        "window": { "from": "21:00", "to": "23:00" },
        "displayText": "Ближайшая доставка сегодня 21:00–23:00"
      },
      "action": {
        "type": "EXTERNAL_LINK",
        "externalUrl": "https://partner.example/metro"
      }
    },
    {
      "id": "auchan",
      "name": "Ашан",
      "logoUrl": "https://cdn.petrushka.example/partners/auchan.png",
      "delivery": {
        "type": "TIME_WINDOW",
        "title": "Ближайшая доставка",
        "dayLabel": "сегодня",
        "window": { "from": "18:00", "to": "20:00" },
        "displayText": "Ближайшая доставка сегодня 18:00–20:00"
      },
      "action": {
        "type": "EXTERNAL_LINK",
        "externalUrl": "https://partner.example/auchan"
      }
    },
    {
      "id": "vkusvill",
      "name": "ВкусВилл",
      "logoUrl": "https://cdn.petrushka.example/partners/vkusvill.png",
      "delivery": {
        "type": "EXPRESS_RANGE",
        "title": "Быстрая доставка",
        "rangeMinutes": { "min": 20, "max": 60 },
        "displayText": "Быстрая доставка от 20 до 60 минут"
      },
      "action": {
        "type": "EXTERNAL_LINK",
        "externalUrl": "https://partner.example/vkusvill"
      }
    },
    {
      "id": "victoria",
      "name": "ВИКТОРИЯ",
      "logoUrl": "https://cdn.petrushka.example/partners/victoria.png",
      "delivery": {
        "type": "TIME_WINDOW",
        "title": "Ближайшая доставка",
        "dayLabel": "сегодня",
        "window": { "from": "17:00", "to": "19:00" },
        "displayText": "Ближайшая доставка сегодня 17:00–19:00"
      },
      "action": {
        "type": "EXTERNAL_LINK",
        "externalUrl": "https://partner.example/victoria"
      }
    }
  ],
  "meta": {
    "generatedAt": "2026-02-10T21:30:00+03:00",
    "cityId": "msk"
  }
}
