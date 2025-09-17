# Домашнее задание к занятию «SQL. Часть 1» Шелухин Юрий

### Задание 1 
Получите уникальные названия районов из таблицы с адресами, которые начинаются на “K” и заканчиваются на “a” и не содержат пробелов. 
---

#### Решение 1.
1.1. Запустим MYSQL в контейнере Docker.    
`~$ docker run --name test-db -p 3306:3306 -e MYSQL_ROOT_PASSWORD=secret -d mysql:latest`   
Dbeaver не соединяется - "Public Key Retrieval is not allowed". Исправим во вкладке Driver Properties параметр "allowPublicKeyRetrieval" на значение "true".    
<img src = "img/1-1.png" width = 60%>     
1.2. Откроем SQL-скрипт.    
<img src = "img/1-2-1.png" width = 60%>  
С помощью SQ-команды создадим пользователя.    
`CREATE USER 'sys_temp'@'%' IDENTIFIED BY 'password';`  Вместо localhost  используем `%`, так как был конфликт при смене аутентификации (localhost root  и sys_temp имели разные ip). В работе этот вариант небезопасен, но для решения подойдет.  
<img src = "img/1-2-2.png" width = 60%>      
1.3 Выполним запрос для вывода всех пользователей.      
`SELECT user FROM mysql.user;`  
<img src = "img/1-3-1.png" width = 60%>    
<img src = "img/1-3-2.png" width = 60%>   
1.4 Выполним запрос для выдачи всех прав для пользователя sys_temp.   
`GRANT ALL PRIVILEGES ON *.* TO 'sys_temp'@'%';`  
<img src = "img/1-4.png" width = 60%>  
1.5 Выполним запрос для просмотра прав для пользователя sys_temp.   
`SHOW GRANTS FOR 'sys_temp'@'%';`  
<img src = "img/1-5.png" width = 60%>  
1.6 Переподключимся к базе данных от имени sys_temp и проверим.  
`SELECT USER(), CURRENT_USER();`  
<img src = "img/1-5.png" width = 60%>    
1.7 Скачать дамп базы данных посредством dbeaver не получилось, так как local client не cмог найти mysql 
в докере. Поэтому пришлось заталкивать дамп базы из хоста в докер, а затем загружать в mysql.  
`docker cp /home/yury/HW/SQL/DDL-DML/files/sakila-schema.sql  747aebd89200:/tmp/sakila-schema.sql    
 docker exec -it 747aebd89200 ls -la /tmp/    
 docker exec -it 747aebd89200 mysql -u sys_temp -p -e "CREATE DATABASE IF NOT EXISTS sakila;"    
 docker exec -it 747aebd89200 bash -c "mysql -u sys_temp -ppassword sakila < /tmp/dump.sql"`    
<img src = "img/1-7.png" width = 60%>   
1.8 Сформируем ER-диаграмму и, используя команду, выведем все таблицы базы данных.  
<img src = "img/1-8-1.png" width = 60%>  
`SHOW TABLES FROM sakila;`  
<img src = "img/1-8-2.png" width = 60%>  
Также загрузим данные.  
`docker cp /home/yury/HW/SQL/DDL-DML/files/sakila-data.sql 747aebd89200:/tmp/data.sql
docker cp /home/yury/HW/SQL/DDL-DML/files/sakila-data.sql 747aebd89200:/tmp/data.sql`
<img src = "img/1-8-2-2.png" width = 60%>   

Простыня со всеми запросами.  
<img src = "img/1-8-3.png" width = 60%>  
---


### Задание 2
Получите из таблицы платежей за прокат фильмов информацию по платежам, которые выполнялись в промежуток с 15 июня 2005 года по 18 июня 2005 года **включительно** и стоимость которых превышает 10.00.
---

#### Решение 2.
Получить данные можно визуальным изучением таблиц  
<img src = "img/2-1.png" width = 60%>   
<img src = "img/2-2.png" width = 60%>   
или с помощью sql-скрипта  
`SELECT 
    t.TABLE_NAME,
    k.COLUMN_NAME AS PRIMARY_KEY,
    k.CONSTRAINT_NAME
FROM information_schema.TABLES t
LEFT JOIN information_schema.KEY_COLUMN_USAGE k 
    ON t.TABLE_SCHEMA = k.TABLE_SCHEMA 
    AND t.TABLE_NAME = k.TABLE_NAME 
    AND k.CONSTRAINT_NAME = 'PRIMARY'
WHERE t.TABLE_SCHEMA = 'sakila'
    AND t.TABLE_TYPE = 'BASE TABLE'
ORDER BY t.TABLE_NAME;`  
<img src = "img/2-3.png" width = 60%>

[файл в формате Excel](files/task_2.ods)
---


### Задание 3
Получите последние пять аренд фильмов.

#### Решение 3*.
Выдадим все права.    
`GRANT ALL PRIVILEGES ON sakila.* TO 'sys_temp'@'%';`    
Исключим права на модификацию.    
`REVOKE INSERT, UPDATE, DELETE ON sakila.* FROM 'sys_temp'@'%';`  
Применим изменения.    
`FLUSH PRIVILEGES;`  
Проверим права.  
`SHOW GRANTS FOR 'sys_temp'@'%';`  
<img src = "img/3-1.png" width = 60%>  
---


### Задание 4
Одним запросом получите активных покупателей, имена которых Kelly или Willie. 
Сформируйте вывод в результат таким образом:
- все буквы в фамилии и имени из верхнего регистра переведите в нижний регистр,
- замените буквы 'll' в именах на 'pp'.

## Дополнительные задания (со звёздочкой*)
Эти задания дополнительные, то есть не обязательные к выполнению, и никак не повлияют на получение вами зачёта по этому домашнему заданию. Вы можете их выполнить, если хотите глубже шире разобраться в материале.

### Задание 5*
Выведите Email каждого покупателя, разделив значение Email на две отдельных колонки: в первой колонке должно быть значение, указанное до @, во второй — значение, указанное после @.

### Задание 6*
Доработайте запрос из предыдущего задания, скорректируйте значения в новых колонках: первая буква должна быть заглавной, остальные — строчными.

