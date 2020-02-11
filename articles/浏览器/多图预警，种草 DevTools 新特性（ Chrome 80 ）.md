##多图预警，种草 DevTools 新特性（ Chrome 80 ）

### 一、在控制台中支持 let 和 class 重新声明

控制台现在支持 `let` 和 `class` 语句的重新声明，当我们在使用控制台调试 JavaScript 代码代码段时，对于已通过 `let` 或 `class` 声明的，无法再次声明是一个普遍的烦恼。

> 在控制台外部或单个控制台输入的脚本中，重新声明 let 或 class 语句仍然会导致 SyntaxError 。

例如，当 `x` 已被 `let` 声明，当再次使用 `let` 声明这个局部变量时，控制台将抛出一个错误：

![](http://resource.muyiy.cn/image/20200211192815.png)

现在，控制台允许重新声明：

![](http://resource.muyiy.cn/image/20200211192911.png)

### 二、改进WebAssembly调试

DevTools 已经开始支持 DWARF 调试标准，这意味着在 DevTools 中增加了对代码步进、设置断点和解析源语言中的堆栈跟踪的支持。在 Chrome DevTools 中查看改进的 WebAssembly 调试的完整事件。

![](http://resource.muyiy.cn/image/20200211193035.png)

### 三、Network 更新

#### 1. 在启动程序选项卡中请求启动程序链

现在，你可以将网络请求的启动器和依赖项作为嵌套列表查看，这可以帮助你理解为什么要请求某个资源，或者某个资源(例如脚本)导致了什么网络活动。

![](http://resource.muyiy.cn/image/20200211193720.png)

在网络面板中记录网络活动之后，单击资源，然后转到Initiator选项卡查看其请求引发程序链：

- 检查的资源是粗体。在上面的截图中，https://web.dev/default-627898b5.js 是被检查的资源；
- 被检查资源之上的资源是发起者。在上面的截图中，https://web.dev/bootstrap.js 是 https://web.dev/default-627898b5.js 的发起人，换句话说，是 https://web.dev/bootstrap.js 引起了 https://web.dev/default-627898b5.js 的网络请求；
- 被检查资源下面的资源是依赖关系。在上面的截图中，https://web.dev/chunk-f34f99f7.js 是 https://web.dev/default-627898b5.js 的属地换句话说，是 https://web.dev/default-627898b5.js 引起了 https://web.dev/chunk-f34f99f7.js 的网络请求；

> 还可以通过按住Shift并将鼠标悬停在网络资源上来访问启动程序和依赖项信息。

#### 2. 在 Overview 中突出显示选中的网络请求

在单击网络资源以便对其进行检查之后，Network 面板现在在 **Overview** 中在该资源周围放置了一个蓝色框。 这可以帮助你检测网络请求是否比预期的发生得早或更晚。

![](http://resource.muyiy.cn/image/20200211194415.png)

#### 3. Network 面板中显示 Url 列 和 Path 列

使用 Network 面板中新的 Path 列和 Url 列查看每个网络资源的绝对路径或完整URL。

![](http://resource.muyiy.cn/image/20200211194747.png)

#### 4. 更新了 User-Agent 字符串

DevTools支持通过Network Conditions 选项卡设置自定义 User-Agent 字符串，User-Agent 字符串会影响连接到网络资源的 `User-Agent` HTTP 标头以及 `navigator.userAgent` 的值。

预定义的 User-Agent 字符串已更新，以反映现代浏览器版本。

![](http://resource.muyiy.cn/image/20200211195421.png)

> 你还可以在 Device 模式下设置 User-Agent 字符串。

### 四、Audits 更新

#### 新的配置界面

配置 UI 采用了新的响应式设计，并且节流配置选项也得到了简化。查看  [Audits Panel Throttling](https://github.com/GoogleChrome/lighthouse/blob/master/docs/throttling.md#devtools-audits-panel-throttling)  了解更多变更信息。

![](http://resource.muyiy.cn/image/20200211195639.png)

### 五、覆盖标签更新

#### 1. 按功能或按块覆盖模式

Coverage 选项卡有一个新的下拉菜单，允许你指定应该为每个函数还是每个块收集代码覆盖率数据。每个块的覆盖范围更详细，但收集起来也更昂贵。DevTools现在默认使用每个函数覆盖。

> 在HTML文件中，你可能会看到较大的代码覆盖率差异，具体取决于你是按功能使用还是按块模式使用。 使用按功能模式时，HTML文件中的内联脚本被视为功能。 如果脚本完全执行，则DevTools会将整个脚本标记为已使用的代码。 只有脚本完全不执行时，DevTools才会将该脚本标记为未使用的代码。

![](http://resource.muyiy.cn/image/20200211200012.png)

#### 2. 必须通过页面重新加载来启动覆盖率

由于覆盖率数据不可靠，在不重载页面的情况下切换代码覆盖率已经被移除。例如，如果一个函数的执行时间很长，并且V8的垃圾收集器已经清理了它，那么这个函数可以被报告为未使用。                                                                                                                                   

