# 🧾 오답노트 — 연속된 부분 수열의 합 (2025.11.25)

> “연속된 부분 수열 중 합이 k가 되는 가장 짧은 구간을 찾아라.”
> (투 포인터 / 슬라이딩 윈도우)

---

## ❌ 나의 시도

```js
function solution(sequence, k) {
  let answer = [0, Infinity];
  let left = 0;
  let right = 0;
  let sum = sequence[0];

  while (right < sequence.length) {
    if (sum === k) {
      if (right - left < answer[1] - answer[0]) {
        answer[0] = left;
        answer[1] = right;
      }
      right++;
      sum += sequence[right];
    } else if (sum < k) {
      right++;
      if (right < sequence.length) sum += sequence[right];
    } else {
      sum -= sequence[left];
      left++;
    }
  }

  return answer;
}
```

---

## 🧩 문제점

| 구분 | 실수 내용 / 위험 구간                                  | 원인                                             | 교정 방법                         |
| ---- | ------------------------------------------------------ | ------------------------------------------------ | --------------------------------- |
| ①    | `right++` 후 `sum += sequence[right]` → 범위 오류 위험 | right가 배열 끝에 닿는 순간 undefined 접근 발생  | `if(right < length)` 가드 추가    |
| ②    | `sum === k` 분기에서 sum-sync 불안정                   | 분기마다 포인터/합 업데이트 순서가 일관되지 않음 | left/right 이동 시 역산 규칙 적용 |
| ③    | sum과 [left, right] 구간 불변조건 깨질 수 있음         | 포인터 이동 타이밍과 합 조정이 매칭되지 않음     | "포인터 이동 → 합 역산” 순서 유지 |
| ④    | `Infinity` 초기값 가독성 낮음                          | 길이 비교 로직은 맞지만 의미 전달이 모호함       | 플래그(found) 기반 갱신 방식 사용 |

---

## 🔧 내가 이해한 핵심 오류

### **1) 포인터 이동과 합(sum) 업데이트의 불일치**

투 포인터에서 가장 중요한 조건:

> **sum은 항상 `[left, right]` 구간의 합이어야 한다.**

right를 늘리면 → 새로운 값 포함 → sum에 더해줘야 하고
left를 늘리면 → 기존 시작 값 빠짐 → sum에서 빼줘야 함.

이걸 **역산(inverse update)** 로 이해하면 구조가 매우 명확해짐.

---

## ✅ 정답 접근 (투 포인터 / 슬라이딩 윈도우)

```js
function solution(sequence, k) {
  let left = 0;
  let sum = 0;
  let answer = [0, sequence.length - 1];
  let found = false;

  for (let right = 0; right < sequence.length; right++) {
    sum += sequence[right];

    while (sum > k && left <= right) {
      sum -= sequence[left];
      left++;
    }

    if (sum === k) {
      if (!found || right - left < answer[1] - answer[0]) {
        answer = [left, right];
        found = true;
      }
    }
  }

  return answer;
}
```

---

## ⚙️ 로직 요약

| 단계 | 설명                                            |
| ---- | ----------------------------------------------- |
| 1️⃣   | right를 0 → n-1까지 이동하며 sum에 값 추가      |
| 2️⃣   | sum이 k보다 크면 left를 이동해 구간 줄이기      |
| 3️⃣   | sum이 k면 최적 구간인지 비교 후 갱신            |
| 4️⃣   | 포인터 이동할 때마다 sum을 역산으로 맞춤        |
| 5️⃣   | 전체 복잡도 O(n), 각 포인터는 최대 한 번만 전진 |

---

## 🧠 투 포인터 핵심 개념 (불변조건)

> **sum = sequence[left] ~ sequence[right] 구간의 합**

즉,

- right++ 하기 전에 → 추가될 값 sum에 더하기
- left++ 하기 전에 → 빠질 값 sum에서 빼기

이 순서를 지키면 sum과 포인터 구간이 항상 정확하게 일치한다.

---

## 💡 배운 점

1. 투포인터는 "포인터 이동 = 합 역산"이라는 개념으로 이해하면 단순해짐.
2. right가 끝 근처일 때 반드시 범위 체크 필요.
3. 최단 구간 문제는 "먼저 발견된 최단 구간이 가장 왼쪽"이라는 투포인터 특성 활용.
4. BFS/DFS와 달리 투포인터는 **경계 관리 + 불변조건**이 핵심.

---

## 🔁 복습 체크

- [ ] 다시 풀기 (1회차)
- [ ] 다시 풀기 (2회차)
- [ ] 설명 없이 구현 가능하기 (투 포인터 자동화)
