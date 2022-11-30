---
title: "[Android] Application Class"
author: deBaeloper08
date: 2022-11-30 14:01:00 +0900
categories: [Android]
tags: [application]
---

# Application Class

## Application Class란?

Application Class는 어느 Component에서나 공유할 수 있는 전역 클래스다. Component들 사이에서 공동으로 관리할 데이터가 있다면 Application Class를 상속 받는 Class를 만든다. Class를 생성한 후, Manifest.xml에 등록하면 된다.

## Application Class 사용

Manifest.xml에 application 태그에 android:name 특성에 생성한 클래스 이름을 넣으면 된다.

override 할 수 있는 함수가 여러 개 있지만 그 중 3개만 소개하면
- onCreate : 앱이 생성될 때 호출된다.
- onLowMemory : 시스템의 리소스가 부족할 때 발생한다.
- onConfigurationChanged : 컴포넌트가 실행되는 동안 기기의 설정 정보가 변경될 때 시스템에서 호출된다.