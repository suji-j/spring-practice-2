# 6. 컴포넌트 스캔

## 1. 컴포넌트 스캔과 의존관계 자동 주입

- 지금까지 스프링 빈을 등록할 때는 자바 코드의 `@Bean`이나 XML의 `<bean>` 등을 통해서 설정 정보에 직접 등록할 스프링 빈을 나열했다.
- 스프링은 설정 정보가 없어도 자동으로 스프링 빈을 등록하는 컴포넌트 스캔이라는 기능을 제공한다.
- 의존 관계도 자동으로 주입하는 `@Autowried` 라는 기능도 제공한다.

<br/>

### 1️⃣ 코드로 컴포넌트 스캔과 의존관계 자동 주입

`AutoAppConfig.java`

```java
@Configuration
@ComponentScan(
    excludeFilters = @ComponentScan.Filter(type = FilterType.ANNOTATION, classes = Configuration.class)
)
public class AutoAppConfig {

}

```

- 컴포넌트 스캔을 사용하려면 먼저 `@ComponentScan` 을 설정 정보에 붙여주면 된다.
- 기존의 AppConfig와는 다르게 `@Bean`으로 등록한 클래스가 하나도 없다.

> 참고 : 컴포넌트 스캔을 사용하면 `@Configuration` 이 붙은 설정 정보도 자동으로 등록되기 때문에, AppConfig, TestConfig 등 앞서 만들어두었던 설정 정보도 함께 등록되고, 실행되어 버린다. `excludeFilter` 를 이용해서 설정 정보는 컴포넌트 스캔 대상에서 제외했다. 보통 설정 정보를 컴포넌트 스캔 대상에서 제외하지는 않는다.
>

<br/>

- 컴포넌트 스캔은 이름 그대로 `@Component` 애노테이션이 붙은 클래스를 스캔해서 스프링 빈으로 등록한다.

> 참고 : `@Configuration` 이 컴포넌트 스캔의 대상이 된 이유도 `@Configuration` 소스 코드를 열어보면 `@Component` 애노테이션이 붙어있기 때문이다.
>

<br/>

### 2️⃣ @Component 추가

- `MemoryMemberRepository`

```java
@Component
public class MemoryMemberRepository implements MemberRepository {}
```

<br/>

- `RateDiscountPolicy`

```java
@Component
public class RateDiscountPolicy implements DiscountPolicy {}
```

<br/>

- `MemberServiceImpl`

```java
@Component
public class MemberServiceImpl implements MemberService {
    private final MemberRepository memberRepository;
	
    @Autowired
    public MemberServiceImpl(MemberRepository memberRepository) {
        this.memberRepository = memberRepository;
    }
}
```

- 이전에 `AppConfig` 에서는 `@Bean` 으로 직접 설정 정보를 작성했고, 의존관계도 직접 명시했다.
- 이제는 이런 설정 정보 자체가 없기 때문에, 의존 관계 주입도 이 클래스 안에서 해결해야 한다.
- `@Autowried` 는 의존 관계를 자동으로 주입해준다.

<br/>

- `OrderServiceImpl`

```java
@Component
public class OrderServiceImpl implements OrderService {
    private final MemberRepository memberRepository;
    private final DiscountPolicy discountPolicy;
		
    @Autowired
    public OrderServiceImpl(MemberRepository memberRepository, DiscountPolicy discountPolicy) {
        this.memberRepository = memberRepository;
        this.discountPolicy = discountPolicy;
    }
}
```

- `@Autowried` 를 사용하면 생성자에서 여러 의존 관계도 한 번에 주입받을 수 있다.

<br/>

### 3️⃣ AutoAppConfigTest

```java
public class AutoAppConfigTest {
    @Test
    void basicScan() {
        AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(AutoAppConfig.class);

        MemberService memberService = ac.getBean(MemberService.class);
        assertThat(memberService).isInstanceOf(MemberService.class);
    }
}
```

- `AnnotationConfigApplicationContext` 를 사용하는 것은 기존과 동일하다.
- 설정 정보로 `AutoAppConfig` 클래스를 넘겨준다.
- 로그를 보면 컴포넌트 스캔이 잘 동작하는 것을 확인할 수 있다.

    ```text
    ClassPathBeanDefinitionScanner - Identified candidate component class:
    .. RateDiscountPolicy.class
    .. MemberServiceImpl.class
    .. MemoryMemberRepository.class
    .. OrderServiceImpl.class
    ```

<br/>

## 2. 컴포넌트 스캔과 자동 의존관계 주입의 동작 과정

### 1️⃣ @ComponentScan

- `@ComponentScan` 은 `@Component` 가 붙은 모든 클래스를 스프링 빈으로 등록한다.
- 이 때 스프링 빈의 기본 이름은 클래스명을 사용하되, 맨 앞글자만 소문자를 사용한다.
- 빈 이름 기본 전략 : `MemberServiceImpl 클래스`  → `memberServiceImpl`
- 빈 이름 직접 지정 : 만약 스프링 빈의 이름을 직접 지정하고 싶으면 `@Component(”memberService2”)` 으로 이름을 부여하면 된다.

<br/>

### 2️⃣ @Autowired 의존 관계 자동 주입

- 생성자에 `@Autowired` 를 지정하면 스프링 컨테이너가 자동으로 해당 스프링 빈을 찾아서 주입한다.
- 이때 기본 조회 전략은 타입이 같은 빈을 찾아서 주입한다.
    - `getBean(MemberRepository.class)` 와 동일하다고 이해한다.
- 생성자에 파라미터가 많아도 다 찾아서 자동으로 주입한다.

<br/>

## 2. 탐색 위치와 기본 스캔 대상

### 1️⃣ 탐색할 패키지의 시작 위치 지정

- 모든 자바 클래스를 다 컴포넌트 스캔하면 시간이 오래 걸린다. 그래서 필요한 위치부터 탐색하도록 시작 위치를 지정할 수 있다.

```java
@ComponentScan(
	basePackages = "hello.core",
}
```

- `basePackages`  : 탐색할 패키지의 시작 위치를 지정한다. 이 패키지를 포함해서 하위 패키지를 모두 탐색한다.
  - `basePackages = {"hello.core", "hello.service"}` : 여러 개 지정할 수도 있다.
- `basePackagesClasses`  : 지정한 클래스의 패키지를 탐색 시작 위치로 지정한다.
- 만약 지정하지 않으면 `@ComponentScan` 이 붙은 설정 정보 클래스의 패키지가 시작 위치가 된다.

<br/>

**[ 권장하는 방법 ]**

- 패키지 위치를 지정하지 않고, 설정 정보 클래스의 위치를 프로젝트 최상단에 두는 것이다.
  - com.hello
  - com.hello.service
  - com.hello.repository
- 이 구조라면, 프로젝트 시작 루트인 `com.hello` 에 AppConfig 같은 메인 설정 정보를 두고, `@ComponentScan` 애노테이션을 붙이고, `basePackages` 지정은 생략한다.
- `com.hello` 를 포함한 하위는 모두 자동으로 컴포넌트 스캔의 대상이된다.

<br/>

### 2️⃣ 컴포넌트 스캔 기본 대상

컴포넌트 스캔은 `@Component` 뿐만 아니라 다음과 같은 내용도 추가로 대상에 포함한다.

- `@Component`  : 컴포넌트 스캔에서 사용
- `@Controller`  : 스프링 MVC 컨트롤러에서 사용
- `@Service`  : 스프링 비즈니스 로직에서 사용
- `@Repository`  : 스프링 데이터 접근 계층에서 사용
- `@Configuration` : 스프링 설정 정보에서 사용

<br/>

### 3️⃣ 애노테이션에 따른 부가 기능

- `@Controller`  : 스프링 MVC 컨트롤러로 인식
- `@Repository`  : 스프링 데이터 접근 계층으로 인식하고, 데이터 계층의 예외를 스프링 예외로 변환해준다.
- `@Configuration`  : 스프링 설정 정보로 인식하고, 스프링 빈이 싱긇톤을 유지하도록 추가처리를 한다.
- `@Service` : 특별한 처리가 없다. 대신 개발자들이 핵심 비즈니스 계층을 인식하도록 한다.