---
layout: post
title: "[Android] MediaPipe를 통해 손 LandMark 추출"
author: deBaeloper08
date: "2022-11-03 21:04:00 +0900"
categories: [Android]
tags: [mediapipe]
---

최근 프로젝트에서 수어를 통역하는 기능을 구현해야 해서 MediaPipe를 사용하여 손의 LandMark를 추출해야 했다. 사실 필자도 코드를 완벽하게 이해하진 못했지만 기능을 성공적으로 구현했기에 어떻게 했는지 공유하고자 글을 쓴다.

---

```gradle
implementation 'com.google.mediapipe:solution-core:latest.release'
implementation 'com.google.mediapipe:hands:latest.release'
```

우선 app gradle 파일에 해당 코드를 추가하고 sync를 한다.

```xml
<FrameLayout
        android:id="@+id/preview_display_layout"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        app:layout_constraintTop_toBottomOf="@id/cl_voice"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintBottom_toTopOf="@id/cl_record">
</FrameLayout>
```

xml에 FrameLayout을 생성한다.

```kotlin
/** A custom implementation of [ResultGlRenderer] to render [HandsResult].  */
class HandsResultGlRenderer : ResultGlRenderer<HandsResult?> {
    private var program = 0
    private var positionHandle = 0
    private var projectionMatrixHandle = 0
    private var colorHandle = 0
    private fun loadShader(type: Int, shaderCode: String): Int {
        val shader = GLES20.glCreateShader(type)
        GLES20.glShaderSource(shader, shaderCode)
        GLES20.glCompileShader(shader)
        return shader
    }

    override fun setupRendering() {
        program = GLES20.glCreateProgram()
        val vertexShader = loadShader(GLES20.GL_VERTEX_SHADER, VERTEX_SHADER)
        val fragmentShader = loadShader(GLES20.GL_FRAGMENT_SHADER, FRAGMENT_SHADER)
        GLES20.glAttachShader(program, vertexShader)
        GLES20.glAttachShader(program, fragmentShader)
        GLES20.glLinkProgram(program)
        positionHandle = GLES20.glGetAttribLocation(program, "vPosition")
        projectionMatrixHandle = GLES20.glGetUniformLocation(program, "uProjectionMatrix")
        colorHandle = GLES20.glGetUniformLocation(program, "uColor")
    }

    override fun renderResult(result: HandsResult?, projectionMatrix: FloatArray) {
        if (result == null) {
            return
        }
        GLES20.glUseProgram(program)
        GLES20.glUniformMatrix4fv(projectionMatrixHandle, 1, false, projectionMatrix, 0)
        GLES20.glLineWidth(CONNECTION_THICKNESS)
        val numHands = result.multiHandLandmarks().size
        for (i in 0 until numHands) {
            val isLeftHand = result.multiHandedness()[i].label == "Left"
            drawConnections(
                result.multiHandLandmarks()[i].landmarkList,
                if (isLeftHand) LEFT_HAND_CONNECTION_COLOR else RIGHT_HAND_CONNECTION_COLOR
            )
            for(ind in result.multiHandLandmarks()[i].landmarkList.indices) {
                val lm = result.multiHandLandmarks()[i].landmarkList[ind]
                Log.d(TAG, "LandMark[$ind] | x : ${lm.x}, y : ${lm.y}, z : ${lm.z}")
            }
            for (landmark in result.multiHandLandmarks()[i].landmarkList) {
                // Draws the landmark.
                drawCircle(
                    landmark.x,
                    landmark.y,
                    if (isLeftHand) LEFT_HAND_LANDMARK_COLOR else RIGHT_HAND_LANDMARK_COLOR
                )
                //Log.d(TAG, "LandMark | x : ${landmark.x}, y : ${landmark.y}, z : ${landmark.z}")
                // Draws a hollow circle around the landmark.
                drawHollowCircle(
                    landmark.x,
                    landmark.y,
                    if (isLeftHand) LEFT_HAND_HOLLOW_CIRCLE_COLOR
                    else RIGHT_HAND_HOLLOW_CIRCLE_COLOR
                )
            }
        }
    }

    /**
     * Deletes the shader program.
     *
     *
     * This is only necessary if one wants to release the program
       while keeping the context around.
     */
    fun release() {
        GLES20.glDeleteProgram(program)
    }

    private fun drawConnections(
        handLandmarkList: List<NormalizedLandmark>,
        colorArray: FloatArray
    ) {
        GLES20.glUniform4fv(colorHandle, 1, colorArray, 0)
        for (c in Hands.HAND_CONNECTIONS) {
            val start = handLandmarkList[c.start()]
            val end = handLandmarkList[c.end()]
            val vertex = floatArrayOf(start.x, start.y, end.x, end.y)
            val vertexBuffer = ByteBuffer.allocateDirect(vertex.size * 4)
                .order(ByteOrder.nativeOrder())
                .asFloatBuffer()
                .put(vertex)
            vertexBuffer.position(0)
            GLES20.glEnableVertexAttribArray(positionHandle)
            GLES20.glVertexAttribPointer(positionHandle, 2, GLES20.GL_FLOAT,
            	false, 0, vertexBuffer)
            GLES20.glDrawArrays(GLES20.GL_LINES, 0, 2)
        }
    }

    private fun drawCircle(x: Float, y: Float, colorArray: FloatArray) {
        GLES20.glUniform4fv(colorHandle, 1, colorArray, 0)
        val vertexCount = NUM_SEGMENTS + 2
        val vertices = FloatArray(vertexCount * 3)
        vertices[0] = x
        vertices[1] = y
        vertices[2] = 0f
        for (i in 1 until vertexCount) {
            val angle = 2.0f * i * Math.PI.toFloat() / NUM_SEGMENTS
            val currentIndex = 3 * i
            vertices[currentIndex] = x + (LANDMARK_RADIUS * Math.cos(angle.toDouble())).toFloat()
            vertices[currentIndex + 1] =
                y + (LANDMARK_RADIUS * Math.sin(angle.toDouble())).toFloat()
            vertices[currentIndex + 2] = 0f
        }
        val vertexBuffer = ByteBuffer.allocateDirect(vertices.size * 4)
            .order(ByteOrder.nativeOrder())
            .asFloatBuffer()
            .put(vertices)
        vertexBuffer.position(0)
        GLES20.glEnableVertexAttribArray(positionHandle)
        GLES20.glVertexAttribPointer(positionHandle, 3, GLES20.GL_FLOAT,
        	false, 0, vertexBuffer)
        GLES20.glDrawArrays(GLES20.GL_TRIANGLE_FAN, 0, vertexCount)
    }

    private fun drawHollowCircle(x: Float, y: Float, colorArray: FloatArray) {
        GLES20.glUniform4fv(colorHandle, 1, colorArray, 0)
        val vertexCount = NUM_SEGMENTS + 1
        val vertices = FloatArray(vertexCount * 3)
        for (i in 0 until vertexCount) {
            val angle = 2.0f * i * Math.PI.toFloat() / NUM_SEGMENTS
            val currentIndex = 3 * i
            vertices[currentIndex] =
                x + (HOLLOW_CIRCLE_RADIUS * Math.cos(angle.toDouble())).toFloat()
            vertices[currentIndex + 1] =
                y + (HOLLOW_CIRCLE_RADIUS * Math.sin(angle.toDouble())).toFloat()
            vertices[currentIndex + 2] = 0f
        }
        val vertexBuffer = ByteBuffer.allocateDirect(vertices.size * 4)
            .order(ByteOrder.nativeOrder())
            .asFloatBuffer()
            .put(vertices)
        vertexBuffer.position(0)
        GLES20.glEnableVertexAttribArray(positionHandle)
        GLES20.glVertexAttribPointer(positionHandle, 3, GLES20.GL_FLOAT,
        	false, 0, vertexBuffer)
        GLES20.glDrawArrays(GLES20.GL_LINE_STRIP, 0, vertexCount)
    }

    companion object {
        private const val TAG = "HandsResultGlRenderer"
        private val LEFT_HAND_CONNECTION_COLOR = floatArrayOf(0.2f, 1f, 0.2f, 1f)
        private val RIGHT_HAND_CONNECTION_COLOR = floatArrayOf(1f, 0.2f, 0.2f, 1f)
        private const val CONNECTION_THICKNESS = 25.0f
        private val LEFT_HAND_HOLLOW_CIRCLE_COLOR = floatArrayOf(0.2f, 1f, 0.2f, 1f)
        private val RIGHT_HAND_HOLLOW_CIRCLE_COLOR = floatArrayOf(1f, 0.2f, 0.2f, 1f)
        private const val HOLLOW_CIRCLE_RADIUS = 0.01f
        private val LEFT_HAND_LANDMARK_COLOR = floatArrayOf(1f, 0.2f, 0.2f, 1f)
        private val RIGHT_HAND_LANDMARK_COLOR = floatArrayOf(0.2f, 1f, 0.2f, 1f)
        private const val LANDMARK_RADIUS = 0.008f
        private const val NUM_SEGMENTS = 120
        private const val VERTEX_SHADER = ("uniform mat4 uProjectionMatrix;\n"
                + "attribute vec4 vPosition;\n"
                + "void main() {\n"
                + "  gl_Position = uProjectionMatrix * vPosition;\n"
                + "}")
        private const val FRAGMENT_SHADER = ("precision mediump float;\n"
                + "uniform vec4 uColor;\n"
                + "void main() {\n"
                + "  gl_FragColor = uColor;\n"
                + "}")
    }
}
```

**HandsResultGlRenderer** 클래스를 생성한다. 만약 LandMark를 화면에 띄우는걸 원하지 않는다면 **drawConnections** 함수, **drawCircle** 함수, **drawHollowCircle** 함수를 지우고 **renderResult** 함수 내에 방금 3가지 함수를 지우면 된다.

```xml
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.CAMERA" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
```

Manifest에 해당 권한들을 추가한다.

```kotlin
private lateinit var hands : Hands
private lateinit var cameraInput: CameraInput
private lateinit var glSurfaceView: SolutionGlSurfaceView<HandsResult>

private val REQUIRED_PERMISSIONS = mutableListOf(Manifest.permission.INTERNET,
	Manifest.permission.RECORD_AUDIO, Manifest.permission.CAMERA,
	Manifest.permission.ACCESS_NETWORK_STATE).toTypedArray()
```

전역 변수로 필요한 변수들을 선언한다.

```kotlin
val permissionListener = object: PermissionListener {
        override fun onPermissionGranted() {
            setupStreamingModePipeline()

            glSurfaceView.post { startCamera() }
            glSurfaceView.visibility = View.VISIBLE

            initView()
        }

        override fun onPermissionDenied(deniedPermissions: MutableList<String>?) {
            Toast.makeText(requireContext(), "권한을 다시 설정해주세요!", Toast.LENGTH_SHORT).show()
        }

    }

TedPermission.create()
    .setPermissionListener(permissionListener)
    .setPermissions(*REQUIRED_PERMISSIONS)
    .check()
```

권한이 모두 허용이 되었다면 onCreate에 셋팅을 해준다.

```kotlin
private fun setupStreamingModePipeline() {
        hands = Hands(
            this@MainActivity,
            HandsOptions.builder()
                .setStaticImageMode(false)
                .setMaxNumHands(1)
                .setRunOnGpu(true)
                .build()
        )
        hands.setErrorListener { message, e -> Log.e("TAG", "MediaPipe Hands error: $message") }

        cameraInput = CameraInput(this@MainActivity)
        cameraInput.setNewFrameListener { hands.send(it) }

        glSurfaceView = SolutionGlSurfaceView(this@MainActivity, hands.glContext,
        	hands.glMajorVersion)
        glSurfaceView.setSolutionResultRenderer(HandsResultGlRenderer())
        glSurfaceView.setRenderInputImage(true)

        hands.setResultListener {
            glSurfaceView.setRenderData(it)
            glSurfaceView.requestRender()
        }

        glSurfaceView.post(this::startCamera)

		// activity_main.xml에 선언한 FrameLayout에 화면을 띄우는 코드
        binding.previewDisplayLayout.apply {
            removeAllViewsInLayout()
            addView(glSurfaceView)
            glSurfaceView.visibility = View.VISIBLE
            requestLayout()
        }
    }

private fun startCamera() {
    cameraInput.start(
        this@MainActivity,
        hands.glContext,
        CameraInput.CameraFacing.FRONT,
        glSurfaceView.width,
        glSurfaceView.height
    )
}
```

![](https://velog.velcdn.com/images/debaeloper08/post/f5c6759c-f3f2-4266-a95e-8b03a2ef76ff/image.png)이렇게 하면 성공적으로 FrameLayout에 자신의 손을 실시간으로 인식하는 영상을 볼 수 있을 것이다. 코드를 이해해가면서 작성하는 것을 추구하지만 프로젝트를 하면서 기능 구현할 시간이 많지 않았기에 해당 코드를 완벽하게 이해하진 못했다.
![](https://velog.velcdn.com/images/debaeloper08/post/3a73c8cd-4a01-42c3-b34f-914f001ee573/image.png)Log에는 손의 21개 LandMark들의 x,y,z 값을 확인할 수 있다.

손 말고 얼굴 인식도 아래 코드를 참고해서 하면 될 것 같다.
[Mediapipe Github 참고자료](https://github.com/google/mediapipe/tree/master/mediapipe/examples/android/solutions)
