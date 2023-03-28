## Terminal - 실행정책 막힘

Pycharm Terminal을 이용하여 가상환경 설정시 실행 정책 막힘 발생

```
이 시스템에서 스크립트를 실행할 수 없으므로 ~ venv\Scripts\activate.ps1 파일을 로드할 수 없습니다.
자세한 내용은 about_Execution_Policies(https://go.microsoft.com/fwlink/?LinkID=135170)를 참조하십시오.
    + CategoryInfo          : 보안 오류: (:) [], ParentContainsErrorRecordException
    + FullyQualifiedErrorId : UnauthorizedAccess
```

**해결방법**

1. powershell 관리자 권한 실행

2. 명령어 입력 : `Set-ExecutionPolicy Unrestricted`
