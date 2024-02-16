# 챕터 1 - 오브젝트와 의존 관계 (1)

**스프링**

- 자바를 중요하게 여기는 이유?
    - 객체지향 언어
    - 스프링의 철학
- 객체지향 프로그래밍이 제공하는 폭넓은 혜택을 누릴 수 있도록 **기본으로 돌아가자**
- 스프링 이해를 위해서는 오브젝트에 관심을 가져야함
- 특히 **오브젝트 설계 - 객체지향 설계**
- 스프링은 설계, 구현 기법을 강요하지 않음
- 대신 어떻게 설계하고 구현하고 사용하고 개선해야 하는지 기준을 마련해 줌

## **1.1 초난감 DAO**

**초기 DAO**

```jsx
public class UserDao{

	public void add(User user) throws ClassNotFoundException, SQLException {
		Class.forName("com.mysql.jdbc.Driver");
		Connection c = DriverManager.getConnection(...);

		PreparedStatement ps = c.prepareStatement(
			"insert into users(id, name, password) values(?,?,?)");

		// 파라미터 등록
		ps.setString(1, user.getId());
		...

		ps.executeUpdate();

		// 공유 리소스 반환
		ps.close();
		c.close();
	}

	public User get(String id) throws ClassNotFoundException, SQLException {
		Class.forName("com.mysql.jdbc.Driver");
		Connection c = DriverManager.getConnection(...);

		PreparedStatement ps = c.prepareStatement(
			"select * from users where id = ?");

		// 파라미터 등록
		ps.setString(1, id);

		// 쿼리 실행 후 유저에 저장

		// 공유 리소스 반환
	}
}
```

**문제점**

- **관심사 분리**가 되어있지 않음

변화는 보통 한 가지 관심에 대해 일어남

대신 그 변화를 적용하려다 다른 것들을 수정하게 되는 상황이 발생

**같은 관심사를 묶어서 관리하는 것이 필요**

**UserDao 관심사항**

>1. DB 연결<br>2. SQL문으로 Statement만들고 실행<br>3. 공유 리소스 반환

**DB 연결 분리한 UserDao**

```jsx
Class.forName("com.mysql.jdbc.Driver");
		Connection c = DriverManager.getConnection(...);
```

- 아래 부분이 공통으로 들어가니 따로 메소드로 추출
    - **메소드 추출 기법**

```jsx
public class UserDao{

	public void add(User user) throws ClassNotFoundException, SQLException {
		Connection c = getConnection();

		PreparedStatement ps = c.prepareStatement(
			"insert into users(id, name, password) values(?,?,?)");

		// 파라미터 등록
		ps.setString(1, user.getId());
		...

		ps.executeUpdate();

		// 공유 리소스 반환
		ps.close();
		c.close();
	}

	public User get(String id) throws ClassNotFoundException, SQLException {
		Connection c = getConnection();

		PreparedStatement ps = c.prepareStatement(
			"select * from users where id = ?");

		// 파라미터 등록
		ps.setString(1, id);

		// 쿼리 실행 후 유저에 저장

		// 공유 리소스 반환
	}

	private Connection getConnection() throws ClassNotFoundException, SQLException{
		Class.forName("com.mysql.jdbc.Driver");
		Connection c = DriverManager.getConnection(...);
		return c;
	}
}
```

- UserDao를 사용하는 기업이 여러개인 경우, 각 기업마다 사용하는 DB가 달라 DB connect하는 방법이 다르면?
    - **상속의 개념 추가**

**상속의 개념 추가 추가 후 UserDao**

```jsx
public abstract class UserDao{

	public void add(User user) throws ClassNotFoundException, SQLException {
		Connection c = getConnection();
		...
	}

	public User get(String id) throws ClassNotFoundException, SQLException {
		Connection c = getConnection();
		...
	}

	private abstract Connection getConnection() throws ClassNotFoundException, SQLException;
}

class NUserDao extends UserDao{
	private Connection getConnection() throws ClassNotFoundException, SQLException{
		// N사 DB connection 생성코드
	}
}

class DUserDao extends UserDao{
	private Connection getConnection() throws ClassNotFoundException, SQLException{
		// D사 DB connection 생성코드
	}
}
```

UserDao를 변경할 필요 없이 확장 가능

> 💡 템플릿 메소드 패턴<br>
기본 로직의 흐름은 구현하고 기능의 일부를 추상 메소드나 protected 메소드 등으로 만든 뒤 서브클래스에서 필요에 맞게 구현하는 방법

> 💡 팩토리 메소드 패턴<br>
서브클래스에서 상위클래스가 사용할 구체적인 오브젝트 생성 방법을 결정하게 하는 것

UserDao가 관심사를 분리하고 확장까지 가능하게 했지만 ‘상속’이라는 개념을 사용했기에 아직 단점이 많음

아예 독립적인 class를 만들어 해결

```jsx
public class UserDao{
	private SimpleConnectionMaker simpleConnectionMaker;

	public UserDao(){
		simpleConnectionMaker = new SimpleConnectionMaker();
	}

	public void add(User user) throws ClassNotFoundException, SQLException {
		Connection c = simpleConnectionMaker.makeNewConnection();
		...
	}

	public User get(String id) throws ClassNotFoundException, SQLException {
		Connection c = simpleConnectionMaker.makeNewConnection();
		...
	}
}

public class SimpleConnectionMaker {
	public Connection makeNewConnection() throws ClassNotFoundException, SQLException {
		Class.forName("com.mysql.jdbc.Driver");
		Connection c = DriverManager.getConnection(...);
		return c;
	}
}
```

이제 자유로운 확장이 가능하게 해야함

**문제점**

- 각 기업마다 connection을 하는 함수의 시그니처가 다를 수 있음
- UserDao가 DB connection 제공 클래스를 구체적으로 알아야함

**해결**

- **인터페이스 적용**

인터페이스를 적용한 UserDao

```jsx
public class UserDao{
	private ConnectionMaker connectionMaker;

	public UserDao(){
//  connectionMaker = new NConnectionMaker();
		// 초기에 어떤 클래스의 오브젝트를 사용할 것인지 정해줘야함
		// 즉, 사용할 클래스가 바뀌면? UserDao코드도 수정해야함!
		connectionMaker = new DConnectionMaker();
	}

	public void add(User user) throws ClassNotFoundException, SQLException {
		Connection c = connectionMaker.makeNewConnection();
		...}

	public User get(String id) throws ClassNotFoundException, SQLException {
		Connection c = connectionMaker.makeNewConnection();
		...
	}
}

interface ConnectionMaker {
		Connection makeNewConnection() throws ClassNotFoundException, SQLException;
}

public class NConnectionMaker implements ConnectionMaker  {
	public Connection makeNewConnection() throws ClassNotFoundException, SQLException {
		// N사 DB 연결
	}
}

public class DConnectionMaker implements ConnectionMaker  {
	public Connection makeNewConnection() throws ClassNotFoundException, SQLException {
		// D사 DB 연결
	}
}
```

이처럼 DB 커넥션 제공 클래스에 대한 구체적 정보는 제거

하지만 초기에 어떤 클래스의 오브젝트를 사용할 것인지 결정하는 생성자의 코드는 제거되지 않고 남음

```jsx
public UserDao(){
	// 초기에 어떤 클래스의 오브젝트를 사용할 것인지 정해줘야함
	// 즉, 사용할 클래스가 바뀌면? UserDao코드도 수정해야함!
	connectionMaker = new DConnectionMaker();
}
```

관계설정 책임의 분리 (의존성 주입?)

```java
new DConnectionMaker()
```

UserDao가 어떤 ConnectionMaker 구현 클래스의 오브젝트를 사용하게 할지에 대한 관심이 존재

이를 분리해야 UserDao가 독립적이 됨

**사용되는 오브젝트**

- 서비스

**사용하는 오브젝트**

- 클라이언트

클라이언트에서 **서비스에서 사용될 오브젝트를 주입**해줌

UserDao를 사용하는 클라이언트에서 관계 설정을 해주면 됨

```jsx
public class UserDao{
	private ConnectionMaker connectionMaker;

	public UserDao(ConnectionMaker connectionMaker){
		// connectionMaker를 구현한 오브젝트가 달라져도
		// 코드가 바뀌지 않음
		this.connectionMaker = connectionMaker;
	}

	public void add(User user) throws ClassNotFoundException, SQLException {
		Connection c = connectionMaker.makeNewConnection();
		...}

	public User get(String id) throws ClassNotFoundException, SQLException {
		Connection c = connectionMaker.makeNewConnection();
		...
	}
}

interface ConnectionMaker {
		Connection makeNewConnection() throws ClassNotFoundException, SQLException;
}

public class NConnectionMaker implements ConnectionMaker  {
	public Connection makeNewConnection() throws ClassNotFoundException, SQLException {
		// N사 DB 연결
	}
}

public class DConnectionMaker implements ConnectionMaker  {
	public Connection makeNewConnection() throws ClassNotFoundException, SQLException {
		// D사 DB 연결
	}
}

// User Client Code
public class UserDaoTest{
	public static void main(String[] args) throws ClassNotFoundException, SQLException{
		ConnectionMaker connectionMaker = new DConnectionMaker();

		// 두 오브젝트 사이의 의존관계 설정 효과
		UserDao userDao = new UserDao(connectionMaker);	

	}
}
```

### **원칙과 패턴**

- 개방 폐쇄 원칙 (OCP - Open-Closed Principle)
    - 확장에는 열려있고 변경에는 닫혀있다
    - DB 연결 방법 기능 확장에는 열려있고
    - Dao의 변경에는 닫혀있음
    - 기능을 확장하더라도 Dao 코드에 변함이 없음
- 단일 책임 원칙 (SRP - Single Responsibility Principle)
    - 하나의 모듈 및 클래스는 하나의 책임,관심사만 가지고 있어야 함
- 리스코프 치환 원칙
- 인터페이스 분리 원칙
- 의존관계 역전 원칙

### **높은 응집도와 낮은 결합도**

**높은 응집도**

- **하나의 모듈, 클래스가 하나의 책임 또는 관심사에만 집중되어 있다는 뜻** (SRP와 관련 있는듯?)
- 변경이 일어날 때 모듈의 많은 부분이 함께 바뀜
- 만약 모듈의 일부분만 바뀌면 어떤 부분이 바뀌어야 하는지 찾는 번거로움이 생김

**낮은 결합도**

- 책임과 관심사가 다른 모듈과 느슨하게 연결된 형태 유지
- **느슨한 연결**
    - 관계 유지를 위한 최소한의 방법만 간접적인 형태로 제공
- **의존성 주입**

> **결합도** <br>
하나의 오브젝트가 변경이 일어날 때 관계를 맺고 있는 다른 오브젝트에게 변화를 요구하는 정도, 연쇄 변경의 정도 

### **전략 패턴**

변경이 필요한 알고리즘을 인터페이스를 통해 분리

이를 구현한 구체적인 클래스를 필요에 따라 바꿔 사용할 수 있게 하는 디자인 패턴

### **제어의 역전**

IoC (Inversion of Control)

### **오브젝트 팩토리**

위 상황에서는 test코드에 UserDao가 사용할 오브젝트를 생성하는 관심사가 묶여있음

- 이를 분리

분리될 기능

- UserDao, ConnectionMaker 구현 클래스 오브젝트 생성
- 이 오브젝트 연결

흔히 **팩토리**라고 부름

- 객체 생성 결정하는 클래스

```jsx
public class DaoFactory{
	public UserDao userDao(){
		UserDao userDao = new UserDao(new DConnectionMaker());
		return userDao;		
	}
}
```

만약 DAO 생성 메소드가 추가되면 ConnectionMaker 구현 클래스 인스턴스 생성 기능이 반복해서 나타남

만약 ConnectionMaker가 변경되면 3개의 코드를 변경해야함

```jsx
public class DaoFactory{
	public UserDao userDao(){
		// new DConnectionMaker() 반복
		UserDao userDao = new UserDao(new DConnectionMaker());
		return userDao;		
	}

	public MessageDao messageDao(){
		// new DConnectionMaker() 반복
		MessageDao messageDao = new MessageDao(new DConnectionMaker());
		return messageDao ;		
	}

	public AccountDao accountDao(){
		// new DConnectionMaker() 반복
		AccountDao accountDao = new AccountDao(new DConnectionMaker());
		return accountDao;		
	}
}
```

ConnectionMaker 생성 기능 분리

```jsx
public class DaoFactory{
	public UserDao userDao(){
		// new DConnectionMaker() 반복
		UserDao userDao = new UserDao(connectionMaker());
		return userDao;		
	}

	public MessageDao messageDao(){
		// new DConnectionMaker() 반복
		MessageDao messageDao = new MessageDao(connectionMaker());
		return messageDao ;		
	}

	public AccountDao accountDao(){
		// new DConnectionMaker() 반복
		AccountDao accountDao = new AccountDao(connectionMaker());
		return accountDao;		
	}

	public ConnectionMaker connectionMaker(){
		return new DConnectionMaker();
	}
}
```

### **제어의 역전**

프로그램 제어 흐름 구조 뒤바뀜

**일반적인 프로그램 흐름**

---

- 프로그램 시작 지점에서 사용 오브젝트 결정, 생성, 그 오브젝트의 메소드 호출, 그 메소드 안에서 다음에 사용할 것을 결정, 호출
- 즉, **각 오브젝트가 사용할 오브젝트를 결정하고 생성**
- 모든 종류의 작업을 **사용하는 쪽에서 제어**하는 구조

**제어의 역전**

---

- **오브젝트가 자신이 사용할 오브젝트를 스스로 선택하지 않음**
- 자신도 어떻게 만들어지고 어디서 사용되는지 알 수 없음
- 제어를 위임
- 서블릿 개발 → 배포 가능 but 실행을 개발자가 직접 제어 불가

**템플릿 메소드**

---

- 제어의 역전 개념 활용
- 서브 클래스에서는 슈퍼 클래스에서 정의한 추상 기능을 구현만 할 뿐 언제, 어떻게 사용될지 모름
- 슈퍼 클래스의 메소드를 사용할 때 호출되서 사용

**프레임워크 VS 라이브러리**

---

**라이브러리**

- 라이브러리 사용 **애플리케이션 코드가** **애플리케이션 흐름을 직접 제어**
- 동작 중 능동적으로 라이브러리 사용

**프레임워크**

- 애플리케이션 코드가 프레인워크에 의해 사용됨
- 개발자는 프레임 워크 위에 클래스를 만들고
- **프레임워크가 애플리케이션 실행** 중에 그 클래스, 애플리케이션 코드를 사용
- **제어의 역전 개념 적용**
- 위 DaoFactory를 IoC 컨테이너, IoC 프레임워크라 할 수 있음