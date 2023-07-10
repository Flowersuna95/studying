## 총 거래 / 수익 수

```sql
select 
    event_date as date,
    count(distinct ecommerce.transaction_id) as transactions,
    sum(ecommerce.purchase_revenue) as purchase_revenue
from
    `project_name.analytics_.events_*`
group by 
    date
order by
    date
```

## Conversion Rate

```sql
with prep as (
select
    user_pseudo_id,
    (select value.int_value from unnest(event_params) where key = 'ga_session_id') as session_id,
    event_date,
    max((select value.string_value from unnest(event_params) where key = 'session_engaged')) as session_engaged,
    ecommerce.transaction_id,
    ecommerce.purchase_revenue
from
    `project_name.analytics_.events_*`
group by
    user_pseudo_id,
    session_id,
    event_date,
    transaction_id,
    purchase_revenue
)

select
    event_date as date,
    count(distinct transaction_id) as transactions,
    sum(purchase_revenue) as purchase_revenue,
    count(distinct transaction_id) / count(distinct concat(user_pseudo_id,session_id)) as ecommerce_conversion_rate_all_sessions,
    count(distinct transaction_id) / count(distinct case when session_engaged = '1' then concat(user_pseudo_id,session_id) else null end) as ecommerce_conversion_rate_engaged_sessions
from
    prep
group by 
    date
order by 
    date
```

## Lifetime Value

```sql
select
    user_pseudo_id,
    sum(ecommerce.purchase_revenue) as lifetime_value
from
    `project_name.analytics_.events_*`
where
    ecommerce.purchase_revenue is not null
group by 
    user_pseudo_id
order by 
    lifetime_value desc
```

## Average Lifetime Revenue

```sql
with prep as (
select
    user_pseudo_id,
    (select value.int_value from unnest(event_params) where key = 'ga_session_id') as session_id,
    parse_date('%Y%m%d',event_date) as session_date,
    first_value(parse_date('%Y%m%d',event_date)) over (partition by user_pseudo_id order by event_date) as first_session_date,
    sum(ecommerce.purchase_revenue) as revenue
from
    `project_name.analytics_.events_*`
group by
    user_pseudo_id,
    session_id,
    event_date
order by
    user_pseudo_id,
    session_id
),

prep_ltv as (
select 
    date_diff(session_date,first_session_date,day) as day,
    count(distinct user_pseudo_id) as users,
    sum(revenue) as ltv_revenue_per_day,
    sum(revenue) / max(count(distinct user_pseudo_id)) over () as avg_ltv_revenue_per_day
from 
    prep
group by 
    day
order by 
    day
)

select 
    day,
    sum(avg_ltv_revenue_per_day) over (order by day) as average_ltv_revenue
from 
    prep_ltv
group by 
    day,
    avg_ltv_revenue_per_day
order by 
    day
```
