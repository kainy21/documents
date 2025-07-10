<!-- @format -->

# React 표준 가이드

본 가이드는 Create React App(CRA)을 기반으로 JS 프로젝트와 TS 프로젝트를 모두 고려한 팀 표준을 제시합니다.

[빌드 및 런타임 의존성](#빌드-및-런타임-의존성)

[개발환경 설정](#개발환경-설정)

[Git 훅 설정](#git-훅-설정)

[테스트 구성](#테스트-구성)

[CI/CD권장 구성](#cicd권장-구성)

[상태관리](#상태관리)

[백앤드 API 통신 구조](#백앤드-api-통신-구조)

[VSCode Extensions](#vscode-extensions)

[개발-가이드라인](#개발-가이드라인)

## 빌드 및 런타임 의존성

1. 공통 권장 라이브러리

   - CRA(create-react-app)
   - Recoil
     - 상태관리를 일관되게 사용하기 위해 Redux 등의 라이브러리는 지양합니다.
   - sass
   - react-router-dom

   - 필요에 따라 React Hook Form, date-fns등 추가

## 개발환경 설정

1. ESLint 설정 (코드 품질 및 스타일가이드)

2. 팀의 일관된 코드 스타일을 위해 ESLint를 설정합니다. CRA에 기본으로 ESLint 설정이 포함되어 있으나, Airbnb JS 스타일 가이드를 확장하는 것을 권장합니다.

3. Prettier 통합 (코드 포매터)

   - ESLint와 호환: ESLint의 코드 스타일 규칙 중 Prettier와 충돌할 수 있는 규칙을 비활성화 하기 위해 eslint-config-prettier를 설치하고 ESLint 설정의 extends 마지막에 “prettier”를 추가. (이를 통해 ESLint는 포매팅 관련 규칙을 중복 검사하지 않고 Prettier가 포매팅을 책임짐)

   - 에디터 연동: VSCode등 에디터에서는 Prettier 확장을 설치하고 “Format on save”를 활성화해 파일 저장시 자동 포맷되도록 설정.

4. 절대경로 설정

   CRA 환경에서 `jsconfig.json` 또는 `tsconfig.json`에서 절대경로 설정

   ### 예시 - jsconfig.json

   ```json
   {
       "compilerOptions": {
       "baseUrl": "src",
           "path": {
               "@components/_": ["components/_"],
               "@hooks/_": ["hooks/_"],
               "@sahred/_": ["shared/_"],
               ...
           }
       }
   }
   ```

   ### 사용예시

   ```javscript
   // before
   import LoginForm from '../../components/auth/LoginForm';

   // after
   import LoginForm from '@components/auth/Loginform';
   ```

## Git 훅 설정

> Husky & lint-staged

Git Hooks를 활용해 커밋 단계에서 자동으로 린트/포맷을 돌려 코드 품질을 보장합니다.

`"lint-staged":{
"_.js":"eslint --fix",
"_.{js,ts,tsx,css,md,html,json}": "prettier --cache --write"
}`

위 설정은 staged 된 JS파일에 ESLint를 실행해 린트하고, JS/CSS/MD/HTML/JSON 파일에 Prettier를 적용합니다.( --cache 옵션은 Prettier가 반영된 파일만 포매팅하도록 해주며, 성능 향상을 위한 선택 사항)

## 테스트 구성

테스트 구성

1. 테스트 도구 선정
   : Jest, RTL(React Testing Library)가 CRA 템플릿에 기본 포함되어 있습니다.

CRA는 Zero-Config 지향이므로 가급적 기본 설정을 활용합니다.

2. 테스트 전략

3. Mocking

4. 코드 커버리지 기준

   - 테스트 시 네트워크 호출이나 전역 상태에 의존하는 부분은 Mock으로 대체합니다.
     Jest의 jest.mock() 함수를 사용하여 모듈을 모킹하거나, 필요 시 의존성 주입 패턴을 활용합니다.

   - 스냅샷 테스트: 필요에 따라 Jest의 toMatchSnapshot()을 사용한 스냅샷 테스트를 도입할 수 있습니다. 이는 UI변경에 민감하게 반응하므로, 스냅샷 테스트는 중요한 UI의 회귀 방지 용도로 최소한만 사용합니다. React Testing Library는 스냅샷보다는 DOM 쿼리를 통한 명시적 검증을 권장합니다.

   - 코드 커버리지: 코드 커버리지는 80% 이상을 권장 기준으로 삼습니다.

## CI/CD권장 구성

지속적 통합/배포를 통해 코드 퀄리티 검사와 배포 빌드를 자동화 합니다.

## 프로젝트 구조 및 명명 규칙

디렉토리 레이아웃(scalable구조)

소규모 구조

```bash
src/
├── api/
├── assets/
├── components/ # 소규모 구조
│ ├── auth/ # 라우트 단위 종속 컴포넌트
│ │ ├── LoginForm.jsx
│ │ └── LoginForm.scss
│ └── ...
├── pages/ # 페이지 컴포넌트 (각 페이지에 components 조합)
│ ├── Home/
│ ├── Home.jsx
│ └── Home.module.scss
├── hooks/ # 커스텀 React 훅 모음
│ ├── useAuth.js
│ └── ...
├── state/ # 전역 상태 (Recoil atoms/selectors)
├── styles/ # 전역 스타일 및 스타일 변수 (SCSS 파일 등)
├── utils/ # 유틸리티 함수 (예: 포맷터, API 클라이언트 등)
├── App.tsx
├── index.tsx
├── routes.tsx
└── **tests**/ # (선택) 테스트 전용 디렉토리 (또는 각 폴더별로 \*.test.js 작성)
└── ...
```

중~대형 구조 (도메인/기능별 구조로 확장)

```bash
src/
├── assets/
├── features/ # 도메인 또는 기능 단위로 구성된 폴더 (예: auth, product 등)
│ ├── auth/ # 도메인/(기능 영역: Auth 관련 컴포넌트)
│ │ ├── components/
│ │ ├── pages/
│ │ ├── hooks/
│ │ ├── state/
│ │ └── api.ts
│ └── ...
├── shared/ # 공통 유틸, 공통 컴포넌트, 공통 훅 등 도메인에 독립적인 재사용 가능 모듈
│ ├── components/
│ ├── hooks/
│ ├── utils/
│ └── layouts/
├── styles/
│ ├── \_variables.scss
│ ├── \_mixins.scss
│ └── tailwind.config.js
...
```

폴더별 파일 명명 규칙

pages/ PascalCase (Dashboard.js, UserList.js)

components/ PascalCase (Button.js, UserProfile.js)

contexts/ PascalCase, 'Context’ 접미어 (NavigationContext.js)

hooks/: camelCase, ‘use’ 접두어 (useAuth.js)

state/: camelCase, 'State’접미어 (userDateState.js)

utils/: camelCase (format.js, validation.js)

## 상태관리

1. Recoil

   1. 네이밍 규칙
      1. Atom: {상태명}State
      2. Selector: {상태명}Selector
   2. 기본 패턴

      1. Atom 정의

         ```javascript
         import { atom } from "recoil";

         export const exampleState = atom({
           key: "ExampleState", // 고유 키값(PascalCase)
           default: initialValue, //초기값
         });
         ```

      2. Selector 정의

         ```javascript
         import { selector } from 'recoil';

         export const exampleSelector = selector ({
             key: 'exampleSelector',
             get: ({ get }) > {
                 const value = get(someState);
                 return transformedValue;
             }
         })
         ```

## 백앤드 API 통신 구조

1. Axios 기반 HTTP 클라이언트
   - 공통 Axios 인스턴스

```javascript
// api/Api.js
import axios from "axios";

const SERVER_ENDPOINT = process.env.REACT_APP_SERVER_ENDPOINT;

const axiosInstance = axios.create({
  baseURL: SERVER_ENDPOINT,
});

export default axiosInstance;
```

2. 인터셉터 설정
   - 인증 토큰 및 에러 처리 공통으로 수행

```javascript
class InvalidTokenError extends Error {
  constructor(status, message) {
    super(message);
    this.message = message;
    this.status = status;
  }
}

axiosInstance.interceptors.response.use(
  (response) => response,
  (error) => {
    const { response } = error;
    const status = response?.status;
    const url = error.config?.url;

    if ((status === 401 || status === 403) && !url.includes("/login")) {
      logout(); // 토큰 초기화
      navigate("/login"); // 로그인 페이지로 이동
      return Promise.reject(
        new InvalidTokenError(
          status,
          "토큰이 만료되었거나, 유효하지 않은 토큰입니다."
        )
      );
    } else if (error.request) {
      addToast("서버에서 응답이 없습니다. 잠시 후 다시 시도해 주세요.");
    } else {
      addToast("알 수 없는 오류가 발생하였습니다. 잠시 후 다시 시도해 주세요.");
    }

    return Promise.reject(error); // 그 외 에러는 그대로 넘김
  }
);
```

## VSCode Extensions

- Prettier - 코드 포맷 자동화
- ESLint - 리얼타임 코드 린트
- GitLens - Git 히스토리 보기

## 개발 가이드라인

1. 컴포넌트 분리

   - UI 컴포넌트를 명확하게 분리
   - 각 컴포넌트의 책임을 명확히 정의

2. 오류 처리
   - 오류 발생 시 API 공통 error handler로 처리
   - 오류 발생 시 토스트 알림으로 사용자에게 피드백 제공
3. 일관된 UI 패턴
   - 디자인 시스템 생성하여 일관된 스타일과 레이아웃 유지
   - 어드민의 경우 coreUI 템플릿 사용
