# [토비 스프링] 3장 템플릿

스프링에 적용된 템플릿 기법을 살펴보고, 이를 적용해 완성도 있는 DAO 코드를 만드는 방법을 알아보자!

# 3.1 다시 보는 초난감 DAO

## **3.1.1** 예외처리 기능을 갖춘 **DAO**

정상적인 JDBC 코드의 흐름을 따르지 않고 중간에 어떤 이유로든 예외가 발생했을 경우에도 사용한 리소스를 반환하도록 하기위해서 **예외처리**를 갖춰야 한다!

**JDBC APl를 이용한 DAO 코드 deleteAll()**

```java
public void deleteAll() throws ClassNotFoundException, SQLException {
	Connection c = dataSource.getConnection();

	PreparedStatement ps = c.prepareStatement("delete from users");
	ps.executeUpdate(); //여기서 예외가 발생하면 메소드 실행이 중단된다.
	
  //예외가 발생하면 close() 메서드들이 실행되지 못한다.
  //그렇게 될떄마다 제대로 ps와 c의 리소스가 반환되지 않게 되고, 
  //커넥션 풀에 여유가 생기지 않게 되어 리소스가 모자라게 된다.
	ps.close();
	c.close();
}
```

- Connection, PreparedStatement의 반환이 확실하지 않다.
- 반환되지 못한 리소스들이 쌓여 어느 순간 **커넥션 풀이 모자라 서버가 중단될 수 있는 위험도**가 있다.

**예외 발생 시에도 리소스를 반환하도록 수정한 deleteAII()**

```java
public void deleteAll() throws SQLException { 
	Connection c = null;
	PreparedStatement ps = null; 
	try {
		c = dataSource.getConnection(); 
		ps = c.prepareStatement("delete from users"); 
		ps.executeUpdate();
		//예외가 발생할 기능성이 있는 코드를 모두 try block 으로 묶어준다.
	} catch (SQLException e) { 
			throw e;
	} finally { //언제나 실행
			if (ps != null) {

			 //ps.close() 메소드에서도 SQLException이 발생할 수 있기 때문에 try/catch로 묶는다
			 try {
					ps.close();
			 } catch (SQLException e) {}	
			                    
			if (c != null) {
				try {
					c.close(); // Connection 빈환 
				} catch (SQLException e) {}
			}
	}
}
```

- 어느 시점에서 예외가 발생했는 지에 따라 close()를 사용할 수 있는 변수가 달라질 수 있기에 finally에서는 반드시 **c와 ps가 null이 아닌지 먼저 확인한 후에 close() 메소드를 호출**해야 한다.
- close() 메소드에서도 예외가 발생할 수 있기에 try/catch로 묶는다. 이렇게 해야 ps.close()에서 예외가 터져도 c.close()를 호출할 수 있다.

**JDBC 예외처리를 적용한 getCount() 메소드**

```java
public int getCount() throws SQLException ( 
	Connection c = null; 
	PreparedStatement ps = null; 
	ResultSet rs = null; 

	try { 
		c = dataSource.getConnection(); 
		ps = c.prepareStatement("select count(*) from users"); 
		
		//ResultSet도 다양한 SQLException이 발생할 수 있는 코드이므로 try 안에 둬야 한다.
		rs.ps.executeQuery();	
		rs.next();

		return rs.getlnt(1);
	} catch (SQLException e) {
			throw e; 
	} finally { 
		//참고로 close는 만들어진 순서의 반대로 하는 것이 원칙이다!
		if (rs != null) { 
			try {
				rs.close();
			} catch (SQLException e) {}
		}

		if (ps != null) { 
			try {
				ps.close(); 
			} catch (SQLException e) {}

		if (c != null) {
			try {
				c.close();
			}
		}
	}
}
```

- 조회를 위한 JDBC 코드의 경우 ResultSet이 추가되고 마찬가지로 이를 반환해야한다.

# **3.2** 변하는 것과 변하지 않는 것

## **3.2.1 JDBC try/catch/finally** 코드의 문제점

1. 복잡한 try/catch/finally 블록이 모든 메소드에서 반복된다.
2. 실수하여 close()와 같은 메소드를 빼먹는 경우 언젠가 커넥션 풀이 모자라 서버가 중단될 수 있다.
3. 처음에 완벽하게 작성했더라도 추후에 잘못 수정하여 폭탄이 될 수 있는 위험이 있다.

## **3.2.2** 분리와 재사용을 위한 디자인 패턴 적용

**개선할 deleteAll() 메소드**

```java
public void deleteAll() throws SQLException {
	Connection c = null;
	PreparedStatement ps = null; 
	try { 
		c = dataSource.getConnection(); 
	
	  //변하는 부분
		ps = c.prepareStatement("delete from users");

	  ps.executeUpdate(); 
	} catch (SQLException e) { 
		throw e; 
	} finally { 
		if (ps != null) { try { ps.close(); } catch (SQLException e) {} } 
		if (c != null) { try { c.close(); } catch (SQLException e) {} }
	}
}
```

- Connect 객체를 사용하여 새로운 PreparedStatement 객체를 생성하는 부분의 코드만 변하는 부분이다.

**개선할 add() 메소드**

```java
...
ps = c.prepareStatement("insert into users(id, name, password) values(?,?,?)")；
ps.setString(1, user.getld())；
ps.setString(2, user.getName())；
ps.setString(3, user.getPassword())；
...
```

- add() 메소드에서는 위 코드가 변하는 부분이 된다.

### 메소드 추출

변하는 부분을 메소드로 빼보자!

**변하는 부분을 메소드로 추출한 후의 deleteAll()**

```java
public void deleteAll() throws SQLException {
	c = dataSource.getConnection()；

	try {
		ps = makeStatement(c)； //변하는 부분 메소드 추출
		ps.executeUpdate()；
	} catch (SQLException e) {...}
	
}

private PreparedStatement makeStatement(Connection c) throws SQLException {
	PreparedStatement ps；
	ps = c.prepareStatement("delete from users")；
	return ps；
}
```

- 분리시키고 남은 메소드 **deleteAll()이 재사용 가능**하고 분리시킨 **makeStatement() 메소드는 재사용이 불 가능**하다. 반대로 되었다..

### 템플릿 메소드 패턴의 적용

템플릿 메소드 패턴은 **상속을 통해 기능을 확장**한다.

**변하지 않는 부분은 슈퍼클래스**에 두고 **변하는 부분은 추상 메소드**로 정의해둬서 서브클래스에서 오버라이드하여 새롭게 정의해 쓰도록 하는 것이다.

**UserDao 클래스를 추상 클래스로 선언**

```java
public abstract class UserDao {
	...

	//변하는 부분을 추상 메소드로 정의
	abstract protected PreparedStatement makeStatement(Connection c) throws SQLException；
	
	...
}
```

- UserDao 클래스를 추상 클래스로 선언하고 변하는 부분을 추상 메소드로 정의한다.

**makeStatement()를 구현한 UserDao 서브클래스**

```java
public class UserDaoDeleteAll extends UserDao {

	protected PreparedStatement makeStatement(Connection c) throws SQLException {
		PreparedStatement ps = c.prepareStatement("delete from users")；
		return ps；
	}
}
```

- DAO 로직마다 새로운 클래스를 작성해야하는 문제가 있다.
- 또한 확장구조가 클래스를 설계하는 시점에서 고정이 되므로 유연성이 떨어진다.

### 전략 패턴의 적용

개방 폐쇄 원칙을 잘 지키면서 템플릿 메소드 패턴보다 유연하고 확장성이 뛰어난 **전략 패턴**을 적용해보자!

**전략 패턴**

![Untitled](imgs/1.png)

- 오브젝트를 아예 둘로 분리하여 클래스 레벨에서는 **인터페이스를 통해서만 의존하도록** 한다.
- **확장**에 해당하는 **변하는 부분을 별도의 클래스**로 만든다.
- Context의 contextMethod() 에서 일정한 구조를 가지고 동작하다가 특정 확장 기능은 Strategy  인터페이스를 통해 외부의 독립된 전략 클래스에 위임한다.
- 변하지 않는 부분이 contextMethod() 이고 바뀌는 부분이 전략!

**StatementStrategy 인터페이스**

```java
public interface Statementstrategy {
	PreparedStatement makePreparedStatement(Connection c) throws SQLException；
}
```

- 위 인터페이스를 상속해서 바뀌는 부분 **makePreparedStatement()**을 **오버라이딩**하면 된다.

**전략 패턴을 적용해 deleteAII() 메소드를 구현한 DeleteAllStatement**

```java
public class DeleteAllStatement implements StatementStrategy {
	public PreparedStatement makePreparedStatement(Connection c) throws SQLException { 
		PreparedStatement ps = c.prepareStatement("delete from users");
		return ps;
  }
}
```

**DeleteAIIStatement가 적용된 deleteAll() 메소드**

```java
public void deleteAll() throws SQLException {
	try {
		c = dataSource.getConnection()；

		Statementstrategy strategy = new DeleteAllStatement()；
		ps = strategy.makePreparedStatement(c)；

		ps.executeUpdateO；
	} catch (SQLException e) {
	}
}
```

- 문제점이 있다. 전략 패턴은 필요에 따라 전략을 바꿔쓸 수 있는데 위 코드는 전략이 고정되어 있다.
- OCP에도 맞지 않는다.

### DI 적용을 위한 클라이언트 컨텍스트 분리

![Untitled](imgs/2.png)

- 전략 패턴에 따르면 Context가 어떤 전략을 사용하게 할 것인가는 Context를 사용하는 Client가 결정하는 게 일반적이다.
- Client ⇒ UserDao
- Context ⇒ deleteAll()의 내부
- Strategy ⇒ StatementStrategy

**메소드로 분리한 try/catch/finally 컨텍스트 코드**

```java
public void jdbcContextWithStatementStrategy(Statementstrategy stmt) throws SQLException {
 Connection c = null；
 PreparedStatement ps = null；

	try {
		c = dataSource.getConnection();
		ps = stmt.makePreparedStatement(c)；
		ps.executeUpdateO；
	} catch (SQLException e) {
		throw e；
	} finally {
		if (ps != null) { try { ps.closeO； } catch (SQLException e) {} }
		if (c != null) { try {c.closeO； } catch (SQLException e) {} }
	}
}
```

- 클라이언트로부터 Statementstrategy 타입의 전략 오브젝트를 제공받고 JDBC try/catch/finally 구조로 만들어진 컨텍스트 내에서 작업을 수행한다.

**클라이언트 책임을 담당할 deleteAll() 메소드**

```java
public void deleteAll() throws SQLException { //client
	StatementStrategy st = new DeleteAllStatement()； //strategy
	jdbcContextWithStatementStrategy(st)； //context
}
```

- 클라이언트와 컨텍스트를 분리하진 않았지만 의존관계와 책임으로 볼 때 이상적인 클라이언트 컨텍스트 관계를 갖는다.
- 이 구조를 기반으로 개선해보자!

# **3.3 JDBC** 전략 패턴의 최적화

## **3.3.1** 전략클래스의 추가 정보

add() 메소드에도 적용해보자!

**add() 메소드의 PreparedStatement 생성 로직을 분리한 클래스**

```java
public class AddStatement implements StatementStrategy { 
	public PreparedStatement makePreparedStatement(Connection c) throws SQLException { 
	 PreparedStatement ps = c.prepareStatement("insert into users(id, name , password)"+
																						" values(?, ?, ?)"); 
 	 ps.setString(1, user.getId());
	 ps.setString(2, user.getName()); 
	 ps.setString(3, user.getPassword()); 
	
	 return ps;
	}
}
```

- add()에서는 PreparedStatement를 만들 때 user라는 부가적인 정보가 필요하기 때문에 오류가 발생한다.
- user는 클라이언트인 add() 메소드가 갖고 있기에 클라이언트로부터 user를 제공받아야 한다.

**User 정보를 생성자로부터 제공받도록 만든 AddStatement**

```java
public class AddStatement implements Statementstrategy {
	User user；

	public AddStatement(User user) {
		this.user = user；
	}
	...
}
```

**User 정보를 AddStatement에 넘겨주는 add()**

```java
public void add(User user) throws SQLException {
	StatementStrategy st = new AddStatement(user)；
	jdbcContextWithStatementStrategy(st)；
}
```

## **3.3.2** 전략과 클라이언트의 동거

**문제점**

- DAO 메소드마다 새로운 StatementStrategy 구현 클래스를 만들어야 한다.
- Statementstrategy에 전달받은 부가 정보가 있을 경우 이를 위해 해당 정보를 받을 생성자와 인스턴스 변수를 만들어야 한다.

### 익명 내부 클래스

**익명 내부 클래스 선언 방법**

- new 인터페이스 이름() { 클래스 본문 };

**AddStatement를 익명 내부 클래스로 전환**

```java
Statementstrategy st = new StatementStrategy() {
	public PreparedStatement makePreparedStatement(Connection c) throws SQLException {
		PreparedStatement ps = c.prepareStatement("insert into users(id, name, password) values(?,?,?)")；
		ps.setString(1, user.getld())；
		ps.setString(2z user.getName())；
		ps.setString(3z user.getPassword())；

		return ps；
	}
}；
```

- 선언과 동시에 오브젝트를 생성한다.

**익명 내부 클래스를 적용한 add() 메소드**

```java
public void add(final User user) throws SQLException {
	jdbcContextWithStatementStrategy(new Statementstrategy() {
			public PreparedStatement makePreparedStatement(Connection c) throws SQLException {
				PreparedStatement ps = c.prepareStatement("insert into users(id, name,password) values(?,?,?)")；
				ps.setStringd, user.getldO)；
				ps.setString(2, user.getNameO)；
				ps.setString(3z user.getPasswordO)；
				return ps；
			}
		}
	)；
}
```

# **3.4** 컨텍스트와 **Dl**

## **3.4.1 JdbcContext**의 분리

JDBC의 일반적인 작업 흐름을 담고 있는 jdbcContextWithStatementStrategy()는 다른 DAO에서도 사용 가능하므로 UserDao 클래스 밖으로 독립시켜서 모든 DAO가 사용할 수 있게 해보자!

### 클래스 분리

**JdbcContext 클래스**

```java
public class JdbcContext {
	DataSource dataSource;

	public void setDataSource(DataSource dataSource) {
		this.dataSource = dataSource;
	}

	public void workWithStatementStrategy(StatementStrategy stmt) throws SQLException {
		Connection c = null;
		PreparedStatement ps = null;

		try {
			c = dataSource.getConnection();
	
			// 전략패턴 적용
			ps = stmt.makePreparedStatement(c);
      ps.executeUpdate();
		} catch (SQLException e) {
			throw e;
		} finally {
			if (ps != null) { try { ps.close(); } catch (SQLException e) {} }
			if (c != null) { try { c.close(); } catch (SQLException e) {} }
		}
	}

	public void add(User user) throws SQLException {
	   StatementStrategy st = new AddStatement(user);
	   workWithStatementStrategy(st); // 이렇게 주입 
	}
}

```

- jdbcContextWithStatementStrategy()를 JdbcContext 클래스의 workWithStatementStrategy()라는 이름으로 옮겼다.
- DB 커넥션을 필요로 하는 코드가 JdbcContext 안에 있기 때문에 JdbcContext에서 DataSource 타입의 빈을 DI 받는다.

**UserDao에서 DataSource를 DI받는 XML**

```java
<beans>
	<bean id="userDao" class="springbook.user.dao.UserDao">
		<property name-'dataSource" ref="dataSource" />
	</bean>

	〈bean id="dataSource" class="org.springframework.jdbc.datasource.SimpleDriverDataSource" >
	</bean>
</beans>
```

- JdbcContext는 빈이 아니기 때문에 UserDao에서 DataSource를 주입받아 JdbcContext에 넘겨주는 것이다.

**JdbcContext에 DataSource를 DI**

```java
public class UserDao {
	private JdbcContext dbcContext；

	public void setDataSource(DataSource dataSource) {
		this.jdbcContext = new JdbcContext()；
		this.jdbcContext.setDataSource(dataSource)；
	}
	...
}
```

- 수동 DI하여 주입한다.

# **3.5** 템플릿과 콜백

**템플릿 콜백 패턴**

- 전략 패턴의 기본 구조에 **익명 내부 클래스**를 활용한 방식
    - **템플릿** ⇒ 고정된 작업 흐름을 가진 코드를 재활용 (전략 패턴의 Context)
    - **콜백** ⇒ 템플릿 안에서 호출되는 것을  목적으로 만들어진 오브젝트(익명 내부 클래스)

## **3.5.1** 템플릿/콜백의 동작원리

### 템플릿 콜백 특징

- 템플릿의 작업 흐름 중 특정 기능을 위해 한 번 호출되는 경우가 일반적이기 때문에 **단일 메소드 인터페이스**를 사용한다. ⇒ 그 인터페이스를 익명 내부 클래스로 구현한 것이 콜백!

### JdbcContext에 적용된 템플릿/콜백

![Untitled](imgs/3.png)

## **3.5.2** 편리한 콜백의 재활용

### 콜백의 분리와 재활용

**현재 deleteAll()**

```java
public void deleteAll() throws SQLException {
	this.jdbcContext.workWithStatementStrategy(
		new Statementstrategy() {
			public PreparedStatement makePreparedStatement(Connection c) throws SQLException {
				return c.prepareStatement("delete from users")； //변하는 SQL 문장
			} 
		}
	); 
}
```

- deleteAll()에서는 SQL만 변할 가능성이 있다.
- 이처럼 바인딩할 파라미터 없이 미리 만들어진 SQL을 이용해 PreparedStatement를 만들기만 하면 되는 콜백이 적지 않을 것이기에 이를 분리시켜 중복되는 코드의 반복을 막아보자!

**변하지 않는 부분을 분리시킨 deleteAll() 메소드**

```java
public void deleteAll() throws SQLException {
	executeSql("delete from users")；
}
--------------------------------------------------------------------------------
private void executeSql(final String query) throws SQLException {
	this.jdbcContext.workWithStatementStrategy(
		new Statementstrategy() {
			public PreparedStatement makePreparedStatement(Connection c) throws SQLException {
				return c.prepareStatement(query)； //변하는 SQL 문장
			} 
		}
	); 
}
```

- 이제 고정된 SQL만을 사용하는 메소드는 executeSql()를 사용하면 된다.

### 콜백과 템플릿의 결합

executeSql() 메소드를 UserDao에서만 사용하는 것은 아쉬우니 다른 곳에서도 사용할 수 있도록 JdbcContext 안으로 옮기자!

**옮긴 후 JdbcContext와 UserDao.deleteAll()**

```java
public class JdbcContext {
	...
	public void executeSql(final String query) throws SQLException {
		workWithStatementStrategy(
		new StatementStrategy() {
			public PreparedStatement makePreparedStatement(Connection c) throws SQLException {
				return c.prepareStatement(query)；
			}
		}
	)；
}

--------------------------------------------------------------------------------

public void deleteAll() throws SQLException {
	this.jdbcContext.executeSql("delete from users")；
}
```

- 이제 모든 DAO 메소드에서 executeSql() 메소드를 사용할 수 있다!

## **3.5.3** 템플릿/콜백의 응용

```java
// 콜백 인터페이스 정의
public interface LineCallback<T> {
  T doSomethingWithLine(String line, T value);
}

------------------------------------------------------------------------------------

// 컨텍스트와 콜백을 정의하는 클래스 정의
public class LineTemplate {

	//템플릿
	public <T> T lineReadTemplate(String filepath, LineCallback<T> callback, T initVal) 
		throws IOException { 
			BufferedReader br = null; 
			try {
				br = new BufferedReader(new FileReader(filepath)); 
				T res = initVal; 
				String line = null; 
				
				while ((line = br.readLine()) != null) {
				 	 res = callback.doSomethingWithLine(line, res); //변 부분. (콜백)
				}
	
				return res;
	
			} catch (IOException e) { ... } finally { ... }
	}

	// 콜백 1: 파일을 하나 열어서 문자열을 모두 하나로 잇는 메서드
	public String concatenate(String filepath) throws IOException {
			LineCallback<String> concatenateCallback = 
					new LineCallback<String>() {
						// 익명 클래스 정의 및 생성
						public String doSomethingWithLine(String line, String value) { 
								return value + line; 
						}
					};               
			return lineReadTemplate(filepath, concatenateCallback, "");
	}

	// 콜백 2: 파일을 하나 열어서 모든 라인의 숫자를 더한 합을 돌려주는 메서드
	public Integer calcSum(String filepath) throws IOException {
			LineCallback sumCallback = new LineCallback () { 
				public Integer doSomethingWithLine(String line, Integer value) {
							return value + Integer.valueOf(line); 
				}
			};	
		return lineReadTemplate(filepath, sumCallback, e);
	}
}

// 사용
public class FileModifier {
	
	// 직접 생성 대신 DI를 이용해야 한다. 간편한 예시를 위해 여기서는 직접 생성방식을 사용했다.
	private LineTemplate lineTemplate = new LineTemplate;

	public String concat() {
			return lineTemplate.concatenate("C:\text.txt");
	}
	
	public Integer sumAllLines() {
		 return lineTemplate.calcSum("C:\text.txt");
	}

}
```

# **3.6 스프링의 JdbcTemplate**

이번에는 스프링이 제공하는 **템플릿/콜백 기술**을 살펴보자!

**JdbcTemplate**

- 스프링이 제공하는 **JDBC 코드용 기본 템플릿**이다.
- 앞서 만들었던 JdbcContext와 유사하지만 훨씬 강력하고 편리한 기능을 제공한다.

**JdbcContext를 JdbcTemplate로 변경한 UserDao**

```java
public class UserDao {
	private JdbcTemplate JdbcTemplate；

	public void setDataSource(DataSource dataSource) {
		this.jdbcTemplate = new JdbcTemplate(dataSource)；
		this.dataSource = dataSource；
	}
	...
}
```

- JdbcTemplate은 생성자의 파라미터로 DataSource를 받는다.

## 3.6.1 Update()

**JdbcTemplate을 적용한 deleteAll() 메소드**

```java
public class UserDao {

  public void deleteAll() {
		this.jdbcTemplate.update(new PreparedStatementCreator() {
		public PreparedStatement createPreparedStatement(Connection con) throws SQLException {
				return con.preparedStatement("delete from users");
		}
	 }
  };
}
```

- JdbcContext.makePreparedStatement() ⇒ JdbcTemplate.createPreparedStatement()
- PreparedStatementCreator 타입의 콜백을 받아서 사용하는 JdbcTemplate의 템플릿 메소드는 update()이다.

**JdbcTemplate의 내장 콜백을 사용하는 메소드를 호출하도록 deleteAll() 메소드 수정**

```java
public class UserDao {
	public void deleteAll() {
			this.jdbcTemplate.update("delete from users");
	}
}
```

- 앞서 만들었던 JdbcContext의 executeSql() 메소드와 비슷한 메소드이다.
    - 이전 코드의 콜백을 받는 update() 메소드와 이름은 동일한데 파라미터로 SQL 문장을 전달한다는 것만 다르다.

**기존 add() 메소드의 콜백 내부**

```java
Prepared5tatement ps;
ps = c.prepare5tatement("insert into users(id, name, password) values(?, ?, ?)"); 
ps.set5tring(1 ,user.getld());
ps.set5tring(2, user.getName()); 
ps.set5tring(3, user.getPassword());
```

**위 기능을 JdbcTemplate를 사용해 간단하게 만들 수 있다.**

```java
this.jdbcTemplate.update("insert into users(id, name, password) values(?, ?, ?)", 
	user.getld(), user.getName(), user.getPassword());
```

## **3.6.2 queryForInt()**

템플릿/콜백 방식을 적용하지 않았던 메소드에 **JdbcTemplate**을 적용해보자!

**ResultSetExtractor**

- PreparedStatement의 쿼리를 실행해서 얻은 **ResultSet을 전달받는 콜백**이다.

**PreparedStatement과 ResultSetExtractor를 사용한 getCount()**

```java
// 직접 만든 PreparedStatementCreator에 JDBCTemplete 적용
public int getCount() {
	return this.jdbcTemplate.query(new PreparedStatementCreator() {
		public PreparedStatement createPreparedStatement(Connection con) throws SQLException {
			return con.preparedStatement("select count(*) from users");
		}, new ResultSetExtractor<Integer>() { // 두번째 콜백 (ResultSet으로부터 값 추출)
			public Integer extractData(ResultSet rs) throws SQLException, DataAccessException {
				rs.next();
				return rs.getInt(1);
			}
  });
}

// 람다로 변경
public int getCount() {
	 return this.jdbcTemplate.query(con ->
															con.preparedStatement("select count(*) from users"),
				(resultSet) -> {
  					resultSet.next();
	  				return resultSet.getInt(1);
				}
}
```

- **query()**는 **PreparedStatementCreator 콜백**과 **ResultSetExtractor 콜백**을 파라미터로 받는다.
- 현재 코드의 기능을 가진 **queryForInt()**를 JdbcTemplate이 제공한다.

**queryForInt()를 사용한 getCount()**

```java
public int getCount() {
		return this.jdbcTemplate.queryForInt("select count(*) from users");
}
```

- 이전 코드를 이렇게 한 줄로 변경할 수 있다.(제공해주기 때문에)

## **3.6.3 queryForObject(**)

get() 메소드에 JdbcTemplate을 적용해보자!

get() 메소드는 ResultSet에서 getCount()처럼 단순한 값이 아니라 **복잡한 User 오브젝트를** 만들어야한다.

### ResultSetExtractor vs RowMapper

- **ResultSetExtractor**는 ResultSet을 한 번 전달받아 알아서 추출 작업을 모두 진행하고 최종 결과만 리턴한다.
- **RowMapper**는 ResultSet의 로우 하나를 매핑하기 위해 사용되기 때문에 여러 번 호출될 수 있다.

**queryForObject()와 RowMapper를 적용한 get() 메소드**

```java
public User get(String id) {
	return this.jdbcTemplate.queryForObject("select * from users where id = ?",
				 new Object[] {id}, // SQL에 바인딩할 파라미터 값. 가변인자 대신 배열을 사용한다.
				 new RowMapper<User>() {
						public User mapRow(ResultSet rs, int rowNum) throws SQLException {
							 User user = new User();
							 user.setId(rs.getString("id"));
							 user.setName(rs.getString("name"));
							 user.setPassword(rs.getString("password"));
							 return user;
						}
				}
}

```

- new queryForObject(PreparedStatement를 만들기위한 SQL, SQL에 바인딩할 값, 결과값 매퍼)
- RowMapper 콜백은 첫 번째 로우에 담긴 정보를 하나의 User 오브젝트로 매핑되도록 만들면 된다.
- queryForObject() 내부에는 이미 예외 처리가 되어있어서 조회 결과가 없는 예외상황 등을 처리하지 않아도 된다!

## **3.6.4 query(**)

### 기능 정의와 테스트 작성

현재 등록되어 있는 모든 사용자 정보를 가져오는 **getAll() 메소드**를 추가해보자!

**getAll()**

- List<User> 타입으로 반환
- List에 id(기본키) 기준으로 오름차순 정렬하여 저장

**테스트 동작 방식**

1. User 타입의 오브젝트인 user1, user2, user3 세 개를 DB에 등록한다.
2. getAll()을 호출하여 List〈User〉타입으로 결과를 받는다.
3. 리스트의 크기는 3이어야 하고, user1. user2, user3과 동일한 내용을 가진 오브젝트가 id 순서대로 담겨있어야 한다.

**getAll() 테스트**

```java
@Test
public void getAll() {
	dao.deleteAll();

	dao.add(user1); //Id： gyumee
	List<User> users1 = dao.getAll();
	assertThat(users1.size(), is(1));
	checkSameUser(user1, users1.get(0));

	dao.add(user2); //Id： leegw700
	List<User> users2 = dao.getAll();
	assertThat(users2.size(), is(2));
	checkSamellser(user1, users2.get(0));
	checkSameUser(user2, users2.get(1));

	dao.add(user3); //Id： bumjin
	List<User> users3 = dao.getAll();
	assertThat(users3.size(), is(3));
	checkSameUser(user3, users3.get(0));
	checkSameuser(user1, users3.get(1));
	checkSameUser(user2, users3.get(2));
} 

private void checkSameUser(User userl, User user2) {
	assertThat(user1.getld), is(user2.getld()))；
	assertThat(user1.getName(), is(user2.getName()))；
	assertThat(user1.getPassword(), is(user2.getPassword()))；
}
```

**getAll()**

```java
// query()의 리턴타입은 List<T>이다. 여러값을 받을 때 사용하면 편리하다.
public List<User> getAll() {
		return this.jdbcTemplate.query("select * from users order by id",
			new RowMapper<User>() {
					public User mapRow(ResultSet rs, int rowNum) throws SQLException { 
							User user = new User(); 
							user.setld(rs.getString("id")); 
							user.setName(rs.getString("name")); 
							user.setPassword(rs.getString("password")); 
						  return user;
					}
			});
}
```

- **query()**는 제네릭 메소드로 타입은 파라미터로 넘기는 **RowMapper<T>** 콜백 오브젝트에서 결정된다.

<aside>
💡 예외상황에 대한 테스트를 빼먹지 말자!

</aside>

## **3.6.5** 재사용 가능한 콜백의 분리

### DI를 위한 코드 정리

<aside>
💡 UserDao의 DataSource 인스턴스 변수 제거

- JdbcTemplate에서 DataSource를 사용하니 이제 필요없기 때문이다.
- 하지만 JdbcTemplate에 직접 DI해줘야 하니 setDataSource()는 남겨두자!
</aside>

### 중복 제거

get()과 getAll()을 보면 사용한 RowMapper의 내용이 똑같다. 두 메소드에서 사용되는 상황은 다르지만 **ResultSet 1개를 User 오브젝트 하나로 변환해주는 동일한 기능**을 가진 콜백이다.

### 해결 방법

- **RowMapper 콜백**을 메소드에서 분리해 중복을 없애고 **재사용되게** 만들자!

- **RowMapper 콜백**에는 **상태정보가 없기에** 하나의 콜백 오브젝트를 멀티 스레드에서 **동시에 사용**해도 문제가 되지 않는다!

**완성된 UserDao**

```java
public class UserDao {
	
  private JdbcTemplate jdbcTemplate; 
	
  public void setDataSource(DataSource dataSource) {
		 this.jdbcTemplate = new JdbcTemplate(dataSource);
	}
	
	//get(), getAll()에서 재사용된다.
	private RowMapper<User> userMapper = new RowMapper<User>() {
			public User mapRow(ResultSet rs, int rowNum) throws SQLException {
				  User user = new User(); 
				  user.setld(rs.getString("id"); 
				  user.setName(rs.getString("name")); 
				  user.setPassword(rs.getString("password")); 
				  return user;
			}
	}

	public void add(final User user) { 
			this.jdbcTemplate.update("insert into users(id, name, password) values(?, ?, ?)",
						user.getId(), user.getName(), user.getPassword()); 
	}

	public User get(String id) { 
			return this.jdbcTemplate.queryForObject("select * from users where id = ?" , 
					new Object[] {id}, this.userMapper);

	public void deleteAll() {
			this.jdbcTemplate.update("delete from users"); 
	}

	public int getCount() { 
			return this.jdbcTemplate.queryForlnt("select count(*) from users"); 
	}

	public List<User> getAll() { 
			return this.jdbcTemplate.query("select * from users order by id", this.userMapper);
	}
}
```
