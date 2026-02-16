# 배럴 파일 제거로 75% 더 빠른 빌드를 달성한 방법

> 원문: [How We Achieved 75% Faster Builds by Removing Barrel Files](https://www.atlassian.com/blog/atlassian-engineering/faster-builds-when-removing-barrel-files)
> 저자: Tim Sebastian (Principal Software Engineer, Atlassian)
> 게시일: 2025년 6월 26일

Jira 프론트엔드 코드베이스에서 JavaScript 배럴 파일을 제거한 결과, 빌드 시간이 75% 단축되었으며 TypeScript 하이라이팅과 유닛 테스트 속도도 크게 개선되었습니다. 이 대규모 자동화 작업은 CI 효율성도 높였고, 개발자의 코드 탐색 경험도 훨씬 나아졌습니다.

---

## "모범 사례"가 성능 병목이 되는 순간

Atlassian의 Jira 프론트엔드 코드베이스는 수천 개의 내부 패키지를 포함하는 규모로 성장했습니다. 이 정도 규모에서는 작은 프로젝트에서 잘 작동하던 아키텍처 결정이 심각한 성능 병목이 될 수 있습니다.

배럴 파일은 모듈의 단일 진입점 역할을 하는, 널리 사용되는 JavaScript 패턴입니다.

```js
// 개별 컴포넌트 파일들
// components/Button/Button.js
export const Button = () => { /* 컴포넌트 로직 */ };

// components/Modal/Modal.js
export const Modal = () => { /* 컴포넌트 로직 */ };

// 배럴 파일: components/index.js
export { Button } from './Button/Button';
export { Modal } from './Modal/Modal';

// 배럴 파일 방식 — 배럴을 통한 깔끔한 import
import { Button, Modal } from './components';
```

문제는 디렉토리 계층 전반에 걸친 연쇄적인 배럴 파일로 인해 더욱 악화되었습니다:

```js
// 깊은 컴포넌트
export const UserCard = () => { /* 컴포넌트 로직 */ };

// 1단계 배럴: features/UserManagement/components/index.js
export { UserCard } from './UserCard/UserCard';

// 2단계 배럴: features/UserManagement/index.js
export { UserCard, UserList } from './components';

// 최상위 배럴: features/index.js
export { UserCard, UserList, userManagementUtils } from './UserManagement';

// 최종 import — 깔끔해 보이지만 거대한 의존성 체인을 생성
import { UserCard } from './features';
```

코드베이스가 성장하면서 개발 경험에서 우려스러운 추세를 발견했습니다:

- **로컬 TypeScript 하이라이팅**이 최대 2분까지 걸려, 많은 개발자가 시스템이 아예 작동하지 않는다고 여기게 되었음
- **단일 유닛 테스트 실행**에 로컬에서 수 분이 걸림
- **CI 빌드**가 필요 이상으로 많은 테스트를 실행

---

## 근본 원인 조사: 데이터 따라가기

배럴 파일이 의존성 그래프의 비대화를 유발하고 있을 수 있다고 의심했습니다.

```js
// components/index.js (배럴 파일)
export { Button } from './Button/Button';
export { DataTable } from './DataTable/DataTable'; // 많은 의존성을 가진 무거운 컴포넌트
export { Chart } from './Chart/Chart';
// ... 50개 이상의 컴포넌트 export

// Button만 필요한데
import { Button } from './components';
// → DataTable, Chart 등 50개 전부 파싱
```

`Button`만 import하더라도, TypeScript와 Jest 같은 도구들은 전체 배럴 파일을 처리해야 합니다. 컴포넌트 하나 import했을 뿐인데, 관련 없는 모듈 수만 개를 읽고 파싱하고 의존성까지 해결하는 연쇄가 벌어지는 셈입니다.

전체 코드베이스를 대상으로 "충분히 좋은" 코드모드를 시도하기로 했고, 결과는 예상보다 명확했습니다:

- **로컬 테스트 성능**: 단일 테스트 실행이 훨씬 빨라짐
- **TypeScript 처리**: 하이라이팅 속도가 크게 개선됨
- **CI 테스트 선택**: 최대 70% 이상 개선
- **번들 생성**: 번들 크기가 약간 줄어들었고 번들 시간도 단축됨

---

## 솔루션 엔지니어링: 대규모 리팩토링 자동화

### 기술적 기반 구축

`factsmap`이라는 내부 도구를 사용하여 코드베이스 전반에 걸쳐 import/export 메타데이터를 수집했습니다.

이전에 `a → b → c → d` 같은 체인이 있었다면, `a → d`로 직접 import를 생성:

```js
// 이전: 배럴 파일을 통한 import 체인
import { Button } from './ui';
// → ui/index.js → components/index.js → Button/Button.js

// 이후: 직접 import
import { Button } from './components/Button/Button';
```

린터이자 코드 변환기 역할을 동시에 하는 ESLint 규칙을 만들었습니다:

```js
module.exports = {
  meta: { fixable: "code" },
  create(context) {
    return {
      ImportDeclaration(node) {
        if (isBarrelFileImport(node.source.value)) {
          context.report({
            node,
            message: "Avoid barrel file imports",
            fix(fixer) {
              const directImport = resolveToDirectImport(node);
              return fixer.replaceText(node, directImport);
            }
          });
        }
      }
    };
  }
};
```

### 3단계 웨이브 랜딩 전략

1,000명 이상의 개발자의 일상 업무를 차단하지 않으면서 90,000개 이상의 파일을 변경해야 했습니다.

**코드베이스의 최대 80%가 특정 시점에 휴면 상태**라는 통찰이 핵심이었습니다.

VCS에서 활성 브랜치와 변경된 파일 데이터를 가져오는 스크립트를 만들어 충돌을 자동으로 방지했습니다.

- **Wave 1**: 코드베이스의 80%인 휴면 패키지 대상 (활성 PR 없는 영역). 패키지 단위로 충돌 회피.
- **Wave 2**: 전체 패키지가 아닌 개별 파일 단위로 타겟팅. 풀 리퀘스트에 의해 변경되지 않는 파일만 대상.
- **Wave 3**: 남은 수백 개 파일. 선착순으로 개발자와 직접 경쟁하며 랜딩.

며칠 만에 90,000개 이상의 파일에 변경을 랜딩하면서 최소한의 충돌만 경험했습니다.

### 정리 단계

각 웨이브 이후, 더 이상 어떤 의존성 그래프에도 나타나지 않는 파일을 식별하고 자동으로 삭제. 더 이상 목적이 없는 수천 개의 폐기된 파일이 정리되었습니다.

---

## 영향 측정

- **TypeScript 하이라이팅** 속도가 30% 이상 향상
- **로컬 유닛 테스트**가 평균 약 50% 빨라졌으며, 특정 패키지에서는 최대 10배 개선
- **CI 유닛 테스트**: 빌드당 1,600개 → 200개 (88% 감소), 평균 실행 시간 73% 단축
- **CI 통합 테스트**: 빌드당 130개 → 20개 (85% 감소)
- **VR 테스트**: 50개 → 25개 (50% 감소)
- **전체 빌드 시간**: 커밋당 소비 빌드 분 기준 **75% 감소**

---

## 예상치 못한 이점

- **IDE 네비게이션 향상**: import를 클릭하면 소스 파일로 직접 이동 (배럴 체인 대신)
- **명확한 의존성 관계**: 어떤 코드가 실제로 무엇에 의존하는지 투명해짐
- **빌드 도구 단순화**: 번들링과 분석 도구의 복잡성 감소
- **추가 최적화 가능**: 더 나은 테스트 선택을 통한 동적 CI 파이프라인 구현

---

## 트레이드오프

- 패키지가 배럴 파일을 통해 "공개 API"를 쉽게 제어할 수 없어 캡슐화의 한 계층을 잃음
- 소스 파일을 이동하면 모든 직접 import를 업데이트해야 하므로 리팩토링이 더 취약해짐
- 직접 import가 모듈 간 더 깊은 의존성을 유도하여 결합도 증가 가능성

Atlassian의 판단: **추상화와 성능이 충돌하면 성능을 택하고, 내부 코드는 직접적인 관계를 선호하며, 관습보다 측정 결과를 믿는다.**

---

## 핵심 교훈

- 특정 규모에 맞지 않을 때 "모범 사례"에 의문을 제기하라
- 변경 전후로 모든 것을 측정하라
- 겉보기에 작은 결정들의 복합 효과를 고려하라
- 때로는 가장 큰 성과가 최적화를 추가하는 것이 아니라 복잡성을 제거하는 데서 나온다
