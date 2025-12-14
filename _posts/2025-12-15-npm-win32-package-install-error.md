---
title: Windows에서 npm 설치 시 win32 바이너리 모듈 누락 오류 해결 (rollup, lightningcss)
categories:
  - Development
tags:
  - npm
  - Node.js
  - Windows
  - Rollup
  - Vite
  - Next.js
toc: true
comments: true
---

Windows에서 Vite/Next.js 프로젝트를 실행하려고 서버를 실행하면, `*-win32-x64-msvc` 바이너리 모듈을 찾지 못한다는 오류가 발생했다. 결론부터 말하면 npm 설정에서 OS가 `linux`로 강제되어 Windows용 optional dependency가 설치되지 않은 것이 원인이었다.

## 증상

대표적으로 아래와 같은 메시지가 출력됐다.

```plaintext
Error evaluating Node.js code
Error: Cannot find module '../lightningcss.win32-x64-msvc.node'
Require stack:
...
```

```plaintext
Error: Cannot find module @rollup/rollup-win32-x64-msvc. npm has a bug related to optional dependencies (https://github.com/npm/cli/issues/4828).
Please try `npm i` again after removing both package-lock.json and node_modules directory.
...
  [cause]: Error: Cannot find module '@rollup/rollup-win32-x64-msvc'
  Require stack:
...
```

에러 메시지에 npm optional dependencies 버그 링크가 함께 포함되어 있어서, 처음에는 npm 자체 문제나 rollup 문제로 착각하기 쉬웠다.

검색해보면 보통 아래 같은 해결책이 나온다.

1. Microsoft Visual C++ Redistributable 설치
2. `*-win32-x64-msvc` 패키지(예: `@rollup/rollup-win32-x64-msvc`)를 직접 설치
3. npm 설정에서 OS 강제값을 수정

1번은 최신 버전을 설치해도 내 환경에서는 동일한 문제가 재현됐다.

2번은 로컬에서는 우회가 되지만, 프로젝트 의존성/락파일에 Windows 전용 패키지가 섞일 수 있어 Linux/CI/도커 환경에서 역으로 문제가 되는 경우가 생길 수 있다.

결국 3번이 근본 원인이었다.

## 원인

npm 설정에서 OS가 `linux`로 고정되어 있었다.

```bash
npm config list
```

위 명령을 실행하면 현재 npm 설정을 볼 수 있는데, 아래처럼 `os = "linux"`가 찍혀 있었다.

![npm config list 결과](/assets/blobs/251215-npm-config-list.png)

이 상태에서는 npm이 Windows에서 설치되어야 할 optional dependency(예: `@rollup/rollup-win32-x64-msvc`, `lightningcss.win32-x64-msvc`)를 설치 대상에서 제외할 수 있다. 결과적으로 런타임에서 해당 모듈을 찾지 못해 에러가 발생한다.

## 해결 방법

핵심은 `.npmrc`에 들어있는 OS 강제 설정을 제거(권장)하거나, `win32`로 바꾸는 것이다.

### 1) `.npmrc`에서 OS 강제 설정 제거 또는 수정

우선 사용자 설정 파일을 확인한다.

- `C:\Users\<유저>\.npmrc`

여기에 아래처럼 들어있다면,

```ini
os = "linux"
```

다음 중 하나로 처리한다.

- 권장: 해당 줄을 삭제한다(기본값으로 되돌리는 방식)
- 대안: `os = "win32"`로 수정한다

프로젝트 루트(레포지토리)에 `.npmrc`가 따로 있는 경우도 있으니, 그 파일도 함께 확인한다.

### 2) 의존성 캐시를 지우고 재설치

설정을 고쳤다면, 프로젝트 폴더에서 아래 항목을 삭제한 뒤 다시 설치한다.

- `node_modules` 디렉토리
- `package-lock.json` 파일

그 다음 `npm i`을 다시 실행한다.

### 3) 실행 확인

이후 dev 서버나 빌드를 실행했을 때 `*-win32-x64-msvc` 관련 모듈 누락 오류가 사라지면 해결된 것이다.
