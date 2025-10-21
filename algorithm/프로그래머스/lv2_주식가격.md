# 🧾 오답노트 — 주식가격 (2025.10.21)

> “가격이 떨어지지 않은 기간 계산”
> (Queue / 이중 for문 / Monotonic Stack 비교)

---

## ❌ 나의 시도

```js
while (prices.length) {
  const fp = prices.shift();
  for (const nextPrice of prices) {
    if (fp <= nextPrice) time++;
    else {
      time++;
      break;
    }
  }
}
```

### 🧩 문제점

- `shift()`는 **O(n)** → 반복문과 결합 시 **O(n²~n³)** 로 비효율적.
- 배열이 매번 재정렬되어 불필요한 연산 발생.
- 큐 구조 의도는 좋았지만, JS의 `shift()`는 성능상 느림.

### 💡 교훈

> JS에서 큐를 구현하려면 `shift()` 대신 **head 포인터** 또는 **직접 큐 클래스**를 사용하는 것이 좋다.

---

## ✅ 정답 접근 (이중 for문)

```js
function solution(prices) {
  const answer = [];
  for (let i = 0; i < prices.length; i++) {
    let time = 0;
    for (let j = i + 1; j < prices.length; j++) {
      time++;
      if (prices[i] > prices[j]) break;
    }
    answer.push(time);
  }
  return answer;
}
```

### ⚙️ 특징

- 단순하고 명확한 구조.
- JS 엔진 최적화로 **O(n²)** 도 충분히 빠름.
- 프로그래머스 효율성 테스트 없음 → 완벽 통과.

### 💡 교훈

> 효율성 테스트가 없는 문제에서는 **이중 for문**이 가장 깔끔하고 안전한 선택이다.

---

## ⚡ 최적해 (모노토닉 스택)

```js
function solution(prices) {
  const answer = new Array(prices.length).fill(0);
  const stack = [];

  for (let i = 0; i < prices.length; i++) {
    while (stack.length && prices[stack.at(-1)] > prices[i]) {
      const idx = stack.pop();
      answer[idx] = i - idx;
    }
    stack.push(i);
  }

  while (stack.length) {
    const idx = stack.pop();
    answer[idx] = prices.length - 1 - idx;
  }

  return answer;
}
```

### 🧩 동작 원리

- 스택에는 **가격의 인덱스**를 저장한다.
- 새로운 가격이 들어올 때,

  - 스택 top보다 작으면 가격 하락 발생 → 시간 계산: `i - idx`

- 남은 인덱스는 끝까지 안 떨어진 주식 → `마지막 시점 - 현재 시점`으로 처리.

### ⚙️ 복잡도

- 모든 원소를 **push/pop 한 번씩만 처리** → **O(n)**

### 💡 교훈

> **모노토닉 스택**은 “이전 상태를 기억하다가 조건이 깨질 때만 계산”하는 방식으로, 중복 연산을 줄여 효율성을 극대화한다.

---

## 🧭 전체 비교 요약

| 방식          | 시간 복잡도 | 구조      | JS 효율성    | 비고               |
| ------------- | ----------- | --------- | ------------ | ------------------ |
| while + shift | ⚠️ O(n²~n³) | 큐 유사   | ❌ 느림      | 배열 재배치 발생   |
| 이중 for문    | ✅ O(n²)    | 단순 비교 | 👍 빠름      | 실전 코테용        |
| 모노토닉 스택 | ✅ O(n)     | 스택      | 🚀 최고 효율 | 효율성 문제 대비형 |

---

## 🧠 배운 점

1. `shift()`는 큐 개념과 유사하지만 JS에서는 비효율적이다.
2. 이중 for문은 단순하지만 명확한 해법이다.
3. **모노토닉 스택**은 효율성 문제가 포함된 문제에서 강력한 도구다.

   - 주가, 온도, 높이, 가격 등 **단조성 비교 문제**에 반복적으로 등장한다.

---

## 📌 다음에 비슷한 유형이 나오면

**키워드:** “단조 스택”, “가격/온도 유지 기간”, “O(n) 효율화”

### ✅ 절차 체크리스트

- [ ] 반복적 비교 구조가 O(n²) 이상인지 판단한다.
- [ ] 이전 상태를 저장해 중복 계산을 줄일 수 있는지 확인한다.
- [ ] 스택/큐 구조로 최적화 가능한지 탐색한다.
- [ ] 효율성 테스트 존재 여부에 따라 전략을 선택한다.

---

## 🔁 복습 체크

- [ ] 다시 풀기 (1회차)
- [ ] 다시 풀기 (2회차)
- [ ] 설명 없이 구현 가능 ✅
