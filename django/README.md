# DJango


### 프로젝트 생성

```shell
$ django-admin startproject mysite
```


### 애플리케이션 생성

```shell
$ python manage.py startapp myapp
```


### DB 마이그레이션

: DB 변경사항 반영

```shell
$ python manage.py makemigrations
$ python manage.py migrate
```


### 웹 서버 실행

```shell
$ python manage.py runserver 0.0.0.0:8000
```

- 서버 실제 IP 주소와 포트 번호 입력
    - IP 주소에 `0.0.0.0`를 입력하면 서버 실제 IP 주소와 무관하게 모든 웹 접속 요청을 받는다는 의미


### Admin 사이트 관리자(슈퍼유저) 생성

```shell
$ python manage.py createsuperuser
```

- Admin 화면에서 기본적으로 User와 Groups 테이블이 보임
    - settings.py 파일에 `django.contrib.auth` 애플리케이션이 등록되어 있기 때문
