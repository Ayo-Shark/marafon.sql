# marafon.sql
1. Все бегуны, которые зарегистрированы на текущий марафон отображаются в таблице, как: имя, фамилия - BibNumber (CountryCode).
```sql
select distinct u."firstname", u."lastname", rnr."countrycode"
from "User" u
inner join "Runner" rnr on u."email" = rnr."email"
inner join "Role" r on u."roleid" = r."roleid"
inner join "Registration" reg on rnr."runnerid" = reg."runnerid"
inner join "RegistrationEvent" re on reg."registrationid" = re."registrationid"
inner join "Event" ev on re."eventid" = ev."eventid"
where r."roleid" = 'R' and ev."marathonid" = 5;

```
2. Таблица с результатами бегуна с предыдущих соревнований. Должен отображаться пол, возраст 
Список должен показать каждое соревнование, в котором бегун соревновался ранее.
```sql
select distinct u."firstname", u."lastname", g."gender", DATE_PART('YEAR', AGE(CURRENT_DATE, dateofbirth)) as age, ev."eventname"
from "User" u
inner join "Role" r on u."roleid" = r."roleid"
inner join "Runner" rnr on u."email" = rnr."email"
inner join "Registration" reg on rnr."runnerid" = reg."runnerid"
inner join "RegistrationEvent" re on reg."registrationid" = re."registrationid"
inner join "Event" ev on re."eventid" = ev."eventid"
inner join "Gender" g on rnr."genderid" = g."genderid"
where r."roleid" = 'R' and ev."marathonid" < 5;
```

3. Выводятся следующие поля для каждого события:
• Марафон: полное название марафона.
• Событие: полное название события.
• Время: время гонки бегуна на события в часах, минутах и секундах.
• В целом положение бегуна в марафоне среди всех участников.
• Отдельно по категории положение бегуна на событии, среди бегунов одного того же пола и той же возрастной категории.
```sql
select m."marathonname", ev."eventname", to_char( (re."racetime" ||'seconds')::interval, 'HH24:MI:SS' )
as race_time, ROW_NUMBER() OVER(PARTITION BY ev."eventname" ORDER BY re."racetime" asc) AS position
from "Event" ev
inner join "Marathon" m on ev."marathonid" = m."marathonid"
inner join "RegistrationEvent" re on ev."eventid" = re."eventid"
where ev."marathonid" = '3' and re."racetime" != 0
order by eventname asc, race_time asc
```

4.1. Необходимо выводить сводные статистические данные по всем результатам, которые соответствуют критериям:
• Всего бегунов закончило: общее количество бегунов, у которые есть зафиксированное время соревнования.
• Среднее время гонки: Среднее значение всех результатов.
```sql
select ev."eventname", count(re."racetime") as vseg_runners, to_char( (round(avg(re."racetime"), 0) ||'seconds')::interval, 'HH24:MI:SS' ) as sred_time
from "Event" ev
inner join "RegistrationEvent" re on ev."eventid" = re."eventid"
where re."racetime" != 0
group by ev."eventname"
order by eventname
```
4.2. В списке показать всех бегунов, которые соответствуют критериям поиска. Следующие поля должны быть отображены:
• Ранг: положение бегуна для выбранного события, пола и возрастной категории.
• Время гонки: время гонки бегуна в категориях в часах, минутах и секундах.
• Имя бегуна: имя бегуна.
• Страна: страна бегуна.
• Должны отображаться данные бегуна, которая содержит следующую информацию: Фамилия Имя бегуна, страна, возраст в годах полных, на дату марафона, а далее в табличной форме
```sql
select u."firstname", u."lastname",  DATE_PART('YEAR', AGE(CURRENT_DATE, dateofbirth)) as age, g."gender", m."marathonname", ev."eventname",  c."countryname", 
TO_CHAR((re."racetime" || ' seconds')::interval, 'HH24:MI:SS') as race_time,
ROW_NUMBER() OVER (PARTITION BY ev."eventname" ORDER BY re."racetime" ASC) AS position
FROM "Event" ev
INNER JOIN "Marathon" m ON ev."marathonid" = m."marathonid"
inner join "Country" c on m."countrycode" = c."countrycode"
INNER JOIN "RegistrationEvent" re ON ev."eventid" = re."eventid"
inner join "Registration" reg on re."registrationid" = reg."registrationid" 
inner join "Runner" rnr on reg."runnerid" = rnr."runnerid"
inner join "User" u on u."email" = rnr."email"
inner join "Gender" g on rnr."genderid" =  g."genderid" 
WHERE ev."marathonid" = '3' AND re."racetime" != 0
ORDER BY ev."eventname" asc, g."gender" asc, race_time asc;
```
5.1. Запрос должен позволить просматривать список всех бегунов, которые зарегистрированы на текущий марафон. Составить запрос, который может фильтровать бегунов по статусу регистрации и выбранным типам марафона, и он может сортировать все поля таблицы результатов. 
```sql
select distinct u.firstname, u.lastname, rnr.email, rgs.registrationstatus, ev.eventtypeid
from "User" u
inner join "Role" r on u.roleid = r.roleid
inner join "Runner" rnr on u.email = rnr.email
inner join "Registration" reg on rnr.runnerid = reg.runnerid
inner join "RegistrationEvent" re on reg."registrationid" = re.registrationid
inner join "Event" ev on re."eventid" = ev."eventid"
inner join "RegistrationStatus" rgs on reg.registrationstatusid = rgs.registrationstatusid
where rgs.registrationstatusid = 1
order by eventtypeid asc;
```
5.2. Общее количество бегунов, которые выведены в список, должно быть отображено над списком.
Имя, Фамилия, адрес электронной почты и регистрационный статус должны быть указаны для каждого бегуна. 
```sql
select count(*) from (select distinct u.firstname, u.lastname, rnr.email, rgs.registrationstatus
from "User" u
inner join "Role" r on u.roleid = r.roleid
inner join "Runner" rnr on u.email = rnr.email
inner join "Registration" reg on rnr.runnerid = reg.runnerid
inner join "RegistrationEvent" re on reg."registrationid" = re.registrationid
inner join "Event" ev on re."eventid" = ev."eventid"
inner join "RegistrationStatus" rgs on reg.registrationstatusid = rgs.registrationstatusid
where ev.eventtypeid = 'FM' and rgs.registrationstatusid = 1) as asd
```
6.1. Спонсоры должны быть сгруппированы по благотворительным организациям, которым они перечисляют деньги.
Также должны выводится сводные данные:
•Благотворительные организации: общее количество благотворительных организаций.
•Всего спонсорских взносов: общая сумма спонсорских пожертвований, полученная для всех благотворительных организаций. 

```sql
select count(distinct c."charityname") as charity, sum(s."amount") as vseg_spon
from "Charity" c
inner join "Registration" r on c."charityid" = r."charityid"
inner join "Sponsorship" s on r."registrationid" = s."registrationid";
```
6.2. Список должен показать наименование и общее количество спонсоров получил для каждой благотворительной организации.
```sql
select c."charityname" as charity, count(distinct s."sponsorshipid") as vseg_sponsors, sum(s."amount") AS vsego_poshertvovani       
from "Charity" c 
inner join "Registration" r on c."charityid" = r."charityid"
inner join "Sponsorship" s on r."registrationid" = s."registrationid"
group by c."charityname"  
order by charity;
```
7. Проверка бегуна по условиям:
Все поля обязательно заполнены.
Пароль должен отвечать следующим требованиям:
•Минимум 6 символов
•Минимум 1 прописная буква
•Минимум 1 цифра
•По крайней мере один из следующих символов: ! @ # $ % ^
"Дата рождения" должна быть правильной датой и бегуну должен быть не менее 10 лет на момент регистрации.
```sql
select u.password, DATE_PART('YEAR', AGE(CURRENT_DATE, dateofbirth)) as age
from "User" u
inner join "Runner" r on r.email = u.email
where u.password like '______%' and u.password ~ '[a-z]' and u.password ~ '[1-9]'
and u.password ~ '[!@#$%^]' and DATE_PART('YEAR', AGE(CURRENT_DATE, dateofbirth)) > 10;
```

