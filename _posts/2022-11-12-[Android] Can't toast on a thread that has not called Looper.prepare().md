---
title: "[Android] java.lang.RuntimeException: Can't toast on a thread that has not called Looper.prepare()"
author: deBaeloper08
date: 2022-11-12 23:52:00 +0900
categories: [Android, Error]
tags: [toast]
---

# `java.lang.RuntimeException: Can't toast on a thread that has not called Looper.prepare()` 에러

해당 에러 같은 경우 Toast 메시지를 Main Thread가 아닌 다른 Thread에서 띄우려고 해서 나오는 에러다.

이 경우 `runOnUiThread`를 사용하면 된다.

```kotlin
runOnUiThread{
    Toast.makeText(
        getApplicationContext().getApplicationContext(),
        getApplication().getString("sample toast message"), Toast.LENGTH_SHORT)
        .show()
}
```

만약 코루틴을 사용하여 `Dispatchers.IO`에서 작업하고 있었다면 `withContext(Dispatchers.Main)`를 사용하여 작업 환경을 메인으로 바꾼후 토스트를 출력하면 된다.

```kotlin
CoroutineScope(Dispatchers.IO).launch {
    ...
    withContext(Dispatchers.Main) {
        // Toast 출력
    }
}
```
