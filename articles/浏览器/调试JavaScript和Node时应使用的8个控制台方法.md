超越console.log，学习从未用于调试的控制台功能！

![console](https://miro.medium.com/max/6600/1*0d5jlHyPf6IIrSnodPb3mA.png)



### console API

**每个** JavaScript开发人员都使用过`console.log(‘text’)`。该`console`模块是JavaScript中最常见的实用程序之一，而API在Node中实现：

> 提供了一个简单的调试控制台，类似于Web浏览器提供的JavaScript控制台机制。

这是在[Node.js文档](https://nodejs.org/dist/latest-v12.x/docs/api/console.html)页面上为控制台模块written 编写的定义。但是，初学者倾向于在不使用新技术的情况下就咨询在线教程，而不是阅读文档，从而失去了学习如何正确利用此新工具的100％潜力的机会。

在谈到控制台API，通常新手只使用一些功能，如👌 `console.log()`*，⚠️* `console.warn()`*，*或❌ `console.error()` 调试他们的应用程序的同时，往往也有其可以完美地实现我们的要求，提高调试效率许多其他的方法。

本文旨在`console`通过我在[**Codeworks**](https://codeworks.me/?utm_source=medium&utm_medium=organic&utm_campaign=marco_ghiani_hackernoon_learning_nodejs_5_tips)授课时使用的相关示例，展示一些最有趣的方法**。**因此，让我们从“控制台”模块中查看8种最佳功能的列表！

**以下所有方法在全局实例中都可用**`**console**`**，因此不需要控制台模块。**

## 1）[console.assert](https://nodejs.org/dist/latest-v12.x/docs/api/console.html#console_console_assert_value_message) ✅

该`console.assert`函数用于测试传递的参数是真还是假值。在传递的值为false的情况下，该函数记录在第一个参数之后传递的额外参数，否则，代码执行将继续进行，而不会记录任何日志。

![img](https://miro.medium.com/max/60/1*gB2TX8yPs05RZXWjjlIjvw.png?q=20)

这两种情况都是真实或虚假的断言。

当您想要检查值的存在同时保持控制台干净（避免记录较长的属性列表等）时，assert方法特别有用。

## 2）[console.count](https://nodejs.org/dist/latest-v12.x/docs/api/console.html#console_console_count_label)和[console.countReset](https://nodejs.org/dist/latest-v12.x/docs/api/console.html#console_console_countreset_label) 💯

这两种方法用于设置和清除计数器，以记录特定字符串在控制台中的登录次数：

![img](https://miro.medium.com/max/60/1*bgOpNwOlWOljyH5Q-RvYOQ.png?q=20)

计算并重置“ Hello”字符串的日志出现次数。

## 3）[console.group](https://nodejs.org/dist/latest-v12.x/docs/api/console.html#console_console_group_label)和[console.groupEnd](https://nodejs.org/dist/latest-v12.x/docs/api/console.html#console_console_groupend) 🎳

`.group`并`.groupEnd`在控制台中创建并结束一组日志。您可以传递标签作为`.group()`描述所关注内容的第一个参数：

![img](https://miro.medium.com/max/60/1*yOnPTlsOx0oyD05GlRAobQ.jpeg?q=20)

三组描述家庭角色。

## 4）[console.table](https://nodejs.org/dist/latest-v12.x/docs/api/console.html#console_console_table_tabulardata_properties) 📋

此特定方法对于描述人性化的表中的对象或数组内容非常有用：

![img](https://miro.medium.com/max/60/1*VjNJVAF6nTkRykoDBY2B8g.png?q=20)

用户对象列表表。

`console.table` 使嵌套和复杂的数组/对象的检查和记录变得更加容易。

## 5）[console.time](https://nodejs.org/dist/latest-v12.x/docs/api/console.html#console_console_time_label)和[console.timeEnd](https://nodejs.org/dist/latest-v12.x/docs/api/console.html#console_console_timeend_label) ⏱

如果您想在执行时检查代码的性能，并解决该问题，您可以`Date`使用API 创建一个开始时间戳，并使用它来计算代码执行后的差异？像这样：

![img](https://miro.medium.com/max/60/1*L1drTd-MiNSlAWlxO5rBNg.png?q=20)

好了，使用`time`和`timeEnd`函数，没有必要执行此技巧。您只需执行以下操作即可创建时序报告：

![img](https://miro.medium.com/max/60/1*ALu56icy-rJTeisq0Wp-8w.png?q=20)

如您所见，console.time还返回了更准确的结果。

# 摘要

现在只有3分钟的时间，您现在可以在Console API中使用更多出色的工具。将它们与您的调试习惯集成在一起，您的开发速度将成倍提高！