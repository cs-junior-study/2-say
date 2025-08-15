# 8장 실무에서 꼭 필요한 보안 지식

생성일: 2025년 8월 1일 오후 9:57

<br><br>

서버 개발에서 가장 기본적인 보안은 인증과 인가이다. 인증(Authentication)은 사용자가 누구인지 확인하는 과정이고, 인가(authorization)는 사용자에게 자원에 접근할 수 있는 권한을 부여하는 것이다.

서버는 인증에 성공하면 고유한 토큰을 응답에 전송한다.

토큰을 이용해서 사용자를 식별하려면 토큰과 사용자 간의 매핑 정보를 어딘가에 저장해야 한다. 

<br>

- 서버 별도 저장소
- 토큰 : 토큰 자체에 사용자 식별자 정보를 저장

<br><br>

### 별도 저장소에 토큰과 사용자 식별자 정보 저장하기

<br>

<img width="1767" height="738" alt="image" src="https://github.com/user-attachments/assets/cd50e85a-afad-4d62-a1ec-69108d5a3b45" />

<br><br>

외부 저장소에 보관되는 정보는 다음과 같다. 

- 토큰
- 사용자 식별자
- 생성 시간
- 최근 사용 시간
- 그 외 유효 시간, 클라이언트 버전 등 추가 데이터

<br>

토큰 정보를 아무리 많이 저장해도 340만 개의 토큰도 2.4G 정도이다. → 레디스 충분히 사용 가능

서버 Session에 저장하는 방법도 있지만, 고정 세션이 필요하다. 그 이유는 서버마다 다른 토큰 집합을 저장하고 있기 때문이다. 또한, **서버를 재시작하면 모두 사라진다.**

세션을 레디스나 DB에 저장시켜주는 스프링 세션 의존성도 있다.

<br><br>

### 토큰 자체에 사용자 식별자 정보 저장하기

<br>

JWT를 사용하면 된다.

- 별도 공간이 필요없음
- 서버 무 상태 유지 가능, 확장성
- 단점 : 서버와 클라이언트 주고받는 데이터 고정 증가

<br>

쿠키나, 헤더에 토큰을 전송할 수 있다. 

<br><br>

### 토큰 보안

<br>

토큰 탈취에 따른 보안 문제를 완화하는 방법은 토큰 유효 시간에 제한을 두는 것이다.

<br>

크게 두 가지 방법이 있는데 

- 토큰 생성 기준으로 제한 시간 두는 방법
- 마지막 접근 시간을 기준으로 토큰 유효 시간 정하는 방법

여기에 접속 IP 비교까지 하면 더욱 강한 보안을 만들 수 있다.

<br><br>

### 인가와 접근 제어 모델

<br>

관리자 페이지처럼, 사용자가 접근할 수 있는 기능(또는 자원)을 관리하기 위한 모델을 접근 제어 모델이라고 한다.

<br>

대표적인 접근 제어 모델로은 역할 기반 접근 제어(Role-Based-Access-Control, RBAC) 모델이 있다.

<br>

역할별 권한 부여 방식과 사용자별 권한 부여 방식은 각각 장단점이 있어서 함께 사용하는 경우 가 많다.

<br>

RBAC를 사용하면 역할 몇 개만 계정에 부여하면 된다. 하지만, 계정마다 섬세한 커스텀마이징은 어렵다.

속성 기반 접근 제어(Attribute-Based-Access Control, ABAC)모델도 있다. 

예를 들어 IP 주소에 따라 특정 기능의 접근을 허용하거나 제한할 수 있다. 하지만, 구현 복잡

<br><br>

### 데이터 암호화

- 양방향 암호화
- 단방향 암호화

<br><br>

### 단방향 암호화

<br>

해시함수를 이용해서 데이터를 해시 값으로 변환한다. 

- SHA-256, MD5, Bcrypt

<br>

단방향 암호화는 바이트 데이터를 기준으로 동작한다.

<br><br>

```java
byte[] origin = input.getBytes("UTF-8");
MessageDigest digest = MessageDigest.getInstance("SHA-256");
byte[] hash = digest.digest(origin); // byte 배열을 암호화
```

<br><br>

DB와 같은 저장소에 읽을 수 있는 형태로 저장하려면 바이트 배열을 문자열로 표현해야 한다.

이를 위해 Base64표기법을 사용한다. 

단방향 암호화는 원본 데이터로 복호화할 수 없기 때문에, 원본을 알지 못하면 알 방법이 없다.


<br><br>

### Salt로 보안 강화하기

같은 해시 알고리즘을 사용하면, 동일한 원본 데이터에 대해 항상 동일한 해시 값이 생성된다. 따라서, 솔트라는 임의의 값을 통해서 해시값을 다르게 생성한다.


<br><br>

### 양방향 암호화

암호화와 복호화가 모두 가능한 방식, 키(대칭키, 비대칭 키)를 사용 

- AES, RSA 알고리즘

<br>

대칭키는 암호화 복호화 동일한 키 사용, 따라서 하나 털리면 다 털림

비대칭 키는 다른 키 사용, (공개 키(암호화), 개인 키(복호화))

개인키로 암호화를 할 때에는 서명과 같은 이증 목적으로 사용된다. SSH 서버는 공개 키를 등록하고, 클라이언트는 서버에 접속할 때 개인 키를 이용해서 인증한다.

공개 키와 개인 키로 암호화/복호호하는 예제

<br>

```java
public static String encrypt(String plain, PublicKey publickey) {
	Cipher cipher = Cipher.getInstance("RSA");
	cipher.init(Cipher.ENCRYPT_MODE, publicKey);
	byte[] encryptedBytes = cipher.doFinal(plain.getBytes("UTF-8"));
	return Base64.getEncoder().encodeToString(encryptedBytes);
}

public static String decrypt(String encrypted, PrivateKey privateKey) {
	Cipher cipher = Cipher.getInstance("RSA");
	cipher.init(Cipher.DECRY_MODE, privateKey);
	byte[] decodedBytes = Base64.getDecoder().decode(encrypted);
	byte[] decryptedBytes = cipher.doFinal(decodedBytes);
	return new String(decryptedBytes, "UTF-8");
}
```

<br><br>

### HMAC을 이용한 데이터 검증

<br>

만약 클라이언트에서 요청을 보낼 때, 위변조되지 않았음을 확인해야한다. 이때 HMAC를 주로 사용한다.

메시지 발신자와 수신자는 둘 만 알고 있는 비밀 키(secret key)를 공유한다.

메시지 암호화(MAC)해서 서로 같은 키로 복호화해서 같으면 인증 완료.

<br><br>

## 방화벽으로 필요한 트래픽만 허용하기

방화벽은 네트워크 통신을 두 방향으로 제어한다.

- 인바운드 트래픽 : 외부에서 내부로 유입되는 것
- 아웃바운드 트래픽 : 내부에서 외부로 유출되는 것

<br>

바운드는 필수 트래픽만 허용하고 나머지는 모두 차단, (ex 443 허용)


<br><br>

### 감사 로그(audit log)남기기

다음은 대표적인 감사 로그 기록 대상

- 사용자의 로그인/로그아웃 내역
- 암호 초기화 등 설정 변경 내역
- 환자 기록을 조회한 의료진 정보
- 계약서의 수정 이력


<br><br>

### 데이터 노출 줄이기

서버에서 클라이언트로 전달하는 데이터는 보안을 철저히 하자.

혹은, 접근을 빈번하게 하거나, 짧은 시간 간격으로 조회하는 사용자가 있다면 접근 차단하도록 하는 것도 보안 방법이다.


<br><br>

### 비정상 접근 처리

- 평소와 다른 장소 로그인
- 평소와 다른 기기에서 로그인
- 로그인에 여러 차례 실패

위 처리를 막아야, 브루트 포스(brute force)공격을 막을 수 있다.

<br><br>

### 시큐어 코딩

- SQL 인젝션
- 입력 값 검증
- 개인 정보/민감 정보 암호화
- 에러 메시지에 시스템 정보 미노출
- 보안 통신
- CORS
- CSRF
