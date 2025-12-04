# Coworkers: 업무 배정 및 현황 공유 서비스

## 소개

- 코드잇 스프린트 FE 18기 과정에서 4주간 진행한 세 번째 팀 프로젝트
- 주제 선정 이유
  - [이전 팀 프로젝트](https://github.com/cskime/Taskify)에서 개발했던 할 일 관리 서비스와 비슷하지만, 할 일 반복 주기를 설정하는 등 더 복잡한 구조의 고도화된 서비스를 개발해보는 경험
  - 높은 난이도의 프로젝트에 도전
- 이전 프로젝트들과 달리 개발 효율과 편의성을 높여줄 수 있는 라이브러리를 적극적으로 활용하여 결과물의 quality를 높이는 것에 집중

## 사용 기술

- Language : TypeScript
- Framework : Next.js(page router)
- State management : React Context API, React Query, Zustand
- Style : Tailwind CSS
- Network : Axios
- Deploy : GitHub Actions, Vercel
- Packages : overlay-kit, motion,

## 역할 및 성과

### 공통 컴포넌트 개발

- 목표 : 앱 전체에서 사용하는 공통 컴포넌트를 전담해서 개발
- 활동
  - 특정 기능에 국한되지 않고 재사용 할 수 있는 요소들을 공통 컴포넌트로 개발 ([관련 issue](https://github.com/codeit-fe18-4-3/coworkers/issues/2))
  - [Storybook](https://codeit-fe18-4-3.github.io/coworkers/)을 활용하여 개발된 공통 컴포넌트 문서화
- 성과
  - 프로젝트 초반에 공통 컴포넌트를 빠르게 개발하여 팀원들이 page를 개발할 때 곧바로 활용할 수 있었음
  - 공통 컴포넌트를 한 명이 전담해서 개발하여 기능 추가, 버그 수정 등 요청에 빠르게 대응할 수 있었음
  - Storybook을 활용하여 별도의 공통 컴포넌트 page를 추가 개발하지 않고 빠르게 문서화할 수 있었음

### 유저 인증 로직 개발

- 목표 : 보안과 개발 편의성을 모두 고려한 access token과 refresh token 관리 로직 개발
- 활동
  - 로그인 성공 시 access token과 refresh token을 response body로 받고 있는 상황에서, token을 안전하게 보관하되 개발 편의성을 높이고 구현 복잡도는 낮추는 방법으로 설계
  - Refresh token을 사용하여 access token을
  - access token과 refresh token을 안전하게 관리할 수 있는 방법 조사 및 설계
  - Next.js의 Dynamic API routes 기능을 활용하여 API proxy 서버 구현
  - Client와 proxy 서버 간 통신 및 proxy 서버와 실제 API 서버 간 통신을 담당하는 모듈 개발
- 성과
  - access token을 `HttpOnly`, `Secure`, `SameSite=strict` cookie로 저장하여 XSS, CSRF 공격에 대응할 수 있었음
  - Client가 access token을 직접 관리하지 않으므로 브라우저를 통해 외부로 노출되지 않음
  - API 요청 시마다 token을 자동으로 주입하여 개발 편의성 향상
- 한계 및 대안
  - Page를 새로 요청할 때마다 access token을 재발급받아야 하므로 사용자가 많아지면 server에 부하가 발생할 수 있음
  - Access token을 local storage에 저장하여 token 재발급 요청 횟수를 줄이고, 그 대신 access token의 만료 기간을 아주 짧게 설정하여 보안 상 위험을 감소시킬 수 있음
  - 백엔드 개발자와 소통할 수 있는 상황이라면, server가 token을 cookie로 내려주고 client는 요청을 보낼 때마다 cookie를 자동으로 전송하도록 설정하는 방법도 고려 가능

### 프로젝트 관리 역할

- 목표 : 프로젝트를 기한 내에 성공적으로 완료하고 모든 팀원들이 성장하는 것을 목표로 진행
- 활동
  - 팀원들에게 담당한 부분의 기획 및 요구사항을 분석하여 개발 범위를 파악하고 공수를 산정하는 방법을 공유하고, [GitHub project의 timeline](https://github.com/orgs/codeit-fe18-4-3/projects/2/views/1)으로 일정 관리
  - 팀원들이 올린 PR에 상세한 코드 리뷰 진행 (e.g. [예시 1](https://github.com/codeit-fe18-4-3/coworkers/pull/65#discussion_r2533600277), [예시 2](https://github.com/codeit-fe18-4-3/coworkers/pull/66#discussion_r2529530465))
  - 개발 중 겪은 문제를 해결하면서 알게 된 내용을 팀원들과 공유하여 함께 성장할 수 있는 프로젝트 지향 ([관련 글](https://github.com/codeit-fe18-4-3/coworkers/discussions/58))
- 성과
  - 매일 아침 팀 미팅 시간에 timeline을 통해 진행 상황을 공유하여 일정을 유동적으로 관리하고, 개발 일정이 지연되는 상황을 빠르게 파악하여 대응할 수 있었음
  - 요구사항 분석 및 공수 산정 활동을 통해 팀원들이 개발해야 하는 항목과 범위를 명확히 이해하고 개발을 시작하여 불필요한 작업을 줄일 수 있었음
  - 문제 해결 경험을 공유하여 팀원들이 같은 실수를 반복하지 않게 되었고, 모든 팀원들의 성장에 기여함

## 문제 해결

### overlay-kit 사용 시 modal이 열리지 않는 문제

- 문제 상황
- 원인 파악
- 해결 방법

### SSR 적용 page에서 API 요청 시 access token을 사용하기 위한 boilerplate code 문제

- 문제 상황
- 원인 파악
- 해결 방법

### Page에서 `useMediaQuery` custom hook을 사용할 때 error가 발생하는 문제

- 문제 상황
  - TypeScript 코드에서 responsive UI 구성을 위해 `@uidotdev/usehooks` package의 `useMediaQuery`를 활용한 [`useResponsive` custom hook](https://github.com/Codeit-FE18-Part3-Team4/Taskify/blob/develop/src/hooks/use-responsive.tsx) 구현
  - Modal 컴포넌트에서는 정상적으로 사용 가능했지만, page 컴포넌트에서 사용하면 "useMediaQuery is a client-only hook" error 발생
    <br><img src="/docs/images/image-01.png" /><br>
- 문제 원인
  - Next.js 서버에서 page가 pre-rendering 될 때, `useMediaQuery` 내부에서 `matchMedia` Web API를 사용하여 발생하는 문제
  - `@uidotdev/usehooks` package는 `useSyncExternalStore` hook을 사용하여 server에서 실행될 때 해당 error를 throw 하는 것을 확인 ([source code](https://github.com/uidotdev/usehooks/blob/945436df0037bc21133379a5e13f1bd73f1ffc36/index.js#L785-L807))
  - Modal component는 page 로드 후 CSR 방식으로 rendering 되므로 문제가 없었음
- 해결 방법
  - `matchMedia` Web API가 client에서만 실행될 수 있도록 구현해야 함
  - Server에서도 error 없이 `matchMedia`를 사용할 수 있는 [`react-responsive` package로 교체](https://github.com/Codeit-FE18-Part3-Team4/Taskify/blob/develop/src/hooks/use-ssr-responsive.tsx)
    - `react-responsive` package는 [server에서 실행될 때 device 정보를 주입하여 Web API가 아닌 별도의 라이브러리에 구현된 `matchMedia`를 실행](https://github.com/yocontra/react-responsive/blob/ff23a19f49576a3f06c8334affbef5996f147e12/src/useMediaQuery.ts#L72-L92)
    - Web API의 `matchMedia`는 [component가 mount 된 시점 이후에 client 에서만 호출되도록 보장](https://github.com/yocontra/react-responsive/blob/ff23a19f49576a3f06c8334affbef5996f147e12/src/useMediaQuery.ts#L72-L92)

### Response body로 받는 access token을 안전하게 관리하기

- 문제 상황
  - 실습 API 서버는 만료 기간이 없는 access token만 제공하고 refresh token은 제공하지 않음
  - 탈취당한 access token을 갱신할 방법이 없으므로, access token을 외부에 노출시키지 않고 관리할 수 있는 방법 필요
- 해결 방법
  - Dynamic API routes 기능을 활용하여 Next.js 서버를 API proxy로 사용
    - [로그인 요청 proxy](https://github.com/cskime/Taskify/blob/portfolio/src/pages/api/auth/login.ts)
    - [기타 API 요청 proxy](https://github.com/cskime/Taskify/blob/portfolio/src/pages/api/%5B...slug%5D.ts)
  - Client는 실제 API endpoint 앞에 `/api`를 붙여서 요청
  - Proxy 서버를 통해 로그인 요청을 보내면 실제 API 서버가 반환하는 access token을 `HttpOnly`, `Secure`, `SameSite=strict` cookie로 저장
  - Proxy에서 [실제 API 서버로 보내는 요청의 `Authorization` header에 token을 담아서 전송](https://github.com/cskime/Taskify/blob/portfolio/src/services/proxy-client.ts)
- 성과
  - Access token을 `HttpOnly` cookie로 저장하여 XSS 공격에 대응
  - Cookie를 `Secure`, `SameSite=strict` 설정하여 CSRF 공격에 대응
  - Client가 access token을 직접 관리하지 않으므로 브라우저를 통해 외부로 노출되지 않음
