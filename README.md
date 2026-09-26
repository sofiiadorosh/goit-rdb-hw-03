# goit-rdb-hw-03 — Завантаження даних та основи SQL. DQL-команди

Домашнє завдання 2 (Тема 3) курсу реляційних БД.

**Датасет:** варіант A — [NYC TLC Yellow Taxi Trip Records](https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page),
січень 2024, випадкова вибірка 100 000 поїздок (`random_state=42`).

## Вміст

- [`hw3_nyc_taxi.ipynb`](hw3_nyc_taxi.ipynb) — notebook з усіма кроками та збереженими результатами:
  1. setup pgserver + отримання даних з офіційного CDN TLC (Parquet → контрольований CSV);
  2. staging-таблиця `hw3_taxi_staging` (усі колонки `TEXT`) + `COPY FROM STDIN`;
  3. типізована таблиця `hw3_taxi_trips` (`TIMESTAMPTZ`, `DATE`, `NUMERIC(10,2)`, `BOOLEAN`, CHECK-обмеження) і очищення через `INSERT INTO ... SELECT`;
  4. data-quality audit (row count, дублікати, null-rate, спецмаркери);
  5. розподіли, числова статистика, доменний outlier-check;
  6. 10 DQL/EDA-запитів «business-question → SQL → interpretation»;
  7. Reflection.

## Відтворення

**Google Colab:** відкрити notebook і виконати `Runtime → Restart and run all`. Дані завантажуються автоматично:

- `https://d37ci6vzurychx.cloudfront.net/trip-data/yellow_tripdata_2024-01.parquet`
- `https://d37ci6vzurychx.cloudfront.net/misc/taxi_zone_lookup.csv`

**Локально** (Python 3.10–3.12; для 3.13+ у `pgserver` поки немає wheel-файлів):

```bash
python3.12 -m venv .venv && source .venv/bin/activate
pip install pgserver psycopg2-binary sqlalchemy pandas pyarrow jupyter
jupyter nbconvert --to notebook --execute --inplace hw3_nyc_taxi.ipynb
```

Локально файли зберігаються у `./data/` (у git не комітяться), у Colab — у `/content/`.
