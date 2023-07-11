[BigQuery Basics](https://www.ga4bigquery.com/sessions-dimensions-metrics-ga4/)

## 특정 기간 조회

```SQL
-- Example: Query a specific date range for selected events.
--
-- Counts unique events by date and by event name for a specifc period of days and
-- selected events(page_view, session_start, and purchase).

SELECT
  event_date,
  event_name,
  COUNT(*) AS event_count
FROM
  -- Replace table name.
  `bigquery-public-data.ga4_obfuscated_sample_ecommerce.events_*`
WHERE
  event_name IN ('page_view', 'session_start', 'purchase')
  -- Replace date range.
  AND _TABLE_SUFFIX BETWEEN '20201201' AND '20201202'
GROUP BY 1, 2;
```

## 사용자 수 및 신규 사용자 수

```SQL
-- Example: Get 'Total User' count and 'New User' count.

WITH
  UserInfo AS (
    SELECT
      user_pseudo_id,
      MAX(IF(event_name IN ('first_visit', 'first_open'), 1, 0)) AS is_new_user
    -- Replace table name.
    FROM `bigquery-public-data.ga4_obfuscated_sample_ecommerce.events_*`
    -- Replace date range.
    WHERE _TABLE_SUFFIX BETWEEN '20201101' AND '20201130'
    GROUP BY 1
  )
SELECT
  COUNT(*) AS user_count,
  SUM(is_new_user) AS new_user_count
FROM UserInfo;
```

## 구매자 유형당 평균 거래수

```SQL
-- Example: Average number of transactions per purchaser.

SELECT
  COUNT(*) / COUNT(DISTINCT user_pseudo_id) AS avg_transaction_per_purchaser
FROM
  -- Replace table name.
  `bigquery-public-data.ga4_obfuscated_sample_ecommerce.events_*`
WHERE
  event_name IN ('in_app_purchase', 'purchase')
  -- Replace date range.
  AND _TABLE_SUFFIX BETWEEN '20201201' AND '20201231';
```

## 특정 이벤트 매개변수 값 조회

```SQL
-- Example: Query values for a specific event name.
--
-- Queries the individual timestamps and values for all 'purchase' events.

SELECT
  event_timestamp,
  (
    SELECT COALESCE(value.int_value, value.float_value, value.double_value)
    FROM UNNEST(event_params)
    WHERE key = 'value'
  ) AS event_value
FROM
  -- Replace table name.
  `bigquery-public-data.ga4_obfuscated_sample_ecommerce.events_*`
WHERE
  event_name = 'purchase'
  -- Replace date range.
  AND _TABLE_SUFFIX BETWEEN '20201201' AND '20201202';
```

## 장바구니에 추가된 상위 10개 상품

```SQL
-- Example: Top 10 items added to cart by most users.

SELECT
  item_id,
  item_name,
  COUNT(DISTINCT user_pseudo_id) AS user_count
FROM
  -- Replace table name.
  `bigquery-public-data.ga4_obfuscated_web_ecommerce.events_*`, UNNEST(items)
WHERE
  -- Replace date range.
  _TABLE_SUFFIX BETWEEN '20201101' AND '20210131'
  AND event_name IN ('add_to_cart')
GROUP BY
  1, 2
ORDER BY
  user_count DESC
LIMIT 10;
```

## 구매자 유형별 평균 제품 페이지 조회수

```SQL
-- Example: Average number of pageviews by purchaser type.

WITH
  UserInfo AS (
    SELECT
      user_pseudo_id,
      COUNTIF(event_name = 'page_view') AS page_view_count,
      COUNTIF(event_name IN ('in_app_purchase', 'purchase')) AS purchase_event_count
    -- Replace table name.
    FROM `bigquery-public-data.ga4_obfuscated_sample_ecommerce.events_*`
    -- Replace date range.
    WHERE _TABLE_SUFFIX BETWEEN '20201201' AND '20201202'
    GROUP BY 1
  )
SELECT
  (purchase_event_count > 0) AS purchaser,
  COUNT(*) AS user_count,
  SUM(page_view_count) AS total_page_views,
  SUM(page_view_count) / COUNT(*) AS avg_page_views,
FROM UserInfo
GROUP BY 1;
```

## 페이지 조회 시퀀스

```SQL
-- Example: Sequence of pageviews.

SELECT
  user_pseudo_id,
  event_timestamp,
  (SELECT value.int_value FROM UNNEST(event_params) WHERE key = 'ga_session_id') AS ga_session_id,
  (SELECT value.string_value FROM UNNEST(event_params) WHERE key = 'page_location')
    AS page_location,
  (SELECT value.string_value FROM UNNEST(event_params) WHERE key = 'page_title') AS page_title
FROM
  -- Replace table name.
  `bigquery-public-data.ga4_obfuscated_sample_ecommerce.events_*`
WHERE
  event_name = 'page_view'
  -- Replace date range.
  AND _TABLE_SUFFIX BETWEEN '20201201' AND '20201202'
ORDER BY
  user_pseudo_id,
  ga_session_id,
  event_timestamp ASC;
```

## 이벤트 매개변수 목록

```SQL
-- Example: List all available event parameters and count their occurrences.

SELECT
  EP.key AS event_param_key,
  COUNT(*) AS occurrences
FROM
  -- Replace table name.
  `bigquery-public-data.ga4_obfuscated_sample_ecommerce.events_*`, UNNEST(event_params) AS EP
WHERE
  -- Replace date range.
  _TABLE_SUFFIX BETWEEN '20201201' AND '20201202'
GROUP BY
  event_param_key
ORDER BY
  event_param_key ASC;
```

## 특정 제품을 구매한 고객이 구매한 제품

```SQL
-- Example: Products purchased by customers who purchased a specific product.
--
-- `Params` is used to hold the value of the selected product and is referenced
-- throughout the query.

WITH
  Params AS (
    -- Replace with selected item_name or item_id.
    SELECT 'Google Navy Speckled Tee' AS selected_product
  ),
  PurchaseEvents AS (
    SELECT
      user_pseudo_id,
      items
    FROM
      -- Replace table name.
      `bigquery-public-data.ga4_obfuscated_sample_ecommerce.events_*`
    WHERE
      -- Replace date range.
      _TABLE_SUFFIX BETWEEN '20201101' AND '20210131'
      AND event_name = 'purchase'
  ),
  ProductABuyers AS (
    SELECT DISTINCT
      user_pseudo_id
    FROM
      Params,
      PurchaseEvents,
      UNNEST(items) AS items
    WHERE
      -- item.item_id can be used instead of items.item_name.
      items.item_name = selected_product
  )
SELECT
  items.item_name AS item_name,
  SUM(items.quantity) AS item_quantity
FROM
  Params,
  PurchaseEvents,
  UNNEST(items) AS items
WHERE
  user_pseudo_id IN (SELECT user_pseudo_id FROM ProductABuyers)
  -- item.item_id can be used instead of items.item_name
  AND items.item_name != selected_product
GROUP BY 1
ORDER BY item_quantity DESC;
```

## 사용자별 구매 세션당 평균 지출 금액

```SQL
-- Example: Average amount of money spent per purchase session by user.

SELECT
  user_pseudo_id,
  COUNT(
    DISTINCT(SELECT EP.value.int_value FROM UNNEST(event_params) AS EP WHERE key = 'ga_session_id'))
    AS session_count,
  AVG(
    (
      SELECT COALESCE(EP.value.int_value, EP.value.float_value, EP.value.double_value)
      FROM UNNEST(event_params) AS EP
      WHERE key = 'value'
    )) AS avg_spend_per_session_by_user,
FROM
  -- Replace table name.
  `bigquery-public-data.ga4_obfuscated_sample_ecommerce.events_*`
WHERE
  event_name = 'purchase'
  -- Replace date range.
  AND _TABLE_SUFFIX BETWEEN '20201101' AND '20210131'
GROUP BY
  1;
```

## 사용자의 최신 세션 ID 및 세션 번호

```SQL
-- Get the latest ga_session_id and ga_session_number for specific users during last 4 days.

-- Replace timezone. List at https://en.wikipedia.org/wiki/List_of_tz_database_time_zones.
DECLARE REPORTING_TIMEZONE STRING DEFAULT 'America/Los_Angeles';

-- Replace list of user_pseudo_id's with ones you want to query.
DECLARE USER_PSEUDO_ID_LIST ARRAY<STRING> DEFAULT
  [
    '1005355938.1632145814', '979622592.1632496588', '1101478530.1632831095'];

CREATE TEMP FUNCTION GetParamValue(params ANY TYPE, target_key STRING)
AS (
  (SELECT `value` FROM UNNEST(params) WHERE key = target_key LIMIT 1)
);

CREATE TEMP FUNCTION GetDateSuffix(date_shift INT64, timezone STRING)
AS (
  (SELECT FORMAT_DATE('%Y%m%d', DATE_ADD(CURRENT_DATE(timezone), INTERVAL date_shift DAY)))
);

SELECT DISTINCT
  user_pseudo_id,
  FIRST_VALUE(GetParamValue(event_params, 'ga_session_id').int_value)
    OVER (UserWindow) AS ga_session_id,
  FIRST_VALUE(GetParamValue(event_params, 'ga_session_number').int_value)
    OVER (UserWindow) AS ga_session_number
FROM
  -- Replace table name.
  `bigquery-public-data.ga4_obfuscated_sample_ecommerce.events_*`
WHERE
  user_pseudo_id IN UNNEST(USER_PSEUDO_ID_LIST)
  AND RIGHT(_TABLE_SUFFIX, 8)
    BETWEEN GetDateSuffix(-3, REPORTING_TIMEZONE)
    AND GetDateSuffix(0, REPORTING_TIMEZONE)
WINDOW UserWindow AS (PARTITION BY user_pseudo_id ORDER BY event_timestamp DESC);
```
