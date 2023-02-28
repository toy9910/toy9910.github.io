---
title: "[Android] Context"
author: deBaeloper08
date: 2022-11-29 21:00:00 +0900
categories: [Android]
tags: [context]
---

## Context란?

안드로이드 개발을 하다보면 Context가 필요한 순간이 굉장히 많다. Context를 넣어야 할 때 항상 아무 생각없이 Activity값을 넣었는데 막상 Context가 무엇을 의미하는지는 모르고 있었다.

Android 공식 문서에 따르면

> Interface to global information about an application environment. This is an abstract class whose implementation is provided by the Android system. It allows access to application-specific resources and classes, as well as up-calls for application-level operations such as launching activities, broadcasting and receiving intents, etc.

라고 되어있는데 대략적으로 해석하면 Context는 Android 시스템에서 제공하는 추상 클래스로 어플리케이션 별 리소스와 클래스 접근을 허용하고 Activity 시작, Intent 수신, Broadcasting 등 Application 수준 작업의 호출이 가능하다.

## 역할

- 애플리케이션의 현재 상태를 나타낸다.
- 액티비티와 어플리케이션의 정보를 얻기 위해 사용할 수 있다.
- 리소스, 데이터베이스, shared preference 등에 접근하기 위해 사용할 수 있다.
- 액티비티와 애플리케이션 클래스는 Context 클래스를 확장한 클래스다.

## 종류

### Application Context

- Application lifecycle에 귀속됨
- Singletion Instance
- `getApplicationContext()`로 접근
- 어떤 Context보다도 오래 유지됨

Application Context는 현재의 Context와 분리된 lifecycle을 가진 Context가 필요할 때나 Activity의 범위를 넘어서 Context를 전달할 때에 사용한다.

만약, Application에서 Singleton 객체를 생성했는데 그 객체에서 Context가 필요하다면 어떤 Context를 사용해야 할까?

바로, Application Context를 사용해야 한다.

만약, Activity Context를 전달했다면, Activity가 사용되지 않는 경우에도 불필요하게 Activity를 참조하여 메모리 누수가 발생한다. 그러므로 Singleton의 경우 항상 Application Context를 전달해야 한다.

**Application은 Garbage Collector에 의해 수집되지 않는데 Activity Context는 Activity에 대한 참조를 계속 유지하기 때문이다.**

그 어떤 Context보다 오래 유지되는 Context가 필요할 때에만 Application Context를 사용해야 한다.

### Activity Context

- Activity lifecycle에 귀속됨
- `getContext()`로 접근
- Activity 범위 내에서 Context 전달

Activity Context는 Activity에서 사용 가능하고 Activity의 범위 내에서 Context를 전달하거나, lifecycle이 현재의 Context에 attached 된 Context가 필요할 때 사용한다.

예를 들어, Toast, Dialog 등의 UI operation에서 Context가 필요하다면 이 때 Activity Context를 사용해야 한다.

---

**항상 가능한 한 가까운 Context를 사용하자. Activity에 있다면 Activity Context를, Application에 있다면 Application Context를 사용하자.**

## Rule of Thumb

대부분의 경우 가장 가까운, 스코프에 해당하는 컨택스트를 직접 사용하세요. 참조가 해당 컴포넌트의 라이프사이클을 넘어서지 않는 이상 메모리 누수 걱정 없이 컴포넌트를 유지할 수 있습니다. 액티비티나 서비스 이외의 객체에서 컨택스트를 참조해야 하는 경우 액티비티 컨택스트가 아닌, 애플리케이션 컨텍스트로 전환하세요.
