## 유저별 활성 세션 여부

```SQL
select
    user_pseudo_id,
    (select value.int_value from unnest(event_params) where key = 'ga_session_id') as session_id,
    max((select value.string_value from unnest(event_params) where key = 'session_engaged')) as session_engaged
from
    `project_name.analytics_.events_*`
group by 
    user_pseudo_id,
    session_id
```

## 토탈 유저(세션) 수, 활성 세션 수, 유저 수, 활성 유저 수

```SQL
with prep as (
select
    user_pseudo_id,
    (select value.int_value from unnest(event_params) where key = 'ga_session_id') as session_id,
    max((select value.string_value from unnest(event_params) where key = 'session_engaged')) as session_engaged,
    max((select value.int_value from unnest(event_params) where key = 'engagement_time_msec')) as engagement_time_msec
from
    `project_name.analytics_.events_*`
group by 
    user_pseudo_id,
    session_id
)

select 
    count(distinct concat(user_pseudo_id,session_id)) as sessions,
    count(distinct case when session_engaged = '1' then concat(user_pseudo_id,session_id) else null end) as engaged_sessions,
    count(distinct user_pseudo_id) as users,
    count(distinct case when engagement_time_msec > 0 then user_pseudo_id else null end) as active_users
from 
    prep
```

## 세션 트래픽별 토탈 유저(세션) 수, 활성 세션 수, 유저 수, 활성 유저 수

```SQL
with prep as (
select
    user_pseudo_id,
    (select value.int_value from unnest(event_params) where key = 'ga_session_id') as session_id,
    max((select value.int_value from unnest(event_params) where key = 'engagement_time_msec')) as engagement_time_msec,
    max((select value.string_value from unnest(event_params) where key = 'session_engaged')) as session_engaged,
    max((select value.string_value from unnest(event_params) where key = 'medium')) as medium,
    max((select value.string_value from unnest(event_params) where key = 'source')) as source,
from
    `project_name.analytics_.events_*`
group by
    user_pseudo_id,
    session_id
)

select
    concat(ifnull(source,'(direct)'),' / ',ifnull(medium,'(none)')) as session_source_medium,
    count(distinct concat(user_pseudo_id,session_id)) as sessions,
    count(distinct case when session_engaged = '1' then concat(user_pseudo_id,session_id) end) as engaged_sessions,
    count(distinct user_pseudo_id) as users,
    count(distinct case when engagement_time_msec > 0 then user_pseudo_id end) as active_users
from
    prep
group by 
    session_source_medium
order by 
    sessions desc
```
