# 🧾 오답노트 — 더 맵게 (2025.10.19)

> “가장 맵지 않은 두 음식을 섞어 새로운 스코빌 지수를 만든다. 모든 음식의 스코빌 지수가 K 이상이 될 때까지 반복한다.”

---

### ❌ 나의 시도

- 매 스텝마다 `scoville.sort((a,b)=>a-b)`로 최소 2개 뽑아 섞고 재귀 호출.
- `slice`, 재귀, `sort` 반복 → **TLE(시간 초과)** 가능성 높음.

---

### ⚙️ 문제 의도

> “가장 작은 2개를 반복적으로 꺼내서 섞는 연산”을 효율적으로 구현하라.

---

### 💣 내가 놓친 부분

| 구분 | 문제점                | 설명                                                               |
| ---- | --------------------- | ------------------------------------------------------------------ |
| ①    | **정렬 반복 호출**    | `sort()`는 O(n log n)이며 반복 시 전체 O(n² log n) 수준으로 증가   |
| ②    | **shift/splice 사용** | 배열 앞쪽에서 꺼내면 O(n) 비용 발생 → 루프와 결합 시 비효율 극대화 |
| ③    | **재귀 사용**         | JS 재귀 깊이 제한으로 스택오버플로 위험                            |
| ④    | **힙 미활용**         | 최솟값 2개를 반복 추출하는 패턴은 `Min-Heap`이 정답                |

---

### ✅ 정답 접근

#### 핵심 인사이트

- 연산 패턴: “가장 작은 2개를 반복적으로 꺼내서 하나를 넣는다” → **Min-Heap**.
- 힙을 사용하면:

  - `pop`(최솟값): O(log n)
  - `push`: O(log n)
  - 전체: O(n log n)

---

### 🧩 정답 코드 (복붙용)

```js
function solution(scoville, K) {
  class MinHeap {
    constructor(arr = []) {
      this.h = arr.slice();
      this.heapify();
    }
    size() {
      return this.h.length;
    }
    peek() {
      return this.h[0];
    }
    push(x) {
      const h = this.h;
      h.push(x);
      for (let i = h.length - 1; i > 0; ) {
        const p = (i - 1) >> 1;
        if (h[p] <= h[i]) break;
        [h[p], h[i]] = [h[i], h[p]];
        i = p;
      }
    }
    pop() {
      const h = this.h;
      if (!h.length) return undefined;
      const top = h[0],
        last = h.pop();
      if (h.length) {
        h[0] = last;
        this.down(0);
      }
      return top;
    }
    heapify() {
      for (let i = (this.h.length - 2) >> 1; i >= 0; i--) this.down(i);
    }
    down(i) {
      const h = this.h,
        n = h.length;
      while (true) {
        let l = i * 2 + 1,
          r = l + 1,
          s = i;
        if (l < n && h[l] < h[s]) s = l;
        if (r < n && h[r] < h[s]) s = r;
        if (s === i) break;
        [h[i], h[s]] = [h[s], h[i]];
        i = s;
      }
    }
  }

  const heap = new MinHeap(scoville);
  let cnt = 0;
  while (heap.size() >= 2 && heap.peek() < K) {
    const a = heap.pop();
    const b = heap.pop();
    heap.push(a + b * 2);
    cnt++;
  }
  return heap.peek() >= K ? cnt : -1;
}
```

---

### ⚡ 엣지/함정 체크리스트

- [ ] 시작부터 `min >= K`면 0 반환.
- [ ] `size < 2`인데 `peek() < K`면 -1.
- [ ] 큰 K 대비 작은 값만 있을 때 무한루프 없이 종료.
- [ ] 음수 입력이 들어와도 동작하는지 확인.

---

### ⏱️ 시간·공간 복잡도

| 구분                 | 복잡도     | 설명            |
| -------------------- | ---------- | --------------- |
| 힙 초기화(`heapify`) | O(n)       | 초기 구성       |
| 루프                 | O(n log n) | 최대 n-1회 섞기 |
| 공간                 | O(n)       | 힙 저장         |

---

### 💡 JS vs Python 포인트

| 언어       | 방식              | 비고               |
| ---------- | ----------------- | ------------------ |
| Python     | `heapq` 모듈      | 기본 제공, 간단    |
| JavaScript | 직접 MinHeap 구현 | 템플릿 재사용 가능 |

---

### 🧪 빠른 자가 테스트

```js
console.log(solution([1, 2, 3, 9, 10, 12], 7)); // 2
console.log(solution([1, 1, 1], 10)); // -1
console.log(solution([7, 8], 7)); // 0
console.log(solution([0, 0, 3, 9], 7)); // 2
```

---

### 🧠 배운 점

1. “가장 작은 두 개 반복 추출” → **Min-Heap** 떠올리기.
2. `shift()`는 O(n) → 코테에서 금지급 연산.
3. 힙 템플릿 한 번 구현해두면 **디스크 컨트롤러**, **이중우선순위큐** 등 재사용 가능.

---

### 📌 다음에 비슷한 유형이 나오면

**키워드:** “우선순위큐”, “힙”, “최솟값 2개 조합”, “반복 갱신”

### ✅ 절차 체크리스트

- [ ] 힙 구조 필요 여부 판단
- [ ] `pop`, `push` 연산 복잡도 확인
- [ ] 종료 조건(`peek >= K` or size < 2) 명확히 설정
- [ ] 엣지 케이스(빈 배열, 음수 등) 고려

---

### 🔁 복습 체크

- [ ] 다시 풀기 (1회차)
- [ ] 다시 풀기 (2회차)
- [ ] 설명 없이 구현 가능 ✅
