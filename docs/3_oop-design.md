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