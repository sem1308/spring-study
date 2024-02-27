# 챕터 2 - 테스트

애플리케이션은 계속 변화함, 이에 따른 전략 2가지

1. 확장과 변경을 고려한 객체지향 설계
2. 내 코드에 대한 확신을 가지게 하고 변화에 유용하게 대처하는 테스트 기술

## 2.1 UserDaoTest 다시 보기

### 2.1.1 테스트의 유용성

코드를 리팩토링 했을 때 이게 처음과 동일한 기능을 수행함을 보장하는 방법은 테스트밖에 없음

> 💡 테스트란?<br>
코드가 예상하고 의도했던 대로 동작하는지 확인해서 코드를 확신할 수 있게 해주는 작업
> 

### 2.1.2 UserDaoTest의 특징

main() 메소드로 작성된 테스트

```jsx
public class UserDaoTest {
	public static void main(String[] args) throws SQLException { 
		ApplicationContext context = new GenericXmlApplicationContext( 
		"applicationContext.xml");
		UserDao dao = context.getBean("userDao", UserDao.class);

		User user = new User();
		user.setld("user");
		user.setName("백기선");
		user.setPassword("married");

		dao.add(user);

		System.out.println(user.getId() + " 등록 성공");

		User user2 = dao.get(user.getId());
		System.out.printin(user2.getName());
		System.out.printin(user2.getPassword());

		System.out.println(user2.getld() + "조회 성공");
	}
}
```

- 자바에서 가장 손쉽게 실행 가능한 main() 메소드를 이용한다.
- 테스트할 대상인 **UserDao의 오브젝트를 가져와** 메소드를 호출한다.
- 테스트에 사용할 입력 값(User 오브젝트)을 직접 코드에서 만들어 넣어준다.
- 테스트의 결과를 콘솔에 출력해준다.
- 각 단계의 작업이 에러 없이 끝나면 콘솔에 성공 메시지로 출력해준다.

<br>

**웹을 통한 DAO 테스트 방법의 문제점**

---

보통 웹 프로그램에서 사용하는 DAO 테스트하는 방법

- 테스트용 웹 애플리케이션을 서버에 배치한 뒤 간단한 입력을 통해
- User 오브젝트 만들고 UserDao 호출 기능 만들어져야함
- 그리고 생성된 데이터를 불러오는 기능도 만들어야함

단점

- DAO만 테스트하고 싶은데 다른 서비스, 컨트롤러, JSP 뷰 등 다른 기능들을 추가적으로 만들어야함
- 문제 발생 지점 찾기 어려움
- 테스트하고 싶은 DAO가 아니라 다른 곳에서 오류 발생 가능

<br>

**작은 단위의 테스트**

---

테스트하는 대상만 집중해서 테스트 하는 것이 바람직

> 💡 단위 테스트란?
>
>**작은 단위**의 코드에 대해 테스트를 수행하는 것
>
>**작은 단위?**<br>
>**하나의 관심**에 집중해서 효율적으로 테스트할 만한 범위의 단위
> 

때로는 각종 기능을 묶어서 테스트해야하는 상황 발생

- 각 기능은 잘 동작하나 실제로 연결했을 때 잘 동작해야 하기 때문

단위 테스트를 하는 이유

- 개발자가 설계하고 만든 **코드가 원래 의도한 대로 동작**하는지를 **개발자 스스로 빨리 확인**받기 위해
- 개발자 테스트, 프로그래머 테스트라고도 함

확인의 대상과 조건이 간단하고 명확하면 좋음
<br><br>

**자동수행 테스트 코드**

---

UserDaoTest 특징

- 테스트 **데이터**가 **코드를 통해** 제공
- 테스트 **작업** 역시 **코드를 통해** 자동으로 실행

**테스트는 자동으로 수행되도록 코드로 만들어지는 것이 중요**

자동 수행 테스트 장점

- 자주 반복할 수 있음
- 언제든지 코드를 수정하고 테스트 할 수 있음

<br>

**지속적인 개선과 점진적인 개발을 위한 테스트**

---

테스트 장점

- **새로운 기능**도 **기대한 대로 동작**하는지 확인
- **기존에 만들어뒀던기능들이** 새로운 기능을 추가하느라 수정한 코드에 영향을 받지 않고 **여전히 잘 동작하는지를 확인**

### 2.1.3 UserDaoTest 문제점

1. 수동 확인 작업의 번거로움
    1. add, get 정보 일치를 눈으로 확인해야함
2. 실행 작업의 번거로움

## 2.2 UserDaoTest 개선

### 2.2.1 테스트 검증의 자동화

첫 번째 문제점인 **테스트 결과의 검증 부분을 코드로** 만들어보자.

add한 정보, get한 정보가 정확히 일치하는지 확인

[수정 전]

```jsx
System.out.printin(user2.getName());
System.out.println(user2.getPassword());
System.out.printin(user2.getld() + " 조회 성공");
```

[수정 후]

```jsx
if (!user.getName().equals(user2.getName())) {
	System.out.printin("테스트 실패 (name)");
}
else if (!user.getPassword().equals(user2.getPassword())) {
	System.out.printin("테스트 실패 (password)");
}
else {
	System.out.printin("조회 테스트 성공");
}
```

### 2.2.2 테스트의 효율적인 수행과 결과 관리

테스트가 많아지면 이를 관리하고 시각화해줄 도구가 필요

**JUnit 테스트로 전환**

---

main 메소드를 JUnit으로 재작성

**테스트 메소드 전환**

---

```jsx
import org.junit.Test；
public class UserDaoTest {
	// JUnit에게 테스트용 메소드임을 알려준다.
	@Test // ------> JUnit 테스트 메소드는 반드시 public으로 선언돼야 한다.
	public void addAndGet() throws SQLException { 
		ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
		UserDao dao = context.getBean("userDao", UserDao.class);
	}
}
```

1. main() 대신 일반 메소드로 변경
2. public 권한 부여
3. @Test 애노테이션 추가

**검증 코드 전환**

---

```jsx
[기존]
if(!user.getName().equals(user2.getName())) { ... }

[JUnit]
assertThat(user2.getNameO() is(user.getName()));
// is는 매처의 일종으로 equals로 비교해주는 기능
```

[JUnit 적용 UserDaoTest]

```jsx
import static org.hamcrest.CoreMatchers.is；
import static org.junit.Assert.assertThat；

public class UserDaoTest {
	©Test
	public void addAndGet() throws SQLException {
		ApplicationContext context = new GenericXmlApplicationContext("applicationContext.xml");
		UserDao dao = context.getBean("userDao", UserDao.class);
		User user = new User();
		user.setld("gyumee");
		user.setName("박성 철");
		user.setPassword("springnol");

		dao.add(user);
		User user2 = dao.get(user.getld());
		assertThat(user2.getName(), is(user.getName()));
		assertThat(user2.getPassword(), is(user.getPassword()));
	}
}
```

**JUnit 테스트 실행**

---

- JUnitCore 클래스의 main 메소드 호출
    - 메소드 파라미터에는 @Test 테스트 메소드를 가진 클래스의 이름

```jsx
import org.junit.runner.JUnitCore;
public static void main(String[] args) {
	JUnitCore.main("springbook.user.dao.UserDaoTest");
}
```

## 2.3 개발자를 위한 테스팅 프레임워크 JUnit

### 2.3.1 JUnit 테스트 실행 방법

### 2.3.2 테스트 결과의 일관성

불편한점

- 테스트 실행 전 DB의 USER 테이블 데이터를 모두 삭제해줘야 함

코드에 변경사항이 없다면 테스트는 항상 동일한 결과를 내야 함

UserDaoTest에서 테스트 수행 후 DB 정보를 삭제하는 기능 추가

**deleteAll(), getCount() 추가**

---

**deleteAll()**

- USER 테이블 모든 레코드 삭제 기능

```jsx
public void deleteAll() throws SQLException {
	Connection c = dataSource.getConnection();
	PreparedStatement ps = c.prepareStatement("delete from users");
	ps.executellpdate();
	ps.close();
	c.close();
}
```

**getCount()**

- USER 테이블 레코드 개수 반환

```jsx
public int getCount() throws SQLException {
	Connection c = dataSource.getConnection();
	PreparedStatement ps = c.prepareStatement("select count(*) from users");
	ResultSet rs = ps.executeQuery();
	rs.next();
	int count = rs.getlnt(1);
	rs.close();
	ps.close();
	c.close();
	return count;
}
```

**deleteAll()과 getCount() 테스트**

---

독립적인 테스트가 어려움

기존의 addAndGet() 테스트 확장

deleteAll()을 한 후 getCount()를 했을 때 0이 나오는지 확인

- 하지만 getCount()가 신뢰 불가능
- add수행 후 getCount()를 하여 1이 나오는지 확인!

[deleteAll(), getCount() 추가된 addAndGet() 테스트]

```jsx
@Test
public void addAndGetO throws SQLException {
	dao.deleteAll();
	assertThat(dao.getCount(), is(0));

	User user = new User();
	user.setld("gyumee");
	user. setName("박성 철");
	user.setPassword("springnol");
	dao.add(user);
	assertThat(dao.getCount(), is(1));

	User user2 = dao.get(user.getId());

	assertThat(user2.getName(), is(user.getName()));
	assertThat(user2.getPassword(), is(user.getPassword()));
}
```

**동일한 결과를 보장하는 테스트**

---

단위 테스트는 항상 일관성 있는 결과가 보장돼야 함

테스트를 실행하기 전과 실행한 후 DB의 상태가 동일해야함

위와 같은 경우는 만약 기존에 DB에 데이터가 있으면 전부 삭제해버리는 참사가 일어날 수 있음

### 2.3.3 포괄적인 테스트

테스트를 안 만드는 건 위험

성의 없이테스트를 만드는 바람에 문제가 있는 코드인데도 테스트가 성공하게 만드는 건 더 위험

특히 한 가지 결과만 검증하고 마는 것은 상당히 위험

**getCount() 테스트**

---

여러 개의 User 등록 후 getCount() 결과 확인

getCount()를 위한 새로운 테스트 메소드 작성

시나리오

1. USER 테이블의 데이터를 모두 지우고 getCount() 결과가 0임 확인
2. 3개 사용자 정보 추가할 때 마다 getCount()결과가 하나씩 증가하는지 확인

[ 테스트 용이성을 위한 파라미터가 있는 생성자 추가 ]

```jsx
public User(String id. String name, String password) {
	this.id = id;
	this.name = name;
	this.password = password;
}

public User() {}
```

[ getCount() 테스트 ]

```jsx
@Test
public void count() throws SQLException {
	ApplicationContext context = new GenericXmlApplicationContext (
	"applicationContext.xml");
	
	UserDao dao = context.getBean("userDao", UserDao.class);
	User userl = new User("gyumee", "박성철", "springnol");
	User user2 = new User("leegw700", "이길원", "springno2");
	User user3 = new User("bumjin", "박범진", "springno3");
	
	dao.deleteAll();
	assertThat(dao.getCount(), is(0));
	
	dao.add(user1);
	assertThat(dao.getCount()z is(1));
	
	dao.add(user2);
	assertThat(dao.getCount(), is(2));
	
	dao.add(user3)；
	assertThat(dao.getCount(), is(3));
}
```

JUnit 실행시 테스트들이 어떤 순서로 실행되는지 알 수 없음

**테스트의 결과가 테스트 실행 순서에 영향을 받는다면 테스트를 잘못 만든 것**

**addAndGet() 테스트 보완**

---

add()의 기능은 검증되었지만 get()의 기능은 완전히 검증되지 않음

두 개의 User add 후 각 User id를 파라미터로 전달해서 동일성 확인

[ get() 테스트 기능을 보완한 addAndGet() 테스트 ]

```jsx
©Test
public void addAndGet() throws SQLException {
	UserDao dao = context.getBean("userDao", UserDao.class);

  // 중복되지 않는 값을 가진 두 개의 user 오브젝트를준비해둔다
	User user1 = new User("gyumee", "박성철", "springnol");
	User user2 = new User("leegw700", "이길원", "springnoZ");
	dao.deleteAll(); 
	assertThat(dao.getCount(), is(0));

	dao.add(user1);
	dao.add(user2);
	assertThat(dao.getCount(), is(2));

	User userget1 = dao.get(user1 .getld());
	assertThat(userget1.getName(), is(user1 .getName()));
	assertThat(userget1.getPassword(), is(user1 .getPassword()));

	User userget2 = dao.get(user2.getld());
	assertThat(userget2.getName(), is(user2.getName()));
	assertThat(userget2.getPassword(), is(user2.getPassword()));
}
```

**get() 예외조건에 대한 테스트**

---

id에 해당하는 사용자 정보가 없는 경우 처리

- id에 해당하는 정보를 찾을 수 없다는 **예외를 던지는 것으로 처리**

[ get() 메소드의 예외상황에 대한 테스트 ]

```jsx
@Test(expected=EmptyResultDataAccessException.class) // 발생 할 것으로 기대하는 예외 클래스를 지정해줌
public void getUserFailure() throws SQLException { 
	ApplicationContext context = new GenericXmlApplicationContext (
	"applicationContext.xml");

	UserDao dao = context.getBean("userDao", UserDao.class);
	dao.deleteAll();
	assertThat(dao.getCount(), is(0));

	dao.get("unknown_id");// 이 메소드 실행 중에 예외가 발생해야 한다. 
												// 예외가 발생하지 않으면 테스트가 실패한다.
}
```

- **위의 경우는 JUnit4까지는 되지만 JUnit5에서는 메소드를 통해 하는 것으로 기억!!**
- 아직 테스트를 실행하면 실패함
    - dao에서 rs.next()를 할 때 아무런 row도 없으므로 SQLException을 발생시키기 때문

**테스트를 성공시키기 위한 코드의 수정**

---

[ 데이터를 찾지 못하면 예외를 발생시키도록 수정한 get() 메소드 ]

```jsx
public User get(String id) throws SQLException {
	ResultSet rs = ps.executeQuery();
	User user = null; // —> User는 null 상태로 초기히해늏는다.
	if (rs.next()) {
		// id를 조건으로 한 쿼리의 결과가 있으면 User 오브젝트를만들고 값을 넣어준다.
		user = new User()；
		user.setld(rs.getString("id"))；
		user.setName(rs.getString("name"))；
		user.setPassword(rs.getString("password"))；
	}

	rs.close();
	ps.close();
	c.close(); 

	// 결과가 없으면 User는 null 상태 그대로일 것이다.이를 확인해서 예외를 던져준다.
	if (user == null) throw new EmptyResultDataAccessException(1)；
	return user；
}
```

**포괄적인 테스트**

---

테스트를 할 땐 성공하는 테스트만 골라서 만들지 말 것!

**예외적인 테스트를 항상 먼저 생각할 것!**

> 👨🏻 로드 존슨 (스프링 창시자)

“항상 네거티브 테스트를 먼저 만들라”
> 

### 2.3.4 테스트가 이끄는 개발

작업 순서

1. 테스트를 먼저 만들어 테스트가 실패하는것을 봄
2. UserDao 코드 수정

새로운 기능을 넣기 위해 UserDao 코드 수정하고 그 코드를 검증하기 위해 테스트를 만들지 않음

**기능설계를 위한 테스트**

---

“존재하지 않는 id로 get() 메소드를 실행하면 특정한 예외가 던져져야 한다”는 식으로 만들어야 할 기능을 결정

getUserFailure() 테스트를 먼저 만듦

**테스트할 코드도 없는데 테스트를 만들 수 있었던 방법**

- **추가하고 싶은 기능을 코드로 표현**하려고 했기 때문

getUserFailure()에는 조건, 행위, 결과에 대한 내용이 잘 표현

[ getUserFailure() 테스트 코드에 나타난 기능 ]

|  | 단계 | 내용 | 코드 |
| --- | --- | --- | --- |
| 조건 (given) | 어떤 조건을 가지고 | 가져올 사용자 정보자 존재하지 않는 경우에 | dao.deleteAII( )；
assertThat(dao.getCount(), is(0))； |
| 행위 (when) | 무엇을 할 때 | 존재하지 않는 id로 get()을 실행하면 | get(”unknown_id”); |
| 결과 (then) | 어떤 결과가 나온다 | 특별한 예외가 던져진다 | @Test(expected=
EmptyResultDataAccessException.class) |

테스트 코드가 마치 설계도로 보임

해당 설계도에 맞게 코드를 작성하지 않으면 그 코드가 제대로 된 코드가 아님을 알 수 있음

**테스트 주도 개발**

---

> ⚙️ **테스트 주도 개발 ( TDD - Test Driven Development )**
> 
> 
> 만들고자 하는 기능의 내용을 담고 있으면서 만들어진 코드를 검증도 해줄 수 있도록 **테스트 코드를 먼저 만들고**, 테스트를 성공하게 해주는 **코드를 작성하는 방식**의 개발 방법
> 
> **“실패한 테스트를 성공시키기 위한 목적이 아닌 코드는 만들지 않는다”**
> 
> **테스트 우선 개발** ( TFD - Test First Development ) 라고도 함
> 

TDD 장점

1. 테스트를 빼먹지 않고 꼼꼼하게 만들어낼 수 있다
2. 코드를 만들어 테스트를 실행하는 그 사이 간격이 매우 짧음
    1. 코드에 대한 피드백을 매우 빠르게 받을 수 있음

테스트 작성하고 이를 성공시키는 코드를 만드는 작업의 주기를 가능
한 한 짧게 가져가도록 권장

테스트는 **코드를 작성한 후에 가능한 빨리 실행**할 수 있어야 함

테스트없이 **한 번에 너무 많은 코드를 만드는 것은 좋지 않음**

### 2.3.5 테스트 코드 개선

**@Before**

---

매번 테스트 메소드를 실행하기 전에 먼저 실행시켜주는 기능

[ 중복 코드를 제거한 UserDaoTest ]

```java
import org.junit.Before;

public class UserDaoTest {
	private UserDao dao;
	// setUp() 메소드에서 만드는 오브젝트를 테스트 메소드에서
	// 사용할 수 있도록 인스턴스 변수로 선언한다.
	@Before —> // JUnit이 제공하는 애노테이션. @Test 메소드가 실행되기전에 먼저 실행돼야 하는 메소드를 정의한다.
	public void setUp() {
		ApplicationContext context =
		new GenericXmlApplicationContext("applicationContext.xml")；
		this.dao = context.getBean("userDao", UserDao.class)；
	} 
	// 각 테스트 메소드에 반복적으로 나타났던 코드를 제거하고 별도의 메소드로 옮긴다.

	@Test
	public void addAndGet() throws SQLException {
		...
	}

	@Test
	public void count() throws SQLException {
		...
	}

	@Test(expected=EmptyResultDataAccessException.class)
	public void getUserFailure() throws SQLException {
		...
	}
}
```

JUnit이 하나의 테스트 클래스를 가져와 테스트를 수행하는 방식

1. 테스트 클래스에서 @Test가 붙은 public이고 void형이며 파라미터가 없는 테스트 메소드를 모두 찾는다.
2. **테스트 클래스의 오브젝트를 하나 만든다.**
    1. 각 테스트가 서로 영향을 주지 않고 독립적으로 실행됨을
    확실히 보장해주기 위해
3. @Before가 붙은 메소드가 있으면 실행한다.
4. @Test가 붙은 메소드를 하나 호출하고 테스트 결과를 저장해둔다.
5. @After가 붙은 메소드가 있으면 실행한다.
6. 나머지 테스트 메소드에 대해 2~5번을 반복한다.
7. 모든 테스트의 결과를 종합해서 돌려준다.

**픽스처**

---

> 🔶 픽스처

테스트를 수행하는 데 필요한 정보나 오브젝트
> 

UserDaoTest에서라면 dao가 대표적인 픽스처

[ User 픽스처를 적용한 UserDaoTest ]

```java
public class UserDaoTest {
	private UserDao dao;
	private User user1;
	private User user2;
	private User user3;

	@Before
	public void setUp() {
		this.user1 = new User("gyumee", "박성철", "springnol");
		this.user2 = new User("leegw700", "이길원", "springnoZ");
		this.user3 = new User("bumjin", "박범진", "springno3");
	}
}
```

## 2.4 스프링 테스트 적용

@Before 메소드가 테스트 메소드 개수만큼 반복 되므로 **애플리케이션 컨텍스트** 도 그 개수만큼 생성됨

- 빈이 많아지고 복잡해지면 **컨텍스트 생성 비용이 클 것**

**애플리케이션 컨텍스트는 한 번만 만들고 여러 테스트가 공유해서 사용해도 됨**

@BeforeClass 스태틱 메소드

- 테스트 클래스 전체에 걸쳐 딱 한번만 실행

### 2.4.1 테스트를 위한 애플리케이션 컨텍스트 관리

**스프링 테스트 컨텍스트 프레임워크 적용**

---

[ 스프링 테스트 컨텍스트를 적용한 UserDaoTest ]

```jsx
@RunWith(SpringJUnit4ClassRunner.class)—> // 스프링의 테스트 컨텍스트 프레임워크의JUnit 확장기능 지정
@ContextConfiguration(locations="/applicationContext.xml") // 테스트 컨텍스트가 자동으로 만들어줄 애플리케이션 컨텍스트의 설정파일 위치를 지정
public class UserDaoTest {
  @Autowired
	private ApplicationContext context;
	// 테스트 오브젝트가 만들어지고 나면 스프링 테스트 컨텍스트에 의해 자동으로 값이 주입된다.

	©Before
	public void setUp() {
		this.dao = this.context.getBean("userDao",UserDao.class);
	}
}
```
<br>

**테스트 메소드의 컨텍스트 공유**

---

테스트 오브젝트가 여러개 생성되어도 **컨텍스트는 1개만 생성**
<br><br>

**테스트 클래스의 컨텍스트 공유**

---

모두 같은 설정파일을 가진 애플리케이션 컨텍스트를 사용한다면. 스프링은 테스트 클래스 사이에서도 애플리케이션 컨텍스트를 공유

**@Autowired**

---

스프링의 DI에 사용되는 특별한 애노테이션

방식

1. 테스트 컨텍스트 프레임워크는 **변수 타입과 일치**하는 컨텍스트 내의 빈 찾음
2. 타입이 일치하는 빈이 있으면 인스턴스 변수에 주입

**스프링 애플리케이션 컨텍스트는 초기화할 때 자기 자신도 빈으로 등록**

스프링 DI를 사용하여 dao를 바로 주입받을 수도 있음

```jsx
public class UserDaoTest {
	@Autowired
	UserDao dao;
	...
}
```

타입으로 가져올 빈 하나를 선택할 수 없는 경우에는 변수의 이름과 같은 이름의 빈이 있는지 확인

### 2.4.2 DI와 테스트

인터페이스를 두고 이를 적용해야 하는 이유

1. 소프트웨어 개발에서 절대로 바뀌지 않는 것은 없기 때문
2. 클래스의 구현 방식은 바뀌지 않는다 하더라도 다른 차원의 서비스 기능 도입 가능
3. 테스트

<br>

**테스트 코드에 의한 DI**

---

오브젝트를 테스트 코드에서 변경할 수 있음

- 테스트 코드에서 DI를 통해 의존 오브젝트 변경 가능

만약 테스트할때 기존 운용 서버의 DB를 사용한다면?

- deleteAll 때문에 정보 삭제 → 위험

그렇다고 설정파일에서 DB를 바꾸면?

- 다시 서버 실행할 때 DB 바꿔주는 번거로움 발생

⇒ DI로 의존 오브젝트 변경!!

[ 테스트를 위한 수동 이를 적용한 UserDaoTest ]

```jsx
@DirtiesContext // 테스트 메소드에서 애플리케이션 컨텍스트의 구성이나 상태를 변경한다는 것을 테스트 컨텍스트 프레임워크에 알려준다.
public class UserDaoTest {
	@Autowired
	UserDao dao;
	
	@Before
	public void setUp() {
		// 테스트에서 UserDao가 사용할 DataSource 오브젝트를 직접 생성한다
		DataSource dataSource = new SingleConnectionDataSource(
		"jdbc:mysql://localhost/testdb", "spring", "book", true);

		dao.setDataSource(dataSource); // 수동 DI
	}
}
```

- 설정파일 수정 없이 오브젝트 관계 재구성 가능

하지만 매우 주의해서 사용해야함

- 의존관계 강제 변경
- 모든 테스트에서 공유
    - 다른 테스트에서도 변경된 컨텍스트가 사용됨
- 애플리케이션 컨텍스트 구성, 상태를 테스트 내에서 변경하지 않는 것이 원칙

@DirtiesContext

- 변경한 컨텍스트가 뒤의 테스트에 영향을 주지 않음
- 매번 새로운 애플리케이션 컨텍스트를 만듦

<br>

**테스트를 위한 별도의 DI 설정**

---

테스트에서 사용될 DataSource 클래스가 빈으로 정의된 테스트 전용 설정파일을 따로 만들어두는 방법

즉, 운영용, 테스트용 설정파일 총 2개를 만드는 것
<br><br>

**컨테이너 없는 DI 테스트**

---

스프링 컨테이너를 사용하지 않고 테스트를 만드는 것

[ 애플리케이션 컨텍스트 없는 DI 테스트 ]

 

```java
public class UserDaoTest {
	UserDao dao; // @Autowired가 없다.
	@Before
	public void setUp() {
		dao = new UserDao();
		DataSource dataSource = new SingleConnectionDataSource(
		"jdbc：mysql：//localhost/testdb", "spring", "book", true);
		dao.setDataSource(dataSource); // 오브젝트 생성, 관계설정 등을 모두 직접 해준다.	
	}
}
```
<br>

**DI를 이용한 테스트 방법 선택**

---

1. 스프링 컨테이너 없이 테스트할 수 있는 방법 우선
2. 여러 오브젝트와 복잡한 의존관계 오브젝트 테스트 
    
    → 스프링 설정을 통한 DI
    
3. 예외적인 의존관계를 강제로 구성해서 테스트해야 할 경우
    
    → 수동 DI 해서 테스트
    

## 2.5 학습 테스트로 배우는 스프링

> 🔶 학습 테스트
> 
> 
> 자신이 만들지 않은 프레임워크나 다른개발팀에서 만들어서 제공한 라이브러리 등에 대해서도 테스트를 작성
> 

학습 테스트 목적

- 자신이 사용할 API나 프레임워크 사용 방법 익히기 위해

### 2.5.1 학습 테스트의 장점

1. 다양한 조건에 따른 기능을 손쉽게 확인해볼 수 있다
2. 학습 테스트 코드를 개발 중에 참고할 수 있다
3. 프레임워크나 제품을 업그레이드할 때 호환성 검증을 도와준다
4. 테스트 작성에 대한 좋은 훈련이 된다
5. 새로운 기술을 공부하는 과정이 즐거워진다

### 2.5.2 학습 테스트 예제

### 2.5.3 버그 테스트

필요성 및 장점

1. 테스트의 완성도를 높여준다
2. 버그의 내용을 명확하게 분석하게 해준다
3. 기술적인 문제를 해결하는 데 도움이 된다

> **테스트의 필요성과 작성 방법**
> 
> 
> • 테스트는 자동화돼야 하고. 빠르게 실행할 수 있어야 한다.
> • main() 테스트 대신 JUnit 프레임워크를 이용한 테스트 작성이 편리하다.
> • 테스트 결과는 일관성이 있어야 한다. 코드의 변경 없이 환경이나 테스트 실행 순서에 따라서 결과가 달라지면 안 된다.
> • 테스트는 포괄적으로 작성해야 한다. 충분한 검증을 하지 않는 테스트는 없는 것보다 나쁠수 있다.
> • 코드 작성과 테스트 수행의 간격이 짧을수록 효과적이다.
> • 테스트하기 쉬운 코드가 좋은 코드다.
> • 테스트를 먼저 만들고 테스트를 성공시키는 코드를 만들어가는 테스트 주도 개발 방법도 유용하다.
> • 테스트 코드도 애플리케이션 코드와 마찬가지로 적절한 리팩토링이 필요하다.
> • @Before, @After를 사용해서 테스트 메소드들의 공통 준비 작업과 정리 작업을 처리할 수있다.
> • 스프링 테스트 컨텍스트 프레임워크를 이용하면 테스트 성능을 향상시킬 수 있다.
> • 동일한 설정파일을 사용하는 테스트는 하나의 애플리케이션 컨텍스트를 공유한다.
> • @Autowired를 사용하면 컨텍스트의 빈을 테스트 오브젝트에 DI 할 수 있다.
> • 기술의 사용 방법을 익히고 이해를 돕기 위해 학습 테스트를 작성하자.
> • 오류가 발견될 경우 그에 대한 버그 테스트를 만들어두면 유용하다
>