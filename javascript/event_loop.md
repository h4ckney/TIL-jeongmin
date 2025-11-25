[11월 25일]

이벤트 루프

javascript에서는 스택, web API, 콜백큐를 일단 설명한다.

스택: JavaScript가 동작할 때 순차적으로 코드를 읽고 실행한다. 또한 **한 번 Call Stack에 올라온 작업은 끝날 때까지 중단되지 않는다(Run-to-Completion)**.

web API:  **브라우저가 제공하는 기능**이고, JS 엔진은 그걸 사용한다

콜백 큐:  큐 안에는 두 종류(TASK, MICROTASK)가 있다

Task: `setTimeout`, `setInterval`, DOM 이벤트, 사용자 입력 등 **브라우저가 큐에 넣는 일반 비동기 작업**

MicroTask: `Promise.then`, `queueMicrotask`, `MutationObserver`처럼 **즉시성·우선순위가 높은 비동기 작업**

MacroTask: 표준 용어는 아니지만, 일반적으로 **Task Queue에 들어가는 비동기 작업 전체**를 가리키는 비공식 용어이다. 즉, Microtask와 대비시키기 위해 사용되는 표현으로, 의미상 Task와 거의 동일하게 사용된다.

일반적으로 우리는 JavaScript가 코드를 위에서 아래로 한 줄씩 실행한다고 생각하지만, 이는 **절반만 맞다**. Call Stack은 순차적으로 실행되지만, 비동기 작업은 Web API로 이동하여 타이머·이벤트가 처리된 뒤 **Event Loop의 판단에 따라 다시 스택으로 재진입**하기 때문에 실제 실행 순서는 직관과 다를 수 있다.

아래 예제는 이벤트 루프가 실제 실행 순서에 어떤 영향을 주는지 잘 보여준다.

```javascript
console.log("hi");
setTimeout(() => {
  console.log("how are you");
}, 300);
console.log("im fine");
```

실행 과정:

- `console.log("hi")` → 즉시 실행
- `setTimeout` → Web API로 넘어가 타이머 시작
- `console.log("im fine")` → 바로 실행
- 타이머가 끝나면 콜백은 **Task Queue**에 들어감
- Event Loop는 Call Stack이 비는 시점을 기다렸다가 Task Queue의 콜백을 Stack으로 올림

따라서 최종 출력:

```
hi
im fine
how are you
```
