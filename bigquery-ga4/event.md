## 세션 수

```SQL
select
    count(event_name) as sessions
from
    `project_name.analytics_.events_`
where
    event_name = 'session_start'
```
