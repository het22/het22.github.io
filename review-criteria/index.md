# Code Review Criteria

## 1. 타입은 존재할 수 없는 상태를 표현하지 않는다

- 타입이 요구사항에 없는 상태를 표현하고 있지 않은지 확인한다.
- 주석으로 적힌 제약을 타입 정의로 옮길 수 있는지 확인한다.

cases: [step-state-machine.md](cases/step-state-machine.md)

## 2. 요구사항 구현 책임을 호출부에게 전가하지 않는다

- 호출부에서 모듈 내부 규칙을 몰라도 올바르게 사용할 수 있는지 확인한다.
- 내부 로직이 복잡해진다는 이유로 호출부에 구현 책임을 전가하지 않는다.

cases: [step-state-machine.md](cases/step-state-machine.md)
