# goit-rdb-hw-03 — Завантаження даних та основи SQL. DQL-команди

Домашнє завдання 2 (Тема 3) курсу реляційних БД: відтворюваний шлях від публічного датасету
до типізованої таблиці PostgreSQL, аудиту якості даних та EDA на SQL.

## Джерело даних

**Варіант A — NYC TLC Yellow Taxi Trip Records**, офіційна сторінка
[TLC Trip Record Data](https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page)
(2024 → January → Yellow Taxi Trip Records).

| Файл | Посилання |
|---|---|
| Поїздки, січень 2024 (Parquet, ≈50 МБ, 2 964 624 рядки) | https://d37ci6vzurychx.cloudfront.net/trip-data/yellow_tripdata_2024-01.parquet |
| Довідник зон (CSV, 265 рядків) | https://d37ci6vzurychx.cloudfront.net/misc/taxi_zone_lookup.csv |
| Data dictionary | https://www.nyc.gov/assets/tlc/downloads/pdf/data_dictionary_trip_records_yellow.pdf |

## Вибірка

- **100 000 рядків** — випадкова вибірка з повного місяця: `df.sample(n=100_000, random_state=42)`.
  Випадкова, а не `head()`, бо файл упорядкований за часом і `head()` охопив би лише перші 1–2 дні.
- Усі **19 колонок** оригінального файлу (потрібні для звірки `total_amount` із сумою складових).
- pandas використовується лише для Parquet → CSV; COPY, очищення, аудит і EDA виконуються SQL-запитами в PostgreSQL.

## Структура репозиторію

```
.
├── hw3_nyc_taxi.ipynb        # notebook з output усіх клітинок
├── README.md
└── data/
    ├── hw3_taxi_sample.csv   # контрольований CSV (100 000 рядків, ≈10 МБ), що завантажується через COPY
    └── taxi_zone_lookup.csv  # офіційний довідник зон TLC
```

Notebook сам завантажує Parquet з офіційного CDN і відтворює той самий `hw3_taxi_sample.csv`
(фіксований `random_state`), тому файли в `data/` потрібні лише для перевірки без запуску.

## Вміст notebook

1. **Setup і завантаження** — запуск PostgreSQL, отримання даних, staging-таблиця `hw3_taxi_staging` (усі колонки `TEXT`), `COPY FROM STDIN`.
2. **Staging і DDL** — типізована таблиця `hw3_taxi_trips` (`TIMESTAMPTZ` з локалізацією `America/New_York`, згенерована `DATE`, `NUMERIC(10,2)`, `BOOLEAN`, identity PK, CHECK-обмеження), очищення через `INSERT INTO ... SELECT`, обґрунтування типів.
3. **Data-quality audit** — контроль кількості рядків, дублікати за business key, null-rate і спецмаркери (`RatecodeID = 99`, `payment_type = 0`, `passenger_count = 0`, зони 264/265).
4. **Розподіли та аномалії** — категорії, числова статистика, доменний outlier-check.
5. **10 DQL/EDA-запитів** у форматі «business-question → SQL → interpretation».
6. **Reflection**.

## Запуск

### Google Colab

Відкрити `hw3_nyc_taxi.ipynb` → `Runtime → Restart and run all`.

Перша клітинка пробує `pgserver`. На актуальному Python у Colab для `pgserver` немає збірок
(`No matching distribution found`), тому клітинка автоматично піднімає системний PostgreSQL через
`apt-get install postgresql` (як у ДЗ 02) і підключається до `postgresql://postgres:postgres@127.0.0.1:5432/postgres`.

### Локально

Потрібен Python 3.10–3.12 (для 3.13+ у `pgserver` поки немає збірок):

```bash
python3.12 -m venv .venv && source .venv/bin/activate
pip install pgserver psycopg2-binary sqlalchemy pandas pyarrow jupyter
jupyter nbconvert --to notebook --execute --inplace hw3_nyc_taxi.ipynb
```

Локально дані зберігаються у `./data/`, у Colab — у `/content/`.
