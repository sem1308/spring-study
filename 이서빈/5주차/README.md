# Vol. 1 4장 예외

예외가 관련된 코드는 자주 엉망이 되거나 무성의하게 만들어지기 쉽다. 

때론 잘못된 예외 처리 코드 때문에 찾기 힘든 버그를 낳을 수도 있고, 생각지 않았던 예외 상황이 발생했을 때 상상 이상으로 난처해질 수도 있다.

## 4.1 사라진 SQLEXCEPTION

```java
// JdbcTemplate 적용 전
public void deleteAll() throws SQLException { 
    this.jdbcContext.executeSql("delete from users");
)

// JdbcTemplate 적용 후
public void deleteAll() { 
    this.jdbcTemplate.update("delete from users");
}
```

JdbcTemplate 적용 이전 - throws SQLException 존재

⬇

JdbcTemplate 적용 이후 - SQLException 없어짐

### 4.1.1 초난감 예외 처리

> **💌 초난감 예외처리의 대표**
> 
1. **예외 블랙홀**

**예외가 발생하면 catch 블록을 사용하여 잡고 넘어가 버리는 것**

예외가 발생하면 그것을 **catch 블록**을 써서 잡아내는 것까지는 좋은데 그리고 아무것도 하지 않고 별문제 없는 것처럼 **넘어가 버리는 건 정말 위험📛한 일**이다. 

그것은 **예외가 발생했는데 무시하고 계속 진행**해버리기 때문이다.

```java
} catch (SQLException e) { 
    System.out.println(e);
}
```

```java
} catch (SQLException e) { 
    e.printStackTraceO;
)
```

⇒ 최종적으로 시스템 오류나 이상한 결과의 **원인이 무엇인지 찾아내기가 매우 힘들다**

**⇒ 모든 예외는 적절하게 복구 되든지 아니면 작업을 중단시키고 운영자 또는 개발자에게 분명하게 통보돼야 한다.** 

2. **무의미하고 무책임한 throws**

**무책임한 throws Exception 사용** 

다음과 같이 메소드 선언에 throws Exception을 기계적으로 붙이는 개발자도 있다.

```java
public void method1() throws Exception {
    method2();
)
public void method2() throws Exception {
    method3();
}
public void method3() throws Exception {
    ...
}
```

예외를 흔적도 없이 먹어 치우는 예외 블랙홀보다는 조금 낫긴 하지만 이런 무책임한 throws 선언도 심각한 문제점이 있다. 

자신이 **사용하려는 메소드에 throws Exception이 선언**되어 있다면 

- **정말 무엇인가 실행 중에 예외적인 상황이 발생할 수 있다는 것인지,**
- 아니면 그냥 **습관적으로 복사해서 붙여놓은 것인지 알 수가 없다.**

결국 **이런 메소드를 사용하는 메소드에서도 역시 throws Exception을 따라서 붙이는 수밖에 없다.**

### 4.1.2 예외의 종류와 특징

> **🛑 throw를 통해 발생시킬 수 있는 예외 종류**
> 
1. Error
2. Exception과 체크 예외
3. RuntimeException과 언체크/런타임 예외

> **🚫 Error**
> 

java.lang.Error 클래스의 서브클래스들

그래서 주로 자바 VM에서 발생시키는 것이고, 애플리케이션 코드에서 잡으려고 하면 안 된다.🚫 

OutOfMemoryError나 ThreadDeath 같은 에러는 catch 블록으로 잡아봤자 아무런 대응 방법이 없기❌ 때문이다.

> **✅ Exception과 체크 예외**
> 

java.lang.Exception 클래스와 그 서브클래스로 정의되는 예외들은 에러와 달리 **개발자들이 만든 애플리케이션 코드의 작업 중에 예외상황이 발생**했을 경우에 사용된다. 

Exception 클래스는 다시 체크 예외와 언체크 예외로 구분된다.

![Untitled](https://github.com/leeseobin00/spring-study/assets/70849467/c0e99933-6f45-40d6-91bb-0d72bf59682e)

일반적으로 예외라고 하면 Exception 클래스의 서브클래스 중에서 RuntimeException을 상속하지 않은 것만을 말하는 체크 예외라고 생각해도 된다. 

사용할 메소드가 체크 예외를 던진다면 

- catch 문으로 잡든지
- 아니면 다시 throws를 정의해서 메소드 밖으로 던져야 함

그렇지 않으면 컴파일 에러가 발생한다.

> **❎ RuntimeException과 언체크/런타임 예외**
> 

java.lang.RuntimeException 클래스를 상속한 예외들은 명시적인 예외처리를 강제하지 않기 때문에 언체크 예외라고 불린다. 

런타임 예외는 주로 프로그램의 오류가 있을 때 발생하도록 의도된 것들이다. 

피할 수 있지만 개발자가 부주의해서 발생할 수 있는 경우에 발생하도록 만든 것이 런타임 예외다.

### 4.1.3 예외 처리 방법 💬

1. **예외 복구**

**예외상황을 파악하고 문제를 해결해서 정상 상태로 돌려놓는 것**

예를 들어 사용자가 요청한 파일을 읽으려고 시도했는데 해당 파일이 없다거나 다른 문제가 있어서 읽히지가 않아서 IOException이 발생했다고 생각해보자. 

이때는 사용자에게 상황을 알려주고 다른 파일을 이용하도록 안내해서 예외상황을 해결할 수 있다. 

**예외처리 코드를 강제하는 체크 예외들은 이렇게 예외를 어떤 식으로든 복구할 가능성이 있는 경우에 사용**

```java
int maxretry = MAX_RETRY; 
while(maxretry — > 0) {
    try {
        ... // 예외가 발생할 가능성이 있는 시도
        return;
    }
    catch(SomeException e) {
        // 작업 성공
        // 로그 출력. 정해진 시간만큼 대기
    } finally {
        // 리소스 반납. 정리 작업
    } 
}
throw new RetryFailedException(); // 최대 재시도 횟수를 넘기면 직접 예외 발생
```

2. **예외 처리 회피**

**예외처리를 자신이 담당하지 않고 자신을 호출한 쪽으로 던져버리는 것**

throws 문으로 선언해서 예외가 발생하면 알아서 던져지게 하거나 catch 문으로 일단 예외를 잡은 후에 로그를 남기고 다시 예외를 던지는 것이다.

```java
public void add() throws SQLException { 
  try {
    // JDBC API
  }
  catch(SQLException e) {
    // 로그 출력
	  throw e;
	} 
}
```

DAO가 SQLException을 생각 없이 던져버리면 어떻게 될까? 

DAO에서 던진 SQLException을 서비스 계층 메소드가 다시 던지고, 컨트롤러도 다시 지도록 선언해서 예외는 그냥 서버로 전달되고 말 것이다. 

**예외를 회피하는 것은 예외를 복구하는 것처럼 의도가 분명해야 한다.**

1. **예외 전환**

예외 회피와 달리, **발생한 예외를 그대로 넘기는 게 아니라 적절한 예외로 전환해서 던진다**는 특징

예외 전환은 두 가지 목적으로 사용된다. 

1. 첫 째는 **의미를 분명하게 해줄 수 있는 예외로 변경**
    - **내부에서 발생한 예외를 그대로 던지는 것이 그 예외 상황에 대한 적절한 의미를 부여해주지 못하는 경우**에
    - **의미를 분명하게 해줄 수 있는 예외로 바꿔주기 위해서**다.

```java
public void add(User user) throws DuplicateUserldException, SQLException { 
    try {
        // ]DBC를 이용해 user 정보를 애에 추가하는 코드 또는
        // 그런 기능을 가진 다른 SQLException을 던지는 메소드를 호출하는 코드 
    }
    catch(SQLException e) {
        // ErrorCode가 MySQL의 "Duplicate Entry(1062)"이면 예외 전환 
        if (e.getErrorCode() == MysqlErrorNumbers.ER_DUP_ENTRY)
            throw DuplicateUserIdException(); 
        else
            throw e; // 그 외의 경우는 SQLException 그대로
    } 
}
```

보통 전환하는 예외에 원래 발생한 예외를 담아서 중첩 예외로 만드는 것이 좋다.

```jsx
// case 1
catch(SQLException e) {
    ...
    throw DuplicateUserldException(e);
}

// case 2
catch(SQLException e) {
    ...
    throw DuplicateUserIdException().initCause(e);
}
```

2. 두 번째 전환 방법은 **예외를 처리하기 쉽고 단순하게 만들기 위해 포장하는 것**이다. 
    - 중첩 예외를 이용해 새로운 예외를 만들고 원인이 되는 예외를 내부에 담아서 던지는 방식은 같다.
    - 예외 전환은 주로 예외처리를 제하는 체크 예외를 언체크 예외인 런타임 예외로 바꾸는 경우에 사용한다.

### 4.1.4 예외 처리 전략

일관된 예외처리 전략 정리!!

1. **런타임 예외의 보편화**

예외 종류

- 일반적으로는 체크 예외가 일반적인 예외를 다루고,
- 언체크 예외는 시스템 장애나 프로그램상의 오류에 사용한다

말 그대로 예외적인 상황이기 때문에 자바는 이를 처리하는 catch 블록이나 throws 선언을 강제하고 있다는 점이다.

2. **add() 메소드의 예외 처리**
- DuplicatedUserIdException은 충분히 복구 가능한 예외이므로 add() 메소드를 사용하는 쪽에서 잡아서 대응할 수 있다.
- 하지만 SQLException은 대부분 복구 불가능한 예외이므로 잡아봤자 처리할 것도 없고, 결국 throws를 타고 계속 앞으로 전달되다가 애플리케이션 밖으로 던져질 것이다.
- 그럴 바에는 그냥 런타임 예외로 포장해 던져버려서 그 밖의 메소드들이 신경 쓰지 않게 해주는 편이 낫다.

```jsx
public class DuplicateUserldException extends RuntimeException { 
    public DuplicateUserIdException(Throwable cause) {
        super(cause); 
    }
}
```

- DuplicatedUserIdException 외에 시스템 예외에 해당하는 SQLException은 언체크 예외가 됐다.
- 따라서 메소드 선언의 throws에 포함시킬 필요가 없다.

```jsx
public void add() throws DuplicateUserldException { 
    try {
        // ]DBC를 이용해 user 정보를 애에 추가하는 코드 또는
        // 그런 기능이 있는 다른 SQLException을 던지는 메소드를 호출하는 코드
    }
    catch (SQLException e) {
        if (e.getErrorCode() == MysqlErrorNumbers.ER_DUP_ENTRY)
            throw new DuplicateUserldException(e); // 예외 전환
        else
            throw new RuntimeException(e); // 에외 포장
    }
}
```

런타임 예외를 사용하는 경우에는 API 문서나 레퍼런스 문서 등을 통해, 메소드를 사용할 때 발생할 수 있는 예외의 종류와 원인, 활용 방법을 자세히 설명해두자.

1. **애플리케이션 예외**

시스템 또는 외부의 예외상황이 원인이 아니라 **애플리케이션 자체의 로직에 의해 의도적으로 발생**시키고, 

**반드시 catch 해서 무엇인가 조치를 취하도록 요구하는 예외**

이런 기능을 담은 메소드를 설계하는 방법

1. **정상 / 예외 상황 각각 다른 종류의 리턴 값을 돌려주는 것**
    - 경우에 따라 작업 흐름이 달라짐
    - 시스템 오류가 아니므로 기술적으로 보면 두 가지 경우 모두 정상 흐름
    - BUT 단점
        - 예외상황에 대한 리턴 값을 명확하게 코드화하고 잘 관리하지 않으면 혼란이 생길 수 있음
        - 정상적인 처리가 안 됐을 때 전달하는 값의 표준 같은 것은 없음
2. 정상적인 흐름을 따르는 코드는 그대로 두고, **예외 상황에서는 비즈니스적인 의미를 띤 예외를 던지도록 만드는 것**
    - 정상적인 흐름을 따르지만 예외가 발생할 수 있는 코드를 try 블록 안에 깔끔하게 정리해두고
    - 예외상황에 대한 처리는 catch 블록에 모아둘 수 있음

### 4.1.5 SQLException은 어떻게 됐나? ⚒

드디어 **JdbcTemplate을 적용하는 중에 throws SQLException 선언이 왜 사라졌는가?**

- 99%의 SQLException은 코드 레벨에서는 복구할 방법이 없음
- 시스템의 예외라면 애플리케이션 레벨에서 복구할 방법이 없음
- 필요도 없는 기계적인 throws 선언이 등장하도록 방치하지 말고 가능한 한 빨리 **언체크/런타임 예외로 전환해줘야 함**
- **스프링의 JdbcTemplate은 바로 이 예외처리 전략을 따르고 있음**
- **스프링의 API 메소드에 정의되어 있는 대부분의 예외는 런타임 예외**
- ⇒ **따라서 발생 가능한 예외가 있다고 하더라도 이를 처리하도록 강제하지 않음❌❌❌**

## 4.2 예외 전환

예외 전환의 목적은 두 가지라고 설명했다. 

1. 하나는 앞에서 적용해본 것처럼 **런타임 예외로 포장해서 굳이 필요하지 않은 catch/throws를 줄여주는 것**
2. 다른 하나는 **로우레벨의 예외를 좀 더 의미 있고 추상화된 예외로 바꿔서 던져주는 것**

스프링의 JdbcTemplate이 던지는 DataAccessException은 일단 런타임 예외로 SQLException을 포장해주는 역할을 한다. 

그래서 대부분 복구가 불가능한 예외인 SQLException에 대해 애플리케이션 레벨에서는 신경 쓰지 않도록 해주는 것이다. 

또한 DataAccessException은 SQLException에 담긴 다루기 힘든 상세한 예외정보를 의미 있고 일관성 있는 예외로 전환해서 추상화해주려는 용도로 쓰이기도 한다. 

### 4.2.1 JDBC의 한계 😿

**JDBC** 

- JDBC는 자바 표준 JDK에서도 가장 많이 사용되는 기능 중의 하나
- DB를 이용해 데이터를 저장하고. 필요한 정보를 조회하는 기능은 대부분의 프로그램에서 필요하기 때문
- JDBC는 자바를 이용해 DB에 접근하는 방법을 추상화된 API 형태로 정의
- 각 DB 업체가 JDBC 표준을 따라 만들어진 드라이버를 제공
- **현실적으로 DB를 자유롭게 바꾸어 사용할 수 있는 DB 프로그램을 작성하는 데는 두 가지 걸림돌이 있음**

1. **비표준 SQL**
- JDBC 코드에서 사용하는 SQL
- SQL은 어느 정도 표준화된 언어이고 몇 가지 표준 규약이 있긴 하지만
- 대부분의 DB는 표준을 따르지 않는 비표준 문법과 기능도 제공
- 비표준 특정 DB 전용 문법은 매우 폭넓게 사용
- **하지만 DB의 변경 가능성을 고려해서 유연하게 만들어야 한다면 SQL은 제법 큰 걸림돌**

해결책 🩹
- 호환 가능한 표준 SQL만 사용하는 방법과, DB별로 별도의 DAO를 만들거나 SQL을 외부에 독립시켜서 DB에 따라 변경해 사용하는 방법이 있음 ⇒ 현실성이 없음
- **DAO를 DB별로 만들어 사용하거나 SQL을 외부에서 독립시켜서 바꿔 쓸 수 있게 하는 것**

1. **호환성 없는 SQLException의 DB 에러정보**
- SQLException 때문!
- DB를 사용하다가 발생할 수 있는 예외의 원인은 다양
- DB마다 SQL만 다른 것이 아니라 에러의 종류와 원인도 제각각
- **JDBC API는 이 SQLException 한 가지만 던지도록 설계**
- **결국 호환성 없는 에러 코드와 표준을 잘 따르지 않는 상태 코드를 가진 SQLException만으로 DB에 독립적인 유연한 코드를 작성하는 건 불가능**

### 4.2.2 DB 에러 코드 매핑을 통한 전환 🛠

DB 종류가 바뀌더라도 DAO를 수정하지 않으려면 위의 두 가지 문제를 해결해야 함

SQLException의 비표준 에러 코드와 SQL 상태정보에 대한 해결책

1. **SQL 상태정보에 대한 해결책**
    - DB 에러 코드는 DB에서 직접 제공해주는 것이니 버전이 올라가더라도 어느 정도 일관성이 유지
    - DB별 에러 코드를 참고해서 발생한 예외의 원인이 무엇인지 해석해주는 기능을 만드는 것
2. **SQLException의 비표준 에러 코드**
    - JdbcTemplate을 이용한다면 JDBC에서 발생하는 DB 관련 예외는 거의 신경 쓰지 않아도 됨
    - 하지만 SQLExecption의 서브클래스이므로 여전히 체크 예외라는 점과 그 예외를 세분화하는 기준이 SQL 상태정보를 이용한다는 점에서 여전히 문제점

### 4.2.3 DAO 인터페이스와 DataAccessException 계층구조 🗂

DataAccessException

- JDBC의 SQLException을 전환하는 용도 + JDBC 외의 자바 데이터 액세스 기술에서 발생하는 예외에도 적용
- 의미가 같은 예외라면 데이터 액세스 기술의 종류와 상관없이 일관된 예외가 발생하도록 만들어 줌
- 데이터 액세스 기술에 독립적인 추상화된 예외를 제공하는 것

> **DAO 인터페이스와 구현의 분리**
> 

DAO를 굳이 따로 만들어서 사용하는 이유는 무엇일까?

1. 가장 중요한 이유는 **데이터 액세스 로직**을 담은 코드를 성격이 **다른 코드에서 분리**해놓기 위해서
2. **분리된 DAO**는 **전략 패턴을 적용**해 **구현 방법을 변경해서 사용**할 수 있게 만들기 위해서

DAO는 인터페이스를 사용해 구체적인 클래스 정보와 구현 방법을 감추고 DI를 통해 제공되도록 만드는 것이 바람직

⇒ BUT 메소드 선언에 나타나는 예외정보가 문제

⇒ 사용 불가 because DAO에서 사용하는 데이터 액세스 기술의 API가 예외를 던지기 때문

⇒ 단순한 해결 방법은 모든 예외를 다 받아주는 throws Exception으로 선언하는 것 (무책임한 선언)

⇒ JDBC를 이용한 DAO에서 모든 SQLException을 런타임 예외로 포장

⇒ 완전히 독립적인 인터페이스 선언이 가능

> **데이터 액세스 예외 추상화와 DataAccessException 계층구조**
> 

스프링은 자바의 다양한 데이터 액세스 기술을 사용할 때 발생하는 예외들을 추상화해서 DataAccessException 계층구조 안에 정리해 놓았음

스프링의 DataAccessException은 이런 일부 기술에서만 공통적으로 나타나는 예외를 포함해서 데이터 액세스 기술에서 발생 가능한 대부분의 예외를 계층구조로 분류해 놓았음

<img width="492" alt="Untitled2" src="https://github.com/leeseobin00/spring-study/assets/70849467/d92742f5-e5ed-4dfe-a3f1-bc752ee27c7b">

⇒ 스프링의 데이터 액세스 지원 기술을 이용해 DAO를 만들면 사용 기술에 독립적인 일관성 있는 예외를 던질 수 있음

### 4.2.4 기술에 독립적인 UserDao 만들기

> **인터페이스 적용**
> 

UserDao 인터페이스에는 기존 UserDao 클래스에서 DAO의 기능을 사용하려는 클라이언트들이 필요한 것만 추출해내면 됨.

```java
public interface UserDao {
		void add(User user);
		User get(String id);
		List<User> getAll();
		void deleteAll();
		int getCount();
}
```

⬇

기존 UserDao의 이름 → UserDaoJdbc로 변경하고

UserDao 인터페이스를 구현하도록 implements로 선언

```java
public class UserDaoJdbc implements UserDao {
```

⬇

스프링 설정 파일의 userDao 빈 클래스 이름을 변경

- 빈의 이름은 인터페이스 이름을 따르는 경우가 일반적?
    
    ㅇㅇ 보통 빈의 이름은 클래스 이름이 아니라, dataSource 빈처럼 클래스의 구현 인터페이스 이름을 따르는 경우가 일반적
    
    ⇒ 나중에 구현 클래스를 바꿔도 혼란이 없기 때문
    

> **테스트 보완**
> 

UserDaoTest 테스트 코드에서 UserDao 인스턴스 변수 선언도 UserDaoJdbc로 변경해야할까?

⮕ 아니다! 

**@Autowired 사용하여 스프링의 컨텍스트 내에서 정의된 빈 중에서 인스턴스 변수에 주입 가능한 타입의 빈을 찾아줌**

> **DataAccessException 활용 시 주의사항**
> 

DuplicateKeyException - JDBC를 이용하는 경우에만 발생

데이터 엑세스 기술에 따라 다른 예외가 던져짐

JDBC - SQLException에 담긴 DB의 에러 코드를 바로 해석함

JPA, 하이버네이트, JDO 등 - 각 기술이 재정의한 예외들을 가져와 스프링이 최종적으로 DataAccessException으로 변환함

⇒ **DB의 에러 코드와 달리 예외들이 세분화 되어있지 ❌**

## 4.3 정리

- 예외를 잡아서 아무런 조취를 취하지 않거나 **의미 없는 throws 선언을 남발하는 것은 위험**하다.
- **예외는 복구**하거나 예외처리 오브젝트로 의도적으로 전달하거나 **적절한 예외로 전환**해야 한다.
- **좀 더 의미 있는 예외로 변경**하거나, 불필요한 catch/throws를 피하기 위해 **런타임 예외로 포장**하는 두 가지 방법의 예외 전환이 있다.
- **복구할 수 없는 예외는 가능한 한 빨리 런타임 예외로 전환하는 것이 바람직**하다.
- **애플리케이션의 로직을 담기 위한 예외는 체크 예외로** 만든다.
- JDBC의 **SQLException은 대부분 복구할 수 없는 예외이므로 런타임 예외로 포장**해야 한다.
- SQLException의 에러 코드는 DB에 종속되기 때문에 DB에 독립적인 예외로 전환될 필요가 있다.
- 스프링은 **DataAccessException을** 통해 DB에 **독립적으로 적용 가능한 추상화된 런타임 예외 계층을 제공**한다.
- **DAO를 데이터 엑세스 기술에서 독립시키려면 인터페이스 도입과 런타임 예외 전환, 기술에 독립적인 추상화된 예외로 전환이 필요**하다.
