## 상품 종류

```sql
select 
    event_name,
    items
from 
    `project_name.analytics_.events_*`,
    unnest(items) as items
```

## 구매 상품 조회

```sql
select 
    sum(items.quantity) as items,
    sum(items.item_revenue) as item_revenue
from 
    `project_name.analytics_.events_*`,
    unnest(items) as items
where 
    event_name = 'purchase'
```

## 이벤트별 상품 종류 조회

```sql
with prep as (
select 
    event_name,
    items.item_name,
    # If items.quantity is not set, default to 1:
    sum(ifnull(items.quantity, 1)) as items
from 
    `project_name.analytics_.events_*`,
    unnest(items) as items
group by 
    event_name,
    item_name
)

select 
    item_name,
    sum(case when event_name = 'view_item' then items else 0 end) as view_item,
    sum(case when event_name = 'add_to_cart' then items else 0 end) as add_to_cart,
    sum(case when event_name = 'begin_checkout' then items else 0 end) as begin_checkout,
    sum(case when event_name = 'purchase' then items else 0 end) as purchase,
    safe_divide(sum(case when event_name = 'purchase' then items else 0 end),
                sum(case when event_name = 'view_item' then items else 0 end)) as view_to_purchase_rate
from 
    prep
group by 
    item_name
order by 
    view_item desc
```
