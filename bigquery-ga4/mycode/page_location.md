
## Page URL / Get Parameter 분리

```sql
with raw as (
        select
                (select value.string_value from unnest(event_params) where key = 'page_location') as page_location,
                --regexp_extract_all(replace((select value.string_value from unnest(event_params) where key = 'page_location'), '.html', ''), '([^?#]+)')[offset(0)] as page_path,
                --regexp_extract(replace((select value.string_value from unnest(event_params) where key = 'page_location'), '.html', ''), '([^?]*)+') as page_path,
                split(replace((select value.string_value from unnest(event_params) where key = 'page_location'), '.html', ''), '?')[OFFSET(0)] as page_path,
                split((select value.string_value from unnest(event_params) where key = 'page_location'), '?')[SAFE_OFFSET(1)] as page_parameter
        from `project_name`
)
select distinct page_path from raw
```
