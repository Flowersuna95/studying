## 유니크 유저 수

```SQL
select
    count(distinct user_pseudo_id) as users
from
    `project_name.analytics_.events_*`
```

## 유저별 HIT 수

```SQL
select
    user_pseudo_id as user,
    count(*) as hits
from
    `project_name.analytics_.events_*`
group by
    user
having
    user is not null
```

## 년도별 첫 방문 유저 수

```SQL
select
    extract(year from timestamp_micros(user_first_touch_timestamp)) as year_first_touch,
    count(distinct user_pseudo_id) as users
from
    `project_name.analytics_.events_*`
group by 
    year_first_touch
having 
    year_first_touch is not null
order by
    year_first_touch desc
```

## 유입 채널별 유저 수

```SQL
select
    concat(traffic_source.source, " / ", traffic_source.medium) as source_medium,
    count(distinct user_pseudo_id) as users
from
    `project_name.analytics_.events_*`
group by
    source_medium
order by
    users desc
```

```SQL
select
    traffic_source.source,
    traffic_source.medium,
    traffic_source.name as campaign,
    count(distinct user_pseudo_id) as users
from
    `project_name.analytics_.events_*`
group by 
    source,
    medium,
    campaign
order by
    users desc
```

## 지역별 유저 수

```SQL
select
    geo.continent,
    geo.country,
    nullif(geo.city,'') as city,
    count(distinct user_pseudo_id) as users
from
    `project_name.analytics_.events_*`
group by 
    continent,
    country,
    city
order by 
    users desc
```
