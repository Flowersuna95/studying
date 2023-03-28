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
   ```
   실행 규칙 변경
   실행 정책은 신뢰하지 않는 스크립트로부터 사용자를 보호합니다.
   실행 정책을 변경하면 about_Execution_Policies 도움말 항목(https://go.microsoft.com/fwlink/?LinkID=135170)에 설명된 보안 위험에 노출될 수 있습니다.
   실행 정책을 변경하시겠습니까?
   [Y] 예(Y)   [A] 모두 예(A)   [N] 아니요(N)   [L] 모두 아니요(L)   [S] 일시 중단(S)   [?] 도움말  (기본값은 "N"):
   ```
3. `Y` 입력
