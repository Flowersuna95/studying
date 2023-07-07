## 세션 수

```SQL
select
    count(event_name) as sessions
from
    `project_name.analytics_.events_*`
where
    event_name = 'session_start'
```

## 날짜별 세션 수

```SQL
select
    event_date,
    count(event_name) as sessions
from
    `project_name.analytics_.events_*`
where
    event_name = 'session_start'
    and _table_suffix between '20210501' and '20210510'
group by
    event_date
order by
    event_date desc
```

## 동적 세션 수

```SQL
select
    event_date,
    count(event_name) as sessions
from
    `project_name.analytics_.events_*`
where
    event_name = 'session_start'
    and _table_suffix between '20210501' and format_date('%Y%m%d', date_sub(current_date(), interval 1 day))
group by
    event_date
order by
    event_date desc
```

## 주차별 세션수

```SQL
select
    event_date as date,
    case
        when extract(dayofweek from parse_date('%Y%m%d',event_date)) in (1,7) then 'weekend'
        else 'week' end as part_of_week,
    count(event_name) as sessions
from
    `project_name.analytics_.events_*`
where
    _table_suffix between '20210501' and format_date('%Y%m%d',date_sub(current_date(), interval 1 day))
    and event_name = 'session_start'
group by 
    date
order by
    date desc
```

## 요일별 시간대별 세션수

```SQL
select
    case
        when extract(dayofweek from parse_date('%Y%m%d',event_date)) in (1,7) then 'weekend'
        else 'week' end as part_of_week,
    case
        when extract(hour from timestamp_micros(event_timestamp)) between 0 and 5 then 'night'
        when extract(hour from timestamp_micros(event_timestamp)) between 6 and 11 then 'morning'
        when extract(hour from timestamp_micros(event_timestamp)) between 12 and 17 then 'afternoon'
        else 'evening' end as part_of_day,
    count(event_name) as sessions
from
    `project_name.analytics_.events_*`
where
    _table_suffix between '20210501' and format_date('%Y%m%d',date_sub(current_date(), interval 1 day))
    and event_name = 'session_start'
group by 
    part_of_week,
    part_of_day
order by
    sessions desc
```
