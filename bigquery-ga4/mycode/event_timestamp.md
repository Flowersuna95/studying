

## 타임존 변경

```
select 
    extract(datetime from TIMESTAMP_MICROS(event_timestamp) at time zone 'Asia/Jakarta'),
    extract(date from TIMESTAMP_MICROS(event_timestamp) at time zone 'Asia/Jakarta'),
    extract(time from TIMESTAMP_MICROS(event_timestamp) at time zone 'Asia/Jakarta'),
    FORMAT_TIME('%T', TIME(TIMESTAMP_MICROS(event_timestamp))) time
from `project_name`
```
