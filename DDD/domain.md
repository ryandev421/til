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