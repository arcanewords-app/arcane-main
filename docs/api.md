# 🔗 REST API Reference

Arcane Reader provides a REST API for managing projects and translations.

**Base URL:** `http://localhost:3000`

---

## 📊 Status

### GET /api/status

Check server and AI provider status.

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

Get list of all projects.

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

Create a new project.

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

Get project by ID.

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

Delete a project.

**Response:** `200 OK`

```json
{
  "success": true
}
```

---

## 📖 Chapters

### POST /api/projects/:id/chapters

Add a chapter to the project.

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

Upload a chapter from a file (multipart/form-data).

**Form fields:**

- `file` — .txt file with chapter text
- `title` (optional) — chapter title

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

Get a chapter.

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

Start chapter translation.

**Response:** `200 OK`

```json
{
  "status": "started",
  "chapterId": "ch_001"
}
```

Translation runs asynchronously. Check chapter status via GET.

**Statuses:**

- `pending` — awaiting translation
- `translating` — in progress
- `completed` — finished
- `error` — error occurred

---

## 📚 Glossary

### GET /api/projects/:id/glossary

Get project glossary.

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
    "notes": "Category: magic",
    "firstAppearance": 1
  }
]
```

---

### POST /api/projects/:id/glossary

Add an entry to the glossary.

**Request:**

```json
{
  "type": "character",
  "original": "Alexander",
  "translated": "Александр",
  "gender": "male",
  "description": "Main protagonist, young mage-researcher",
  "notes": "Additional notes for verification",
  "firstAppearance": 1
}
```

> **Auto-declensions**: If `type: "character"` and `declensions` are not specified,
> the system will automatically generate case forms.

**Response:** `201 Created`

```json
{
  "id": "gl_004",
  "type": "character",
  "original": "Alexander",
  "translated": "Александр",
  "gender": "male",
  "description": "Main protagonist, young mage-researcher",
  "notes": "Additional notes for verification",
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

Update a glossary entry.

**Request:**

```json
{
  "type": "character",
  "original": "Alexander",
  "translated": "Александр",
  "gender": "male",
  "description": "Main protagonist, young mage-researcher",
  "notes": "Additional notes",
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
  "description": "Main protagonist, young mage-researcher",
  "notes": "Additional notes",
  "firstAppearance": 1,
  "declensions": { ... }
}
```

> **Note**: If `type: "character"` and `declensions` are not specified, the system will automatically regenerate declensions.

---

### DELETE /api/projects/:projectId/glossary/:entryId

Delete a glossary entry.

**Response:** `200 OK`

```json
{
  "success": true
}
```

---

### POST /api/projects/:projectId/glossary/:entryId/image

Add an image to the glossary entry gallery.

**Request:** `multipart/form-data`

- `image` (file, required) — image file

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

Delete a specific image from the gallery by index.

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

Delete all images from the entry gallery.

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

Export project to EPUB or FB2 format.

**Request:**

```json
{
  "format": "epub",
  "author": "Translated by Arcane"
}
```

**Parameters:**

- `format` (required) — `"epub"` or `"fb2"`
- `author` (optional) — author for metadata (default: "Translated by Arcane")

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

**Features:**

- Only chapters with `completed` status are exported
- Chapters are sorted by number (`number`)
- Text is converted to HTML (EPUB) or XML (FB2)
- Files are saved in `data/exports/` and accessible via URL `/exports/{filename}`

**Errors:**

- `400` — Invalid format (must be "epub" or "fb2")
- `404` — Project not found
- `500` — No translated chapters to export

---

## 🔧 Error Responses

All errors are returned in the format:

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

## 📝 curl Examples

### Create project

```bash
curl -X POST http://localhost:3000/api/projects \
  -H "Content-Type: application/json" \
  -d '{"name": "Test Novel"}'
```

### Add chapter

```bash
curl -X POST http://localhost:3000/api/projects/{id}/chapters \
  -H "Content-Type: application/json" \
  -d '{"title": "Chapter 1", "originalText": "Hello world..."}'
```

### Start translation

```bash
curl -X POST http://localhost:3000/api/projects/{projectId}/chapters/{chapterId}/translate
```

### Add character to glossary

```bash
curl -X POST http://localhost:3000/api/projects/{id}/glossary \
  -H "Content-Type: application/json" \
  -d '{"type": "character", "original": "John", "gender": "male"}'
```

### Export project to EPUB

```bash
curl -X POST http://localhost:3000/api/projects/{id}/export \
  -H "Content-Type: application/json" \
  -d '{"format": "epub", "author": "Translated by Arcane"}'
```

### Export project to FB2

```bash
curl -X POST http://localhost:3000/api/projects/{id}/export \
  -H "Content-Type: application/json" \
  -d '{"format": "fb2"}'
```
