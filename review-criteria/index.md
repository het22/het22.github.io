# Code Review Criteria

## 1. 존재할 수 없는 상태는 애초에 만들 수 없도록 타입으로 제약을 준다

상태 타입이 명세에 없는 조합이나 전이를 표현할 수 있으면 지적한다. 주석으로 금지한 조합은 타입으로 막지 않은 것과 같다.

- references: [react-query-use-query-result.md](references/react-query-use-query-result.md)
- changelog: [202610281530.md](changelog/202610281530.md)
