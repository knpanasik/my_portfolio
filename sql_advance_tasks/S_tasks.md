 1. Найдите количество вопросов, которые набрали больше 300 очков или как минимум 100 раз были добавлены в «Закладки».

```SQL
SELECT COUNT(DISTINCT p.id)
FROM stackoverflow.posts AS p
JOIN stackoverflow.post_types AS pt ON p.post_type_id = pt.id
WHERE pt.type = 'Question' AND (p.score > 300 OR p.favorites_count >= 100)
```
___

 2. Сколько в среднем в день задавали вопросов с 1 по 18 ноября 2008 включительно? Результат округлить до целого числа.

```SQL
SELECT ROUND(AVG(q_number.count),0)
FROM (
        SELECT COUNT(id),
               creation_date::date
        FROM stackoverflow.posts
        WHERE post_type_id = (SELECT id FROM stackoverflow.post_types WHERE type = 'Question')
              AND (creation_date::date BETWEEN '2008-11-01' AND '2008-11-18')
        GROUP BY creation_date::date) AS q_number
```
___

3. Сколько пользователей получили значки сразу в день регистрации? Вывести количество уникальных пользователей.

```SQL
SELECT COUNT(DISTINCT u.id)
FROM stackoverflow.badges AS b
JOIN stackoverflow.users AS u ON b.user_id=u.id
WHERE b.creation_date::date = u.creation_date::date
```
___

4. Сколько уникальных постов пользователя с именем Joel Coehoorn получили хотя бы один голос?.

```SQL
SELECT COUNT(distinct p.id)
FROM stackoverflow.posts p
JOIN stackoverflow.users u on p.user_id = u.id
JOIN stackoverflow.votes v on p.id = v.post_id
WHERE u.display_name LIKE '%Joel Coehoorn%';
```
___

5. Выгрузить все поля таблицы `vote_types`. Добавить к таблице поле `rank`, в которое войдут номера записей в обратном порядке.
   Таблица должна быть отсортирована по полю id.

```SQL
SELECT *,
       ROW_NUMBER() OVER(ORDER BY id DESC) AS rank
FROM stackoverflow.vote_types
ORDER BY id
```
___

6. Отобрать 10 пользователей, которые поставили больше всего голосов типа `Close`. Отобразить таблицу из двух полей:
   идентификатором пользователя и количеством голосов. Отсортировать данные сначала по убыванию количества голосов,
   потом по убыванию значения идентификатора пользователя.

```SQL
SELECT *
FROM (
      SELECT v.user_id,
             COUNT(vt.id) AS cnt
      FROM stackoverflow.votes AS v
      JOIN stackoverflow.vote_types as vt ON vt.id = v.vote_type_id
      WHERE vt.name = 'Close'
      GROUP BY v.user_id
      ORDER BY cnt DESC 
      LIMIT 10
    ) AS temp
ORDER BY temp.cnt DESC, temp.user_id DESC
```
___

7. Отобрать 10 пользователей по количеству значков, полученных в период с 15 ноября по 15 декабря 2008 года включительно.
Отобразить: идентификатор пользователя; число значков; место в рейтинге — чем больше значков, тем выше рейтинг.
Пользователям, которые набрали одинаковое количество значков, присвойте одно и то же место в рейтинге. <br/>
Отсортировать записи по количеству значков по убыванию, а затем по возрастанию значения идентификатора пользователя.

```SQL
SELECT user_id,
       count(id) badg,
       DENSE_RANK() OVER(ORDER BY count(id) DESC)
FROM stackoverflow.badges
WHERE creation_date::date BETWEEN '2008-11-15' AND '2008-12-15'
GROUP BY user_id
ORDER BY badg DESC, user_id
LIMIT 10
```
___

8. Сколько в среднем очков получает пост каждого пользователя?
Вывести: заголовок поста; идентификатор пользователя; число очков поста;
среднее число очков пользователя за пост, округлённое до целого числа.
Не учитывать посты без заголовка, а также те, что набрали ноль очков.

```SQL
SELECT title,
       user_id,
       score,
       ROUND(AVG(score) OVER(PARTITION BY user_id))
FROM stackoverflow.posts
WHERE title IS NOT NULL AND score <> 0
```
___

9. Отобразить заголовки постов, которые были написаны пользователями, получившими более 1000 значков. <br/>
Посты без заголовков не должны попасть в список.

```SQL
SELECT title
FROM stackoverflow.posts
WHERE title IS NOT NULL AND user_id IN (
                  SELECT user_id
                  FROM stackoverflow.badges AS b
                  GROUP BY user_id
                  HAVING COUNT(b.id) > 1000
                  )
```
___

10. Написать запрос, который выгрузит данные о пользователях из Канады (англ. Canada).
    Разделите пользователей на три группы в зависимости от количества просмотров их профилей:
- пользователям с числом просмотров больше либо равным 350 присвойте группу 1;
- пользователям с числом просмотров меньше 350, но больше либо равно 100 — группу 2;
- пользователям с числом просмотров меньше 100 — группу 3.    
Отобразить в итоговой таблице идентификатор пользователя, количество просмотров профиля и группу.    
Пользователи с количеством просмотров меньше либо равным нулю не должны войти в итоговую таблицу.

```SQL
SELECT id,
       views,
       CASE
          WHEN views>=350 THEN 1
          WHEN views<100 THEN 3
          ELSE 2
       END AS group
FROM stackoverflow.users
WHERE location LIKE '%Canada%'
   AND views > 0
```
___

11. Дополнить предыдущий запрос. Отобразить лидеров каждой группы — пользователей, которые набрали максимальное число просмотров в своей группе.
    Вывести поля с идентификатором пользователя, группой и количеством просмотров.

```SQL
WITH tab AS
(SELECT temp.id,
        temp.views,
        temp.group,
        MAX(temp.views) OVER (PARTITION BY temp.group) AS max	
 FROM (SELECT id,
              views,
              CASE
                 WHEN views >= 350 THEN 1
                 WHEN views < 100 THEN 3
                 ELSE 2
              END AS group
       FROM stackoverflow.users
       WHERE location LIKE '%Canada%'
          AND views > 0
          ) as temp
  )
  
SELECT tab.id, 
       tab.views,  
       tab.group
FROM tab
WHERE tab.views = tab.max
ORDER BY tab.views DESC, tab.id
```
___

12. Посчитать ежедневный прирост новых пользователей в ноябре 2008 года. Сформируйте таблицу с полями:
номер дня; число пользователей, зарегистрированных в этот день; сумму пользователей с накоплением.

```SQL
SELECT *,
       SUM(temp.cnt_id) OVER (ORDER BY temp.days) as cum_num
FROM (
      SELECT EXTRACT(DAY FROM creation_date::date) AS days,
             COUNT(id) AS cnt_id
      FROM stackoverflow.users
      WHERE creation_date::date BETWEEN '2008-11-01' AND '2008-11-30'
      GROUP BY EXTRACT(DAY FROM creation_date::date)
      ) as temp
```
___

13. Для каждого пользователя, который написал хотя бы один пост, найти интервал между регистрацией и временем создания первого поста.
    Отобразить: идентификатор пользователя; разницу во времени между регистрацией и первым постом.

```SQL
WITH reg AS (
            SELECT id AS uid,
                creation_date AS reg_date
            FROM stackoverflow.users
            GROUP  BY id, creation_date
            ),
     p_user AS (
                 SELECT user_id AS user_p,
                     creation_date AS post_date,
                     ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY creation_date) AS d_num    
                 FROM stackoverflow.posts  
                 )
                 
SELECT uid,
       (post_date - reg_date) AS diff
FROM p_user LEFT JOIN reg ON p_user.user_p = reg.uid
WHERE d_num = 1
```
___

14. Вывести общую сумму просмотров у постов, опубликованных в каждый месяц 2008 года. <br/>
Результат отсортировать по убыванию общего количества просмотров.
    
```SQL
SELECT SUM(views_count) AS num_views,
       CAST(DATE_TRUNC('month', creation_date) AS date) AS mnth
FROM stackoverflow.posts
WHERE EXTRACT(YEAR FROM CAST(creation_date as date)) = 2008
GROUP BY CAST(DATE_TRUNC('month', creation_date) AS date)
ORDER BY SUM(views_count) DESC
```
___

15. Вывести имена самых активных пользователей, которые в первый месяц после регистрации (включая день регистрации) дали больше 100 ответов. <br/>
    Вопросы, которые задавали пользователи, не учитываются. <br/>
    Для каждого имени пользователя вывести количество уникальных значений user_id.
    
```SQL
SELECT u.display_name,
       COUNT(DISTINCT p.user_id)
FROM stackoverflow.posts AS p
JOIN stackoverflow.users AS u ON p.user_id = u.id
JOIN stackoverflow.post_types AS pt ON pt.id = p.post_type_id
WHERE p.creation_date::date BETWEEN u.creation_date::date AND (u.creation_date::date + INTERVAL '1 month')
AND pt.type LIKE '%Answer%'
GROUP BY u.display_name
HAVING COUNT(DISTINCT p.id) > 100
ORDER BY u.display_name
```
___

16. Вывести количество постов за 2008 год по месяцам. Отобрать посты от пользователей, которые зарегистрировались в сентябре 2008 года
    и сделали хотя бы один пост в декабре того же года. Отсортировать таблицу по значению месяца по убыванию.
    
```SQL
WITH tab AS (
           SELECT u.id AS uid
           FROM stackoverflow.posts AS p
           JOIN stackoverflow.users AS u ON p.user_id = u.id
           WHERE CAST(DATE_TRUNC('month', u.creation_date) AS date) = '2008-09-01'
                 AND CAST(DATE_TRUNC('month', p.creation_date) AS date) = '2008-12-01'
           GROUP BY u.id
           HAVING COUNT(p.id) > 0
          )

SELECT COUNT(p.id),
       CAST(DATE_TRUNC('month', p.creation_date) AS date)      
FROM stackoverflow.posts AS p
WHERE p.user_id IN (SELECT uid FROM tab)
      AND EXTRACT(YEAR FROM p.creation_date) = '2008'
GROUP BY CAST(DATE_TRUNC('month', p.creation_date) AS date)
ORDER BY CAST(DATE_TRUNC('month', p.creation_date) AS date) DESC
```
___

17. Используя данные о постах, вывести несколько полей:
идентификатор пользователя, который написал пост; дата создания поста; количество просмотров у текущего поста; сумма просмотров постов автора с накоплением.
Данные в таблице должны быть отсортированы по возрастанию идентификаторов пользователей, а данные об одном и том же пользователе — по возрастанию даты создания поста.
    
```SQL
SELECT user_id,
       creation_date,
       views_count,
       SUM(views_count) OVER (PARTITION BY user_id ORDER BY creation_date)						
FROM stackoverflow.posts
ORDER BY user_id, creation_date
```
___

18. Сколько в среднем дней в период с 1 по 7 декабря 2008 года включительно пользователи взаимодействовали с платформой?
    Для каждого пользователя отобрать дни, в которые он или она опубликовали хотя бы один пост. Результат округлить до целого.
    
```SQL
WITH profiles AS (
                SELECT DISTINCT user_id,
                       DATE_TRUNC('day', creation_date)::date AS p_days
                FROM stackoverflow.posts
                WHERE DATE_TRUNC('day', creation_date)::date BETWEEN '2008-12-01' AND '2008-12-07'
                ),
     sorted_profiles AS (
                         SELECT user_id,
                                COUNT(p_days) AS d_cnt
                         FROM profiles
                         GROUP BY user_id
                        )
SELECT ROUND(AVG(d_cnt)) AS avg_posts_per_day
FROM sorted_profiles
```
___

19. На сколько процентов менялось количество постов ежемесячно с 1 сентября по 31 декабря 2008 года? Отобразите таблицу со следующими полями:
Номер месяца. Количество постов за месяц. Процент, который показывает, насколько изменилось количество постов в текущем месяце по сравнению с предыдущим.
Округлить значение процента до двух знаков после запятой.
    
```SQL
WITH temp_table AS (
                    SELECT EXTRACT(MONTH from creation_date::date) AS month,
                           COUNT(DISTINCT id) AS cnt   
                    FROM stackoverflow.posts
                    WHERE creation_date::date BETWEEN '2008-09-01' AND '2008-12-31'
                    GROUP BY EXTRACT(MONTH from creation_date::date)
)

SELECT *,
       ROUND(((cnt::numeric / LAG(cnt) OVER (ORDER BY month)) - 1) * 100,2) AS user_growth
FROM temp_table
```
___

20. Найти пользователя, который опубликовал больше всего постов за всё время с момента регистрации.
    Вывести данные его активности за октябрь 2008 года в таком виде: номер недели; дата и время последнего поста, опубликованного на этой неделе.
    
```SQL
WITH profile AS (
            SELECT user_id,
                   COUNT(DISTINCT id) AS cnt
            FROM stackoverflow.posts
            GROUP BY user_id
            ORDER BY cnt DESC
            LIMIT 1),

     p_info AS (
            SELECT p.user_id,
                   p.creation_date,
                   extract('week' from p.creation_date) AS week_number
           FROM stackoverflow.posts AS p
           JOIN profile ON profile.user_id = p.user_id
           WHERE DATE_TRUNC('month', p.creation_date)::date = '2008-10-01'
           )

SELECT DISTINCT week_number::numeric,
       MAX(creation_date) OVER (PARTITION BY week_number)
FROM p_info
ORDER BY week_number
```
___
