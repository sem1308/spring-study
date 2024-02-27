# 2주차

# **1.5 스프링의 IoC**

## **1.5.1** 오브젝트 팩토리를 이용한 스프링 **loC**

스프링 빈

- 스프링이 제어권을 가지고 직접 만들고 관계를 부여하는 오브젝트

빈 팩토리

- 빈의 생성과 관계설정 같은 제어를 담당하는 IoC 오브젝트

애플리케이션 컨텍스트

- 빈 팩토리를 확장한 것

### 빈 팩토리 vs 애플리케이션 컨텍스트

빈 팩토리는 빈 오브젝트의 생성, 관계설정 등의 제어 작업을 총괄한다. 이에 반해 애플리케이션 컨텍스트는 별도로 설정정보를 담고 있는 무엇인가를 가져와 이를 활용하는 범용적인 loC엔진이라고 보면 된다.

## DaoFactory를 사용하는 애플리케이션 컨텍스트

### DaoFactory

- 스프링의 빈 팩토리가 사용할 수 있는 **설정정보**

### ©Configuration

- 스프링이 빈 팩토리를 위한 **오브젝트 설정을 담당하는 클래스**로 인식할 수 있도록 선언하는 애노테이션

### @Bean

- **오브젝트(빈)을 만들어주는** 애노테이션

**DaoFactory**

```java
@Configuration
public class DaoFactory{
	@Bean
	public UserDao userDao(){
		UserDao userDao = new UserDao(connectionMaker());
		return userDao;		
	}

	@Bean
	public ConnectionMaker connectionMaker(){
		return new DConnectionMaker();
	}
}
```

- @Configuration 선언하여 설정을 담당하는 클래스로 인식되게함
- @Bean 선언하여 UserDao와 DConnectionMaker를 스프링 빈으로 등록

### AnnotationConfigApplicationContext

- @Configuration이 붙은 자바 코드를 설정정보로 사용하기 위해 사용한다.
- 생성자 파라미터에 설정 클래스를 넣어준다.
- getBean() 메소드를 사용해 원하는 빈 오브젝트를 가져올 수 있다.

**사용 예시**

```java
public class UserDaoTest {
	public static void main(String[] args) throws ClassNotFoundException, SQLException {
		ApplicationContext context = new AnnotationConfigApplicationContext(DaoFactory.class);
		UserDao userDao = context.getBean("userDao", UserDao.class);		

	}
}
```

## **1.5.2** 애플리케이션 컨텍스트의 동작방식

### DaoFactory vs 애플리케이션 컨텍스트

- DaoFactory가 UserDao를 비롯한 Dao 오브젝트를 생성하고 DB 생성 오브젝트와 관계를 맺어주는 역할
- 애플리케이션 컨텍스트는 애플리케이션에서 loC를 적용해서 관리할 모든 오브젝트에 대한 생성과 관계설정을 담당하는 역할

### 애플리케이션 컨텍스트의 장점

- 클라이언트는 구체적인 팩토리 클래스를 알 필요가 없다.
- 종합 **loC** 서비스를 제공해준다.
- 빈을 검색하는 다양한 방법을 제공한다.

## **1.5.3** 스프링 **loC**의 용어 정리

### 빈(Bean)

- 스프링이 loC 방식으로 관리하는 오브젝트
- 스프링을 사용하는 애플리케이션에서 만들어지는 모든 오브젝트가 다 빈은 아니고 그 중 스프링이 직접 그 생성과 제어를 담당하는 오브젝트만을 빈이라고 한다.

### 빈 팩토리(Bean factory)

- 스프링의 IoC를 담당하는 핵심 컨테이너이다.
- 빈을 등록, 생성, 조회하고 돌려주고, 그 외에 부가적인 빈을 관리하는 기능을 담당한다.

### 애플리케이션 컨텍스트(Application context)

- 빈 팩토리를 확장한 loC 컨테이너이다.
- 빈 팩토리에 스프링이 제공하는 각종 부가 서비스를 추가로 제공한다.

### 설정정보(configuration)

- 애플리케이션 컨텍스트 또는 빈 팩토리가 loC를 적용하기 위해 사용하는 메타정보이다.

### 컨테이너(container)

- IoC 방식으로 빈을 관리한다는 차원에서 애플리케이션 컨텍스트나 빈 팩토리를 컨테이너라고 한다.

### 스프링 프레임워크

- loC 컨테이너, 애플리케이션 컨텍스트를 포함해서 스프링이 제공하는 모든 기능을 통틀어 말할 때 주로 사용한다.

# 1.6 싱글톤 레지스트리와 오브젝트 스코프

스프링의 애플리케이션 컨텍스트는 기존에 직접 만들었던 오브젝트 팩토리와 중요한 차이점이 있는데 과연 어떤 차이점이 있는지 확인해보자!

### **DaoFactory의 userDao()를 두 번 호출해서 리턴되는 UserDao 비교**

- 각기 다른 값을 가진 동일하지 않은 오브젝트이다.
- 매 번 new를 통해 새로운 객체를 생성하기 때문이다.

### 애플리케이션 컨텍스트에 DaoFactory를 설정 정보 등록 후 getBean()을 두 번 호출해서 UserDao 비교

- 두 오브젝트가 동일하다.
- 빈 생성 전략이 싱글톤 방식을 따르기 때문이다.

## **1.6.1** 싱글톤 레지스트리로서의 애플리케이션 컨텍스트

애플리케이션 컨텍스트는 IoC 컨테이너이면서 싱글톤 레지스트리다.

### 스프링이 싱글톤으로 빈을 만드는 이유?

- 서버 환경에서의 부하를 막아 성능을 높이기 위함이다.

### 싱글톤 패턴의 한계

- **private** 생성자를 갖고 있기 때문에 상속할 수 없다.
- 테스트하기가 어렵다.
- 서버환경에서 싱글톤이 하나만 만들어지는 것을 보장하지 못한다.
- 싱글톤의 사용은 전역 상태를 만들 수 있기 때문에 바람직하지 못하다.

### **싱글톤 레지스트리**

싱글톤 형태의 오브젝트를 만들고 관리하는 기능을 제공하는 것

## **1.6.2** 싱글톤과 오브젝트의 상태

싱글톤은 멀티스레드 환경에서 상태 정보를 내부에 갖고 있지 않은 **무상태 방식**으로 만들어야 한다.

인스턴스 변수를 사용하는 UserDao

```java
public class UserDao{
	private ConnectionMaker connectionMaker; // 초기 설정 후 바뀌지 않음
	private Connection c; // 매번 새로운 값으로 바뀌는 정보가 저장 (위험!)
	private User user; // 매번 새로운 값으로 바뀌는 정보가 저장 (위험!)

	public Uesr get(String id) throws ClassNotFoundException, SQLException {
		this.c = connectionMaker.makeConnection();
		...
		this.user = new User();
		...
		return this.user;
	}

}
```

- 기존에 로컬 변수였던 Connection과 User를 클래스의 인스턴스 필드로 선언
- ConnectionMaker는 읽기전용이기에 인스턴스 변수로 사용해도 괜찮다.
    - 또한 자신이 사용하는 다른 싱글톤 빈을 저장하려는 용도라면 인스턴스 변수를 사용해도 된다.

## **1.6.3** 스프링 빈의 스코프

### **빈 스코프**

- 스프링 빈이 생성되고 존재하고 적용되는 범위를 의미한다.
- 스프링 빈의 **기본 스코프**는 **싱글톤**이다.

# **1.7** 의존관계주입(DI)

## 1.7.1 제어의 역전(loC)과 의존관계 주입

스프링 loC 기능의 **대표적인 동작원리**는 주로 **의존관계 주입**이라고 불린다.

스프링의 기능은 **의존관계 주입**이라는 새로운 용어를 사용할 때 분명하게 드러난다.

## **1.7.2** 런타임 의존관계 설정

### 의존관계

- 어떤 대상 A가 변하면 그것이 상대방 B에게 영향을 미친다면 B는 A를 의존한다라고 말한다.
- 방향성이 있는 관계

### UserDao의 의존관계

- UserDao가 ConnectionMaker에 의존한다.
- 그러므로 ConnectionMaker 인터페이스가 변한다면 그 영향을 UserDao
가 직접적으로 받게된다.
- 하지만 ConnectionMaker 인터페이스를 구현한 클래스로 바뀐다면 영향을 받지  않는다. (느슨한 결합)

### 의존 오브젝트

- 프로그램이 시작되고 UserDao 오브젝트가 만들어지고 나서 런타임 시에 의존관계를 맺는 대상
- 의존관계 주입은 이런 의존 오브젝트와 이를 실제로 사용할 주체를 런타임시에 연결해준다.

### 의존관계 주입 3가지 조건

- 인터페이스에 의존하여 클래스 모델이나 코드에는 런타임 시점의 의존관계가 드러나지 않게한다.
- 런타임 시점의 의존관계는 컨테이너나 팩토리 같은 제3의 존재가 결정한다.
- 의존관계는 사용 할 오브젝트를 외부(DaoFactory)에서 주입해줌으로써 만들어진다.

### UserDao와 ConnectionMaker의 문제점

```java
public UserDao {
	ConnectionMaker = new DConnectionMaker();
}
```

- UserDao가 사용할 구체적인 클래스를 알고 있어야 한다.

### 생성자 파라미터를 통해 의존관계 주입

```java
public UserDao(ConnectionMaker ConnectionMaker) {
	this.ConnectionMaker = ConnectionMaker；
}
```

- 생성자를 이용해 파라미터로 구체적인 클래스를 주입받아 런타임 의존관계가 만들어 졌다.

## **1.7.3** 의존관계 검색과 주입

### 의존관계 검색

- 외부로부터의 주입받지 않고 스스로 검색하여 의존관계를 맺는 방법이다.
- 자신이 어떤 클래스의 오브젝트를 사용할 것인지 결정하는 것이 아니다.
- 단순히 의존관계 설정된 오브젝트를 가져올 때 메소드나 생성자를 통한 주입 대신 스스로 컨테이너에게 요청하는 것이다.

**단순한 예시**

```java
public UserDao() {
	DaoFactory daoFactory = new DaoFactory()；
	this.ConnectionMaker = daoFactory.connectionMaker()；
}
```

- 메소드를 호출하여 런타임 의존관계를 맺을 오브젝트를 검색한다.

**애플리케이션 컨텍스트를 통해 의존관계 검색을 사용한 UserDao 생성자**

```java
public UserDaoO {
	AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(DaoFactory.class)；
	this.ConnectionMaker = context.getBean("ConnectionMaker"z ConnectionMaker.class)；
}
```

- getBean() 메소드를 의존관계 검색에 사용한다.

### 의존관계 검색(DL) vs 의존관계 주입(DI)

- 의존관계 검색 방식에서 검색하는 오브젝트는 자신이 스프링의 빈일 필요가 없다.
- 의존관계 주입에서는 UserDao와 ConnectionMaker 사이에 주입받을 UserDao도 반드시 컨테이너가 만드는 빈 오브젝트여야한다.

## **1.7.5 메소드를 이용한 의존관계 주입**

지금까지는 UserDao의 의존관계 주입을 위해 생성자를 사용했는데 의존관계 주입 시 반드시 생성자를 사용해야 하는 것은 아니다.

### 수정자 메소드를 이용한 주입

- 외부로부터 제공받은 오브젝트를 저장해뒀다가 내부의 메소드에서 사용하게 하는 Dl 방식에서 활용하기에 적당하다.
- 메소드는 항상 set으로 시작한다.
- 한 번에 1개의 파라미터를 가진다.

### 일반 메소드를 이용한 주입

- 한 번에 여러 개의 파라미터를 받을 수 있다.

**수정자 메소드를 사용한 UserDao**

```java
public class UserDao {
	private ConnectionMaker connectionMaker;
	
	@Bean
	public UserDao userDao() {
		UserDao userDao = new UserDao()；
		userDao.setConnectionMaker(connectionMaker())；
		return userDao；
	}    

  // 수정자 메소드 DI의 전형적인 코드다. 잘 기억해두자.
	public void setConnectionMaker(ConnectionMaker connectionMaker) {
		this.connectionMaker = ConnectionMaker;
  }
  ...
}
```

## **1.8 XML**을 이용한 설정

본격적인 범용 DI 컨테이너를 사용하면서 오브젝트 사이의 의존정보를 일일이 DaoFactory처럼 자바 코드로 만들어주려면 번거롭다. 

스프링은 자바 클래스를 사용하는 것 외에도 다양한 방법을 통해 **DI 의존관계 설정정보**를 만들 수 있는데 대표적으로 **XML**이 있다.

## **1.8.1 XML** 설정

하나의 @Bean 메소드를 통해 얻을 수 있는 **빈의 DI 정보**

- 빈의 이름: @Bean 메소드 이름이 빈의 이름이다. getBean()에서 사용된다.
- 빈의 클래스: 빈 오브젝트를 어떤 클래스를 이용해서 만들지 정의한다.
- 빈의 의존 오브젝트: 빈의 생성자나 수정자 메소드를 통해 의존 오브젝트를 넣어준다.

XML 파일은 **〈beans〉**를 **루트 엘리먼트**로 사용한다.

**<beans>** 안에는 **여러 개의 <bean>**을 정의할 수 있다.

### connectionMaker() 전환

| 빈 설정파일 | @Configuration | < beans> |
| --- | --- | --- |
| 빈의 이름 | @Bean methodName() | <bean id="methodName" |
| 빈의 클래스 | return new BeanClass(); | class="a.b.c...BeanClass"> |
- <bean> 태그의 class 속성에 지정하는 것은 자바 오브젝트를 만들 때 사용하는 클래스 이름이다.
    - 클래스 이름을 작성할 때 패키지까지 포함해야 한다.
- XML에서는 리턴하는 타입을 지정하지 않아도 된다.

### userDao() 전환

**<property name=”” ref=”” />**

- 의존 오브젝트와의 관계를 정의한다.
- name은 수정자 메소드의 프로퍼티 이름을 의미한다.
- ref는 주입할 오브젝트를 정의한 빈의 id를 의미한다.

**UserDao 빈 설정**

```xml
userDao.setConnectionMaker(ConnectionMaker())；

============================================================

<bean id = "userDao" class="springbook.dao.UserDao"〉
	<property name="connectionMaker" ref="connectionMaker" />
</bean>
```

### XML의 의존관계 주입 정보

**완성된 XML 설정정보**

```xml
<beans>
	<bean id="ConnectionMaker" class-'springbook.user.dao.DConnectionMaker" />
	<bean id="userDao" class="springbook.user.dao.UserDao">
		<property name="connectionMaker" ref="connectionMaker" />
	</bean>
</beans>
```

- 2개의 〈bean〉 태그를 이용해 @Bean 메소드를 모두 XML로 변환했다.

**같은 인터페이스 타입의 빈을 여러 개 정의한 경우**

```xml
<beans>
	<bean id="localDBConnectionMaker" class="...LocalDBConnectionMaker" />
	<bean id="testDBConnectionMaker" class="...TestDBConnectionMaker" />
	<bean id="productionDBConnectionMaker" class="...ProductionDBConnectionMaker" />
  
	<bean id="userDao" class="springbook.user.dao.UserDao">
		<property name="connectionMaker" ref="localDBConnectionMaker" />
	</bean>
</beans>
```

1.  빈의 이름을 독립적으로 만들어둔다.
2. ref 애트리뷰트를 이용해 DI 받을 빈을 지정한다.

## **1.8.2 XML**을 이용하는 애플리케이션 컨텍스트

애플리케이션 컨텍스트가 DaoFactory 대신 **XML 설정정보**를 활용하도록 만들어보자!

XML에서 빈의 의존관계 정보를 이용하는 **GenericXmlApplicationContext**를 사용해야한다.

**XML 설정정보를 담은 applicationContext.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
			http://www.springframework.Org/schema/beans/spring-beans-3.0.xsd">
	<bean id="connectionMaker" class="springbook.user.dao.DConnectionMaker" />
  
	<bean id="userDao" class="springbook.user.dao.UserDao">
		<property name="connectionMaker" ref="connectionMaker" />
	</bean>
</beans>
```

**UserDaoTest의 애플리케이션 컨텍스트 생성 부분 수정**

```java
ApplicationContext context = new GenericXmlApplicationContext("applicationContext.xml");
```

- AnnotationConfigApplicationContext 대신 **GenericXmlApplicationContext**를 이용해 애플리케이션 컨텍스트를 생성한다.
- 생성자에는 **applicationContext.xml**의 클래스패스를 넣는다.

## **1.8.3 DataSource** 인터페이스로 변환

### DataSource 인터페이스 적용

자바에서는 DB 커넥션을 가져오는 오브젝트의 기능을 추상화해서 비슷한 용도로 사용할 수 있게 만들어진 **DataSource**라는 **인터페이스**가 이미 존재한다.

**DataSource 인터페이스**

```java
package javax.sql

public interface DataSource extends CommonDataSource,Wrapper {
	Connection getConnection() throws SQLException;
    ...
}
```

### UserDao 리팩토링

**DataSource를 사용하는 UserDao**

```java
import javax.sql.DataSource;

public class UserDao {
	private DataSource dataSource;
    
	public void setDataSource(DataSource dataSource) {
		this.dataSource = dataSource;
	}
    
	public void add(User user) throws SQLException {
		Connection c = dataSource.getConnection();
        ...
	}
    ...
}
```

- UserDao에 주입될 의존 오브젝트의 타입을 ConnectionMaker에서 DataSource로 수정한다.
- DB 커넥션을 가져오는 코드를 makeConnection()에서 getConnection() 메소드로 수정한다.

### 1. 자바 코드 설정 방식

**DataSource 타입의 dataSource 메소드**

```java
@Bean
public DataSource dataSource() {
	SimpleDriverDataSource dataSource = new SimpleDriverDataSource();
    
  // DB 연결정보를 수정자 메소드를 통해 넣어준다.
	dataSource.setDriverClass(com.mysql.jdbc.Driver.class);
	dataSource.setUrl("jdbc:mysql://localhost/springbook");
	dataSource.setUsername("spring");
	dataSource.setPassword("book");
  // 이렇게 하면 오브젝트 레벨에서 DB 연결 방식을 변경할 수 있다.
    
  return dataSource;
}
```

- 기존의 connectionMaker() 메소드를 dataSource()로 변경하고 SimpleDriverDataSource의 오브젝트를 리턴한다.
- DB 연결과 관련된 정보를 수정자 메소드를 이용해 설정한다.

 **DataSource 타입의 빈을 DI 받는 userDao() 메소드**

```java
@Bean
public UserDao userDao() {
	UserDao userDao = new UserDaoO;
	userDao.setDataSource(dataSource());
	return userDao;
}
```

### 2. XML 설정 방식

**dataSource 빈**

```xml
<bean id="dataSource" class="org.springframework.jdbc.datasource.SimpleDriverDataSource" />
```

- id가 ConnectionMaker인〈bean〉을 없애고 dataSource라는 이름의〈bean>을 등록한다.
- 클래스를 SimpleDriverDataSource로 변경한다.

**문제점**

- DB 접속정보가 나타나 있지 않다.

## **1.8.4** 프로퍼티 값의 주입

**XML을 이용한 DB 연결정보 설정**

```xml
<property name="driverClass" value="com.mysql.jdbc.Driver" />
<property name="url" value="jdbc:mysql://localhost/springbook" />
<property name="username" value="spring" />
<property name="password" value="book" />
```

- 텍스트나 단순 오브젝트 등을 수정자 메소드에 넣어주는 것을 스프링에서는 “값을 주
입한다”고 말한다.
- 다른 빈 오브젝트의 레퍼런스(ref)가 아니라 단순 값(value)을 주입해주는 것이기 때문에 ref 속성 대신 **value 속성**을 사용한다.

### value 값의 자동 변환

스프링은 프로퍼티의 값을, 수정자 메소드의 파라미터 타입을 참고로 해서 적절한 형태로 변환한다.

**DataSource를 적용 완료한 최종 applicationContext.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
		http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">

	<bean id="dataSource"
		class="org.springframework.jdbc.datasource.SimpleDriverDataSource">
		<property name="driverClass" value="com.mysql.jdbc.Driver" />
		<property name="url" value="jdbc:mysql://localhost/springbook" />
		<property name="username" value="spring" />
		<property name="password" value="book" />
	</bean>

	<bean id="userDao" class="springbook.user.dao.UserDao">
		<property name="dataSource" ref="dataSource" />
	</bean>
</beans>
```
