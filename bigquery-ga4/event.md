## 세션 수

```SQL
select
    count(event_name) as sessions
from
    `project_name.analytics_.events_`
where
    event_name = 'session_start'
```

## 날짜별 세션수

```SQL
select
    event_date,
    count(event_name) as sessions
from
    `project_name.analytics_.events_`
where
    event_name = 'session_start'
    and _table_suffix between '20210501' and '20210510'
group by
    event_date
order by
    event_date desc
```
