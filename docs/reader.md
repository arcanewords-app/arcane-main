# 📖 Arcane Reader

Web-интерфейс и REST API сервер для переводчика новелл.

---

## 🚀 Запуск

```bash
# Из корня монорепы
npm run dev

# Или напрямую
cd arcane-reader
npm run dev
```

Сервер запустится на `http://localhost:3000`

---

## ⚙️ Конфигурация

Создайте файл `.env` в `arcane-reader/`:

```env
# Обязательно
OPENAI_API_KEY=sk-your-key-here

# Опционально
OPENAI_MODEL=gpt-4-turbo-preview
PORT=3000
MAX_TOKENS_PER_CHUNK=2000
TRANSLATION_TEMPERATURE=0.7
SKIP_EDITING=false
```

### Переменные окружения

| Переменная | По умолчанию | Описание |
|------------|--------------|----------|
| `OPENAI_API_KEY` | — | API ключ OpenAI (обязателен для перевода) |
| `OPENAI_MODEL` | `gpt-4-turbo-preview` | Модель для перевода |
| `PORT` | `3000` | Порт сервера |
| `MAX_TOKENS_PER_CHUNK` | `2000` | Макс. токенов на чанк |
| `TRANSLATION_TEMPERATURE` | `0.7` | Температура генерации |
| `SKIP_EDITING` | `false` | Пропускать стадию редактуры |

---

## 📂 Структура

```
arcane-reader/
├── public/
│   ├── index.html          # Главная страница
│   └── arcane_icon.png     # Иконка приложения
│
├── src/
│   ├── server.ts           # Express сервер
│   ├── config.ts           # Загрузка конфигурации
│   │
│   ├── services/
│   │   └── engine-integration.ts   # Интеграция с arcane-engine
│   │
│   └── storage/
│       └── database.ts     # LowDB операции
│
├── data/
│   └── arcane-db.json      # База данных (создаётся автоматически)
│
├── .env                    # Конфигурация (не в git)
└── env.example.txt         # Пример конфигурации
```

---

## 💾 База данных

Используется **LowDB** — файловая JSON база данных.

### Расположение

```
arcane-reader/data/arcane-db.json
```

### Схема

```typescript
interface DatabaseSchema {
  projects: Project[];
  settings: {
    lastOpenedProject?: string;
  };
}

interface Project {
  id: string;
  name: string;
  sourceLanguage: string;  // 'en'
  targetLanguage: string;  // 'ru'
  chapters: Chapter[];
  glossary: GlossaryEntry[];
  settings: ProjectSettings;
  createdAt: string;
  updatedAt: string;
}

interface Chapter {
  id: string;
  number: number;
  title: string;
  originalText: string;
  translatedText?: string;
  status: 'pending' | 'translating' | 'completed' | 'error';
  translationMeta?: {
    tokensUsed: number;
    duration: number;
    model: string;
    translatedAt: string;
  };
}

interface GlossaryEntry {
  id: string;
  type: 'character' | 'location' | 'term';
  original: string;
  translated: string;
  gender?: 'male' | 'female' | 'neutral';
  declensions?: Declensions;
  notes?: string;
  autoDetected?: boolean;
}
```

---

## 🔗 Интеграция с Engine

Файл `src/services/engine-integration.ts` связывает Reader с Engine:

### Функции

```typescript
// Получить агента для проекта
getAgentForProject(project: Project): NovelAgent

// Создать пайплайн перевода
createPipeline(config: AppConfig, project: Project): TranslationPipeline

// Перевести главу через полный пайплайн
translateChapterWithPipeline(
  config: AppConfig,
  project: Project,
  chapter: Chapter,
  options?: PipelineOptions
): Promise<{
  translatedText: string;
  tokensUsed: number;
  duration: number;
  glossaryUpdates?: GlossaryEntry[];
}>

// Простой перевод (без пайплайна, для коротких текстов)
translateSimple(
  config: AppConfig,
  text: string,
  glossary: GlossaryEntry[]
): Promise<{ text: string; tokensUsed: number }>

// Авто-склонения для имён
getNameDeclensions(
  englishName: string,
  gender?: 'male' | 'female' | 'neutral' | 'unknown'
): {
  translatedName: string;
  declensions: Declensions;
  gender: string;
}

// Очистить кэш агента
clearAgentCache(projectId: string): void
```

---

## 🖥️ UI Интерфейс

### Главная страница

- Список проектов
- Создание нового проекта
- Статус подключения к AI

### Страница проекта

- Список глав с прогрессом
- Drag-n-drop загрузка .txt файлов
- Глоссарий с редактированием
- Предпросмотр переводов

### Особенности UI

- Интерфейс на русском языке
- Логотип Arcane с анимацией
- Индикатор статуса OpenAI API
- Консольный лог переводов

---

## 📊 Логирование

При переводе выводится подробный лог:

```
════════════════════════════════════════════════════════════
🔮 ЗАПРОС НА ПЕРЕВОД
────────────────────────────────────────────────────────────
📖 Глава: Chapter 1
📊 Размер: 5420 символов, ~980 слов
🔑 API ключ: ✅ Настроен
🤖 Модель: gpt-4-turbo-preview
────────────────────────────────────────────────────────────
🚀 Запуск arcane-engine TranslationPipeline...
   Этапы: ✅ Анализ | ✅ Перевод | ✅ Редактура
────────────────────────────────────────────────────────────
✅ ПЕРЕВОД ЗАВЕРШЁН (arcane-engine)
⏱️  Время: 12.3s
📝 Токенов: 2840
════════════════════════════════════════════════════════════
```

---

## 🔒 Безопасность

### .gitignore

```gitignore
.env
node_modules/
dist/
data/arcane-db.json
```

### Важно

- **Никогда** не коммитьте `.env` с API ключами
- База данных `.json` также в gitignore
- Используйте `env.example.txt` как шаблон

