---
title: GitHub Organizations 서드파티 툴 접근 권한 해결 방법
categories:
  - GitHub
tags:
  - Git
  - Organization
description: GitHub Organizations에서 서드파티 애플리케이션 접근 제한으로 인한 Repository Not Found 오류 해결법
toc: true
comments: true
mermaid: true
---
이전에 [GitHub 레포지토리를 찾을 수 없는 오류 해결방법 (HTTPS, SSH 관련 오류)](/blog/posts/github-ssh/)에서 SSH 관련 문제를 다뤘다면, 이번에는 GitHub Organizations의 접근 정책으로 인한 문제를 다루어 보려고 한다.

JetBrains Rider에서 Git 작업을 하던 중 Repository Not Found 오류가 발생했다.

## 오류 상황
JetBrains Rider의 내장 Git 툴로 push나 clone을 진행하려 하면 대상 레포지토리를 찾지 못한다는 오류가 발생했다. 흥미로운 점은 평소에 사용하던 Fork에서는 동일한 레포지토리에 대해 아무런 문제없이 작업이 가능했다는 것이다. 즉, JetBrains Rider에서만 유독 원격 Git 조작을 할 때 오류가 발생하는 상황이었다.

문제가 된 레포지토리는 내 개인 계정이 아닌 Organization에 속한 레포지토리였다.

## 원인
문제의 원인은 레포지토리가 속한 Organization의 액세스 정책 설정에 있었다. GitHub Organization 설정을 살펴보면 Third-party application access policy라는 항목이 있는데, 이 설정이 기본적으로 `Access restricted`로 되어 있다. 이 제한 정책이 활성화되어 있으면 Organization에서 명시적으로 허용하지 않은 서드파티 애플리케이션은 해당 Organization의 레포지토리에 접근할 수 없게 된다.

여기서 재미있는 점은 Fork 같은 일부 애플리케이션들은 GitHub와 연동할 때 사용자에게 Organization 권한을 요청하고 승인받는 과정이 포함되어 있다는 것이다. 내가 기존에 사용하던 Fork도 이런 과정을 통해 Organization에 등록되어 정상적으로 작동했던 것으로 보인다.

![GitHub Organization Third-party application access policy 설정 화면](/assets/blobs/250911-githubscreenshot.png)

## 해결 방법

가장 간단한 해결방법은 Organization 관리자가 위 스크린샷에서 보이는 "Remove restrictions" 버튼을 클릭하여 서드파티 애플리케이션 접근 제한을 해제하는 것이다. 제한이 해제되면 JetBrains Rider를 포함한 모든 서드파티 툴에서 해당 Organization의 레포지토리에 자유롭게 접근할 수 있게 된다.

다만 이 방법을 사용할 때는 주의해야 할 점이 있다. 특히 규모가 큰 Organization이나 보안이 중요한 프로젝트를 다루는 경우에는 무분별한 서드파티 애플리케이션의 접근을 허용하는 것이 보안상 위험할 수 있다. 따라서 제한을 해제하기 전에 조직의 보안 정책과 부합하는지, 그리고 어떤 잠재적 위험이 있을 수 있는지 충분히 검토해보는 것이 좋다.