##  Соединение таблиц
1. Вывести название, жанр и цену тех книг, количество которых больше 8, 
в отсортированном по убыванию цены виде.
```sql
SELECT title, name_genre, price
FROM
    genre INNER JOIN book b on genre.genre_id = b.genre_id
WHERE amount > 8
ORDER BY price DESC;
```
2. Вывести все жанры, которые не представлены в книгах на складе.
```sql
SELECT name_genre
FROM genre
    LEFT JOIN book b on genre.genre_id = b.genre_id
WHERE b.genre_id IS NULL;

```
3. Необходимо в каждом городе провести выставку книг каждого автора в течение 2020 года. 
Дату проведения выставки выбрать случайным образом. Создать запрос, который выведет город, 
автора и дату проведения выставки. Последний столбец назвать Дата. Информацию вывести, 
отсортировав сначала в алфавитном порядке по названиям городов, а потом по убыванию дат проведения выставок.
```sql
CREATE TABLE city
    (
        city_id serial PRIMARY KEY,
        name_city varchar(50)
    );

INSERT INTO city (name_city)
VALUES ('Москва'),
       ('Санкт-Петербург'),
       ('Владивосток');
```
```sql
SELECT name_city, name_author, (DATE_ADD('2020-01-01', INTERVAL FLOOR(RAND() * 365) DAY)) AS Дата
FROM
    author CROSS JOIN city
ORDER BY name_city, Дата DESC;
```
```postgresql
SELECT name_city, name_author, generate_series('2010-01-01'::date, '2020-01-01'::date, '1 day'::interval) AS Дата
FROM
    author CROSS JOIN city
ORDER BY name_city, Дата DESC;
```
