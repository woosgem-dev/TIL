# 배럴 파일 & TypeScript 컨벤션

> 2025-02-15 공부 정리
> 주제: 배럴 파일 안티패턴, import 전략, TypeScript 컨벤션

---

## 1. 배럴 파일이란

여러 모듈을 하나의 `index.ts`에서 re-export하여 import 경로를 단순화하는 JavaScript 관행.

```ts
// components/index.ts (배럴 파일)
export { Button } from './Button/Button'
export { Modal } from './Modal/Modal'

// 소비자
import { Button, Modal } from '@/components'
```

---

## 2. 왜 안티패턴인가

### 2-1. 빌드 성능 (검증된 데이터)

**Atlassian Jira (90,000개 파일 대상)**

| 항목 | 개선 |
|------|------|
| TypeScript 하이라이팅 | 30%+ 빨라짐 |
| 로컬 유닛 테스트 | 평균 50% 빨라짐, 일부 패키지 10배 |
| CI 유닛 테스트 | 1,600개 → 200개 (88% 감소) |
| CI 통합 테스트 | 130개 → 20개 (85% 감소) |
| 전체 빌드 시간 | **75% 단축** |

> 상세: [references/01-atlassian.md](./references/01-atlassian.md)

**Next.js / Vercel 벤치마크 (M2 MacBook Air)**

| 라이브러리 | 이전 | 이후 | 차이 |
|------------|------|------|------|
| @mui/material | 7.1초 (2,225 모듈) | 2.9초 (735 모듈) | -4.2초 |
| @material-ui/icons | 10.2초 (11,738 모듈) | 2.9초 (632 모듈) | -7.3초 |
| lucide-react | 5.8초 (1,583 모듈) | 3.0초 (333 모듈) | -2.8초 |

프로덕션 빌드 28% 빨라짐, 서버리스 콜드부트 40% 빨라짐.

> 상세: [references/02-vercel-nextjs.md](./references/02-vercel-nextjs.md)

### 2-2. 트리쉐이킹 파괴

- Next.js Issue #12557: 100개 컴포넌트 배럴에서 2개만 써도 전부 번들에 포함
- 직접 import로 바꾸자 번들 크기가 확 줄었음
- `sideEffects: false`만으로는 해결 불충분 (모든 모듈 파싱 비용은 그대로)

> 상세: [references/03-nextjs-issue.md](./references/03-nextjs-issue.md)

### 2-3. 근본 원인

TypeScript 컴파일러와 Jest는 배럴 파일 하나를 resolve할 때 **그 안의 모든 모듈을 로드**해야 한다. Button 하나만 import해도 배럴에 50개 컴포넌트가 있으면 50개를 전부 파싱. 배럴이 배럴을 참조하는 **중첩 배럴**은 이 비용이 눈덩이처럼 불어남.

```
import { Button } from '@/components'
  → components/index.ts (30개 re-export 전부 로드)
    → Button/index.ts (또 로드)
      → Button.tsx (최종 도착)
```

한 번 import하는데 파일 3개를 거침. 컴포넌트 30개면 61개 파일 처리.

### 2-4. 순환 참조 위험

배럴 파일끼리 서로 import하면 circular dependency가 생기기 쉽고 런타임에 `undefined` 에러가 나는데 원인 추적이 까다롭다.

### 2-5. 유지보수 비용

컴포넌트 추가/삭제할 때마다 `index.ts`에 export 라인을 넣고 빼야 함. 잊으면 데드 export가 남거나 새 컴포넌트가 import 안 되는 상황 발생.

---

## 3. 디자인 시스템은 왜 괜찮은가

디자인 시스템 라이브러리(`import { Button } from '@some-ds/components'`)가 배럴을 써도 되는 이유:

1. **빌드를 미리 해놓음**: 소비자의 TypeScript 컴파일러가 소스를 전부 파싱하는 게 아니라, 빌드된 `.js`와 `.d.ts`만 읽음
2. **번들러 최적화가 별도로 들어감**: `sideEffects: false`, `exports` 필드, per-component 빌드 출력
3. **public API 계약**: 내부 구조를 숨기고 버전 간 경로 변경에도 소비자 코드가 안 깨지게 하는 역할

**핵심**: resolve 비용을 누가 언제 부담하느냐의 차이. 라이브러리는 배포 시점에 이미 resolve가 끝나 있고, 앱 내부 배럴은 개발자가 저장 누를 때마다 매번 그 비용을 냄.

---

## 4. AI 에이전트와 배럴

### 에이전트 성능에 미치는 영향

공식 테스트 결과는 **존재하지 않음**. Anthropic, OpenAI 등 AI 회사에서 "배럴 파일 vs 직접 import"에 대한 에이전트 성능 비교를 발표한 적 없음.

**grep 기반이면 배럴 자체가 큰 장애물은 아님:**
- `grep -r "export.*Button" --include="*.tsx"` → 배럴 유무와 무관하게 최종 소스 발견 가능

**배럴이 에이전트한테 실제로 귀찮은 경우:**
- import를 역추적할 때 순차적으로 파일을 열어야 함 (tool call + 토큰 소모)
- 파일명이 전부 `index.ts`면 디렉토리 목록만으로 역할 파악 불가 → 각각 열어봐야 함

**결론**: 배럴을 피해야 하는 근거는 에이전트보다 **빌드 성능과 트리쉐이킹** 쪽이 훨씬 더 확실하다.

---

## 5. 권장 아키텍처

### 5-1. 앱 내부 코드

```
components/
  Button/
    Button.tsx          ← named export: export function Button()
    Button.styles.ts
    Button.test.tsx
  Modal/
    Modal.tsx
    Modal.styles.ts
```

- **index.ts 배럴 없음**
- 직접 경로 import: `import { Button } from '@/components/Button/Button'`
- tsconfig paths: `"@/*": ["./src/*"]`
- auto-import에서 index.ts 제외: `autoImportFileExcludePatterns: ["**/index.ts"]`

### 5-2. 모노레포 패키지 경계

```json
{
  "name": "@pkg/ui",
  "exports": {
    "./Button": "./src/Button/Button.tsx",
    "./Modal": "./src/Modal/Modal.tsx"
  }
}
```

- subpath exports로 진입점을 쪼개서 제공
- 소비자: `import { Button } from '@pkg/ui/Button'`
- 단일 엔트리 배럴(`".": "./src/index.ts"`)은 쓰지 않음

### 5-3. 외부 라이브러리

- Next.js `optimizePackageImports`에 맡기기 (컴파일러 수준에서 배럴 → 직접 import 자동 변환)
- 패키지 구조가 허용하면 직접 import 사용

---

## 6. 컴포넌트 디렉토리에 index.ts 쓰기 vs ComponentName.tsx 쓰기

```
# index.ts 패턴 — 파일 목록만 보면 뭐가 뭔지 모름
components/Button/index.ts
components/ListItem/index.ts
components/Modal/index.ts

# 명시적 파일명 — 바로 파악 가능
components/Button/Button.tsx
components/ListItem/ListItem.tsx
components/Modal/Modal.tsx
```

- 에디터 탭에서 `index.ts`만 잔뜩 열리는 문제 해소
- 스택 트레이스에서 파일명 바로 식별 가능
- AI 에이전트가 파일명에서 바로 역할 파악 가능
- `Button/Button`이 중복스러워 보이지만, auto-import가 다 잡아주니 직접 타이핑할 일 거의 없음

---

## 7. tsconfig paths로 경로 정리

```json
{
  "compilerOptions": {
    "paths": {
      "@/*": ["./src/*"]
    }
  }
}
```

`import { Button } from '@/components/Button/Button'` — 경로가 파일 시스템과 1:1 대응되어 **가장 예측 가능한 구조**. AI 에이전트도 경로만 보고 바로 파일을 찾을 수 있음.

---

## 8. Export 스타일: Named vs Default

**항상 named export 사용:**

```ts
// CORRECT
export function Button() { ... }

// WRONG
export default function Button() { ... }
```

왜 named export인가:
- 소비자가 import 이름을 마음대로 바꾸는 것 방지 (코드베이스 전체 일관성)
- auto-import 정확도 향상
- 개별 export 단위 트리쉐이킹
- grep/AST로 사용처 추적 용이

### React.lazy 워크어라운드

```ts
const Page = React.lazy(() =>
  import('./Page').then(module => ({ default: module.Page }))
)

// 또는 헬퍼 함수
function lazyNamed<T extends Record<string, any>>(
  factory: () => Promise<T>,
  name: keyof T
) {
  return React.lazy(() =>
    factory().then(module => ({ default: module[name] }))
  )
}
```

### Vue 예외

`.vue` SFC는 구조적으로 default export만 가능 (프레임워크 제약). 허용되는 이유:

- 파일 하나 = 컴포넌트 하나 → 파일명이 곧 컴포넌트명
- `unplugin-vue-components` 등이 파일명 기반으로 이름 강제
- composables, utils 등 `.ts` 파일은 named export 유지

```ts
// CORRECT — 파일명과 이름 일치
import Button from '@/components/Button/Button.vue'

// WRONG — 파일명과 다른 이름
import Btn from '@/components/Button/Button.vue'
```

---

## 9. 함수 선언: function 키워드 vs 화살표

**최상위 선언에는 function 키워드:**

```ts
// CORRECT
export function Button() { ... }
export function useAuth() { ... }

// WRONG
export const Button = () => { ... }
```

왜 function 키워드인가:
- 호이스팅 → 파일 내 순서 유연
- 스택 트레이스에 항상 함수명 표시
- grep 친화적: `function Button`은 명확한 패턴
- "이건 이름 있는 재사용 단위다"라는 신호가 분명함

**화살표 함수는 콜백과 인라인에:**

```ts
const items = list.map((item) => transform(item))
useEffect(() => { ... }, [])
```

---

## 10. interface vs type

### 에이전트가 interface를 기본으로 쓰는 이유

학습 데이터에 interface가 압도적으로 많아서. TypeScript 공식 문서가 오랫동안 "잘 모르겠으면 interface 써라"고 권장했고, 대부분 오픈소스가 그 관습을 따름.

### 기능적 차이

```ts
// interface만 가능
interface A { x: string }
interface A { y: number }  // declaration merging — 같은 이름으로 확장

// type만 가능
type Result = Success | Failure           // union
type Mapped = { [K in Keys]: boolean }    // mapped type
type Inferred = ReturnType<typeof fn>     // conditional/utility
```

### type 우선이 유리한 점

**일관성**: type은 뭐든 다 됨 (union, intersection, mapped, conditional, object shape 전부). interface는 object shape만 가능해서 결국 type도 같이 써야 함 → "이건 interface, 저건 type" 고민하는 비용이 생김.

**안전성 (declaration merging 방지)**:

```ts
// 다른 파일에서 실수로 같은 이름 쓰면
interface Config { debug: boolean }
interface Config { logLevel: string }
// → 조용히 합쳐짐. 에러 없음.

type Config = { debug: boolean }
type Config = { logLevel: string }
// → 즉시 컴파일 에러. 안전함.
```

**에이전트 지침 단순화**: "interface인지 type인지" 판단하는 불필요한 분기가 사라짐.

**에러 메시지 가독성 (디버깅 유리)**:

```ts
// interface 에러 — 이름만 보여줌
Type 'X' is not assignable to type 'ButtonProps'
// → ButtonProps 정의를 찾아가야 뭐가 틀렸는지 알 수 있음

// type 에러 — 구조를 풀어서 보여줌
Type 'X' is not assignable to type '{ label: string } & { size: "sm" | "md" | "lg" }'
// → 에러 메시지 자체에서 바로 수정 가능
```

에이전트 입장에서도 에러 메시지만으로 바로 수정할 수 있어 추가 tool call이 필요 없음.

### interface가 유리한 점

- `extends`가 `&`보다 가독성이 좋음
- declaration merging이 필요한 라이브러리 타입 확장에는 필수

### 결론

**앱 코드에서는 type 우선**. 일관성·안전성·에러 메시지·에이전트 지침 단순화 어디서 봐도 이점. interface는 declaration merging이 진짜 필요할 때만.

---

## 11. Next.js의 해결책: optimizePackageImports

배럴 문제를 컴파일러 수준에서 해결:

1. 빌드 시 배럴 엔트리 파일을 분석
2. export 이름 → 실제 파일 경로 매핑 생성
3. import 자동 재작성: `from 'lib'` → `from 'lib/dist/esm/Button'`
4. 중첩 배럴과 `export *` 재귀 처리
5. 배럴을 완전히 우회 — 트리쉐이킹보다 비용이 적음

lucide-react, @headlessui/react 등 일반적인 라이브러리에 대해 사전 구성됨.

---

## 12. Atlassian의 마이그레이션 전략 (90,000개 파일)

1,000명 이상의 개발자가 활발히 작업하는 중 진행.

**기술적 기반:**
- `factsmap` 내부 도구로 전체 import/export 메타데이터 수집
- 배럴 체인 resolve: `a → b → c → d` → `a → d`
- ESLint fixable 규칙을 코드모드로 활용 (lint + 자동 변환 이중 목적)
- 병렬화된 ESLint 러너로 전체 코드베이스에 실행

**3단계 웨이브 랜딩:**
- **Wave 1**: 코드베이스의 80%인 휴면 패키지 대상 (활성 PR 없는 영역)
- **Wave 2**: 패키지가 아닌 개별 파일 단위로 타겟팅 (hot 파일 제외)
- **Wave 3**: 남은 수백 개 파일 — 선착순으로 개발자와 직접 경쟁

VCS 데이터로 활성 브랜치에서 변경 중인 파일을 식별 → 충돌 자동 방지.

**정리**: 더 이상 의존성 그래프에 나타나지 않는 수천 개 파일 자동 삭제.

---

## 13. 에이전트 가이드 적용 방법

### CLAUDE.md에서 참조

```markdown
# CLAUDE.md
See @docs/typescript-import-conventions.md for import and module rules.
```

### 핵심 원칙

- 에이전트는 기존 코드베이스 패턴을 학습해서 따라하므로, **명시적 규칙**이 중요
- 규칙 + CORRECT/WRONG 예시 → 에이전트가 패턴 매칭으로 바로 따를 수 있음
- "왜"보다 "무엇을" 위주로 짧게 작성

---

## 14. 한 줄 결론

| 영역 | 규칙 |
|------|------|
| 앱 내부 | 배럴 금지. 직접 경로 import |
| 패키지 경계 | subpath exports로 진입점 제공 |
| 외부 라이브러리 | optimizePackageImports에 맡기기 |
| 파일 구조 | `ComponentName/ComponentName.tsx` (index.ts 금지) |
| export | named export only (Vue .vue 예외) |
| 함수 선언 | 최상위는 function 키워드 |
| 타입 | type 우선 (interface는 merging 필요 시만) |

**"Atlassian이 90,000개 파일을 바꿔가며 뒤늦게 제거한 걸, 처음부터 안 만들면 된다."**

---

## 참고 자료

- [Atlassian: 75% Faster Builds by Removing Barrel Files](https://www.atlassian.com/blog/atlassian-engineering/faster-builds-when-removing-barrel-files) — [정리](./references/01-atlassian.md)
- [Vercel: How We Optimized Package Imports in Next.js](https://vercel.com/blog/how-we-optimized-package-imports-in-next-js) — [정리](./references/02-vercel-nextjs.md)
- [Next.js GitHub Issue #12557: Tree Shaking and Barrel Files](https://github.com/vercel/next.js/issues/12557) — [정리](./references/03-nextjs-issue.md)
