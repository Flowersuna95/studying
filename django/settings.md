## Settings

### settings.py

- 프로젝트의 전반적인 사항들을 설정해주는 곳
- 루트 디렉토리를 포함한 각종 디렉토리 위치, 로그의 형식, 프로젝트에 포함된 애플리케이션의 이름 등이 지정

##### DEBUG

```
- True : 개발 모드
- False : 운영 모드
```
    
##### ALLOWED_HOSTS

```
- 운영 모드인 경우 반드시 서버의 IP나 도메인을 지정해야함
- 개발 모드인 경우 지정하지 않아도 ['localhost', '127.0.0.1']로 간주
```
    
##### INSTALLED_APPS

```
- 프로젝트에 포함되는 애플리케이션 등록
```

##### DATABASES

```
- 프로젝트에 사용할 데이터베이트 엔진 설정
```

##### TIME_ZONE

```
- 타임존 지정
```
