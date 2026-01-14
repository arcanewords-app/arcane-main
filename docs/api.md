# 🔗 REST API Reference

Arcane Reader предоставляет REST API для управления проектами и переводами.

**Base URL:** `http://localhost:3000`

---

## 📊 Status

### GET /api/status

Проверка состояния сервера и AI провайдера.

**Response:**

```json
{
  "status": "ok",
  "timestamp": "2026-01-08T12:00:00.000Z",
  "ai": {
    "provider": "OpenAI",
    "model": "gpt-4-turbo-preview",
    "configured": true
  },
  "database": {
    "type": "lowdb",
    "projects": 3
  }
}
```

---

## 📁 Projects

### GET /api/projects

Получить список всех проектов.

**Response:**

```json
[
  {
    "id": "abc123xyz",
    "name": "My Novel",
    "sourceLanguage": "en",
    "targetLanguage": "ru",
    "chapters": [...],
    "glossary": [...],
    "createdAt": "2026-01-08T10:00:00.000Z",
    "updatedAt": "2026-01-08T12:00:00.000Z"
  }
]
```

---

### POST /api/projects

Создать новый проект.

**Request:**

```json
{
  "name": "New Novel",
  "sourceLanguage": "en",
  "targetLanguage": "ru"
}
```

**Response:** `201 Created`

```json
{
  "id": "def456uvw",
  "name": "New Novel",
  "sourceLanguage": "en",
  "targetLanguage": "ru",
  "chapters": [],
  "glossary": [],
  "settings": {
    "model": "gpt-4-turbo-preview",
    "temperature": 0.7,
    "skipEditing": false
  },
  "createdAt": "2026-01-08T14:00:00.000Z",
  "updatedAt": "2026-01-08T14:00:00.000Z"
}
```

---

### GET /api/projects/:id

Получить проект по ID.

**Response:**

```json
{
  "id": "abc123xyz",
  "name": "My Novel",
  ...
}
```

**Errors:**

- `404` — Project not found

---

### DELETE /api/projects/:id

Удалить проект.

**Response:** `200 OK`

```json
{
  "success": true
}
```

---

## 📖 Chapters

### POST /api/projects/:id/chapters

Добавить главу в проект.

**Request:**

```json
{
  "title": "Chapter 1: The Beginning",
  "originalText": "It was a dark and stormy night..."
}
```

**Response:** `201 Created`

```json
{
  "id": "ch_001",
  "number": 1,
  "title": "Chapter 1: The Beginning",
  "originalText": "It was a dark and stormy night...",
  "status": "pending"
}
```

---

### POST /api/projects/:projectId/chapters/upload

Загрузить главу из файла (multipart/form-data).

**Form fields:**

- `file` — .txt файл с текстом главы
- `title` (optional) — название главы

**Response:** `201 Created`

```json
{
  "id": "ch_002",
  "number": 2,
  "title": "chapter-2.txt",
  "originalText": "...",
  "status": "pending"
}
```

---

### GET /api/projects/:projectId/chapters/:chapterId

Получить главу.

**Response:**

```json
{
  "id": "ch_001",
  "number": 1,
  "title": "Chapter 1",
  "originalText": "...",
  "translatedText": "...",
  "status": "completed",
  "translationMeta": {
    "tokensUsed": 2840,
    "duration": 12345,
    "model": "gpt-4-turbo-preview",
    "translatedAt": "2026-01-08T12:00:00.000Z"
  }
}
```

---

### POST /api/projects/:projectId/chapters/:chapterId/translate

Запустить перевод главы.

**Response:** `200 OK`

```json
{
  "status": "started",
  "chapterId": "ch_001"
}
```

Перевод выполняется асинхронно. Проверяйте статус главы через GET.

**Statuses:**

- `pending` — ожидает перевода
- `translating` — в процессе
- `completed` — завершён
- `error` — ошибка

---

## 📚 Glossary

### GET /api/projects/:id/glossary

Получить глоссарий проекта.

**Response:**

```json
[
  {
    "id": "gl_001",
    "type": "character",
    "original": "John",
    "translated": "Джон",
    "gender": "male",
    "description": "Main protagonist, young mage-researcher",
    "firstAppearance": 1,
    "imageUrls": ["/uploads/project1/gl_001/image1.jpg"],
    "declensions": {
      "nominative": "Джон",
      "genitive": "Джона",
      "dative": "Джону",
      "accusative": "Джона",
      "instrumental": "Джоном",
      "prepositional": "Джоне"
    },
    "autoDetected": true
  },
  {
    "id": "gl_002",
    "type": "location",
    "original": "Crystal Palace",
    "translated": "Хрустальный дворец",
    "description": "Capital city, major trading hub",
    "firstAppearance": 2,
    "imageUrls": []
  },
  {
    "id": "gl_003",
    "type": "term",
    "original": "mana",
    "translated": "мана",
    "description": "Magical energy used for casting spells",
    "notes": "Категория: магия",
    "firstAppearance": 1
  }
]
```

---

### POST /api/projects/:id/glossary

Добавить запись в глоссарий.

**Request:**

```json
{
  "type": "character",
  "original": "Alexander",
  "translated": "Александр",
  "gender": "male",
  "description": "Главный герой, молодой маг-исследователь",
  "notes": "Дополнительные заметки для проверки",
  "firstAppearance": 1
}
```

> **Авто-склонения**: Если `type: "character"` и `declensions` не указаны,
> система автоматически сгенерирует падежные формы.

**Response:** `201 Created`

```json
{
  "id": "gl_004",
  "type": "character",
  "original": "Alexander",
  "translated": "Александр",
  "gender": "male",
  "description": "Главный герой, молодой маг-исследователь",
  "notes": "Дополнительные заметки для проверки",
  "firstAppearance": 1,
  "declensions": {
    "nominative": "Александр",
    "genitive": "Александра",
    "dative": "Александру",
    "accusative": "Александра",
    "instrumental": "Александром",
    "prepositional": "Александре"
  }
}
```

---

### PUT /api/projects/:projectId/glossary/:entryId

Обновить запись в глоссарии.

**Request:**

```json
{
  "type": "character",
  "original": "Alexander",
  "translated": "Александр",
  "gender": "male",
  "description": "Главный герой, молодой маг-исследователь",
  "notes": "Дополнительные заметки",
  "declensions": {
    "nominative": "Александр",
    "genitive": "Александра",
    ...
  }
}
```

**Response:** `200 OK`

```json
{
  "id": "gl_004",
  "type": "character",
  "original": "Alexander",
  "translated": "Александр",
  "gender": "male",
  "description": "Главный герой, молодой маг-исследователь",
  "notes": "Дополнительные заметки",
  "firstAppearance": 1,
  "declensions": { ... }
}
```

> **Примечание**: Если `type: "character"` и `declensions` не указаны, система автоматически пересоздаст склонения.

---

### DELETE /api/projects/:projectId/glossary/:entryId

Удалить запись из глоссария.

**Response:** `200 OK`

```json
{
  "success": true
}
```

---

### POST /api/projects/:projectId/glossary/:entryId/image

Добавить изображение в галерею записи глоссария.

**Request:** `multipart/form-data`

- `image` (file, required) — файл изображения

**Response:** `200 OK`

```json
{
  "id": "gl_001",
  "imageUrls": [
    "/uploads/project1/gl_001/image1.jpg",
    "/uploads/project1/gl_001/image2.jpg"
  ]
}
```

---

### DELETE /api/projects/:projectId/glossary/:entryId/image/:index

Удалить конкретное изображение из галереи по индексу.

**Response:** `200 OK`

```json
{
  "id": "gl_001",
  "imageUrls": [
    "/uploads/project1/gl_001/image1.jpg"
  ]
}
```

---

### DELETE /api/projects/:projectId/glossary/:entryId/image

Удалить все изображения из галереи записи.

**Response:** `200 OK`

```json
{
  "id": "gl_001",
  "imageUrls": []
}
```

---

## 📤 Export

### POST /api/projects/:id/export

Экспортировать проект в формат EPUB или FB2.

**Request:**

```json
{
  "format": "epub",
  "author": "Переведено Arcane"
}
```

**Параметры:**

- `format` (required) — `"epub"` или `"fb2"`
- `author` (optional) — автор для метаданных (по умолчанию: "Переведено Arcane")

**Response:** `200 OK`

```json
{
  "success": true,
  "format": "epub",
  "filename": "My_Novel.epub",
  "url": "/exports/My_Novel.epub",
  "path": "My_Novel.epub"
}
```

**Особенности:**

- Экспортируются только главы со статусом `completed`
- Главы сортируются по номеру (`number`)
- Текст преобразуется в HTML (EPUB) или XML (FB2)
- Файлы сохраняются в `data/exports/` и доступны по URL `/exports/{filename}`

**Ошибки:**

- `400` — Неверный формат (должен быть "epub" или "fb2")
- `404` — Проект не найден
- `500` — Нет переведенных глав для экспорта

---

## 🔧 Error Responses

Все ошибки возвращаются в формате:

```json
{
  "error": "Error message description"
}
```

### HTTP Status Codes

| Code | Meaning               |
| ---- | --------------------- |
| 200  | OK                    |
| 201  | Created               |
| 400  | Bad Request           |
| 404  | Not Found             |
| 500  | Internal Server Error |

---

## 📝 Примеры с curl

### Создать проект

```bash
curl -X POST http://localhost:3000/api/projects \
  -H "Content-Type: application/json" \
  -d '{"name": "Test Novel"}'
```

### Добавить главу

```bash
curl -X POST http://localhost:3000/api/projects/{id}/chapters \
  -H "Content-Type: application/json" \
  -d '{"title": "Chapter 1", "originalText": "Hello world..."}'
```

### Запустить перевод

```bash
curl -X POST http://localhost:3000/api/projects/{projectId}/chapters/{chapterId}/translate
```

### Добавить персонажа в глоссарий

```bash
curl -X POST http://localhost:3000/api/projects/{id}/glossary \
  -H "Content-Type: application/json" \
  -d '{"type": "character", "original": "John", "gender": "male"}'
```

### Экспортировать проект в EPUB

```bash
curl -X POST http://localhost:3000/api/projects/{id}/export \
  -H "Content-Type: application/json" \
  -d '{"format": "epub", "author": "Переведено Arcane"}'
```

### Экспортировать проект в FB2

```bash
curl -X POST http://localhost:3000/api/projects/{id}/export \
  -H "Content-Type: application/json" \
  -d '{"format": "fb2"}'
```
