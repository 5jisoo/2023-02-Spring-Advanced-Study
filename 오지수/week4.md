# 4장 예외

## 🌿 사라진 SQLException

JdbcTemplate 바꾸기 전과 후의 deleteAll() 메소드를 비교해보자

- `throws SQLException` 선언이 적용 후에는 사라졌음.

```java
public void deleteAll() throws SQLException {
    this.jdbcContext.executeSql("delete from users");
}
```

```java
public void deleteAll() {
    this.jdbcTemplate.update("delete from users");
}
```

### 🌱 초난감 예외처리

#### 🌵 예외 블랙홀

예외를 잡고는 아무것도 하지 않음!

예외 발생을 무시해버리고 정상적인 상황인 것처럼 다음 라인으로 넘어가겠다는 분명한 의도가 있는 게 아니라면 연습 중에도 절대 만들어서는 안되는 코드.

```java
try {
    ...
} catch (SQLException e) {

}
```

- 예외가 발생하면 그것은 catch 블록을 써서 잡아내는 것까지는 좋은데 그리고 아무것도 하지 않고 별문제 없는 것처럼 넘어가 버리는 건 정말 위험한 일이다.

화면에 출력해주는 것 역시도 다른 로그나 메세지에 금방 묻혀버리면 놓치기 쉽다.

- 운영 서버에 올라가도 콘솔 로그를 누군가가 계속 모니터링하지 않는 한 이 예외 코드는 심각한 폭탄으로 남아 있을 것.

<br>

예외를 처리할 때 반드시 지켜야 할 핵심 원칙은 한 가지이다.

**모든 예외는 적절하게 복구되는지 아니면 작업을 중단시키고 운영자 또는 개발자에게 분명하게 통보되어야 한다.**

<br>

SQLException이 발생하는 이유는 SQL에 문법 에러가 있거나 DB에서 처리할 수 없을 정도로 데이터 액세스 로직에 심각한 버그가 있거나, 서버가 죽거나 네트워크가 끊기는 등의 심각한 상황이 벌어졌기 때문이다.

#### 🌵 무의미하고 무책임한 throws

메소드 선언에 기계적으로 throws Exception을 붙이는 개발자도 있다.

- 이 습관도 굉장히 유의해야 함.

```java
public void method1() throws Exception {
    method2();
    ...
}

public void method2() throws Exception {
    ...
}
```

이런 무책임한 throws 선언도 심각한 문제점이 있다.

- 의미 있는 정보를 얻을 수 없다.
  - 정말 실행 중에 예외적인 상황이 발생할 수 있다는 것인지,
  - 아니면 습관적으로 복사해서 붙여놓은 것인지 알 수 없음.

### 🌱 예외의 종류와 특징

#### 🌵 Error

- `java.lang.Error` 클래스의 서브클래스들.

에러는 시스템에 뭔가 비정상적인 상황이 발생했을 경우에 사용된다.

- Java VM에서 발생되기 때문에 애플리케이션 코드에서 잡으려고 하면 안됨.

딱히 애플리케이션에서 이 에러에 대한 처리는 신경 쓰지 않아도 된다!

#### 🌵 Exception과 체크 예외

- `java.lang.Exception` 클래스와 그 서브클래스로 정의되는 예외들은 개발자들이 만든 애플리케이션 코드의 작업 중에 예외상황이 발생했을 경우에 사용됨.

Exception 클래스는 다시 **체크 예외**와 **언체크 예외**로 구분된다.

- 체크 예외: Excpetion 클래스의 서브 클래스, RuntimeException 클래스를 상속받지 않은 것.
  - 에러처리를 하지 않으면 컴파일 에러가 발생함. 반드시 try-catch 또는 throws 문을 활용한 에러 처리가 필요.
- 언체크 예외: RuntimeException 클래스를 상속받은 것.
  - 물론 언체크 예외는 Exception 클래스를 상속받았으나, **자바는 이 RuntimeException과 이를 상속받은 서브 클래스들은 특별하게 다룬다!**

#### 🌵 RuntimeException과 언체크/런타임 예외

- `java.lang.RuntimeException` 클래스를 상속한 예외들은 명시적인 예외처리를 강제하지 않는다. 그렇기에 **언체크 예외** 혹은 **런타임 예외**라고 불린다.

런타임예외는 주로 프로그램의 오류가 있을 때 발생하도록 의도된 것들이다. 대표적으로

- NullPointerException: 오브젝트를 할당하지 않은 레퍼런스 변수를 사용하려고 시도했을 때 발생
- IllegalArgumentException: 허용되지 않은 값을 사용하여 메소드를 호출할 때 발생

위 예외들은 코드에서 미리 조건을 체크하여 주의 깊게 만들어야 한다!

- 따라서 **런타임 예외는 예상하지 못했던 예외상황에서 발생하는 게 아니기 때문에 굳이 catch 또는 throws를 사용하지 않아도 되는 것.**

### 🌱 예외처리 방법

#### 🌵 예외 복구

- 예외상황을 파악하고 문제를 해결해서 정상 상태로 돌려놓는 것.

예외로 인해 기본 작업 흐름이 불가능함녀 다른 작업 흐름으로 자연스럽게 유도해주는 것!

이런 경우 예외상황은 다시 정상으로 돌아오고 예외를 복구했다고 볼 수 있다.

- 단 비록 기능적으로는 사용자에게 예외상황으로 비쳐도 애플리케이션에서는 정상적으로 설계된 흐름을 따라 진행되어야 한다.

```java
int maxretry = MAX_RETRY;

while(maxretry-- > 0) {
    try {
        ...         // 예외가 발생할 가능성이 있는 시도
        return ;    // 작업 성공
    } catch (SomeException e) {
        // 로그 출력. 정해진 시간만큼 대기
    } finally {
        // 리소스 반납. 정리 작업
    }
}

throw new RetryFailedException(); // 최대 재시도 횟수를 넘기면 직접 예외 발생

```

#### 🌵 예외처리 회피

예외처리를 자신이 담당하지 않고 자신을 호출한 쪽으로 던져버리는 것!

- throws 문으로 선언하여 예외가 발생하면 알아서 던져지게 하거나 catch 문으로 일단 예외를 잡은 후에 로그를 남기고 다시 예외를 던지는 것.

**콜백과 템플릿처럼 긴밀하게 역할을 분담하고 있는 관계가 아니라면** 자신의 코드에서 발생하는 예외를 그냥 던져버리는 것은 **무책임한 책임회피**이다.

<br>

예외를 회피하는 것은 **예외를 복구하는 것처럼 의도가 분명**해야 함.

1. 콜백/템플릿처럼 긴밀한 관계에 있는 다른 오브젝트에게 예외처리 책임을 분명히 지게 하거나
2. 자신을 사용하는 쪽에서 예외를 다루는 게 최선의 방법이라는 분명한 확신이 있어야 한다.

#### 🌵 예외 전환

예외 회피와 비슷하게 예외를 복구해서 정상적인 상태로는 만들 수 없기 때문에 예외를 메소드 밖으로 던지는 것.

단 예외 회피와는 달리, **발생한 예외를 그대로 넘기는 게 아니라 적절한 예외로 전환해서 던진다는 특성이 존재**함.

보통 두 가지 목적으로 사용됨.

1. 내부에서 발생한 예외를 그대로 던지는 것이 그 예외상황에 대한 적절한 의미를 부여해주지 못하는 경우에, 의미를 분명하게 해줄 수 있는 예외로 바꿔주기 위해서.

   - API가 발생하는 기술적인 로우레벨을 상황에 적합한 의미를 가진 예외로 변경하는 것.

   ```java
   public void add(User user) throws DuplicateUserIdException, SQLException {
    try {
        // JDBC를 이용해 user 저옵를 DB에 추가하는 코드 or
        // 그런 기능을 가진 다른 SQLException을 던지는 메소드를 호출하는 코드
    } catch (SQLException e) {
        // ErrorCode가 MySQL의 "Duplicate Entry" 이면 예외 전환
        if (e.getErrorCode() == MysqlErrorNumbers.ER_DUP_ENTRY)
            throw DuplicateUserException();
        else
            throw e; // 그 외는 SQLException 그대로 던짐.
    }
   }
   ```

   - 보통 **전환되는 예외에 원래 발생한 예외를 담아서** 중첩 예외로 만드는 것이 가장 좋음. <br>
     중첩 예외는 `getCause()` 메소드를 이용해서 처음 발생한 예외가 무엇인지 확인할 수 있음.

   ```java
   catch (SQLException e) {
   ...
   throw DuplicateUserIdException(e);
   }
   ```

   ```java
   catch (SQLException e) {
   ...
   throw DuplicateUserIdException().initCause(e);
   }
   ```

2. 예외를 처리하기 쉽고 단순하게 만들기 위해 포장하는 것.

   - 중첩 예외를 이용해 새로운 예외를 만들고 원인이 되는 예외를 내부에 담아서 던지는 방식은 동일함.
   - 의미를 명확하게 하고자 다른 예외로 전환하는 것이 아닌, 주로 예외처리를 강제하는 체크 예외를 언체크 예외인 런타임 예외로 바꾸는 경우에 사용.

   ```java
   try {
       OrderHome orderHome = EJBHomeFactory.getInstance().getOrderHome();
       Order order = orderHome.findByPrimaryKey(Integer id);
   } catch (NamingException ne) {
       throw new EJBException(ne);
   } catch (SQLException se) {
       throw new EJBException(se);
   } catch (RemoteException re) {
       throw new EJBException(re);
   }
   ```

   - 여기서 EJBException은 RuntimeException 클래스를 상속한 런타임 예외.
   - 이렇게 예외로 만들어 전달하면 EJB는 이를 시스템 Exception으로 인식하고 트랜잭션을 자동으로 롤백함.

<br>

일반적으로 체크 예외를 계속 throws를 사용해 넘기는 것은 무의미하다.

어차피 복구가 불가능한 예외라면 가능한 한 빨리 런타임 예외로 포장하여 던지게 해서 다른 계층의 메소드를 작성할 때 불필요한 throws 선언이 들어가지 않도록 해줘야 한다.

### 🌱 예외처리 전략

#### 🌵 런타임 예외의 보편화

자바 엔터프라이즈 서버환경에서는 하나의 요청을 처리하는 중에 예외가 발생하면? <br>
해당 작업만 중단시키면 그만이다!

차라리 애플리케이션 차원에서 예외 상황을 미리 파악하고, 예외가 발생하지 않도록 차단하는 게 좋음.

또는 프로그램의 오류나 외부 환경으로 인해 예외가 발생하는 경우라면 빨리 해당 요청의 작업을 취소하고 서버 관리자나 개발자에게 통보해주는 편이 나음!

자바 초기부터 있었던 JDK의 API와 달리 최근에 등장하는 표준 스펙 또는 오픈소스 프레임워크에서는 **API가 발생시키는 예외를 체크 예외 대신 언체크 예외로 정의하는 것이 일반화**되고 있음.

- 지금은 항상 복구할 수 있는 예외가 아니라면 일단 언체크 예외로 만듦.

#### 🌵 add() 메소드의 예외처리

### 🌱 SQLException은 어떻게 됐나?

## 🌿 예외 전환

### 🌱 JDBC의 한계

### 🌱 DB 에러 코드 매핑을 통한 전환

### 🌱 DAO 인터페이스와 DataAccessException 계층구조

### 🌱 기술에 독립적인 UserDao 만들기
