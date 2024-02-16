## 스프링의 IoC
### 오브젝트 팩토리를 이용한 스프링 IoC
#### 애플리케이션 컨텍스트와 설정정보
스프링이 제어권을 가지고 직접 만들고 관계를 부여하는 오브젝트, 빈(bean)

빈의 생성과 관계설정 같은 제어를 담당하는 IOC 오브젝트, 빈 팩토리(bean factory)

애플리케이션 전반에 걸쳐 모든 구성요소의 제어 작업을 담당하는 IoC 엔진, 애플리케이션 컨텍스트(application context)
#### DaoFactory를 사용하는 애플리케이션 컨텍스트
`@Configuration`: 빈 팩토리를 위한 오브젝트 설정을 담당하는 클래스

`@Bean`: 오브젝트를 만들어 주는 메소드
```java
/*스프링 빈 팩토리가 사용할 설정정보를 담은 DaoFactory 클래스*/
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
...
@Configuration // 애플리케이션 컨텍스트 또는 빈 팩토리가 사용할 설정정보라는 표시
public class DaoFactory {
	@Bean // 오브젝트 생성을 담당하는 IoC용 메소드라는 표시
	public UserDao userDaoO {
		return new UserDao(connectionMaker())；
	}
    
	@Bean
	public ConnectionMaker connectionMakerO {
		return new DConnectionMaker()；
	}
}
```
`Applicationcontext`의 `getBean()` 메소드를 이용해 `UserDao`의 오브젝트를 가져올 수 있다.
```java
/*애플리케이션 컨텍스트를 적용한 UserDaoTest*/
public class UserDaoTest {
	public static void main(String[] args) throws ClassNotFoundException, SQLException {
		Applicationcontext context = new AnnotationConfigApplicationContext(DaoFactory.class);
		UserDao dao = context.getBean("userDao", UserDao.class);
        ...
	}
}
```
`getBean()`
- 첫 번째 파라미터: `ApplicationContext`에 등록된 빈의 이름
- 두 번째 파라미터: 오브젝트의 리턴 타입
### 애플리케이션 컨텍스트의 동작방식
`DaoFactory`: `UserDao`를 비롯한 DAO 오브젝트를 생성하고 DB 생성 오브젝트와
관계를 맺어주는 제한적인 역할

`ApplicationContext`: 애플리케이션에서 IoC를 적용해서 관리할 모든 오브젝트에 대한 생성과 관계설정을 담당

애플리케이션 컨텍스트의 장점
- 클라이언트는 구체적인 팩토리 클래스를 알 필요가 없다.
- 애플리케이션 컨텍스트는 종합 IoC 서비스를 제공해준다.
- 애플리케이션 컨텍스트는 빈을 검색하는 다양한 방법을 제공한다.
### 스프링 IoC의 용어 정리
빈(bean): 스프링이 IoC 방식으로 관리하는 오브젝트

빈 팩토리(bean factory): 스프링의 IoC를 담당하는 핵심 컨테이너

애플리케이션 컨텍스트(application context): 빈 팩토리를 확장한 IoC 컨테이너

설정정보/설정 메타정보(configuration metadata): 애플리케이션 컨텍스트 또는 빈 팩토리가 IoC를 적용하기 위해 사용하는 메타정보

컨테이너(container, IoC 컨테이너): IoC 방식으로 빈을 관리한다는 의미에서의 애플리케이션 컨텍스트나 빈 팩토리

스프링 프레임워크: IoC 컨테이너, 애플리케이션 컨텍스트를 포함해서 스프링이 제공하는 모든 기능을 통틀어 일컫는 말
## 싱글톤 레지스트리와 오브젝트 스코프
스프링은 여러 번에 걸쳐 빈을 요청하더라도 매번 동일한 오브젝트를 돌려준다.

`getBean()`을 실행할 때마다 `userDao()` 메소드를 호출하고, 매번 `new`에 의해 새로운 `UserDao`가 만들어지지 않는다.
### 싱글톤 레지스트리로서의 애플리케이션 컨텍스트
애플리케이션 컨텍스트는 싱글톤을 저장하고 관리하는 싱글톤 레지스트리(singleton registry)이다.

디자인 패턴에서의 싱글톤 패턴과는 다름
#### 서버 애플리케이션과 싱글톤
서블릿 클래스당 하나의 오브젝트만 만들어두고, 사용자의 요청을 담당하는 여러 스레드에서 하나의 오브젝트를 공유해 동시에 사용
#### 싱글톤 패턴의 한계
- `private` 생성자를 갖고 있기 때문에 상속할 수 없다.
- 싱글톤은 테스트하기가 힘들다.
- 서버환경에서는 싱글톤이 하나만 만들어지는 것을 보장하지 못한다.
- 싱글톤의 사용은 전역 상태를 만들 수 있기 때문에 바람직하지 못하다.
#### 싱글톤 레지스트리
싱글톤 레지스트리(singleton registry): 싱글톤 패턴의 구현 방식의 여러 가지 단점으로 인해 스프링은 직접 싱글톤 형태의 오브젝트를 만들고 관리하는 기능을 제공
### 싱글톤과 오브젝트의 상태
주의할 점
- 싱글톤이 멀티스레드 환경에서 서비스 형태의 오브젝트로 사용되는 경우에 상태정보를 내부에 갖고 있지 않은 무상태(stateless) 방식으로 만들어져야 한다.
- 읽기전용의 값이라면 초기화 시점에서 인스턴스 변수에 저장해두고 공유하는 것은 괜찮다.
### 스프링 빈의 스코프
스코프(scope): 빈이 생성되고, 존재하고, 적용되는 범위

스프링 빈의 기본 스코프는 싱글톤이다.

프로토타입(prototype) 스코프: 컨테이너에 빈을 요청할 때마다 매번 새로운 오브젝트를 생성

요청(request) 스코프: 웹을 통해 새로운 HTTP 요청이 생길 때마다 생성

세션(session) 스코프: 웹의 세션과 스코프가 유사
## 의존관계 주입(DI)
### 제어의 역전(IoC)과 의존관계 주입
의존관계 주입(Dependency Injection): 오브젝트 레퍼런스를 외부로부터 제공(주입) 받고 이를 통해 여타 오브젝트와 다이내믹하게 의존관계가 만들어지는 것
### 런타임 의존관계 설정
#### 의존관계
A가 B에 의존: B가 변하면 그것이 A에 영향을 미친다.

의존관계에는 방향성이 있다. - A가 B에 의존하고 있다고 해서 B가 A에 의존하지 않는다.
#### UserDao의 의존관계
`UserDao`가 `ConnectionMaker`에 의존한다.

의존 오브젝트(dependent object): 런타임 시에 의존관계를 맺는 대상, 즉 실제 사용대상인 오브젝트

의존관계 주입
- 클래스 모델이나 코드에는 런타임 시점의 의존관계가 드러나지 않는다. 인터페이스에만 의존하고 있어야 한다.
- 런타임 시점의 의존관계는 컨테이너나 팩토리 같은 제3의 존재가 결정한다.
- 의존관계는 사용할오브젝트에 대한 레퍼런스를 외부에서 제공(주입) 해줌으로써 만들어진다.
#### UserDao의 의존관계 주입
```java
/*관계설정 책임 분리 전의 생성자*/
public UserDao() {
	ConnectionMaker = new DConnectionMaker();
}
```
클래스 모델의 의존관계이므로 코드에 반영되고, 런타임 시점에서도 변경되지 않는다.
```java
/*의존관계 주입을 위한 코드*/
public class UserDao {
	private ConnectionMaker connectionMaker;
    
	public UserDao(ConnectionMaker connectionMaker) {
		this.ConnectionMaker = connectionMaker;
    }
    ...
}
```
DI 컨테이너가 자신이 결정한 의존관계를 맺어줄 클래스의 오브젝트를 만들고 이 생성자의 파라미터로 오브젝트의 레퍼런스를 전달해준다.
### 의존관계 검색과 주입
의존관계 검색(dependency lookup): 의존관계를 맺는 방법이 외부로부터의 주입이 아니라 스스로 검색을 이용
```java
/*DaoFactory를 이용하는 생성자*/
public UserDao() {
	DaoFactory daoFactory = new DaoFactory();
	this.ConnectionMaker = daoFactory.connectionMaker();
}
```
스스로 IoC 컨테이너인 `DaoFactory`에게 요청했다.
```java
/*의존관계 검색을 이용하는 UserDao 생성자*/
public UserDao() {
	AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(DaoFactory.class);
	this.ConnectionMaker = context.getBean("connectionMaker", ConnectionMaker.class);
}
```
애플리케이션 컨텍스트를 사용해서 의존관계 검색 방식으로 `ConnectionMaker` 오브젝트를 가져오게 만들었다.

`main()` 메소드와 비슷한 역할을 하는 서블릿에서 스프링 컨테이너에 담긴 오브젝트를 사용하려면 한 번은 의존관계 검색 방식을 사용해 오브젝트를 가져와야 한다.

- 의존관계 검색 방식에서는 검색하는 오브젝트는 자신이 스프링의 빈일 필요가 없다.
- 의존관계 주입에서는 `UserDao`와 `ConnectionMaker` 사이에 DI가 적용되려면 UserDao도 반드시 컨테이너가 만드는 빈 오브젝트여야 한다.
### 의존관계 주입의 응용
#### 기능 구현의 교환
개발 중에는 개발자 PC에 설치한 로컬 DB로 사용

DI 적용 X - 로컬 DB에 대한 연결 기능이 있는 `LocalDBConnectionMaker`라는 클래스를 만들고, 모든 DAO에서 이 클래스의 오브젝트를 매번 생성해서 사용

DI 적용 - 사용 클래스 이름은 컨테이너가 사용할 설정정보에 들어 있음
```java
/*개발용 ConnectionMaker 생성 코드*/
@Bean
public ConnectionMaker connectionMaker() {
	return new LocalDBConnectionMaker();
}
```
```java
/*운영용 ConnectionMaker 생성 코드*/
@Bean
public ConnectionMaker connectionMaker() {
	return new ProductionDBConnectionMaker();
}
```
#### 부가기능 추가
DAO가 DB를 얼마나 많이 연결해서 사용하는지 파악

DI 적용 X - 모든 DAO의 `makeConnection()` 메소드를 호출하는 부분에 새로 추가한 카운터를 증가시키는 코드를 넣고, 분석 작업이 끝나면 모두 제거

DI 적용 - DAO와 DB 커넥션을 만드는 오브젝트 사이에 연결횟수를 카운팅하는 오브젝트를 하나 더 추가
```java
/*연결횟수 카운팅 가능이 있는 클래스*/
package springbook.user.dao;
...
public class CountingConnectionMaker implements ConnectionMaker {
	int counter = 0;
	private ConnectionMaker realConnectionMaker;

	public CountingConnectionMaker(ConnectionMaker realConnectionMaker) {
		this.realConnectionMaker = realConnectionMaker;
	}
    
	public Connection makeConnection() throws ClassNotFoundException, SQLException {
		this.counter++;
        return realConnectionMaker.makeConnection();
    }
    
    public int getCounter() {
		return this.counter;
	}
}
```
`UserDao` 오브젝트가 DI 받는 대상의 설정을 조정해서 `DConnection` 오브젝트 대신 `CountingConnectionMaker` 오브젝트로 바꿔치기한다.

`CountingConnectionMaker`가 다시 실제 사용할 DB 커넥션을 제공해주는 `DConnectionMaker`를 호출하도록 만든다.
```java
/*CountingConnectionMaker 의존관계가 추가된 DI 설정용 클래스*/
package springbook.user.dao;
...
@Configuration
public class CountingDaoFactory {
	@Bean
	public UserDao userDao() {
	return new UserDao(connectionMaker());
    
    // 모든 DAO는 여전히 connectionMaker()에서 만들어지는 오브젝트를 DI 받는다.
	@Bean
	public ConnectionMaker connectionMaker() {
		return new CountingConnectionMaker(realConnectionMaker());
	}
    
	@Bean
	public ConnectionMaker realConnectionMaker() {
		return new DConnectionMaker();
    }
}
```
설정용 클래스를 `CountingDaoFactory`로 변경한 실행 코드를 만든다.
```java
/*CountingConnectionMaker에 대한 테스트 클래스*/
package springbook.user.dao;
public class UserDaoConnectionCountingTest {
	public static void main(String[] args) throws ClassNotFoundException, SQLException {
		AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(CountingDaoFactory.class);
		UserDao dao = context.getBeanC'userDao", UserDao.class);
        
        //
        // DAO 사용 코드
        //
		CountingConnectionMaker ccm = context.getBean("connectionMaker", CountingConnectionMaker.class);
        // DL(의존관계 검색)을 사용하면 이름을 이용해 어떤 빈이든 가져올 수 있다.
		System.out.println("Connection counter : " + ccm.getCounter());
	}
}

```
다시 `CountingDaoFactory` 설정 클래스를 `DaoFactory`로 변경하거나 `connectionMaker()` 메소드를 수정하는 것만으로 DAO의 런타임 의존관계는 이전 상태로 복구된다.
### 메소드를 이용한 의존관계 주입
의존관계 주입 시 생성자가 아닌 일반 메소드를 이용하는 경우가 많다.

수정자(setter) 메소드를 이용한 주입: 외부에서 오브젝트 내부의 애트리뷰트 값을 변경하려는 용도로 사용

일반 메소드를 이용한 주입: 여러 개의 파라미터를 갖는 일반 메소드를 DI용으로 사용할 수 있다.
```java
/*수정자 메소드 DI 방식을 사용한 UserDao*/
public class UserDao {
	private ConnectionMaker connectionMaker;
    
    // 수정자 메소드 DI의 전형적인 코드다. 잘 기억해두자.
	public void setConnectionMaker(ConnectionMaker connectionMaker) {
		this.connectionMaker = ConnectionMaker;
    }
    // setter 메소드는 보통 IDE의 자동생성 기능을 사용해서 만드는 것이 편리하다.
	...
}
```
## XML을 이용한 설정
스프링은 DaoFactory와 같은 자바 클래스를 이용하는 것 외에도, 다양한 방법을 통해 DI 의존관계 설정정보를 만들 수 있다.
### XML 설정
하나의 `@Bean` 메소드를 통해 얻을 수 있는 빈의 DI 정보
- 빈의 이름: `@Bean` 메소드 이름이 빈의 이름이다. 이 이름은 `getBean()`에서 사용된다.
- 빈의 클래스: 빈 오브젝트를 어떤 클래스를 이용해서 만들지를 정의한다.
- 빈의 의존 오브젝트: 빈의 생성자나 수정자 메소드를 통해 의존 오브젝트를 넣어준다.

XML 파일은 `<beans>`를 루트 엘리먼트로 사용

`<beans>` 안에는 여러 개의 `<bean>`을 정의할 수 있다.
#### connectionMaker() 전환
||자바 코드 설정정보|XML 설정정보|
|-|-|-|
|빈 설정파일|`@Configuration`|`<beans>`|
|빈의 이름|`@Bean methodName()`|`<bean id="methodName"`|
|빈의 클래스|`return new BeanClass();`|`class="a.b.c... BeanClass">`|

`<bean>` 태그의 `class` 애트리뷰트는 메소드의 리턴 타입이 아닌 자바 메소드에서 오브젝트를 만들 때 사용하는 클래스 이름
```java
@Bean									// <bean
public ConnectionMaker
connectionMaker() {						// id="connectionMaker"
	return new DConnectionMaker();		// class="springbook...DConnectionMaker" />
}
```
#### userDao() 전환
```java
userDao.setConnectionMaker(connectionMaker());
```
`<property>` 태그를 사용해 의존 오브젝트와의 관계를 정의
```xml
<!-- userDao 빈 설정 -->
<bean id="userDao" class="springbook.dao.UserDao">
	<property name="connectionMaker" ref="connectionMaker" />
</bean>
```
#### XML의 의존관계 주입 정보
`name`: DI에 사용할 수정자 메소드의 프로퍼티 이름
`ref`: 주입할 오브젝트를 정의한 빈의 ID
```xml
<!-- 완성된 XML 설정정보 -->
<beans>
	<bean id="connectionMaker" class-'springbook.user.dao.DConnectionMaker" />
	<bean id="userDao" class="springbook.user.dao.UserDao">
		<property name="connectionMaker" ref="connectionMaker" />
	</bean>
</beans>
```
같은 인터페이스를 구현한 의존 오브젝트를 여러 개 정의해두고 그중에서 원하는 걸 골라서 DI 하는 경우도 있다.

각 빈의 이름을 독립적으로 만들어두고 ref 애트리뷰트를 이용해 DI 받을 빈을 지정한다.
```xml
<!-- 같은 인터페이스 타입의 빈을 여러 개 정의한 경우 -->
<beans>
	<bean id="localDBConnectionMaker" class="...LocalDBConnectionMaker" />
	<bean id="testDBConnectionMaker" class="...TestDBConnectionMaker" />
	<bean id="productionDBConnectionMaker" class="...ProductionDBConnectionMaker" />
  
	<bean id="userDao" class="springbook.user.dao.UserDao">
		<property name="connectionMaker" ref="localDBConnectionMaker" />
	</bean>
</beans>
```
`<beans>` 태그를 기본 네임스페이스로 하는 스키마 선언은 다음과 같다.
```xml
<beans xmlns="http://www.springframework.org/schema/beans"
 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
 xsi:schemaLocation="http://www.springframework.org/schema/beans
 		http://www.springframework.org/schema/beans/spring-beans-3.0.xsd”>
```
### XML을 이용하는 애플리케이션 컨텍스트
애플리케이션 컨텍스트가 `DaoFactory` 대신 XML 설정정보를 활용하도록 변경

XML에서 빈의 의존관계 정보를 이용하는 IoC/DI 작업에는 `GenericXmlApplicationContext`를 사용

애플리케이션 컨텍스트가 사용하는 XML 설정파일의 이름은 관례를 따라 `applicationContext.xml`이라고 만든다.
```xml
<!-- XML 설정정보를 담은 applicationContext.xml -->
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
`DaoFactory`를 설정정보로 사용했을 때 썼던 `AnnotationConfigApplicationContext` 대신 `GenericXmlApplicationContext`를 이용해 애플리케이션 컨텍스트를 생성하게 만든다.
```java
ApplicationContext context = new GenericXmlApplicationContext("applicationContext.xml");
```
`ClassPathXmlApplicationContext`를 이용하여 XML 파일과 같은 클래스패스에 있는 클래스 오브젝트를 넘겨서 클래스패스를 상대적으로 지정할 수도 있다.
```java
new ClassPathXmlApplicationContext("daoContext.xml", UserDao.class);
```
### DataSource 인터페이스로 변환
#### DataSource 인터페이스 적용
`DataSource`: 자바에서 DB 커넥션을 가져오는 오브젝트의 기능을 추상화해서 비슷한 용도로 사용할 수 있게 만들어진 인터페이스

`getConnection()` 메소드를 사용해 DB 커넥션을 가져올 수 있음
```java
/*DataSource 인터페이스*/
package javax.sql

public interface DataSource extends CommonDataSource,Wrapper {
	Connection getConnection() throws SQLException;
    ...
}
```
`DataSource` 인터페이스와 다양한 `DataSource` 구현 클래스를 사용할 수 있도록 `UserDao`를 리팩토링

`getConnection()`은 SQLException만 던진다.
```java
/*DataSource를 사용하는 UserDao*/
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
#### 자바 코드 설정 방식
`SimpleConnectionMaker`: 테스트환경에서 간단히 사용할 수 있는, 스프링이 제공해주는 `DataSource` 구현 클래스

`DataFactory`의 `dataSource()` 메소드로 `SimpleDriverDataSource`를 사용하게 한다.
```java
/*DataSource 타입의 dataSource 빈 정의 메소드*/
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
`UserDao`가 `DataSource` 타입의 `dataSource()`를 DI 받게 한다.
```java
/*DataSource 타입의 빈을 DI 받는 userDao() 빈 정의 메소드*/
©Bean
public UserDao userDao() {
	UserDao userDao = new UserDaoO;
	userDao.setDataSource(dataSource());
	return userDao;
}
```
`UserDaoTest`를 `DaoFactory`를 사용하도록 수정하고 테스트
#### XML 설정 방식
id가 `ConnectionMaker`인 `<bean>`을 없애고 `dataSource`라는 이름의 `<bean>`을
등록

클래스를 `SimpleDriverDataSource`로 변경
```xml
<bean id="dataSource" class="org.springframework.jdbc.datasource.SimpleDriverDataSource" />
```
문제점
- `dataSource()` 메소드에서 `SimpleDriverDataSource` 오브젝트의 수정자로 넣어준 DB 접속정보가 나타나 있지 않음
### 프로퍼티 값의 주입
#### 값 주입
값을 주입한다: 텍스트나 단순 오브젝트 등을 수정자 메소드에 넣어준다.
```java
/*코드를 통한 DB 연결정보 주입*/
dataSource.setDriverClass(com.mysql.jdbc.Driver.class);
dataSource.setUrl("jdbc:mysql://localhost/springbook");
dataSource.setUsername("spring");
dataSource.setPassword("book");
```
```xml
<!-- XML을 이용한 DB 연결정보 설정 -->
<property name="driverClass" value="com.mysql.jdbc.Driver" />
<property name="url" value="jdbc:mysql://localhost/springbook" />
<property name="username" value="spring" />
<property name="password" value="book" />
```
#### value 값의 자동 변환
스프링은 프로퍼티의 값을, 수정자 메소드의 파라미터 타입을 참고로 해서 적절한 형태로 변환
```java
Class driverclass = Class.forName("com.mysql.jdbc.Driver”);
dataSource.setDriverClass(driverClass);
```
```xml
<!-- DataSource를 적용 완료한 applicationContext.xml -->
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
## 정리
- 관심사의 분리, 리팩토링
- 전략 패턴
- 개방 폐쇄 원칙
- 높은 응집도와 낮은 결합도
- 제어의 역전/IoC
- 싱글톤 레지스트리
- DI 컨테이너, 의존관계 주입/DI
- 생성자 주입과 수정자 주입
- XML 설정
