

https://developer.android.com

#6.0（M-23）

1. __运行时权限__

2. 通知api变动

3. OpenSSL -> BoringSSL

4. __硬件标识符访问：mac地址：02:00:00:00:00:00__

   

#7.0（N-24）

1. 多窗口支持

2. 通知增强

3. 低电耗模式

4. __后台优化：target 24收不到`CONNECTIVITY_ACTION`；不管target是不是24，在7系统上应用都无法发送或接收 `ACTION_NEW_PICTURE` 或 `ACTION_NEW_VIDEO` 广播。使用[JobScheduler](https://developer.android.com/reference/android/app/job/JobScheduler.html?hl=zh-cn)__

5. 建议Surfaceview替代TextureView

6. Vulkan API(3D渲染)

7. Quick Setting API(添加快速设置入口)

8. API V2签名

9. __默认受信任的证书颁发机构（https://juejin.im/post/5aa9f35b51882555731bd922，https://developer.android.com/training/articles/security-config.html, https://developer.android.com/training/articles/security-config?hl=zh-cn）__

10. 作用域目录访问

11. Custom Pointer API

    

#8.0（O-26）

1. 又是通知（https://developer.android.com/about/versions/oreo/android-8.0?hl=zh-cn）

2. 自动填充

3. PIP(画中画)

4. 字体相关（可下载字体&XML使用字体）

5. __TextView根据布局调整大小 AutoSize__

6. 固定桌面快捷方式和小部件

7. 多显示器支持

8. [`layout_marginVertical`](https://developer.android.com/reference/android/R.attr.html?hl=zh-cn#layout_marginVertical)，同时定义 [`layout_marginTop`](https://developer.android.com/reference/android/R.attr.html?hl=zh-cn#layout_marginTop) 和 [`layout_marginBottom`](https://developer.android.com/reference/android/R.attr.html?hl=zh-cn#layout_marginBottom)。

   [layout_marginHorizontal`](https://developer.android.com/reference/android/R.attr.html?hl=zh-cn#layout_marginHorizontal)，同时定义 [`layout_marginLeft`](https://developer.android.com/reference/android/R.attr.html?hl=zh-cn#layout_marginLeft) 和 [`layout_marginRight`](https://developer.android.com/reference/android/R.attr.html?hl=zh-cn#layout_marginRight)。

   [`paddingVertical`](https://developer.android.com/reference/android/R.attr.html?hl=zh-cn#paddingVertical)，同时定义 [`paddingTop`](https://developer.android.com/reference/android/R.attr.html?hl=zh-cn#paddingTop) 和 [`paddingBottom`](https://developer.android.com/reference/android/R.attr.html?hl=zh-cn#paddingBottom)。

   [`paddingHorizontal`](https://developer.android.com/reference/android/R.attr.html?hl=zh-cn#paddingHorizontal)，同时定义 [`paddingLeft`](https://developer.android.com/reference/android/R.attr.html?hl=zh-cn#paddingLeft) 和 [`paddingRight`](https://developer.android.com/reference/android/R.attr.html?hl=zh-cn#paddingRight)。

9. 应用类别

10. `AnimatorSet` API 现在支持寻道和倒播功能‘

11. 新的 StrictMode 检测程序

12. JobScheduler 改进



#9.0（P-28）

####Feature api

1. 利用 Wi-Fi RTT 进行室内定位
2. 刘海屏 API
3. 通知
4. ImageDecoder取代BitmapFactory和BitmapFactory.Option
5.  [`AnimatedImageDrawable`](https://developer.android.com/reference/android/graphics/drawable/AnimatedImageDrawable.html?hl=zh-cn) 类，用于绘制和显示 GIF 和 WebP 动画图像
6. HDR VP9视频、HEIF图像压缩和Media API
7. JobScheduler流量费用敏感
8. 文本预先计算：[`PrecomputedText`](https://developer.android.com/reference/android/text/PrecomputedText?hl=zh-cn) 类使您能提前计算和缓存所需信息，改善了文本渲染性能。 它还使您的应用可以在主线程之外执行文本布局

  

####非tartget行为变更：

1. 电源管理：应用分组

2. 后台对传感器限制：不能访问麦克风或摄像头；使用[连续](https://source.android.com/devices/sensors/report-modes?hl=zh-cn#continuous)报告模式的传感器（例如加速度计和陀螺仪）不会接收事件；使用[变化](https://source.android.com/devices/sensors/report-modes?hl=zh-cn#on-change)或[一次性](https://source.android.com/devices/sensors/report-modes?hl=zh-cn#one-shot)报告模式的传感器不会接收事件。

3. Android 9 引入 [`CALL_LOG`](https://developer.android.com/reference/android/Manifest.permission_group?hl=zh-cn#CALL_LOG) [权限组](https://developer.android.com/guide/topics/permissions/overview?hl=zh-cn#perm-groups)并将 [`READ_CALL_LOG`](https://developer.android.com/reference/android/Manifest.permission?hl=zh-cn#READ_CALL_LOG)、[`WRITE_CALL_LOG`](https://developer.android.com/reference/android/Manifest.permission?hl=zh-cn#WRITE_CALL_LOG) 和 [`PROCESS_OUTGOING_CALLS`](https://developer.android.com/reference/android/Manifest.permission?hl=zh-cn#PROCESS_OUTGOING_CALLS) 权限移入该组。 在之前的 Android 版本中，这些权限位于 `PHONE` 权限组。对于需要访问通话敏感信息（如读取通话记录和识别电话号码）的应用，该 `CALL_LOG` 权限组为用户提供了更好的控制和可见性。

4. 限制访问电话号码： [`READ_CALL_LOG`](https://developer.android.com/reference/android/Manifest.permission?hl=zh-cn#READ_CALL_LOG) 

   在未首先获得 [`READ_CALL_LOG`](https://developer.android.com/reference/android/Manifest.permission?hl=zh-cn#READ_CALL_LOG) 权限的情况下，除了应用的用例需要的其他权限之外，运行于 Android 9 上的应用无法读取电话号码或手机状态。

   与来电和去电关联的电话号码可在[手机状态广播](https://developer.android.com/reference/android/telephony/TelephonyManager?hl=zh-cn#action_phone_state_changed)（比如来电和去电的手机状态广播）中看到，并可通过 [`PhoneStateListener`](https://developer.android.com/reference/android/telephony/PhoneStateListener?hl=zh-cn) 类访问。 但是，如果没有 [`READ_CALL_LOG`](https://developer.android.com/reference/android/Manifest.permission?hl=zh-cn#READ_CALL_LOG) 权限，则 `PHONE_STATE_CHANGED` 广播和 `PhoneStateListener` 提供的电话号码字段为空。

   要从手机状态中读取电话号码，请根据您的用例更新应用以请求必要的权限：

   - 要通过 `PHONE_STATE` [Intent 操作](https://developer.android.com/guide/topics/manifest/action-element?hl=zh-cn)读取电话号码，同时需要 [`READ_CALL_LOG`](https://developer.android.com/reference/android/Manifest.permission?hl=zh-cn#READ_CALL_LOG) 权限和 [`READ_PHONE_STATE`](https://developer.android.com/reference/android/Manifest.permission.html?hl=zh-cn#READ_PHONE_STATE) 权限。
   - 要从 [`onCallStateChanged()`](https://developer.android.com/reference/android/telephony/PhoneStateListener?hl=zh-cn#oncallstatechanged) 中读取电话号码，只需要 [`READ_CALL_LOG`](https://developer.android.com/reference/android/Manifest.permission?hl=zh-cn#READ_CALL_LOG) 权限。 不需要 [`READ_PHONE_STATE`](https://developer.android.com/reference/android/Manifest.permission.html?hl=zh-cn#READ_PHONE_STATE) 权限。

5. 限制访问WiFi位置和连接信息

6. __限制非SDK接口的限制__

7. 安全行为变更：

   1). 传输层安全协议（TLS）变更

   2). 更严格的SECCOMP过滤

   3). 加密变更

8. UTF解码器变更

9. 名称解析的网络地址查询可能导致网络违规（不要再主线程）；解析数字 IP 地址不被视为阻塞性操作

10. 套接字标记

11. 更加详细的VPN网络功能报告

12. 应用不再能访问 xt_qtaguid 文件夹中的文件

13. 强制执行 FLAG_ACTIVITY_NEW_TASK 要求

14. 屏幕旋转变更

15. Apache HTTP 客户端弃用影响采用非标准 ClassLoader 的应用

16. 枚举相机

####Tartget行为变更：

1. 前台服务：必须请求 [`FOREGROUND_SERVICE`](https://developer.android.com/reference/android/Manifest.permission.html?hl=zh-cn#FOREGROUND_SERVICE) 权限
2. 构建序列号弃用：[`Build.SERIAL`](https://developer.android.com/reference/android/os/Build.html?hl=zh-cn#SERIAL) 始终设置为 `"UNKNOWN"` 以保护用户的隐私。如果您的应用需要访问设备的硬件序列号，您应改为请求 [`READ_PHONE_STATE`](https://developer.android.com/reference/android/Manifest.permission.html?hl=zh-cn#READ_PHONE_STATE) 权限，然后调用 [`getSerial()`](https://developer.android.com/reference/android/os/Build.html?hl=zh-cn#getSerial())。
3. 为目标平台的应用应采用私有 DNS API
4. 如果您的应用以 Android 9 或更高版本为目标平台，则默认情况下 [`isCleartextTrafficPermitted()`](https://developer.android.com/reference/android/security/NetworkSecurityPolicy.html?hl=zh-cn#isCleartextTrafficPermitted()) 函数返回 `false`。 如果您的应用需要为特定域名启用明文，您必须在应用的[网络安全性配置](https://developer.android.com/training/articles/security-config.html?hl=zh-cn)中针对这些域名将 `cleartextTrafficPermitted` 显式设置为 `true`
5. 为改善 Android 9 中的应用稳定性和数据完整性，应用无法再让[多个进程](https://developer.android.com/guide/components/processes-and-threads.html?hl=zh-cn)共用同一 [`WebView`](https://developer.android.com/reference/android/webkit/WebView.html?hl=zh-cn) 数据目录
6.  Android 9 或更高版本为目标平台的应用无法利用可全局访问的 Unix 权限与其他应用共享数据
7. ICU 库更新
8. 网络流量计数
9. Apache HTTP弃用
10. 0面积的视图不会有焦点，但是可以手动request
11. CSS RGBA 十六进制值处理
12. 来自已暂停应用的通知



