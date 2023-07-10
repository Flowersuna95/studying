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

## 스크롤 이벤트 조회

```SQL
-- METHOD 1: Query all scroll events
select
    (select value.string_value from unnest(event_params) where event_name = 'scroll' and key = 'page_location') as scroll_page,
    countif(event_name = 'scroll') as scrolls
from 
    `simoahava-com.analytics_206575074.events_20210601`
group by 
    scroll_page
order by 
    scrolls desc

-- METHOD 2: Query only scrolls that reach a certain threshold
select
    case 
        when (select value.int_value from unnest(event_params) where event_name = 'scroll' and key = 'percent_scrolled') = 90
        then (select value.string_value from unnest(event_params) where event_name = 'scroll' and key = 'page_location') else null end as scroll_page_90_percent,
    countif(event_name = 'scroll') as scrolls
from 
    `project_name.analytics_.events_*`
group by 
    scroll_page_90_percent
order by 
    scrolls desc
```

## 클릭 이벤트 조회

```SQL
select
    (select value.string_value from unnest(event_params) where event_name = 'click' and key = 'page_location') as page,
    (select value.string_value from unnest(event_params) where event_name = 'click' and key = 'link_domain') as link_domain,
    (select value.string_value from unnest(event_params) where event_name = 'click' and key = 'link_url') as link_url,
    countif(event_name = 'click' and (select value.string_value from unnest(event_params) where event_name = 'click' and key = 'outbound') = 'true') as clicks
from 
    `project_name.analytics_.events_*`
group by 
    page,
    link_domain,
    link_url
order by 
    clicks desc
```

## 검색 이벤트 조회

```SQL
select
    (select value.string_value from unnest(event_params) where event_name = 'view_search_results' and key = 'search_term') as search_term,
    countif(event_name = 'view_search_results') as searches
from 
    `project_name.analytics_.events_*`
group by 
    search_term
order by 
    searches desc
```

## 비디오 이벤트 조회

```SQL
select 
    (select value.string_value from unnest(event_params) where event_name like 'video%' and key = 'video_provider') as video_provider,
    (select value.string_value from unnest(event_params) where event_name like 'video%' and key = 'video_title') as video_title,
    (select value.string_value from unnest(event_params) where event_name like 'video%' and key = 'video_url') as video_url,
    (select value.int_value from unnest(event_params) where event_name like 'video%' and key = 'video_duration') as video_duration,
    countif(event_name = 'video_start') as video_start,
    countif(event_name = 'video_progress' and (select value.int_value from unnest(event_params) where event_name = 'video_progress' and key = 'video_percent') = 50) as video_progress_50_percent,
    countif(event_name = 'video_complete') as video_complete
from 
    `project_name.analytics_.events_*`
group by 
    video_provider,
    video_title,
    video_url,
    video_duration
order by 
    video_start desc
```

## 파일 다운로드 이벤트 조회

```SQL
select
    (select value.string_value from unnest(event_params) where event_name = 'file_download' and key = 'file_extension') as file_type,
    (select value.string_value from unnest(event_params) where event_name = 'file_download' and key = 'file_name') as file_name,
    (select value.string_value from unnest(event_params) where event_name = 'file_download' and key = 'link_text') as link_text,
    (select value.string_value from unnest(event_params) where event_name = 'file_download' and key = 'link_url') as link_url,
    countif(event_name = 'file_download') as downloads
from
    `project_name.analytics_.events_*`
group by
    file_type,
    file_name,
    link_text,
    link_url
order by
    downloads desc
```
