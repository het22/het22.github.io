---
criteria: [1, 2]
type: case-study
---

# 스텝 UI 상태 머신 리뷰: 불가능한 단계와 흩어진 진행 순서

> 플로우를 이루고 있는 노드와 간선에 대한 정의가 명확하지 않고 오픈되어있어서, 플로우를 따로 정의하는 이점을 누리지 못하고 있어요. (...) 선형으로 정의된 기본 플로우는 눈에 보이지만, 수정 플로우를 포함한 전체 플로우는 호출부를 보지 않고서 이 파일만으로 그려낼 수 없어요.

스텝 UI에 사용할 상태 머신 코드를 리뷰하는 중에 지적한 사안이다.
아래는 AS-IS 예시 코드다.

```ts
interface State {
  step:
    | 'shipping'  // 배송지 입력
    | 'payment'   // 결제정보 입력
    | 'review'    // 입력 확인
  isEditing?: boolean   // 수정 모드 진입 여부
}

type Action =
  | { type: 'GO', step: State['step'] }         // 지정한 단계로 가기
  | { type: 'ENTER_EDIT', step: State['step'] } // 입력 수정하러 가기
  | { type: 'EXIT_EDIT' }                       // 수정 마치고 돌아가기

function reducer(state: State, action: Action): State {
  switch (action.type) {
    case 'GO':
      return { step: action.step }
    case 'ENTER_EDIT':
      return { step: action.step, isEditing: true }
    case 'EXIT_EDIT':
      return { step: 'review' }
  }
}
```

작성자는 스텝의 상태와 액션 타입을 지나치게 유연하게 정의했고, 그 결과 상태 머신은 아무런 제약 없이 요구사항에 없는 상태와 상태 전이를 표현할 수 있는 상황이었다.

유연한 타입은 컴파일 에러를 일으키지 않을 뿐, 요구사항을 정확하게 표현하지 못해 여러 가지 부작용이 발생한다.

대표적인 부작용은 타입이 표현하지 못한 요구사항들이 해당 타입의 사용처로 흘러들어간다는 점이다.
'배송지 입력 단계를 마치면 결제정보 입력 단계로 이동한다.'라는 요구사항이 있다면, 배송지 입력 단계 컴포넌트에서 `dispatch({ type: 'GO', step: 'payment' })`를 호출할 것이다. 마찬가지로 결제정보 입력 단계 컴포넌트에서는 `dispatch({ type: 'GO', step: 'review' })`를 호출할 것이다. 이렇게 되면 단계의 진행 순서가 여러 컴포넌트에 흩어져 표현되고, 진행 순서를 알기 위해서는 호출부를 전부 읽어야만 한다.

불가능한 상태와 상태 전이를 허용하게 되는 부작용도 발생한다.
수정 모드는 입력 단계(`shipping`, `payment`)에서만 필요한 속성이지만, 입력 확인 단계에서도 해당 속성을 가질 수 있어 요구사항에 없는 단계가 표현 가능한다(e.g. `{ step: 'review', isEditing: true }`). 수정 모드 진입 액션(`ENTER_EDIT`)은 입력 확인 단계에서만 호출되어야 하지만 다른 단계에서도 자유롭게 호출하도록 허용되어 있다.
이런 허용들은 호출부를 모두 읽지 않고서 단계 흐름을 파악하는 것을 어렵게 만들뿐더러, 불가능한 상태가 생기지 않게 막는 방어 책임을 호출부에게 전가하기도 한다.

이런 부작용들을 막기 위해서는 타입이 실제 존재하는 값만을 표현하도록 정의해야하고, 그 자체로 요구사항과 제약을 충분히 내포해야한다.
다음은 TO-BE 코드 예시이다.

```ts
type State =
  | { step: 'shipping', isEditing?: boolean }   // 배송지 입력/수정
  | { step: 'payment', isEditing?: boolean }    // 결제정보 입력/수정
  | { step: 'review' }                          // 입력 확인

type Action =
  | { type: 'NEXT' }                                // 다음 단계로 가기
  | { type: 'PREV' }                                // 이전 단계로 가기
  | { type: 'EDIT', step: 'shipping' | 'payment' }  // 배송지, 결제정보 수정하러 가기

function reducer(state: State, action: Action): State {
  switch (action.type) {
    case 'NEXT':
      if (state.step === 'shipping') return state.isEditing ? { step: 'review' } : { step: 'payment' }
      if (state.step === 'payment') return { step: 'review' }
      return state
    case 'PREV':
      if (state.step === 'shipping') return state.isEditing ? { step: 'review' } : state
      if (state.step === 'payment') return state.isEditing ? { step: 'review' } : { step: 'shipping' }
      return { step: 'payment' }
    case 'EDIT':
      if (state.step === 'review') return { step: action.step, isEditing: true }
      return state
  }
}
```

`State`는 이제 단계별로 가질 수 있는 속성만 가질 수 있어, `{ step: 'review', isEditing: true }`와 같은 불가능한 상태를 만들려고 하면 타입 에러를 일으킨다. 

`Action`은 지나치게 유연한 상태 전이를 허용하던 `GO` 대신, `NEXT`, `PREV`를 사용한다. 다음과 이전 상태가 어디인지는 단계별 컴포넌트에서 알 필요가 없고, 리듀서 한 곳에서 결정한다. `EDIT`은 다른 단계에서 호출하더라도 아무 전이도 일으키지 않도록 리듀서에서 제한했다.

결과적으로 타입과 리듀서만 읽어도 요구사항에서 제시한 단계 흐름을 모두 파악할 수 있게 되었다.
