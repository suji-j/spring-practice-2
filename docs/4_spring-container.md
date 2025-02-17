# 4. 스프링 컨테이너와 스프링 빈

## 1. 스프링 컨테이너 생성

### 1️⃣ 스프링 컨테이너

```java
ApplicationContext applicationContext = new AnnotationConfigApplicationContext(AppConfig.class);
```

- `ApplicationContext` 를 스프링 컨테이너라 한다.
- `ApplicationContext` 는 인터페이스이다.
- 스프링 컨테이너는 XML 기반으로 만들 수 있고, 어노테이션 기반의 자바 설정 클래스로 만들 수 있다.
- 직전에 `AppConfig` 를 사용했던 방식이 어노테이션 기반의 자바 설정 클래스로 스프링 컨테이너를 만든 것이다.
- 자바 설정 클래스를 기반으로 스프링 컨테이너(`ApplicationContext`)를 만든다.
    - `new AnnotaionConfigApplicationContext(AppConfig.class);`
    - 이 클래스는 `ApplicationContext` 인터페이스의 구현체이다.

> 참고 : 더 정확히는 스프링 컨테이너를 부를 때, `BeanFactory`, `ApplicationContext` 로 구분헤서 이야기한다. `BeanFactory` 를 직접 사용하는 경우는 거의 없으므로 일반적으로 `ApplicationContext` 를 스프링 컨테이너라 한다.

<br/>

## 2. 스프링 컨테이너의 생성 과정

### 1️⃣ 스프링 컨테이너 생성

![Image](https://github.com/user-attachments/assets/d16a51b9-809b-4ea8-887e-8fbfb827e76b)

- `new AnnotaionConfigApplicationContext(AppConfig.class);`
- 스프링 컨테이너를 생성할 때는 구성 정보를 지정해주어야 한다.
- 여기서는 `AppConfig.class` 를 구성 정보로 지정했다.

<br/>

### 2️⃣ 스프링 빈 등록

![Image](https://github.com/user-attachments/assets/3035a1c4-a806-4c9f-ab00-9e8febfd7ef5)

- 스프링 컨테이너는 파라미터로 넘어온 설정 클래스 정보를 사용해서 스프링 빈을 등록한다.
- 빈 이름
  - 빈 이름은 메서드 이름을 사용한다.
  - 빈 이름은 직접 부여할 수도 있다.
  - `@Bean(name = “memberService2”)`

> 주의 : 빈 이름은 항상 다른 이름을 부여해야 한다. 같은 이름을 부여하면, 다른 빈이 무시되거나, 기존 빈을 덮어버리거나 설정에 따라 오류가 발생한다.
>

<br/>

### 3️⃣ 스프링 빈 의존 관계 설정 - 준비

![Image](https://github.com/user-attachments/assets/d9bcbd69-569c-418c-ae6f-b666a4571df5)

<br/>

### 4️⃣ 스프링 빈 의존 관계 설정 - 완료

![Image](https://github.com/user-attachments/assets/113e310b-230b-45f5-9723-ebf6c897df09)

- 스프링 컨테이너는 설정 정보를 참고해서 의존관계를 주입(DI)한다.
- 단순히 자바 코드를 호출하는 것 같지만, 차이가 있다.

> 참고 : 스프링은 빈을 생성하고, 의존관계를 주입하는 단계가 나누어져 있다. 그런데 자바 코드로 스프링 빈을 등록하면 생성자를 호출하면서 의존관계 주입도 한 번에 처리된다.
>
- 정리 : 스프링 컨테이너를 생성하고, 설정(구성) 정보를 참고해서 스프링 빈도 등록하고, 의존관계도 설정했다.