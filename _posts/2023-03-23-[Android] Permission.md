---
title: "[Android] Permission"
author: deBaeloper08
date: 2023-03-23 18:01:00 +0900
categories: [Android]
tags: [permission]
---

## 권한이란?

권한은 앱에서 제한된 데이터 또는 제한된 작업에 대한 액세스가 필요할 수 있는 기능을 제공하는 경우에 요청하여 작업을 수행할 수 있는지 확인한다.

Api 22 이하에서는 manifest.xml 파일에 필요한 권한들을 명시하고 앱을 설치할 때 해당 권한들을 한 번 보여줬다. 그러면서 사용자는 해당 권한이 무엇을 의미하고 어디에 쓰이는지도 모른 체 사용하게 되어 악의적으로 배포되는 앱이 생겨났다. 이를 해결하고자 Api 23 부터는 앱이 실행된 뒤 사용자에게 직접 승인을 받아야 해당 권한과 관련된 리소스에 접근할 수 있도록 변경됐다.

## 권한의 보호수준(Protection level)

Protection level은 Permission의 속성 중 하나로서 &quot;_권한에 내포된 잠재적 위험을 특성화하고 권한을 요청하는 앱에 권한을 부여할지 여부를 결정할 때 시스템에서 따라야 하는 절차_&quot;를 뜻한다.

Protection level의 종류로는

- **normal** : 사용자들의 사생활에 직접적인 위험이 있지않다. 만약 앱의 manifest에 normal 권한을 명시하였다면, 시스템은 자동적으로 권한을 허용한다.
- **dangerous** : 앱이 사용자의 민감한 데이터에 접근할 수 있도록 한다. 만약 dangerous 권한을 명시하였다면, 사용자의 승인을 받도록 앱에 명시해야 한다.
- **signature** : 이미 권한을 승인한 앱이 업데이트 되었을 경우 사용자에게 알리거나 사용자의 명시적인 승인을 요청하지 않고 자동으로 권한을 부여한다.

## 권한 요청

```shouldShowRequestPermissionRationale``` 함수는 _사용자가 이전에 권한 요청을 거부한 경우, 해당 권한이 필요한 이유를 설명하고 추가적인 설명이 필요한지 여부를 확인하기 위해_ 호출된다.

예를 들어, 카메라 권한을 필요로 하는 앱을 설치했다고 가정하자. 앱을 처음에 실행했을 때는 ```shouldShowRequestPermissionRationale``` 함수는 false 값을 반환할 것이고, 카메라 권한을 허용한 후에도 ```shouldShowRequestPermissionRationale``` 함수는 false 값을 반환할 것이다.

하지만 카메라 권한을 허용하지 않고 ```shouldShowRequestPermissionRationale``` 함수를 호출하면 해당 권한이 필요한 이유를 설명해야 하기 때문에 true 값을 반환한다.

따라서, 권한 요청을 할 때 ```shouldShowRequestPermissionRationale``` 함수를 사용하여 반환된 값에 따라 설명이 필요한지 여부를 확인하고, 설명이 필요한 경우에는 사용자에게 권한에 대한 추가적인 설명을 제공해야 한다.

## 예시

1. Manifest.xml에 요청하고자 하는 권한을 명시한다.

```xml
<uses-permission android:name="android.permission.CAMERA"/>
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION"/>
```

2. TedPermission 라이브러리를 implement 한다.

```gradle
// TedPermission
    implementation 'io.github.ParkSangGwon:tedpermission-normal:3.3.0'
```

3. permission 요청

```kotlin
private fun requestPermission() {
        TedPermission.create().setPermissionListener(object : PermissionListener {
            override fun onPermissionGranted() {
                Toast.makeText(this@MainActivity, "완료", Toast.LENGTH_SHORT).show()
            }

            override fun onPermissionDenied(deniedPermissions: MutableList<String>?) {
                Toast.makeText(this@MainActivity,
                    "권한을 허가해주세요.",
                    Toast.LENGTH_SHORT)
                    .show()
            }

        })
            .setPermissions(
                *arrayOf(
                Manifest.permission.CAMERA,
                    Manifest.permission.ACCESS_FINE_LOCATION) )
            .check()
    }
```
