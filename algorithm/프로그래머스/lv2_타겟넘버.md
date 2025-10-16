# 💡 타겟 넘버 (Programmers / Lv2)

---

### ⚙️ 문제 설명

> 주어진 숫자 배열 `numbers`에서 + 또는 -를 붙여 `target` 값을 만드는 **모든 경우의 수**를 구하는 문제.

---

### ✅ 나의 시도 (재귀 DFS)

```js
function solution(numbers, target) {
  let count = 0;
  const n = numbers.length;

  const dfs = (idx, acc) => {
    if (idx === n) {
      if (acc === target) count++;
      return;
    }
    dfs(idx + 1, acc + numbers[idx]);
    dfs(idx + 1, acc - numbers[idx]);
  };

  dfs(0, 0);
  return count;
}

🧩 로직 요약

단계	설명
1	인덱스 0에서 시작 (idx=0, acc=0)
2	각 단계에서 +, - 두 가지 분기로 재귀 호출
3	idx === n일 때 누적합이 target이면 count++
4	모든 경로 탐색 완료 후 count 반환

⚙️ 특징
	•	JS 엔진의 콜스택을 자동 사용
	•	함수 호출이 내부적으로 스택을 대신함
	•	코드가 간결하지만 깊은 트리에서는 Stack Overflow 위험 존재

⸻

✅ 정답 접근 (스택 DFS)

function solution(numbers, target) {
  let count = 0;
  const stack = [{ idx: 0, acc: 0 }];

  while (stack.length > 0) {
    const { idx, acc } = stack.pop();

    if (idx === numbers.length) {
      if (acc === target) count++;
      continue;
    }

    stack.push({ idx: idx + 1, acc: acc + numbers[idx] });
    stack.push({ idx: idx + 1, acc: acc - numbers[idx] });
  }

  return count;
}

🧩 로직 요약

단계	설명
1	스택에 초기 상태 { idx: 0, acc: 0 } 푸시
2	스택에서 상태를 꺼내(pop) 두 가지 분기를 다시 푸시
3	idx === n이면 target 비교 후 count++
4	스택이 빌 때까지 반복

⚙️ 특징
	•	재귀 대신 직접 스택을 조작
	•	탐색 순서를 명시적으로 제어 가능
	•	약간 느리지만 깊은 그래프 탐색 시 안정적

⸻

📊 비교 요약

구분	재귀 DFS	스택 DFS
구조	함수 호출 기반	명시적 배열 스택
속도	빠름 (엔진 최적화 있음)	약간 느림 (힙 객체 생성)
코드 길이	짧고 간결	다소 김
디버깅	스택 내부 보기 어려움	탐색 순서 제어 쉬움
위험	깊은 재귀 시 Stack Overflow	메모리 사용량 ↑


⸻

🧠 배운 점
	1.	두 방식은 논리적으로 동일한 탐색 구조를 가진다.
	2.	재귀는 내부적으로 스택을 사용하므로 “스택 DFS”로 바꾸면 흐름이 보인다.
	3.	탐색 순서, 상태 저장, 백트래킹 구조를 명확히 시각화할 수 있다.
	4.	깊은 트리 탐색 시 재귀보다 스택이 안전하다.

⸻

✍️ 정민’s 회고

	•	continue 조건을 명확히 분리하지 않으면 불필요한 push 발생
	•	스택 DFS가 느린 이유는 객체 생성 오버헤드 때문
	•	스택 DFS는 디버깅/시각화용 학습에는 최적

⸻
```
