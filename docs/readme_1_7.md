##  Таблица "Нарушения ПДД"
1. Создать таблицу fine следующей структуры:
```sql
CREATE TABLE fine
    (
        fine_id INT PRIMARY KEY AUTO_INCREMENT,
        name varchar(30),
        number_plate varchar(6),
        violation varchar(50),
        sum_fine decimal(8, 2),
        date_violation date,
        date_payment date
 );
```
```postgresql
CREATE TABLE fine
    (
        fine_id serial
            PRIMARY KEY,
        name varchar(30),
        number_plate varchar(6),
        violation varchar(50),
        sum_fine decimal(8, 2),
        date_violation date,
        date_payment date
    );
```
```postgresql

INSERT INTO fine (name, number_plate, violation, sum_fine, date_violation, date_payment)
VALUES ('Баранов П.Е.', 'Р523ВТ', 'Превышение скорости(от 40 до 60)', 500.00, '2020-01-12', '2020-01-17'),
       ('Абрамова К.А.', 'О111АВ', 'Проезд на запрещающий сигнал', 1000.00, '2020-01-14', '2020-02-27'),
       ('Яковлев Г.Р.', 'Т330ТТ', 'Превышение скорости(от 20 до 40)', 500.00, '2020-01-23', '2020-02-23'),
       ('Яковлев Г.Р.', 'М701АА', 'Превышение скорости(от 20 до 40)', NULL, '2020-01-12', NULL),
       ('Колесов С.П.', 'К892АХ', 'Превышение скорости(от 20 до 40)', NULL, '2020-02-01', NULL);

CREATE TABLE traffic_violation
    (
        violation_id serial
            PRIMARY KEY,
        violation varchar(50),
        sum_fine decimal(8, 2)
    );
```
2. 
```sql

INSERT INTO traffic_violation (violation, sum_fine)
VALUES ('Превышение скорости(от 20 до 40)', 500),
       ('Превышение скорости(от 40 до 60)', 1000),
       ('Проезд на запрещающий сигнал', 1000);

```
3. В таблицу fine первые 5 строк уже занесены. Добавить в таблицу записи с ключевыми значениями 6, 7, 8.
```sql
INSERT INTO fine (name, number_plate, violation, sum_fine, date_violation, date_payment)
VALUES ('Баранов П.Е.', 'Р523ВТ', 'Превышение скорости(от 40 до 60)', NULL, '2020-02-14 ', NULL),
       ('Абрамова К.А.', 'О111АВ', 'Проезд на запрещающий сигнал', NULL, '2020-02-23', NULL),
       ('Яковлев Г.Р.', 'Т330ТТ', 'Проезд на запрещающий сигнал', NULL, '2020-03-03', NULL);
```
4. Занести в таблицу fine суммы штрафов, которые должен оплатить водитель, в соответствии с данными 
из таблицы traffic_violation. При этом суммы заносить только в пустые поля столбца  sum_fine.
Таблица traffic_violation создана и заполнена.
```sql
UPDATE fine
SET sum_fine = tv.sum_fine
FROM traffic_violation AS tv
WHERE fine.sum_fine IS NULL AND fine.violation=tv.violation;

SELECT * FROM fine;
```

```postgresql
UPDATE fine
SET sum_fine = tv.sum_fine
FROM traffic_violation AS tv
WHERE fine.sum_fine IS NULL AND fine.violation=tv.violation;

SELECT * FROM fine;
```
5. Вывести фамилию, номер машины и нарушение только для тех водителей, которые на одной машине нарушили одно и то же правило   два и более раз. При этом учитывать все нарушения, независимо от того оплачены они или нет. Информацию отсортировать в алфавитном порядке, сначала по фамилии водителя, потом по номеру машины и, наконец, по нарушению.
```sql
SELECT name, number_plate, violation
FROM fine
GROUP BY name, number_plate, violation
HAVING COUNT(violation) > 1
ORDER BY name;
```
6. В таблице fine увеличить в два раза сумму неоплаченных штрафов для отобранных на предыдущем шаге записей. 
```sql
UPDATE fine AS f, (
	   SELECT name, number_plate, violation
  		 FROM fine
 		GROUP BY name, number_plate, violation
	   HAVING count(*) >= 2
	   ) AS dv
   SET f.sum_fine = f.sum_fine*2
 WHERE f.date_payment IS Null
	   AND (f.name = dv.name
	   AND f.violation = dv.violation);

SELECT * FROM fine;
```
7. Водители оплачивают свои штрафы. В таблице payment занесены даты их оплаты:
```sql
CREATE TABLE payment
    (
        fine_id serial
            PRIMARY KEY,
        name varchar(30),
        number_plate varchar(6),
        violation varchar(50),
        date_violation date,
        date_payment date
    );

INSERT INTO payment (name, number_plate, violation, date_violation, date_payment)
VALUES ('Яковлев Г.Р.', 'М701АА', 'Превышение скорости(от 20 до 40)', '2020-01-12', '2020-01-22'),
       ('Баранов П.Е.', 'Р523ВТ', 'Превышение скорости(от 40 до 60)', '2020-02-14', '2020-03-06'),
       ('Яковлев Г.Р.', 'Т330ТТ', 'Проезд на запрещающий сигнал', '2020-03-03', '2020-03-23');
```
```sql
UPDATE fine f, payment p
SET f.date_payment = p.date_payment,
    f.sum_fine = IF(DATEDIFF(f.date_payment, f.date_violation) <= 20, f.sum_fine / 2, f.sum_fine)
WHERE f.name = p.name AND
      f.number_plate = p.number_plate AND
      f.violation = p.violation AND
      f.date_violation = p.date_violation AND
      f.date_payment IS NULL;

SELECT name, violation, sum_fine, date_violation, date_payment
FROM fine;
```
8. Создать новую таблицу back_payment, куда внести информацию о неоплаченных штрафах (Фамилию и инициалы водителя, 
номер машины, нарушение, сумму штрафа  и  дату нарушения) из таблицы fine.
```sql
CREATE TABLE back_payment
    (SELECT name, number_plate, violation, sum_fine, date_violation
     FROM fine
     WHERE date_payment IS NULL);
     
SELECT * FROM back_payment
```
