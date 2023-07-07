## 페이지 타이틀/경로별 페이지뷰 수

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
