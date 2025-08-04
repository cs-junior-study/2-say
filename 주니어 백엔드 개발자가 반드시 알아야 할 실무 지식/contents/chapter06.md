
# 6장 동시성, 데이터가 꼬이기 전에 잡아야 한다.

생성일: 2025년 7월 2일 오후 5:52

<br><br>

트래픽이 많지 않은 서비스라도 초당 100개 이상의 요청이 들어올 수 있다. 1개의 요청을 처리하는 데 0.1초가 걸린다고 가정하면, 100개의 요청을 1초 안에 처리하려면 0.1초마다 10개의 요청을 동시에 처리해야 한다.

<br>

서버가 동시에 여러 클라이언트의 요청을 처리하는 방식은 크게 다음 2가지가 있다.

- 클라이언트 요청마다 스레드를 할당해서 처리
- 비동기 IO(또는 논블로킹 IO)를 사용해서 처리


<br>


이런 상황에서 여러 스레드를 같은 자원에 경쟁 상태(race condition)를 주의해야한다.


<br><br>


### 간단하게 문제를 방지하는 자바 코드

<br>

```java
public class UserSessions {
    private Lock lock = new ReentrantLock();
    private Map<String, UserSession> sessions = new HashMap<>();
    
    public void addUserSession(UserSession session) {
	  lock.lock();
	  try {
		  sessions.put(session.getSessionId(), session);
	  } finally {
		  lock.unlock();
	  }
  }
  
  public UserSession getUserSession(String sessionId) {
	  lock.lock();
	  try {
		  return sessions.get(sessionId);
	  } finally {
		  lock.unlock();
	  }
  }
}
```

<br><br>


- 자바 21에는 ReentrantLock 기능으로 잠금 획득 대기 시간이 있다.
- 한 스레드만 접근 가능


<br><br>


### 세마포어

동시에 실행할 수 있는 스레드 수를 제한한다.

<br>

1. 세마포어에서 퍼밋 획득(허용 가능 숫자 1 감소) 
2. 코드 실행
3. 세마포어에 퍼밋 반환 (허용 가능 숫자 1 증가)

<br>

같은 데이터를 여러명에서 읽는 것은 문제가 되질 않는다. 

따라서, 읽기와 쓰기를 다르게 접근 권한을 주어 성능을 향상 시킬 수 있다.

<br><br>

```java
public class UserSessionsRW {
	private ReadWriteLock lock = new ReentrantReadWriteLock();
	private Lock writeLock = lock.writeLock();
	private Lock readLock = lock.readLock();
	private Map<String, UserSession> sessions = new HashMap<>();
	
	public void addUserSession(UserSession session) {
		writeLock.lock();
		try {
			sessions.put(session.getSessionId(), session);
		} finally {
			writeLock.unlock();
		}
	}
	
	public UserSession getUserSession(String sessionId) {
		readLock.lock();
		try {
			return sessions.get(sessionId);
		} finally {
			readLock.unlock();
		}
	}
}
```

<br><br>

- 그 외에도 원시적 타입을 사용할 수 있다 → CAS 연산으로 빠름
- 컬렉션도 있음 ex) ConcurrentHashMap

<br><br>


### DB와 동시성

- 비관적, 낙관적이 있다.


<br><br>

### 선점(비관적) 잠금

선점 잠금은 데이터에 먼저 접근한 트랜잭션이 잠금을 획득하는 방식이다.

이후 접근한 트랜잭션은 잠금을 얻을 때 까지 대기

<br><br>


### 비선점(낙관적) 잠금

비선적 잠금은 명시적으로 잠금을 사용하지 않느다. 대신 데이터를 조회한 시점의 값과 수정하려는 값이 같은지 비교하는 방식(버전)으로 동시성 문제를 해결한다.

만약, 버전이 바뀐 후라면 트랜잭션을 롤백한다. 이 경우에 잠금을 구하기 위한 대기 과정 없이 바로 빠르게 응답을 할 수 있다는 장점이 있다.

<br><br>

### 외부 연동과 잠금

외부 시스템에 경우에는 비선점을 사용하면, 이미 취소했는데, 데이터 변경을 실패해서 트랜잭션이 롤백하는 문제가 생길 수 있는데, 아웃박스 패턴을 적용하여 처리할 수도 있긴하다.

적절한 트랜잭션 신뢰성을 맞춰야 함.

<br>


- 그 외에 동시성 문제를 해결할 수 있는 TIP 증분 쿼리를 사용해서 하나의 쿼리로 연산을 진행하도록 하자.

<br><br>

```java
update SUBJECT set joinCount = joinCoun + 1 where id = ? //but 원자적 연산이 아닐 수 있음.
```

<br><br>

### 잠금 사용 시 주의 사항

- 잠금 해제하기 try - finally - resource
- 대기 시간 지정하기
- 교착 상태 (deadlock) 피하기 - 대기 시간 지정

<br><br>

### 단일 스레드로 처리하기

애초에 동시성 문제가 발생하지 않게 한 스레드로만 접근하도록 해보자

스레드 1, 2, 3 → 하나의 작업큐
