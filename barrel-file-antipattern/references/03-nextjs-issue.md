# [Next.js Issue #12557] TypeScript 배럴 파일에서 트리쉐이킹이 작동하지 않음

> 원문: [Tree shaking doesn't work with Typescript barrel files · Issue #12557](https://github.com/vercel/next.js/issues/12557)
> 작성자: majelbstoat
> 게시일: 2020년 5월 6일
> 라벨: TypeScript, Webpack, good first issue
> 상태: Closed (해결됨)

---

## 버그 리포트

### 버그 설명

배럴 파일을 사용하여 단일 위치에서 컴포넌트를 re-export할 때, 트리쉐이킹이 올바르게 작동하지 않음.

### 재현 방법

Next 9.3.6 기준. 컴포넌트 구조:

```
components/
  Header/
    Header.tsx
  Sidebar/
    Sidebar.tsx
  index.ts
```

`index.ts` 배럴 파일:

```ts
export * from './Header/Header.tsx'
export * from './Sidebar/Sidebar.tsx'
// ...
```

`_app.tsx`에서 사용:

```ts
import { Header, Sidebar } from "../components"
```

약 100개의 컴포넌트가 정의되어 있고, `_app.tsx`에서는 몇 개만 사용. 하지만 번들을 분석해 보면, **모든 컴포넌트가 포함된** 매우 큰 공유 청크가 존재.

**직접 import로 변경하면:**

```ts
import { Header } from "../components/Header/Header"
import { Sidebar } from "../components/Sidebar/Sidebar"
```

공통 번들과 앱 페이지 크기가 **극적으로 감소**.

### 기대하는 동작

어떤 import 전략을 사용하든 앱 페이지 크기가 동일해야 함.

---

## 주요 커뮤니티 댓글 요약

이 이슈는 151개 이상의 👍 반응을 받았으며, Next.js 사용자들 사이에서 널리 공감을 얻은 문제였음.

### 핵심 논의 포인트

**1. `sideEffects: false`로는 충분하지 않다**

여러 사용자가 `package.json`에 `sideEffects: false`를 추가하면 해결될 거라고 제안했으나, 이것만으로는 완전히 해결되지 않음. Webpack이 트리쉐이킹을 수행하더라도, 배럴 파일에서 모든 모듈을 먼저 파싱하고 분석하는 비용이 여전히 발생.

**2. `_app.tsx`에서의 특수한 문제**

`_app.tsx`는 모든 페이지에서 공유되는 파일이기 때문에, 여기서 배럴 파일을 통해 import하면 **모든 페이지의 번들 크기**가 증가. 코드 분할(code splitting)의 이점을 크게 상쇄시킴.

**3. 내부 컴포넌트 vs 외부 라이브러리**

외부 npm 패키지뿐만 아니라, 프로젝트 내부의 자체 컴포넌트를 배럴 파일로 관리하는 경우에도 동일하게 발생. 외부 라이브러리는 `optimizePackageImports`로 대응할 수 있지만, 내부 코드의 배럴 파일은 개발자가 직접 해결해야 함.

**4. 워크어라운드: 직접 import**

배럴 파일 대신 소스 파일에서 직접 import하는 것이 유일한 확실한 해결책. 이 방식으로 바꾼 사용자는 번들 크기가 크게 줄었다고 보고.

### 해결

이 이슈는 이후 Next.js 13.5에서 도입된 `optimizePackageImports` 기능의 직접적인 동기가 되었음. 이 기능은 컴파일러 수준에서 배럴 파일 import를 자동으로 직접 import로 변환하여, 개발자가 import 경로를 수동으로 변경하지 않고도 동일한 성능 이점을 얻을 수 있게 해줌.

다만 이 솔루션은 **외부 패키지**에 대한 것이며, 프로젝트 내부의 배럴 파일에 대해서는 여전히 직접 import 패턴을 사용하거나 배럴 파일 자체를 제거하는 것이 권장됨.
