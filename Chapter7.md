
# ASP.NET MVC 随想录（7）——锋利的 KATANA

正如上篇文章所讲解的，OWIN 在 Web Server 与 Web Application 之间定义了一套规范（Specs），意在解耦 Web Server 与 Web Application，从而推进跨平台的实现。若要真正使用 OWIN 规范，那么必须要对他们进行实现。目前有两个产品实现了 OWIN 规范——一是由微软主导的 Katana，二是第三方的 Nowin。本文主要关注的还是 Katana，由微软团队主导，开源到 CodePlex 上。

  
> 在介绍 Katana 之前，我觉得有必要先为大家梳理一下十几年以来 ASP.NET 发展历程。

## ASP.NET 发展历程

### ASP.NET Web Form

ASP.NET Web Form 在 2002 正式发布时，面向的开发者主要有两类：

* 使用混合 HTML 标记和服务端脚本开发动态网站的 ASP 开发者，另外，ASP 运行时抽象了底层的 HTTP 连接和 Web Server，并为开发者提供了一系列的对象模型用于交互 Http 请求，当然也提供了额外的服务诸如 Session、Cache、State 等。
* 开发 WinForm 的程序员，他们可能对 HTTP 和 HTML 一无所知，但熟悉拖控件的方式来构建应用程序。

为了迎合这两类开发者，ASP.NET Web Form 通过使用沉重的 ViewState 来保存页面回传过程中的状态值，因为 HTTP 协议是无状态的，通过 ViewState，使原本没有记忆的 Http 协议变得有记忆起来。这在当时是非常好的设计，能通过拖拽控件的形式快速开发 Web，而不必过多的去关注底层原理。同时 ASP.NET 团队还为 ASP.NET 丰富了更多的功能，诸如：Session、Cache、Configuration 等等。

这在当时无疑是成功的，ASP.NET 的发布迅速拉拢了开发者，在 Web 开发中形成了一股新的势力，但同时也买下来一些隐患：

* 所有的功能、特性都发布在一个整体框架上并且紧耦合核心的 Web 抽象库——System.Web
* System.Web 是.NET Framework 的重要组成部分，这意味着要修复更新 System.Web 必须更新.NET Framework，但.NET Framework 是操作系统的基础，为了稳定性往往不会频繁更新。
* ASP.NET Framework （System.Web）紧耦合 IIS
* IIS 只能运行在 Windows系统

### ASP.NET MVC

由于 Web Form 产生一大堆 ViewState 和客户端脚本，这对开发者来说慢慢变成一种累赘，因为我们只想产生纯净的 HTML 标记。所以开发者更想去主动控制而非被动产生额外 HTML 标记。

所以微软基于 MVC 设计模式推出了其重要的 Web Framework——ASP.NET MVC Framework，通过 Model-View-Control 解耦了业务逻辑和表现逻辑，同时没有了服务器端控件，将页面的控制权完全交给了开发者。

为了快速更新迭代，通过 Nuget 来获取更新，故从.NET Framework 中分离开了。但唯一不足的是，ASP.NET MVC 还是基于 ASP.NET Framework（注:ASP.NET MVC 6 已经不依赖 System.Web），所以 Web Application 和 Web Server 依旧没有解耦。

### ASP.NET Web API

随着时间的推移，一些问题开始暴露出来了，由于 Web Server 和 Web Application 紧耦合在一起，微软在开发独立、简单的 Framework 上越发捉襟见肘，这和其他平台下开源社区蓬勃发展形成鲜明对比，幸运的是微软做出了改变，推出了独立的 Web Framework ——ASP.NET Web API，它适用于移动互联网并可以快速通过 Nuget 安装，更为重要的是，它不依赖 System.Web，也不依赖 IIS，你可以使用 Self-Host 或者在其他 Web Server 部署。

### Katana

随着 Web API 能够运行在自己的轻量级的宿主中，并且越来越多简单、模块化、专一的 Framework 问世，开发人员有时候不得不启动单独的进程来处理 Web 应用程序的各种组件（模块）、如静态文件、动态文件、Web API 和 Socket。为了避免进程扩散，所有的进程必须启动、停止并且独立进行管理。**这时，我们需要一个公共的宿主进程来管理这些模块。**

> 这就是 OWIN 诞生的原因，**解耦成最小粒度的组件**，然后这些标准化框架和组件可以很容易地插入到 OWIN Pipeline 中，从而对组件进行统一管理。而 Katana 正是 OWIN 的实现，为我们提供了丰富的 Host 和 Server。

## 走进Katana的世界

Katana 作为 OWIN 的规范实现，除了实现 Host 和 Server 之外，还提供了一系列的 API 帮助开发应用程序，其中已经包括一些功能组件如身份验证（Authentication）、诊断（Diagnostics）、静态文件处理（Static Files）、ASP.NET Web API 和 SignalR 的绑定等。

### Katana的基本原则

* 可移植性：从 HostàServeràMiddleware，每个 Pipeline 中的组件都是可替换的，并且第三方公司和开源项目的 Framework 都是可以在 OWIN Server 上运行，也就是说不受平台限制，从而实现跨平台。
* 模块化：每一个组件都必须保持足够独立性，通常只做一件事，以混合模块的形式来满足实际的开发需求
* 轻量和高效：因为每一个组件都是模块化开发，而且可以轻松的在 Pipeline 中插拔组件，实现高效开发

### Katana 体系结构
 
Katana 实现了 OWIN 的 Layers，所以 Katana 的体系结构和 OWIN 一致，如下所示：

![](images/Chapter7/2.png)

1.）Host ：宿主 Host 被 OWIN 规范定义在第一层（最底层），他的职责是管理底层的进程（启动、关闭）、初始化 OWIN Pipeline、选择 Server 运行等。

Katana 为我们提供了 3 种选择：

* IIS / ASP.NET ：使用 IIS 是最简单和向后兼容方式，在这种场景中 OWIN Pipeline 通过标准的 HttpModule 和 HttpHandler 启动。使用此 Host 你必须使用 System.Web 作为 OWIN Server
* Custom Host ：如果你想要使用其他 Server 来替换掉 System.Web，并且可以有更多的控制权，那么你可以选择创建一个自定义宿主，如使用 Windows Service、控制台应用程序、Winform 来承载Server。
* OwinHost ：如果你对上面两种 Host 还不满意，那么最后一个选择是使用 Katana 提供的OwinHost.exe：他是一个命令行应用程序，运行在项目的根部，启动 HttpListener Server并找到基于约束的 Startup 启动项。OwinHost 提供了命令行选项来自定义他的行为，比如：手动指定 Startup 启动项或者使用其他 Server（如果你不需要默认的 HttpListener Server）。

2.）Server

Host 之后的 Layer 被称为 Server，他负责打开套接字并监听 Http 请求，一旦请求到达，根据Http 请求来构建符合 OWIN 规范的 Environment Dictionary(环境字典)并将它发送到 Pipeline 中交由 Middleware 处理。Katana 对 OWIN Server 的实现分为如下几类：

* System.Web：如前所述那样，System.Web 和 IIS/ASP.NET Host 两者彼此耦合，当你选择使用System.Web 作为 Server ，Katana System.Web Server 把自己注册为 HttpModule 和HttpHandler 并且处理发送给 IIS 的请求，最后将 HttpRequest、HttpResponse 对象映射为 OWIN 环境字典并将它发送至 Pipeline 中处理。
* HttpListener：这是 OwinHost.exe 和自定义 Host 默认的 Server。
* WebListener：这是 ASP.NET vNext 默认的轻量级 Server，他目前无法使用在 Katana 中

3）Middleware

Middleware（中间件）位于 Host、Server 之后，用来处理 Pipeline 中的请求，Middleware 可以理解为实现了 OWIN 应用程序委托 AppFun 的组件。

Middleware 处理请求之后并可以交由下一个 Pipeline 中的 Middleware 组件处理，即链式处理请求，通过环境字典可以获取到所有的 Http 请求数据和自定义数据。Middleware 可以是简单的 Log 组件，亦可以为复杂的大型 Web Framework，诸如：ASP.NET Web API、Nancy、SignlR 等，如下图所示：Pipeline 中的 Middleware 用来处理请求：

![](images/Chapter7/3.png)

4.）Application

最后一层即为 Application，是具体的代码实现，比如 ASP.NET Web API、SignalR 具体代码的实现。

现在，我想你应该了解了什么事 Katana 以及 Katana 的基本原则和体系结构，那么现在就是具体应用到实际当中去了。

### 使用 ASP.NET/IIS 托管 Katana-based 应用程序

在 Startup 的 Configuration 方法中实现 OWIN Pipeline 处理逻辑，如下代码所示：

    public class Startup
    {
	    public void Configuration(IAppBuilder app)
	    
	    {
		    app.Run(context =>
		    
		    {
		    
			    context.Response.ContentType = "text/plain";
			    
			    return context.Response.WriteAsync("Hello World");
		    
		    });
	    
	    }
    
    }

app.Run 方法将一个接受 IOwinContext 对象最为输入参数并返回 Task 的 Lambda 表达式作为 OWIN Pipeline 的最后处理步骤，IOwinContext 强类型对象是对 Environment Dictionary 的封装，然后异步输出"Hello World"字符串。

细心的你可能观察到，在 Nuget 安装 Microsoft.Owin.Host.SystemWeb 程序集时，默认安装了依赖项 Microsoft.Owin 程序集，正式它为我们提供了扩展方法 Run 和 IOwinContext 接口，当然我们也可以使用最原始的方式来输出"Hello World"字符串，即 Owin 程序集为我们提供的最原始方式，这仅仅是学习上参考，虽然我们不会在正式场景下使用：

    using AppFunc = Func<IDictionary<string, object>, Task>;
    
    public class Startup
    
    {
    
	    public void Configuration(IAppBuilder app)
	    
	    {
	    
		    app.Use(new Func<AppFunc, AppFunc>(next => (env =>
		    
		    {
		    
			    string text = "Hello World";
			    
			    var response = env["owin.ResponseBody"] as Stream;
			    
			    var headers = env["owin.ResponseHeaders"] as IDictionary<string, string[]>;
			    
			    headers["Content-Type"] = new[] { "text/plain" };
			    
			    return response.WriteAsync(Encoding.UTF8.GetBytes(text), 0, text.Length);
		    
		    })));
	    
	    }
    
    }

### 使用自定义 Host(self-host)托管 Katana-based 应用程序

使用自定义 Host 托管 Katana 应用程序与使用 IIS 托管差别不大，你可以使用控制台、WinForm、WPF 等实现托管，但要记住，这会失去 IIS 带有的一些功能（SSL、Event Log、Diagnostics、Management…），当然这可以自己来实现。

* 创建控制台应用程序*
* Install-Package Microsoft.Owin.SelfHost
* 在 Main 方法中使用 Startup 配置项构建 Pipeline 并监听端口  

    	using (WebApp.Start("http://localhost:10002"))  
    	{
    	
    	   System.Console.WriteLine("启动站点：http://localhost:10002");
    	
    	   System.Console.ReadLine();
    	
    	}


使用自定义的 Host 将失去 IIS 的一些功能，当然我们可以自己去实现。幸运的是，Katana 为我们默认实现了部分功能，比如 Diagnostic，包含在程序集 Microsoft.Owin.Diagnostic 中。

    public void Configuration(IAppBuilder app)
    {
    
	    app.UseWelcomePage("/");
	    
	    app.UseErrorPage();
	    
	    app.Run(context =>
    	{
    
		    //将请求记录在控制台
		    
		    Trace.WriteLine(context.Request.Uri);
		    
		    //显示错误页
		    
		    if (context.Request.Path.ToString().Equals("/error"))
		    
		    {
		    
			    throw new Exception("抛出异常");
			    
		    }
		    
		    context.Response.ContentType = "text/plain";
		    
		    return context.Response.WriteAsync("Hello World");
	    
	    });
    
    }

在上述代码中，当请求的路径（Request.Path）为根目录时，渲染输出 Webcome Page**并且不继续执行 Pipeline 中的其余 Middleware 组件**，如下所示：

![](images/Chapter7/4.png)

如果请求的路径为 Error时，抛出异常，显示错误页，如下所示：

![](images/Chapter7/5.png)

### 使用 OwinHost.exe 托管 Katana-based 应用程序

当然我们还可以使用 Katana 提供的 OwinHost.exe 来托管应用程序，毫无疑问，通过 Nuget 来安装 OwinHost。

如果你按照我的例子一步一步执行的话，你会发现不管使用 ASP.NET/IIS 托管还是自托管，Startup  配置类都是不变的，改变的仅仅是托管方式。同理 OwinHost 也是一样的，但它更灵活，我们可以使用类库或者 Web 应用程序来作为 Application。

类库作为 Application，可以最小的去引用程序集，创建一个类库后，删除默认的 Class1.cs，然后并且添加 Startup 启动项，这会默认像类库中添加 Owin 和 Microsoft.Owin 程序集的引用。

然后，使用 Nuget 来安装 OwinHost.exe，如 Install-Package OwinHost，注意它并不是一个程序集，而是.exe 应用程序位于<solution root="">/packages/OwinHost.(version)/tools 文件夹。

因为类库不能直接运行，那么只能在它的根目录调用 OwinHost.exe 来托管，它将加载.bin 文件下所有的程序集，所以需要改变类库的默认输出，如下所示：

![](images/Chapter7/6.png)

然后编译解决方案，打开 cmd，键入如下命令：

![](images/Chapter7/7.png)

如上图成功启动了宿主 Host 并且默认监听 5000 端口。

OwinHost.exe 还提供自定义参数，通过追加-h 来查看，如下所示：

![](images/Chapter7/8.png)

既然类库不能直接运行，当然你也不能直接进行调试，我们可以附加 OwinHost 进程来进行调试，如下所示：

![](images/Chapter7/9.png)

注：我在使用 **OwinHost.exe 3.0.1** 时，Startup 如果是如下情况下，它提示转换失败，不知是否是该版本的 Bug。

       using AppFunc = Func<idictionary<string, object="">, Task>;
       public class Startup  
       {  
	       public void Configuration(IAppBuilder app)
	       {
	    
	       //使用OwinHost.exe，报错，提示转换失败
	    
	       app.Run(context=>context.Response.WriteAsync("Hello World"));
	    
	       //使用OwinHost.exe 不报错
	    
	       //app.Use(new Func<appfunc, appfunc="">(next => (env =>
	    
	       //{
	    
		       // string text = "Hello World";
		    
		       // var response = env["owin.ResponseBody"] as Stream;
		    
		       // var headers = env["owin.ResponseHeaders"] as IDictionary<string, string[]="">;
		    
		       // headers["Content-Type"] = new[] { "text/plain" };
		    
		       // return response.WriteAsync(Encoding.UTF8.GetBytes(text), 0, text.Length);
	    
	       //})));
	    
	       }
    
       }
报错信息如下：

![](images/Chapter7/10.png)

Web Application 比类库使用起来轻松多了，你可以直接运行和调试，唯一比较弱的可能是它引用较多的程序集，你完全可以删掉，比如 System.Web。

通过 Nuget 安装了 OwinHost.exe 之后，可以在 Web 中使用它，如下所示：

![](images/Chapter7/11.png)

### 几种指定启动项 Startup 的方法

* 默认名称约束：默认情况下 Host 会去查找 root namespace 下的名为 Startup 的类作为启动项。
* OwinStartup Attribute：当创建 Owin Startup 类时，自动会加上 Attribute 如：[****assembly**: **OwinStartup**(**typeof**(**JKXY.KatanaDemo.OwinHost.Startup**))]**
* 配置文件，如： <add key="owin:appStartup" value="JKXY.KatanaDemo.Web.StartupDefault">
* 如果使用自定义 Host，那么可以通过 WebApp.Start<startup>**(**"http://localhost:10002") 来设置启动项。
* 如果使用 OwinHost，那么可以通过命令行参数来实现，如下截图所示

![](images/Chapter7/12.png)

### 启动项 Startup 的高级应用

启动项 Startup 支持 Friendly Name，通过 Friendly Name 我们可以动态切换 Startup。

比如在部署时，我们会有 UAT 环境、Production 环境，在不同的环境中我们可以动态切换 Startup 来执行不同的操作。

举个例子，我创建来两个带有 Friendly Name 的 Startup，如下所示：

      [assembly: OwinStartup("Production", typeof(JKXY.KatanaDemo.Web.StartupProduction))]
      namespace JKXY.KatanaDemo.Web
      {
          using AppFunc = Func<idictionary<string, object="">, Task&gt;;
          public class StartupProduction
          {
              public void Configuration(IAppBuilder app)
              {
                 app.Run(context=&gt;context.Response.WriteAsync("Production"));
             }
         }
     }

      [assembly: OwinStartup("UAT",typeof(JKXY.KatanaDemo.Web.StartupUAT))]
     
      namespace JKXY.KatanaDemo.Web
      {
          public class StartupUAT
          {
              public void Configuration(IAppBuilder app)
             {
                 app.Run(context=&gt;context.Response.WriteAsync("UAT"));
             }
         }
     }

****根据 Friendly Name 使用配置文件或者 OwinHost 参数来切换 Startup****

      <appsettings>
        <add key="owin:appStartup" value="Production">
      </add></appsettings>

### 小结

> 这篇博客为大家讲解了 Katana 的世界，那么接下来我将继续 OWIN &amp; Katana 之旅，探索 Middleware 的创建，谢谢大家支持。


