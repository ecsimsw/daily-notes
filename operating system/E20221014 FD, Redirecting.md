
# E20221014 Nohup과 &, FD.md
https://blogger.pe.kr/369

## File discriptor

프로세스가 파일 접근하기 위한 추상적인 키

0 : 표준 입력 (stdin)
1 : 표준 출력 (stdout)
2 : 표준 에러 (stderr)


## Redirection 
```
> :  명령의 결과를 파일에 쓰기
>> : 명령의 결과를 파일에 붙여 쓰기
< : 파일의 내용을 입력으로 함
```



## 

cat/etc/hosts 명령은 cat 명령에게 /etc/hsots 파일을 열도록 지정한 것이다.

하지만 cat < /etc/hosts 명령은 아규먼트 없이 실행될 경우 사용하도록 지정된 표준입력장치인 키보드(standard input)를 /etc/hsots 라는 파일로 바꾸도록 지정하여 파일의 내용을 표준입력으로 리다이렉트 받아 실행한다.

