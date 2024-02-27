# Vol. 1 1장 오브젝트와 의존관계 02

## 1.5 스프링의 IoC

### 1.5.1 오브젝트 팩토리를 이용한 스프링 IoC

⁉️ 스프링의 핵심을 담당하는 건

⇒ 빈 팩토리 / 애플리케이션 컨텍스트


💻 **애플리케이션 컨텍스트와 설정 정보**

**⁉️ 빈 (bean)**

- 스프링에서 스프링이 제어권을 가지고 직접 만들고 관계를 부여하는 오브젝트

**⁉️ 스프링 빈**

- 스프링 컨테이너가 생성과 관계 설정, 사용 등을 제어해주는 제어의 역전이 적용된 오브젝트

**⁉️ 빈 팩토리 (bean factory)**

- 스프링에서 빈의 생성과 관계설정과 같은 제어를 담당하는 IoC 오브젝트
- 빈을 생성하고 관계를 설정하는 IoC의 기본 기능에 초점!

**⁉️ 애플리케이션 컨텍스트 (applicaton context)**

- 빈 팩토리보다 좀 더 확장한 것 (동일하다고 생각)
- 애플리케이션 전반에 걸쳐 모든 구성요소의 제어 작업을 담당하는 IoC 엔진에 초점!
- 별도의 정보를 참고해서 빈의 생성, 관계설정 등의 제어 작업을 총괄함


💻 **DaoFactory를 사용하는 애플리케이션 컨텍스트**

**DaoFactory를 설정정보로 만들어보자!**

| @Configuraton  | 스프링이 빈 팩토리를 위한 오브젝트 설정을 담당하는 클래스라고 인식하게 함 | DaoFactory |
| --- | --- | --- |
| @Bean | 오브젝트를 만들어주는 메소드 | userDao(), connectionMaker() |

위 2가지 어노테이션

⇒ 스프링 프레임워크의 빈 팩토리(애플리케이션 컨텍스트)가 IoC 방식의 기능을 제공할 때 사용할 **설정정보**가 됨

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
...
@Configuration		// 애플리케이션 컨텍스트 또는 빈 팩토리가 사용할 설정정보라는 표시
public class DaoFactory {
		@Bean		// 오브젝트 생성을 담당하는 IoC용 메소드라는 표시
		public UserDao userDao() {
				return new UserDao(connectionMaker());
		}

		@Bean
		public ConnectionMaker connectionMaker() {
				return new DConnectionMaker();
		}
}
```

**DaoFactory를 설정 정보로 사용하는 애플리케이션 컨텍스트를 만들어보자!**

애플리케이션 컨텍스트 == **ApplicationContext 타입의 오브젝트**

**사용 방법**

@Configuration이 붙은 자바 코드를 설정정보로 사용하려면?

AnnotaionConfigApplicationContext의 getBean()이라는 메소드를 이용해서 UserDao의 오브젝트를 가져올 수 있음

```java
public class UserDaoTest {
		public static void main(String[] args) throws ClassNotFoundException, SQLException {
				ApplicationContext context = new AnnotationConfigApplicationContext(DaoFactory.class);
				UserDao dao = context.getBean("userDao", UserDao.class);
		}
}
```

| AnnotationConfigApplicationContext | Bean으로 지정된 객체들을 가지고 있는 context 현재 어떤 Bean 객체들이 관리되고 있는지 확인할 수 있고 개수도 확인할 수 있음 |
| --- | --- |
| getBean() | ApplicationContext가 관리하는 오브젝트를 요청하는 메소드 기본적으로 Object 타입을 return하여 매번 return되는 Object를 다시 캐스팅해줘야 함 특정 Bean 객체를 호출하여 해당 객체 내부에 있는 함수 기능도 사용할 수 있음 |


📍 **추가할 라이브러리**

com.springsource.net.sf.cglib-2.2.0.jar

com.springsource.org.apache.commons.logging-1.1.1.jar

org.springiramework.asm-3.0.7.RELEASE.jar

org.springiramework,beans-3.0.7.RELEASE.jar

org.springiramework.context- 3.0.7.RELEASE.jar org.springframework.core-3.0.7.RELEASE.jar

org.springiramework.expression- 3.0.7.RELEASE.jar

### 1.5.2 애플리케이션 컨텍스트의 동작방식

**기존에 오브젝트 팩토리를 이용했던 방식 VS 스프링의 애플리케이션 컨텍스트를 사용한 방식**

| 오브젝트 팩토리를 이용했던 방식 | 스프링의 애플리케이션 컨텍스트를 사용한 방식 |
| --- | --- |
| 오브젝트 팩토리 | 스프링의 애플리케이션 컨텍스트 |
| / | 애플리케이션에서 IoC를 적용해서 관리할 모든 오브젝트에 대한 생성과 관계 설정을 담당함 |
| DAO 오브젝트를 생성하고 DB 생성 오브젝트와 관계를 맺어주는 제한적인 역할 | 직접 오브젝트를 생성하고 관계를 맺어주는 코드 X |

**애플리케이션 컨텍스트가 사용되는 방식**
![IMG_7427](https://github.com/leeseobin00/spring-study/assets/70849467/dd52dd7a-aef0-432d-9fdf-111bbf073386)

- 애플리케이션 컨텍스트는 DaoFactory 클래스를 설정정보로 등록
- @Bean이 붙은 메소드의 이름을 가져와 빈 목록을 만듬
- 클라이언트가 애플리케이션 컨텍스트의 getBean() 메소드를 호출하면
- 빈 목록에서 요청한 이름이 있는 지 찾음
- 있다면, 빈을 생성하는 메소드를 호출해서 오브젝트를 생성시킨 후 클라이언트에 돌려줌
    
    ⇒ **범용적이고 유연한 방법으로 IoC 기능을 확장함**
    

**애플리케이션 컨텍스트를 사용했을때 얻을 수 있는 장점**

1. **클라이언트는 구체적인 팩토리 클래스를 알 필요가 없다**
    - 애플리케이션 컨텍스트를 사용하면 오브젝트 팩토리가 아무리 많아져도 이를 알아야 하거나 직접 사용할 필요가 ❌
    - 애플리케이션 컨텍스트를 이용하면 일관된 방식으로 원하는 오브젝트를 가져올 수 ⭕️
    - DaoFactory처럼 자바 코드를 작성하는 대신 XML처럼 단순한 방법을 사용해 애플리케이션 컨텍스트가 사용할 IoC 설정정보를 만들 수 ⭕️
2. **애플리케이션 컨텍스트는 종합 IoC 서비스를 제공해준다.** 
    - 애플리케이션 컨텍스트의 역할은 다른 오브젝트와의 관계 설정 + 오브젝트가 만들어지는 방식, 시점, 전략을 다르게 가져갈 수 있음
    - 자동생성, 오브젝트에 대한 후처리, 정보의 조합, 설정 방식의 다변화, 인터셉팅 등 오브젝트를 효과적으로 활용할 수 있는 다양한 기능을 제공
    - 빈이 사용할 수 있는 기반 기술 서비스나 외부 시스템과의 연동 등을 컨테이너 차원에서 제공
3. **애플리케이션 컨텍스트는 빈을 검색하는 다양한 방법을 제공한다.** 
    - 애플리케이션 컨텍스트의 getBean() 메소드의 빈의 이름을 이용해 빈을 찾아줌
    - 타입만으로 빈을 검색하거나 특별한 어노테이션 설정이 되어 있는 빈을 찾을 수도 있음

### 1.5.3 스프링 IoC의 용어 정리


🥜 **빈 bean (빈 오브젝트)**

- **스프링이 IoC 방식으로 관리하는 오브젝트**
- == 관리되는 오브젝트 (managed object)
- 스프링을 사용하는 애플리케이션에서 만들어지는 모든 오브젝트가 다 빈은 ❌
- **스프링이 직접 그 생성과 제어를 담당하는 오브젝트만 빈!!🫛**


🏭 **빈 팩토리 bean factory**

- **스프링의 IoC를 담당하는 핵심 컨테이너**
- **빈을 등록하고, 생성하고, 조회하고, 돌려주고, 그 외에 부가적인 빈을 관리하는 기능을 담당함**
- 보통은 빈 팩토리를 바로 사용하지 않고, 이를 확장한 애플리케이션 컨텍스트를 이용함
- **주로 빈의 생성과 제어의 관점**


📱 **애플리케이션 컨텍스트 application context**

- **빈 팩토리를 확장한 IoC 컨테이너**
- **빈을 등록하고 관리하는 기본적인 기능은 빈 팩토리와 동일함**
- **스프링이 제공하는 각종 부가 서비스를 추가로 제공함**
- **스프링이 제공하는 애플리케이션 지원 기능을 모두 포함**


⚙ **설정정보 / 설정 메타정보 configuration metadata**

- **애플리케이션 컨텍스트 또는 빈 팩토리가 loC를 적용하기 위해 사용하는 메타정보**
- 스프링의 설정정보는 컨테이너에 어떤 기능을 세팅하거나 조장하는 경우 + **IoC 컨테이너에 의해 관리되는 애플리케이션 오브젝트를 생성하고 구성하는 경우**


🧰 **컨테이너 container 또는 IoC 컨테이너**

- **IoC 방식으로 빈을 관리한다는 의미**
- **== 애플리케이션 컨텍스트 == 빈 팩토리**


🍃 **스프링 프레임워크**

- IoC 컨테이너, 애플리케이션 컨텍스트를 포함해서 스프링이 제공하는 모든 기능을 통틀어 말할 때 주로 사용

## 1.6 싱글톤 레지스트리와 오브젝트 스코프

스프링의 애플리케이션 컨텍스트는 기존에 직접 만들었던 오브젝트 팩토리와 차이점


📄 **오브젝트의 동일성과 동등성**

- 동일성
    - == 연산자
- 동등성
    - equals() 메소드


**⁉️ 오브젝트를 직접 생성하면** 

```java
// 리스트 1-20 직접 생성한 DaoFactory 오브젝트 출력 코드
DaoFactory factory = new DaoFactory();
UserDao dao1 = factory.userDao();
UserDao dao2 = factory.userDao();

System.out.println(dao1);
System.out.println(dao2);
```

결과값

```python
springbook.dao.UserDao@118f375 
springbook.dao.UserDao@117a8bd
```

**⁉️ 스프링 컨텍스트로부터 오브젝트를 가져오면**

```java
// 리스트 1-21 스프링 컨텍스트로부터 가져온 오브젝트 출력 코드
ApplicationContext context = new AnnotionConfigApplicationContext(DaoFactory.class);

UserDao dao3 = context.getBean("userDao", UserDao.class);
UserDao dao4 = context.getBean("userDao", UserDao.class);

System.out.println(dao3);
System.out.println(dao4);
```

결과값

```python
springbook.dao.UserDao@ee22f7
springbook.dao.UserDao@ee22f7
```

⇒ 두 오브젝트의 출력값이 같으므로, getBean()을 두 번 호출해서 가져온 오브젝트가 동일

⇒ 매번 new에 의해 새로운 UserDao가 만들어지지 ❌

### 1.6.1 싱글톤 레지스트리로서의 애플리케이션 컨텍스트

**애플리케이션 컨텍스트는** 

- 오브젝트 팩토리와 비슷한 방식으로 동작하는 IoC 컨테이너
- **싱글톤을 저장하고 관리하는 싱글톤 레지스트리 signleton registry**
- **스프링은 기본적으로 내부에서 생성하는 빈 오브젝트는 모두 싱글톤으로 만듬**


🤷🏻 **서버 애플리케이션과 싱글톤**



왜 스프링은 싱글톤으로 빈을 만들까?

- 스프링이 주로 적용되는 대상이 자바 엔터프라이즈 기술을 사용하는 서버 환경이므로!
- 서버 환경에서는 싱글톤 사용이 권장됨
    - 애플리케이션 안에 제한된 수, 대개 한 개의 오브젝트만 만들어서 사용하는 것이 싱글톤 패턴의 원칙
    - BUT 디자인 패턴에서의 싱글톤 패턴은 안티 패턴이라고도 불림


📄 **싱글톤 패턴 (Singleton Pattern)**



- 어떤 클래스를 애플리케이션 내에서 제한된 인스턴스 개수, 주로 하나만 존재하도록 강제하는 패턴
- 하나만 만들어지는 클래스의 오브젝트는 애플리케이션 내에서 전역적으로 접근이 가능함
- 단일 오브젝트만 존재해야하고, 이를 애플리케이션의 여러 곳에서 공유하는 경우에 주로 사용함


**싱글톤을 구현하는 방법**

1. 클래스 밖에서는 오브젝트를 생성하지 못하도록 생성자를 private
2. 생성된 싱글톤 오브젝트를 저장할 수 있는 자신과 같은 타입의 스태틱 필드를 정의
3. 스태틱 팩토리 메서드인 getInstance() 생성 ⇒ 최초로 호출되는 시점에서 한 번만 오브젝트가 만들어지게 함

```java
// 리스트 1-22 싱글톤 패턴을 적용한 UserDao
public class UserDao {
		private static UserDao INSTANCE;

		private 
}
```


💣 **싱글톤 패턴의 한계**



1. **private 생성자를 갖고 있기 때문에 상속할 수 없다**
    - 싱글톤 패턴은 생성자를 private으로 제한
    - 싱글톤 클래스 자신만이 자기 오브젝트를 만들도록 제한
    - private 생성자를 가진 클래스는 다른 생성자가 없다면 상속이 불가능
    - 객체지향의 장점인 상속과 이를 이용한 다형성을 적용할 수 없음
2. **싱글톤은 테스트하기가 힘들다**
    - 싱글톤은 만들어지는 방식이 제한적이기 때문에 테스트에서 사용될 때 목 오브젝트 등으로 대체하기 힘듬
3. **서버환경에서는 싱글톤이 하나만 만들어지는 것을 보장하지 못한다**
    - 서버에서 클래스 로더를 어떻게 구성하고 있느냐에 따라서 싱글톤 클래스임에도 하나 이상의 오브젝트가 만들어질 수 있음
4. **싱글톤의 사용은 전역 상태를 만들 수 있기 때문에 바람직하지 못한다.** 
    - 싱글톤의 스태틱 메소드를 이용해 싱글톤에 쉽게 접근할 수 있기 때문에 애플리케이션 어디서든지 사용될 수 있음


📄 **싱글톤 레지스트리**



- **스프링은 직접 싱글톤 형태의 오브젝트를 만들고 관리하는 기능을 제공**
- 장점
    - 스태틱 메소드와 private 생성자를 사용해야 하는 비정상적인 클래스가 아니라 **평범한 자바 클래스를 싱글톤으로 활용하게 해줌**
- 싱글톤 방식으로 사용될 애플리케이션 클래스라도 **public 생성자를 가질 수 있음**
- 싱글톤으로 사용돼야 하는 환경이 아니라면 **간단히 오브젝트를 생성해서 사용할 수 있음**
- **싱글톤 패턴과 달리 스프링이 지지하는 객체지향적인 설계방식과 원칙, 디자인 패턴 등을 적용하는데 아무런 제약이 없다**

### 1.6.2 싱글톤과 오브젝트의 상태

- 싱글톤은 멀티스레드 환경이라면 여러 스레드가 동시에 접근해서 사용할 수 있음 ⇒ 주의!! ⚠️
- **싱글톤이 멀티스레드 환경에서 서비스 형태의 오브젝트로 사용되는 경우에는 상태 정보를 내부에 갖고 있지 않은 무상태 stateless 방식으로 만들어져야 함**

**⁉️ 인스턴스 필드로 선언하면**

```java
// 리스트 1-23 인스턴스 변수를 사용하도록 수정한 UserDao
public class UserDao {
		private ConnectionMaker connectionMaker;
		private Connection c;
		private User user;

		public User get(String id) throws ClassNotFoundException, SQLException {
				this.c = connectionMaker.makeConnection();
				...
				this.user
		}
}
```

- 싱글톤으로 만들어져서 멀티 스레드 환경에서 사용하면 위에서 설명한대로 심각한 문제가 발생
- 따라서 스프링의 싱글톤 빈으로 사용되는 클래스를 만들 때는 개별적으로 바뀌는 정보는 로컬 변수로 정의하거나, 파라미터로 주고 받으면서 사용하게 해야 함

**⁉️ 인스턴스 필드로 정의 가능한 것**

- 인스턴스 변수
    - 읽기 전용 정보
    - 자신이 사용하는 다른 싱글톤 빈을 저장하려는 용도라면 인스턴스 변수를 사용해도 됨
    - 스프링이 한 번 초기화해주고 나면 이후에는 수정되지 않기 때문에 멀티스레드 환경에서 사용해도 아무런 문제가 없다.

### 1.6.3 스프링 빈의 스코프

**⁉️ 스코프**

- 스프링이 관리하는 오브젝트(빈)가 생성되고, 존재하고, 적용되는 범위
- 스프링 빈의 기본 스코프는 싱글톤 스코프

**⁉️ 싱글톤 스코프**

- 컨테이너 내에 한 개의 오브젝트만 만들어짐
- 강제로 제거하지 않는 한 스프링 컨테이너가 존재하는 동안 계속 유지됨

**⁉️ 프로토타입 스코프**

- 컨테이너에 빈을 요청할 때마다 매번 새로운 오브젝트를 만들어줌

## 1.7 의존관계 주입(DI)

### 1.7.1 제어의 역전(IoC)과 의존관계 주입


📄 **DI (Dependency Injection)**



- 의존관계 주입, 의존성 주입, 의존 오브젝트 주입
- 오브젝트 레퍼런스를 외부로부터 제공(주입)받고 이를 통해 여타 오브젝트와 다이나믹하게 의존관계가 만들어지는 것


### 1.7.2 런타임 의존관계 설정


📄 **의존관계**



두 개의 클래스(모듈)가 의존관계에 있을 때 항상 방향성을 부여함

A가 B에 의존하고 있음

<img width="333" alt="img" src="https://github.com/leeseobin00/spring-study/assets/70849467/73bd47af-5656-4e25-baa1-f3c0c861a8f7">

**⁉️ 의존하고 있다**

- B가 변하면 그것은 A에 영향을 미친다
- B는 A의 변화에 영향을 받지 않는다.


📄 **UserDao의 의존관계**



- UserDao가 ConnectionMaker에 의존하고 있는 형태
- ConnectionMaker 인터페이스가 변하면 그 영향을 UserDao가 직접적으로 받게 됨
- BUT ConnectionMaker 인터페이스를 구현한 클래스(DConnectionMaker 등)가 다른 것으로 바뀌거나 그 내부에서 사용하는 메소드에 변화가 생겨도 UserDao에 영향을 주지 ❌
    
    ⇒ 인터페이스에 대해서만 의존관계를 만들어두면 인터페이스 구현 클래스와의 관계는 느슨해지면서 변화에 영향을 덜 받는 상태가 됨
    
    ⇒ 결합도가 낮다
    

**⁉️ 의존 오브젝트**

- 프로그램이 시작되고 오브젝트가 만들어지고 나서 런타임 시에 의존관계를 맺는 대상
- 실제 사용대상인 오브젝트

**⁉️ 의존관계 주입**

1. **클래스 모델이나 코드에는 런타임 시점의 의존관계가 드러나지 않는다. 그러기 위해서는 인터페이스에만 의존하고 있어야 한다.**
2. **런타임 시점의 의존관계는 컨테이너나 팩토리 같은 제3의 존재가 결정한다.**
3. **의존관계는 사용할 오브젝트에 대한 레퍼런스를 외부에서 제공(주입)해줌으로써 만들어진다.**

- **설계 시점에는 알지 못했던 두 오브젝트의 관계를 맺도록 도와주는 제 3의 존재가 있다는 것**
- 스프링의 애플리케이션 컨텍스트, 빈 팩토리, IoC 컨테이너 등이 모두 외부에서 오브젝트 사이의 런타임 관계를 맺어주는 책임을 지닌 제 3의 존재라고 볼 수 있음


📄 **UserDao의 의존관계 주입**



```python
// 리스트 1-24 관계설정 책임 분리 전의 생성자
public UserDao() {
		connectionMaker = new DConnectionMaker();
}
```

모델링 때의 의존관계, 런타임 의존관계까지 UserDao가 결정하고 관리

```python
// 리스트 1-25 의존관계 주입을 위한 코드
public class UserDao {
		private ConnectionMaker connectionMaker;

		public UserDao(connectionMaker connectionMaker) {
				this.connectionMaker = connectionMaker;
		}
}
```

두 개의 오브젝트 간에 런타임 의존관계가 생성됨

### 1.7.3 의존관계 검색과 주입

**⁉️ 의존관계 검색**

- 자신이 필요로 하는 의존 오브젝트를 능동적으로 찾음
- 런타임 시 의존관계를 맺을 오브젝트를 결정하는 것과 오브젝트의 생성 작업은 외부 컨테이너에게 IoC로 맡김
- 이를 가져올 때는 메소드나 생성자를 통한 주입 대신 스스로 컨테이너에게 요청하는 방법을 사용함

```python
// 리스트 1-26 DaoFactory를 이용하는 생성자
public UserDao() {
		DaoFactory daoFactory = new DaoFactory();
		this.connectionMaker = daoFactory.connectionMaker();
}
```

**스프링의 IoC 컨테이너인 애플리케이션 컨텍스트의 의존관계 검색**

- getBean()이라는 메소드 제공
- 애플리케이션 컨텍스트를 사용해서 의존관계 검색 방식으로 ConnectionMaker 오브젝트를 가져오게 만들 수 있음

```python
// 리스트 1-27 의존관계 검색을 이용하는 UserDao 생성자
public UserDao() {
		AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(DaoFactory.class);
		this.connectionMaker = context.getBean("connectionMaker", ConnectionMaker.class);
}
```

**의존관계 검색 VS 의존관계 주입**

의존관계 검색

- 코드 안에 오브젝트 팩토리 클래스나 스프링 API가 나타남
- 애플리케이션 컴포넌트가 다른 오브젝트에 의존하게 되므로 바람직 ❌
- 검색하는 오브젝트는 자신이 스프링의 빈일 필요가 없다

의존 관계 주입

- 코드 훨씬 단순 & 깔끔
- 보통 의존관계 주입 방식을 사용하는 것이 나음
- UserDao와 ConnectionMaker 사이에 DI가 적용되려면 userDao도 반드시 컨테이너가 만드는 빈 오브젝트여야 함


📄 **DI 받는다**

- DI의 동작방식은 외부로부터의 주입
- 주입받는 메소드 파라미터가 이미 특정 클래스 타입으로 고정되어 있다면 DI가 일어날 수 없다.
- DI에서 말하는 주입은 다이나믹하게 구현 클래스를 결정해서 제공받을 수 있도록 인터페이스 타입의 파라미터를 통해 이뤄져야 함


### 1.7.4 의존관계 주입의 응용

**⁉️ DI 기술의 장점**

1. 코드에는 런타임 클래스에 대한 의존관계가 나타나지 않고, 
2. 인터페이스를 통해 결합도가 낮은코드를 만들므로, 다른 책임을 가진 사용 의존 관계에 있는 대상이 바뀌거나 변경되더라도 자신은 영향을 받지 않으며, 
3. 변경을 통한 다양한 확장 방법에는 자유롭다


📄 **기능 구현의 교환**

개발 환경과 운영 환경에서 DI의 설정 정보에 해당하는 DaoFactory만 다르게 만들어두면 

⇒ 나머지 코드에는 전혀 손대지 않고 개발 시와 운영 시에 각각 다른 런타임 오브젝트에 의존관계를 갖게 해줘서 문제를 해결할 수 있음


📄 **부가기능 추가**

DAO가 DB를 얼마나 많이 연결해서 사용하는지 파악하는 기능 추가

```java
// 리스트 1-30 연결 횟수 카운팅 기능이 있는 클래스
package springbook.user.dao;

public class CountingConnectionMaker implements ConnectionMaker {
		int counter = 0;
		private ConnectionMaker realConnectionMaker;

		public CountingConnectionMaker(ConnectionMaker realConnectionMaker) {
				this.realConnectionMaker = realConnectionMaker;
		}

		public Connection makeConnection() throws ClassNotFoundException, SQLException {
				this.counter++;
				return realConnectionMaker.makerConnection();
		}

		public int getCounter() {
				return this.counter;
		}
}
```


런타임 의존관계를 변경함

새로운 의존관계를 컨테이너가 사용할 설정 정보를 이용해서 만듬

```java
// 리스트 1-31 CountingConnectionMaker 의존관계가 추가된 DI 설정용 클래스
package springbook.user.dao;

@Configuration
public class CountingDaoFactory{
		@Bean
		public UserDao userDao() {
				return new UserDao(connectionMaker());
		}

		// 모든 DAO는 여전히 connectionMaker()에서 만들어지는 오브젝트 DI를 받는다.
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

### 1.7.5 메소드를 이용한 의존관계 주입

**생성자가 아닌 일반 메소드를 이용해 의존 오브젝트와 관계를 주입해주는 방법**

1. **수정자 메소드를 이용한 주입**
    - 수정자 setter 메소드는 외부에서 오브젝트 내부의 attribute 값을 변경하려는 용도로 주로 사용됨
    - 수정자 메소드의 핵심 기능은 파라미터로 전달된 값을 보통 내부의 인스턴스 변수에 저장하는 것
    - 생성자가 수정자 메소드보다 나은 점은 한번에 여러 개의 파라미터를 받을 수 있다는 점
    
    ```python
    // 리스트 1-33 수정자 메소드 DI 방식을 사용한 UserDao
    public class UserDao {
    		private ConnectionMaker connectionMaker;
    
    		public void setConnectionMaker(ConnectionMaker connectionMaker) {
    				this.connectionMaker = connectionMaker;
    				// 수정자 메소드 DI의 전형적인 코드
    				// 잘 기억해두자
    				// setter 메소드는 보통 IDE의 자동생성 기능을 사용해서 만드는 것이 편리하다
    		}
    }
    ```
    
    ```python
    // 리스트 1-34 수정자 메소드 DI 방식을 사용한 팩토리 메소드
    @Bean
    public UserDao userDao() {
    		UserDao userDao = new UserDao();
    		userDao.setConnectionMaker(connectionMaker());
    		return userDao;
    }
    ```
    
2. **일반 메소드를 이용한 주입**
    - 여러 개의 파라미터를 갖는 일반 메소드를 DI용으로 사용할 수도 있음

## 1.8 XML을 이용한 설정

### 1.8.1 XML 설정

- 스프링의 애플리케이션 컨텍스트는 XML에 담긴 DI 정보를 활용할 수 있음
- DI 정보가 담긴 XML 파일은 <beans>를 루트 엘리먼트로 사용함
- <beans>안에 여러 개의 <bean>을 정의할 수 있음

| @Configuration | `<beans>` |
| --- | --- |
| @Bean   | `<bean>` |

- 빈의 이름: @Bean 메소드 이름, getBean()에서 사용됨
- 빈의 클래스: 빈 오브젝트를 어떤 클래스를 이용해서 만들지 정의함
- 빈의 의존 오브젝트: 빈의 생성자나 수정자 메소드를 통해 의존 오브젝트를 넣어줌


➡️ **connectionMaker() 전환**

| / | 자바 코드 설정정보 | XML 설정정보 |
| --- | --- | --- |
| 빈 설정파일 | @Configuration | <beans> |
| 빈의 이름 | @Bean methodName() | <bean id = “methodName” |
| 빈의 클래스 | return new BeanClass(); | class=”a.b.c… BeanClass”> |

DaoFactory의 @Bean 메소드에 담긴 정보를 1:1로 XML의 태그와 어트리뷰트로 전환해주면 됨

```jsx
@Bean    >    <bean
public ConnectionMaker  
connctionMaker() {    >    id="connectionMaker"
		return new DConnectionMaker();    >    class="springbook...DConnectionMaker" />
}
```

➡️ **userDao() 전환**

```jsx
userDao.setConnectionMaker(connectionMaker());

> <property name="connectionMaker" ref="connectionMaker"/>
```

```jsx
<bean id="userDao" class="springbook.dao.UserDao">
		<property name="connectionMaker" ref="connectionMaker" />
</bean>
```

➡️ **XML의 의존관계 주입 정보**

**완성된 XML 설정정보**

```jsx
// 리스트 1-38 빈의 이름과 참조 ref의 변경
<beans>
		<bean id="connectionMaker" class="springbook.user.dao.DConnectionMaker" />
		<bean id="userDao" class="springbook.user.dao.UserDao">
				<property name="connectionMaker" ref="connectionMaker" />
		</bean>
</beans>
```

**같은 인터페이스를 구현한 의존 오브젝트를 여러 개 정의**

**⇒ 원하는 것을 DI** 

```jsx
// 리스트 1-39 같은 인터페이스 타입의 빈을 여러 개 정의한 경우
<beans>
		<bean id="localDBConnectionMaker" class="...LocalDBConnectionMaker" />
		<bean id="testDBConnectionMaker" class="...TestDBConnectionMaker" />
		<bean id="productionDBConnectionMaker" class="...ProductionDBConnectionMaker" />

		<bean id="userDao" class="springbook.user.dao.UserDao">
				<property name="connectionMaker" ref="localDBConnectionMaker" />
		</bean>
</beans>
```

📄 **DTD와 스키마**

- XML 문서는 미리 정해진 구조를 따라서 작성됐는지 검사할 수 있음
- XML 문서의 구조를 정의 하는 방법
    - DTD
    - 스키마 (schema) - 바람직

### 1.8.2 XML을 이용하는 애플리케이션 컨텍스트

**XML 설정정보 활용**

- XML에서 빈의 의존관계 정보를 이용하는 IoC/DI 작업에는
GenericXmlApplicationContext를 사용
- GenericXmlApplicationContext의 생성자 파라미터로 XML 파일의 클래스패스를 지정

## 1.9 정리
- 스프링이란 '어떻게 오브젝트가 설계되고, 만들어지고, 어떻게 관계를 맺고 사용되는지에 관심을 갖는 프레임워크'
- 하지만 오브젝트를 어떻게 설계하고, 분리하고, 개선하고, 어떤 의존관계를 가질지 결정하는 일은 스프링이 아니라 개발자의 역할이며 책임
- 스프링은 단지 원칙을 잘 따르는 설계를 적용하려고 할 때 필연적으로 등장하는 번거로운 작업을 편하게 할 수 있도록 도와주는 도구일 뿐임을 잊지 말자!
- 스프링을 사용한다고 좋은 객체지향 설계와 깔끔하고 유연한 코드가 저절로 만들어지는건 절대 아니라는 것을 명심해야 한다!
