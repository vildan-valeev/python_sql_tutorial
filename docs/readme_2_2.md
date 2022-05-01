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
