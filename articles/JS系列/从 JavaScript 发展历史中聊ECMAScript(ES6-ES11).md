### 前言

JavaScript 是当今使用最广泛的、发展最好的前后端（后端主要是 Nodejs）语言，如果我们想要灵活使用 JavaScript，我们首先需要了解的就是 JavaScript 和 ECMAScript(ES) 的基础知识及发展历程。

### 一、JavaScript 的诞生

JavaScript 因为互联网而生，紧跟着浏览器的出现而问世。回顾它的历史，就要从浏览器的历史讲起。

1990年底，欧洲核能研究组织（CERN）科学家 Tim Berners-Lee，在全世界最大的电脑网络——互联网的基础上，发明了万维网（World Wide Web），从此可以在网上浏览网页文件。最早的网页只能在操作系统的终端里浏览，也就是说只能使用命令行操作，网页都是在字符窗口中显示，这当然非常不方便。

1992年底，美国国家超级电脑应用中心（NCSA）开始开发一个独立的浏览器，叫做 Mosaic。这是人类历史上第一个浏览器，从此网页可以在图形界面的窗口浏览。

1994年10月，NCSA 的一个主要程序员 Marc Andreessen 联合风险投资家 Jim Clark，成立了 Mosaic 通信公司（Mosaic Communications），不久后改名为 Netscape。这家公司的方向，就是在 Mosaic 的基础上，开发面向普通用户的新一代的浏览器 Netscape Navigator。

1994年12月，Navigator 发布了1.0版，市场份额一举超过90%。

Netscape 公司很快发现，Navigator 浏览器需要一种可以嵌入网页的脚本语言，用来控制浏览器行为。当时，网速很慢而且上网费很贵，有些操作不宜在服务器端完成。比如，如果用户忘记填写“用户名”，就点了“发送”按钮，到服务器再发现这一点就有点太晚了，最好能在用户发出数据之前，就告诉用户“请填写用户名”。这就需要在网页中嵌入小程序，让浏览器检查每一栏是否都填写了。

管理层对这种浏览器脚本语言的设想是：功能不需要太强，语法较为简单，容易学习和部署。那一年，正逢 Sun 公司的 Java 语言问世，市场推广活动非常成功。Netscape 公司决定与 Sun 公司合作，浏览器支持嵌入 Java 小程序（后来称为 Java applet）。但是，浏览器脚本语言是否就选用 Java，则存在争论。后来，还是决定不使用 Java，因为网页小程序不需要 Java 这么“重”的语法。但是，同时也决定脚本语言的语法要接近 Java，并且可以支持 Java 程序。这些设想直接排除了使用现存语言，比如 Perl、Python 和 TCL。

1995年，Netscape 公司雇佣了程序员 Brendan Eich 开发这种网页脚本语言。Brendan Eich 有很强的函数式编程背景，希望以 Scheme 语言（函数式语言鼻祖 LISP 语言的一种方言）为蓝本，实现这种新语言。

1995年5月，Brendan Eich 只用了10天，就设计完成了这种语言的第一版。它是一个大杂烩，语法有多个来源。

- 基本语法：借鉴 C 语言和 Java 语言。
- 数据结构：借鉴 Java 语言，包括将值分成原始值和对象两大类。
- 函数的用法：借鉴 Scheme 语言和 Awk 语言，将函数当作第一等公民，并引入闭包。
- 原型继承模型：借鉴 Self 语言（Smalltalk 的一种变种）。
- 正则表达式：借鉴 Perl 语言。
- 字符串和数组处理：借鉴 Python 语言。

为了保持简单，这种脚本语言缺少一些关键的功能，比如块级作用域、模块、子类型（subtyping）等等，但是可以利用现有功能找出解决办法。这种功能的不足，直接导致了后来 JavaScript 的一个显著特点：对于其他语言，你需要学习语言的各种功能，而对于 JavaScript，你常常需要学习各种解决问题的模式。而且由于来源多样，从一开始就注定，JavaScript 的编程风格是函数式编程和面向对象编程的一种混合体。

Netscape 公司的这种浏览器脚本语言，最初名字叫做 Mocha，1995年9月改为 LiveScript。12月，Netscape 公司与 Sun 公司（Java 语言的发明者和所有者）达成协议，后者允许将这种语言叫做 JavaScript。这样一来，Netscape 公司可以借助 Java 语言的声势，而 Sun 公司则将自己的影响力扩展到了浏览器。

之所以起这个名字，并不是因为 JavaScript 本身与 Java 语言有多么深的关系（事实上，两者关系并不深，详见下节），而是因为 Netscape 公司已经决定，使用 Java 语言开发网络应用程序，JavaScript 可以像胶水一样，将各个部分连接起来。当然，后来的历史是 Java 语言的浏览器插件失败了，JavaScript 反而发扬光大。

1995年12月4日，Netscape 公司与 Sun 公司联合发布了 JavaScript 语言，对外宣传 JavaScript 是 Java 的补充，属于轻量级的 Java，专门用来操作网页。

1996年3月，Navigator 2.0 浏览器正式内置了 JavaScript 脚本语言。

### 二、JavaScript 与 ECMAScript 的关系

要讲清楚这个问题，需要回顾历史。1996年11月，JavaScript 的创造者 Netscape 公司，决定将 JavaScript 提交给国际标准化组织ECMA，希望这种语言能够成为国际标准。次年，ECMA 发布 262 号标准文件（ECMA-262）的第一版，规定了浏览器脚本语言的标准，并将这种语言称为 ECMAScript ，这个版本就是1.0版。

该标准从一开始就是针对 JavaScript 语言制定的，但是之所以不叫 JavaScript ，有两个原因。一是商标，Java是 Sun 公司的商标，根据授权协议，只有 Netscape 公司可以合法地使用 JavaScript 这个名字，且 JavaScript 本身也已经被 Netscape 公司注册为商标。二是想体现这门语言的制定者是 ECMA ，不是 Netscape ，这样有利于保证这门语言的开放性和中立性。

**因此，ECMAScript 和 JavaScript 的关系是，ECMAScript 是一个简单的 JavaScript 标准规范，JavaScript 是 ECMAScript 的一种实现（另外的 ECMAScript 方言还有 JScript 和 ActionScript ）。并且，ECMAScript 持续不断的为 JavaScript 添加新功能。**

从 1997年7月 ECMAScript 1.0发布到现在，ECMAScript 已经正式发布了 11 版，下面我们主要介绍从ES6（ES2015）到ES11（最新 ES2020 ）期间，每版发布的新功能。

### 三、ECMAScript 6(2015)-ECMAScript 10(2019)

[ES6、ES7、ES8、ES9、ES10新特性一览](https://juejin.im/post/5ca2e1935188254416288eb2)

### 四、ECMAScript 11(2020)

[种草 ES2020 新特性](https://juejin.im/post/5e09ca40518825499a5abff7)