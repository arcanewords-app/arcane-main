# 📖 Arcane Reader

Web-интерфейс и REST API сервер для переводчика новелл.

**Стек клиента:** Preact + Vite + TypeScript

---

## 🚀 Запуск

### Development (раздельные серверы)

```bash
cd arcane-reader

# Запустить оба сервера параллельно
npm run dev

# Или по отдельности:
npm run dev:server  # Express API на :3000
npm run dev:client  # Vite HMR на :5173
```

### Production

```bash
cd arcane-reader

# Собрать клиент
npm run build:client

# Запустить сервер (раздаёт статику из dist/client)
npm run start
```

Приложение будет доступно на `http://localhost:3000`

---

## 📜 NPM Scripts

| Скрипт | Описание |
|--------|----------|
| `npm run dev` | Запуск dev сервера (API + Vite) |
| `npm run dev:server` | Только Express API |
| `npm run dev:client` | Только Vite HMR |
| `npm run build` | Полная сборка (клиент + сервер) |
| `npm run build:client` | Сборка клиента в `dist/client` |
| `npm run build:server` | Компиляция TypeScript сервера |
| `npm run start` | Запуск production сервера |
| `npm run kill-port` | Освободить порт 3000 |

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
│   ├── index.html          # Entry point для Vite
│   └── arcane_icon.png     # Иконка приложения
│
├── src/
│   ├── server.ts           # Express API сервер
│   ├── config.ts           # Загрузка конфигурации
│   │
│   ├── services/
│   │   ├── engine-integration.ts   # Интеграция с arcane-engine
│   │   └── translation-service.ts  # Сервис перевода
│   │
│   ├── storage/
│   │   └── database.ts     # LowDB операции
│   │
│   └── client/             # ⚡ Preact SPA
│       ├── main.tsx        # Точка входа
│       ├── App.tsx         # Главный компонент
│       ├── vite-env.d.ts   # Vite типы
│       │
│       ├── api/
│       │   └── client.ts   # Типизированный API клиент
│       │
│       ├── types/
│       │   └── index.ts    # TypeScript интерфейсы
│       │
│       ├── styles/
│       │   └── index.css   # Все стили
│       │
│       └── components/
│           ├── ui/         # Базовые компоненты
│           │   ├── Button.tsx
│           │   ├── Card.tsx
│           │   ├── Modal.tsx
│           │   ├── Input.tsx
│           │   ├── Badge.tsx
│           │   └── index.ts
│           │
│           ├── Header.tsx
│           ├── ProjectInfo.tsx
│           │
│           ├── Sidebar/
│           │   ├── index.tsx
│           │   ├── ProjectList.tsx
│           │   └── ChapterList.tsx
│           │
│           ├── ChapterView/
│           │   ├── index.tsx
│           │   ├── ChapterHeader.tsx
│           │   ├── ReaderSettings.tsx
│           │   └── ParagraphList.tsx
│           │
│           ├── Glossary/
│           │   ├── index.ts
│           │   └── GlossaryModal.tsx
│           │
│           └── ReadingMode/
│               └── index.tsx
│
├── dist/
│   └── client/             # Production билд (генерируется)
│
├── data/
│   ├── arcane-db.json      # База данных (создаётся автоматически)
│   ├── images/             # Изображения для глоссария
│   └── exports/             # Экспортированные файлы (EPUB/FB2)
│
├── vite.config.ts          # Конфигурация Vite
├── tsconfig.client.json    # TypeScript для клиента
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

### Архитектура

Клиент построен на **Preact** — легковесной альтернативе React (3KB gzip):

- **Preact + Hooks** — компонентный подход с хуками
- **Vite** — быстрая сборка и HMR
- **TypeScript** — полная типизация
- **CSS Variables** — темизация через переменные

### Компоненты

| Компонент | Описание |
|-----------|----------|
| `Header` | Логотип, статус API |
| `Sidebar` | Список проектов и глав |
| `ProjectList` | Выбор/создание проектов |
| `ChapterList` | Главы с фильтрами, drag-n-drop загрузка |
| `ProjectInfo` | Настройки проекта, экспорт EPUB/FB2, массовый перевод |
| `ChapterView` | Просмотр/редактирование главы |
| `ParagraphList` | Параллельный текст (оригинал/перевод) |
| `GlossaryModal` | Управление глоссарием |
| `ReadingMode` | Полноэкранный режим чтения переведенных глав |

### UI Kit

Базовые компоненты в `components/ui/`:

```tsx
import { Button, Card, Modal, Input, Badge, Select } from './components/ui';

<Button variant="primary" loading={isLoading}>Перевести</Button>
<Card title="📁 Проекты">...</Card>
<Modal isOpen={show} onClose={close} title="Заголовок">...</Modal>
```

### Особенности

- 🇷🇺 Интерфейс на русском языке
- 🎨 Тёмная тема с градиентами
- ⚡ Мгновенные обновления (HMR)
- 📱 Адаптивный дизайн
- ✨ Анимации и переходы

### API Клиент

Типизированный клиент в `api/client.ts`:

```typescript
import { api } from './api/client';

// Проекты
const projects = await api.getProjects();
const project = await api.getProject(id);
await api.createProject(name);
await api.deleteProject(id);

// Главы
await api.uploadChapter(projectId, file, title);
await api.translateChapter(projectId, chapterId);
await api.deleteChapter(projectId, chapterId);

// Глоссарий
await api.addGlossary(projectId, entry);
await api.updateGlossaryEntry(projectId, entryId, data);
await api.deleteGlossaryEntry(projectId, entryId);

// Экспорт
await api.exportProject(projectId, 'epub');
await api.exportProject(projectId, 'fb2');
```

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

---

## ⚡ Vite Конфигурация

Файл `vite.config.ts`:

```typescript
import { defineConfig } from 'vite';
import preact from '@preact/preset-vite';

export default defineConfig({
  plugins: [preact()],
  server: {
    proxy: {
      '/api': 'http://localhost:3000',
      '/images': 'http://localhost:3000',
    },
  },
  build: {
    outDir: 'dist/client',
    emptyOutDir: true,
  },
});
```

### Proxy

В dev режиме Vite проксирует API запросы на Express сервер:
- `/api/*` → `http://localhost:3000`
- `/images/*` → `http://localhost:3000`

### Production Build

```bash
npm run build:client
# Результат: dist/client/
#   ├── index.html
#   └── assets/
#       ├── index-*.css  (~5 KB gzip)
#       └── index-*.js   (~15 KB gzip)
```

Express сервер автоматически раздаёт статику из `dist/client/` в production режиме.

---

## 📤 Экспорт в EPUB/FB2

Система поддерживает экспорт переведенных проектов в форматы электронных книг.

### Поддерживаемые форматы

- **EPUB** — стандартный формат для большинства читалок
- **FB2** — популярный формат в России (FictionBook 2.0)

### Использование

1. Откройте проект с переведенными главами
2. На странице проекта нажмите кнопку "📚 Экспорт EPUB" или "📖 Экспорт FB2"
3. Файл автоматически скачается в браузере

### Особенности

- Экспортируются только главы со статусом `completed`
- Главы сортируются по номеру (`number`)
- Текст автоматически форматируется (параграфы, заголовки)
- Метаданные включают название проекта, автора, дату перевода

### API

```typescript
// Экспорт в EPUB
const result = await api.exportProject(projectId, 'epub');
// result.url - URL для скачивания файла

// Экспорт в FB2
const result = await api.exportProject(projectId, 'fb2');
```

### Расположение файлов

Экспортированные файлы сохраняются в:
```
arcane-reader/data/exports/
```

И доступны по URL:
```
http://localhost:3000/exports/{filename}
```

---

## 📖 Режим чтения

Полноэкранный интерфейс для комфортного чтения переведенных глав.

### Функции

- **Навигация** — переход между главами (клавиши ← →)
- **Оглавление** — быстрый переход к любой главе
- **Настройки** — шрифт, размер, межстрочный интервал, тема
- **Поделиться** — генерация ссылки на текущую главу
- **URL параметры** — поддержка прямых ссылок на главы

### Использование

1. Откройте проект с переведенными главами
2. Нажмите "📖 Режим чтения" на странице проекта
3. Или нажмите "📖 Читать" в просмотре главы

### URL параметры

```
# Открыть проект
?project={projectId}

# Открыть конкретную главу
?project={projectId}&chapter={chapterId}

# Открыть в режиме чтения
?project={projectId}&reading=true

# Открыть конкретную главу в режиме чтения
?project={projectId}&chapter={chapterId}&reading=true
```

### Настройки чтения

- **Шрифт** — выбор семейства шрифтов
- **Размер** — размер текста (12-24px)
- **Межстрочный интервал** — расстояние между строками
- **Тема** — темная, светлая, сепия, контрастная
