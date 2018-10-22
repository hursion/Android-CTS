# Trade Federation 测试基础架构
## 简介
Trade Federation（简称 tradefed 或 TF）是一种连续的测试框架，专门用于在 Android 设备上运行测试。TF 可以在本地、在桌面设备上以及在平台检验处运行功能测试。要在 TF 中运行测试，您必须具备两个文件，一个是 Java 测试源文件，另一个是 XML 配置文件。有关示例，请参阅 [RebootTest.java](https://android.googlesource.com/platform/tools/tradefederation/contrib/+/master/src/com/android/example/RebootTest.java) 和 [reboot.xml](https://android.googlesource.com/platform/tools/tradefederation/contrib/+/master/res/config/example/reboot.xml)。
## 开始使用
TF 用户主要担任三种角色：开发者、集成者和测试运行者

## 开发环境
- 创建 TradeFed
在 Android 源代码树的根目录处：
```
source ./build/make/envsetup.sh
lunch <device-target>
make tradefed-all -j8
```
- 使用命令行运行
TradeFed 要求使用 $PATH 中的实用工具 adb：
```
export PATH=$PATH:<path/to/adb>
```
创建 TradeFed 后，tradefed.sh 启动器脚本便可供从您的路径进行访问。要启动 Trade Federation 控制台：
```
tradefed.sh
```

## 9.0 TF example
### 创建测试类
我们来创建一个仅将消息转储到 stdout 的 hello world 测试。Tradefed 测试通常会实现 IRemoteTest 接口。以下为 HelloWorldTest 的实现过程：
```
package com.android.tradefed.example;

import com.android.tradefed.device.DeviceNotAvailableException;
import com.android.tradefed.result.ITestInvocationListener;
import com.android.tradefed.testtype.IRemoteTest;

public class HelloWorldTest implements IRemoteTest {
    @Override
    public void run(ITestInvocationListener listener) throws DeviceNotAvailableException {
        System.out.println("Hello, TF World!");
    }
}
```
请将此示例代码保存到 <tree>/tools/tradefederation/core/prod-tests/src/com/android/tradefed/example/HelloWorldTest.java 并从您的 shell 中重建 tradefed：
```
$m tradefed-all
```
请注意，在实际操作中，上述示例中的 System.out 可能并不直接将输出导向至控制台。虽然这对于此测试示例而言是可接受的，但您应按照日志记录（D、I、R）这一部分所述，在 Trade Federation 中建立日志记录。

如果测试创建失败，请参阅机器设置，并确保您没有漏掉任何步骤。

### 创建配置
Trade Federation 测试可通过创建配置来执行。此配置为 XML 文件，告诉 tradefed 在哪个（或哪些）测试上运行以及要执行哪些其他模块并按何种顺序执行。

我们来为 HelloWorldTest 创建一个新的配置（请注意 HelloWorldTest 的完整类名）：
```
<configuration description="Runs the hello world test">
    <test class="com.android.tradefed.example.HelloWorldTest" />
</configuration>
```
请将此数据保存到位于本地文件系统任意位置的 helloworld.xml 文件上（例如：/tmp/helloworld.xml）。TF 将解析配置 XML 文件（也称为 config）、使用反射功能加载指定的类、对其实例化、将其发送到 IRemoteTest，并调用其 run 方法。

运行配置文件
从您的 shell 中启动 tradefed 控制台：
```
$tradefed.sh
```
确保设备已连接至主机，而且对 tradefed 可见：
```
tf> list devices
Serial            State      Product   Variant   Build   Battery
004ad9880810a548  Available  mako      mako      JDQ39   100
```
您可以使用 run <config> 控制台命令执行配置。请尝试输入：
```
tf> run /tmp/helloworld.xml
05-12 13:19:36 I/TestInvocation: Starting invocation for target stub on build 0 on device 004ad9880810a548
Hello, TF World!
```
您应该可以在终端看到“Hello, TF World！”输出内容(linux上没看到，具体原因不明)

### 将配置文件添加到类路径
为了方便部署，您还可以将配置文件捆绑到 tradefed JAR 文件自身中。Tradefed 将自动识别类路径下的“config”文件夹中存放的所有配置。

为进行详细说明，我们将 helloworld.xml 文件移到 tradefed 核心库 (<tree>/tools/tradefederation/core/prod-tests/res/config/example/helloworld.xml) 中。重建 tradefed，重启 tradefed 控制台，然后请求 tradefed 显示类路径中的配置列表：
```
tf> list configs
[…]
example/helloworld: Runs the hello world test
```
您现在可以使用以下命令运行 helloworld 配置文件：

tf> run example/helloworld
05-12 13:21:21 I/TestInvocation: Starting invocation for target stub on build 0 on device 004ad9880810a548
Hello, TF World!

### 与设备交互
可以通过实现 IDeviceTest 接口来获得 Android 设备引用。以下为展示此步骤的实现示例：
```
public class HelloWorldTest implements IRemoteTest, IDeviceTest {
    private ITestDevice mDevice;
    @Override
    public void setDevice(ITestDevice device) {
        mDevice = device;
    }

    @Override
    public ITestDevice getDevice() {
        return mDevice;
    }
…
}
```
在调用 IRemoteTest#run 方法之前，Trade Federation 框架将通过 IDeviceTest#setDevice 方法将 ITestDevice 引用注入到测试中。

我们来修改下 HelloWorldTest 输出消息，以显示设备的序列号：
```
@Override
public void run(ITestInvocationListener listener) throws DeviceNotAvailableException {
    System.out.println("Hello, TF World! I have device " + getDevice().getSerialNumber());
}
```
现在重新编译 tradefed 并检查设备列表：
```
tradefed.sh

tf> list devices
Serial            State      Product   Variant   Build   Battery
004ad9880810a548  Available  mako      mako      JDQ39   100
```
记下列为 Available 的序列号；表示应分配到 HelloWorld 的设备：
```
tf> run example/helloworld
05-12 13:26:18 I/TestInvocation: Starting invocation for target stub on build 0 on device 004ad9880810a548
Hello, TF World! I have device 004ad9880810a548
```
您应该可以看到显示设备序列号的新输出消息。

### 发送测试结果
IRemoteTest 会对提供给 #run 方法的 实例调用相关方法，从而报告结果。TF 框架本身负责报告每个调用的开始位置（通过 ITestInvocationListener#invocationStarted）和结束位置（通过 ITestInvocationListener#invocationEnded）。

测试运行是测试的逻辑集合。要报告测试结果，IRemoteTest 负责报告测试运行的开始之处，每个测试的开始和结束之处以及测试运行的结束之处。

单次测试结果为失败的 HelloWorldTest 实现可能如下所示。
```
@Override
public void run(ITestInvocationListener listener) throws DeviceNotAvailableException {
    System.out.println("Hello, TF World! I have device " + getDevice().getSerialNumber());

   TestDescription testId = new TestDescription("com.example.TestClassName", "sampleTest");
        listener.testRunStarted("helloworldrun", 1);
        listener.testStarted(testId);
        listener.testFailed(testId, "oh noes, test failed");
        listener.testEnded(testId, Collections.emptyMap());
        listener.testRunEnded(0, Collections.emptyMap());
}
```
TF 包括几个可以重复使用的 IRemoteTest 实现，因而您无需从头开始编写您自己的实现。例如，InstrumentationTest 可在 Android 设备上远程运行 Android 应用测试、解析结果，并将这些结果转发到 ITestInvocationListener。

### 存储测试结果
TF 配置的默认测试监听器实现为 TextResultReporter，它会将调用结果转储到 stdout。请运行上一节中的 HelloWorldTest 配置，以进行详细说明：
```
./tradefed.sh

tf> run example/helloworld
05-16 20:03:15 I/TestInvocation: Starting invocation for target stub on build 0 on device 004ad9880810a548
Hello, TF World! I have device 004ad9880810a548
05-16 20:03:15 I/InvocationToJUnitResultForwarder: run helloworldrun started: 1 tests
Test FAILURE: com.example.TestClassName#sampleTest
 stack: oh noes, test failed
05-16 20:03:15 I/InvocationToJUnitResultForwarder: run ended 0 ms
```
要将调用结果存储在其他位置（如某个文件中），请在配置中使用 result_reporter 标签来指定自定义 ITestInvocationListener 实现。

TF 还包括 XmlResultReporter 监听器，该监听器会将测试结果写入 XML 文件，并且所采用的格式与 ant JUnit XML 写入器所采用的格式类似。要在配置中指定 result_reporter，请修改 …/res/config/example/helloworld.xml 配置：
```
<configuration description="Runs the hello world test">
    <test class="com.android.tradefed.example.HelloWorldTest" />
    <result_reporter class="com.android.tradefed.result.XmlResultReporter" />
</configuration>
```
现在重新编译 tradefed 并重新运行 hello world 示例：
```
tf> run example/helloworld
05-16 21:07:07 I/TestInvocation: Starting invocation for target stub on build 0 on device 004ad9880810a548
Hello, TF World! I have device 004ad9880810a548
05-16 21:07:07 I/XmlResultReporter: Saved device_logcat log to /tmp/0/inv_2991649128735283633/device_logcat_6999997036887173857.txt
05-16 21:07:07 I/XmlResultReporter: Saved host_log log to /tmp/0/inv_2991649128735283633/host_log_6307746032218561704.txt
05-16 21:07:07 I/XmlResultReporter: XML test result file generated at /tmp/0/inv_2991649128735283633/test_result_536358148261684076.xml. Total tests 1, Failed 1, Error 0
```
请留意表明已生成 XML 文件的日志消息；所生成的文件应如下所示：
```
<?xml version='1.0' encoding='UTF-8' ?>
<testsuite name="stub" tests="1" failures="1" errors="0" time="9" timestamp="2011-05-17T04:07:07" hostname="localhost">
  <properties />
  <testcase name="sampleTest" classname="com.example.TestClassName" time="0">
    <failure>oh noes, test failed
    </failure>
  </testcase>
</testsuite>
```
您还可以编写您自己的自定义调用监听器 - 它们只需要实现 ITestInvocationListener 接口即可。

Tradefed 支持多个调用监听器，因此您可以将测试结果发送到多个独立的目的地。只需在配置中指定多个 <result_reporter> 标签，即可完成此步骤。

### 日志记录
TF 的日志记录设备具有以下功能：

从设备捕获日志（也称为设备 logcat）
记录在主机上运行的 TradeFederation 框架中的日志（也称为主机日志）
TF 框架自动从分配的设备中捕获 logcat，并将其发送到调用监听器以进行处理。 然后 XmlResultReporter 将捕获的设备 logcat 保存为文件。

系统使用 ddmlib 日志类的 CLog 封装容器报告主机日志。让我们将 HelloWorldTest 中之前的 System.out.println 调用转换为 CLog 调用：
```
@Override
public void run(ITestInvocationListener listener) throws DeviceNotAvailableException {
    CLog.i("Hello, TF World! I have device %s", getDevice().getSerialNumber());
```
CLog 直接处理字符串插入，类似于 String.format。当您在重建和重新运行 TF 时，您应该可以在 stdout 上看到此日志消息：
```
tf> run example/helloworld
…
05-16 21:30:46 I/HelloWorldTest: Hello, TF World! I have device 004ad9880810a548
…
```
默认情况下，tradefed 会将主机日志消息输出到 stdout。TF 还包括将消息写入文件的日志实现：FileLogger。要添加文件日志记录，请将 logger 标签添加到配置中，指定 FileLogger 的完整类名：
```
<configuration description="Runs the hello world test">
    <test class="com.android.tradefed.example.HelloWorldTest" />
    <result_reporter class="com.android.tradefed.result.XmlResultReporter" />
    <logger class="com.android.tradefed.log.FileLogger" />
</configuration>
```
现在，再次重新编译并运行 helloworld 示例：
```
tf >run example/helloworld
…
05-16 21:38:21 I/XmlResultReporter: Saved device_logcat log to /tmp/0/inv_6390011618174565918/device_logcat_1302097394309452308.txt
05-16 21:38:21 I/XmlResultReporter: Saved host_log log to /tmp/0/inv_6390011618174565918/host_log_4255420317120216614.txt
…
```
该日志消息指出了主机日志的路径，当您查看该日志时，其中应当包含您的 HelloWorldTest 日志消息：
```
more /tmp/0/inv_6390011618174565918/host_log_4255420317120216614.txt
```
输出示例：
```
…
05-16 21:38:21 I/HelloWorldTest: Hello, TF World! I have device 004ad9880810a548
```
### 处理选项
从 TF 配置中加载的对象（也称为配置对象）亦可通过使用 @Option 注解接收命令行参数中的数据。

要参与其中，配置对象类会将 @Option 注解应用于相关成员字段，并为其指定一个唯一的名称。这样您便可以通过命令行选项填充该成员字段值（并自动将该选项添加到配置帮助系统）。

注意：部分字段类型可能不受支持。要了解受支持的字段类型，请参阅 OptionSetter。

我们将 @Option 添加到 HelloWorldTest 中：
```
@Option(name="my_option",
        shortName='m',
        description="this is the option's help text",
        // always display this option in the default help text
        importance=Importance.ALWAYS)
private String mMyOption = "thisisthedefault";
```
接下来，我们添加一条日志消息来显示 HelloWorldTest 中的选项的值，以便证明已正确接收该值：
```
@Override
public void run(ITestInvocationListener listener) throws DeviceNotAvailableException {
    …
    CLog.logAndDisplay(LogLevel.INFO, "I received option '%s'", mMyOption);
```
最后，重新编译 TF 并运行 helloworld；您应该会看到一条带有 my_option 默认值的日志消息：
```
tf> run example/helloworld
…
05-24 18:30:05 I/HelloWorldTest: I received option 'thisisthedefault'
```
从命令行传递值
为 my_option 传入值；您应该可以看到使用此值填充的 my_option：
```
tf> run example/helloworld --my_option foo
…
05-24 18:33:44 I/HelloWorldTest: I received option 'foo'
```
TF 配置还包括帮助系统，该系统会自动显示 @Option 字段的帮助文本。现在试试看吧，您应该可以看到 my_option 的帮助文本：
```
tf> run example/helloworld --help
Printing help for only the important options. To see help for all options, use the --help-all flag

  cmd_options options:
    --[no-]help          display the help text for the most important/critical options. Default: false.
    --[no-]help-all      display the full help text for all options. Default: false.
    --[no-]loop          keep running continuously. Default: false.

  test options:
    -m, --my_option      this is the option's help text Default: thisisthedefault.

  'file' logger options:
    --log-level-display  the minimum log level to display on stdout. Must be one of verbose, debug, info, warn, error, assert. Default: error.
```
请注意有关“仅输出重要选项的帮助文本”的消息。为了减少选项帮助的混乱情况，TF 使用 Option#importance 属性来确定是否在指定 --help 时显示特定的 @Option 字段帮助文本。无论字段重要与否，--help-all 始终显示针对所有 @Option 字段的帮助。有关详情，请参阅 Option.Importance。

从配置传递值
您还可以通过添加 <option name="" value=""> 元素在配置文件中指定“选项”值。使用 helloworld.xml 进行测试：
```
<test class="com.android.tradefed.example.HelloWorldTest" >
    <option name="my_option" value="fromxml" />
</test>
```
重建和运行 helloworld 后现在应产生以下输出内容：
```
05-24 20:38:25 I/HelloWorldTest: I received option 'fromxml'
```
配置帮助也应经过更新以显示 my_option 的默认值：
```
tf> run example/helloworld --help
  test options:
    -m, --my_option      this is the option's help text Default: fromxml.
```
helloworld 配置中包含的其他配置对象（如 FileLogger）也接受选项。选项 --log-level-display 比较有意思，因为它会过滤在 stdout 上显示的日志。在本教程前面的部分中，您可能已经注意到：当我们改用 FileLogger 后，stdout 上不再显示“Hello, TF World! I have device …”这一日志消息。您可以通过传入 --log-level-display 参数提高 stdout 上日志记录的详细程度。

请立即尝试，您应该可以看到“I have device”这一日志消息再次出现在 stdout 上，并被记录到某个文件中：
```
tf> run example/helloworld --log-level-display info
…
05-24 18:53:50 I/HelloWorldTest: Hello, TF World! I have device 004ad9880810a548
```