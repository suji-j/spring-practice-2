## 2. 스프링 핵심 원리 이해1 - 예제 만들기

## 1. 비즈니스 요구사항과 설계

### 1️⃣ 회원

- 회원은 가입하고 조회할 수 있다.
- 회원은 두 가지 등급(일반, VIP)이 있다.
- 회원 데이터는 자체 DB를 구축할 수 있고, 외부 시스템과 연동할 수 있다. (미확정)

<br/>

### 2️⃣ 주문과 할인 정책

- 회원은 상품을 주문할 수 있다.
- 회원 등급에 따라 할인 정책을 적용할 수 있다.
- 할인 정책은 모든 VIP는 1,000원을 할인해주는 고정 금액 할인을 적용해달라. (변경 가능성 O)
- 할인 정책은 변경 가능성이 높다. 회사의 기본 할인 정책을 아직 정하지 못했고, 오픈 직전까지 미루고 싶다. 최악의 경우 할인을 적용하지 않을 수도 있다.

변경 가능성이 높지만, 인터페이스를 만들고 구현체를 언제든지 갈아끼울 수 있도록 설계한다.
- 현재는 스프링 없는 순수 자바로만 개발을 진행한다.

---

## 2. 회원 도메인 설계

### **회원 도메인 요구사항**

- 회원을 가입하고 조회할 수 있다.
- 회원은 두 가지 등급(일반, VIP)이 있다.
- 회원 데이터는 자체 DB를 구축할 수 있고, 외부 시스템과 연동할 수 있다. (미확정)

<br/>

### 1️⃣ 회원 도메인 협력 관계
![Image](https://github.com/user-attachments/assets/205f0b6b-4895-493a-aec8-0c214c4e4e29)

<br/>

### 2️⃣ 회원 클래스 다이어그램
![Image](https://github.com/user-attachments/assets/47d51c2e-f2f9-4ec2-8eeb-1a328e6b2de7)

<br/>

### 3️⃣ 회원 객체 다이어그램
![Image](https://github.com/user-attachments/assets/f80277a7-28ae-48eb-b4b6-6da2b4da492c)

---

## 3. 회원 도메인 개발

### 1️⃣ 회원 엔티티

- **회원 등급 : `member/Grade.java`**
- **회원 저장소 인터페이스** **:** **`member/MemberRepository.java`**
- **메모리 회원 저장소 구현체 :** **`member/MemoryMemberRepository.java`**
    - DB가 아직 확정이 안되었으니 가장 단순한 메모리 회원 저장소를 구현한다.
    - `HashMap`은 동시성 이슈가 발생할 수 있으니 `ConcurrentHashMap`을 사용한다.

<br/>

### 2️⃣ 회원 서비스

- **회원 서비스 인터페이스 : `member/MemberService.java`**
- **회원 서비스 구현체 : `member/MemberServiceImpl.java`**

---

## 4. 회원 도메인 실행과 테스트

### 1️⃣ 회원 도메인 - 회원 가입 main

`MemberApp.java`

```java
package hello.core;

import hello.core.member.Grade;
import hello.core.member.Member;
import hello.core.member.MemberService;
import hello.core.member.MemberServiceImpl;

public class MemberApp {
    public static void main(String[] args) {
        MemberService memberService = new MemberServiceImpl();
        Member member = new Member(1L, "memberB", Grade.VIP);
        memberService.join(member);

        Member findMember = memberService.findMember(1L);
        System.out.println("new member = " + member.getName());
        System.out.println("find Member = " + findMember.getName());
    }
}
```

- 애플리케이션 로직으로 테스트하는 것은 좋은 방법이 아니다.
- JUnit 테스트를 사용한다.

<br/>

### 2️⃣ 회원 도메인 - 회원 가입 테스트

`test/.../member/MemberServiceTest`

```java
MemberService memberService = new MemberServiceImpl();

@Test
void join() {
        //given
        Member member = new Member(1L, "memberA", Grade.VIP);

        //when
        memberService.join(member);
        Member findMember = memberService.findMember(1L);

        //then
        Assertions.assertThat(member).isEqualTo(findMember);
}
```

<br/>

### 3️⃣ 회원 도메인 설계의 문제점

- 다른 저장소로 변경할 때 OCP 원칙을 잘 준수할까?
- DIP를 잘 지키고 있을까?

→ **의존관계가 인터페이스 뿐만 아니라 구현까지 모두 의존하는 문제점이 있음**

---

## 5. 주문과 할인 도메인 설계

### **주문과 할인 정책**

- 회원은 상품을 주문할 수 있다.
- 회원 등급에 따라 할인 정책을 적용할 수 있다.
- 할인 정책은 모든 VIP는 1,000원을 할인해주는 고정 금액 할인을 적용해달라. (변경 가능성 O)
- 할인 정책은 변경 가능성이 높다. 회사의 기본 할인 정책을 아직 정하지 못했고, 오픈 직전까지 미루고 싶다. 최악의 경우 할인을 적용하지 않을 수도 있다.

<br/>

### 1️⃣ 주문 도메인 협력, 역할, 책임

![Image](https://github.com/user-attachments/assets/ff0d7402-029a-467b-a69e-168185ae50cd)

1. **주문 생성 :** 클라이언트는 주문 서비스에 주문 생성을 요청한다.
2. **회원 조회 :** 할인을 위해서는 회원 등급이 필요하기 때문에 주문 서비스는 회원 저장소에서 회원을 조회한다.
3. **할인 적용 :** 주문 서비스는 회원 등급에 따른 할인 여부를 할인 정책에 위임한다.
4. **주문 결과 반환 :** 주문 서비스는 할인 결과를 포함한 주문 결과를 반환한다.

<br/>

### 2️⃣ 주문 도메인 전체

![Image](https://github.com/user-attachments/assets/fe5bfd4f-68d8-4674-933f-e82ef65a4f8c)

- **역할과 구현을 분리**해서 자유롭게 구현 객체를 조립할 수 있게 설계했다. 덕분에 회원 저장소와 할인 정책을 유연하게 변경할 수 있다.

<br/>

### 3️⃣ 주문 도메인 클래스 다이어그램

![Image](https://github.com/user-attachments/assets/3d85ece8-0f37-4264-b894-6445805763ae)

<br/>

### 4️⃣ 주문 도메인 객체 다이어그램 - 1

![Image](https://github.com/user-attachments/assets/af018574-b98c-4c1b-b9a8-6c2ff3153f00)

- 회원을 메모리에서 조회하고, 정액 할인 정챌(고정 금액)을 지원해도 주문 서비스를 변경하지 않아도 된다.
- 역할들의 협력 관계를 그대로 재사용 할 수 있다.

<br/>

### 5️⃣ 주문 도메인 객체 다이어그램 - 2

![Image](https://github.com/user-attachments/assets/d754bfc1-08f3-42e3-ae73-20d5fb934c0c)

- 회원을 메모리가 아닌 실제 DB에서 조회하고, 정률 할인 정책(주문 금액에 따라 % 할인)을 지원해도 주문 서비스를 변경하지 않아도 된다.
- 협력 관계를 그대로 재사용 할 수 있다.

---

## 6. 주문과 할인 도메인 개발

- **할인 정책 인터페이스 : `discount/discountPolicy`**
- **정액 할인 정책 구현체 : `discount/FixDiscountPolicy`**
  - VIP면 1000원 할인
- **주문 엔티티 : `order/Order`**
- **주문 서비스 인터페이스 : `order/OrderService`**
- **주문 서비스 구현체 : `order/OrderServiceImpl`**
  - 주문 생성 요청이 오면, 회원 정보를 조회하고, 할인 정책을 적용한 다음 주문 객체를 생성해서 반환한다.
  - **메모리 회원 리포지토리와 고정 금액 할인 정책을 구현체로 생성한다.**

---

## 7. 주문과 할인 정책 테스트

### 1️⃣ 주문과 할인 정책 테스트

`test/.../order/OrderServiceTest`
```java
MemberService memberService = new MemberServiceImpl();
OrderService orderService = new OrderServiceImpl();

@Test
void createOrder() {
        Long memberId = 1L;
        Member member = new Member(memberId, "memberA", Grade.VIP);
        memberService.join(member);

        Order order = orderService.createOrder(memberId, "itemA", 10000);
        Assertions.assertThat(order.getDiscountPrice()).isEqualTo(1000);
}
```