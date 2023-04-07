---
title: "[Android] onBackPressedDispatcher"
author: deBaeloper08
date: 2023-04-07 14:01:00 +0900
categories: [Android]
tags: [onbackpresseddispatcher, onbackpressed]
---

## onBackPressed Deprecated

Android API Level 33부터 ```onBackPressed``` 함수가 deprecated 되었다. 공식 문서에서는 이제 OnBackPressedDispatcher를 통해 뒤로가기 기능을 구현하는 것을 권장한다.

## onBackPressedDispatcher

OnBackPressedCallback 변수를 만들어 OnBackPressedCallback 내에 있는 handleOnBackPressed 함수를 구현한다. 해당 callback을 onBackPressedDispatcher에 추가한다.

이렇게 되면 뒤로가기 버튼을 누르면 callback 함수에 정의된 ```handleOnBackPressed``` 함수가 실행된다.

addCallback 함수의 인자로 들어오는 것은 lifecycleowner와 callback 함수다.
첫 번째 인자로 lifecycleowner를 정의하여 해당 lifecycle에서만 실행되도록 할 수 있다.

```kotlin
private lateinit var callback: OnBackPressedCallback

callback = object : OnBackPressedCallback(true) {
            override fun handleOnBackPressed() {

                test++
                Log.d("TAG", "handleOnBackPressed: $test")
                if(test>5) {
                    this.remove()
                }
            }
        }

onBackPressedDispatcher.addCallback(this, callback)
```


위 코드가 MainActivity 코드고 MainActivity2에서 MainActivity를 상속받으면 OnBackPressedCallback도 같이 상속받는다.

MainActivity2에서 뒤로가기를 누르면 test 변수가 올라가는 것을 확인할 수 있다.

만약, MainActivity2만의 뒤로가기를 구현하고 싶으면 MainActivity의 ```onResume```과 ```onPause```에 callback 함수를 등록하고 해제하는 코드를 짜야한다.

MainActivity 코드

```kotlin
override fun onResume() {
        super.onResume()
        addCallback()
    }

    open fun addCallback() {
        Log.d("TAG", "handleOnBackPressed: resume")
        onBackPressedDispatcher.addCallback(this, callback)
    }

    override fun onPause() {
        super.onPause()
        removeCallback()
    }

    open fun removeCallback() {
        Log.d("TAG", "handleOnBackPressed: pause")
        callback.remove()
    }
```

MainActivity2 코드
```kotlin
    override fun addCallback() {
        Log.d("TAG", "handleOnBackPressed2: resume1")
        onBackPressedDispatcher.addCallback(this, callback2)
    }

    override fun removeCallback() {
        Log.d("TAG", "handleOnBackPressed2: pause")
        callback2.remove()
    }
```
이러면 MainActivity의 화면이 출력될 때 callback이 등록되고 ```onPause```가 실행될 때 등록되었던 callback이 해제된다. 그리고 MainActivity2가 실행될 때는 callback2가 등록되고 종료될 때 callback2가 해제된다.
