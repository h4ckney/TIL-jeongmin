# 🧾 오답노트 — 게임 맵 최단거리 (2025.10.29)

> “(0,0)에서 (n-1,m-1)까지의 최단 거리를 구하라.”
> (그래프 탐색 / BFS)

---

## ❌ 나의 시도

```js
function solution(maps) {
  let answer = 0;

  const rowCell = maps.length;
  const colCell = maps[0].length;
  const visited = Array.from({ length: rowCell }, () =>
    Array(colCell).fill(false)
  );
  const queue = [[0, 0, 1]];

  visited[0][0] = true;

  while (queue.length) {
    const [row, col, dist] = queue.shift();
    const dr = [1, -1, 0, 0];
    const dc = [0, 0, 1, -1];

    for (let i = 0; i < 4; i++) {
      const nextRow = row + dr[i];
      const nextCol = col + dc[i];

      if (
        nextRow >= 0 &&
        nextCol >= 0 &&
        nextRow < rowCell &&
        nextCol < colCell &&
        maps[nextRow][nextCol] === 1 &&
        !visited[nextRow][nextCol]
      ) {
        visited[nextRow][nextCol] = true;
        queue.push([nextRow, nextCol, dist + 1]);
      }
    }

    if (row === rowCell - 1 && col === colCell - 1) return dist;
  }

  return answer;
}
```

---

## 🧩 문제점

| 구분 | 실수 내용                 | 원인                     | 교정 방법                                                                |
| ---- | ------------------------- | ------------------------ | ------------------------------------------------------------------------ |
| ①    | `while(maps.length)` 사용 | 큐가 아닌 맵 길이로 반복 | `while(queue.length)`로 수정                                             |
| ②    | `maps.shift()` 사용       | 탐색 대상과 맵 혼동      | 탐색용 큐(`queue.shift()`)로 분리                                        |
| ③    | `visited` 1차원 배열      | 2D 탐색 불가             | `Array.from({length: rowCell}, () => Array(colCell).fill(false))`로 생성 |
| ④    | `return answer`           | BFS는 누적 아님          | 도달 불가 시 `return -1`로 수정                                          |

---

## ✅ 정답 접근 (BFS)

```js
function solution(maps) {
  const n = maps.length;
  const m = maps[0].length;
  const visited = Array.from({ length: n }, () => Array(m).fill(false));
  const queue = [[0, 0, 1]];

  const dr = [1, -1, 0, 0];
  const dc = [0, 0, 1, -1];
  visited[0][0] = true;

  while (queue.length) {
    const [r, c, dist] = queue.shift();
    if (r === n - 1 && c === m - 1) return dist;

    for (let i = 0; i < 4; i++) {
      const nr = r + dr[i];
      const nc = c + dc[i];
      if (
        nr >= 0 &&
        nc >= 0 &&
        nr < n &&
        nc < m &&
        maps[nr][nc] === 1 &&
        !visited[nr][nc]
      ) {
        visited[nr][nc] = true;
        queue.push([nr, nc, dist + 1]);
      }
    }
  }

  return -1; // 도달 불가
}
```

---

## ⚙️ 로직 요약

| 단계 | 설명                                                |
| ---- | --------------------------------------------------- |
| 1️⃣   | `(0,0)`부터 시작, 큐에 `[row, col, dist]` 저장      |
| 2️⃣   | `while(queue.length)` 동안 탐색 반복                |
| 3️⃣   | 상·하·좌·우(`dr`, `dc`) 네 방향 확인                |
| 4️⃣   | 이동 가능 + 방문 X → `queue.push([다음칸, dist+1])` |
| 5️⃣   | 도착점 `(n-1, m-1)` 도달 시 `dist` 반환             |
| 6️⃣   | 도달 불가 → `-1` 반환                               |

---

## 🧠 DFS vs BFS 비교

| 비교 항목   | DFS            | BFS                     |
| ----------- | -------------- | ----------------------- |
| 탐색 방식   | 깊이 우선      | 너비 우선               |
| 자료 구조   | 스택(재귀)     | 큐(Queue)               |
| 주요 사용처 | 모든 경우 탐색 | **최단거리 탐색**       |
| 코드 구조   | 간결           | 반복문 + 방문 배열 필요 |
| 예시 문제   | 타겟 넘버      | **게임 맵 최단거리**    |

---

## 💡 배운 점

1. **BFS = 물이 번지는 것처럼, 거리 단위로 확장**하는 탐색.
2. `queue.shift()`는 비효율적일 수 있으므로, 대규모 탐색에서는 deque로 최적화 고려.
3. 방문 관리(`visited`)는 중복 탐색 방지의 핵심.
4. DFS와 BFS는 모두 그래프 탐색이지만 **목표(최단 vs 전체)**가 다르다.

---

## 📌 다음에 비슷한 유형이 나오면

**키워드:** “BFS”, “최단거리”, “Queue”, “2D 탐색”

### ✅ 절차 체크리스트

- [ ] 큐 초기화 (`[[0,0,1]]`)
- [ ] 방문 배열 생성 및 표시
- [ ] 방향 벡터(`dr`, `dc`) 적용
- [ ] 조건문: 범위 + 방문 여부 + 통로 체크
- [ ] 도착 지점(`n-1, m-1`) 확인 후 거리 반환

---

## 🔁 복습 체크

- [ ] 다시 풀기 (1회차)
- [ ] 다시 풀기 (2회차)
- [ ] 설명 없이 구현 가능 ✅
