---
title: Windows 11에서 Gemini CLI 실행 시 @lydell/node-pty 오류 해결
categories:
  - Gemini
tags:
  - Gemini
  - CLI
  - Windows
  - node-pty
toc: true
comments: true
---

Windows 11 환경에서 Gemini CLI를 실행하려 할 때 다음과 같은 오류 메시지와 마주했다. 이 오류는 Gemini CLI의 핵심 기능 중 하나인 터미널 인터페이스와 관련된 문제로, 특히 `node-pty` 패키지의 바이너리 종속성 문제로 인해 발생했다.

### 문제 발생 상황

Gemini CLI를 설치하고 실행했을 때, 아래와 같은 메시지가 출력되며 정상적인 동작이 불가능했다.

```
The @lydell/node-pty package supports your platform (win32-x64), but it could not find the binary package for it: @lydell/node-pty-win32-x64/conpty.node
```

### 원인

이 오류는 `node-pty` 패키지가 특정 플랫폼(여기서는 `win32-x64`)에 맞는 바이너리 파일을 찾지 못해 발생했다. `node-pty`는 Node.js 환경에서 의사 터미널(pseudo-terminal) 기능을 제공하는 데 사용되는데, 이 과정에서 OS별로 컴파일된 바이너리 모듈이 필요하다. Windows 11 `win32-x64` 환경에서 해당 바이너리가 제대로 설치되지 않았거나 경로를 찾지 못할 때 위와 같은 오류가 발생한다.

이 문제는 [google-gemini/gemini-cli](https://github.com/google-gemini/gemini-cli/issues/11272) GitHub 이슈에서도 논의된 바 있다.

### 해결 방법

두 가지 주요 해결 방법을 정리한다.

#### 1. 누락된 `node-pty` 바이너리 패키지 직접 설치

가장 직접적인 해결책은 오류 메시지에서 언급된 플랫폼별 바이너리 패키지를 수동으로 설치하는 것이다. 터미널에서 다음 명령어를 실행하면 된다.

```bash
npm i -g @lydell/node-pty-win32-x64
```

이 명령어를 실행하면 `win32-x64` 플랫폼에 특화된 `node-pty` 바이너리 모듈이 설치되어, Gemini CLI가 필요한 파일을 찾을 수 있게 된다.

#### 2. Gemini CLI의 대화형 셸 기능 비활성화

만약 첫 번째 방법으로 해결되지 않거나, 다른 이유로 `node-pty` 패키지 설치가 어려운 경우, Gemini CLI의 대화형 셸(interactive shell) 기능을 비활성화하여 대체 실행기(fallback executor)를 사용하도록 설정할 수 있다. **하지만 이 옵션을 사용할 경우 대화형 셸을 사용할 수 없게 되어 일부 기능에서 불편함이 발생할 수 있다.**

Windows 환경에서 `~/.gemini/settings.json` 파일을 생성하거나 편집하여 다음과 같이 내용을 추가한다.

```json
{
  "tools": {
    "shell": {
      "enableInteractiveShell": false
    }
  }
}
```

이 설정을 추가하면 Gemini CLI가 `node-pty`를 사용하는 대화형 셸 대신 다른 방식으로 명령을 실행하게 되어, 해당 오류를 우회할 수 있다.

**나의 경우에는 1번 방법을 통해 문제를 해결했다.**
