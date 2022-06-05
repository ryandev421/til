# 도메인 모델

### 도메인?

- 도메인(Domain): 소프트웨어로 해결하고자 하는 문제 영역
- 한 도메인은 하위 도메인으로 나눌 수 있음



### 도메인 모델

- 도메인 모델을 객체로만 모델링 할 수 있는 것은 아님
- 도메인을 이해하는데 도움이 된다면 표현 방식은 상관 없음
  - 관계가 중요 → 그래프 이용
  - 계산 규칙이 중요 → 수학 공식
  - 등등
- 하위 도메인과 모델
  - 카탈로그의 상품과 배송의 상품은 다른 의미를 갖음
    - 카탈로그에서의 상품: 가격, 상세 내용 등
    - 배송에서의 상품: 실제 물건, 재고
  - 모델의 각 구성요소는 특정 도메인으로 한정할 때 비로소 의미가 완전해지기 때문에 각 하위 도메인마다 별도로 모델을 만들어야 한다



### 유비쿼터스 언어

- 도메인에서 사용하는 용어를 코드에 반영해야함
- 코드의 가독성을 높여서 코드를 분석하고 이해하는 시간을 줄일 수 있음
- 최대한 도메인 용어를 사용해서 도메인 규칙을 코드로 작성하게 되므로 버그도 줄 수 있음



# 엔티티와 밸류

### 엔티티(Entity)

- 식별자를 가짐(엔티티마다 고유함)
  - 식별자가 같으면 같은 엔티티라고 판단할 수 있음
  - 식별자 생성방법
    - 특정 규칙에 따라
    - UUID or Nano ID 같은 고유 식별자
    - 값을 직접 입력
    - 일련번호 사용(시퀀스나 DB Auto Increment)

### 값 객체(Value Object)

- 서로 다른 데이터지만 개념적으로 하나의 개념을 말하면 합침

  - 받는 사람 이름, 받는 사람 전화번호 ⇒ 받는 사람

- 반드시 2개 이상의 데이터를 가져야 하는 것은 아님

  ```java
  # example
  public class Money {
  	private int value;
  	
  	public Money(int value) {
  		this.value = value;
  	}
  }
  ```

- Value Type은 불변으로 함

  - 불변으로 하지 않았을 때 생길 수 있는 문제 예시

  ```java
  Money price = new Money(1000);
  OrderLine line = new OrderLine(product, price, 2); -> price=1000 / quantity=2 / ammounts=2000
  price.setValue(2000); -> price=2000 /quantity=2 / amounts=2000
  ```

- 두 밸류 객체를 비교할 때는 모든 속성이 같은지 비교

- 식별자를 위한 밸류 타입을 사용해서 의미를 더 잘 드러나게 할 것



# 도메인 서비스

### 외부 시스템 연동 or 다른 도메인과 연동

- 외부 시스템 또는 다른 도메인과의 연동 기능도 도메인 서비스가 될 수 있음
- 예를 들어 어드민 페이지에서 특정 고객의 주문을 취소할 수 있는 기능이 있다고 했을 때, 주문 도메인에선 주문 취소를 요청한 어드민이 권한이 있는지 확인을 위해 역할 관리 시스템과 연동해야 한다.
```java
# Domain Service에 인터페이스 구현
public interface OrderPermissionChecker {
	boolean hasAdminOrderCacelPermission(String adminId)
}
# Infrastructure Layer에 구현체 구현
public PermissionChecker implements OrderPermissionChecker {
	boolean hasAdminOrderCacelPermission(String adminId) {
		...
	}
}
```