# RunCourse3D Mobile

React Native와 Expo 기반의 RunCourse3D 앱 프로젝트다. 지도와 3D 러너 콘텐츠는 추후 이 프로젝트 내부에 WebView용 콘텐츠로 구현

## 요구 환경

- Node.js 22.13 이상
- npm
- iOS: Xcode 및 iOS Simulator
- Android: Android Studio 및 Android Emulator

## 설치 및 실행

```bash
npm install
npm start
```

플랫폼별 실행:

```bash
npm run ios
npm run android
```

백그라운드 위치 추적은 Expo Go에서 지원되지 않으므로 `expo-dev-client`를 포함한 development build에서 검증해야 한다.

## 환경 변수

`.env.example`을 참고해 로컬 `.env`를 생성한다.

```dotenv
EXPO_PUBLIC_SUPABASE_URL=
EXPO_PUBLIC_SUPABASE_ANON_KEY=
```

실제 Supabase 값은 저장소에 커밋하지 않는다. Supabase 서버 전용 service role 키는 모바일 앱에 넣지 않는다.

## 기술 스택

- React Native 0.86.3
- Expo SDK 57
- TypeScript
- **화면 탐색**: Expo Router
- **WebView**: react-native-webview
- **지도(Map) 렌더링**: MapLibre GL JS (WebView)
- **GPS 및 위치 권한**: Expo Location
- **백그라운드 GPS**: Expo Task Manager
- **3D 러너 렌더링**: Three.js (WebView)
- **상태 관리**: Zustand
- **서버 상태 관리**: TanStack Query
- **로컬 데이터베이스**: Expo SQLite
- **서버 데이터베이스**: PostgreSQL + PostGIS
- **백엔드**: Supabase

MapLibre GL JS와 Three.js 패키지는 설치되어 있다. 두 라이브러리는 React Native 화면에서 직접 실행하지 않고, 추후 이 프로젝트 내부에 추가할 WebView 브라우저 콘텐츠에서 사용한다.


## FSD (Feature-Sliced Design) 아키텍처 가이드

## FSD가 필요한 이유

### FSD의 핵심 아이디어

- **기능(Feature) 중심 분리**
- 의존성 방향을 강제
- 규모가 커져도 구조가 무너지지 않음

## FSD의 6가지 레이어

> 아래로 갈수록 구체적, 위로 갈수록 범용적입니다.

| 레이어 | 설명 |
|--------|------|
| `app` | 앱 초기화, 전역 설정 |
| `pages` | 라우트 단위 페이지 |
| `widgets` | 조합된 UI 블록 |
| `features` | 사용자 행위/유즈케이스 |
| `entities` | 도메인 모델 |
| `shared` | 완전 공용 리소스 |

### 의존성 규칙 (중요)

```text
app
 └─ pages
     └─ widgets
         └─ features
             └─ entities
                 └─ shared
```

> **아래 레이어는 위 레이어를 절대 참조하지 않음**

## 레이어별 역할 설명

### 1. shared (완전 공용)

- 비즈니스 의미 없음
- 재사용 가능한 순수 리소스

**예시:**
- UI 컴포넌트 (Button, Modal)
- 공통 hooks
- API client
- util 함수
- design token

```text
shared/
├─ ui/
│   └─ Button/
├─ lib/
├─ hooks/
└─ config/
```

### 2. entities (도메인 모델)

- 비즈니스 엔티티
- 상태 + 타입 + API + 기본 UI

**예시:** `User`, `Product`, `Article`

```text
entities/
└─ user/
    ├─ model/      // store, types
    ├─ api/        // user 관련 API
    └─ ui/         // UserAvatar
```


> 👉 **판단 기준:** "이건 시스템의 **명사**인가?"

### 3. features (행위 / 유즈케이스)

- 사용자가 수행하는 행동 단위
- 하나의 목적을 가진 기능

**예시:** 로그인, 장바구니 담기, 게시글 좋아요

```text
features/
└─ auth-login/
    ├─ model/
    ├─ ui/
    └─ api/
```

> 👉 **판단 기준:** "이건 **동사**인가?"

### 4. widgets (조합 UI 블록)

- 여러 feature/entity를 조합한 화면 블록
- 독립적 의미를 가진 UI 영역

**예시:** `Header`, `Sidebar`, `ProductList`

```text
widgets/
└─ header/
    └─ ui/
```

> 👉 **판단 기준:** "이건 페이지의 **한 덩어리 영역**인가?"

### 5. pages (라우트 단위)

- URL과 1:1 대응
- 레이아웃 조립만 담당 (로직 최소화)

```text
pages/
└─ login/
    └─ ui/
```

### 6. app (전역 설정)

- 앱 초기화
- 라우터
- 전역 스타일
- Provider

```text
app/
├─ providers/
├─ router/
└─ styles/
```

## 실제 예시 구조

```text
src/
├─ app/
├─ pages/
│   └─ login/
├─ widgets/
│   └─ header/
├─ features/
│   └─ auth-login/
├─ entities/
│   └─ user/
└─ shared/
```

## 폴더 내부 규칙 (Slice 구조) ( 현재 mfront 불가능 단계적 수정)

각 slice는 보통 아래처럼 나눕니다:

```text
feature-name/
├─ ui/
├─ model/
├─ api/
└─ index.ts
```

- ✅ 외부에서는 `index.ts`만 import
- ✅ 내부 구현은 숨김

## 설계할 때 가장 많이 헷갈리는 기준

### Feature vs Entity

| 구분 | 설명 |
|------|------|
| **Entity** | 데이터 중심 (`User`, `Product`) |
| **Feature** | 행동 중심 (`Login`, `AddToCart`) |

### Widget vs Feature

| 구분 | 설명 |
|------|------|
| **Feature** | 버튼 하나여도 의미 있으면 Feature |
| **Widget** | 여러 기능을 묶은 UI 덩어리 |

### 공용 컴포넌트는 어디?

| 조건 | 위치 |
|------|------|
| 비즈니스 의미 없음 | `shared` |
| 의미 있음 (`UserAvatar`) | `entities` |

## FSD 도입 시 팁

1. **처음부터 완벽히 나누지 말 것**
2. `shared` → `entities` → `features` 순서로 점진적 분리
3. slice가 애매하면 일단 `features`에 두고 커지면 분리
4. **ESLint + path alias**로 의존성 규칙 강제 추천