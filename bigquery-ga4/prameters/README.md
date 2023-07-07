### GA4 스키마 공식 문서

[GA4 - BigQuery Export schema](https://support.google.com/analytics/answer/7029846?hl=en)


## Parameter

#### user_id

- 사이트 로그인 유저 ID

#### user_pseudo_id

- 유저 식별 코드로 주로 사용
- 정의 : **Cookie ID** (web browsers) 또는 **Device ID** (mobile apps) 기반 값
- 한 사람은 여러 cookie id 또는 device id를 가질 수 있음

#### user_first_touch_timestamp

- 사용자가 앱을 처음 열거나 사이트를 방문한 시간(마이크로초)

#### event_params.key = 'ga_session_id'

- 유니크한 값 아님
- 이벤트 발생 시간의 타임스탬프일 뿐이므로 세션 간 충돌 가능성 있음
- 따라서 `user_pseudo_id`와 함께 사용해야 함

#### event_params.key = 'session_engaged'

- 세션이 활성화되었는지 여부
- 정의 : 10초 이상 지속되었거나, 1개 이상의 변환 이벤트 또는 2개 이상의 페이지 뷰를 가진 세션의 수

#### event_params.key = 'engagement_time_msec'

- 세션 활성화 유지 시간

#### event_params.key = 'medium'

- 세션 트래픽 매체

#### event_params.key = 'source'

- 세션 트래픽 소스

