# CameraX的优势

## 不同设备的兼容性和一致性

兼容5.0以上的设备，Google有设立自动化CameraX测试实验室，保证各个系统版本下的相机行为的一致性。

## 易用性

简化不同需求下的实现，将使用象棋的case分为四种用例：预览、图片分析（ml套件）、拍照、拍视频。

## 相机扩展

提供一个可选的 Extensions 库，用来简单实现一些原生相机的特性和功能，比如焦外成像（人像）、高动态范围 (HDR)、夜间模式和脸部照片修复功能。



# CameraX的使用要求

CameraX 具有以下最低版本要求：

- Android API 级别 21
- Android 架构组件 1.1.1

对于能够感知生命周期的 Activity，请使用 `FragmentActivity` 或 `AppCompatActivity`。



# CameraX架构

## 用例

将四种典型的case抽象成四种用例：

- **预览**：接受用于显示预览的 Surface，例如 `PreviewView`。
- **图片分析**：为分析（例如机器学习）提供 CPU 可访问的缓冲区。
- **图片拍摄**：拍摄并保存照片。
- **视频拍摄**：通过 `VideoCapture`拍摄视频和音频

不同用例可以组合使用，也可以同时处于活跃状态



## API

不同的用例会有不同的api，但通常使用步骤如下：

- 配置用例
- 添加 lisenter 处理输出数据
- 绑定 Lifecycle 让 CameraX自动处理相机的启动、关闭及生成数据的时机。

以下是一个预览用例与 `Surface` 互动进行创建和配置的示例：

```kotlin
val preview = Preview.Builder().build()
val viewFinder: PreviewView = findViewById(R.id.previewView)

// The use case is bound to an Android Lifecycle with the following code
val camera = cameraProvider.bindToLifecycle(lifecycleOwner, cameraSelector, preview)

// PreviewView creates a surface provider and is the recommended provider
preview.setSurfaceProvider(viewFinder.getSurfaceProvider())
```



## CameraX生命周期

不用手动在onResume和onPause里面手动调用相机的启动和停止，只要绑定Lifecycle，CameraX会监听生命周期自动配置相机拍摄会话并确保相机状态随生命周期的转换相应地变化。

组合用例有一些生命周期的注意，这次用不到，就不写了。



## 权限

- 需要 `CAMERA`权限
- 除非所用设备搭载 Android 10 或更高版本，否则应用还需要 `WRITE_EXTERNAL_STORAGE` 权限。



## 依赖项

`settings.gradle`:

```groovy
dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
    repositories {
        google()
        mavenCentral()
    }
}
```

`build.gradle`

```groovy
android {
    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }
    // For Kotlin projects
    kotlinOptions {
        jvmTarget = "1.8"
    }
}

dependencies {
  // CameraX core library using the camera2 implementation
  def camerax_version = "1.3.0-alpha04"
  // The following line is optional, as the core library is included indirectly by camera-camera2
  implementation "androidx.camera:camera-core:${camerax_version}"
  implementation "androidx.camera:camera-camera2:${camerax_version}"
  // If you want to additionally use the CameraX Lifecycle library
  implementation "androidx.camera:camera-lifecycle:${camerax_version}"
  // If you want to additionally use the CameraX VideoCapture library
  implementation "androidx.camera:camera-video:${camerax_version}"
  // If you want to additionally use the CameraX View class
  implementation "androidx.camera:camera-view:${camerax_version}"
  // If you want to additionally add CameraX ML Kit Vision Integration
  implementation "androidx.camera:camera-mlkit-vision:${camerax_version}"
  // If you want to additionally use the CameraX Extensions library
  implementation "androidx.camera:camera-extensions:${camerax_version}"
}
```



# 配置 

## 配置每个 CameraX 用例

例如，对于图片拍摄用例，您可以设置目标宽高比和闪光灯模式。以下代码显示了一个示例：

```kotlin
val imageCapture = ImageCapture.Builder()
    .setFlashMode(...)
    .setTargetAspectRatio(...)
    .build()
```

## CameraXConfig

首先，CameraX 具有适合大多数使用场景的默认配置，但是，如果您的应用有特殊要求或希望自定义这些配置，可使用 `CameraXConfig`接口实现此目的。

借助 `CameraXConfig`，应用可以执行以下操作：

- 使用 `setAvailableCameraLimiter()` 优化启动延迟时间。
- 使用 `setCameraExecutor()` 向 CameraX 提供应用执行器。
- 将默认调度器处理程序替换为 `setSchedulerHandler()`。
- 使用 `setMinimumLoggingLevel()` 更改日志记录级别。

使用方式：在 `Application`中实现 [`CameraXConfig.Provider`](https://developer.android.google.cn/reference/androidx/camera/core/CameraXConfig.Provider?hl=zh-cn) 接口，并在 [`getCameraXConfig()`](https://developer.android.google.cn/reference/androidx/camera/core/CameraXConfig.Provider?hl=zh-cn#getCameraXConfig()) 中返回 `CameraXConfig` 对象。示例如下：

```kotlin
class CameraApplication : Application(), CameraXConfig.Provider {
   override fun getCameraXConfig(): CameraXConfig {
       return CameraXConfig.Builder.fromConfig(Camera2Config.defaultConfig())
           .setMinimumLoggingLevel(Log.ERROR).build()
   }
}
```



### 限制使用的摄像头，优化启动时间

因为要和硬件组件通信，CarmeraX在枚举和查询设备上可用摄像头的过程会比较长，尤其是在低端设备上。如果应用有特定使用的摄像头，就可以限制只使用此摄像头，则CameraX会忽略其他摄像头。如下代码就是指定了后置摄像头：

```kotlin
class MainApplication : Application(), CameraXConfig.Provider {
   override fun getCameraXConfig(): CameraXConfig {
       return CameraXConfig.Builder.fromConfig(Camera2Config.defaultConfig())
              .setAvailableCamerasLimiter(CameraSelector.DEFAULT_BACK_CAMERA)
              .build()
   }
}
```



### 设置自定义的线程池

CameraX 要和硬件之间进行阻塞的IPC，因此会使用一个后台线程的 Excutor 调用对应的API。可以通过设置 [`CameraXConfig.Builder.setCameraExecutor()`](https://developer.android.google.cn/reference/androidx/camera/core/CameraXConfig.Builder?hl=zh-cn#setCameraExecutor(java.util.concurrent.Executor)) 和 [`CameraXConfig.Builder.setSchedulerHandler()`](https://developer.android.google.cn/reference/androidx/camera/core/CameraXConfig.Builder?hl=zh-cn#setSchedulerHandler(android.os.Handler)) 指定使用的后台线程。



### 设置Scheduler

Sheduler用于对内部任务的调度，包含一些特定的case的以及对旧版API 平台的特殊处理。CameraX 会分配和管理内部 [`HandlerThread`](https://developer.android.google.cn/reference/android/os/HandlerThread?hl=zh-cn) 来执行这些任务，但您可以将其替换为 `CameraXConfig.Builder.setSchedulerHandler()`。



### 日志记录

借助 CameraX 日志记录，应用可以过滤 logcat 消息，因为在正式版代码中应尽量避免包含详细消息。CameraX 支持四种日志记录级别（从最详细到最严重）：

- `Log.DEBUG`（默认）
- `Log.INFO`
- `Log.WARN`
- `Log.ERROR`

可以使用 [`CameraXConfig.Builder.setMinimumLoggingLevel(int)`](https://developer.android.google.cn/reference/androidx/camera/core/CameraXConfig.Builder?hl=zh-cn#setMinimumLoggingLevel(int)) 设置。



## 自动选择

CameraX 会根据运行您的应用的设备自动提供专用的功能。例如，如果您未指定分辨率或您指定的分辨率不受支持，CameraX 会自动确定要使用的最佳分辨率。所有这些操作均由库进行处理，无需您编写设备专属代码。

CameraX 的目标是成功初始化摄像头会话。这意味着，CameraX 会根据设备功能降低分辨率和宽高比。发生这种情况的原因如下：

- 设备不支持请求的分辨率。
- 设备存在兼容性问题，例如需要特定分辨率才能正常运行的旧设备。
- 在某些设备上，某些格式仅在某些宽高比下可用。
- 对于 JPEG 或视频编码，设备首选“最近的 mod16”。如需了解详情，请参阅 [`SCALER_STREAM_CONFIGURATION_MAP`](https://developer.android.google.cn/reference/android/hardware/camera2/CameraCharacteristics?hl=zh-cn#SCALER_STREAM_CONFIGURATION_MAP)。

尽管 CameraX 会创建并管理会话，您也应始终在代码中检查用例输出所返回的图片大小，并进行相应调整。



## 旋转

首先，用例创建期间，摄像头的旋转角度会设置为与默认的显示屏旋转角度保持一致。其次，可以通过 [`ImageAnalysis.setTargetRotation()`](https://developer.android.google.cn/reference/androidx/camera/core/ImageAnalysis?hl=zh-cn#setTargetRotation(int))来更新旋转设置。

如下代码展示了如何为屏幕方向事件设置旋转角度：

```kotlin
override fun onCreate() {
    val imageCapture = ImageCapture.Builder().build()

    val orientationEventListener = object : OrientationEventListener(this as Context) {
        override fun onOrientationChanged(orientation : Int) {
            // Monitors orientation values to determine the target rotation value
            val rotation : Int = when (orientation) {
                in 45..134 -> Surface.ROTATION_270
                in 135..224 -> Surface.ROTATION_180
                in 225..314 -> Surface.ROTATION_90
                else -> Surface.ROTATION_0
            }

            imageCapture.targetRotation = rotation
        }
    }
    orientationEventListener.enable()
}
```

每个用例都会根据设定的旋转角度直接旋转图片数据，或者向用户提供未旋转图片数据的旋转元数据。

- **Preview**：提供元数据输出，以便使用 [`Preview.getTargetRotation()`](https://developer.android.google.cn/reference/androidx/camera/core/Preview?hl=zh-cn#getTargetRotation()) 了解目标分辨率的旋转设置。
- **ImageAnalysis**：提供元数据输出，以便了解图片缓冲区坐标相对于显示坐标的位置。
- **ImageCapture**：更改图片 Exif 元数据、缓冲区或同时更改两者，从而反映旋转设置。更改的值取决于 HAL 实现。



## ViewPort

`ViewPort` 用于指定最终用户可看到的缓冲区矩形。CameraX 会根据视口的属性及附加的用例计算出可能的最大剪裁矩形。获取视口的一种简单方法是使用 [`PreviewView`](https://developer.android.google.cn/training/camerax/preview?hl=zh-cn#implementation):

```kotlin
val viewport = findViewById<PreviewView>(R.id.preview_view).viewPort
```



## 选择摄像头

- 使用 [`CameraSelector.DEFAULT_FRONT_CAMERA`](https://developer.android.google.cn/reference/androidx/camera/core/CameraSelector?hl=zh-cn#DEFAULT_FRONT_CAMERA) 请求默认的前置摄像头。
- 使用 [`CameraSelector.DEFAULT_BACK_CAMERA`](https://developer.android.google.cn/reference/androidx/camera/core/CameraSelector?hl=zh-cn#DEFAULT_BACK_CAMERA) 请求默认的后置摄像头。
- 使用 [`CameraSelector.Builder.addCameraFilter()`](https://developer.android.google.cn/reference/androidx/camera/core/CameraSelector.Builder?hl=zh-cn#addCameraFilter(androidx.camera.core.CameraFilter)) 按 [`CameraCharacteristics`](https://developer.android.google.cn/reference/android/hardware/camera2/CameraCharacteristics?hl=zh-cn) 过滤可用设备列表。

Sample，创建`CameraSelector`来影响设备选择：

```kotlin
fun selectExternalOrBestCamera(provider: ProcessCameraProvider):CameraSelector? {
   val cam2Infos = provider.availableCameraInfos.map {
       Camera2CameraInfo.from(it)
   }.sortedByDescending {
       // HARDWARE_LEVEL is Int type, with the order of:
       // LEGACY < LIMITED < FULL < LEVEL_3 < EXTERNAL
       it.getCameraCharacteristic(CameraCharacteristics.INFO_SUPPORTED_HARDWARE_LEVEL)
   }

   return when {
       cam2Infos.isNotEmpty() -> {
           CameraSelector.Builder()
               .addCameraFilter {
                   it.filter { camInfo ->
                       // cam2Infos[0] is either EXTERNAL or best built-in camera
                       val thisCamId = Camera2CameraInfo.from(camInfo).cameraId
                       thisCamId == cam2Infos[0].cameraId
                   }
               }.build()
       }
       else -> null
    }
}

// create a CameraSelector for the USB camera (or highest level internal camera)
val selector = selectExternalOrBestCamera(processCameraProvider)
processCameraProvider.bindToLifecycle(this, selector, preview, analysis)
```



## 同时选择多个摄像头

从 CameraX 1.3 开始，您还可以同时选择多个摄像头。 例如，您可以对前置和后置摄像头进行绑定，以便从两个视角同时拍摄照片或录制视频。

```kotlin
// Build ConcurrentCameraConfig
val primary = ConcurrentCamera.SingleCameraConfig(
    primaryCameraSelector,
    useCaseGroup,
    lifecycleOwner
)

val secondary = ConcurrentCamera.SingleCameraConfig(
    secondaryCameraSelector,
    useCaseGroup,
    lifecycleOwner
)

val concurrentCamera = cameraProvider.bindToLifecycle(
    listOf(primary, secondary)
)

val primaryCamera = concurrentCamera.cameras[0]
val secondaryCamera = concurrentCamera.cameras[1]
```



## 分辨率

### 自动分辨率

在`bindToLifecycle()`中指定了用例，CameraX 会根据指定的用例及设备情况自动确定分辨率。多用例要在一个`bindToLifecyle()`中同时指定。

图片拍摄和图片分析用例的默认宽高比为 4:3。

一些用例可以配置宽高比，CameraX 会根据设备情况选择一个尽量满足条件的分辨率。



### 指定分辨率

使用 `setTargetResolution(Size resolution)` 方法构建用例时，您可以设置特定分辨率，如以下代码示例所示：

```kotlin
val imageAnalysis = ImageAnalysis.Builder()
    .setTargetResolution(Size(1280, 720))
    .build()
```

无法针对同一个用例设置目标宽高比和目标分辨率。如果这样做，则会在构建配置对象时抛出 `IllegalArgumentException`。



## 控制相机输出

所有用例的通用操作：

- 利用 [`CameraControl`](https://developer.android.google.cn/reference/androidx/camera/core/CameraControl?hl=zh-cn)，您可以配置通用摄像头功能。
- 利用 [`CameraInfo`](https://developer.android.google.cn/reference/androidx/camera/core/CameraInfo?hl=zh-cn)，您可以查询这些通用摄像头功能的状态。

以下是 CameraControl 支持的摄像头功能：

- 变焦
- 手电筒
- 对焦和测光（点按即可对焦）
- 曝光补偿

详细使用暂时不看。需要时再看。



# 预览用例

