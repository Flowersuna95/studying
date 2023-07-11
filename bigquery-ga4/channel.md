## 유저 기반 채널(유입경로)

```sql
select
    case 
        when traffic_source.source = '(direct)' and (traffic_source.medium in ('(not set)','(none)')) then 'Direct'
        when (regexp_contains(traffic_source.source,'alibaba|amazon|google shopping|shopify|etsy|ebay|stripe|walmart')
            or regexp_contains(traffic_source.name, '^(.*(([^a-df-z]|^)shop|shopping).*)$'))
            and regexp_contains(traffic_source.medium, '^(.*cp.*|ppc|paid.*)$') then 'Paid Shopping'
        when regexp_contains(traffic_source.source,'baidu|bing|duckduckgo|ecosia|google|yahoo|yandex')
            and regexp_contains(traffic_source.medium,'^(.*cp.*|ppc|paid.*)$') then 'Paid Search'
        when regexp_contains(traffic_source.source,'badoo|facebook|fb|instagram|linkedin|pinterest|tiktok|twitter|whatsapp')
            and regexp_contains(traffic_source.medium,'^(.*cp.*|ppc|paid.*)$') then 'Paid Social'
        when regexp_contains(traffic_source.source,'dailymotion|disneyplus|netflix|youtube|vimeo|twitch|vimeo|youtube')
            and regexp_contains(traffic_source.medium,'^(.*cp.*|ppc|paid.*)$') then 'Paid Video'
        when traffic_source.medium in ('display', 'banner', 'expandable', 'interstitial', 'cpm') then 'Display'
        when regexp_contains(traffic_source.source,'alibaba|amazon|google shopping|shopify|etsy|ebay|stripe|walmart')
            or regexp_contains(traffic_source.name, '^(.*(([^a-df-z]|^)shop|shopping).*)$') then 'Organic Shopping'
        when regexp_contains(traffic_source.source,'badoo|facebook|fb|instagram|linkedin|pinterest|tiktok|twitter|whatsapp')
            or traffic_source.medium in ('social','social-network','social-media','sm','social network','social media') then 'Organic Social'
        when regexp_contains(traffic_source.source,'dailymotion|disneyplus|netflix|youtube|vimeo|twitch|vimeo|youtube')
            or regexp_contains(traffic_source.medium,'^(.*video.*)$') then 'Organic Video'
        when regexp_contains(traffic_source.source,'baidu|bing|duckduckgo|ecosia|google|yahoo|yandex')
            or traffic_source.medium = 'organic' then 'Organic Search'
        when regexp_contains(traffic_source.source,'email|e-mail|e_mail|e mail')
            or regexp_contains(traffic_source.medium,'email|e-mail|e_mail|e mail') then 'Email'
        when traffic_source.medium = 'affiliate' then 'Affiliates'
        when traffic_source.medium = 'referral' then 'Referral'
        when traffic_source.medium = 'audio' then 'Audio'
        when traffic_source.medium = 'sms' then 'SMS'
        when traffic_source.medium like '%push'
            or regexp_contains(traffic_source.medium,'mobile|notification') then 'Mobile Push Notifications'
        else 'Unassigned' end as channel_grouping,
    traffic_source.source,
    traffic_source.medium,
    traffic_source.name as campaign,
    count(distinct user_pseudo_id) as users
from
    `project_name.analytics_.events_*`
group by
    channel_grouping,
    source,
    medium,
    campaign
order by
    users desc
```

## 세션 기반 채널(유입경로)

```sql
with prep as (
    select
    user_pseudo_id,
    (select value.int_value from unnest(event_params) where key = 'ga_session_id') as session_id,
    max((select value.int_value from unnest(event_params) where key = 'engagement_time_msec')) as engagement_time_msec,
    max((select value.string_value from unnest(event_params) where key = 'session_engaged')) as session_engaged,
    ifnull(max((select value.string_value from unnest(event_params) where key = 'medium')),'(none)') as medium,
    ifnull(max((select value.string_value from unnest(event_params) where key = 'source')),'(direct)') as source,
    ifnull(max((select value.string_value from unnest(event_params) where key = 'campaign')),'(not set)') as name
from
    `project_name.analytics_.events_*`
group by
    user_pseudo_id,
    session_id)

select
    case 
        when source = '(direct)' and (medium in ('(not set)','(none)')) then 'Direct'
        when (regexp_contains(source,'alibaba|amazon|google shopping|shopify|etsy|ebay|stripe|walmart')
            or regexp_contains(name, '^(.*(([^a-df-z]|^)shop|shopping).*)$'))
            and regexp_contains(medium, '^(.*cp.*|ppc|paid.*)$') then 'Paid Shopping'
        when regexp_contains(source,'baidu|bing|duckduckgo|ecosia|google|yahoo|yandex')
            and regexp_contains(medium,'^(.*cp.*|ppc|paid.*)$')
            and lower(name) like '%simmer%' then 'Branded Paid Search'
        when regexp_contains(source,'baidu|bing|duckduckgo|ecosia|google|yahoo|yandex')
            and regexp_contains(medium,'^(.*cp.*|ppc|paid.*)$')
            and lower(name) not like '%simmer%' then 'Generic Paid Search'
        when regexp_contains(source,'baidu|bing|duckduckgo|ecosia|google|yahoo|yandex')
            and regexp_contains(medium,'^(.*cp.*|ppc|paid.*)$') then 'Paid Search'
        when regexp_contains(source,'badoo|facebook|fb|instagram|linkedin|pinterest|tiktok|twitter|whatsapp')
            and regexp_contains(medium,'^(.*cp.*|ppc|paid.*)$') then 'Paid Social'
        when regexp_contains(source,'dailymotion|disneyplus|netflix|youtube|vimeo|twitch|vimeo|youtube')
            and regexp_contains(medium,'^(.*cp.*|ppc|paid.*)$') then 'Paid Video'
        when medium in ('display', 'banner', 'expandable', 'interstitial', 'cpm') then 'Display'
        when regexp_contains(source,'alibaba|amazon|google shopping|shopify|etsy|ebay|stripe|walmart')
            or regexp_contains(name, '^(.*(([^a-df-z]|^)shop|shopping).*)$') then 'Organic Shopping'
        when regexp_contains(source,'badoo|facebook|fb|instagram|linkedin|pinterest|tiktok|twitter|whatsapp')
            or medium in ('social','social-network','social-media','sm','social network','social media') then 'Organic Social'
        when regexp_contains(source,'dailymotion|disneyplus|netflix|youtube|vimeo|twitch|vimeo|youtube')
            or regexp_contains(medium,'^(.*video.*)$') then 'Organic Video'
        when regexp_contains(source,'baidu|bing|duckduckgo|ecosia|google|yahoo|yandex')
            or medium = 'organic' then 'Organic Search'
        when regexp_contains(source,'email|e-mail|e_mail|e mail')
            or regexp_contains(medium,'email|e-mail|e_mail|e mail') then 'Email'
        when medium = 'affiliate' then 'Affiliates'
        when medium = 'referral' then 'Referral'
        when medium = 'audio' then 'Audio'
        when medium = 'sms' then 'SMS'
        when medium like '%push'
            or regexp_contains(medium,'mobile|notification') then 'Mobile Push Notifications'
        else 'Unassigned' end as channel_grouping,
    count(distinct concat(user_pseudo_id,session_id)) as sessions,
    count(distinct case when session_engaged = '1' then concat(user_pseudo_id,session_id) end) as engaged_sessions,
    count(distinct user_pseudo_id) as users,
    count(distinct case when engagement_time_msec > 0 then user_pseudo_id end) as active_users
from
    prep
group by 
    channel_grouping
order by 
    sessions desc
```
