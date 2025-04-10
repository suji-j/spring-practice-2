# 3-1. 전체 흐름 정리

## 1. 전체 흐름

- 새로운 할인 정책 개발
- 새로운 할인 정책 적용과 문제점
- 관심사의 분리
- AppConfig 리팩터링
- 새로운 구조와 할인 정책 적용

<br/>

### 1️⃣ 새로운 할인 정책 개발

- 다형성 덕분에 새로운 정률 할인 정책 코드를 추가로 개발하는 것 자체는 아무 문제가 없다.

<br/>

### 2️⃣ 새로운 할인 정책 적용과 문제점

- 새로 개발한 정률 할인 정책을 적용하려고 하니 **클라이언트 코드인 주문 서비스 구현체도 함께 변경해야 한다.**
- 주문 서비스 클라이언트가 인터페이스인 `DiscountPolicy` 뿐만 아니라, 구체 클래스인 `FixDiscountPolicy` 도 힘께 의존한다. → **DIP 위반**

<br/>

### 3️⃣ 관심사의 분리

- 기존에는 클라이언트가 의존하는 서버 구현 객체를 직접 생성하고 실행했다.
- 하지만, 지정하는 책임을 담당하는 별도의 `AppConfig`가 등장.
- `AppConfig`는 애플리케이션의 전체 동작 방식을 구성(Config)하기 위해, 구현 객체를 생성하고, 연결하는 책임이 있다.
- 이제부터 클라이언트 객체는 자신의 역할을 실행하는 것만 집중한다.

→ 권한이 줄어들고 책임이 명확해졌다.

<br/>

### 4️⃣ AppConfig 리팩터링

- 구성 정보에서 역할과 구현을 명확하게 분리한다.
- 역할이 잘 드러난다.
- 중복을 제거한다.

<br/>

### 5️⃣ 새로운 구조와 할인 정책 적용

- 정액 할인 정책 → 정률% 할인 정책으로 변경한다.
- `AppConfig`의 등장으로 애플리케이션이 크게 **사용 영역**과, 객체를 생성하는 **구성 영역**으로 분리되었다.
- 할인 정책을 변경해도 `AppConfig`가 있는 구성 영역만 변경하면 된다.
- 사용 영역은 변경할 필요가 없다.
- 물론 클라이언트 코드인 주문 서비스 코드도 변경하지 않는다.

<br/>

## 2. 좋은 객체 지향 설계의 5가지 원칙의 적용

### 1️⃣ SRP - 단일 책임 원칙

**한 클래스는 하나의 책임만 가져야 한다.**

- 클라이언트 객체는 직접 구현 객체를 생성하고, 연결하고, 실행하는 다양한 책임을 가지고 있다.
- SRP(단일 책임 원칙)을 따르면서 관심사를 분리한다.
- 구현 객체를 생성하고 연결하는 책임은 AppConfig가 담당한다.
- 클라이언트 객체는 실행하는 책임만 담당한다.

<br/>

### 2️⃣ DIP - 의존관계 역전 원칙

**프로그래머는 “추상화에 의존해야지, 구체화에 의존하면 안 된다.” 의존성 주입은 이 원칙을 따르는 방법 중 하나다.**

- 새로운 할인 정책을 개발하고, 적용하려고 하니 클라이언트 코드도 함께 변경해야 했다.
- 왜냐하면, 기존 클라이언트 코드(`OrderServiceImpl`)는 DIP를 지키며 `DiscountPolicy` 추상화 인터페이스에 의존하는 것 같았지만, `FixDiscountPolicy` 구체화 구현 클래스에도 함께 의존했다.
- 클라이언트 코드가 `DiscountPolicy` 추상화 인터페이스에만 의존하도록 코드를 변경했다.
- 하지만 클라이언트 코드는 인터페이스만으로는 아무것도 실행할 수 없다.
- `AppConfig` 가 `FixDiscountPolicy` 객체 인스턴스를 클라이언트 코드 대신 생성해서 클라이언트 코드에 의존관계를 주입했다.

→ DIP 원칙을 따르면서 문제도 해결했다.

<br/>

### 3️⃣ OCP - 개방 폐쇄 원칙

**소프트웨어 요소는 확장에는 열려 있으나 변경에는 닫혀 있어야 한다.**

- 다형성 사용하고 클라이언트가 DIP를 지킨다.
- 애플리케이션을 사용 영역과 구성 영역으로 나눈다.
- `AppConfig` 가 의존관계를 `FixDiscountPolicy` → `RateDiscountPolicy`로 변경해서 클라이언트 코드에 주입하므로 클라이언트는 코드를 변경하지 않아도 된다.
- **소프트웨어 요소를 새롭게 확장해도 사용 영역의 변경은 닫혀있다.**

<br/>

## 3. IoC, DI, 그리고 컨테이너

### 1️⃣ 제어의 역전 - IoC (Inversion of Control)

- 기존 프로그램은 클라이언트 구현 객체가 스스로 필요한 서버 구현 객체를 생성하고, 연결하고 실행했다.
- 한마디로 구현 객체가 프로그램의 제어 흐름을 스스로 제어했다. (개발자 입장에서는 자연스러운 흐름이다.)
- 반면에, `AppConfig` 가 등장한 이후에 구현 객체는 자신의 로직을 실행하는 역할만 담당한다.
- 프로그램의 제어 흐름은 이제 `AppConfig` 가 가져간다. (ex. `OrderServiceImpl` 은 필요한 인터페이스들을 호출하지만 어떤 구현 객체들이 실행될지 모른다.)
- 프로그램에 대한 제어 흐름에 대한 권한은 모두 `AppConfig` 가 가지고 있다. (심지어 OrderServiceImpl 도 AppConfig가 생성한다.)
- `AppConfig` 는 `OrderServiceImpl` 이 아닌 OrderService 인터페이스의 다른 구현 객체를 생성하고 실행할 수도 있다. (OrderServiceImpl은 묵묵히 자신의 로직을 실행할 뿐이다.)
- **이렇듯 프로그램의 제어 흐름을 직접 제어하는 것이 아니라 외부에서 관리하는 것을 제어의 역전(IoC)이라 한다.**

<br/>

### 2️⃣ 프레임워크 vs 라이브러리

- 프레임워크가 내가 작성한 코드를 제어하고, 대신 실행하면 그것은 프레임워크가 맞다. (ex. JUnit)
- 반면에 내가 작성한 코드가 직접 제어의 흐름을 담당한다면 그것은 프레임워크가 아니라 라이브러리다.

<br/>

## 4. 의존관계 주입 - DI (Dependency Injection)

- `OrderServiceImpl` 은 `DiscountPolicy` 인터페이스에 의존한다.
- 실제 어떤 구현 객체가 사용될지는 모른다.
- 의존관계는 **정적인 클래스 의존 관계와, 실행 시점에 결정되는 동적인 객체(인스턴스) 의존 관계** 둘을 분리해서 생각해야 한다.

<br/>

### 1️⃣ 정적인 클래스 의존 관계

- 클래스가 사용하는 Import 코드만 보고 의존 관계를 쉽게 판단할 수 있다.
- 정적인 의존 관계는 애플리케이션을 실행하지 않아도 분석할 수 있다.

<br/>

### 2️⃣ 클래스 다이어그램

![Image](https://github.com/user-attachments/assets/7717c788-086f-4c8e-9efa-af99fe02a647)

- `OrderServiceImpl` 은 `MemberRepository` , `DiscountPolicy` 에 의존한다는 것을 알 수 있다.
- 그런데 이러한 클래스 의존관계 만으로는 실제 어떤 객체가 `OrderServiceImpl` 에 주입될지 알 수 없다.

<br/>

### 3️⃣ 동적인 객체 인스턴스 의존 관계

- 애플리케이션 실행 시점에 실제 생성된 객체 인스턴스의 참조가 연결된 의존 관계이다.

<br/>

### 4️⃣ 객체 다이어그램

<img width="848" alt="Image" src="https://github.com/user-attachments/assets/ed6ecb16-7e3b-4b50-a13b-9580c0b02e24" />

- 애플리케이션 **실행 시점(런타임)** 에 의부에서 실제 구현 객체를 생성하고 클라이언트에 전달해서 클라이언트와 서버의 실제 의존 관계가 연결되는 것을 **의존관계 주입**이라 한다.
- 객체 인스턴스를 생성하고, 그 참조값을 전달해서 연결된다.
- 의존 관계 주입을 사용하면 클라이언트 코드를 변경하지 않고, 클라이언트가 호출하는 대상의 타입 인스턴스를 변경할 수 있다. (ex. `FixDiscountPolicy` → `RateDiscountPolicy` )
- 의존 관계 주입을 사용하면 정적인 클래스 의존 관계를 변경하지 않고, 동적인 객체 인스턴스 의존 관계를 쉽게 변경할 수 있다.

<br/>

## 5. IoC 컨테이너, DI 컨테이너

- `AppConfig` 처럼 객체를 생성하고 관리하면서 의존 관계를 연결해 주는 것을 IoC 컨테이너, 또는 `DI 컨테이너`라고 한다.
- 의존 관계 주입에 초점을 맞추어 최근에는 주로 DI 컨테이너라 한다.
- 또는 어샘블러, 오브젝트 팩토리 등으로 불리기도 한다.

<br/>

## 6. 스프링으로 전환하기

### 1️⃣ AppConfig 스프링 기반으로 변경

```java
@Configuration
public class AppConfig {
    @Bean
    public MemberService memberService() {
        return new MemberServiceImpl(memberRepository());
    }

    @Bean
    public MemberRepository memberRepository() {
        return new MemoryMemberRepository();
    }

    @Bean
    public OrderService orderService() {
        return new OrderServiceImpl(memberRepository(), discountPolicy());
    }

    @Bean
    public DiscountPolicy discountPolicy() {
        return new RateDiscountPolicy();
    }
}

```

- AppConfig에 설정을 구성한다는 뜻의 @`Configuration` 을 붙여준다
- 각 메서드에 `@Bean` 을 붙여준다. → 스프링 컨테이너에 스프링 빈으로 등록한다.

<br/>

### 2️⃣ MemberApp에 스프링 컨테이너 적용

```java
public class MemberApp {
    public static void main(String[] args) {
//      AppConfig appConfig = new AppConfig();
//      MemberService memberService = appConfig.memberService();

        ApplicationContext applicationContext = new AnnotationConfigApplicationContext(AppConfig.class);
        MemberService memberService = applicationContext.getBean("memberService", MemberService.class);

        Member member = new Member(1L, "memberB", Grade.VIP);
        memberService.join(member);

        Member findMember = memberService.findMember(1L);
        System.out.println("new member = " + member.getName());
        System.out.println("find Member = " + findMember.getName());
    }
}
```

- `getBean(”memberService”)`  == AppConfig의 `memberService` 메서드 명이 같아야 한다.

```java
@Bean
public MemberService memberService() {
    return new MemberServiceImpl(memberRepository());
}
```

<br/>

### 3️⃣ OrderApp에 스프링 컨테이너 적용

```java
public class OrderApp {
    public static void main(String[] args) {
//     AppConfig appConfig = new AppConfig();
//     MemberService memberService = appConfig.memberService();
//     OrderService orderService = appConfig.orderService();
		
        ApplicationContext applicationContext = new 
        AnnotationConfigApplicationContext(AppConfig.class);
				
        MemberService memberService = applicationContext.getBean("memberService", MemberService.class);
        OrderService orderService = applicationContext.getBean("orderService", OrderService.class);
				
        Member member = new Member(1L, "memberA", Grade.VIP);
        memberService.join(member);
				
        Order order = orderService.createOrder(memberId, "itemA", 10000);
        System.out.println("order = " + order);
    }
}
```

<br/>

### 4️⃣ 스프링 컨테이너

- `ApplicationContext` 를 `스프링 컨테이너`라 한다.
- 기존에는 개발자가 `AppConfig` 를 사용해서 직접 객체를 생성하고 DI를 했지만, 이제부터는 스프링 컨테이너를 통해서 사용한다.
- 스프링 컨테이너는 `@Configuration` 이 붙은 `AppConfig` 를 설정(구성) 정보로 사용한다.
- 여기서 `@Bean` 이라 적힌 메서드를 모두 호출해서 반환된 객체를 스프링 컨테이너에 등록한다. → 스프링 컨테이너에 등록된 객체를 스프링 빈이라 한다.
- 스프링 빈은 `@Bean` 이 붙은 메서드의 명을 스프링 빈의 이름으로 사용한다. (`memberService` ,`orderService` )
- 이전에는 개발자가 필요한 객체를 `AppConfig` 를 사용해서 직접 조회했지만, 이제부터는 스프링 컨테이너를 통해서 필요한 스프링 빈(객체)을 찾아여 한다. → 스프링 빈은 `applicationContext.getBean()` 메서드를 사용해서 찾을 수 있다.
- 기존에는 개발자가 직접 자바 코드로 모든 것을 했다면 이제부터는 스프링 컨테이너에 객체를 스프링 빈으로 등록하고, 스프링 컨테이너에서 스프링 빈을 찾아서 사용하도록 변경되었다.