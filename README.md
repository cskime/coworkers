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
  - 팀원들이 공통 컴포넌트를 신경쓰지 않고 요구사항 분석 및 개발에 집중할 수 있었음
  - 전체 공통 컴포넌트를 전담해서 개발하여 기능 추가, 버그 수정 등 요청에 빠르게 대응
  - Storybook을 활용하여 별도의 공통 컴포넌트 page를 추가 개발하지 않고 빠르게 문서화

### 유저 인증 로직 개발

- 목표 : 보안과 개발 편의성을 모두 고려한 access token 및 refresh token 관리 로직 개발
- 활동
  - Access token과 refresh token을 response body로 받을 때 token을 안전하게 보관하면서도 개발 편의성을 높이고 구현 복잡도는 낮추는 방향으로 설계
  - 인증 관련 API들은 Next.js의 API routes 기능을 활용하여 proxy를 통해 요청하도록 구현 ([source code](https://github.com/codeit-fe18-4-3/coworkers/tree/develop/src/pages/api/auth))
    - Response body로 받은 refresh token을 `HttpOnly`, `Secure`, `SameSite=strict` cookie로 저장
    - 그 외에는 response를 client에 그대로 전달하고, client는 response body의 access token을 전역 상태로 관리
    - Page가 새로 load 될 때 access token이 유실되는 문제를 해결하기 위해, page가 load 될 때마다 access token을 재발급받고 전역 상태 갱신
  - Client는 interceptor에서 API 요청 시 access token 설정 ([source code](https://github.com/codeit-fe18-4-3/coworkers/blob/develop/src/services/interceptor/client.ts))
    - Client side에서 API 요청을 보낼 떄 사용하는 axios instance에 request interceptor 추가
    - Request interceptor는 전역 상태에 접근하여 client에서 API server로 보내는 요청의 `Authorization` header에 access token 주입
  - SSR 환경에서 API 요청 시 access token 재발급 및 client 전역 상태에 동기화 ([source code](https://github.com/codeit-fe18-4-3/coworkers/blob/develop/src/libs/ssr/with-auth.ts))
    - Page가 새로 로드될 때마다 access token을 재발급받고 API 호출 시 직접 `Authorization` header에 주입
    - 재발급받은 access token을 component에 prop으로 전달하여 client의 access token 전역 상태와 동기화
  - Next.js middleware를 활용하여 인증되지 않은 사용자의 특정 page 접근 차단 ([source code](https://github.com/codeit-fe18-4-3/coworkers/blob/develop/src/proxy.ts))
    - 인증되지 않은 사용자가 유저 기능을 사용하는 page로 접근 요청 시 로그인 페이지(`/login`)로 redirect
    - 인증된 사용자가 로그인 또는 회원가입(`/signup`) page로 접근 요청 시 메인 페이지(`/`)로 redirect
- 성과
  - Access token을 client에서 전역 상태로 관리하여 API 요청 시 `Authorization` request header에 token을 쉽게 설정 가능
  - Refresh token은 `HttpOnly`, `Secure`, `SameSite=strict` cookie로 저장하여 XSS, CSRF 공격에 대응
  - 인증되지 않은 사용자의 page 접근 요청을 middleware에서 사전에 차단하여, 접근을 허용하지 않는 page가 렌더링되지 않도록 방지
- 한계 및 대안
  - Access token을 memory에 저장하기 때문에 page를 새로 요청할 때마다 access token이 유실되어 재발급해야 함
  - Token 재발급 횟수가 많아지면 사용자가 늘어남에 따라 server에 부하가 발생할 수 있음
  - Access token을 local storage에 저장하여 불필요한 token 재발급 요청 횟수를 줄일 수 있음
  - Local storage의 보안 취약점은 access token의 만료 기간을 아주 짧게 설정하는 방법으로 보완
  - 백엔드 개발자와 소통할 수 있는 상황이라면, server가 token을 cookie로 전달해서 client가 요청을 보낼 때마다 server에 token cookie를 자동으로 전송하는 방법도 고려할 수 있음

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

### SSR 적용 page에서 API 요청 시 access token을 사용하기 위한 boilerplate code 정리

- 문제 상황
  - SSR을 적용한 page들은 React Query를 사용할 때 `getServerSideProps` 함수에서 아래 작업을 반복해야 함
    1. Refresh token cookie가 없거나 token이 만료된 경우 로그인 페이지로 redirect
    2. Proxy로 access token 재발급 요청
    3. Page component로 query params 및 access token 전달
       - 새로 발급받은 access token을 client의 전역 상태에 동기화하기 위해 props로 전달
    4. Server에서 사용한 query client를 dehydrate해서 page component로 전달
  - Page component 에서는 아래 작업을 반복해야 함
    1. Server로부터 전달받은 access token을 `useEffect` 안에서 전역 상태에 동기화
- 해결 방법
  - 반복되는 패턴의 코드를 재사용 가능한 함수로 만들고, 세부 구현을 신경쓰지 않고 개발할 수 있도록 추상화 필요
  - 반복되는 코드를 제거하고 `getServerSideProps` 함수를 만들 수 있는 [`gsspWithAuth`](https://github.com/codeit-fe18-4-3/coworkers/blob/3d24658d9e36ad794ef35eade4bb5b8d2d00e8ca/src/libs/ssr/with-auth.ts#L12-L40) 함수 구현
    - 이 때, `getServerSideProps` 함수의 반환값(`GetServerSidePropsResult`)을 생성해 주는 [`gsspPropsWithTokenReturn`](https://github.com/codeit-fe18-4-3/coworkers/blob/3d24658d9e36ad794ef35eade4bb5b8d2d00e8ca/src/libs/ssr/gssp-return.ts#L12-L30) 함수를 사용
      - `getServerSideProps` 함수 내부에서 반드시 반환해야 하는 값(e.g. access token, dehydrated data)들이 실수로 누락되는 것 방지
      - 복잡한 반환 값 구조를 신경쓰지 않고 함수 호출로 간단히 사용할 수 있도록 추상화
  - HOC pattern을 적용하여 반복되는 코드를 제거하고 SSR page component 함수를 만들 수 있는 [`serverSideComponentWithAuth`](https://github.com/codeit-fe18-4-3/coworkers/blob/3d24658d9e36ad794ef35eade4bb5b8d2d00e8ca/src/libs/ssr/with-auth.ts#L42-L50) 함수 구현
- 실제 사용 예시 ([source code](https://github.com/codeit-fe18-4-3/coworkers/blob/develop/src/pages/%5BteamId%5D/index.tsx))

  ```typescript
  // 1. Page component에서 server로부터 받아서 사용할 prop type 정의
  interface PageProps {
    groupId: number;
  }

  // 2. `gsspWithAuth` 함수로 `getServerSideProps` 구현
  export const getServerSideProps = gsspWithAuth(async (context, accessToken) => {
    const params = context.params;
    const teamId = Number(params?.teamId);
    if (isNaN(teamId)) {
      return GSSP_NOT_FOUND_RETURN;
    }

    const queryClient = new QueryClient();
    await prefetchGroup(queryClient, { groupId: teamId, accessToken }); // Prefetch group data
    await prefetchUser(queryClient, { accessToken }); // Prefetch user data

    return gsspPropsWithTokenReturn({
      props: { groupId: teamId },
      dehydratedState: dehydrate(queryClient),
      accessToken,
    });
  });

  // 3. `serverSideComponentWithAuth` 함수로 SSR page component 구현
  export default serverSideComponentWithAuth<PageProps>(({ groupId }) => {
    const { group, isFetching } = useGroupQuery({ groupId });
    // ...
    return (
      <div>
        {/* Page content */}
      </div>
    );
  });
  ```

- 성과
  - 팀원들이 SSR을 적용할 page에서 유저 인증 관련 코드들을 신경쓰지 않고 핵심 로직만 구현할 수 있었음
  - 복잡하고 반복되는 boilerplate code를 제거하여 개발자 실수를 방지하고 가독성 및 유지보수성 향상

### overlay-kit 사용 시 modal이 열리지 않는 문제

- 문제 상황
  - Modal 등 overlay 요소를 화면에 표시하는 코드를 state 선언 없이 선언적으로 관리하기 위해 'overlay-kit' 라이브러리 사용
  - 'overlay-kit' 문서에 따라 overlay 요소를 닫은 뒤 메모리에서도 제거하기 위해 `useEffect`의 cleanup 함수에서 `unmount()`가 호출되도록 구현
  - 이 때, overlay 요소가 열리지 않는 문제 발생
- 원인 파악
  - 'overlay-kit'이 동작하는 방식
    - `open()` 함수 호출 시 전달하는 callback 함수의 첫 번쨰 argument로 `isOpen` 상태값을 전달
    - 이 값을 사용해서 overlay 요소의 rendering 여부를 결정
    - Callback의 두 번째 argument로 전달되는 `close` 함수를 호출해서 animation과 함께 overlay 요소를 닫음
    - 이후 `unmount` 함수를 호출해서 overlay 요소를 메모리에서 제거
  - Overlay를 열 때 log를 추적하여 `isOpen` 상탯값이 `open()` 함수 호출 직후 `false`로 설정되는 것을 확인
  - `useEffect`의 cleanup 함수에서 `unmount`를 호출하지 않으면 문제가 발생하지 않는 것을 확인
  - 즉, overlay 요소가 처음 렌더링될 때 `useEffect`의 cleanup 함수가 한 번 호출되면서 `unmount`가 실행되어 즉시 메모리에서 해제되는 것이 원인
    - `useEffect`의 cleanup 함수가 실행되었다는 것은 컴포넌트가 리렌더링 되었다는 것
    - React의 strict mode에 의해 `useEffect`의 callback이 두 번 실행되는 과정에서 cleanup 함수가 한 번 실행될 수 있음
    - 따라서, React의 strict mode를 사용할 때 `useEffect`의 cleanup 함수가 한 번 실행되어 발생하는 문제임을 확인
- 해결 방법
  - `unmount` 호출 시점을 정확히 overlay의 animation 종료 이후 시점으로 변경
  - `motion` 라이브러리를 사용하고 있으므로, `AnimatePresence` 컴포넌트의 `onExitComplete` event handler에서 `unmount` 함수를 호출하여 해결
  - [관련 코드(`Overlay` component)](https://github.com/codeit-fe18-4-3/coworkers/blob/develop/src/components/overlay/index.tsx)

## 회고

### Keep

- 개발을 시작하기 전에 기획 및 요구사항을 분석하고 GitHub issue에 개발해야 하는 것과 개발하지 않아도 되는 것을 명확히 정리함
- 요구사항 분석 및 설계 후 개발 공수를 산정하고 GitHub project의 timeline에 공유하여 개발 일정을 체계적으로 관리함
- 매일 아침 스크럼을 통해 주기적으로 진행 상황을 공유하고 팀원들이 겪는 문제를 빠르게 파악하여 대응함

### Problem

- 새로운 기술을 도입했을 때 팀원들과 해당 기술을 충분히 학습하지 않은 상태로 프로젝트를 시작하여 좋은 코드를 고민하기 어려웠음

### Try

- 팀 프로젝트에서 새로운 기술을 도입할 계획이 있다면 모든 팀원들이 비슷한 수준의 이해도를 갖고 프로젝트에 참여할 수 있도록 사전에 학습
