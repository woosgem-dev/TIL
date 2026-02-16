# Next.js에서 패키지 import를 최적화한 방법

> 원문: [How we optimized package imports in Next.js](https://vercel.com/blog/how-we-optimized-package-imports-in-next-js)
> 저자: Shu Ding (Software Engineer, Vercel)
> 게시일: 2023년 10월 13일

40% 더 빠른 콜드 부트와 28% 더 빠른 빌드.

---

## 배럴 파일이란?

여러 모듈을 단일 파일에서 그룹화하고 export하는 방식. 그룹화된 모듈에 접근할 수 있는 중앙화된 위치를 제공하여 import를 편리하게 만듦.

```js
// index.js
export { default as module1 } from './module1';
export { default as module2 } from './module2';
export { default as module3 } from './module3';

// 배럴 사용
import { module1, module2, module3 } from './utils';
```

일부 인기 있는 아이콘 및 컴포넌트 라이브러리는 **엔트리 배럴 파일에 최대 10,000개의 re-export**를 가지고 있음.

## 배럴 파일의 문제점

모든 `require(...)`와 `import '...'`에는 JavaScript 런타임에서의 숨겨진 비용이 있음. 수천 개의 다른 것들을 import하는 배럴 파일에서 하나의 export만 사용하고 싶더라도, 불필요한 모듈들을 import하는 비용을 여전히 지불하게 됨.

인기 있는 React 패키지 중에는 **import하는 것만으로 200~800ms**가 걸리는 것도 있음. 심한 경우 수 초.

로컬 개발과 프로덕션 성능 둘 다 영향받음. 서버리스 환경은 앱 시작할 때마다 전부 다시 import해야 하니 더 심각.

## 트리쉐이킹으로 해결할 수 없나?

트리쉐이킹은 *번들러* 기능(Webpack, Rollup 등)이지, JavaScript 런타임 기능이 아님. 라이브러리가 `external`로 표시되어 있으면 블랙박스로 남음.

라이브러리를 앱 코드와 함께 번들링하면 `sideEffects` 설정으로 트리쉐이킹이 작동하긴 하지만, 전체 모듈 그래프를 컴파일하고 분석해야 하므로 빌드가 상당히 느려짐.

## 첫 번째 시도: modularizeImports

Next.js의 초기 접근 방식. export된 이름과 실제 모듈 경로 간의 매핑 관계를 수동으로 설정.

```js
// 설정: my-lib/{{member}} 변환
// import { module2 } from 'my-lib'
// → import module2 from 'my-lib/module2'
```

문제점:
- 라이브러리의 내부 디렉토리 구조에 의존
- 수동 설정이 과도하게 필요
- 라이브러리 버전이 바뀌면 내부 구조도 바뀌어 변환이 무효화됨
- 수백만 개의 npm 패키지에 대해 확장 불가능

## 새로운 솔루션: optimizePackageImports

Next.js 13.5에서 도입. 배럴 파일을 자동으로 분석하여 직접 import로 변환.

```js
// next.config.js
module.exports = {
  experimental: {
    optimizePackageImports: ["my-lib"]
  }
}
```

작동 방식:
1. 엔트리 파일을 분석하여 배럴 파일인지 판단
2. 한 번의 패스로 스캔하여 import 매핑 자동 생성
3. 중첩된 배럴 파일과 와일드카드 export(`export * from`)를 재귀적으로 처리
4. 배럴 파일이 아닌 파일에 도달하면 프로세스 중단

트리쉐이킹보다 비용이 적음. 엔트리 배럴 파일만 한 번 스캔하면 됨.

`lucide-react`, `@headlessui/react` 등 일반적인 라이브러리에 대해 사전 구성됨.

## 성능 개선 측정

### 로컬 개발 (M2 MacBook Air)

| 라이브러리 | 이전 | 이후 | 차이 |
|------------|------|------|------|
| @mui/material | 7.1초 (2,225 모듈) | 2.9초 (735 모듈) | -4.2초 |
| recharts | 5.1초 (1,485 모듈) | 3.9초 (1,317 모듈) | -1.2초 |
| @material-ui/core | 6.3초 (1,304 모듈) | 4.4초 (596 모듈) | -1.9초 |
| react-use | 5.3초 (607 모듈) | 4.4초 (337 모듈) | -0.9초 |
| lucide-react | 5.8초 (1,583 모듈) | 3.0초 (333 모듈) | -2.8초 |
| @material-ui/icons | 10.2초 (11,738 모듈) | 2.9초 (632 모듈) | -7.3초 |
| @tabler/icons-react | 4.5초 (4,998 모듈) | 3.9초 (349 모듈) | -0.6초 |
| rxjs | 4.3초 (770 모듈) | 3.3초 (359 모듈) | -1.0초 |

여러 라이브러리를 사용하는 경우 이 수치는 빠르게 누적됨.

### 프로덕션 빌드

`lucide-react`와 `@headlessui/react`를 사용하는 Next.js App Router 페이지 벤치마크에서, `next build`가 **약 28% 더 빠르게** 실행.

### 콜드 부트

- 로컬: Node.js 서버가 **약 10% 더 빠르게** 시작
- 서버리스 (Vercel): 다른 개선 사항과 함께 최대 **40% 더 빠른 콜드 스타트**

### 재귀 배럴 파일

4단계의 10개 `export *` 표현식을 가진 모듈 (총 10,000개 모듈):
- 이전: 약 30초
- 이후: 약 7초
- 100,000개 이상의 모듈을 가진 일부 고객에서 **90% 더 빠른 리로드**
