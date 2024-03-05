# [토비 스프링] 2장 테스트

## UserDaoTest

```java
public class UserDaoTest {
	public static void main(String[] args) throws SQLException {
		ApplicationContext context = new GenericXmlApplicationContext("applicationContext.xml") ；
		UserDao dao = context.getBean("userDao", UserDao.class)；

		User user = new User();
		user.setld("user");
		user.setName("백기선");
		user.setPassword("married")；

		dao.add(user)；

		System.out.println(user.getId() + " 등록 성공");
		
		User user2 = dao.get(user.getId());
		System.out .printin(user2.getName())；
		System.out.printin(user2.getPassword())；

		System.out.println(user2.getld() + ",조회 성공");
	}
}
```

- 자바에서 가장 손쉽게 실행 가능한 main() 메소드를 이용한다.
- 테스트할 대상인 UserDao의 오브젝트를 가져와 메소드를 호출한다.

## 웹을 통한 DAO 테스트 방법의 문제점

- DAO뿐만 아니라 서비스 클래스, 컨트롤러, JSP 뷰 등 모든 레이어의 기능을 다 만들고 나서야 테스트가 가능하다.

## 작은 단위의 테스트

- 테스트하고자 하는 대상이 명확하다면 그 대상에만 집중해서 테스트하는 것이 바람직하다.
    - 한꺼번에 너무 많은 것을 몰아서 테스트하면 테스트 수행 과정도 복잡해지고,오류가 발생했을 때 정확한 원인을 찾기가 힘들어지기 때문이다.

## 단위 테스트

- 하나의 관심에 집중해서 효율적으로 테스트할 만한 범위의 단위를 테스트하는 것이다.
- 단위를 넘어서는 다른 코드들은 신경 쓰지 않고, 참여하지도 않고 테스트가 동작할 수 있으면 좋다.

## 단위 테스트를 하는 이유

- 개발자가 설계하고 만든 코드가 **원래 의도한 대로 동작하는 지**를 개발자 **스스로 빨리 확인받기 위함**.

## 자동수행 테스트 코드

- UserDaoTest는 테스트할 데이터가 코드를 통해 제공되고, 테스트 작업 역시 코드를 통해 자동으로 실행한다.
- 테스트 자체가 사람의 수작업을 거치는 방법을 사용하기보다는 **자동으로 수행되도록** 코드로 만들어지는 것이 중요하다.
- 유연한 설계구조를 위해 **별도의 테스트용 클래스**를 만들어서 테스트 코드를 넣자!

## **UserDaoTest**의 문제점

**수동 확인 작업의 번거로움**

- 테스트 수행은 코드에 의해 자동으로 진행되긴 하지만 테스트의 결과를 확인하는 일은 사람의 책임이다.
- 그러므로 완전히 자동으로 테스트되는 방법이라고 말할 수가 없다.

**실행 작업의 번거로움**

- 테스트 코드가 많다면 ****전체 기능을 테스트해보기 위해 main() 메소드를 수백 번 실행하는 수
고가 필요하다.

## **UserDaoTest 개선**

### 테스트 검증의 자동화

첫 번째 문제점인 테스트 결과의 검증 부분을 코드로 만들어보자!

**수정 전 테스트 코드**

```java
System.out.printin(user2.getName())；
System.out.println(user2.getPassword())；
System.out.printin(user2.getld() + " 조회 성공");
```

**수정 후 테스트 코드**

```java
if (!user.getName().equals(user2.getName())) {
	System.out.printin("테스트 실패 (name)")；
}
else if (user.getPassword().equals(user2.getPassword())) {
	System.out.printin("테스트 실패 (password)")；
}
else {
	System.out.printin("조회 테스트 성공");
}
```

- 전달한 User 오브젝트와 get()을 통해 가져오는 User 오브젝트의 값을 비교해서 일치하는지 확인했다.
- 만약 다른 값이 있다면 그때는 테스트가 실패했다고 출력하고 테스트를 마치면 된다.

### 테스트의 효율적인 수행과 결과 관리

- 더 편리하게 테스트를 수행하고 편리하게 결과를 확인하려면 단순한 main() 메소드로는 한계가 있다.
- 일정한 패턴을 가진 테스트를 만들 수 있고, 많은 테스트를 간단히 실행시킬 수 있으며, 테스트 결과를 종합해서 볼 수 있고, 테스트가 실패한 곳을 빠르게 찾을 수 있는 기능을 갖춘 **테스트 지원 도구**가 존재한다.

### JUnit 테스트로 전환

- 지금까지 만들었던 main 메소드 테스트를 **JUnit**을 이용해 다시 작성해보자. JUnit은 **프레임워크**이다.
- 그러므로 클래스의 오브젝트를 생성하고 실행하는 일은 프레임워크에 의해 진행된다. main() 메소드도 만들 필요도 없고 오브젝트를 만들어서 실행을 시킬 필요도 없다.

### 테스트 메소드 전환

- 테스트가 main() 메소드로 만들어졌다는 건 제어권을 직접 갖는다는 의미이기에 가장 먼저 할 일은 main() 메소드에 있던 **테스트 코드를 일반 메소드로 옮기자!**
- 새로 만들 테스트 메소드는 JUnit 프레임워크가 요구하는 조건 두가지를 따라야 한다.
- 첫째는 메소드가 **public**으로 선언돼야 하는 것이고, 다른 하나는 메소드에 **@Test**라는 애노테이션을 붙여주는 것이다.

**JUnit 테스트 메소드로 전환**

```java
import org.junit.Test；
...

public class UserDaoTest {
	@Test
	public void addAndGet() throws SQLException { 
	
		ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml")；
		UserDao dao = context.getBean("userDao", UserDao.class)；
		...
	}
}
```

## 검증 코드 전환

테스트의 결과를 검증하는 if/else 문장을 JUnit이 제공하는 방법을 이용해 전환해보자.

```java
import static org.hamcrest.CoreMatchers.is；
import static org.junit.Assert.assertThat；

public class UserDaoTest {

	@Test
	public void addAndGet() throws SQLException {
		ApplicationContext context = new GenericXmlApplicationContext("applicationContext.xml")；
		UserDao dao = context.getBeanC'userDao", UserDao.class)；

		User user = new User()；
		user.setld("gyumee")；
		user.setName("박성철");
		user.setPassword("springnol");

		dao.add(user)；

		User user2 = dao.get(user.getld())；

		//if (!user2.getName().equals(user2.getName())) {...}
		assertThat(user2.getNameO, is(user.getName()))；

		//if (!user2.getPassword().equals(user2.getPassword())) {...}
		assertThat(user2.getPassword, is(user.getPassword()))；
	}
}
```

- JUnit은 예외가 발생하거나 assertThat()에서 실패하지 않고 테스트 메소드의 실행이 완료되면 테스트가 성공했다고 인식한다.

## JUnit 테스트 실행

JUnit 프레임워크를 이용해 앞에서 만든 테스트 메소드를 실행하도록 코드를 만들어보자!
스프링 컨테이너와 마찬가지로 JUnit 프레임워크도 자바 코드로 만들어진 프로그램이므로 한 번은 JUnit 프레임워크를 시작시켜 줘야한다.

**JUnit을 이용해 테스트를 실행해주는 main() 메소드**

```java
import org.junit.runner.JUnitCore；
...

public static void main(String[] args) {
	JUnitCore.main("springbook.user.dao.UserDaoTest")；
}
```

**검증 실패**

- JUnit은 assertThat()을 이용해 검증을 했을 때 기대한 결과가 아니면 이 **AssertionError**를 던진다.
- 따라서 assertThat()의 조건을 만족하지 못하면 테스트는 더 이상 진행되지 않고 JUnit은 테스트가 실패했음을 알게 된다.
- 또한 테스트 수행 중에 **일반 예외**가 발생한 경우에도 마찬가지로 테스트 수행은 중단되고 테스트는 실패한다.

## 테스트 결과의 일관성

테스트를 실행하면서 매번 UserDaoTest 테스트를 실행하기 전에 **DB의 USER 테이블 데이터를 모두 삭제**해줘야 한다. 깜빡잊고 그냥 테스트를 실행했다가는 이전 테스트를 실행했을 때 등록됐던 사용자 정보와 **기본키가 중복되어 add() 메소드 실행 중에 에러**가 발생할 것이다.

여기서 생각해볼 문제는 테스트가 **외부 상태**에 따라 성공하기도 하고 실패하기도 한다는 점이다. 반복적으로 테스트를 했을 때 테스트가 실패하기도 하고 성공하기도 한다면 이는 **좋은 테스트라고 할 수가 없다**. 코드에 변경사항이 없다면 **테스트는 항상 동일한 결과를 내야 한다.**

UserDaoTest의 문제는 이전 테스트 때문에 DB에 등록된 중복 데이터가 있을 수 있다는 점이다!
가장 좋은 해결책은 addAndGet() 테스트를 마치고 나면 테스트가 등록한 사용자 정보를 삭제해서, 테스트를 수행하기 이전 상태로 만들어주는 것이다.

### deleteAII()와 getCount() 추가

**deleteAll() - USER 테이블의 모든 레코드 삭제**

```java
public void deleteAll() throws SQLException {
	Connection c = dataSource.getConnection()；
	PreparedStatement ps = c.prepareStatement("delete from users")；

	ps.executellpdate()；

	ps.close()；
	c.close()；
}
```

**getCount() - USER 테이블의 레코드 개수를 반환**

```java
public int getCount() throws SQLException {
	Connection c = dataSource.getConnection()；
	PreparedStatement ps = c.prepareStatement("select count(*) from users")；
	ResultSet rs = ps.executeQuery()；
	rs.next()；
	int count = rs.getlnt(1)；

	rs.close()；
	ps.close()；
	c.close()；
	return count；
}
```

### deleteAII()과 getCount()의 테스트

- deleteAll()과 getCount() 메소드의 기능은 테스트를 하자면 USER 테이블에 수동으로 데이터를 넣고 deleteAll()을 실행한 뒤에 테이블에 남은 게 있는지 확인해야 한다.
- 사람이 테스트 과정에 참여해야 하니 자동화돼서 반복적으로 실행 가능한 테스트 방법은 아니다.
- 그래서 새로운 테스트를 만들기보다는 차라리 기존에 만든 **addAndGet() 테스트를 확장**하자.

**deleteAII()과 getCount()가 추가된 addAndGet() 테스트**

```java
@Test
public void addAndGet() throws SQLException {
	...
	dao.deleteAll()；
	assertThat(dao.getCount(), is(0))；

	User user = new User()；
	user.setld("gyumee") ；
	user. setName("박성철");
	user.setPassword("springnol") ；

	dao.add(user)；
	assertThat(dao.getCount(), is(1))；

	User user2 = dao.get(user.getId());

	assertThat(user2.getName(), is(user.getName()))；
	assertThat(user2.getPassword(), is(user.getPassword()))；
}
```

## 동일한 결과를 보장하는 테스트

- 테스트를 반복해서 여러 번 실행해도 계속 성공할 것이다.
- 이전에는 테스트를 하기 전에 매번 직접 DB에서 데이터를 삭제해야 했지만, 이제는 그런 번거로운 과정이 필요 없어졌다.
- 단위 테스트는 항상 일관성 있는 결과가 보장돼야 한다!

## addAndGet() 테스트 보완

id를 조건으로 해서 사용자를 검색하는 기능을 가진 get()이 파라미터로 주어진 id에 해당하는 사용자를 가져온 것인지, 그냥 아무거나 가져온 것인지 테스트에서 검증해보자!

**get() 테스트 기능을 보완한 addAndGet() 테스트**

```java
©Test
public void addAndGet() throws SQLException {
	...

	UserDao dao = context.getBean("userDao", UserDao.class)；
	User userl = new User("gyumee"z "박성철", "springnol")；
	User user2 = new User("leegw700", "이길원", "springno2")；
	// 중복되지 않은 값을 가진 두 개의 User 오브젝트를 준비해둔다. 	

	dao.deleteAll();
	assertThat(dao.getCount(), is(0));
	
	dao.add(userl);
	dao.add(user2);
	assertThat(dao.getCountO, is(2));

	User usergetl = dao.get(user1 .getld());
	// 첫 번째 User의 id로 get()을 실행하면 첫 번째 User의 값을 가진 오브젝트를 돌려주는지 확인한다. 
	assertThat(usergetl .getNameO, is(user1 .getName()));
	assertThat(usergetl .getPasswordO, is(user1 .getPassword()));

	User userget2 = dao.get(user2.getld());
	assertThat(userget2.getNameO, is(user2.getName()));
	assertThat(userget2.getPasswordO, is(user2.getPassword()));
	// 두 번째 User에 대해서도 같은 방법으로 검증한다. 
}
```

## get() 예외조건에 대한 테스트

get() 메소드에 전달된 id 값에 해당하는 사용자 정보가 없다면 null과 같은 특별한 값을 리턴하거나 id에 해당하는 정보를 찾을 수 없다고 예외를 던질 수 있다. 각기 장단점이 있는데 후자의 방법을 써보자!

**get() 메소드의 예외상황에 대한 테스트** 

```java
@Test(expected=EmptyResultDataAccessException.class)  // 테스트 중에 발생할 것으로 기대하는 예외 클래스를 지정해준다. 
public void getllserFailue() throws SQLException {
	ApplicationContext context = new GenericXmlApplicationContext("applicationContext.xml");

	UserDao dao = context.getBean("userDao", UserDao.class)；
	dao.deleteAll()；
	assertThat(dao.getCount(), is(0));

	dao.get("unknown_icr");  // 이 메소드 실행 중에 예외가 발생해야 한다. 예외가 발생하지 않으면 테스트가 실패한다.
}
```

- @Test에 **expected**를 추가해놓으면 보통의 테스트와는 반대로, 정상적으로 테스트 메소드를 마치면 테스트가 실패하고, expected에서 지정한 예외가 던져지면 테스트가 성공한다.
- 예외가 반드시 발생해야 하는 경우를 테스트하고 싶을 때 유용하게 쓸수 있다.

하지만 이 테스트는 실패한다. get()메소드에서 쿼리 결과의 첫 번째 로우를 가져오게 하는 rs.next()를 실행할 때 가져 올 로우가 없다는 SQLException이 발생하기 때문이다.

## 테스트를 성공시키기 위한 코드의 수정

이 테스트가 성공하도록 get() 메소드 코드를 수정해보자!

**데이터를 찾지 못하면 예외를 발생시키도록 수정한 get() 메소드**

```java
public User get(String id) throws SQLException 
	ResultSet rs = ps.executeQueryO；
	User user = null；
	
	if (rs.nextO) {
		user = new User()；
		user.setld(rs.getStringf'id"))；
		user.setName(rs.getString("name"))；
		user.setPassword(rs.getString("password"))；
	}
	rs.closeO；
	ps.closeO；
	
	c.closeO；
	
	if (user == null) throw new EmptyResultDataAccessException(1)；
	return user；
}
```

## 테스트가 이끄는 개발

get() 메소드의 예외 테스트를 만드는 작업의 순서를 잘 생각해보면 새로운 기능을 넣기 위해 UserDao 코드를
수정하고, 그런 다음 수정한 코드를 검증하기 위해 테스트를 만드는 순서로 진행한 것이 아닌 반대로 테스트를 먼저 만들어 테스트가 실패하는 것을 보고 나서 UserDao의코드에 손을 대기 시작했다. 

### 기능설계를 위한 테스트

가장 먼저 ‘존재하지 않는 id로 get() 메소드를 실행하면 특정한 예외가 던져져야 한다’는 식으로 만들어야 할 기능을 결정했다. 그러고나서 UserDao 코드를 수정하는 대신 getUserFailure() 테스트를 먼저 만들었다.

|  | 단계 | 내용 | 코드 |
| --- | --- | --- | --- |
| 조건 | 어떤 조건을 가지고 | 가져올 사용자 정보가 존재하지 않는 경우에 | dao.deleteAll(); assertThat(dao.getCount(), is(0)); |
| 행위 | 무엇을 할 때 | 존재하지 않는 id로 get()을 실행하면 | get(”unknown_id”); |
| 결과 | 어떤 결과가 나온다 | 특별한 예외가 던져진다. | @Test(expected=EmptyResultDataAccessException.class) |
- 테스트 코드는 마치 잘 작성된 하나의 **기능 정의서**처럼 볼 수 있다.
- 기능설계, 구현, 테스트라는 일반적인 개발 흐름의 기능설계에 해당하는 부분을 이 테스트 코드가 일부분 담당하고 있다고 볼 수도 있다.

### 테스트 주도 개발(TDD)

- 만들고자 하는 기능의 내용을 담고 있으면서 만들어진 코드를 검증도 해줄 수 있도록 테스트 코드를 먼저 만들고, 테스트를 성공하게 해주는 코드를 작성하는 방식의 개발 방법이다.
- TDD에서는 테스트 작성하고 이를 성공시키는 코드를 만드는 작업의 주기를 가능한 한 짧게 가져가도록 권장한다.

### 테스트코드 개선

지금까지 세 개의 테스트 메소드를 만들었다. 이쯤 해서 테스트 코드를 리팩토링해보자!

**UserDaoTest 코드에서 반복되는 부분**

```java
ApplicationContext context = new GenericXmlApplicationContext("applicationContext.xml")；
User dao = context.getBean("userDao", UserDao.class)；
```

**해결 방법**

- 중복된 코드는 별도의 메소드로 뽑아내는 것이 가장 손쉬운 방법이다. 그런데 이번에는 일반적으로 사용했던 메소드 추출 리팩토링 방법 말고 JUnit이 제공하는 기능을 활용해보자!

### @Before

- 테스트가 실행되기 전에 먼저 실행된다.
- 테스트 실행 전 필요한 로직을 선언하는 식으로 활용한다.

**중복 코드를 제거한 UserDaoTest**

```java
import org.junit.Before；
...
public class UserDaoTest {
	private UserDao dao； 
	
	//@Test 메소드가 실행되기 전에 먼저 실행돼야 하는 메소드를 정의한다.
	@Before
	public void setUp() {
		ApplicationContext context =
		new GenericXmlApplicationContext("applicationContext.xml")；
		this.dao = context.getBeanC'userDao", UserDao.class)；
	}
	
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

이를 이해하려면 **JUnit 프레임워크가 테스트 메소드를 실행하는 과정**을 알아야 한다. 

1. 테스트 클래스에서 **@Test**가 붙은 **public**이고 **void형**이며 **파라미터가 없는 테스트 메소드**를 모두 찾는다.
2. 테스트 클래스의 오브젝트를 하나 만든다.
3. **@Before**가 붙은 메소드가 있으면 실행한다.
4. **@Test**가 붙은 메소드를 하나 호줄하고 테스트 결과를 저장해둔다.
5. **@After**가 붙은 메소드가 있으면 실행한다.
6. 나머지 테스트 메소드에 대해 **2〜5번을 반복**한다.
7. 모든 테스트의 결과를 종합해서 돌려준다.

<aside>
💡 한 가지 꼭 기억해야 할 사항은 **각 테스트 메소드를 실행할 때마다 테스트 클래스의 오브젝트를 새로 만든다는 점이다.** 한번 만들어진 테스트 클래스의 오브젝트는 하나의 테스트 메소드를 사용하고 나면 버려진다.

</aside>

## 스프링 테스트 적용

**문제점**

- @Before을 선언한 메소드가 테스트 메소드 개수만큼 반복되기 때문에 애플리케이션 컨텍스트도 세 번 만들어진다.

테스트는 가능한 한 독립적으로 매번 새로운 오브젝트를 만들어서 사용하는 것이 원칙이다. 하지만 애플리케이션 컨텍스트처럼 생성에 많은 시간과 자원이 소모되는 경우에는 테스트 전체가 공유하는 오브젝트를 만들기도 한다. 

애플리케이션 컨텍스트는 무상태이기에 초기화되고 나면 내부의 상태가 바뀌는 일은 거의 없다. 따라서 애플리케이션 컨텍스트는 한 번만 만들고 여러 테스트가 공유해서 사용해도 된다.

### 스프링 테스트 컨텍스트 프레임워크 적용

먼저 @Before 메소드에서 애플리케이션 컨텍스트를 생성하는 다음 코드를 제거한다.

```java
ApplicationContext context = new GenericXmlApplicationContext("applicationContext.xml")；
```

**스프링 테스트 컨텍스트를 적용한 UserDaoTest**

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations="/applicationContext.xml")
public class UserDaoTest { 
	@Autowired
	private ApplicationContext context；
	...

	@Before
	public void setUp() {
		this.dao = this.context.getBean("userDao"z UserDao.class);
		...
	}

	...
}
```

- context 변수에 **@Autowired**를 선언하여 어플리케이션 컨텍스트를 주입받았다.
- **@RunWith**는 **JUnit 프레임워크의 테스트 실행 방법을 확장**할 때 사용하는 어노테이션이다.
    - SpringJUnit4ClassRunner라는 JUnit용 테스트 컨텍스트 프레임워크 확장 클래스를 지정하면 JUnit이 테스트를 진행하는 중에 테스트가 사용할 애플리케이션 컨텍스트를 만들고 관리하는 작업을 진행해준다.
- **@Contextconfiguration**은 자동으로 만들어줄 애플리케이션 컨텍스트의 **설정파일 위치를 지정**한 것이다.

### @Autowired

- @Autowired가 붙은 인스턴스 변수가 있으면. 테스트 컨텍스트 프레임워크는 **변수 타입과 일치하는 컨텍스트 내의 빈을 찾아 일치하는 빈이 있으면 인스턴스 변수에 주입**해준다

<aside>
💡 스프링 애플리케이션 컨텍스트는 초기화할 때 자기 자신도 빈으로 등록한다.

</aside>

@Autowired를 이용해 애플리케이션 컨텍스트가 갖고 있는 빈을 DI 받을 수 있다면 굳이 컨텍스트를 가져와 getBean()을 사용하는 것이 아니라, 아예 UserDao 빈을 직접 DI 받을 수 있다.

**UserDao를 직접 DI 받도록 만든 테스트**

```java
public class UserDaoTest {
	@Autowired
	UserDao dao；
	...
```

- 단, **@Autowired는 같은 타입의 빈이 두 개 이상 있는 경우**에는 타입만으로는 어떤 빈을 가져올지 결정할 수 없다.
    - 이때는 변수의 이름과 같은 이름의 빈이 있는지 확인하여 그 빈을 가져온다. 민약 변수 이름으로도 빈을 찾을 수 없는 경우에는 예외가 발생한다.

## 학습 테스트로 배우는 스프링

종종 자신이 만들지 않은 프레임워크나 다른 개발팀에서 만들어서 제공한 라이브러리 등에 대해서도 테스트를 작성해야 한다. 이런 테스트를 **학습 테스트**라고 한다.

학습 테스트의 목적은 **자신이 사용할 API나 프레임워크의 기능**을 테스트로 보면서 사용 방법을 익히려는 것이다.

따라서 테스트이지만 **검증이 목적이 아니다.**

### 학습 테스트의 장점

1. 다양한 조건에 따른 기능을 손쉽게 확인해볼 수 있다.
2. 학습 테스트 코드를 개발 중에 참고할 수 있다.
3. 프레임워크나 제품을 업그레이드할 때 호환성 검증을 도와준다.
4. 테스트 작성에 대한 좋은 훈련이 된다.
5. 새로운 기술을 공부하는 과정이 즐거워진다.

### 버그 테스트(Bug test)

버그 테스트란 코드에 **오류가 있을 때 그 오류를 가장 잘 드러내줄 수 있는 테스트**를 말한다.

버그 테스트는 일단 **실패하도록** 만들어야 한다. 버그가 원인이 되서 테스트가 실패하는 코드를 만드는 것이다. 그러고 나서 버그 테스트가 성공할 수 있도록 애플리케이션 코드를 수정한다. 테스트가 성공하면 버그는 해결된 것이다.

**버그 테스트의 장점**

- 테스트의 완성도를 높여준다
- 버그의 내용을 명확하게 분석하게 해준다
- 기술적인 문제를 해결하는 데 도움이 된다
