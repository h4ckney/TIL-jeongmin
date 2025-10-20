# React 19.2 업데이트 & 실무 적용 정리 (TIL-jeongmin)

> 🧠 **목표:** React 19.2의 주요 변경점을 단순 요약이 아니라, 실제 **실무 적용 관점**에서 분석하고 정리. 새로운 API의 동작 원리, 장단점, 주의점을 모두 포함.

---

## 🧩 1. 새로운 React 기능

### ⚙️ `<Activity />`

React 19.2에서 추가된 `<Activity />`는 **UI 가시성(visible/hidden)**에 따라 **Effect와 업데이트의 우선순위**를 제어할 수 있는 새로운 컨테이너 컴포넌트입니다.

```jsx
// Before
{
  isVisible && <Page />;
}

// After
<Activity mode={isVisible ? "visible" : "hidden"}>
  <Page />
</Activity>;
```

#### ▪️ 작동 방식

- **`visible`** → 자식이 렌더링되고, Effect가 마운트됨.
- **`hidden`** → 자식이 숨겨지고, Effect가 언마운트됨.
  React는 가시 영역의 작업이 모두 끝난 뒤에 **지연(defer)** 처리.

#### ✅ 장점

- 보이지 않는 페이지(탭/모달)를 **미리 렌더링**해 전환 성능 향상.
- 현재 보이는 화면의 렌더링 성능에 **우선순위 부여**.
- 백그라운드에서 **데이터 프리패치 및 상태 유지** 가능.

#### ⚠️ 주의점

- `hidden` 상태에서 **Effect 언마운트** → 재구독 비용 발생 가능.
- 비동기 요청이나 구독 해제 로직을 꼼꼼히 관리해야 함.

#### 💡 실무 활용 팁

- 사용자 전환이 예상되는 UI(탭, 상세 페이지 등)에 적용.
- `hidden` 모드 시 상태는 상위 컴포넌트로 lift-up.

---

### ⚙️ `useEffectEvent`

`useEffectEvent`는 Effect 내부에서 발생하는 **이벤트 콜백**을 **deps 의존성에서 분리**하기 위한 새로운 Hook입니다.

#### 🧱 문제 예시

```jsx
function ChatRoom({ roomId, theme }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.on("connected", () => showNotification("Connected!", theme));
    connection.connect();
    return () => connection.disconnect();
  }, [roomId, theme]); // ❌ theme 변경 시 재연결 발생
}
```

#### ✅ 개선 코드

```jsx
function ChatRoom({ roomId, theme }) {
  const onConnected = useEffectEvent(() => {
    showNotification("Connected!", theme);
  });

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.on("connected", onConnected);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]); // ✅ 불필요한 재실행 제거
}
```

#### 💬 핵심 요약

- Effect Event는 항상 **최신 props/state**를 본다.
- deps 배열에 넣지 않는다.
- 같은 컴포넌트 or 훅 내부에서만 선언 가능.

#### 💡 사용 시점

- **사용자 이벤트(onClick)** X
- **Effect 내부 이벤트(WebSocket, Timer, 메시지 수신 등)** O

#### ⚠️ ESLint 플러그인

`eslint-plugin-react-hooks@latest`가 올바른 사용을 자동 감지.

---

### ⚙️ `cacheSignal`

React Server Components 전용 API.

```jsx
import { cache, cacheSignal } from "react";
const dedupedFetch = cache(fetch);

async function Component() {
  await dedupedFetch(url, { signal: cacheSignal() });
}
```

#### 🧠 핵심 개념

- `cacheSignal`은 `cache()`의 생명주기를 추적하여 렌더링 성공/중단/실패 시점을 감지.
- 필요 없어지면 해당 네트워크 요청을 **Abort**하거나 리소스를 **정리** 가능.

#### ⚠️ 유의사항

- **RSC(Server Components)** 전용.
- 클라이언트 컴포넌트에서는 동작하지 않음.

#### 💡 장점

- 서버 렌더링 취소 시 **리소스 낭비 방지**.
- 스트리밍 렌더링 최적화.

---

### ⚙️ React Performance Tracks (DevTools)

React 19.2부터 Chrome DevTools에 **React 전용 트랙(Scheduler, Components)**이 추가됨.

#### 🪄 Scheduler Track

- React가 작업을 어떤 **우선순위(Blocking, Transition)**로 수행하는지 표시.
- 특정 업데이트가 왜 지연되었는지 시각적으로 확인 가능.

#### 🧩 Components Track

- 컴포넌트 렌더링 및 Effect 실행 시간 표시.
- "Mount", "Blocked" 등 상태 레이블 제공.

#### 💡 실무 활용

- Transition과 Blocking 업데이트가 겹칠 때, 성능 병목 구간 식별.
- 복잡한 렌더링의 **작업 분할(concurrent scheduling)**을 분석.

---

## ⚙️ 2. React DOM: 부분 사전 렌더링 (Partial Pre‑Rendering)

React 19.2의 핵심 기능 중 하나로, 앱의 **정적 부분을 미리 렌더링**하고 이후 **동적 부분을 재개(resume)**하는 구조.

#### 🧱 prerender 단계

```jsx
const { prelude, postponed } = await prerender(<App />, {
  signal: controller.signal,
});
await savePostponedState(postponed);
```

→ **prelude shell**을 클라이언트/CDN으로 먼저 전송.

#### 🔁 resume 단계

```jsx
const postponed = await getPostponedState(request);
const stream = await resume(<App />, postponed);
```

→ SSR 스트림을 이어받아 렌더링 재개.

#### 🧱 SSG용 prerender + resume

```jsx
const { prelude } = await resumeAndPrerender(<App />, postponed);
```

#### 💡 장점

- 초기 로딩 시간 단축.
- SEO 및 CDN 캐싱에 유리.
- React 19.2부터 SSR/SSG 경계가 더욱 유연.

#### ⚠️ 주의점

- **postponed state 저장 및 버전 관리** 필요.
- CDN 캐시 무효화 전략 필수.

---

## 🧭 3. 총평

| 항목                  | 핵심 변화                 | 실무 의의               |
| --------------------- | ------------------------- | ----------------------- |
| `<Activity />`        | 비가시 영역 업데이트 제어 | 탭/모달 성능 최적화     |
| `useEffectEvent`      | Effect 이벤트 분리        | 불필요한 재실행 방지    |
| `cacheSignal`         | RSC 캐시 수명 추적        | 서버 리소스 관리 효율화 |
| Performance Tracks    | DevTools 시각화 개선      | 병목 구간 추적 가능     |
| Partial Pre‑Rendering | SSR/SSG 융합 렌더링       | 초기 LCP·SEO 개선       |

---

## 📚 참고 자료

- [React Docs](https://react.dev/blog/2025/10/01/react-19-2)
- [Performance Tracks](https://react.dev/reference/dev-tools/react-performance-tracks)
- [cacheSignal](https://react.dev/reference/react/cacheSignal)
- [Partial Pre-Rendering](https://react.dev/reference/react-dom/server/resume)
