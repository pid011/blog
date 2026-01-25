---
title: Unreal Engine LNK 2019 컴파일 문제 해결 방법
categories: ["Unreal Engine"]
tags: ["Unreal Engine", "Game"]
toc: true
comments: true
---

Unreal Engine에서 C++ 코드를 컴파일 할 때 LNK 2019 오류를 종종 접한다.

이 오류는 대부분 모듈을 임포트하지 않아서 발생한다. 해당 문제의 코드를 잘 살펴보고 코드가 참조하고 있는 변수의 타입이 속한 모듈이 `Build.cs` 의 `PublicDependencyModuleNames` 또는 `PrivateDependencyModuleNames`에 포함되어 있는지 확인해야 한다.
