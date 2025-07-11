# 2장 느려진 서비스, 어디부터 봐야 할까

<br><br>


- 전체 처리 시간 : 348ms
- API 연동 1 (외부 네트워크에 존재하는 API 호출) : 186ms(53%)
- API 연동 2 (내부 네트워크에 존재하는 API 호출) : 44ms(13%)
- DB 연동(SQL 실행 6회): 101ms(29%)
- 로직 수행: 17ms(5%)

<br><br>

## 처리량과 응답 시간  
<br>
응답 시간의 증가는 사용자 이탈로 이어질 수 있다.   

- 서버가 동시에 처리할 수 있는 요청 수를 늘려 대기 시간 줄이기
- 처리 시간 자체를 줄여 대기 시간 줄이기
<br><br>

TPS 확인 방법  

- 스카우터, 핀포인트, 뉴렐릭 도구 사용  

<br><br>

### 서버 성능 개선 기초  

TPS를 높이려면 먼저 성능 문제가 발생하는 지점을 찾아야 한다. 이때 모니터링 도구가 유용  
 
<br><br>
### 수직 확장과 수평 확장

무턱대고 서버를 추가해서는 안 된다. DB에서 성능 문제가 발생하고 있는데 서버를 추가로 투입하면 불에 기름을 붓는 격이다.  
<br>
실제 병목 지점이 어디인지 파악하는 게 중요하다.  


<br><br><br>
### DB 커넥션 풀

db 사용 시

- DB에 연결한다.
- 쿼리를 실행한다.
- 사용이 끝나면 연결을 종료한다.     


<br><br>
연결하는 데 소요하는 시간이 매우 길다. 따라서, 이런 과정이 계속 반복된다면 매우 지연     
 
→ 커넥션 Pool 사용   

<br><br>

커넥션 풀은 다양한 설정을 제공, 이를 잘 설정하는 것도 중요    

- 커넥션 풀 크기(최소 크기, 최대 크기)
- 풀에 커넥션이 없을 때 커넥션을 구할 때까지 대기할 시간
- 커넥션의 유지 시간(최대 유휴 시간, 최대 유지 시간)

<br><br>
<aside>
📝 트래픽이 순간적으로 급증하는 패턴을 보인다면 커넥션 풀의 최소 크기를 최대 크기에 맞추는 것이 좋다. 트래픽이 점진적으로 증가할 때는 DB 연결 시간이 성능에 큰 영향을 주지 않지만 트래픽이 급증할 경우 DB 연결 시간도 성능 저하의 주요 원인이 될 수 있기 때문
</aside>
<br><br>      
그렇다고, 무턱대고 늘리면 안됨!

<br><br><br><br>

### 커넥션 대기 시간

기본 HikariCP 대기 시간은 30초. 보통 0.5 ~ 3초 지정하는 것이 사용자에게 빠르게 응답을 주고(새로고침 가능성), 서버의 부하도 줄일 수 있다.     
<br><br><br>


### 최대 유휴 시간, 유효성 검사, 최대 유지 시간

MYSQL과 같은 DB는 자동으로 연결을 끊는 기능을 제공. 아무도 안쓰는 새벽에 DB 연결이 모두 해제 → DB와 연결이 끊긴 커넥션을 사용하면 에러 발생한다. 

따라서, DB 연결 해제 시간 보다 더 작은 시간으로 커넥션을 해제하는 유휴 시간을 설정하자.

유효성 검사는 커넥션을 정상적으로 사용할 수 있는 상태인지 확인하는 절차.

최대 유지 시간은 커넥션이 생성 후 유지되는 시간을 설정할 수 있다.
<br><br><br><br>
        

### 서버 캐시

DB 서버를 확장하지 않고도 응답 시간과 처리량을 개선하고 싶다면 캐시cache 사용을 고려할 수 있다.

         
<br><br><br>
### 적중률과 삭제 규칙

- 적중률(hit rate) = 캐시에 존재한 건수 / 캐시에서 조회를 시도한 건수

적중률을 높이는 간단한 방법은 캐시에 최대한 많은 데이터를 저장하는 것이다. 하지만, 용량 한계가 있다.

<br><br><br>   

삭제할 대상을 선택할 규칙은 아래와 같다.

- LRU (Least Recently Used): 가장 오래전에 사용된 데이터를 제거
- LFU(Least Frequently Used) : 가장 적게 사용된 데이터를 유지
- FIFO

        
<br><br><br>
### 로컬 캐시와 리모트 캐시

- 로컬 캐시 : 서버 프로세스와 동일한 메모리를 캐시 저장소로 사용 (인 메모리 캐시 - but, redis와 용어 다름)
    - Caffeine(자바), node-cache(Node.js) 등이 있다.
    - 사용할 수 있는 공간 한계, 재시작 시 모두 삭제
- 리모트 캐시 : 리모트 캐시는 별도 프로세스를 캐시 저장소로 사용 - redis
    - 유연한 확장 가능, but 서버와 별도 통신 필요

         
<br><br>
캐시 사용 규모가 작고 변경 빈도가 낮다면 로컬 캐시 충분

배포 빈도가 높다면, 리모트 캐시를 사용하자. 로컬 캐시는 다시 시작하면 초기화 되기때문에,

캐시 데이터를 관리할 때, 다중 서버라면 로컬 캐시보다는 리모트 캐시를 사용하는 것이 좋다.

<br><br>

### 캐시 사전 적재

트래픽이 순간적으로 급증하는 패턴을 보인다면 캐시에 데이터를 저장하는 것도 고려할 필요가 있다. 

상황

- G 앱 사용자는 300만 명이다.
- G 앱 서비스는 사용자에게 매달 정해진 날에 이달의 요금 정보를 보여준다.
- 해당 일자가 되면 전체 회원을 대상으로 요금 안내 푸시알림을 보낸다.
- 푸시를 받은 사용자 중 일부는 G앱을 통해 이달의 요금 정보를 조회한다.

첫 접속 시, 캐시 적중률은 0%이므로 부하 증가, 전체 응답 시간 지연이 발생한다.

따라서, 이러한 상황에서 미리 넣어서 캐시 적중률을 높이는 방법도 전략이다.


<br><br>


### 가비지 컬렉터와 메모리 사용

GC가 사용하는 메모리를 늘리면 GC 시간을 줄일 수 있다. GC가 실행되는 동안 애플리케이션이 멈추는 stop the world가 발생하기 때문에, GC 시간을 줄여야한다. 

하지만, 그 만큼 필요 메모리가 있어야한다.

<br>

객체가 대량 생상되는 것을 막아야한다. 10년치 데이터를 한번에 가져온다든지 10만 개의 객체를 만들면 하나 객체가 0.5kb 일때, 전체 메모리 4.9gb를 사용하게 된다.

파일을 불러올 때에도, 한번에 모든 파일을 불러오지않고 적절하게 나눠서 File.read() 하도록 하자. 메모리가 터질 수 있다.


<br><br>

### 응답 데이터 압축

전송 시간은 2가지 요소에 영향을 받는다.

- 네트워크 속도
- 전송 데이터 크기 → 압축 방법 사용 (gzip)

<br>

응답을 압축할 때 고려사항

- html, css, json 과 같은 텍스트 형식은 압축률이 높아 효과적이다. 반면 jpeg나 이미지나 zip 파일처럼 이미 압축한 데이터를 다시 압축해도 효과가 없다.
- 웹 서버에서 압축을 적용했더라도 방화벽이 이를 해제해 응답할 수 있다.


<br><br>

### 정적 자원과 브라우저 캐시

접속할 때마다 js를 매번 새로 받으면 화면 표시 속도도 느려진다.

서버가 전송하는 트래픽을 줄이면서 브라우저가 더 빠르게 화면을 보여줄 수 있는 방법은 클라이언트 캐시를 이용하는 것이다. 
<br>

```
Cache-Control: max-age=60
```

<br>
또한, 서비스가 성장하기 시작했다면, 오리진 서버의 트래픽 감소, 콘텐츠의 빠른 제공, 트래픽 절감이라는 장점을 가진 CDN을 이용해보자.


<br><br>

### 대기 처리

대표적인 예시 콘서트 예매 시스템 → 트래픽 급증

<br>
해결 방안 

- 서버 증설, DB 증설 → DB 증설은 줄이고 키우고가 쉽지 않음.
- 수용할 수 있는 처리만 받고 나머지는 대기 처리
    - 장점
    - 서버를 증설하지않고도 서비스를 안정적으로 제공가능
    - 사용자의 지속적인 새로고침으로 인한 트래픽 폭증도 방지할 수 있다. 새로고침을 할 경우 뒤로 밀리기 때문에 안함.
