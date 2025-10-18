# 🧾 오답노트 — 베스트앨범 (2025.10.18)

> “장르별 총 재생 수를 기준으로 정렬한 뒤, 각 장르에서 재생 수가 가장 많은 최대 두 곡을 뽑는다.”

---

## ❌ 나의 시도

```js
for (let i = 0; i < genres.length; i++) {
  if (genresPlaysSum[genres[i]]) {
    genresPlaysSum[genres[i]] += plays[i];
    genresPlay[genres[i]].push(i);
  } else {
    genresPlaysSum[genres[i]] = plays[i];
    genresPlay[genres[i]] = [i];
  }
}
```

---

## ⚙️ 문제 의도

> 장르별 총 재생 수를 기준으로 내림차순 정렬하고, 각 장르에서 재생 수가 많은 곡을 최대 두 개 선택한다. (동점이면 인덱스가 작은 곡 우선)

---

## 💣 내가 놓친 부분

| 구분 | 문제점                      | 설명                                                                                            |
| ---- | --------------------------- | ----------------------------------------------------------------------------------------------- |
| ①    | **정렬 비교 함수의 반환값** | `sort()`는 true/false가 아니라 **음수/0/양수**를 반환해야 함. `plays[b] - plays[a]` 형태로 작성 |
| ②    | **가독성/중복**             | 장르 내 상위 2곡을 뽑는 로직을 **함수로 분리**해 흐름을 단순화                                  |
| ③    | **정렬 중복 수행**          | 장르별 내부 정렬은 불가피하지만, 전체 인덱스 한 번 정렬로도 해결 가능한 대안 존재               |
| ④    | **원본 변형 위험**          | 정렬 시 원본 배열 보호를 원하면 `slice()` 후 정렬 (선택 사항)                                   |

---

## ✅ 정답 접근

### 핵심 로직

- **장르별 총합 집계 → 장르 총합 내림차순 정렬 → 장르별 상위 2곡 추출**
- **정렬 기준 우선순위:** 장르 총합 ↓ → 곡 재생수 ↓ → 인덱스 ↑

---

## 🧩 정답 코드 (가독성 우선)

```js
function solution(genres, plays) {
  const genresPlaysSum = {};
  const genresPlay = {};

  // 1) 집계 & 그룹화
  for (let i = 0; i < genres.length; i++) {
    const g = genres[i];
    const p = plays[i];
    if (genresPlaysSum[g] !== undefined) {
      genresPlaysSum[g] += p;
      genresPlay[g].push(i);
    } else {
      genresPlaysSum[g] = p;
      genresPlay[g] = [i];
    }
  }

  // 2) 장르 정렬 (총합 내림차순)
  const sortedGenres = Object.keys(genresPlaysSum).sort(
    (a, b) => genresPlaysSum[b] - genresPlaysSum[a]
  );

  // 3) 장르 내 상위 2곡 선택
  const pickTop2 = (indices) => {
    return indices
      .slice() // 원본 보호 (선택)
      .sort((a, b) => {
        if (plays[b] === plays[a]) return a - b; // 인덱스 오름차순
        return plays[b] - plays[a]; // 재생수 내림차순
      })
      .slice(0, 2);
  };

  // 4) 결과 조합
  const answer = [];
  for (const g of sortedGenres) {
    answer.push(...pickTop2(genresPlay[g]));
  }
  return answer;
}
```

---

## 🥇 참고: 한 번 정렬로 끝내는 압축형

> 가독성은 떨어지지만, 전체 인덱스를 한 번에 정렬해서 필터링하는 방식.

```js
function solution(genres, plays) {
  const total = {};
  for (let i = 0; i < genres.length; i++) {
    total[genres[i]] = (total[genres[i]] || 0) + plays[i];
  }

  const indices = Array.from({ length: genres.length }, (_, i) => i).sort(
    (a, b) =>
      total[genres[b]] - total[genres[a]] || plays[b] - plays[a] || a - b
  );

  const seen = new Map();
  const answer = [];

  for (const i of indices) {
    const g = genres[i];
    const count = seen.get(g) || 0;
    if (count < 2) {
      answer.push(i);
      seen.set(g, count + 1);
    }
  }
  return answer;
}
```

---

## 🧠 배운 점

1. `sort()` 비교 함수는 **값이 아닌 차이**를 반환한다.
2. **그룹화 → 정렬 → 부분 선택**은 코테 전형 패턴.
3. **정렬 기준이 다중**인 경우 조건을 순차 적용하거나 `||` 단축 평가를 사용.
4. 유지보수를 위해 **핵심 서브로직(상위 N 선택)**은 함수로 분리.

---

## 📌 다음에 비슷한 유형이 나오면

**키워드:** “그룹별 상위 N”, “다중 정렬 기준”, “집계 후 선택”

### ✅ 절차 체크리스트

- [ ] 집계(Map/Object)
- [ ] 그룹화 배열 구축
- [ ] 그룹 정렬 기준 확정
- [ ] 그룹 내 정렬 기준 확정
- [ ] 상위 N 슬라이싱 및 합치기

---

## 🔁 복습 체크

- [ ] 다시 풀기 (1회차)
- [ ] 다시 풀기 (2회차)
- [ ] 설명 없이 구현 가능 ✅
