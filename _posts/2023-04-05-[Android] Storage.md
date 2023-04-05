---
title: "[Android] Storage"
author: deBaeloper08
date: 2023-04-05 14:01:00 +0900
categories: [Android]
tags: [storage, internal, external]
---

## Data and File Storage

안드로이드에서는 다른 플랫폼과 같이 디스크 기반의 파일 시스템을 사용하고 있다. 해당 시스템에는 다양한 옵션들이 존재한다.

- App-specific Storage
- Shared Storage
- Preferences
- Databases

### App-specific Storage

앱에서만 사용하는 파일을 저장한 공간이다. 내부 저장소에 저장된 파일도 있고 외부 저장소에 저장된 파일도 있을 수 있다. 만약 민감한 정보를 다른 앱에서 보지 않기를 원한다면 내주 저장소에 저장되어야한다.

- 파일 타입 : 해당 앱에서만 사용되는 파일들
- 접근 메소드 : 내부 저장소에 있는 파일은 ```getFilesDir()```, ```getCacheDir()``` 그리고 외부 저장소에 있는 파일은 ```getExternalFilesDir()```, ```getExternalCacheDir()```
- 필요한 권한 : 내부 저장소에 접근할 때는 권한 필요 없음, 외부 저장소에 접근할 때는 Android 4.4(API level 19)미만은 external storage에 대한 권한 필요하고 그 이상은 권한 필요 없음
- 다른 앱에서 접근 가능 여부 : ❌
- 앱을 지웠을 때 데이터 삭제 여부 : ⭕

### Shared Stroage

다른 앱들과 함께 공유하는 파일을 저장한 공간. 예를 들면, 사진, 동영상, 문서 등등을 저장한다.

- 파일 타입 : 다른 앱과 공유 가능한 파일들(사진, 동영상 등등)

영상의 경우

- 접근 메소드 : MediaStore API를 사용
- 필요한 권한
    - Android 11 (API level 30) 이상 : ```READ_EXTERNAL_STORAGE```
    - Android 10 (API level 29) : ```READ_EXTERNAL_STORAGE``` 또는 ```WRITE_EXTERNAL_STORAGE```
    - Android 9 (API level 28) 이하 : ```READ_EXTERNAL_STORAGE``` 와 ```WRITE_EXTERNAL_STORAGE``` 모두
- 다른 앱에서 접근 가능 여부 : 다른 앱에서 ```READ_EXTERNAL_STORAGE``` 권한을 허용했을 경우 ⭕
- 앱을 지웠을 때 데이터 삭제 여부 : ❌

일반 파일의 경우
- 접근 메소드 : Storage Access Framework 사용
- 필요한 권한 : ❌
- 다른 앱에서 접근 가능 여부 : System File Picker를 사용하여 ⭕
- 앱을 지웠을 때 데이터 삭제 여부 : ❌


### Preferences

key-value 형태로 private, primitive한 정보 저장

- 파일 타입 : key-value 형태로 된 데이터
- 접근 메소드 : Jetpack Preferences Library
- 필요한 권한 : ❌
- 다른 앱에서 접근 가능 여부 :  ❌
- 앱을 지웠을 때 데이터 삭제 여부 : ⭕


## Databases

Room persistence library를 사용하여 private database에 정보 저장

- 파일 타입 : Structured Data
- 접근 메소드 : Room persistence library
- 필요한 권한 : ❌
- 다른 앱에서 접근 가능 여부 :  ❌
- 앱을 지웠을 때 데이터 삭제 여부 : ⭕

----

안드로이드에서는 앱 데이터가 저장되는 영역을 내부 저장소, 사진이나 동영상 등을 저장하는 영역을 외부 저장소라고 한다.

## Internal Storage(내부 저장소)

내부 저장소는 별도의 permission 추가 없이 사용할 수 있는 저장장치다. 내부 저장소에 저장된 데이터는 자신의 앱에서만 접근 가능하며, 사용자가 앱을 삭제할 경우 시스템이 내부 저장소의 앱 데이터를 모두 삭제한다. 따라서, 다른 앱에서 자신의 데이터에 접근할 수 없도록 하고 싶을 경우에 사용하는 것이 가장 적합하다. 내부 저장소에 데이터를 저장할 때는 ```openFileOutput()``` 함수를 사용한다.

## External Storage(외부 저장소)

외부 저장소는 내부 저장소와는 달리 사용하기 위해서는 파일을 읽고 쓰는 permission이 필요하다. 외부 저장소는 항상 사용 가능한 것이 아니고 사용자가 sdcard 등의 외부 저장소를 mount 했을 경우에만 사용 가능하다. 외부 저장소가 사용 가능한지에 대한 여부는 ```getExternalStorageState()``` 함수를 사용하여 return값이 MEDIA_MOUNTED면 외부 저장소에 read/write가 가능한 것이다. 외부 저장소는 내부 저장소와 다르게 모든 앱에서 데이터를 읽을 수 있기 때문에 다른 앱과 공유하고 싶은 데이터를 저장하기 적합하다.

### Android 9 (API level 28)이하

외부 저장소는 Android 9 (API level 28)이하에서는 스토리지를 legacy storage라고 불렀고 구조는
- 외부 저장소
    - public(공유 공간)
    - private(개별 공간)
        - app packages(각각의 패키지)

와 같고 권한만 있다면 데이터 접근이 가능했다.

### Android 10 (API level 29)이상

스토리지를 scoped storage라고 부르며 구조는
- 외부 저장소
    - private
    - public
        - 이미지 비디오
        - 오디오
        - 다운로드 파일

private은 권한이 없어도 접근할 수 있고 다른 앱의 저장 공간에 접근할 수 없다.
public은
1. 해당 파일 형식에 맞는 데이터만 저장 가능
2. 다운로드 파일은 SAF(Storage Access Framework)로 접근
3. public 공간은 MediaStore API로 접근 권장