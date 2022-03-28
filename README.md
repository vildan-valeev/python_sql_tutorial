`docker-compose up`

`docker exec -it postgres_db bash`

`psql -U postgres`

`INSERT INTO book (title, author, price, amount) VALUES ('Идиот', 'Достоевский Ф.М.', 460.00, 10);`

```sql
CREATE TABLE book (
    book_id SERIAL PRIMARY KEY ,
    title VARCHAR(30),
    author VARCHAR(30),
    price DECIMAL(8, 2),
    amount INT
);

INSERT INTO book (title, author, price, amount) VALUES ('Мастер и Маргарита', 'Булгаков М.А.', 670.99, 3);

INSERT INTO book (title, author, price, amount) VALUES ('Белая гвардия', 'Булгаков М.А.', 540.00, 5);

INSERT INTO book (title, author, price, amount) VALUES ('Идиот', 'Достоевский Ф.М.', 460.00, 10);

INSERT INTO book (title, author, price, amount) VALUES ('Братья Карамазовы', 'Достоевский Ф.М.', 799.01, 2);

INSERT INTO book (title, author, price, amount) VALUES ('Игрок', 'Достоевский Ф.М.', 480.50, 10);

INSERT INTO book (title, author, price, amount) VALUES ('Стихотворения и поэмы', 'Есенин С.А.', 650.00, 15);
```
1. Для упаковки каждой книги требуется один лист бумаги, цена которого 1 рубль 65 копеек. Посчитать стоимость упаковки для каждой книги 
(сколько денег потребуется, чтобы упаковать все экземпляры книги). В запросе вывести название книги, ее количество и стоимость упаковки, последний столбец назвать pack. 
```sql
SELECT title, amount, 1.65 * amount as pack
FROM book;
```
2. В конце года цену всех книг на складе пересчитывают – снижают ее на 30%. Написать SQL запрос, который из таблицы book выбирает названия, авторов, количества и вычисляет новые цены книг. Столбец с новой ценой назвать new_price, цену округлить до 2-х знаков после запятой.
```sql
SELECT title, author, amount, round(price * 0.7, 2) as new_price
FROM book;
```
3. При анализе продаж книг выяснилось, что наибольшей популярностью пользуются книги Михаила Булгакова, на втором месте книги Сергея Есенина. Исходя из этого решили поднять цену книг Булгакова на 10%, а цену книг Есенина - на 5%. Написать запрос, куда включить автора, название книги и новую цену, последний столбец назвать new_price. Значение округлить до двух знаков после запятой.
```sql
SELECT author, title,
       round(
           (CASE
               WHEN author = 'Булгаков М.А.' THEN price * 1.1
               WHEN author= 'Есенин С.А.' THEN price * 1.05
               ELSE price END),
           2) AS new_price
FROM book;
```
4. Вывести автора, название  и цены тех книг, количество которых меньше 10.
```sql
SELECT author, title, price
FROM book
WHERE amount < 10;
```
5. Вывести название, автора,  цену  и количество всех книг, цена которых меньше 500 или больше 600, а стоимость всех экземпляров этих книг больше или равна 5000.
```sql
SELECT *
FROM book
WHERE (price < 500 OR price > 600) AND price * amount >= 5000;
```
6. Вывести название и авторов тех книг, цены которых принадлежат интервалу от 540.50 до 800 (включая границы),  а количество или 2, или 3, или 5, или 7 .
```sql
SELECT title, author
FROM book
WHERE (price BETWEEN 540.5 AND 800) AND (amount=2 OR amount=3 OR amount=5 OR amount=7);
```
7. Вывести автора и название книг, количество которых принадлежит интервалу от 2 до 14 (включая границы). Информацию  отсортировать сначала по авторам (в обратном алфавитном порядке), а затем по названиям книг (по алфавиту).
```sql
SELECT author, title
FROM book
WHERE amount BETWEEN 2 AND 14
ORDER BY author DESC , title;
```
8. 
Вывести название и автора тех книг, название которых состоит из двух и более слов, а инициалы автора содержат букву «С». Считать, что в названии слова отделяются друг от друга пробелами и не содержат знаков препинания, между фамилией автора и инициалами обязателен пробел, инициалы записываются без пробела в формате: буква, точка, буква, точка. Информацию отсортировать по названию книги в алфавитном порядке.
```sql
SELECT title, author FROM book
WHERE title LIKE '_% _%' AND (author LIKE '% С%' OR author LIKE 'С%' OR author LIKE '%.С');
```
9. Вывести информацию о книгах(автор, название, цена), цена которых меньше самой большой из минимальных цен, вычисленных для каждого автора.
```sql
SELECT author, title, price
FROM book
WHERE price < ANY (
    SELECT AVG(price)
    FROM book
    GROUP BY author
          );
```
10. Посчитать сколько и каких экземпляров книг нужно заказать поставщикам, чтобы на складе стало одинаковое количество экземпляров каждой книги, равное значению самого большего количества экземпляров одной книги на складе. Вывести название книги, ее автора, текущее количество экземпляров на складе и количество заказываемых экземпляров книг. Последнему столбцу присвоить имя Заказ. В результат не включать книги, которые заказывать не нужно.
```sql
SELECT title, author, amount, ((
    SELECT max(amount)
    FROM book
    ) - amount) AS Заказ
FROM book
WHERE (amount - (SELECT max(amount) FROM book)) != 0;
```
11. при продаже всех книг, какая книга принесет больше всего выручки, в процентах.
Судя по результату, магазин хорошо вложился в Стихи Есенина
```sql
SELECT *, round((100*price*amount/(SELECT SUM(price*amount) FROM book)), 2) AS income_percent
FROM book
ORDER BY income_percent DESC;
```
##  Запросы корректировки данных
1.
2. Создать таблицу поставка (supply), которая имеет ту же структуру, что и таблиц book.
```sql
CREATE TABLE supply (
    supply_id SERIAL PRIMARY KEY ,
    title VARCHAR(50),
    author VARCHAR(30),
    price DECIMAL(8, 2),
    amount INT
);
```
3. Занесите в таблицу supply четыре записи, чтобы получилась следующая таблица:
```sql
INSERT INTO supply (title, author, price, amount)
VALUES
    ('Лирика', 'Пастернак Б.Л.', 518.99, 2),
    ('Черный человек', 'Есенин С.А.', 570.20, 6),
    ('Белая гвардия', 'Булгаков М.А.', 540.50, 7),
    ('Идиот', 'Достоевский Ф.М.', 360.80, 3);

SELECT * FROM supply;
```
4. Добавить из таблицы supply в таблицу book, все книги, кроме книг, написанных Булгаковым М.А. и Достоевским Ф.М.
```sql
INSERT INTO book (title, author, price, amount)
SELECT title, author, price, amount
FROM supply
WHERE author NOT IN ('Булгаков М.А.', 'Достоевский Ф.М.');

SELECT * FROM book;
```
5. Занести из таблицы supply в таблицу book только те книги, авторов которых нет в book.
```sql
INSERT INTO book (title, author, price, amount)
SELECT title, author, price, amount
FROM supply
WHERE author NOT IN (
    SELECT author
    FROM book
    );

SELECT * FROM book;
```
6. Уменьшить на 10% цену тех книг в таблице book, количество которых принадлежит интервалу от 5 до 10, включая границы.
```sql
UPDATE book
SET price = price * 0.9
WHERE amount BETWEEN 5 AND 10;
SELECT * FROM book;
```
7. В таблице book необходимо скорректировать значение для покупателя в столбце buy таким образом, 
чтобы оно не превышало количество экземпляров книг, указанных в столбце amount. 
А цену тех книг, которые покупатель не заказывал, снизить на 10%.
```sql
UPDATE book
SET buy = IF(buy >= amount, amount, buy),
    price = IF(buy = 0, price * 0.9, price);

SELECT * FROM book;
```
```postgresql
UPDATE book
SET
    buy = CASE WHEN buy >= amount THEN amount ELSE buy END,
    price = CASE WHEN buy = 0 THEN price * 0.9 ELSE price END;

SELECT * FROM book;
```
8. Для тех книг в таблице book, которые есть в таблице supply, не только увеличить их количество в таблице book 
(увеличить их количество на значение столбца amount таблицы supply), 
но и пересчитать их цену (для каждой книги найти сумму цен из таблиц book и supply и разделить на 2).
```sql
UPDATE book, supply
SET book.amount = book.amount + supply.amount,
    book.price = (book.price + supply.price)/2
WHERE book.title = supply.title AND book.author = supply.author;
```
```postgresql
UPDATE book
SET amount = book.amount + supply.amount,
    price = (book.price + supply.price)/2
FROM supply
WHERE book.title = supply.title
  AND book.author = supply.author;

SELECT * FROM book;
```
9. Удалить из таблицы supply книги тех авторов, общее количество экземпляров книг которых в таблице book превышает 10.
```sql
DELETE FROM supply
WHERE author IN(
    SELECT author
    FROM book
    GROUP BY author
    HAVING sum(amount) >= 10
    );

SELECT * FROM supply;

```
10. Создать таблицу заказ (ordering), куда включить авторов и названия тех книг, 
количество экземпляров которых в таблице book меньше среднего количества экземпляров книг в таблице book. 
В таблицу включить столбец amount, в котором для всех книг указать одинаковое значение - 
среднее количество экземпляров книг в таблице book.
```sql
CREATE TABLE ordering AS
SELECT author, title,
    (
    SELECT ROUND(AVG(amount))
    FROM book
    ) as amount
FROM book
WHERE amount  < (
    SELECT ROUND(AVG(amount))
    FROM book
    );

SELECT * FROM ordering;
```
## Таблица "командировки"
1. 
```sql
CREATE TABLE trip
(
trip_id BIGSERIAL,
name VARCHAR(30),
city VARCHAR(25),
per_diem DECIMAL(8,2),
date_first DATE,
date_last DATE
);
```
```sql
INSERT INTO trip(name, city, per_diem, date_first, date_last)
VALUES
('Баранов П.Е.', 'Москва', '700', '2020-01-12', '2020-01-17'),
('Абрамова К.А.', 'Владивосток', '450', '2020-01-14', '2020-01-27'),
('Семенов И.В.', 'Москва', '700', '2020-01-23', '2020-01-31'),
('Ильиных Г.Р.', 'Владивосток', '450', '2020-01-12', '2020-02-02'),
('Колесов С.П.', 'Москва', '700', '2020-02-01', '2020-02-06'),
('Баранов П.Е.', 'Москва', '700', '2020-02-14', '2020-02-22'),
('Абрамова К.А.', 'Москва', '700', '2020-02-23', '2020-03-01'),
('Лебедев Т.К.', 'Москва', '700', '2020-03-03', '2020-03-06'),
('Колесов С.П.', 'Новосибирск', '450', '2020-02-27', '2020-03-12'),
('Семенов И.В.', 'Санкт-Петербург', '700', '2020-03-29', '2020-04-05'),
('Абрамова К.А.', 'Москва', '700', '2020-04-06', '2020-04-14'),
('Баранов П.Е.', 'Новосибирск', '450', '2020-04-18', '2020-05-04'),
('Лебедев Т.К.', 'Томск', '450', '2020-05-20', '2020-05-31'),
('Семенов И.В.', 'Санкт-Петербург', '700', '2020-06-01', '2020-06-03'),
('Абрамова К.А.', 'Санкт-Петербург', '700', '2020-05-28', '2020-06-04'),
('Федорова А.Ю.', 'Новосибирск', '450', '2020-05-25', '2020-06-04'),
('Колесов С.П.', 'Новосибирск', '450', '2020-06-03', '2020-06-12'),
('Федорова А.Ю.', 'Томск', '450', '2020-06-20', '2020-06-26'),
('Абрамова К.А.', 'Владивосток', '450', '2020-07-02', '2020-07-13'),
('Баранов П.Е.', 'Воронеж', '450', '2020-07-19', '2020-07-25');
SELECT * FROM trip;
```
2. Вывести из таблицы trip информацию о командировках тех сотрудников, 
фамилия которых заканчивается на букву «а», в отсортированном по убыванию даты 
последнего дня командировки виде. 
В результат включить столбцы name, city, per_diem, date_first, date_last.
```sql
SELECT name, city, per_diem, date_first, date_last
FROM trip
WHERE name LIKE '%а %.%.'
ORDER BY date_last DESC;
```
3. Вывести в алфавитном порядке фамилии и инициалы тех сотрудников, которые были в командировке в Москве.
```sql
SELECT DISTINCT name
FROM trip
WHERE city = 'Москва'
ORDER BY name;
```
4. Для каждого города посчитать, сколько раз сотрудники в нем были.  Информацию вывести в отсортированном 
в алфавитном порядке по названию городов. Вычисляемый столбец назвать Количество. 
```sql
SELECT city, COUNT(*) as Количество
FROM trip
GROUP BY city
ORDER BY city;
```
5. Вывести два города, в которых чаще всего были в командировках сотрудники. 
Вычисляемый столбец назвать Количество.
```sql
SELECT city, COUNT(*) as Количество
FROM trip
GROUP BY city
ORDER BY Количество DESC 
LIMIT 2;
```
6. Вывести информацию о командировках во все города кроме Москвы и Санкт-Петербурга
(фамилии и инициалы сотрудников, город ,  длительность командировки в днях, 
при этом первый и последний день относится к периоду командировки). 
Последний столбец назвать Длительность. Информацию вывести в упорядоченном по убыванию 
длительности поездки, а потом по убыванию названий городов (в обратном алфавитном порядке).
```sql
SELECT name, city, DATEDIFF(date_last, date_first)+1 AS Длительность
FROM trip
WHERE city NOT IN ('Москва', 'Санкт-Петербург')
ORDER BY Длительность DESC;
```
```postgresql
SELECT name, city, DATE_PART('day', date_last::timestamp - date_first::timestamp) + 1 AS Длительность
FROM trip
WHERE city NOT IN ('Москва', 'Санкт-Петербург')
ORDER BY Длительность DESC;
```
7. Вывести информацию о командировках сотрудника(ов), которые были самыми короткими по времени. 
В результат включить столбцы name, city, date_first, date_last.
```sql
SELECT name, city, date_first, date_last
FROM trip
WHERE DATEDIFF(date_last, date_first) = (
    SELECT MIN(DATEDIFF(date_last, date_first))
    FROM trip
    );
```
```postgresql
SELECT name, city, date_first, date_last
FROM trip
WHERE DATE_PART('day', date_last::timestamp - date_first::timestamp) = (
    SELECT MIN(DATE_PART('day', date_last::timestamp - date_first::timestamp))
    FROM trip
    );
```
8. Вывести информацию о командировках, начало и конец которых относятся к одному месяцу 
(год может быть любой). В результат включить столбцы name, city, date_first, date_last.
Строки отсортировать сначала в алфавитном порядке по названию города, а затем по фамилии сотрудника.
```sql
SELECT name, city, date_first, date_last
FROM trip
WHERE MONTH(date_first)=MONTH(date_last)
ORDER BY city, name;
```
```postgresql
SELECT name, city, date_first, date_last
FROM trip
WHERE EXTRACT(MONTH FROM date_first)=EXTRACT(MONTH FROM date_last)
ORDER BY city, name;
```
