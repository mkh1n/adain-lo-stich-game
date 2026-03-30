# Инструкция по настройке синхронизации для Vercel + Supabase

## 🚀 Быстрый старт

Этот проект использует **Supabase** (бесплатная альтернатива Firebase) для синхронизации состояния между всеми пользователями в реальном времени.

---

## Шаг 1: Создайте проект в Supabase

1. Перейдите на https://supabase.com
2. Нажмите **"Start your project"** или **"New Project"**
3. Зарегистрируйтесь через GitHub (рекомендуется) или email
4. Создайте новый проект:
   - Выберите организацию (или создайте новую)
   - Придумайте название проекта
   - Установите пароль для базы данных (запомните его!)
   - Выберите регион (ближайший к вашей аудитории)
5. Дождитесь создания проекта (~2 минуты)

---

## Шаг 2: Получите ключи доступа

1. В левом меню перейдите в **Settings** (шестерёнка) → **API**
2. Скопируйте два значения:
   - **Project URL** (выглядит как `https://xxxxx.supabase.co`)
   - **anon/public key** (длинная строка, начинающаяся с `eyJ...`)

---

## Шаг 3: Настройте таблицу в базе данных

1. В левом меню перейдите в **SQL Editor**
2. Нажмите **"New query"**
3. Вставьте следующий SQL-код и нажмите **Run**:

```sql
-- Создаём таблицу для хранения позиций персонажей
CREATE TABLE game_positions (
  id INTEGER PRIMARY KEY CHECK (id >= 0 AND id <= 3),
  x NUMERIC NOT NULL DEFAULT 40,
  y NUMERIC NOT NULL DEFAULT 40,
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- Вставляем начальные позиции для 4 персонажей
INSERT INTO game_positions (id, x, y) VALUES 
  (0, 40, 40),
  (1, 1160, 40),
  (2, 40, 660),
  (3, 1160, 660)
ON CONFLICT (id) DO NOTHING;

-- Включаем Realtime для таблицы (обязательно для синхронизации!)
ALTER PUBLICATION supabase_realtime ADD TABLE game_positions;
```

4. Убедитесь, что все 3 запроса выполнены успешно (зелёные галочки)

---

## Шаг 4: Настройте политики доступа (RLS)

По умолчанию Supabase блокирует прямой доступ к таблице. Нужно разрешить чтение и запись:

1. В SQL Editor вставьте и выполните:

```sql
-- Отключаем RLS для простоты (для публичного проекта)
ALTER TABLE game_positions DISABLE ROW LEVEL SECURITY;

-- ИЛИ включаем RLS с политиками для всех:
-- ALTER TABLE game_positions ENABLE ROW LEVEL SECURITY;
-- CREATE POLICY "Allow all access" ON game_positions FOR ALL USING (true) WITH CHECK (true);
```

---

## Шаг 5: Вставьте ключи в код

1. Откройте файл `index.html`
2. Найдите строки 176-177:
   ```javascript
   const SUPABASE_URL = 'YOUR_SUPABASE_URL';
   const SUPABASE_ANON_KEY = 'YOUR_SUPABASE_ANON_KEY';
   ```
3. Замените на ваши значения из Шага 2:
   ```javascript
   const SUPABASE_URL = 'https://abcdefgh.supabase.co';
   const SUPABASE_ANON_KEY = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...';
   ```

---

## Шаг 6: Разместите на Vercel

### Вариант A: Через GitHub (рекомендуется)

1. Создайте репозиторий на GitHub и запушьте код:
   ```bash
   git init
   git add .
   git commit -m "Initial commit"
   git branch -M main
   git remote add origin https://github.com/USERNAME/REPO.git
   git push -u origin main
   ```

2. Перейдите на https://vercel.com
3. Нажмите **"Add New..."** → **"Project"**
4. Импортируйте ваш GitHub репозиторий
5. Нажмите **"Deploy"**
6. Готово! Ваш сайт доступен по ссылке вида `https://your-project.vercel.app`

### Вариант B: Через Vercel CLI

```bash
npm install -g vercel
cd /workspace
vercel login
vercel --prod
```

---

## ✅ Проверка работы

1. Откройте ваш сайт на Vercel в **двух разных браузерах** (или в режиме инкогнито)
2. Перетащите кружок в одном браузере
3. Во втором браузере кружок должен переместиться **автоматически**!

---

## 🔧 Решение проблем

### Кружки не синхронизируются?

1. Откройте консоль разработчика (F12) на обеих вкладках
2. Проверьте, нет ли ошибок красным цветом
3. Убедитесь, что:
   - Ключи Supabase вставлены правильно (без лишних пробелов)
   - Таблица `game_positions` создана
   - Realtime включён (`ALTER PUBLICATION...` выполнен)

### Ошибка CORS?

Supabase по умолчанию разрешает запросы с любых доменов. Если есть проблема:

1. В Dashboard Supabase перейдите в **Settings** → **API**
2. В разделе **Allowed HTTP Referrers** добавьте ваш домен Vercel

### Изменения не сохраняются после перезагрузки?

Проверьте, что данные есть в базе:

1. В Supabase Dashboard перейдите в **Table Editor**
2. Откройте таблицу `game_positions`
3. Вы должны видеть 4 строки с актуальными координатами

---

## 💡 Как это работает?

1. При загрузке страницы код загружает позиции из Supabase
2. При перетаскивании кружка новая позиция отправляется в базу
3. Supabase Realtime рассылает изменения всем подключённым клиентам
4. Другие браузеры получают обновление и перерисовывают кружки

---

## 📚 Дополнительные ресурсы

- [Документация Supabase](https://supabase.com/docs)
- [Документация Vercel](https://vercel.com/docs)
- [Supabase JS Client](https://supabase.com/docs/reference/javascript/introduction)

---

## 🎉 Готово!

Теперь у вас есть работающий мультиплеерный проект на бесплатном хостинге!
