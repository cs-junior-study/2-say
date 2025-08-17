# 9장 최소한 알고 있어야 할 서버 지식

생성일: 2025년 8월 6일 오후 9:56

<br>

### 사용자 권한

<br><br>

(u소유자, g그룹, o 다른 사용자, a모두 ) - (+추가, -제거, = 지정) - (rwx)

이렇게 권한을 설정할 수 있다. 

ex) g-x : 그룹에서 실행 권한 제거

<br><br>

### sudo

sudo 명령어를 통해서 실행할 수 있는 명령어는 별도 설정 파일로 관리한다.

일반적으로 /etc/sudoers 파일이나 /etc/sudoers.d 디렉토리에 위치한다.

user1 계정에 sudo 명령어로 모든 명령어 실행할 수 있는 권한 부여 예시

<br>

```java
// sudoers.d
user1 ALL=(ALL) ALL
// sudo 명령어 비번 없이
user1 ALL=(ALL) NOPASSWD: ALL
// sudo 특정 명령어만 허용
user1 ALL=(ALL) NOPASSWD: /usr/bin/systemctl
```

<br><br>

### 프로세스 확인

프로세스를 확인하기 위해서 ps aux, ps -eaf 명령어가 있다.

또한 메모리나 CPU 사용량이 높은 프로세스 상위 5개 확인하는 명령어

<br>

```java
$ ps aux --sort -rss | head -n 6
```

<br>

### 프로세스 종료

<br><br>

kill [옵션] PID

- 옵션
- -15 (기본값, TERM 신호 전달)
- -9 (강제 종료)


<br><br>

### 디스크 용량 관리

<br>

```java
// 디스크 사용량
$ df -h(MB 단위)

// 디렉토리나 파일 사용량
$ du -s(용량 합)h(MB단위) .
```

<br>

find 명령어로 오래된 파일 삭제하기

<br>

```java
$ find ./logs -mtime +29 -type f -delete
```

<br>

한 폴더안에만 파일이 급격하게 많으면 매우 느려질 수 있다.


<br><br>

### 파일 디스크립터 제한

OS는 사용자나 시스템 수준에서 생성할 수 있는 파일 디스크럽터 개수를 제한한다. 프로세스는 제한된 개수의 파일 디스크립터를 초과해서 생성할 수 없다.

따라서, 개수 제한을 변경할 수 있다.

<br>

```java
$ unlimit -n [개수] (현재 세션에만 적용)

//혹은 /etc/security/limits.conf 를 설정하면 된다.
// systemd가 가지는 제한값은 또 별도라서 설정필요하다.
```

<br>
<br>

### 크론으로 스케줄링하기

<br>

- 조회 crontab -l

<br>

*(분) *(시) *(일1-31) *(월1-12) *(요일 0:일요일)

<br>

- 일회성 작업을 스케줄링할 때에는 at을 사용한다.

<br><br>

### Alias 등록하기

서버를 운영하면 자주 사용하는 명령어가 생긴다. → alias로 번거로움 해결 가능

<br>

```java
$ alias cdweb='cd /var/www/html' // 매번 등록해야함 -> .bashrc 추가하면 됨.
```

<br>

### NC 명령어로 연결 확인하기

<br>

요즘 서버 프로그램은 다양한 내부/외부 서비스와 연동을 하는데, 종종 연결이 불안정할 때가 있다.

이럴 때는 네트워크 연결이 정상적으로 이루어지는지 확인할 필요가 있다. 

nc는 특정 포트로 연결이 잘 되는지 확인하는 것이다.

<br>

```java
$nc -z(특정 포트 열려 있는지만 확인) -v(추가 정보) www.daum.net 443 
Ncat: Version 7.92 ( https://nmap.org/ncat )
Ncat: Connected to 211.249.220.24:443.
Ncat: 0 bytes sent, 0 bytes received in 0.07 seconds.
```

<br>

### netstat 명령어로 포트 사용 확인

서버가 켜져 있는데 포트 연결이 안된다면 netstat으로 서버 포트가 열려있는지 확인할 수 있다.

<br><br>

옵션

- -l: 리스닝 서버 소켓을 출력한다.
- -p: 소켓을 사용하는 PID/프로그램 이름을 출력한다.
- -u: UDP 소켓을 출력한다.
- -t: TCP 소켓을 출력한다.
- -n: 포트나 주소로 숫자를 출력한다.

<br>

```java
$ netstat -ant | grep [포트] //현재 포트를 사용하고 있는 프로세스 정보
```
