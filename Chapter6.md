
# ASP.NET MVC 随想录（6）——漫谈 OWIN 

## 什么是 OWIN

OWIN 是 Open Web Server Interface for .NET 的首字母缩写，他的定义如下：

> OWIN 在.NET Web Servers 与 Web Application 之间定义了一套标准接口，OWIN 的目标是用于解耦 Web Server 和 Web Application。基于此标准，鼓励开发者开发简单、灵活的模块，从而推进.NET Web Development 开源生态系统的发展。

正如你看到的这样，OWIN 是接口、契约，而非具体的代码实现，仅仅是规范([**specifications**][1])，所以要实现自定义基于 OWIN 的 Web Server 必须要实现此规范。

历时两年（2010-2012），OWIN 的规范终于完成并且当前版本是 1.0，在 OWIN 的官网上可以看到更具体的信息。

## 为什么我们需要 OWIN

过去，IIS 作为.NET 开发者来说是最常用的 Web Server（没有之一），源于微软产品的**紧耦合**关系，我们不得不将 Website、Web Application、Web API 等部署在 IIS 上，事实上在 2010 年前并没有什么不妥，但随着近些年来 Web 的发展，特别是移动互联网飞速发展，IIS 作为 Web Server 已经暴露出他的不足了。主要体现在两个方面，ASP.NET (System.Web)紧耦合 IIS，IIS 紧耦合 OS，这就意味着，我们的 Web Framework 必须部署在微软的操作系统上，难以**跨平台**。

### ASP.NET 和 IIS 

我们知道，不管是 ASP.NET MVC 还是 ASP.NET WEB API 等都是基于 ASP.NET Framework 的，这种关系从前缀就可以窥倪出来。而 ASP.NET 的核心正是 System.Web 这个程序集，而且 System.Web紧耦合 IIS，他存在于.NET Framework 中。所以，这导致了 Web Framework 严重的局限性：

* ASP.NET 的核心 System.Web，而 System.Web 紧耦合 IIS
* System.Web 是.NET Framework 重要组成，已有 15 年以上历史，沉重、冗余，性能差，难于测试，约 2.5M
* System.Web 要更新和发布新功能必须等待.NET Framework 发布
* .但 NET Framework 是 Windows 的基础，往往不会随意更新。

所以要想获取最新的 Web Framework 是非常麻烦的，幸运的事，微软已经意识到了问题的严重性，最新的 Web Framework 都是通过 Nuget 来获取。

当然这是一部分原因，还有一层原因是 ASP.NET &amp; IIS 实在太过于笨重，如何讲呢？

复杂的生命周期已成为累赘？简单来说，当请求到达服务器时，Windows 内核组件 HTTP.SYS 组件捕获请求，他会分析请求并决定是否交给 IIS 来处理，当请求到达 IIS 之后，IIS 会根据处理程序映射来匹配请求并交给对应的程序集（实现了 ISAPI 接口，比如我们熟知的 aspnet_isapi.dll 是专门用来处理 ASP.NET Application）处理，最后加载了 CLR 运行环境，将请求交给 aspnet_wp.exe 去处理，这时复杂的 **ASP.NET 生命周期往往令人头大，但事实上有很多时候我们并不需要他。**

如下图所示 ASP.NET Architecture：

![](images/Chapter6/2.png)

打开 IIS，你会发现他提供了非常丰富的功能：缓存、身份验证、压缩、加密等。但随着移动互联网蓬勃的发展，特别是 HTML 5 越来越成熟的今天，我们看到越来越多的操作发生在客户端，而不是沉重的从服务器产生 HTML 返回，**更多的是通过异步 ****AJAX**** 返回原生的数据**。同理，对于 APP 来说我们只需要 Mobile Service 返回数据。显然  **IIS 显得笨重了点，而且 IIS 作为微软产品系的一环，耦合程度太高。所以我们迫切需要轻量、快速、可扩展的宿主来承载 Web Application 和 Web Service。**

### IIS 和 OS 

IIS 必须是安装并运行在 Windows 操作系统中，这是微软产品的一贯风格，环环相套，但不得不考虑他们的限制和局限性：

* IIS 往往和操作系统（Windows Server）绑定在一起，这意味着对于一些新功能如 WebSocket Protocol&nbsp;，我们不得不等待操作系统 Windows Sever 2012、Windows 8 的发布（IIS 8.0）。
* 为了使用 WebSocket 这类新特性，他仅被 **IIS 8.0** 支持，如下所示：

![](images/Chapter6/3.png)

这时你不得不去升级 IIS，但升级操作系统可能会引发旧系统的不稳定性，所以要想平稳的升级 IIS 并不是简单的。

* IIS 作为经典的 Web Server 必须安装在 Windows 系统中，Windows Server 需要授权使用。

正是由于微软产品系紧耦合的关系，才造成跨平台上的不足，这也是被饱受诟病。**所以我们需要****OWIN 来解耦，在面向对象的世界里，接口往往是解耦的关键，如下图所示：**

![](images/Chapter6/4.png)

使用 OWIN，Web Framework 不再依赖 IIS 和 OS，这意味着你能使用任何你想的来替换 IIS(比如：Katana 或者 Nowin)，并且在必要时随时升级，而不是更新操作系统。当然，如果你需要的话，你可以构建自定义的宿主和 Pipeline 去处理 Http 请求。

这一切的改变都是由于OWIN 的出现，他提供了明晰的规范以便我们快速灵活的去扩展 Pipeline 来处理 Http 请求，甚至可以不写任何一句代码来切换不同的 Web Server，前提是这些 Web Server 遵循 OWIN 规范。

## OWIN 的规范

现在我们已经了解了什么是 OWIN 已经为什么需要 OWIN，现在是时候来分析一下 OWIN 的规范了。

### OWIN Layers

实际上，OWIN 的规范非常简单，他定义了一系列的层（Layer），并且他们的顺序是以堆（Stack）的形式定义，如下所示。OWIN 中的接口被称之为应用程序委托或者 AppFunc，用来在这些层之间通信。

![](images/Chapter6/5.png)

OWIN 定义了 4 层：

Host：主要负责应用程序的配置和启动进程，包括**初始化 OWIN Pipeline、运行 Server。**

Server：这是实际的 Http Server，绑定套接字并监听的 HTTP 请求然后将 Request 和 Response 的 Body、Header 封装成符合 OWIN 规范的字典并发送到 OWIN Middleware Pipeline 中，最后Application 为 Response Data 填充合适的字段输出。

Middleware：称之为中间件、组件，位于 Server 与 Application 之间，用来处理发送到 Pipeline 中的请求，这类组件可以是简单的 Logger 或者是复杂的 Web Framework 比如 Web API、SignalR，只要 Sever 连接成功，Middleware 中间件可以是任何实现应用程序委托的组件。

Application：这是具体的应用程序代码，可能在 Web Framework 之上。对于 Web API、SignalR这类 Web Framework 中间件而言，我们仅仅是改变了他们的托管方式，而不是取代 ASP.NET WEB API、SignalR 原先的应用程序开发。所以该怎么开发就怎么开发，只不过我们将他们注册到 OWIN Pipeline 中去处理 HTTP 请求，成为 OWIN 管道的一部分，所以此处的 Application 即正在意义上的处理程序代码。

### Application Delegate

OWIN 规范另一个重要的组成部分是接口的定义，用于 Server 和 Middleware 的交互。**他并不是严格意义上的接口，而是一个委托并且每个** **OWIN 中间件组件必须提供。**

![](images/Chapter6/6.png)

从字面上理解，每个 OWIN 中间件在必须有一个方法接受类型了 IDictionary<string,object>的变量（俗称环境字典），然后必须返回 Task 来异步执行。

### Environment Dictionary

环境字典包含了 Request、Response 所有信息以及 Server State。通过 Pipeline，每个中间件组件和层都可以添加额外的信息，但环境字典定义了一系列强制必须存在的 Key，如下所示：
![](images/Chapter6/key.png)

## 小结
> 这些规范看起来可能简单到微不足道，但 OWIN 的思想就是简单、灵活——通过要求 OWIN 中间件只依赖 AppFun 类型，为开发基于 OWIN 的中间件提供了的最低门槛。同时，通过使用环境字典在各个中间件之间进行信息的传递，而非传统 ASP.NET（System.Web）中使用 HttpContext 贯穿 ASP.NET 整个生命周期来传递。
> 既然 OWIN 是规范，而非真正实现，所以是无法使用在项目中的，若要使用 OWIN，必须要实现他，所以这也是接下来我想聊的，OWIN 的实现：Katana 。
