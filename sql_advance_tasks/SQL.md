 1. Найдите количество вопросов, которые набрали больше 300 очков или как минимум 100 раз были добавлены в «Закладки».

```SQL
select count(p.id)
from stackoverflow.posts p
where p.post_type_id = 1 and (p.score > 300 or p.favorites_count >= 100);
```

Результат: 
| count | 
|----   |  
| 1355 |
___

 2. Сколько в среднем в день задавали вопросов с 1 по 18 ноября 2008 включительно? Результат округлить до целого числа.

```SQL
select round(avg(daily)) as avg_daily_questions
from (select count(id) as daily
      from stackoverflow.posts
      where post_type_id = 1 and (creation_date::date between '2008-11-01' and '2008-11-18')
      group by creation_date::date) as daily_count;
```

Результат: 
| avg_daily_questions | 
|----   |  
| 383 |
___

3. Сколько пользователей получили значки сразу в день регистрации? Вывести количество уникальных пользователей.

```SQL
select count(distinct u.id)
from stackoverflow.users u
join stackoverflow.badges b on u.id=b.user_id
where u.creation_date::date = b.creation_date::date;
```

Результат: 
| count | 
|----   |  
| 7047 |
___

4. Сколько уникальных постов пользователя с именем Joel Coehoorn получили хотя бы один голос?.

```SQL
select count(distinct p.id)
from stackoverflow.posts p
join stackoverflow.users u on p.user_id = u.id
join stackoverflow.votes v on p.id = v.post_id
where u.display_name like '%Joel Coehoorn%';
```

Результат: 
| count | 
|----   |  
| 12 |
___

5. Выгрузить все поля таблицы `vote_types`. Добавить к таблице поле `rank`, в которое войдут номера записей в обратном порядке.
   Таблица должна быть отсортирована по полю id.

```SQL
select *,
       rank() over(order by id desc)
from stackoverflow.vote_types
order by id;
```

Результат: 
| id	| name |	rank |
|--   |----- |------ |
| 1 | AcceptedByOriginator |	15 |
| 2	| UpMod	| 14 |
| 3	| DownMod	| 13 |
|...| 
___

6. Отобрать 10 пользователей, которые поставили больше всего голосов типа `Close`. Отобразить таблицу из двух полей:
   идентификатором пользователя и количеством голосов. 

```SQL
select v.user_id as userss,
       count(vote_type_id) as votess
from stackoverflow.votes v
join stackoverflow.users u on v.user_id = u.id
where v.vote_type_id = 6
group by v.user_id
order by votess desc, userss desc
limit 10;
```

Результат: 
| users	| votes |
|----   |------ |
| 20646	| 36 |
| 14728	| 36 |
| 27163	| 29 |
| 41158	| 24 |
|...| 
___

7. Отобрать 10 пользователей по количеству значков, полученных в период с 15 ноября по 15 декабря 2008 года включительно.
Отобразить: идентификатор пользователя; число значков; место в рейтинге — чем больше значков, тем выше рейтинг.
Пользователям, которые набрали одинаковое количество значков, присвойте одно и то же место в рейтинге.

```SQL
select user_id,
       count(id) badg,
       DENSE_RANK() OVER(ORDER BY count(id) DESC)
from stackoverflow.badges
where creation_date::date between '2008-11-15' and '2008-12-15'
group by user_id
order by badg desc, user_id
limit 10;
```

Результат: 
| user_id	| badg	| dense_rank |
|-------- |------ |----------- |
| 22656	| 149	| 1 |
| 34509	| 45	| 2 |
| 1288	| 40	| 3 |
| 5190	| 31	| 4 |
|...| 
___

8. Сколько в среднем очков получает пост каждого пользователя?
Вывести: заголовок поста; идентификатор пользователя; число очков поста;
среднее число очков пользователя за пост, округлённое до целого числа.
Не учитывать посты без заголовка, а также те, что набрали ноль очков.

```SQL
select title,
       user_id,
       score,
       round(avg(score) over(partition by user_id))
from stackoverflow.posts
where title IS NOT NULL and score !=0;
```

Результат: 
| title	| user_id	| score	| round |
|------ |-------- |------ |------ |
| Diagnosing Deadlocks in SQL Server 2005	| 1	| 82	| 573 |
| How do I calculate someone's age in C#?	| 1	| 1743	| 573 |
| Why doesn't IE7 copy 'pre''code' blocks to the clipboard correctly?	| 1	| 37	| 573 |
|...| 
___

9. Отобразить заголовки постов, которые были написаны пользователями, получившими более 1000 значков.

```SQL
select p.title
from stackoverflow.posts p
join stackoverflow.users u on p.user_id = u.id
join stackoverflow.badges b on u.id = b.user_id
where p.title is not null
group by p.title
having count(b.id) > 1000;
```

Результат: 
| title |
|------ |
| Project management to go with GitHub |
| What are the correct version numbers for C#? |
| What's the hardest or most misunderstood aspect of LINQ? |
| What's the strangest corner case you've seen in C# or .NET? |
___

10. Написать запрос, который выгрузит данные о пользователях из Канады (англ. Canada).
    Разделите пользователей на три группы в зависимости от количества просмотров их профилей:
- пользователям с числом просмотров больше либо равным 350 присвойте группу 1;
- пользователям с числом просмотров меньше 350, но больше либо равно 100 — группу 2;
- пользователям с числом просмотров меньше 100 — группу 3.
Отобразить в итоговой таблице идентификатор пользователя, количество просмотров профиля и группу. 

```SQL
SELECT id,
      views,
      CASE
          WHEN views >= 350 THEN 1
          WHEN views < 350 AND views >= 100 THEN 2
          ELSE 3                    
      END as category
FROM stackoverflow.users
WHERE location LIKE '%Canada%' AND views !=0;
```

Результат: 
| id	| views	| category |
|---- |------ |--------- |
| 22	| 1079	| 1 |
| 34	| 1707	| 1 |
| 37	| 757	| 1 |
| 41	| 174	| 2 |
|...| 
___

11. Дополнить предыдущий запрос. Отобразить лидеров каждой группы — пользователей, которые набрали максимальное число просмотров в своей группе.
    Вывести поля с идентификатором пользователя, группой и количеством просмотров.

```SQL
with user_cat as (select id,
                        views,
                        case
                            when views >= 350 then 1
                            when views < 350 AND views >= 100 then 2
                            else 3  
                        end as category
                  from stackoverflow.users
                  where location LIKE '%Canada%' and views !=0),
     max_v as (select id,
                      views,
                      category,
                      max(views) over(partition by category) as max_views
               from  user_cat) 
select id,
        category,
        views
from max_v
where views = max_views 
order by views DESC, id;
```

Результат: 
| id	| category	| views |
|---- |---------- |------ |
| 3153	| 1	| 21991 |
| 46981	| 2	| 349 |
| 3444	| 3	| 99 |
| 22273	| 3	| 99 |
|...| 
___

12. Посчитать ежедневный прирост новых пользователей в ноябре 2008 года. Сформируйте таблицу с полями:
номер дня; число пользователей, зарегистрированных в этот день; сумму пользователей с накоплением.

```SQL
with user_per_day as (select extract(DAY from creation_date::date) as day_year,
                             count(id) as user_cnt
                      from stackoverflow.users
                      where creation_date::date between '2008-11-01' and '2008-11-30'
                      group by extract(DAY from creation_date::date))
select day_year,
        user_cnt,
        sum(user_cnt) over(order by day_year)
from user_per_day;
```

Результат: 
| day_year	| user_cnt	| sum |
|---------- |---------- |---- |
| 1	| 34	| 34 |
| 2	| 48	| 82 |
| 3	| 75	| 157 |
| 4	| 192	| 349 |
|...|
___

13. Для каждого пользователя, который написал хотя бы один пост, найти интервал между регистрацией и временем создания первого поста.
    Отобразить: идентификатор пользователя; разницу во времени между регистрацией и первым постом.

```SQL
select user_id,
       created_post - creation_date as interval_date
from stackoverflow.users u
join (select user_id,
              creation_date as created_post,
              row_number() over(partition by user_id order by creation_date) as post_rang
      from stackoverflow.posts) as abc on u.id = abc.user_id
where post_rang = 1;
```

Результат: 
| user_id	| interval_date |
|-------- |-------------- |
| 1	| 9:18:29 |
| 2	| 14:37:03 |
| 3	| 3 days, 16:17:09 |
| 4	| 15 days, 5:44:22 |
|...|
___

14. Вывести общую сумму просмотров у постов, опубликованных в каждый месяц 2008 года.
    
```SQL
select cast(date_trunc('month',creation_date) as date), sum(views_count)
from stackoverflow.posts
where extract(year from cast(creation_date as date)) = 2008
group by cast(date_trunc('month',creation_date) as date)
order by sum(views_count) desc;
```

Результат: 
|date_trunc	| sum|
|---------- |--- |
| 2008-09-01	| 452928568 |
| 2008-10-01	| 365400138 |
| 2008-11-01	| 221759651 |
|...|
___

15. Вывести имена самых активных пользователей, которые в первый месяц после регистрации (включая день регистрации) дали больше 100 ответов.
    Для каждого имени пользователя вывести количество уникальных значений user_id.
    
```SQL
select u.display_name as user_name,
       count(distinct p.user_id) as user_cnt
from stackoverflow.users u
join stackoverflow.posts p on u.id = p.user_id
where p.post_type_id = 2 and p.creation_date::date <= u.creation_date::date + interval '1 month'
group by user_name
having count(p.id) > 100
order by user_name;
```

Результат: 
| user_name	| user_cnt |
|---------- |--------- |
| 1800 INFORMATION |	1 |
| Adam Bellaire	| 1 |
| Adam Davis	| 1 |
|...|
___

16. Вывести количество постов за 2008 год по месяцам. Отобрать посты от пользователей, которые зарегистрировались в сентябре 2008 года
    и сделали хотя бы один пост в декабре того же года. Отсортировать таблицу по значению месяца по убыванию.
    
```SQL
select date_trunc('month', p.creation_date)::date,
       count(p.id)
from stackoverflow.posts p
where p.user_id in (select u.id
                    from stackoverflow.posts p
                    join stackoverflow.users u on p.user_id = u.id
                    where (u.creation_date::date between '2008-09-01' and '2008-09-30') 
                            and (p.creation_date::date between '2008-12-01' and '2008-12-31'))
group by date_trunc('month', p.creation_date)::date
order by date_trunc('month', p.creation_date)::date desc;
```

Результат: 
| date_trunc	| count |
|------------ |------ |
| 2008-12-01	| 17641 |
| 2008-11-01	| 18294 |
| 2008-10-01	| 27171 |
|...|
___

17. Используя данные о постах, вывести несколько полей:
идентификатор пользователя, который написал пост; дата создания поста; количество просмотров у текущего поста; сумма просмотров постов автора с накоплением.
Данные в таблице должны быть отсортированы по возрастанию идентификаторов пользователей, а данные об одном и том же пользователе — по возрастанию даты создания поста.
    
```SQL
select user_id,
       creation_date,
       views_count,
       sum(views_count) over(partition by user_id order by creation_date)
from stackoverflow.posts
order by user_id, creation_date;
```

Результат: 
| user_id	| creation_date	| views_count	| sum |
|-------- |-------------- |------------ |---- |
| 1	| 2008-07-31 | 23:41:00	| 480476	| 480476 |
| 1	| 2008-07-31 | 23:55:38	| 136033	| 616509 |
| 1	| 2008-07-31 | 23:56:41	| 0	| 616509 |
|...|
___

18. Сколько в среднем дней в период с 1 по 7 декабря 2008 года включительно пользователи взаимодействовали с платформой?
    Для каждого пользователя отобрать дни, в которые он или она опубликовали хотя бы один пост.
    
```SQL
with this as (select u.id, count(distinct p.creation_date::date) as activedays
              from stackoverflow.users u 
              join stackoverflow.posts p on u.id = p.user_id
              where p.creation_date::date between '2008-12-01' and '2008-12-07'
              group by u.id)
select round(avg(activedays))
from this;
```

Результат: 
| round	|
|------ |
| 2	| 
___

19. На сколько процентов менялось количество постов ежемесячно с 1 сентября по 31 декабря 2008 года? Отобразите таблицу со следующими полями:
Номер месяца. Количество постов за месяц. Процент, который показывает, насколько изменилось количество постов в текущем месяце по сравнению с предыдущим.
Округлить значение процента до двух знаков после запятой.
    
```SQL
with this as (
    select extract(month from creation_date::date) AS month_date,
           COUNT(DISTINCT id) AS post_month
    from stackoverflow.posts
    where creation_date::date between '2008-09-01' and '2008-12-31'
    group by month_date
)
select *,
       round(((post_month::numeric/LAG(post_month) OVER(ORDER BY month_date))-1)*100,2) AS post_this_month
from this;
```

Результат: 
| month_date	| post_month	| post_this_month |
|------------ |------------ |---------------- |
| 9	| 70371	| |
| 10	| 63102	| -10.33 |
| 11	| 46975	| -25.56 |
| 12	| 44592	| -5.07 |
___

20. Найти пользователя, который опубликовал больше всего постов за всё время с момента регистрации.
    Вывести данные его активности за октябрь 2008 года в таком виде: номер недели; дата и время последнего поста, опубликованного на этой неделе.
    
```SQL
select distinct month_post,
       max(creation_date) over(partition by month_post)
from (select creation_date,
       extract(week from creation_date::date) as month_post
from stackoverflow.posts
where user_id in (select user_id
                  from stackoverflow.posts
                  group by user_id
                  order by count(id) desc
                  limit 1) and creation_date::date between '2008-10-01' and '2008-10-31') as nnn;
```

Результат: 
| month_post	| max |
|------------ |---- |
| 40	| 2008-10-05 09:00:58 |
| 41	| 2008-10-12 21:22:23 |
| 42	| 2008-10-19 06:49:30 |
| 43	| 2008-10-26 21:44:36 |
| 44	| 2008-10-31 22:16:01 |
___
