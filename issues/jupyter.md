## 대용량 ipynb 파일 열리지 않음

Jupyter 사용 시 용량이 큰 `pandas.DataFrame` 데이터 전체 출력된 ipynb 파일 오픈 시 문제 발생

<br>

**해결방법 1**

Command를 입력하여 ipynb 파일 ouputs clear

```
jupyter nbconvert --clear-output --inplace my_notebook.ipynb
```

Or

```
jupyter nbconvert --clear-output --to notebook --output=my_notebook_no_out my_notebook.ipynb
```

<br>

**해결방법 2**

Editor를 이용하여 문제가 되는 Output delete

![image](https://user-images.githubusercontent.com/126761154/231330333-0e6ee233-cdf5-4254-bbdb-2ed1265438b9.png)

