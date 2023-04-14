## URL Config

### urls.py

- URL 정의
- URL 패턴 변수 지정 시 `<type:name>` 형식 → **Path Converter**

  ```python
  path('articles/<int:year>', views.year_archive)
  ```
