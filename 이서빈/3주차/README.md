# Vol. 2 2장 테스트

스프링의 핵심인 IoC와 DI는 오브젝트의 설계와 생성, 관계, 사용에 관한 기술

애플리케이션은 계속 변하고 복잡해지는데 이 변화에 대응하는 방법

1. 확장과 변화를 고려한 `객체지향 설계` + 그것을 효율적으로 담아낼 수 있는 `IoC/DI` 같은 기술
2. 만들어진 코드를 확신할 수 있게 해주고 변화에 유연하게 대처할 수 있는 자신감을 주는 `테스트 기술`

⇒ 스프링으로 개발을 하면서 테스트를 만들지 않는다면 이는 스프링이 지닌 가치의 절반을 초기하는 것

# 2.1 UserDaoTest 다시보기

## 2.1.1 테스트의 유용성

> **📑 테스트란?**
> 
- 내가 **예상하고 의도했던 대로 코드가 정확히 동작하는지를 확인**해서, **만든 코드를 확신**할 수 있게 해주는 작업
- 테스트의 결과가 원하는 대로 나오지 않는 경우, 코드나 설계에 결함이 있음을 알 수 있음
- 이를 통해, 코드의 결함을 제거해가는 작업인 디버깅을 거치게 됨
- 최종적으로 테스트가 성공하면 모든 결함이 제거됐다는 확신을 얻을 수 있음

## 2.1.2. UserDaoTest의 특징

```java
public class UserDaoTest {
	public static void main(String[] args) throws SQLException {
		ApplicationContext context = new GenericXmlApplicationContext("applicationContext.xml");

		UserDao dao = context.getBean("userDao", UserDao.class);

		User user = new User();
		user.setId("user);
		user.setName("백기선");
		uset.setPassword("married");

		dao.add(user);

		System.out.println(user.getId() + "등록 성공");

		User user2 = dao.get(user.getId());
		System.out.println(user2.getName());
		System.out.println(user2.getPassword());

		System.out.println(user2.getId() + "조회 성공");
	}
}
```

- **main() 메소드 이용**
- **UserDao의 오브젝트를 가져와 메소드 호출**
- 테스트에 사용할 입력 값(User 오브젝트)을 직접 코드에서 만들어 넣어줌
- 테스트의 결과를 콘솔에 출력함
- 각 단계의 작업이 에러 없이 끝나면 콘솔에 성공 메시지로 출력함

### 웹을 통한 DAO 테스트 방법의 문제점

웹 화면을 통해 값을 입력 받고, 기능을 수행하고, 결과를 확인하는 방법은 가장 흔히 쓰이는 방법이지만

BUT DAO에 대한 테스트로서는 단점이 많다. 

- **모든 레이어의 기능을 다 만들고 나서야 테스트가 가능**하다는 점이 가장 큰 문제
- **하나의 테스트를 수행하는 데 참여하는 클래스와 코드가 너무 많이 때문**

⇒ 이런 방식으로 테스트하는 것은 번거롭고, 오류가 있을 때 빠르고 정확하게 대응하기가 힘들다

```java
if (!user.getName().equals(user2.getName())) {
	System.out.println("테스트 실패 (name)");
} else if (!user.getPassword().equals(user2.getPassword())) {
	System.out.println("테스트 실패 (name)");
} else {
	System.out.println("조회 테스트 성공");
}
```

### 작은 단위의 테스트

- **테스트는 가능하면 작은 단위로 쪼개서 집중해서 할 수 있어야 한다.**
- **관심사 분리** 원리 적용
- 테스트의 관심이 다르다면 테스트할 대상을 분리하고 집중해서 접근
- **작은 단위의 코드에 대해 테스트를 수행한 것을 단위 테스트 (Unit Test)**
- 단위는 충분히 하나의 관심에 집중해서 효율적으로 테스트할 만한 범위
- 단위 테스트 없이 바로 **긴 테스트를 하는 경우, 문제의 원인을 찾기가 매우 힘들다.**

**단위 테스트를 하는 이유 ⁉ **

- 개발자가 설계하고 만든 코드가 원래 의도한 대로 동작하는 지를 개발자 스스로 빨리 확인받기 위함
- == 개발자 테스트 == 프로그래머 테스트

### 자동수행 테스트 코드

- 테스트 자체가 사람의 수작업을 거치는 방법을 사용하기보다는
- 코드로 만들어져서 자동으로 수행될 수 있어야 한다는 건 매우 중요

**자동 수행 테스트의 장점**

- 자주 반복할 수 있다
- 테스트를 빠르게 실행할 수 있기 때문에 언제든 코드를 수정하고 나서 테스트 해볼 수 있음

**만들어둔 기능에 대한 테스트가 있다면**

⇒ 수정 후 빠르게 전체 테스트를 수행해서 

⇒ 수정 때문에 다른 기능에 문제가 발생하지는 않는지 재빨리 확인하고, 성공한다면

⇒ 마음에 확신을 얻을 수 있음

### 지속적인 개선과 점진적인 개발을 위한 테스트

- 새로운 기능도 기대한 대로 동작하는지 확인
- 기존에 만들어뒀던 기능들이 새로운 기능을 추가하느라 수정한 코드에 영향을 받지 않고 여전히 잘 동작하는지를 확인

## 2.1.3 UserDaoTest의 문제점 💥

**수동 확인 작업의 번거로움**

- **콘솔**에 나온 값을 보고 등록과 조회가 성공적으로 되고 있는 지를 **확인하는 것 사람의 책임**
- UserDaoTest는 간단하지만 검증해야 하는 필드 양이 많고 복잡해지면 불편함을 느낄 수 밖에 없음

**실행 작업의 번거로움**

- 만약 DAO가 수백 개가 되고 그에 대한 main() 메소드도 그만큼 만들어진다면
- 전체 기능을 테스트해보기 위해 main() 메소드를 수백 번 실행해야 함

# 2.2 UserDaoTest 개선

## 2.2.1 테스트 검증의 자동화

첫 번째 문제점인 테스트 결과의 검증 부분을 코드로 만들어보자

리스트 2-2 수정 전 테스트 코드

```java
System.out.printin(나ser2.getName())；
System.out.println(user2.getPasswordO)；
System.out.printin(user2.getld() + " 조회 성공");
```

리스트 2-3 수정 후 테스트 코드

```java
if (!user.getNameO.equals(user2.getNameO)) {
System.out.printin("테스트 실패 (name)")；
}
else if ('user.getPasswordO .equals(user2.getPasswordO)) {
System.out.printin("테스트 실패 (password)")；
}
else {
System.out.printin("조회 테스트 성공,");
```

**포괄적인 테스트comprehensive test**

- 만들어진 코드를 모두 점검
- 테스트를 통해 그 변경에 영향을 받는 부분이 정확하게 확인된다면 빠르게 조치할 수 있음

## 2.2.2 테스트의 효율적인 수행과 결과 관리 💕

- 일정한 패턴을 가진 테스트를 만들 수 있고,
- 많은 테스트를 간단히 실행시킬 수 있으며,
- 테스트 결과를 종합해서 볼 수 있고,
- 테스트가 실패한 곳을 빠르게 찾을 수 있는
- **기능을 갖춘 테스트 지원 도구와 그에 맞는 테스트 작성 방법이 필요**

**⇒ JUnit == 테스트 지원 도구**

### JUnit 테스트로 전환

- **프레임워크**는 개발자가 만든 클래스에  대한 제어 권한을 넘겨 받아서 **주도적으로 애플리케이션의 흐름을 제어함**
- 개발자가 만든 클래스의 오브젝트를 생성하고 실행하는 일은 프레임워크에서 진행됨

### 테스트 메소드 전환

main() 메소드에 있던 테스트 코드를 일반 메소드로 옮기는 것

- 메소드를 public으로 선언되어야 함
- @Test 어노테이션을 붙혀줘야 함
    
    ```java
    public class UserDaoTest {
    	@Test  // JUnit에게 테스트용 메소드임을 알려준다
    	public void addAndGet() throws SQLException {
    	// JUnit 테스트 메소드는 반드시 public으로 선언돼야 한다. 
    		ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
    		
    		UserDao dao = context.getBean("userDao", UserDao.class);
    	}
    }
    ```
    
### 검증 코드 전환

if 문장의 기능을 JUnit이 제공해주는 assertThat이라는 스태틱 메소드를 이용해 다음과 같이 변경할 수 있다. 

```java
assertThat(user2.getName(), is(user.getName()));
// user 오브젝트의 name 값과 get()에서 가져온 user2 오브젝트의 name 값이 같으면 다음으로 넘어가고, 아니면 테스트 실패
```

| assertThat() | 첫 번째 파라미터의 값을 뒤에 나오는 매처라고 불리는 조건으로 비교해서 일치하면 넘어가고, 아니면 테스트 실패 |
| --- | --- |
| is() | 매처의 일종, equals()로 비교해주는 기능 |

### JUnit 테스트 실행

- main 메소드에 Test 실행 클래스 정보를 넣어줌
    
    ```java
    import org.junit.runner.JUnitCore;
    ...
    public static void main(String[] args) {
    	JUnitCore.main("springbookuser.dao.UserDaoTest");
    }
    ```
    
# 2.3 개발자를 위한 테스팅 프레임워크 JUnit ‼

## 2.3.1 JUnit 테스트 실행 방법

자바 IDE에 내장된 JUnit 테스트 지원 도구를 시용

### IDE

**이클립스 테스트 코드 단축키**

| 단축키 | 역할 |
| --- | --- |
| ctrl + 1 | Quick fix |
| Ctrl + Shift + O | import 절에 없는 클래스를 추가하거나 정리 |
| Ctrl + Shirt + M | import 추가. 클래스 import를 static import로 전환할 수 있음. |
| Alt + Shift + X, T | JUnit으로 실행 |
| Ctrl + F11 | Run (JUnit 실행 가능) |
| Ctrl + J | 테스트 코드와 실제 코드 사이를 이동 (moreUnit이 설치되어 있을 때) |
| Ctrl + Q | 가장 마지막에 편집한 코드가 있는 곳으로 돌아가기 |
| Ctrl + T | 인터페이스에서 구현 클래스 찾을 때 |
| Ctrl + Shift + 위아래 방향키 | method 단위로 커서 이동 |
| Alt + Shift + R | 리팩토링 이름 바꾸기 |
| Alt + Shift + V | 리팩토링 – 이동 |
| Alt + Shift + M | 리팩토링 – 메소드 추출 |
| Alt + Shift + I | 리팩토링 - 메서드 인라이닝 (추출의 반대) |
| Alt + Shift+ L | 리팩토링 local 변수 추출 |

### 빌드 툴

프로젝트의 빌드를 위해 빌드 툴과 스크립트를 사용하고 있다면,

빌드 툴에서 제공하는 JUnit 플러그인이나 태스크를 이용해 JUnit 테스트를 실행할 수 있음

## 2.3.2 테스트 결과의 일관성 💞

- **반복적으로 테스트를 했을 때 테스트가 실패하기도 하고 성공하기도 한다면 이는 좋은 테스트라고 할 수 없음**
- 가장 좋은 해결책은 addAndGet() 테스트를 마치고 나면 테스트가 등록한 사용자 정보를 삭제해서, 테스트를 수행하기 이전 상태로 만들어주는 것

### deleteAII()의 getCount() 추가

**deleteAll()**

- USER 테이블의 모든 레코드를 삭제해주는 간단한 기능
    
    ```java
    public void deleteAHO throws SQLException {
    	Connection c = dataSource.getConnection()；
    	PreparedStatement ps = c.prepareStatement("delete from users")l
    
    	ps.executeUpdate();
    	
    	ps.close();
    	c.close();
    }
    ```
    

**getCount()**

- USER 테이블 레코드 개수를 돌려준다.
    
    ```java
    public int getCountO throws SQLException {
    	Connection c = dataSource.getConnectionO；
    
    	PreparedStatement ps = c.prepareStatement("select count(*) from users")；
    
    	ResultSet rs = ps.executeQueryO；
    	rs.nextO；
    	int count = rs.getlnt(1)；
    
    	rs.closeO；
    	ps.closeO；
    	c.closeO；
    
    	return count；
    }
    ```
    

### deleteAII()과 getCount()의 테스트

deleteAll()과 getCount()를 기존 addAndGet() 테스트에 추가한 것

```java
@Test 
public void addAndGet() throws SQLException {
	...

	dao.deleteAll();
	assertThat(dao.getCount(), is(0));

	User user = new User();
	user.setId("gyumee");
	user.setName("박성철");
	user.setPassword("springno1");

	dao.add(user);
	assertThat(dao.getCount(), is(1));

	User user2 = dao.get(user.getId());

	assertThat(user2.getName(), is(user.getName()));
	assertThat(user2.getPassword(), is(user.getPassword()));
}
```

### 동일한 결과를 보장하는 테스트

Now 테스트를 반복해서 여러 번 실행해도 계속 성공

**단위 테스트는 코드가 바뀌지 않는다면 매번 실행할 때마다 동일한 테스트 결과를 얻을 수 있어야 함 (결과가 보장돼야 함)**

## 2.3.3 포괄적인 테스트

### getCount() 테스트

- 테스트 메소드는 한 번에 한 가지 검증 목적에만 충실한 것이 좋음
- JUnit은 하나의 클래스 안에 여러 개의 테스트 메소드가 들어가는 것을 허용함
- @Test가 붙어 있고 public 접근자가 있으며 리턴 값이 void형이고
파라미터가 없다는 조건을 지키기만 하면 됨
    
    ```java
    @Test
    public void countO throws SQLException {
    	ApplicationContext context = new GenericXmlApplicationContext("applicationContext.xml")；
    
    	UserDao dao = context.getBean("userDao", UserDao.class)；
    	User userl = new User("gyumee", "박성철", "springnol")；
    	User user2 = new User("leegw700", "이길원", "springno2")；
    	User user3 = new User("bumjin", "박범진", "springno3")；
    
    	dao.deleteAll();
    	assertThat(dao.getCount(), is(0));
    
    	dao.add(user1)；
    	assertThat(dao.getCount(), is(1));
    
    	dao.add(user2)；
    	assertThat(dao.getCount(), is(2));
    
    	dao.add(user3)；
    	assertThat(dao.getCount(), is(3));
    }
    ```
    

### addAndGet() 테스트 보완

get()에 대한 테스트 기능 보완

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

### get() 예외조건에 대한 테스트

**get() 메소드에 전달된 id 값에 해당하는 사용자 정보가 없다면?**

- null과 같은 특별한 값을 리턴
- id에 해당하는 정보를 찾을 수 없다는 예외 처리

JUnit 예외조건 테스트를 위한 방법

1. 테스트 메소드를 추가
2. EmptyResultDataAccessException이 던져지면 성공, 아니라면 실패

```java
@Test(expected=EmptyResultDataAccessException.class)  // 테스트 중에 발생할 것으로 기대하는 예외 클래스를 지정해준다. 
public void getllserFailue() throws SQLException {
	ApplicationContext context = new GenericXmlApplicationContext("applicationContext.xml");

	UserDao dao = context.getBean("userDao", UserDao.class)；
	dao.deleteAllO；
	assertThat(dao.getCount(), is(0));

	dao.get("unknown_icr");  // 이 메소드 실행 중에 예외가 발생해야 한다. 예외가 발생하지 않으면 테스트가 실패한다.
}
```

**@Test 애노테이션의 expected 엘리먼트**

- expected는 테스트 메소드 실행 중에 발생하리라 기대하는 예외 클래스를 넣어주면 됨
- expected를 추가해놓으면
    - 정상적으로 테스트 메소드를 마치면 테스트 실패
    - expected에서 지정된 예외가 던져지면 테스트 성공

### 테스트를 성공시키기 위한 코드의 수정

리스트 2-14 데이터를 찾지 못하면 예외를 발생시키도록 수정한 get() 메소드

```java
public User get(String id) throws SQLException {
	ResultSet rs = ps.executeQuery();

	User user = null;  // —> User는 null 상태로 초기히해늏는다.

	// id를 조건으로 한 쿼리를 결과가 있으면 User 오브젝트를 만들고 값을 넣어준다. 
	if (rs.next()) {
		user = new User();
		user.setld(rs.getStringf'id"));
		user.setName(rs.getString("name"));
		user.setPassword(rs.getString("password"));
	}

	rs.close();
	ps.close();
	c.close(); 

	// 결과가 없으면 User는 null 상태 그대로일 것이다. 이를 확인해서 예외를 던져준다.
	if (user == null) throw new EmptyResultDataAccessException(1)；
	return user；
}
```

### 포괄적인 테스트

> 부정적인 케이스를 먼저 만드는 습관을 들이는 게 좋다
> 

## 2.3.4 테스트가 이끄는 개발 💨

### 기능설계를 위한 테스트

테스트 코드로 된 설계 문서처럼 만들어놓은 것

⇒ 실제 기능을 가진 애플리케이션 코드를 만들고 나면, 바로 이 테스트를 실행해서 설계한 대로 코드가 동작하는지를 빠르게 검증할 수 있음!

|  | 단계 | 내용 | 코드 |
|  |  |  |  |
| 조건 | 어떤 조건을 가지고 | 가져올 사용자 정보가 존재하지 않는 경우에 | dao.deleteAll();
assertThat(dao.getCount(), is(0)); |
| 행위 | 무엇을 할 때 | 존재하지 않는 id로 get()을 실행하면 | get(”unknown_id”); |
| 결과 | 어떤 결과가 나온다 | 특별한 예외가 던져진다.  | @Test(expected=EmptyResultDataAccessException.class) |

### 테스트 주도 개발 (TDD)

- **“실패한 테스트를 성공시키기 위한 목적이 아닌 코드는 만들지 않는다”**
- **테스트 코드를 먼저 만들고, 테스트를 성공하게 해주는 코드를 작성하는 방식의 개발 방법**
- TDD는 아예 테스트를 먼저 만들고 그 테스트가 성공하도록 하는 코드만 만드는 식으로 진행
- 테스트를 빼먹지 않고 꼼꼼하게 만들어낼 수 있다.
- == 테스트 우선 개발 Test First Development

**TDD의 장점**

- 코드를 만들어 테스트를 실행하는 그 사이의 간격이 매우 짧다
    - 오류는 빨리 발견 & 쉽게 대응 가능

## 2.3.5 테스트코드 개선

기계적으로 반복되는 부분 O ⇒ 중복된 코드는 별도의 메소드로 뽑아내자

### @Before

```java
import org.junit.Before;
...
public class UserDaoTest {
	private UserDao dao;
	// setUp() 메소드에서 만드는 오브젝트를 테스트 메소드에서 사용할 수 있도록 인스턴스 변수로 선언한다. 

	@Before  // JUnit이 제공하는 어노테이션. @Test 메소드가 실행되기 전에 먼저 실행돼야 하는 메소드를 정의한다. 
	public void setUpO {
		ApplicationContext context = new GenericXmlApplicationContext("applicationContext.xml")；
		this.dao = context.getBeanC'userDao", UserDao.class)；
		// 각 테스트 메소드에 반복적으로 나타났던 코드를 제거하고 별도의 메소드로 옮긴다. 
	}
 
	©Test
	public void addAndGetO throws SQLException {
		...	// 
	}

	@Test
	public void countO throws SQLException {
		...	
	}

	@Test(expected=EmptyResultDataAccessException.class)
	public void getUserFailureO throws SQLException {
		...	
	}
}
```

JUnit이 하나의 테스트 클래스를 가져와 테스트를 수행하는 방식

1. 테스트 클래스에서 @Test가 붙은 public이고 void형이며 파라미터가 없는 테스트 메소드를 모두 찾는다.
2. 테스트 클래스의 오브젝트를 하나 만든다.
3. @Before가 붙은 메소드가 있으면 실행한다.
4. @Test가 붙은 메소드를 하나 호출하고 테스트 결과를 저장해둔다.
5. @After가 붙은 메소드가 있으면 실행한다.
6. 나머지 테스트 메소드에 대해 2〜5번을 반복한다.
7. 모든 테스트의 결과를 종합해서 돌려준다.

- JUnit은 @Test가 붙은 메소드를 실행하기 전과 후에 각각 @Before와 @After가 붙은 메소드를 자동으로 실행함
- 공통적인 준비 작업과 정리 작업은 @Before나 @After가 붙은 메소드에 넣어두면 JUnit이 자동으로 메소드를 실행해 줌

```java
@Before
public void init() {
    LOG.info("startup");
    list = new ArrayList<>(Arrays.asList("test1", "test2"));
}

@After
public void finalize() {
    LOG.info("finalize");
    list.clear();
}
```

**@BeforeClass, @AfterClass**

테스트 클래스 실행 전과 실행 후 한번씩만 수행되는 메소드도 지원됨

```java
@BeforeClass
public static void setup() {
    LOG.info("startup - creating DB connection");
}

@AfterClass
public static void tearDown() {
    LOG.info("closing DB connection");
}
```

### 픽스처

- 테스트를 수행하는 데 필요한 정보나 오브젝트
- 일반적으로 픽스처는 여러 테스트에서 반복적으로 사용되기 때문에 @Before 메소드를 이용해 생성해두면 편리함
    
    ```java
    public class UserDaoTest {
    	private UserDao dao；
    	private User userl；
    	private User user2；
    	private User user3；
    
    	@Before
    	public void setUp() {
    		this.userl = new User("gyumee", "박성철", "springnol");
    		this.user2 = new User("leegw700", "이길원", "springnoZ");
    		this.user3 = new User("bumjin", "박범진", "springno3");
    	}
    }
    ```
    

# 2.4 스프링 테스트 적용

- 테스트는 가능한 한 독립적으로 매번 새로운 오브젝트를 만들어서 사용하는 것이 원칙
- 애플리케이션 컨텍스트처럼 생성에 많은 시간과 자원이 소모되는 경우에는 테스트 전체가 공유하는 오브젝트를 만들기도 함
- **애플리케이션 컨텍스트는 한 번만 만들고 여러 테스트가 공유해서 사용해도 됨**

## 2.4.1 테스트를 위한 애플리케이션 컨텍스트 관리

### 스프링 테스트 컨텍스트 프레임워크 적용

**@RunWith**

- JUnit 프레임워크의 테스트 실행 방법을 확장할 때 사용하는 어노테이션
- SpringJUnit4ClassRunner라는 JUnit용 테스트 컨텍스트 프레임워크 확장 클래스를 지정해주면 JUnit이 테스트를 진행하는 중에 테스트가 사용할 애플리케이션 컨텍스트를 만들고 관리하는 작업을 진행해줌

**@ContextConfiguration**

- `@ContextConfiguration`은 자동으로 만들어줄 애플리케이션 컨텍스트의 설정파일 위치를 지정한 것
    
    ```java
    @RunWith(SpringJUnit4ClassRunner.class)  // 스프링의 테스트 컨텍스트 프레임워크의 JUnit 확장가능 지정
    @ContextConfiguration(locations="/application.xml")  // 테스트 컨텍스트가 자동으로 만들어줄 애플리케이션 컨텍스트의 위치 지정
    public class UserDaoTest {
    	@Autowired
    	private ApplicationContext context;  // 테스트 오브젝트가 만들어지고 나면 스프링 테스트 컨텍스트에 의해 자동으로 값이 주입된다. 
    	...
    
    	@Before
    	public void setUp() {
    		this.dao = this.context.getBean("userDao", UserDao.class);
    		...
    	}
    
    	...
    }
    ```
    

**@DirtiesContext**

- 테스트 메소드에서 애플리케이션 컨텍스트의 구성이나 상태를 변경한다는 것을 테스트 컨텍스트 프레임워크에 알려줌
    
    ```java
    @DirtiesContext
    public class UserDaoTest {
    	@Autowired
    	UserDao dao;
    
    	@Before
    	public void setUp() {
    		...
    		DataSource dataSource = new SingleConnetionDataSource(
    			"jdbc:mysql://localhost/testdb". "spring", "book", true);
    	}
    ...
    ```
    
- 테스트에 필요한 빈 설정을 다르게 하고 싶다면 테스트용 applicationcontext.xml 파일을 생성하여 지정해줄 수 도 있다.
    
    ```java
    @RunWith(SpringUnitClassRunner.class)
    @ContextConfiguration(locations="/test-applicationContext.xml")
    public class UserDaoTest {
    	...
    ```
    

### 테스트 메소드의 컨텍스트 공유

- 최초로 애플리케이션 컨텍스트가 처음 만들어 지면서 가장 오랜 시간이 소모
- 그 다음부터는 이미 만들어진 애플리케이션 컨텍스트를 재사용
    
    ⇒ 테스트 실행 시간이 매우 짧아짐
    
    ⇒ 하나의 테스트 클래스 내의 테스트 메소드는 같은 애플리케이션 컨텍스트를 공유해서 사용할 수 있음
    

### 테스트 클래스의 컨텍스트 공유

수백 개의 테스트 클래스를 만들었는데 모두 같은 설정파일을 사용한다고 해도 테스트 전체에 걸쳐 단 한 개의 애플리케이션 컨텍스트만 만들어져 사용

⇒ 테스트 성능이 대폭 향상

### @Autowired

스프링의 이에 사용되는 특별한 애노테이션

## 2.4.2 DI와 테스트

### DI를 이용한 테스트 방법 선택

1. 항상 스프링 컨테이너 없이 테스트할 수 있는 방법을 우선적으로 고려하기
2. 여러 오브젝트와 복잡한 의존관계를 갖고 있는 오브젝트를 테스트해야할 경우라면 스프링의 설정을 이용한 DI 방식의 테스트를 이용하면 편리하다.

# 2.5 학습테스트로 배우는 스프링

## 2.5.1 학습테스트의장점

- 다양한 조건에 따른 기능을 손쉽게 확인해볼 수 있다
- 학습 테스트 코드를 개발 중에 참고할 수 있다
- 프레임워크나 제품을 업그레이드할 때 호환성 검증을 도와준다
- 테스트 작성에 대한 좋은 훈련이 된다
- 새로운 기술을 공부하는 과정이 즐거워진다

## 2.5.2 학습 테스트 예제

- UserDao와 DB 커넥션 생성 클래스 사이에는 DataSource라는 인터페이스 생성
- UserDao는 자신이 사용하는 오브젝트의 클래스가 무엇인지 알 필요 없음
- && DI를 통해 외부에서 사용할 오브젝트를 주입받기 때문에 오브젝트 생성에 대한 부담 ❌

⇒ 반문의 반문: 그래도 인터페이스를 두고 DI를 적용 해야 함!

### 인터페이스를 두고 DI를 적용해야 하는 이유

1. 소프트웨어 개발에서 절대로 바뀌지 않는 것은 없다.
2. 클래스의 구현 방식은 바뀌지 않는다고 하더라도 인터페이스를 두고 DI를 적용하게 해두면 다른 차원의 서비스 기능을 도입할 수 있다.
3. 테스트 때문이다. 효율적인 테스트를 손쉽게 만들기 위해서 DI를 사용해야 한다.

## 2.5.3 버그 테스트

- 테스트의 완성도를 높여준다
- 버그의 내용을 명확하게 분석하게 해준다
- 기술적인 문제를 해결하는 데 도움이 된다

## 정리

- 테스트는 자동화 돼야 하고, 빠르게 실행할 수 있어야 함
- main() 테스트 대신 JUnit 프레임워크를 이용한 테스트 작성이 편리함
- 테스트 결과는 일관성이 있어야 함. 코드의 변경 없이 환경이나 테스트 실행 순서에 따라서 결과가 달라지면 안됨
- 테스트는 포괄적으로 작성해야 함. 충분한 검증을 하지 않는 테스트는 없는 것보다 나쁠 수 있음
- 코드 작성과 테스트 수행의 간격이 짧을수록 효율적
- 테스트하기 쉬운 코드가 좋은 코드
- 테스트를 먼저 만들고 테스트를 성공시키는 코드를 만들어 가는 테스트 주도 개발 방법도 유용함
- 테스트 코드도 애플리케이션 코드와 마찬가지로 적절한 리팩토링이 필요하다.
- `@Before`, `@After`를 사용해서 테스트 메소드들의 `공통 준비 작업`과 `정리 작업`을 처리할 수 있음
- 스프링 테스트 컨텍스트 프레임워크를 이용하면 테스트 성능을 향상시 킬 수 있다.
- 동일한 설정파일을 사용하는 테스트는 하나의 애플리케이션 컨텍스트를 공유한다.
- @Autowired를 사용하면 컨텍스트의 빈을 테스트 오브젝트에 이 할 수 있다.
- 기술의 사용 방법을 익히고 이해를 돕기 위해 학습 테스트를 작성하자.
- 오류가 발견될 경우 그에 대한 `버그 테스트`를 만들어두면 유용
