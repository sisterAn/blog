### 前言

接口调用是每一个软件开发从业者必不可少的一项技能，一个项目的完成必然经过大量的接口测试，实际开发过程中，接口调试的时间不比实际开发所用的时间少。

作为前端开发人员，我们通常使用 Postman（或 PostWoman ） 工具来进行REST API调用。

### 一、Postman 限制

Postman 用来调试接口很方便，但它有很多限制。

#### 1. 需要开启额外的软件，并且占用大量的 RAM

我们已经使用 VSCode 开发项目，为什么我们还需要额外打开 Postman 去调试接口喃？并且 Postman 运行占用了大量的 RAM，这对 RAM 紧缺的设备来说，尤为重要。

#### 2. 高级功能要付费

进行调用测试 API 是可以的，但是如果你要编辑、版本化或仅与你的团队共享它，则不是很方便。

当然，你可以使用 Postman 付费计划，但这意味着你需要付费，并且你所在的所有团队都需要使用 Postman ，这工程量就很大了！

### 二、你知道 REST Client 吗？

![](http://resource.muyiy.cn/image/20200116201023.png)

REST Client 是一个 VS Code 扩展插件，它允许你发送 HTTP 请求并直接在 VS Code 上查看响应结果。

由于它是基于文本格式，所以我们可以轻松的在存储库之间进行版本控制。👍

### 三、Postman 与 REST Client

#### 优点

- 能够进行版本化并可以在团队间共享你的 API 调用；
- 仅仅是一个 HTTP 文件，团队成员间可以通过 git 协作维护这个文件；
- 无需切换窗口，可以在一个 HTTP 文件中去维护多个接口；
- 相比于Postman，REST Client支持了 [cURL](https://link.zhihu.com/?target=https%3A//en.wikipedia.org/wiki/CURL) 和 [RFC 2616](https://link.zhihu.com/?target=https%3A//www.w3.org/Protocols/rfc2616/rfc2616-sec5.html) 两种标准来调用REST API；

#### 缺点

- 你必须使用 VS Code，如果使用其它的工具开发是不可以的；
- Postman 拥有强大的用户操作界面，而 REST Client 没有操作界面，仅仅是在一个 HTTP 文件中编写请求，没有 Postman 方便；

### 四、REST Client

#### 1. 常规

**步骤一：安装 REST Client 插件**

**步骤二： 创建一个 `.http` 或 `.rest` 文件**

**步骤三：写入测试接口**

1. 符合 RFC 2616 标准的 POST 请求

```js
POST http://dummy.restapiexample.com/api/v1/create HTTP/1.1
content-type: application/json

{
    "name":"Hendry",
    "salary":"61888",
    "age":"26"
}
```

2. 符合 cURL 标准的 POST 请求

```js
curl -X POST "http://dummy.restapiexample.com/api/v1/create" -d "Hello World"
```

如果省略 `request` 方法，请求将被视为 `GET` 。

注意：接口间通过 `###` 分隔，同一个 `.http` 文件里可以涵盖多个 HTTP 请求。不像 Postman，不同的 HTTP 请求需要放在不同的 tab 里。

**步骤四：发送请求，测试接口**

点击 `Send Request` ，或者右键选择 `Send Request` ，或者直接用快捷键 `Ctrl+Alt+R`（或`Cmd+Alt+R`） ，你的 REST API 就执行了，然后 API Response 就会显示在右边区域。

![](http://resource.muyiy.cn/image/20200116201054.png)

#### 2. 更远一步

一个 http 文件可能定义许多请求和文件级自定义变量，很难找到你想要的请求/变量。我们可以使用快捷键 `Ctrl+Shift+O`（或`Cmd+Shift+O`）来切换请求/变量。



**自定义环境变量**

Code  => Preferences => Settings 打开设置，切换到 `Workspace Settings` ，配置 `settings.json` 文件：

```js
{
  "rest-client.environmentVariables": {
    "$shared": {
        "version": "v1",
        "prodToken": "foo",
        "nonProdToken": "bar"
    },
    "local": {
        "version": "v2",
        "host": "localhost",
        "dummyhost": "local",
        "token": "{{$shared nonProdToken}}",
        "secretKey": "devSecret"
    },
    "production": {
        "host": "api.apiopen.top",
        "dummyhost": "dummy.restapiexample.com",
        "token": "{{$shared prodToken}}",
        "secretKey" : "prodSecret"
    }
  }
}
```

可以切换不同的环境（Ctrl+Alt+E 或 Cmd+Alt+E），调用相应的配置项（`host` 、 `token` 等）。

```js
### 测试接口 RFC 2616
// host 为环境变量
GET https://{{host}}/musicRankings HTTP/1.1
```



**自定义文件变量**

- 你可以在 HTTP 文件任意位置定义文件变量，它们都可以在整个文件的任何请求中引用；

- 对于文件变量，定义遵循 `@variableName = variableValue` 占用完整行的语法；
- 变量名称（`variableName`）不得包含任何空格。至于变量值（`variableValue`），它可以由任何字符组成，甚至允许空格（前导和尾随空格将被剥离）；
- 如果你想保留一些像换行符这样的特殊字符，你可以使用反斜杠 `\n` ；

```js
// 文件变量
@port = 8080
@contentType = application/json

### 测试接口 RFC 2616
// 文件变量
@name = musicRankings
GET https://{{host}}/{{name}} HTTP/1.1
```



**自定义请求变量**

当单个请求结果的值需要被其它请求使用时，可以使用请求变量，格式为：`@name newname` ，请求变量引用语法为 `{{requestName.(response|request).(body|headers).(JSONPath|XPath|Header Name)}}`。

```js
@contentType = application/json

###

# @name createComment
POST https://{{host}}/comments HTTP/1.1
Content-Type: {{contentType}}

###

# @name getCreatedComment

GET https://{{host}}/comments/{{createComment.response.body.$.id}} HTTP/1.1

Authorization: {{login.response.headers.X-AuthToken}}
```



**系统变量**

系统自带的一些变量，使用系统变量需要有 `$`符号

- `{{$guid}}` 唯一识别号
- `{{$randomInt min max}}` 返回一个`min` 和 `max` 之间的随机数
- `{{$timestamp [offset option]}}`：添加`UTC`时间戳。
- `{{$timestamp number option}}`，例如3小时前`{{$timestamp -3 h}}`;代表后天`{{$timestamp 2 d}}`。

更多系统变量用法请参考 [官方文档](https://marketplace.visualstudio.com/items?itemName=humao.rest-client#overview)

![](http://resource.muyiy.cn/image/20200116201113.png)

**更多功能等待你的挖掘，详见[vscode-restclient](https://github.com/Huachao/vscode-restclient#environment-variables)！！！👏👏👏**

### 五、总结

Postman 有口皆碑，确实是一个不错的工具，但 REST Client 也值得进行尝试，毕竟连后端也已经推出了  IDEA REST Client， 在和同事进行协作开发时，在项目中增加一个 `.http` 接口请求文件，方便自己的同时方便其他人。

如果觉得不错，就分享给你的小伙伴吧！