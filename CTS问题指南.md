# Android CTS指南
## 一.简介
CTS(Compatibility Test Suit)是Google为Android发布的一套兼容性测试用例。由于android是开源的，手机制造商及运营商可以在Android上打造，定制自己特有的手机操作系统，这势必会在源码级别上对Android系统进行代码的增删改。如果不规范，这些更改则会给上层的应用开发带来问题（届时可能会看到华为上跑的飞起的吃鸡在三星上跪了，导致每个应用都需要发布不同厂商的Android手机的版本）。因此，Google为Android提供了一套兼容性测试用例CTS，对手机的硬件，软件，接口，性能进行测试。只有通过CTS测试的Android手机系统，Google才会颁发许可，以保证不同生产商之间的Android系统的兼容。
### 工作原理
CTS  在桌面设备上运行，并直接在连接的设备或模拟器上执行测试用例，是一个自动化测试工具，其中包括两个主要软件组件：

- CTS tradefed 自动化测试框架会在桌面设备上运行，并管理测试执行情况。
- 单独的测试用例会在被测设备 (DUT) 上执行。测试用例采用 Java 语言编写为 JUnit 测试，并打包为 Android .apk 文件，以在实际目标设备上运行。

CTS verifier是对 CTS的补充。为无法在没有手动输入（例如音频质量、加速度计等）的固定设备上进行测试的 API 和功能提供测试。是一款手动测试工具，包含以下软件组件：

- 在 DUT 上执行并收集结果的 CTS 验证程序应用。
- 在桌面设备上执行，以便为 CTS 验证程序应用中的某些测试用例提供数据或额外控制的可执行文件或脚本。

### 工作流程
官方总结图
![](https://source.android.com/compatibility/cts/images/cts-0.png?dcb_=0.6032242971376689) 

### 测试用例的类型
CTS 包含以下类型的测试用例：

- 单元测试会测试 Android 平台中很微小的代码元素；例如 java.util.HashMap 等单个类。

- 功能测试会通过较高级别的用例将 API 组合到一起进行测试。

CTS 的未来版本将包含以下类型的测试用例：

- 稳健性测试会测试系统在压力下的耐久性。

- 性能测试会根据定义的基准测试系统的性能，例如每秒渲染帧数。

在CTS测试结束以后，测试完成会生成相应logs和results，test_result.xml中有Junit TestRunner中所熟悉的状态条，绿色表示通过，红色表示此测试用例没通过，内容为测试结果简述以及各个测试包的测试详情，包括测试通过（ Pass ）的测试用例数和失败（ Fail ）的用例数。

Fail 的大致原因有两种，一种是客制化后的系统有问题，对于这种 Fail 需要检查并修改程序后重新测试。
另一种是 Android 系统本身的问题，这种 Fail 需要可以申请 Google 的豁免。

### 涵盖的领域
单元测试用例涵盖以下领域，以确保兼容性：

|领域|	说明|
| ---| --- |
|签名测试	|对于每个 Android 版本，都存在用于描述这一版本中所含的所有公开 API 签名的 XML 文件。CTS 包含一个实用工具，用于根据设备上可用的 API 检查这些 API 签名。签名检查的结果会记录在测试结果 XML 文件中。|
|平台 API 测试	|按照 SDK 类索引所述测试平台（核心库和 Android 应用框架）API，以确保 API 的正确性，包括正确的类、属性和方法签名以及正确的方法行为；此外执行负面测试，以确保不正确的参数处理产生预期行为。|
|Dalvik 测试	|这类测试侧重于测试 Dalvik 可执行格式的文件。|
|平台数据模型	|CTS 会按照 SDK android.provider 软件包中所述测试通过内容提供程序提供给应用开发者的核心平台数据模型：通讯录、浏览器、设置等。|
|平台 Intent	|CTS 会按照 SDK 可用 Intent 中所述测试核心平台 Intent。|
|平台权限	|CTS 会按照 SDK 可用权限中所述测试核心平台权限。|
|平台资源	|CTS 会按照 SDK 可用资源类型中所述进行测试以正确处理核心类型的平台资源。这包括对以下资源的测试：简单值、可绘制资源、九宫文件、动画、布局、样式和主题背景，以及加载备用资源。|


## 二. CTS Brance & tag & code
1、 CTS Branch
CTS 有自己的开发分支,每个 Android 版本对应不同的 CTS 分支,具体可见下表
注意:该时间表是暂定的,可能会随着给定 Android 版本 CTS 的正式发布而不时更新

| Android 版本 | 分支  |频率|
| ---| ---| --- |
|9 |pie-cts-dev |每月|
|8.1 |oreo-mr1-cts-dev |每月|
|8 |oreo-cts-dev |每月|
|7.1 |nougat-mr1-cts-dev |每月|
|7 |nougat-cts-dev |每月|
|6 |marshmallow-cts-dev |每月|

对于 5.1、5.0、4.4、4.3 和 4.2,尚没有相应的版本发布计划。

**版本发布当月的重要日期**

- 第一周的周末：代码冻结。此时，不再接受与当前分支有关的提交内容，并且提交内容不会包含在下一版本的 CTS 中。我们选择了候选版本后，分支将再次开放并接受新的提交内容。
- 第二周或第三周：在 Android 开放源代码项目 (AOSP) 中发布 CTS。

2、 CTS Tag

(1) CTS Tag 介绍
Google 每个月均会更新测试包,每次更新时,均会 release 一个 build tag 用于标识,便于查看相应 code,
具体可见 Google 的说明
https://source.android.com/compatibility/cts/downloads#android-81

例如,目前8.1测试包已经更新到8.1 R9版本,其对应的 TAG 为:android-cts-8.1_r9

(2) Tag 格式
CTS Tag 格式一般为:android-cts-aa_rb,其中,aa 代表 Android 版本号,b 代表测试包版本号,比如
android-cts-8.1_r1就代表8.1 R1测试包对应的 tag

3、CTS code
有两种方式可查看 CTS code
如果是为了 debug,建议采用方法1;如果只是单纯的查看 code,建议采用方法2

*方法1:从平台代码获取 CTS code*
平台版本的 CTS code 与 google release 的 CTS 测试包的 code 不完全一致
所以哪个测试包报出的问题,结合 code 分析时就需要将 code 切换到对应的版本上


**(1) 获取 code**

`git clone https://android.googlesource.com/platform/cts`

a. 获取特定分支的 code
cts case 问题导致的 failure 需要向 google 提交修复 patch,patch 需提交到 cts 的开发分支
CTS 分支可参上
Branch 切换方法如下:
进入 CTS 仓库,执行如下命令:

$ git fetch

$ git branch -r | grep cts //获得 cts 的相应 branch
![](/home/local/SPREADTRUM/hursion.zhang/Pictures/Selection_008.png) 

$ git checkout -b nougat-cts-dev korg/ nougat-cts-dev
获得 nougat-cts-dev 分支的 cts code

b. 获取特定测试包的 code
即获取特定 tag 的 code
Tag 切换方法如下:
进入 CTS 仓库,执行如下命令

$ git fetch

$ git tag | grep cts //获得 cts 的相应 tag

![](/home/local/SPREADTRUM/hursion.zhang/Pictures/Selection_007.png) 

$ git checkout -b 7.0_r24 android-cts-7.0_r24 获得 android-cts-7.0_r24的 cts code

**(2) 在 cts 中添加 log**
将 code 切换到对应 tag 或者 branch 之后,为更方便定位问题,可以在 cts 代码中添加 log 来定位问题

1)在 cts code 中添加 log 并在对应的 package 中采用 mm 编译,生成相应的测试 apk;

2)将 apk 替换到 cts 测试包的 android-cts/testcases 中,测试时便会自动安装此 apk,执行相应方法
或者在 android 根目录下整体编译 cts:make cts 获取更多添加 log 的 apk

*方法2:在 googlesource 查看 code*

(1) 查看特定 branch/tag 的 code
如果只是单纯的查看 code,并不打算 debug,可以直接进入以下地址来查看

https://android.googlesource.com/platform/cts/

需要根据自己的需求,先从左边的 branches 或者 Tags 选择一个

(2) 查看两个 tag 之间的 code 差异
例如,android-cts-7.0_r23与 android-cts-7.0_r24之间的差异,可通过以下地址查看

https://android.googlesource.com/platform/cts/+log/android-cts-7.0_r23.android-cts-7.0_r24/?no-merges&n=1000

若想查看别的 tag 之间的差异,修改以上地址的 tag 即可

## 三.测试环境配置
### 物理环境
#### 蓝牙 LE 信标
如果 DUT 支持蓝牙 LE 功能，则应在与 DUT 的距离不超过五米的范围内放置至少三个蓝牙 LE 信标，以进行蓝牙 LE 扫描测试。这些信标可以为任何类型，不需要进行配置或发射任何特定信号，并且可以包括 iBeacon、Eddystone，甚至模拟 BLE 信标的设备。

#### 相机
在运行相机 CTS 时，建议您使用正常光照条件，并且测试图案图表（例如棋盘图案）不要与镜头靠得太近（具体距离取决于设备的最小焦距）。

如果 DUT 支持外部相机（如 USB 网络摄像头），则在运行 CTS 时必须将外部相机连接到充电器，否则 CTS 测试将失败。

#### GPS/GNSS
如果 DUT 支持全球定位系统 (GPS)/全球导航卫星系统 (GNSS) 功能，则应该以合适的信号电平向 DUT 提供 GPS/GNSS 信号（GPS 部分符合 ICD-GPS-200C 标准），以便其接收到相应信号并计算 GPS 位置。GPS/GNSS 信号源的种类不限（可以是卫星模拟器，也可以是室外 GPS/GNSS 信号中继器），只需将 DUT 放在距离窗口足够近的位置以使其可以直接接收到足够强的 GPS/GNSS 信号即可。

请注意：在进行 GPS 测试时，请确保互联网连接设置未屏蔽 supl.google.com 的 7276 端口的连接。该端口将用于下载 GPS 辅助数据，以便在本地设备上测试位置计算。
#### WLAN 和 IPv6
CTS 测试需要满足以下要求的 WLAN 网络：支持 IPv6，可以将被测设备 (DUT) 视为隔离客户端，并可以连接到互联网。隔离客户端是一种配置，可使 DUT 无法接收子网络上的广播/多网消息；这种配置可通过 WLAN AP 配置或通过在未连接其他设备的隔离子网络上运行 DUT 来实现。

如果您无法访问原生 IPv6 网络、IPv6 运营商网络或 IPv6 VPN，以致无法通过基于 IPv6 的一些测试，则可以改为使用 WLAN 接入点和 IPv6 隧道。请参阅维基百科 IPv6 隧道代理列表。

#### Wi-Fi RTT（往返时间）
Android 9 针对 Wi-Fi RTT 功能增加了一个 API，此 API 允许设备测量自身与接入点之间的距离（误差幅度在 1 到 2 米内），从而显著提高室内位置信息精确度。以下是支持 Wi-Fi RTT 的两款推荐设备：Google Wifi 和 Compulab 的 Filet2 接入点（设为 40MHz 带宽，频率为 5GHz）。

接入点应接入电源，但无需连接到任何网络。接入点无需紧挨着测试设备，但建议将其放置在距离 DUT 40 英尺的位置。通常情况下，一个接入点就足够了。

### 台式机设置
注意：CTS 目前支持 64 位 Linux 和 Mac OS 主机。CTS 无法在 Windows 操作系统上运行。
#### ADB 和 AAPT
在运行 CTS 之前，请确保您已安装最新版本的 Android 调试桥 (adb) 和 Android 资源打包工具 (AAPT)，并将这些工具的位置添加到计算机的系统路径中。

要安装 ADB，请下载适用于您的操作系统的 Android SDK 工具包，打开它，然后按照附带的 README 文件中的说明进行操作。要了解问题排查相关信息，请参阅安装独立 SDK 工具。

确保 adb 和 aapt 位于您的系统路径下。以下命令假定您已在主目录中打开了软件包归档文件：

export PATH=$PATH:$HOME/android-sdk-linux/build-tools/<version>

注意：请确保起始路径和目录名称均准确无误。

#### Java 开发套件 (JDK)
安装正确版本的 Java 开发套件 (JDK)。对于 Android 7.0 -

在 Ubuntu 上，使用 OpenJDK 8。
在 Mac OS 上，使用 jdk 8u45 或更高版本。
如需了解详情，请参阅 JDK 要求。

#### CTS 文件
下载并打开与您设备的 Android 版本以及您的设备支持的所有应用二进制接口 (ABI) 相匹配的 CTS 包。

下载并打开最新版本的 CTS 媒体文件。

#### 设备检测
请按照相应的步骤设置您的系统以检测设备，例如为 Ubuntu Linux 创建 udev 规则文件。

### Android 设备设置
#### 用户版本
兼容的设备被定义为具有 user/release-key 签名版本的设备，因此您的设备应运行基于代号、标签和版本号中已知兼容的用户版本（Android 4.0 及更高版本）的系统映像。
注意：使用 CTS 确认最终系统映像的 Android 兼容性时，您必须在具有用户版本的设备上执行 CTS。

#### 初始 API 级别版本属性
某些 CTS 要求取决于设备最初搭载的版本。例如，如果设备最初搭载的是较低的版本，则不一定需要遵循适用于搭载较高版本的设备的系统要求。

为了保证 CTS 可读取到这些信息，设备制造商可以定义编译时属性：ro.product.first_api_level。该属性的值是对该设备进行商业化发布时所采用的初始 API 级别。

OEM 可以将 PRODUCT_PROPERTY_OVERRIDES 添加到其 device.mk 文件以设置这项属性，具体如以下示例所示：

\#ro.product.first_api_level indicates the first api level, device has been commercially launched on.
PRODUCT_PROPERTY_OVERRIDES +=\
ro.product.first_api_level=21

注意：对于产品的第一个版本，ro.product.first_api_level 属性应处于未设置状态 (removed)；而对于所有后续版本，该属性应设置为正确的 API 级别值。通过这种方式，该属性可以正确标识新产品，而且我们不会丢失任何关于产品初始 API 级别的信息。如果标记处于未设置状态，则 Android 会将 Build.VERSION.SDK_INT 分配给 ro.product.first_api_level。
#### CTS Shim 应用
Android 7.0 包含以下预编译的应用（根据此处的源代码编译），这些应用不包含除清单以外的任何代码：

frameworks/base/packages/CtsShim/CtsShim.apk
该 apk 文件将复制到系统映像上的 /system/app/CtsShimPrebuilt.apk。
frameworks/base/packages/CtsShim/CtsShimPriv.apk
该 apk 文件将复制到系统映像上的 /system/priv-app/CtsShimPrivPrebuilt.apk。
CTS 会使用这些应用来测试特权和权限。要通过测试，您必须将应用预加载到系统映像上的相应目录下，但不能对它们重新签名。

#### 示例小程序
Android 9 引入了 Open Mobile API 测试用例，用于检查安全元件底层实现是否符合标准。这些测试用例需要安装可供 CTS 应用用于与之通信的专用小程序。用户可以使用提供的示例小程序。

这个小程序适用于配有 eSE（嵌入式安全元件）、SIM 或 SD 的设备。要详细了解 Open Mobile API 测试用例和访问控制测试用例，请参阅安全元件的 CTS 测试。

#### 存储空间要求
CTS 媒体压力测试要求将视频剪辑存放在外部存储设备 (/sdcard) 上。

所需空间取决于设备支持的最高视频播放分辨率（要查看所需分辨率的平台版本，请参阅兼容性定义文档中的第 5 部分）。请注意，被测设备的视频播放功能将通过 android.media.CamcorderProfile API（针对早期 Android 版本）和 android.media.MediaCodecInfo.CodecCapabilities API（针对 Android 5.0）进行检测。

以下是按最大视频播放分辨率列出的存储空间要求：

480x360: 98MB

720x480: 193MB

1280x720: 606MB

1920x1080: 1863MB

#### 屏幕和存储空间
任何没有嵌入式屏幕的设备一律需要连接到屏幕。
如果设备具有存储卡插槽，请插入空的 SD 卡。请使用支持超高速 (UHS) 总线且具有 SDHC 或 SDXC 容量的 SD 卡，或使用至少具有 Class 10 速度的 SD 卡，以确保设备能够通过 CTS。

警告：CTS 可能会修改/清空插入设备的 SD 卡上的数据。

如果设备具有 SIM 卡插槽，请将激活的 SIM 卡插入每个插槽。如果设备支持短信，则应填充每个 SIM 卡的号码字段。
#### 开发者 UICC
为了执行 CTS 运营商 API 测试，该设备需要使用运营商授权的 SIM 卡。请参阅准备 UICC。

### Android 设备配置
1.将设备恢复出厂设置：设置 > 备份和重置 > 恢复出厂设置

警告：这将清空设备中的所有用户数据。

2.将设备的语言设置为英语（美国）：设置 > 语言和输入法 > 语言

3.如果设备具有 GPS 或 WLAN/移动网络功能，则打开位置信息设置：设置 > 位置信息 > 开启

4.连接到满足以下要求的 WLAN 网络：支持 IPv6，可以将被测设备 (DUT) 视为隔离的客户端（请参阅上文的物理环境部分），并可连接到互联网：设置 > WLAN

5.确保设备上未设置锁定图案或密码：设置 > 安全 > 屏幕锁定 > 无

6.在设备上启用 USB 调试：设置 > 开发者选项 > USB 调试。

注意：在 Android 4.2 及更高版本中，默认情况下会隐藏开发者选项。要显示这些选项，请依次转到设置 > 关于手机，然后点按版本号七次。返回上一屏幕以查找开发者选项。要查看其他详细信息，请参阅启用设备上的开发者选项。

7.确保将时间设置为 12 小时格式：设置 > 日期和时间 > 使用 24 小时制 > 关闭

8.依次选择：设置 > 开发者选项 > 不锁定屏幕 > 开启

9.依次选择：设置 > 开发者选项 > 允许模拟位置 > 开启

注意：此模拟位置设置仅适用于 Android 5.x 和 4.4.x。

10.依次选择：设置 > 开发者选项 > 通过 USB 验证应用 > 关闭
注意：此验证应用步骤在 Android 4.2 中为必需步骤。

11.启动浏览器并关闭任何启动/设置屏幕。

12.使用 USB 数据线连接用于测试设备的台式机

注意：将运行 Android 4.2.2 或更高版本的设备连接到计算机时，系统会显示一个对话框，询问您是否接受允许通过此计算机进行调试的 RSA 密钥。选择“允许 USB 调试”。

13.在设备上安装和配置帮助程序应用。

注意：对于 CTS 版本 2.1 R2 至 4.2 R4，请通过以下命令设置您的设备（或模拟器），以便执行无障碍测试：
adb install -r android-cts/repository/testcases/CtsDelegatingAccessibilityService.apk
在设备上，依次启用：设置 > 无障碍 > 无障碍 > Delegating Accessibility Service

注意：对于 7.0 之前的 CTS 版本，请在声明 android.software.device_admin 的设备上设置您的设备，以使用以下命令执行设备管理测试：
adb install -r android-cts/repository/testcases/CtsDeviceAdmin.apk
依次选择“设置”>“安全”>“选择设备管理器”，然后启用两个 android.deviceadmin.cts.CtsDeviceAdminReceiver* 设备管理器。确保 android.deviceadmin.cts.CtsDeviceAdminDeactivatedReceiver 和任何其他预加载的设备管理器均保持停用状态。

14将 CTS 媒体文件复制到设备上，如下所示：

注意：对于 CTS 2.3 R12 及更高版本，如果设备支持视频编解码器，则必须将 CTS 媒体文件复制到设备上。

cd到下载并解压缩媒体文件的目标路径。

更改文件权限：chmod u+x copy_media.sh

运行 copy_media.sh：

要复制分辨率不超过 720x480 的剪辑，请运行：./copy_media.sh 720x480

如果您不确定最大分辨率，请尝试运行 ./copy_media.sh all，以便复制所有文件。

如果 adb 下有多个设备，请将 -s（序列号）选项添加到末尾。例如，要将分辨率不超过 720x480 的文件复制到序列号为 1234567 的设备，请运行：./copy_media.sh 720x480 -s 1234567

## 四.CTS错误形式
1. assert fail
2. 显示5min或者10min超时

## 五.调试技巧
[QUESTION]
调试CTS测试不过时，CTS的源代码中有很多控制的调试信息，如何打开输出更多的调试信息 

[ANSWER]

比如这样的CTS代码，如何让CTS代码执行到红色部分，这样可以输出更多的调试信息

public class ImageWriterTest extends Camera2AndroidTestCase {

    private static final String TAG = "ImageWriterTest";

    private static final boolean VERBOSE = Log.isLoggable(TAG, Log.VERBOSE);

    private static final boolean DEBUG = Log.isLoggable(TAG, Log.DEBUG);



             outputImage = listenerForWriter.getImage(CAPTURE_IMAGE_TIMEOUT_MS);

             mCollector.expectTrue("ImageWriter 1st output image should match 1st input image",

                    isImageStronglyEqual(cameraImage, outputImage));

             if (DEBUG) {

                 String img1FileName = DEBUG_FILE_NAME_BASE + "/" + maxSize + "_image1_copy.yuv";

                 String outputImg1FileName = DEBUG_FILE_NAME_BASE + "/" + maxSize

                        + "_outputImage2_copy.yuv";

                dumpFile(img1FileName, getDataFromImage(cameraImage));

                dumpFile(outputImg1FileName, getDataFromImage(outputImage));

            }

            ….

        mReaderForWriter = createImageReader(maxSize, format, MAX_NUM_IMAGES, listenerForWriter);

        if (VERBOSE) {

            Log.v(TAG, "Created ImageWriter output ImageReader");

         }

 

先注意看前面的TAG名字是ImageWriterTest，设置这个tag的debug log level就可以了

在测CTS之前


运行这个命令就可以


adb shell setprop log.tag.ImageWriterTest D

运行这个命令就可以打开VERBOSE

Verbose打开了debug自然也打开了

adb shell setprop log.tag.ImageWriterTest V

[QUESTION]
android7.0 cts测试常用命令有哪些？

[ANSWER]
以下总结的是针对7.0 cts测试常用命令，其它android产品可能有差异

若想了解更多的cts测试指令，可以在cts命令行模式中输入：help或者run cts --help-all（比help命令更详细）来了解

1、整测命令：run cts

2、单module测试命令：run cts -m module

3、单class测试命令：run cts -m module -t class

4、单case测试命令：run cts -m module -t class#method

5、对所有fail和未执行case复测命令

首先，会有一份整测报告，然后执行以下操作
（1）输入 l r来查看所有报告的session id
例如：
cts-tf > l r
Session  Pass  Fail  Not Executed  Modules Complete  Result Directory     Test Plan  Device serial(s)     Build ID  Product              
0        2     0     0             2 of 2            2017.04.24_15.06.55  cts        SC98602G10067010341  NRD90M    sp9860g_2h10_native  
1        2     0     0             2 of 2            2017.04.24_15.10.51  cts        SC98602G10067010341  NRD90M    sp9860g_2h10_native  
2        324   8     0             2 of 2            2017.04.24_19.51.49  cts        CVH7N15C14006805     N4F26T    angler              
（2）输入run cts --retry session_id会对所有的fail和未执行项重新进行测试
例如：
cts-tf > run cts --retry 2
08-03 20:04:38 I/TestInvocation: Starting invocation for 'cts' on build '3863512' cts --retry 2 Compatibility Test Suite 3863512 CTS 1501761878616 /home/local/SPREADTRUM/juanjuan.hou/cts/7.0/8/arm/android-cts/tools/./../.. 7.0_r8 cts https://androidpartner.googleapis.com/v1/dynamicconfig/suites/CTS/modules/{module}/version/{version}?key=AIzaSyAbwX5JRlmsLeygY2WWihpIJPXFLueOQ3U 2017.08.03_20.04.38 on device SP7731C11005C130691

6、对多个module进行测试方法
A. 如果是针对已经跑过的进行整合测试，则需要执行以下操作
（1）输入 l r来查看fail项所在报告的session_id
（2）输入a s --session session_id --name subplan_name --result-type status来生成一个subplan xml文件

其中subplan_name就是这个subplan的名字，可以随意起;result-type 可选择三种状态：passed、failed、not_executed，即选择的就是session_id报告中的所有pass、fail、not_executed项


这时会在android-cts/subplans中生成subplan名字的xml文件，里面主要定义了所有的module，也可以对subplan xml文件进行随意编辑。其中exclude标签的表示，不需测试的；include标签的表示需要进行测试的
例如：
cts-tf > a s --session 2 --name fail --result-type failed
08-03 20:14:12 I/SubPlanCreator: Created subplan "fail" at /home/local/SPREADTRUM/juanjuan.hou/cts/7.0/8/arm/android-cts/tools/./../../android-cts/subplans/fail.xml

（3）输入run cts --subplan subplan_name命令便会开始进行测试

B.没有整测报告，只是需要对特定module、case进行测试

可以自行在android-cts目录下新建subplans文件夹；并自定义subplan的xml文件

跑测命令为：run cts --subplan subplan_name

C.可以采用命令--include-filter 进行多module测试
比如：若想测试CtsAadbHostTestCases和CtsAbiOverrideHostTestCases module，命令如下
run cts --include-filter CtsAadbHostTestCases  --include-filter CtsAbiOverrideHostTestCases

7、增加 host log/device log 信息
CTS 在 PC 端生成的 log 只留最后20MB,CTS 全测试时前面的 log 会被冲掉,有时候需要保留完整的测试信息，便于分析问题，可采用下面命令
--max-tmp-logcat-file The maximum size of tmp logcat data to retain, in bytes. Only used if --enable-logcat is set Default: 20971520.单位为byte

例如：

run cts --max-tmp-logcat-file 2000000000 表示支持log最大2G

8、在 fail 时截屏
--screenshot-on-failure

例如：run cts --screenshot-on-failure

9、对于偶现问题,验证时一般需要循环测试所在module,命令如下:
--loop –min-loop-time time
其中time是循环的间隔时间，中间可输入 l r 查看测试结果

例如：run cts -m CtsAadbHostTestCases --loop –min-loop-time time 3000

10、跳过预置条件的检查
因为在测试开始时,CTS 会进行预置条件的检测,若有预置条件没设置,则会停止测试.但是若你只是想测试某些 case,且这些 case 不需要翻墙 wifi/gps,则可以在测试方法最后添加如下参数来逃过预置条件的检测。
run cts *** --skip-precondition
![image][tmp]

[tmp]:data:image/png;base64,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