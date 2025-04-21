## 1. Создание таблицы с продажами

```
CREATE TABLE sales
(
    id SERIAL,
    dt_sale DATE,
    qty INT
);

INSERT INTO sales
(
    dt_sale,
    qty
)
SELECT '2025-01-01'::DATE + make_interval(days := (random()*365)::int),
       (random() * 100)::INT
FROM generate_series(1,100);
```

## 2. Функция получения трети года через case
```
CREATE OR REPLACE FUNCTION third_part_case(dt DATE) RETURNS INT
IMMUTABLE
LANGUAGE plpgsql
AS $$
BEGIN
    RETURN CASE
               WHEN date_part('month', dt) BETWEEN 1 AND 4
                   THEN 1
               WHEN date_part('month', dt) BETWEEN 5 AND 8
                   THEN 2
               WHEN date_part('month', dt) BETWEEN 9 AND 12
                   THEN 3
           END;
END;
$$;
```

## 3. Функция получения трети года через математическую операцию
```
CREATE OR REPLACE FUNCTION third_part_math_1(dt DATE) RETURNS INT
IMMUTABLE
LANGUAGE plpgsql
AS $$
BEGIN
    RETURN ceil(date_part('month', dt)::int / 4.0);
END;
$$;
```

## 4. Вывод результата
```
SELECT s.id,
       s.dt_sale,
       s.qty,
       third_part_case(s.dt_sale),
       third_part_math_1(s.dt_sale)
FROM sales s;
```
| id | dt\_sale | qty | third\_part\_case | third\_part\_math\_1 |
| :--- | :--- | :--- | :--- | :--- |
| 1 | 2025-10-31 | 98 | 3 | 3 |
| 2 | 2025-06-17 | 60 | 2 | 2 |
| 3 | 2025-02-06 | 60 | 1 | 1 |
| 4 | 2025-01-15 | 78 | 1 | 1 |
| 5 | 2025-04-22 | 23 | 1 | 1 |
| 6 | 2026-01-01 | 68 | 1 | 1 |
| 7 | 2025-01-09 | 93 | 1 | 1 |
| 8 | 2025-04-06 | 69 | 1 | 1 |
| 9 | 2025-04-27 | 3 | 1 | 1 |
| 10 | 2025-04-10 | 20 | 1 | 1 |
| 11 | 2025-10-10 | 14 | 3 | 3 |

## 5. Что значит предусмотреть NULL на входе не понял, так как если на входе NULL, то и в результате NULL

| id | dt\_sale | qty | third\_part\_case | third\_part\_math\_1 |
| :--- | :--- | :--- | :--- | :--- |
| 30 | null | 98 | null | null |
| 28 | null | 11 | null | null |

### Возможно имелось ввиду, что если на вход NULL, то на выходе выдавать первый квартал. Тогда вот так

```
CREATE OR REPLACE FUNCTION third_part_math_null(dt DATE) RETURNS INT
IMMUTABLE
LANGUAGE plpgsql
AS $$
BEGIN
    RETURN coalesce(ceil(date_part('month', dt)::int / 4.0), 1);
END;
$$;
```