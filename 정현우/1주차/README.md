# 오브젝트와 의존관계
## 초난감 DAO
DAO(Data Access Object): DB를 사용해 데이터를 조작하는 기능을 전담하도록 만든 오브젝트
### User
`id`, `name`, `password` 세 개의 프로퍼티를 가진 `User` 클래스
```java
/* 사용자 정보 저장용 자바빈 User 클래스 */
package springbook.user.domain;

public class User {
	String id;
    String name;
    String password;
    
    public String getId() {
    	return id;
    }
    public void setId(String id) {
    	this.id = id;
    }
    public String getName() {
    	return name;
    }
    public void setName(String name) {
    	this.name = name;
    }
    public String getPassword() {
    	return password;
    }
    public void setPassword(String password) {
    	this.password = password;
    }
}
```
### UserDao
사용자 정보를 DB에 넣고 관리할 수 있는 DAO 클래스, `UserDao`

`add()`: 사용자를 생성
`get()`: `id`로 사용자 정보를 읽음
```java
/* JDBC를 이용한 등록과 조회 기능이 있는 UserDao 클래스 */
package springbook.user.dao;
...
public class UserDao {
	public void add(User user) throws ClassNotFoundException, SQLException {
		Class.forName("com.mysql.jdbc.Driver");
		Connection c = DriverManager.getConnection(
				"jdbc:mysql://localhost/springbook", "spring", "book");

		PreparedStatement ps = c.prepareStatement(
				"insert into users(id, name, password) values(?,?,?)");
		ps.setString(1, user.getId());
		ps.setString(2, user.getName());
		ps.setString(3, user.getPassword());
        
		ps.executeUpdate();
		ps.close();
		c.close();
	}
    
    public User get(String id) throws ClassNotFoundException, SQLException {
    	Class.forName("com.mysql.jdbc.Driver");
        Connection c = DriverManager.getConnection(
        		"jdbc:mysql://localhost/springbook", "spring", "book");
        
        PreparedStatement ps = c.prepareStatement(
        		"select * from users where id = ?");
        ps.setString(1, id);
        
        ResultSet rs = ps.executeQuery();
        rs.next();
        User user = new User();
        user.setId(rs.getString("id"));
        user.setName(rs.getString("name"));
        user.setPassword(rs.getString("password"));
        
        rs.close();
        ps.close();
        c.close();
        
        return user;
    }
}
```
### main()을 이용한 DAO 테스트 코드
`main()`: `UserDao`의 오브젝트를 생성해서 `add()`와 `get()` 메소드 검증
```java
/* 테스트용 main() 메소드 */
public static void main(String[] args) throws ClassNotFoundException, SQLException {
	UserDao dao = new UserDao();
    
	User user = new User();
	user.setId("whiteship");
	user.setName("백기선");
	user.setPassword("married");
	dao.add(user);
    
	System.out.println(user.getId() + " 등록 성공");
    
	User user2 = dao.get(user.getId());
	System.out.println(user2.getName());
    System.out.println(user2.getPassword());
    
    System.out.println(user2.getId() + " 조회 성공");
}
```
## DAO의 분리
### 관심사의 분리
관심사의 분리(Separation of Concerns): 관심이 같은 것끼리는 하나의 객체, 또는 친한 객체로 모이게 하고, 관심이 다른 것은 가능한 한 따로 떨어져서 서로 영향을 주지 않도록 분리하는 것
### 커넥션 만들기의 추출
#### UserDao의 관심사항
1. DB와 연결을 위한 커넥션
2. 사용자 등록을 위해 DB에 보낼 SQL 문장을 담은 `Statement`를 만들고 실행
3. 작업이 끝나면 사용한 리소스인 `Statement`와 `Connection` 오브젝트를 닫음

현재 코드의 문제점:
- DB 커넥션을 가져오는 코드가 다른 관심사와 섞여 `add()` 메소드에 같이 담겨있다.
- 동일한 코드가 `get()` 메소드에도 중복되어 있다.
#### 중복 코드의 메소드 추출
중복된 DB 연결 코드를 `getConnection()`이라는 독립적인 메소드로 만든다.
```java
/* getConnection 메소드를 추출해서 중복을 제거한 UserDao */
public void add(User user) throws ClassNotFoundException, SQLException {
	Connection c = getConection();
    ...
}

public void get(String id) throws ClassNotFoundException, SQLException {
	// DB 연결 기능이 필요하면 getConnection() 메소드를 이용하게 한다.
	Connection c = getConection();
    ...
}

// 중복된 코드를 독립적인 메소드로 만들어서 중복을 제거했다.
private Connection getConnection() throws ClassNotFoundException, SQLException {
	Class.forName("com.mysql.jdbc.Driver");
    Connection c = DriverManager.getConnection(
    		"jdbc:mysql://localhost/springbook", "spring", "book");
	return c;
}
```
#### 변경사항에 대한 검증: 리팩토링과 테스트
- `UserDao`의 기능에는 아무런 변화를 주지 않으면서 코드의 구조만 변경했다.
- 코드 내부의 설계가 개선되어 코드를 이해하기 편해지고, 변화에 효율적으로 대응할 수 있다.

리팩토링(refactoring): 기존의 코드를 외부의 동작방식에는 변화 없이 내부 구조를 변경해서 재구성하는 작업

메소드 추출(extract method): 공통의 기능을 담당하는 메소드로 중복된 코드를 뽑아냄
### DB 커넥션 만들기의 도입
`UserDao` 소스코드를 제공해주지 않으면서 고객 스스로 원하는 DB 커넥션 생성 방식을 적용시키게 만드는 법
#### 상속을 통한 확장
- `getConnection()`을 추상 메소드로 만든다.
- 추상 클래스 `UserDao`를 상속받아 `NUserDao`, `DUserDao` 등의 서브클래스에서 `getConnection()` 메소드를 원하는 방식대로 구현한다.
```java
/* 상속을 통한 확장 방법이 제공되는 UserDao */
public abstract class UserDao {
	public void add(User user) throws ClassNotFoundException, SQLException {
    	Connection c = getConnection();
        ...
    }
    
    public User get(String id) throws ClassNotFoundException, SQLException {
    	Connection c = getConnection();
        ...
    }
    
    // 구현 코드는 제거되고 추상 메소드로 바뀌었다. 메소드 구현은 서브클래스가 담당한다.
    public abstract Connection getConnection() throws ClassNotFoundException, SQLException;
}

public class NUserDao extends UserDao {
	// 상속을 통해 확장된 getConnection() 메소드
	public Connection getConnection() throws ClassNotFoundException, SQLException {
    	// N 사 DB 생성코드
    }
}

public class DUserDao extends UserDao {
	public Connection getConnection() throws ClassNotFoundException, SQLException {
    	// D 사 DB 생성코드
    }
}
```
디자인 패턴: 소프트웨어 설계 시 특정 상황에서 자주 만나는 문제를 해결하기 위해 사용할 수 있는 재사용 가능한 솔루션

템플릿 메소드 패턴(template method pattern): 슈퍼클래스에 기본적인 로직의 흐름을 만들고, 그 기능의 일부를 추상 메소드나 오버라이딩이 가능한 `protected` 메소드 등으로 만든 뒤 서브클래스에서 필요에 맞게 구현해서 사용하도록 하는 방법

훅(hook) 메소드: 슈퍼클래스에서 디폴트 기능을 정의해두거나 비워뒀다가 서브클래스에서 선택적으로 오버라이드할 수 있도록 만든 메소드

팩토리 메소드 패턴(factory method pattern): 서브클래스에서 구체적인 오브젝트 생성 방법을 결정하게 하는 것

문제점:
- 다중상속이 허용되지 않기 때문에 다른 목적으로 상속을 적용하기 힘들다.
- 서브클래스는 슈퍼클래스의 기능을 직접 사용할 수 있다.
- 슈퍼클래스 내부의 변경이 있을 때 모든 서브클래스를 함께 수정해야 할 수 있다.
- DB 커넥션을 생성하는 코드를 다른 DAO 클래스에 적용할 수 없다.
## DAO의 확장
### 클래스의 분리
- DB 커넥션과 관련된 부분을 서브 클래스가 아닌 완전히 독립적인 클래스 `SimpleConnectionMaker`로 만든다.
- 만든 클래스를 `UserDao`가 이용하게 한다.
- 생성자에서 `SimpleConnectionMaker`의 오브젝트를 만들어 두고, `add`/`get` 메소드에서 이를 이용해 DB 커넥션을 가져온다.
```java
/* 독립된 SimpleConnectionMaker를 사용하게 만든 UserDao */
public class UserDao {
	private SimpleConnectionMaker simpleConnectionMaker;
    
    public UserDao() {
    	// 상태를 관리하는 것도 아니니 한 번만 만들어 인스턴스 변수에 저장해두고 메소드에서 사용하게 한다.
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
```
```java
/* 독립시킨 DB 연결 기능인 SimpleConnectionMaker */
package springbook.user.dao;
...
// 더 이상 상속을 이용한 확장 방식을 사용할 필요가 없으니 추상 클래스로 만들 필요가 없다.
public class SimpleConnectionMaker {
	public Connection makeNewConnection() throws ClassNotFoundException, SQLException {
    	Class.forName("com.mysql.jdbc.Driver");
        Connection c = DriverManager.getConnection(
        		"jdbc:mysql://localhost/springbook", "spring", "book");
        return c;
    }
}
```
문제점:
- 상속을 통해 DB 커넥션 기능을 확장해서 사용하게 했던 게 다시 불가능해졌다.
- 다른 방식으로 DB 커넥션을 제공하는 클래스를 사용하기 위해 `UserDao` 코드를 직접 수정해야 한다.
- DB 커넥션을 제공하는 클래스가 어떤 것인지를 `UserDao`가 구체적으로 알고 있어야 한다.
```java
simpleConnectionMaker = new SimpleConnectionMaker();
```
### 인터페이스의 도입
`ConnectionMaker` 인터페이스를 정의하고 `UserDao`가 인터페이스를 사용하게 한다.
```java
/* ConnectionMaker 인터페이스 */
package springbook.user.dao;
...
public interface ConnectionMaker {
	public Connection makeConection() throws ClassNotFoundException, SQLException;
}
```
```java
/* ConnectionMaker 구현 클래스 */
package springbook.user.dao;
...
public class DConnectionMaker implements ConnectionMaker {
	...
    public Connection makeConnection() throws ClassNotFoundException, SQLException {
    	// D 사의 독자적인 방법으로 Connection을 생성하는 코드
    }
}
```
```java
/* ConnectionMaker 인터페이스를 사용하도록 개선한 UserDao */
public class UserDao {
	// 인터페이스를 통해 오브젝트에 접근하므로 구체적인 클래스 정보를 알 필요가 없다.
	private ConnectionMaker connectionMaker;
    
    public UserDao() {
    	// 앗! 그런데 여기에는 클래스 이름이 나오네!!
    	connectionMaker = new DConnectionMaker();
    }
    
    public void add (User user) throws ClassNotFoundException, SQLException {
    	// 인터페이스에 정의된 메소드를 사용하므로 클래스가 바뀐다고 해도 메소드 이름이 변경될 걱정은 없다.
    	Connection c = connectionMaker.makeConnection();
        ...
    }
    
    public User get(String id) throws ClassNotFoundException, SQLException {
    	Connection c = connectionMaker.makeConnection();
        ...
    }
}
```
문제점:
어떤 클래스의 오브젝트를 사용할지를 결정하는 생성자의 코드가 제거되지 않고 남아있다.
```java
connectionMaker = new DConnectionMaker
```
### 관계설정 책임의 분리
- `new DConnectionMaker()`는 충분히 독립적인 관심사를 담고 있다.
- `UserDao`의 클라이언트에게 어떤 `ConnectionMaker`의 구현 클래스를 사용할지 결정하게 한다.
- 메소드 파라미터나 생성자 파라미터를 통해 오브젝트를 전달받는다.

클래스들을 이용해 런타임 오브젝트 관계를 갖는 구조로 만들어주는 것이 클라이언트의 책임이다.
현재는 `main()` 메소드가 `UserDao`의 클라이언트라고 볼 수 있다.
```java
/* 수정한 생성자 */
public UserDao(ConnectionMaker connectionMaker) {
	this.connectionMaker = connectionMaker;
}
```
```java
/* 관계설정 책임이 추가된 UserDao 클라이언트인 main() 메소드 */
public class UserDaoTest {
	public static void main(String[] args) throws ClassNotFoundException, SQLException {
    	// UserDao가 사용할 ConnectionMaker 구현 클래스를 결정하고 오브젝트를 만든다.
    	ConnectionMaker connectionMaker = new DConnectionMaker();
        
        /* 1. UserDao 생성
           2. 사용할 ConnectionMaker 타입의 오브젝트 제공.
              결국 두 오브젝트 사이의 의존관계 설정 효과 */
        UserDao = new UserDao(connectionMaker);
        ...
    }
}
```
### 원칙과 패턴
#### 개방 폐쇄 원칙
개방 폐쇄 원칙(OCP, Open-Closed Principle): 클래스나 모듈은 확장에는 열려 있어야 하고 변경에는 닫혀 있어야 한다.

객체지향 설계 원칙(SOLID)
- SRP(The Single Responsibility Principle): 단일 책임 원칙
- OCP(The Open Closed Principle): 개방 폐쇄 원칙
- LSP(The Liskov Subsitution Principle): 리스코프 치환 원칙
- ISP(The Interface Segregation Principle): 인터페이스 분리 원칙
- DIP(The Dependency Inversion Principle): 의존관계 역전 원칙
#### 높은 응집도와 낮은 결합도
높은 응집도와 낮은 결합도(high coherence and low coupling)

높은 응집도: 변화가 일어날 때 해당 모듈에서 변하는 부분이 크다.

낮은 결합도: 책임과 관심사가 다른 오브젝트 또는 모듈과는 낮은 결합도, 즉 느슨하게 연결된 형태를 유지하는 것이 바람직하다.

결합도: 하나의 오브젝트가 변경이 일어날 때에 관계를 맺고 있는 다른 오브젝트에게 변화를 요구하는 정도
#### 전략 패턴
전략 패턴(Strategy Pattern): 자신의 기능 맥락(context)에서, 필요에 따라 변경이 필요한 알고리즘 클래스를 필요에 따라 바꿔서 사용할 수 있게 하는 디자인 패턴.

`UserDao`가 전략 패턴의 컨텍스트에 해당한다.
## 제어의 역전(IoC)
### 오브젝트 팩토리
현재 문제점:
- 어떤 `ConnectionMaker` 구현 클래스를 사용할지 결정하는 기능을 `UserTest`가 떠맡았다.
- `UserTest`가 기능 테스트 이외에 다른 책임까지 떠맡고 있다.
#### 팩토리
객체의 생성 방법을 결정하고 만들어진 오브젝트를 돌려주는 팩토리(factory) 클래스 `DaoFactory`를 만든다.
```java
/* UserDao의 생성 책임을 맡은 팩토리 클래스 */
package springbook.user.dao;
...
public class DaoFactory {
	public UserDao userDao() {
    	// 팩토리의 메소드는 UserDao 타입의 오브젝트를  어떻게 만들고, 어떻게 준비시킬지를 결정한다.
    	ConnectionMaker connectionMaker = new DConnectionMaker();
        UserDao userDao = new UserDao(connectionMaker);
        return userDao;
    }
}
```
```java
/* 팩토리를 사용하도록 수정한 UserDaoTest */
public class UserDaoTest {
	public static void main(String[] args) throws ClassNotFoundException, SQLException {
    	UserDao dao = new DaoFactory().userDao();
        ...
    }
}
```
#### 설계도로서의 팩토리
`UserDao`와 `ConnectionMaker`는 실질적인 로직을 담당하는 컴포넌트이다.

`DaoFactory`는 애플리케이션을 구성하는 컴포넌트의 구조와 관계를 정의한 설계도 같은 역할을 한다.

`DaoFactory`를 분리한 것은 애플리케이션의 컴포넌트 역할을 하는 오브젝트와 애플리케이션의 구조를 결정하는 오브젝트를 분리했다는 데 가장 의미가 있다.
### 오브젝트 팩토리의 활용
`DaoFactory`에 `UserDao`가 아닌 다른 DAO의 생성 기능을 넣는다.
```java
/* 생성 메소드의 추가로 인해 발생하는 중복 */
public class DaoFactory {
	public UserDao userDao() {
    	return new UserDao(new DConnectionMaker());
    }
    
    public AccountDao accountDao() {
    	return new AccountDao(new DConnectionMaker());
    }
    
    public MessageDao messageDao() {
    	// ConnectionMaker 구현 클래스를 선정하고 생성하는 코드의 중복
    	return new MessageDao(new DConnectionMaker());
    }
}
```
문제점:
`ConnectionMaker` 구현 클래스의 오브젝트를 생성하는 코드가 메소드마다 반복된다.

오브젝트를 만드는 코드를 별도의 메소드로 뽑아낸다.
```java
/* 생성 오브젝트 코드 수정 */
public class DaoFactory {
	public UserDao userDao() {
    	return new UserDao(connectionMaker());
    }
    
    public AccountDao accountDao() {
    	return new AccountDao(connectionMaker());
    }
    
    public MessageDao messageDao() {
    	return new MessageDao(connectionMaker());
    }
    
    public ConnectionMaker connectionMaker() {
    	// 분리해서 중복을 제거한 ConnectionMaker 타입 오브젝트 생성 코드
    	return new DConnectionMaker();
    }
}
```
### 제어권의 이전을 통한 제어관계 역전
제어의 역전(Inversion of Control): 프로그램의 제어 흐름 구조가 뒤바뀌는 것

일반적인 프로그램의 흐름
- `main()` 메소드와 같이 프로그램이 시작되는 지점에서 다음에 사용할 오브젝트를 결정
- 결정한 오브젝트를 생성
- 만들어진 오브젝트에 있는 메소드를 호출
- 그 오브젝트 메소드 안에서 다음에 사용할 것을 결정하고 호출

IoC
- 오브젝트가 자신이 사용할 오브젝트를 스스로 선택하거나 생성하지 않음
- 자기 자신이 어떻게 만들어지고 어디서 사용되는지 알 수 없음
- 모든 오브젝트는 위임받은 제어 권한을 갖는 특별한 오브젝트에 의해 결정되고 만들어진다.

라이브러리: 라이브러리를 사용하는 애플리케이션 코드가 흐름을 직접 제어한다.

프레임워크: 애플리케이션 코드가 프레임워크에 의해 사용된다.

IoC를 애플리케이션 전반에 걸쳐 본격적으로 적용하려면 스프링과 같은 IoC 프레임워크의 도움을 받는 것이 유리하다.
