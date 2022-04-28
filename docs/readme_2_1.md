##  Связи между таблицами
1. Создать таблицу author следующей структуры:
```sql
CREATE TABLE author
    (
        author_id INT PRIMARY KEY AUTO_INCREMENT,
        name_author varchar(50)
 );
```
```postgresql
CREATE TABLE author
    (
        author_id serial PRIMARY KEY,
        name_author varchar(50)
    );
```
2. Заполнить таблицу author. В нее включить следующих авторов:
```sql
INSERT INTO author (name_author)
VALUES ('Пастернак Б.Л.');
```

