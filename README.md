## Домашнее задание к занятию «Индексы»-Рустам Ахмадеев
Инструкция по выполнению домашнего задания
1. Сделайте fork репозитория c шаблоном решения к себе в Github и переименуйте его по названию или номеру занятия, например, https://github.com/имя-вашего-репозитория/gitlab-hw или https://github.com/имя-вашего-репозитория/8-03-hw).
2. Выполните клонирование этого репозитория к себе на ПК с помощью команды git clone.
3. Выполните домашнее задание и заполните у себя локально этот файл README.md:
- впишите вверху название занятия и ваши фамилию и имя;
- в каждом задании добавьте решение в требуемом виде: текст/код/скриншоты/ссылка;
- для корректного добавления скриншотов воспользуйтесь инструкцией «Как вставить скриншот в шаблон с решением»;
- при оформлении используйте возможности языка разметки md. Коротко об этом можно посмотреть в инструкции по MarkDown.
4. После завершения работы над домашним заданием сделайте коммит (git commit -m "comment") и отправьте его на Github (git push origin).
5. Для проверки домашнего задания преподавателем в личном кабинете прикрепите и отправьте ссылку на решение в виде md-файла в вашем Github.
6. Любые вопросы задавайте в чате учебной группы и/или в разделе «Вопросы по заданию» в личном кабинете.
Желаем успехов в выполнении домашнего задания.

### Задание 1

Напишите запрос к учебной базе данных, который вернёт процентное отношение общего размера всех индексов к общему размеру всех таблиц.

Решение 1

```
SELECT table_schema as DB_name,
CONCAT(ROUND((SUM(index_length))*100/(SUM(data_length+index_length)),2),'%') '% of index'
FROM information_schema.TABLES where TABLE_SCHEMA = 'sakila'

```
![alt text](https://github.com/ahmrust/indexes/blob/main/img/1.png)

### Задание 2

Выполните explain analyze следующего запроса:

select distinct concat(c.last_name, ' ', c.first_name), sum(p.amount) over (partition by c.customer_id, f.title)
from payment p, rental r, customer c, inventory i, film f
where date(p.payment_date) = '2005-07-30' and p.payment_date = r.rental_date and r.customer_id = c.customer_id and i.inventory_id = r.inventory_id
перечислите узкие места;
оптимизируйте запрос: внесите корректировки по использованию операторов, при необходимости добавьте индексы.

Решение 2

В запросе происходит излишняя обработка таблиц inventory, rental и film, не связанных с подсчитыванием суммы платежей покупателей за конкретную дату. Все необходимые данные есть в таблицах payment и customer, соответственно, остальные таблицы можно исключить. 
Возможно оптимизировать запрос:

```
explain analyze
SELECT DISTINCT 
       CONCAT(c.last_name, ' ', c.first_name) AS full_name, SUM(p.amount) OVER (PARTITION BY c.customer_id, f.title)
FROM payment p
JOIN rental r ON p.payment_date = r.rental_date
JOIN customer c ON r.customer_id = c.customer_id
JOIN inventory i ON r.inventory_id = i.inventory_id
JOIN film f ON i.film_id = f.film_id
WHERE payment_date >= '2005-07-30' and payment_date < DATE_ADD('2005-07-30', INTERVAL 1 DAY) 
```
![alt text](https://github.com/ahmrust/indexes/blob/main/img/2.png)
![alt text](https://github.com/ahmrust/indexes/blob/main/img/3.png)
![alt text](https://github.com/ahmrust/indexes/blob/main/img/4.png)

### Дополнительные задания (со звёздочкой*)
Эти задания дополнительные, то есть не обязательные к выполнению, и никак не повлияют на получение вами зачёта по этому домашнему заданию. Вы можете их выполнить, если хотите глубже шире разобраться в материале.

### Задание 3*
Самостоятельно изучите, какие типы индексов используются в PostgreSQL. Перечислите те индексы, которые используются в PostgreSQL, а в MySQL — нет.
Приведите ответ в свободной форме.

