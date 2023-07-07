## 페이지별 페이지뷰 수

```SQL
select
    (select value.string_value from unnest(event_params) where key = 'page_title') as page_title,
    (select value.string_value from unnest(event_params) where key = 'page_location') as page,
    count(event_name) as pageviews
from
    `project_name.analytics_.events_*`
where
    event_name = 'page_view'
group by 
    page_title,
    page
order by 
    pageviews desc
```

## 페이지별 페이지뷰 수, 고유 페이지뷰 수

```SQL
with prep as (
select
    user_pseudo_id,
    (select value.int_value from unnest(event_params) where event_name = 'page_view' and key = 'ga_session_id') as session_id,
    (select value.string_value from unnest(event_params) where event_name = 'page_view' and key = 'page_title') as page_title,
    (select value.string_value from unnest(event_params) where event_name = 'page_view' and key = 'page_location') as page
from
    `project_name.analytics_.events_*`
where
    event_name = 'page_view'
)

select 
    page_title,
    page,
    count(*) as total_pageviews,
    count(distinct concat(user_pseudo_id,session_id)) as unique_pageviews
from 
    prep
group by 
    page_title,
    page
order by
    unique_pageviews desc
```

## 랜딩 페이지별 방문 수

```SQL
with prep as (
select
    user_pseudo_id,
    (select value.int_value from unnest(event_params) where key = 'ga_session_id') as session_id,
    (select value.string_value from unnest(event_params) where key = 'page_location') as page,
    case when (select value.int_value from unnest(event_params) where key = 'entrances') = 1 then true else false end as landing_page
from
    `project_name.analytics_.events_*`
where
    event_name = 'page_view'
)

select 
    case when landing_page is true then page else null end as landing_page,
    count(distinct concat(user_pseudo_id,session_id)) as entrances
from 
    prep
group by 
    landing_page
having 
    landing_page is not null
order by
    entrances desc
```

## 이탈 페이지 조회

```SQL
with prep as (
select
    user_pseudo_id,
    (select value.int_value from unnest(event_params) where event_name = 'page_view' and key = 'ga_session_id') as session_id,
    (select value.string_value from unnest(event_params) where event_name = 'page_view' and key = 'page_location') as page,
    event_timestamp
from
    `project_name.analytics_.events_*`
where
    event_name = 'page_view'
order by 
    event_timestamp
)

select 
    user_pseudo_id,
    session_id,
    page,
    event_timestamp,
    first_value(page,event_timestamp) over (partition by user_pseudo_id,session_id order by event_timestamp desc) as exit_page
from 
    prep 
order by 
    event_timestamp
```

## 페이지 경로

```SQL
with prep as (
select
    user_pseudo_id,
    (select value.int_value from unnest(event_params) where event_name = 'page_view' and key = 'ga_session_id') as session_id,
    (select value.string_value from unnest(event_params) where event_name = 'page_view' and key = 'page_location') as page,
    event_timestamp
from
    `project_name.analytics_.events_*`
where
    event_name = 'page_view'
)

select
    user_pseudo_id,
    session_id,
    lag(page,1) over (partition by user_pseudo_id,session_id order by event_timestamp asc) as previous_page,
    page,
    lead(page,1) over (partition by user_pseudo_id,session_id order by event_timestamp asc) as next_page,
    event_timestamp
from 
    prep
```

## 페이지 경로별 방문(토탈유저) 수

```SQL
with prep as (
select
    user_pseudo_id,
    (select value.int_value from unnest(event_params) where event_name = 'page_view' and key = 'ga_session_id') as session_id,
    (select value.string_value from unnest(event_params) where event_name = 'page_view' and key = 'page_location') as page,
    event_timestamp
from
    `project_name.analytics_.events_*`
where
    event_name = 'page_view'
),

prep_navigation as (
select
    user_pseudo_id,
    session_id,
    lag(page,1) over (partition by user_pseudo_id,session_id order by event_timestamp asc) as previous_page,
    page,
    lead(page,1) over (partition by user_pseudo_id,session_id order by event_timestamp asc) as next_page,
    event_timestamp
from 
    prep
)

select 
    ifnull(previous_page,'(entrance)') as previous_page,
    page,
    ifnull(next_page,'(exit)') as next_page,
    count(distinct concat(user_pseudo_id, session_id)) as count
from 
    prep_navigation
group by 
    previous_page,
    page,
    next_page
having 
    page not in (previous_page,next_page)
order by 
    count desc
```
