---
layout: post
title: "[Android] tflite 모델로 수어 지문자 번역"
author: deBaeloper08
date: "2022-11-03 21:07:00 +0900"
categories: [Android]
tags: [mediapipe, tflite]
---

MediaPipe를 사용하여 얻은 손의 LandMark들을 갖고 수어 지문자를 번역하고자 한다. 모델 같은 경우 직접 학습하지 않고 학습된 모델을 사용했다.

[수어 모델 Github](https://github.com/YoungJo-YOO/Singlanguage_Kor)

<img src="https://velog.velcdn.com/images/debaeloper08/post/6fc473ad-5e40-4433-988d-509a0da1b0ba/image.png" width="70%"></img>res폴더에 Assets 폴더를 생성한 후 tflite 모델 파일을 Assets 폴더에 넣어준다.

```gradle
// Tensorflow
implementation 'org.tensorflow:tensorflow-lite:2.10.0'
implementation 'org.tensorflow:tensorflow-lite-task-vision-play-services:0.4.2'
implementation 'com.google.android.gms:play-services-tflite-gpu:16.0.0'
implementation 'org.tensorflow:tensorflow-lite-task-vision:0.4.0'
```

app gradle 파일에 해당 코드를 작성하고 sync를 해준다.

```kotlin
private fun getTfliteInterpreter(path: String): Interpreter? {
        try {
            return Interpreter(loadModelFile(this@MainActivity, path)!!)
        }
        catch (e: Exception) {
            e.printStackTrace()
        }
        return null
    }

private fun loadModelFile(activity: Activity, path: String): MappedByteBuffer? {
        val fileDescriptor = activity.assets.openFd(path)
        val inputStream = FileInputStream(fileDescriptor.fileDescriptor)
        val fileChannel = inputStream.channel
        val startOffset = fileDescriptor.startOffset
        val declaredLength = fileDescriptor.declaredLength
        return fileChannel.map(FileChannel.MapMode.READ_ONLY, startOffset, declaredLength)
    }
```

**getTfliteInterpreter** 함수는 path 경로에 있는 tflite 모델을 불러오는 함수다.

```kotlin
hands.setResultListener {
    translate(it)
    glSurfaceView.setRenderData(it)
    glSurfaceView.requestRender()
}
```

Mediapipe를 통해 얻은 결과물을 **translate** 함수에 인자값으로 넘겨준다.

```kotlin
private fun translate(result : HandsResult){
        if (result.multiHandLandmarks().isEmpty()) {
            return
        }
        val landmarkList = result.multiHandLandmarks()[0].landmarkList
        val joint = Array(21){FloatArray(3)}
        for(i in 0..19) {
            joint[i][0] = landmarkList[i].x
            joint[i][1] = landmarkList[i].y
            joint[i][2] = landmarkList[i].z
        }

        val v1 = joint.slice(0..19).toMutableList()
        for(i in 4..16 step(4)) {
            v1[i] = v1[0]
        }
        var v2 = joint.slice(1..20)
        val v = Array(20) { FloatArray(3) }

        for(i in 0..19) {
            for(j in 0..2) {
                v[i][j] = v2[i][j] - v1[i][j]
            }
        }

        for(i in 0..19) {
            val norm = sqrt(v[i][0] * v[i][0] + v[i][1] * v[i][1] + v[i][2] * v[i][2])
            for(j in 0..2) {
                v[i][j] /= norm
            }
        }

        val tmpv1 = mutableListOf<FloatArray>()
        for(i in 0..18) {
            if(i != 3 && i != 7 && i != 11 && i != 15) {
                tmpv1.add(v[i])
            }
        }
        val tmpv2 = mutableListOf<FloatArray>()
        for(i in 1..19) {
            if(i != 4 && i != 8 && i != 12 && i != 16) {
                tmpv2.add(v[i])
            }
        }

        val einsum = FloatArray(15)
        for( i in 0..14) {
            einsum[i] = tmpv1[i][0] * tmpv2[i][0] + tmpv1[i][1] * tmpv2[i][1] +
            	tmpv1[i][2] * tmpv2[i][2]
        }
        val angle = FloatArray(15)
        val data = FloatArray(15)
        for(i in 0..14) {
            angle[i] = Math.toDegrees(acos(einsum[i]).toDouble()).toFloat()
            data[i] = round(angle[i] * 100000) / 100000
        }

        val interpreter = getTfliteInterpreter("converted_model.tflite")
        val byteBuffer = ByteBuffer.allocateDirect(15*4).order(ByteOrder.nativeOrder())

        for(d in data) {
            byteBuffer.putFloat(d)
        }

        val modelOutput = ByteBuffer.allocateDirect(26*4).order(ByteOrder.nativeOrder())
        modelOutput.rewind()

        interpreter!!.run(byteBuffer,modelOutput)

        val outputFeature0 = TensorBuffer.createFixedSize(intArrayOf(1,26), DataType.FLOAT32)
        outputFeature0.loadBuffer(modelOutput)

        // ByteBuffer to FloatBuffer
        val outputsFloatBuffer = modelOutput.asFloatBuffer()
        val outputs = mutableListOf<Float>()
        for(i in 1..26) {
            outputs.add(outputsFloatBuffer.get())
        }

        val sortedOutput = outputs.sortedDescending()
        val index = outputs.indexOf(sortedOutput[0])

        Log.d("TAG", "translate: ${classes[index]}")
    }
```

**translate** 함수는 Mediapipe로 손의 LandMark 값들을 받아 수어로 번역하는 함수다. 필자가 사용한 모델의 경우 손의 LandMark들이 이루는 각도를 계산하여 수어를 번역한다. main.py에서 데이터를 가공하여 input에 넣는 과정을 kotlin으로 변환하여 입력값을 넣어주었다.

<img src="https://velog.velcdn.com/images/debaeloper08/post/3ab8521c-609b-4cdc-ae22-6d21557c0e88/image.png" width="30%"></img>

<img src="https://velog.velcdn.com/images/debaeloper08/post/ddc426b9-ee04-4007-a9c7-35851b3487ad/image.png" width="70%"></img>

이처럼 Log에 수어를 번역한 결과를 확인할 수 있다.
