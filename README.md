# Ontario Fish (с авторизацией и добавлением спотов)

Готовый проект: Next.js (App Router) + Tailwind + Supabase + Leaflet.

## Что есть
- Вход по email (magic link): `/auth`
- Карта спотов: `/spots`
- Добавление спота (для вошедших): `/spots/new`
- Справочник снастей: `/gear`
- Калькулятор веса грузика: `/calculator`

## Быстрый старт
1. `npm install`
2. Создай `.env.local` из `.env.example` и вставь ключи Supabase.
3. В Supabase выполни SQL из раздела ниже (схема + RLS политики).
4. `npm run dev` → открой `http://localhost:3000`

## SQL (Supabase → SQL Editor)
```sql
create extension if not exists pgcrypto;

-- пользователи (минимально, почта хранится в JWT профиле)
create table if not exists profiles (
  id uuid primary key default gen_random_uuid(),
  email text unique,
  display_name text,
  created_at timestamptz default now()
);

create table if not exists spots (
  id uuid primary key default gen_random_uuid(),
  name text not null,
  lat double precision not null,
  lng double precision not null,
  access_type text check (access_type in ('shore','boat')) default 'shore',
  fish_types text[] default '{}',
  depth_m numeric,
  cover text,
  notes text,
  rating numeric check (rating between 0 and 5),
  created_by uuid default auth.uid(),
  created_at timestamptz default now()
);

-- Тестовые данные
insert into spots (name, lat, lng, access_type, fish_types, notes, rating)
values
('Humber River – Old Mill', 43.65065, -79.49367, 'shore', '{trout,salmon,bass}', 'Утро/вечер, офсет+силикон', 4.2),
('Ontario Place West Channel', 43.62957, -79.41915, 'shore', '{pike,bass}', 'Южный ветер ок', 4.0);

-- Включаем RLS и политики
alter table spots enable row level security;

-- читать всем (публичная карта)
create policy "Read spots for all" on spots
for select using (true);

-- добавлять только аутентифицированным
create policy "Insert spots for authenticated" on spots
for insert with check (auth.role() = 'authenticated');

-- обновление/удаление только автору (опционально)
create policy "Update own spots" on spots
for update using (created_by = auth.uid());

create policy "Delete own spots" on spots
for delete using (created_by = auth.uid());
```

## Настройка Redirect URL
В Supabase → Authentication → URL Configuration:
- Добавь **Redirect URLs**: `http://localhost:3000/auth` и адрес продакшена (когда задеплоишь).

## Примечания
- Если иконки маркеров на карте не видны, установи `leaflet-defaulticon-compatibility` и импортируй в проект.
- Email‑вход: письмо со ссылкой придёт от Supabase; после клика вернёт на `/auth`.
```

## Страницы
- `/` — главная
- `/auth` — вход/выход
- `/spots` — карта
- `/spots/new` — форма добавления (вход обязателен)
- `/gear`, `/calculator`
