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

## device별 유저 수

```SQL
select
    device.category,
    device.operating_system,
    device.operating_system_version,
    device.language,
    device.web_info.browser,
    device.web_info.browser_version,
    device.web_info.hostname,
    count(distinct user_pseudo_id) as users
from
    `project_name.analytics_.events_*`
group by 
    category,
    operating_system,
    operating_system_version,
    language,
    browser,
    browser_version,
    hostname
order by
    users desc
```

## 신규/재방문 유저 수

```SQL
with prep as (
select
    user_pseudo_id,
    (select value.int_value from unnest(event_params) where key = 'ga_session_id') as session_id,
    (select value.int_value from unnest(event_params) where key = 'ga_session_number') as session_number,
    max((select value.int_value from unnest(event_params) where key = 'engagement_time_msec')) as engagement_time_msec
from
    `project_name.analytics_.events_*`
where
    _table_suffix between '20210601' and format_date('%Y%m%d',date_sub(current_date(), interval 1 day))
group by
    user_pseudo_id,
    session_id,
    session_number
)

select
    count(distinct case when session_number = 1 and engagement_time_msec > 0 then user_pseudo_id else null end) as new_users,
    count(distinct case when session_number > 1 and engagement_time_msec > 0 then user_pseudo_id else null end) as returning_users
from 
    prep
```

## 유저별 첫 방문 일

```sql
select
    user_pseudo_id,
    (select value.int_value from unnest(event_params) where key = 'ga_session_id') as session_id,
    (select value.int_value from unnest(event_params) where key = 'ga_session_number') as session_number,
    max((select value.int_value from unnest(event_params) where key = 'engagement_time_msec')) as engagement_time_msec,
    parse_date('%Y%m%d',event_date) as session_date,
    first_value(parse_date('%Y%m%d',event_date)) over (partition by user_pseudo_id order by event_date) as first_session_date
from
    `project_name.analytics_.events_*`
where
    _table_suffix between format_date('%Y%m%d',date_sub(current_date(), interval 43 day)) and format_date('%Y%m%d',date_sub(current_date(), interval 1 day))
group by
    user_pseudo_id,
    session_id,
    session_number,
    event_date
order by
    user_pseudo_id,
    session_id,
    session_number
```

## (재방문 날짜 - 첫 방문 날짜)별 신규/재방문 유저 수

```sql
with prep as (
select
    user_pseudo_id,
    (select value.int_value from unnest(event_params) where key = 'ga_session_id') as session_id,
    (select value.int_value from unnest(event_params) where key = 'ga_session_number') as session_number,
    max((select value.int_value from unnest(event_params) where key = 'engagement_time_msec')) as engagement_time_msec,
    parse_date('%Y%m%d',event_date) as session_date,
    first_value(parse_date('%Y%m%d',event_date)) over (partition by user_pseudo_id order by event_date) as first_session_date
from
    `project_name.analytics_.events_*`
where
    _table_suffix between format_date('%Y%m%d',date_sub(current_date(), interval 43 day)) and format_date('%Y%m%d',date_sub(current_date(), interval 1 day))
group by
    user_pseudo_id,
    session_id,
    session_number,
    event_date
order by
    user_pseudo_id,
    session_id,
    session_number
)

select 
    date_diff(session_date,first_session_date,day) as day,
    count(distinct case when session_number = 1 and engagement_time_msec > 0 then user_pseudo_id else null end) as new_users,
    count(distinct case when session_number > 1 and engagement_time_msec > 0 then user_pseudo_id else null end) as returning_users,
    count(distinct case when session_number > 1 and engagement_time_msec > 0 then user_pseudo_id else null end) / max(count(distinct case when session_number = 1 and engagement_time_msec > 0 then user_pseudo_id else null end)) over () as retention_percentage
from 
    prep
group by 
    day
order by 
    day
```

## (재방문 날짜 - 첫 방문 날짜) 주차별 재방문율

```sql
with prep as (
select
    user_pseudo_id,
    (select value.int_value from unnest(event_params) where key = 'ga_session_id') as session_id,
    (select value.int_value from unnest(event_params) where key = 'ga_session_number') as session_number,
    max((select value.int_value from unnest(event_params) where key = 'engagement_time_msec')) as engagement_time_msec,
    parse_date('%Y%m%d',event_date) as session_date,
    first_value(parse_date('%Y%m%d',event_date)) over (partition by user_pseudo_id order by event_date) as first_session_date
from
    `project_name.analytics_.events_*`
where
    _table_suffix between format_date('%Y%m%d',date_sub(current_date(), interval 100 day)) and format_date('%Y%m%d',date_sub(current_date(), interval 1 day))
group by
    user_pseudo_id,
    session_id,
    session_number,
    event_date
order by
    user_pseudo_id,
    session_id,
    session_number
)

select
    distinct concat(extract(isoyear from first_session_date),'-',format('%02d',extract(isoweek from first_session_date))) as year_week,
    count(distinct case when date_diff(session_date,first_session_date,isoweek) = 0 and session_number >= 1 and engagement_time_msec > 0 then user_pseudo_id end) as week_0,
    count(distinct case when date_diff(session_date,first_session_date,isoweek) = 1 and session_number > 1 and engagement_time_msec > 0 then user_pseudo_id end) as week_1,
    count(distinct case when date_diff(session_date,first_session_date,isoweek) = 2 and session_number > 1 and engagement_time_msec > 0 then user_pseudo_id end) as week_2,
    count(distinct case when date_diff(session_date,first_session_date,isoweek) = 3 and session_number > 1 and engagement_time_msec > 0 then user_pseudo_id end) as week_3,
    count(distinct case when date_diff(session_date,first_session_date,isoweek) = 4 and session_number > 1 and engagement_time_msec > 0 then user_pseudo_id end) as week_4,
    count(distinct case when date_diff(session_date,first_session_date,isoweek) = 5 and session_number > 1 and engagement_time_msec > 0 then user_pseudo_id end) as week_5
from
    prep
group by 
    year_week 
order by
   year_week
```
