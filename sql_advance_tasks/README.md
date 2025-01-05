# Общая информация

Имеется база данных StackOverflow — сервиса вопросов и ответов о программировании. В нашем распоряжении версия базы, где хранятся данные о постах за 2008 год, 
но таблицы также содержат информацию о более поздних оценках, которые эти посты получили.

# ER-диаграмма базы данных:
![](https://github.com/knpanasik/my_portfolio/blob/additional/sql_advance_tasks/ER_StackOverflow.png)

Stackoverflow.badges хранит информацию о значках, которые присуждаются за разные достижения. <br/>
Stackoverflow.post_types содержит информацию о типе постов. <br/>
Stackoverflow.posts - информацию о постах. <br/>
Stackoverflow.users - о пользователях. <br/>
Stackoverflow.vote_types - о типах голосов. <br/>
Stackoverflow.votes - о голосах за посты. 
