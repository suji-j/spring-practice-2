# 3. 스프링 핵심 원리 이해2 - 객체 지향 원리 적용

## 1. 새로운 할인 정책 개발

정률 할인 정책을 추가한다.

- VIP면, 할인률을 10%으로 변경한다. (ex. 10,000원 → 1,000 할인, 20,000원 → 2,000원 할인)

<br/>

### 1️⃣ RateDiscountPolicy 추가

<img width="866" alt="Image" src="https://github.com/user-attachments/assets/89a17664-15e7-4da6-868a-ffc0bb558d71" />

<br/><br/>

## 2. 새로운 할인 정책 적용과 문제점

### 1️⃣ 할인 정책을 애플리케이션에 적용

```java
public class OrderServiceImpl implements OrderService{    
//    private final DiscountPolicy discountPolicy = new FixDiscountPolicy();
    private final DiscountPolicy discountPolicy = new RateDiscountPolicy();
}
```

<br/>

### 2️⃣ 문제점 발견

- 역할과 구현을 충실하게 분리했다. → OK
- 다형성도 활용하고, 인터페이스와 구현 객체를 분리했다. → OK
- **OCP, DIP 같은 객체 지향 설계 원칙을 충실히 준수했다. → NO**
- DIP : 주문 서비스 클라이언트 (`OrderServcieImpl`)는 `DiscountPolicy` 인터페이스에 의존하면서 DIP를 지킨 것 같지만,
    - 클래스 의존 관계 : 추상(인터페이스) 뿐만 아니라 **구체(구현) 클래스에도 의존**하고 있다.
    - 추상 (인터페이스) 의존 : `DiscountPolicy`
    - 구체 (구현) 클래스 : `FixDiscountPolicy`, `RateDiscountPolicy`
- OCP : 변경하지 않고 확장할 수 없다.
    - 지금 코드는 기능을 확장해서 변경하면, 클라이언트 코드에 영향을 준다.
    - **OCP 위반**

<br/>

### 3️⃣ 왜 클라이언트 코드를 변경해야 할까?

**기대했던 의존 관계**

<img width="866" alt="Image" src="https://github.com/user-attachments/assets/89a17664-15e7-4da6-868a-ffc0bb558d71" />

- 지금까지 `DiscountPolicy` 인터페이스에만 의존한다고 생각했다.

<br/>

**실제 의존 관계**

<img width="883" alt="Image" src="https://github.com/user-attachments/assets/5f2f7506-0ec5-4158-92c4-56d7b84a8af9" />

- 클라이언트인 `OrderServiceImpl` 이 `DiscountPolicy` 인터페이스 뿐만 아니라 `FixDiscountPolicy` 인 구체 클래스도 함께 의존하고 있다.
- **DIP 위반**

<br/>

**정책 변경**

<img width="883" alt="Image" src="https://github.com/user-attachments/assets/cf9d063a-9e3b-431f-80cc-9cf718e4e897" />

- ** 그래서 `FixDiscountPolicy` 를 `RateDiscountPolicy` 로 변경하는 순간 `OrderServiceImpl` 의 소스 코드도 함께 변경해야 한다. **
- **OCP 위반**

<br/>

### 4️⃣ 인터페이스에만 의존하도록 설계를 변경한다.

- 클라이언트 코드인 `OrderServiceImpl` 은 `DiscountPolicy` 의 인터페이스 뿐만 아니라 구체 클래스도 함께 의존한다.
- 그래서 구체 클래스를 변경할 때 클라이언트 코드도 함께 변경해야 한다.
- **DIP 위반 → 추상에만 의존하도록 변경 (인터페이스에만 의존)**
- DIP를 위반하지 않도록 인터페이스에만 의존하도록 의존 관계를 변경하면 된다.
  <img width="866" alt="Image" src="https://github.com/user-attachments/assets/89a17664-15e7-4da6-868a-ffc0bb558d71" />

<br/>


**인터페이스에만 의존하도록 코드 변경**

```java
public class OrderServiceImpl implements OrderService{
    private final MemberRepository memberRepository = new MemoryMemberRepository();
    // 인터페이스에만 의존하도록 변경 (DIP)
    private DiscountPolicy discountPolicy;
}
```
<img width="866" alt="Image" src="https://github.com/user-attachments/assets/89a17664-15e7-4da6-868a-ffc0bb558d71" />

- 구현체가 없어서 코드를 실행해보면, NPE(Null Pointer Exception)이 발생한다.

<br/>

**해결방안**

- 누군가가 클라이언트인 `OrderServiceImpl` 에 `DiscountPolicy` 의 구현 객체를 대신 생성하고 주입해주어야 한다.

<br/>

## 3. 관심사의 분리

### 1️⃣ AppConfig

- 애플리케이션의 전체 동작 방식을 구성(config)하기 위해, **구현 객체를 생성**하고, **연결하는 책임**을 가지는 별도의 설정 클래스를 만든다.

```java
public class AppConfig {
    public MemberService memberService() {
        return new MemberServiceImpl(new MemoryMemberRepository());
    }

    public OrderService orderService() {
        return new OrderServiceImpl(new MemoryMemberRepository(), new FixDiscountPolicy());
    }
}
```

- AppConfig는 애플리케이션의 실제 동작에 필요한 **구현 객체를 생성**한다.
  - `MemberServiceImpl`
  - `MemoryMemberRepository`
  - `OrderServiceImpl`
  - `FixDiscountPolicy`
- AppConfig는 생성한 객체 인스턴스의 참조(레퍼런스)를 **생성자를 통해서 주입(연결)**해준다.
  - `MemberServiceImpl`  → `MemoryMemberRepository`
  - `OrderServiceImpl` → `MemoryMemberRepository` , `FixDiscountPolicy`

<br/>

### 2️⃣ MemberServiceImpl - 생성자 주입

```java
public class MemberServiceImpl implements MemberService {
    private final MemberRepository memberRepository;

    public MemberServiceImpl(MemberRepository memberRepository) {
        this.memberRepository = memberRepository;
    }
}
```

- 설계 변경으로 `MemberServiceImpl` 은  `MemoryMemberRepository` 를 의존하지 않는다.
- 단지 `MemberRepository` 인터페이스만 의존한다.
- `MemberServiceImpl` 입장에서 생성자를 통해 어떤 구현 객체가 주입될지는 알 수 없다.
- `MemberServiceImpl` 의 생성자를 통해서 어떤 구현 객체를 주입할지는 오직 외부(`AppConfig`)에서 결정된다.
- `MemberServiceImpl` 은 이제부터 **의존관계에 대한 고민은 외부**에 맡기고 **실행에만 집중**하면 된다.

<br/>

### 3️⃣ 다이어그램

**클래스 다이어그램**

<img width="745" alt="Image" src="https://github.com/user-attachments/assets/b5194ed6-89d5-4b55-941b-f9854f1e48a3" />

- 객체의 생성과 연결은 AppConfig가 담당한다.
- **DIP 완성 :** `MemberServiceImpl` 은 `MemberRepository` 인 추상에만 의존하면 된다. 구체 클래스를 몰라도 된다.
- **관심사의 분리 : 객체를 생성하고 연결하는 역할과 실행하는 역할이 명확히 분리되었다.**

<br/>

**회원 객체 인스턴스 다이어그램**

<img width="848" alt="Image" src="https://github.com/user-attachments/assets/ef3c95aa-dcf1-4f62-a8b4-e12c4b19c173" />

- `AppConfig` 객체는 `MemoryMemberRepository` 객체를 생성하고 그 참조값을 `MemberServiceImpl` 을 생성하면서 생성자로 전달한다.
- 클라이언트인 `MemberServiceImpl` 입장에서 보면 의존관계를 마치 외부에서 주입해주는 것 같다고 해서 `DI (Dependency Injection)` , 우리말로 의존 관계 주입 또는 의존성 주입이라 한다.

<br/>

### 4️⃣ OrderServiceImpl - 생성자 주입

```java
public class OrderServiceImpl implements OrderService{
    private final MemberRepository memberRepository;
    private final DiscountPolicy discountPolicy;

    public OrderServiceImpl(MemberRepository memberRepository, DiscountPolicy discountPolicy) {
        this.memberRepository = memberRepository;
        this.discountPolicy = discountPolicy;
    }
}
```

- 설계 변경으로 `OrderServiceImpl` 은 `FixDiscountPolicy` 를 의존하지 않는다.
- 단지 `DiscountPolicy` 인터페이스만 의존한다.
- `OrderServiceImpl` 입장에서 생성자를 통해 어떤 구현 객체가 주입될지는 알 수 없다.
- `OrderServiceImpl` 의 생성자를 통해서 어떤 구현 객체를 주입할지는 오직 외부(`AppConfig`)에서 결정한다.
- `OrderServiceImpl` 은 이제부터 실행에만 집중하면 된다.
- `OrderServiceImpl` 에는 `MemoryMemberRepository` , `FixDiscountPolicy` 객체의 의존관계가 주입된다.

<br/>

## 4. AppConfig 실행

### 1️⃣ 사용 클래스 - MemberApp

```java
public class MemberApp {
    public static void main(String[] args) {
        AppConfig appConfig = new AppConfig();
        MemberService memberService = appConfig.memberService();

        Member member = new Member(1L, "memberB", Grade.VIP);
        memberService.join(member);

        Member findMember = memberService.findMember(1L);
        System.out.println("new member = " + member.getName());
        System.out.println("find Member = " + findMember.getName());
    }
}
```

<br/>

### 2️⃣ 사용 클래스 - OrderApp

```java
public class OrderApp {
    public static void main(String[] args) {
        AppConfig appConfig = new AppConfig();
        MemberService memberService = appConfig.memberService();
        OrderService orderService = appConfig.orderService();

        Long memberId = 1L;
        Member member = new Member(memberId, "memberA", Grade.VIP);
        memberService.join(member);

        Order order = orderService.createOrder(memberId, "itemA", 10000);
        System.out.println("order = " + order);
    }
}
```

<br/>

### 3️⃣ 테스트 코드 오류 수정

```java
public class OrderServiceTest {
    MemberService memberService;
    OrderService orderService;

    @BeforeEach
    public void beforeEach() {
        AppConfig appConfig = new AppConfig();
        memberService = appConfig.memberService();
        orderService = appConfig.orderService();
    }
}
```

- 테스트 코드에서 `@BeforeEach` 는 각 테스트를 실행하기 전에 호출된다.

<br/>

**정리**

- AppConfig를 통해서 관심사를 확실하게 분리했다.
- AppConfig는 구체 클래스를 선택한다. 애플리케이션이 어떻게 동작해야 할지 전체 구성을 책임진다.
- 각 구현체들은 담당 기능을 실행하는 책임만 지면 된다.
- `OrderServiceImpl` 은 기능을 실행하는 책임만 지면 된다.

<br/>

## 5. AppConfig 리팩터링

- 현재 AppConfig는 중복이 있고, 역할에 따른 구현이 잘 안 보인다.

<br/>

### 1️⃣ 기대하는 그림

![Image](https://github.com/user-attachments/assets/cedb64f9-ae06-47ea-9813-61c7ac6ce743)

<br/>

### 2️⃣ 리팩터링 전

```java
public class AppConfig {
    public MemberService memberService() {
        return new MemberServiceImpl(new MemoryMemberRepository());
    }

    public OrderService orderService() {
        return new OrderServiceImpl(new MemoryMemberRepository(), new FixDiscountPolicy());
    }
}
```

- 중복을 제거하고, 역할에 따른 구현이 보이도록 리팩터링한다.
  - `new MemoryMemberRepository()` 중복

<br/>

### 3️⃣ 리팩터링 후

```java
public class AppConfig {
    public MemberService memberService() {
        return new MemberServiceImpl(memberRepository());
    }

    private MemberRepository memberRepository() {
        return new MemoryMemberRepository();
    }

    public OrderService orderService() {
        return new OrderServiceImpl(memberRepository(), discountPolicy());
    }

    public DiscountPolicy discountPolicy() {
        return new FixDiscountPolicy();
    }
}
```

- `new MemoryMemberRepository()` 이 부분이 중복 제거되었다.
- 이제 `MemoryMemberRepository` 를 다른 구현체로 변경할 때 한 부분만 변경하면 된다.
- `AppConfig` 를 보면 역할과 구현 클래스가 한 눈에 들어온다.
- 애플리케이션 전체 구성이 어떻게 되어있는지 빠르게 파악할 수 있다.

<br/>

## 6. 새로운 구조와 할인 정책 적용

- 정액 할인 정책 → 정률% 할인 정책으로 변경
- `FixDiscountPolicy`  → `RateDiscountPolicy`

→ **AppConfig의 등장으로 애플리케이션이 크게 사용 영역과, 객체를 생성하고 구성(Configuration)하는 영역으로 분리되었다.**

<br/>

### 1️⃣ 사용, 구성의 분리

<img width="848" alt="Image" src="https://github.com/user-attachments/assets/65476ef5-843b-4d3d-82ba-847224c8e07c" />

<br/>

### 2️⃣ 할인 정책의 변경

<img width="848" alt="Image" src="https://github.com/user-attachments/assets/62ff6372-8b82-4014-94e2-e4e77ce9a7ce" />

- `FixDiscountPolicy`  → `RateDiscountPolicy` 로 변경해도 구성 영역만 영향을 받고, 사용 영역은 전혀 영향을 받지 않는다.

<br/>

### 3️⃣ 할인 정책 변경 구성 코드

`AppConfig`

```java
public DiscountPolicy discountPolicy() {
//        return new FixDiscountPolicy();
        return new RateDiscountPolicy();
}
```

- AppConfig에서 할인 정책 역할을 담당하는 구현을 `FixDiscountPolicy`  → `RateDiscountPolicy` 객체로 변경했다.
- 이제 할인 정책을 변경해도, 애플리케이션의 구성 역할을 담당하는 AppConfig만 변경하면 된다.
- 클라이언트 코드인 `OrderServiceImpl` 를 포함해서 사용 영역의 어떤 코드도 변경할 필요가 없다.
- 구성 영역은 당연히 변경된다.
- 구성 영역을 담당하는 AppConfig는 애플리케이션의 구현 객체들을 모두 알아야 한다.