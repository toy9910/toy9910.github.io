---
layout: post
title: "[Android] Clean Architecture"
author: deBaeloper08
date: "2022-11-03 21:10:00 +0900"
categories: [Android]
tags: [clean architecture]
---

# 🤔클린 아키텍처란?

> 클린 아키텍처는 『클린 코드(Clean Code)』를 저술한 로버트 마틴(Robert C.Martin)이 제안한 시스템 아키텍처로, 기존의 계층형 아키텍처가 가지던 의존성에서 벗어나도록 하는 설계 방법입니다.

클린 아키텍처에서는 **경계**를 가장 중요하게 생각한다. 로버트 마틴은 경계에 대해 다음과 같이 설명한다.

> 소프트웨어 아키텍처는 선을 긋는 기술이며, 나는 이러한 선을 경계(boundary)라고 부른다. 경계는 소프트웨어 요소를 서로 분리하고, **경계 한편에 있는 요소가 반대편에 있는 요소**를 알지 못하도록 막는다.
> Robert C. Martin, Clean Architecture

여기서 말하는 경계는 **계층**을 말한다.

## 그럼 계층이 무엇일까?

<img src="https://image.toast.com/aaaadh/real/2022/techblog/02%282%29.png" width="20%">
</img>

사진을 보면 시스템을 구성하는 영역을 세 계층으로 나누고 있다.

### 도메인 계층(Domain Layer)

앱의 중심부로써 이 계층에 포함된 비즈니스 로직은 앱을 구성하고 있는 것 중 가장 중요한 부분이다. 그래서 비즈니스 로직을 망쳐서는 안되기 때문에 이 계층은 **어떠한 계층에도 의존하지 않는다.** Domain 계층은 다음과 같은 코드를 포함한다.

- Entity : 특정 영역을 표현하는 객체. ex) Pojo, VO, DTO 등
- UseCase : Entity와 함께 비즈니스 로직을 수행한다.
- Repository 인터페이스 : 데이터베이스, 원격 서버와 같은 데이터 소스에 접근한다.

### 데이터 계층(Data Layer)

Data 계층은 데이터 소스(DB, 서버 등)와 상호작용을 담당하는 코드가 포함되는 곳이다. **Data 계층은 Domain 계층에 의존**한다.

앱은 아마 여러가지 데이터 소스를 사용할 텐데, 데이터 소스 또한 시간이 지남에 따라 변경될 수 있다. 예를 들면 REST 서버를 GraphQL 서버로 마이그레이션 하거나 또는 Realm DB를 RoomDB로 변경해야하는 경우를 말한다. 이러한 변경사항은 오로지 데이터를 처리하는데 관련된 로직일 뿐 **데이터를 필요로 하는 코드에는 영향을 미치지 않는다.**

Data 계층은 다음과 같은 두 가지 책임을 갖는다.

- 데이터 입출력 코드를 하나의 계층에서 관리한다.
- 데이터 소스들과 데이터를 소비하는 다른 계층과의 경계를 둔다.

Data 계층에서는 Domain 계층에서 정의한 Repository 인터페이스를 구현한다.
<img src="https://velog.velcdn.com/images/debaeloper08/post/20aa6e9d-b721-4ec3-95b3-9dcf83c0d4ca/image.png" width="40%"></img>
그림을 보면 Domain과 Data간의 분리가 이루어져 있기 때문에, 혹시나 데이터 소스를 변경해도 Domain 계층에는 영향이 없기 때문에 비즈니스 로직은 피해없이 안전하다.

### 프레젠테이션 계층(Presentation Layer)

Presentation 계층은 Domain, Data 계층에 의존하고 UI와 관련된 코드에 대해 다룬다. 모든 UI와 관련된 컴포넌트 또는 안드로이드 프레임워크와 관련된 코드들을 이 계층에서 다루게 된다.

---

계층을 횡단하면서 데이터를 전달할 때, 데이터는 항상 내부의 원에서 사용하기에 가장 편리한 형태를 가져야 한다. 예를 들어, 서버에 저장된 사용자 정보를 갱신하기 위해 다음과 같이 가정해보자.

- 사용자 정보를 갱신하기 위해 API를 호출한다.
- Domain 계층에서 사용자 정보는 User라는 형태로 표현한다.
- Presentation 계층에서 View에 사용자 정보를 나타날 때는 UserUiModel로 표현한다.
- Data 계층에서 서버로부터 응답받은 사용자 정보는 UserDto로 표현한다.

사용자 정보 갱신 API 호출을 위해 Activity에서 유즈케이스를 호출할 때 UserUiModel은 User로 변환되어야 한다. 또한 Retrofit을 통해 API요청에 따른 응답 UserDTO를 받았고, 이를 다시 유즈케이스로 넘기기 위해서는 User로 변경해야 한다. 흐름이 반대 방향일 때도 마찬가지로 변경이 필요하다. 이처럼 데이터 형태를 변경할 때 **Mapper**라고 하는 객체를 만들어 사용한다.

_참조_
https://www.charlezz.com/?p=45391
https://velog.io/@ashwon1218/Android-Clean-Architecture
