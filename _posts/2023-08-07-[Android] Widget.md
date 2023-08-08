---
title: "[Android] Widget"
author: deBaeloper08
date: 2023-08-08 09:27:00 +0900
categories: [Android]
tags: [widget]
---

## 위젯의 구성요소

- ```AppWidgetProviderInfo```

    위젯의 레이아웃, 업데이트 주기, ```AppWidgetProvider``` 클래스와 같은 위젯이 사용하는 메타데이터를 포함한다.

- ```AppWidgetProvider```

    위젯에서 사용하는 기본 함수들을 정의한다. 이를 통해, 위젯이 updated, enabled, disabled, deleted 될 때 브로드캐스트를 수신한다. ```AppWidgetProvider```는 ```manifest```에 선언하고 구현한다.

- ```View layout```

    위젯의 초기 레이아웃을 정의한다.


## 위젯의 워크플로우

![image](https://github.com/toy9910/toy9910.github.io/assets/50603217/6f0bd854-48c7-4bc3-b29b-558096656513)

###### 출처: https://developer.android.com/develop/ui/views/appwidgets

## 위젯 사용방법

#### 1. ```AppWidgetProviderInfo``` XML을 선언한다.

```AppWidgetProviderInfo```는 위젯을 생성하는데 있어 필요한 요소들을 정의한다. ```<appwidget-provier>``` 항목을 사용하여 XML 리소스 파일 내 ```AppWidgetProviderInfo``` 객체를 생성하고 ```res/xml/``` 폴더에 저장한다.

```xml
<appwidget-provider xmlns:android="http://schemas.android.com/apk/res/android"
    android:minWidth="40dp"
    android:minHeight="40dp"
    android:targetCellWidth="1"
    android:targetCellHeight="1"
    android:maxResizeWidth="250dp"
    android:maxResizeHeight="120dp"
    android:updatePeriodMillis="86400000"
    android:description="@string/example_appwidget_description"
    android:previewLayout="@layout/example_appwidget_preview"
    android:initialLayout="@layout/example_loading_appwidget"
    android:configure="com.example.android.ExampleAppWidgetConfigurationActivity"
    android:resizeMode="horizontal|vertical"
    android:widgetCategory="home_screen"
    android:widgetFeatures="reconfigurable|configuration_optional">
</appwidget-provider>
```

#### 2. ```AppWidgetProvider``` 클래스를 ```AndroidManifest.xml```에 선언한다.

```xml
<receiver android:name="ExampleAppWidgetProvider"
                 android:exported="false">
    <intent-filter>
        <action android:name="android.appwidget.action.APPWIDGET_UPDATE" />
    </intent-filter>
    <meta-data android:name="android.appwidget.provider"
               android:resource="@xml/example_appwidget_info" />
</receiver>
```

```<intent-filter>``` 항목에는 ```android:name``` 속성과 함께 ```action``` 항목이 반드시 포함되어야 한다. 해당 속성과 항목의 뜻은, ```APPWIDGET_UPDATE``` 브로드캐스트 수신에 동의한다는 뜻이다. 해당 브로드캐스트 외에는 ```AppWidgetManager```가 자동으로 브로드캐스트를 송신하기 때문에 추가로 명시적으로 작성할 브로드캐스트는 없다.

```<meta-data>``` 항목에는 ```AppWidgetProviderInfo```에 대한 리소스와 다음 속성들이 포함된다.
- ```android-name``` : 메타데이터의 이름을 뜻한다. ```android.widget.provider```를 통해 해당 데이터를 확인할 수 있다.
- ```android:resource``` : ```AppWidgetProviderInfo``` 리소스의 위치를 뜻한다.

#### 3. ```AppWidgetProvider``` 클래스를 구현한다.

```AppWidgetPfovider``` 클래스는 위젯 브로드캐스트를 다루기 위해 ```BroadcastReceiver``` 클래스를 상속한다. ```AppWidgetProvider```는 위젯이 updated, deleted, enabled, disabled 와 같은 **위젯과 관련된 이벤트 브로드캐스트만을 수신한다.** 이와 같은 브로드캐스트가 발생하면, ```AppWidgetProvider```는 다음 함수들을 실행한다.

- ```onUpdate()```

    해당 함수는 AppWidgetProviderInfo의 속성 ```updatePeriodMillis```가 설정되어 특정 간격으로 위젯을 update 하도록 하였을 때 실행된다. 
    
    사용자가 해당 위젯을 추가했을 때도 실행되기 때문에 중요하거나 필요한 셋팅의 기능을 수행해야 한다. 예를 들면, event handler를 정의한다거나 위젯에 표시할 데이터들을 불러오는 작업들이 있을 것이다.

    하지만, configuration activity를 ```configuration_optional``` 플래그 없이 선언한다면 ```onUpdate()``` 함수는 사용자가 위젯을 추가할 때 실행되지 않고 후속 업데이트용으로 사용된다.


- ```onAppWidgetOptionsChanged()```

    해당 함수는 위젯이 처음으로 배치되었을 때와 위젯의 크기가 변경됐을 때 실행된다. 위젯의 크기에 따라 어떤 내용을 보여줄지 정하고자 할 때 사용한다.

    Android 12부터는 ```getAppWidgetOptions()``` 함수를 통해 위젯의 가능한 크기들을 ```Bundle```의 형태로 받을 수 있다. ```Bundle```에는 다음 항목들이 포함된다.
    - ```OPTION_APPWIDGET_MIN_WIDTH```
    - ```OPTION_APPWIDGET_MIN_HEIGHT```
    - ```OPTION_APPWIDGET_MAX_WIDTH```
    - ```OPTION_APPWIDGET_MAX_HEIGHT```
    - ```OPTION_APPWIDGET_SIZES``` : Android 12에서 나온 것으로 가능한 크기들을 포함하고 있다.

    fke

- ```onDeleted(Context, Int[])```

    해당 함수는 위젯이 삭제 될 때마다 호출된다.

- ```onEnalbed(Context)```

    해당 함수는 위젯 객체가 처음 생성되었을 때 호출된다. 예를 들어, 사용자가 두 개의 위젯을 추가했으면 함수는 한 번 호출된다.

- ```onDisabled(Context)```

    해당 함수는 마지막 위젯 객체가 삭제 될 때 호출된다. ```onEnabled(Context)``` 함수에서 작업했던 것들을 마무리하고 정리하는 코드가 들어가면 좋다.

- ```onReceived(Context, Intent)```

    해당 함수는 모든 브로드캐스트와 콜백 함수들 전에 실행된다. 기본적으로 ```AppWidgetProvider```가 위젯의 모든 브로드캐스트를 필터링하기 때문에 굳이 해당 함수를 구현할 필요는 없다.

#### 4. ```onUpdate()``` 함수로 이벤트들을 구현한다.

 ```AppWidgetProvider```의 콜백 함수 중 ```onUpdate()``` 함수는 각각의 위젯이 추가 될 때마다 실행이 되기 때문에 가장 중요하다. 만약, 사용자와 상호작용하여 특정 이벤트를 실행하고자 한다면 이벤트 핸들러를 ```onUpdate()```에 등록해야 한다.

 예를 들어, 위젯의 버튼을 눌렀을 때 특정 액티비티가 실행되도록 하고자 한다면 다음의 ```AppWidgetProvider``` 구현한 것을 참고하면 좋을 것이다.

```kotlin
class ExampleAppWidgetProvider : AppWidgetProvider() {

    override fun onUpdate(
            context: Context,
            appWidgetManager: AppWidgetManager,
            appWidgetIds: IntArray
    ) {
        // Perform this loop procedure for each widget that belongs to this
        // provider.
        appWidgetIds.forEach { appWidgetId ->
            // Create an Intent to launch ExampleActivity.
            val pendingIntent: PendingIntent = PendingIntent.getActivity(
                    /* context = */ context,
                    /* requestCode = */  0,
                    /* intent = */ Intent(context, ExampleActivity::class.java),
                    /* flags = */ PendingIntent.FLAG_UPDATE_CURRENT or PendingIntent.FLAG_IMMUTABLE
            )

            // Get the layout for the widget and attach an on-click listener
            // to the button.
            val views: RemoteViews = RemoteViews(
                    context.packageName,
                    R.layout.appwidget_provider_layout
            ).apply {
                setOnClickPendingIntent(R.id.button, pendingIntent)
            }

            // Tell the AppWidgetManager to perform an update on the current
            // widget.
            appWidgetManager.updateAppWidget(appWidgetId, views)
        }
    }
}
```

해당 ```AppWidgetProvider```는 ```setOnClickPendingIntent(int, PendingIntent)```로 위젯의 버튼을 누르면 특정 액티비티를 실행하도록 하는 ```PendingIntent```를 만들어 ```onUpdate()``` 함수만 정의하였다.

참고로 ```onUpdate()``` 내에는 각각의 위젯 ID를 담은 배열 ```appWidgetIds```을 탐색하는 반복문이 있다. 만약, 사용자가 두 개 이상의 위젯을 생성하면 위젯들은 동시에 모두 업데이트 될 것이다. 하지만, 오직 한 개의 ```updatePeriodMillis``` 스케쥴만 모든 위젯을 관리한다. 예를 들어, 업데이트 작업이 매 2시간마다 실행되도록 구현하고 두 번째 위젯이 첫 번째 위젯이 생성되고 나서 1시간 뒤에 생성되면 첫 번째 위젯의 업데이트 작업에 맞춰서 작업이 실행되고 두 번째로 등록된 업데이트 작업은 무시된다.