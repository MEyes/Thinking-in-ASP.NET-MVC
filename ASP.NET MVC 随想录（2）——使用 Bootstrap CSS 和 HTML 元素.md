
# ASP.NET MVC 随想录（2）——使用 Bootstrap CSS 和 HTML 元素

Bootstrap 提供了一套丰富 CSS 设置、HTML 元素以及高级的栅格系统来帮助开发人员快速布局网页。所有的 CSS 样式和 HTML 元素与移动设备优先的流式栅格系统结合，能让开发人员快速轻松的构建直观的界面并且不用担心在较小的设备上响应的具体细节。

## Bootstrap 栅格（Grid）系统

在移动互联网的今天，越来越多的网站被手机设备访问，移动流量在近几年猛增。Bootstrap 提供了一套响应式、移动设备优先的流式栅格系统，随着屏幕或视口（viewport）尺寸的增加，系统会自动分为最多 12 列。

### 栅格参数

Bootstrap 3 提供了一系列的预定义 class 来指定列的尺寸，如下所示：

![][1]

Bootstrap  栅格系统被分割为 12 列，当布局你的网页时，记住所有列的总和应该是 12。为了图示，请看如下 HTML 所示：

    <div class="container">
        <div class="row">
            <div class="col-md-3" style="background-color: green;">
                <h3>green</h3>
            </div>
            <div class="col-md-6" style="background-color: red;">
                <h3>red</h3>
            </div>
            <div class="col-md-3" style="background-color: blue;">
                <h3>blue</h3>
            </div>
        </div>
    </div>

**注：Bootstrap 需要为页面内容和栅格系统包裹一个&nbsp;.container&nbsp; 容器。**

在上述代码中，我添加了一个 class 为 container 的 div 容器，并且包含了一个子的 div 元素 row（行）。row div 元素依次有 3 列。其中 2 列包含了 col-md-3 的 class、一列包含了 col-md-6 的 class。当他们组合在一起时，他们加起来总和是 12.但这段 HTML 代码只作用于显示器分辨率&gt;=992 的设备。所以为了更好的响应低分辨率的设备，我们需要结合不同的 CSS 栅格 class。故添加对平板、手机、低分辨率的 PC 的支持，需要加入如下 class：

    <div class="container">
        <div class="row">
            <div class="col-xs-3 col-sm-3 col-md-3" style="background-color: green;">
                <h3>green</h3>
            </div>
            <div class="col-xs-6 col-sm-6 col-md-6" style="background-color: red;">
                <h3>red</h3>
            </div>
            <div class="col-xs-3 col-sm-3 col-md-3" style="background-color: blue;">
                <h3>blue</h3>
            </div>
        </div>
    </div>

## Bootstrap HTML 元素

Bootstrap 已经为我们准备好了一大堆带有样式的 HTML 元素，如：

* Tables
* Buttons
* Forms
* Images

### Bootstrap Tables（表格）

Bootstrap 为 HTML tables 提供了默认的样式和自定义他们布局和行为的选项。为了更好的演示，我使用精典的 [Northwind][2] 示例数据库以及如下技术：

* 用 ASP.NET MVC 来作为 Web 应用应用程序
* Bootstrap 前端框架
* Entity Framework 来作为 ORM 框架
* StructureMap 执行我们项目的依赖注入和控制反转，使用 Nuget 来安装
* AutoMapper 自动映射 Domain Model 到 View Model，使用 Nuget 来安装

打开 Visual Studio，创建一个 ASP.NET MVC 的项目，默认情况下，VS 已经为我们添加了Bootstrap 的文件。为了查看效果，按照如下的步骤去实施：

在 ASP.NET MVC 项目中的 Models 文件下添加一个 ProductViewModel

       public class ProductViewModel
        {
            public int ProductId { get; set; }
            public string ProductName { get; set; }
            public decimal? UnitPrice { get; set; }
            public int? UnitsInStock { get; set; }
            public bool Discontinued { get; set; }
            public string Status { get; set; }
        }

在 APP_Data 文件夹中添加 AutoMapperConfig 类，通过 AutoMapper，为 ProductViewModel的 Status 属性创建了一个条件映射，如果 Product 是 discontinued，那么 Status为danger；如果 UnitPrice 大于 50，则设置 Status 属性为 info；如果 UnitInStock 小于 20，那么设置 Status 为 warning。代码的逻辑如下:  

    Mapper.CreateMap<Product, ProductViewModel>()
    .ForMember(dest => dest.Status,
    opt => opt.MapFrom
    (src => src.Discontinued
    ? "danger"
    : src.UnitPrice > 50
    ? "info"
    : src.UnitsInStock < 20 ? "warning" : ""));

添加一个 ProductController 并且创建名为 Index 的 Action

     public class ProductController : Controller
        {
            //
            // GET: /Product/
            private readonly ApplicationDbContext _context;

            public ProductController(ApplicationDbContext context)
            {
                this._context = context;
            }
            public ActionResult Index()
            {
                var products = _context.Products.Project().To<productviewmodel>().ToArray();
                return View(products);
            }
        }

上述代码使用依赖注入获取 Entity Framework DbContext 对象，Index Action 接受从数据库中返回 Products 集合然后使用 AutoMapper 映射到每一个 ProductViewModel 对象中，最后为 View 返回数据。  
最后，在视图上使用 Bootstrap HTML table 来显示数据

    <div class="container">
    <h3>Products</h3>
    <table class="table">
        <thead>
            <tr>
                <th>
                    Product Name
                </th>
                <th>
                    Unit Price
                </th>
                <th>
                    Units In Stock
                </th>
                <th>
                    Discontinued
                </th>
            </tr>
        </thead>
        <tbody>
            @foreach (var item in Model)
            {
                <tr>
                    <td>
                        @Html.DisplayFor(modelItem => item.ProductName)
                    </td>
                    <td>
                        @Html.DisplayFor(modelItem => item.UnitPrice)
                    </td>
                    <td>
                        @Html.DisplayFor(modelItem => item.UnitsInStock)
                    </td>
                    <td>
                        @Html.DisplayFor(modelItem => item.Discontinued)
                    </td>
                </tr>
            }
        </tbody>
    </table>
    </div>
呈现的数据如下所示:

![][5]

#### Bootstrap Tables 其余样式

Bootstrap 提供了额外的样式来修饰 table。比如使用 **table-bordered** 来显示边框， **table-striped** 显示奇偶数行间颜色不同（斑马条纹状），**table-hover** 顾名思义，当鼠标移动行时高亮，通过添加&nbsp;.**table-condensed**&nbsp;类可以让表格更加紧凑，单元格中的内补（padding）均会减半，修改后的代码如下所示：

    <table class="table table-bordered table-striped table-hover">
    </table>

显示的效果如下：

![][6]
 
### Bootstrap上下文 Table 样式

Bootstrap 提供了额外的 class 能让我们修饰和的样式，提供的 class 如下：

* Active
* Success
* Info
* Warning
* Danger

修改上述代码，为动态添加样式：  

    @foreach (var item in Model)
    {
    <tr class="@item.Status">
        <td>
            @Html.DisplayFor(modelItem => item.ProductName)
        </td>
        <td>
            @Html.DisplayFor(modelItem => item.UnitPrice)
        </td>
        <td>
            @Html.DisplayFor(modelItem => item.UnitsInStock)
        </td>
        <td>
            @Html.DisplayFor(modelItem => item.Discontinued)
        </td>
    </tr>
    }

更新过后的效果如下所示：

![][7]

### Bootstrap Buttons

Bootstrap 提供了许多各种不同颜色和大小的 buttons，为核心的 buttons 提供 6 种颜色和 4 种尺寸可以选择，同样通过设置 class 属性来显示不同的风格：

• btn btn-primary btn-xs

• btn btn-default btn-sm

• btn btn-default

• btn btn-sucess btn-lg

可以为 Button 设置颜色的 class：

• btn-default

• btn-primary

• btn-success

• btn-info

• btn-warning

• btn-danger

所以可以使用如下代码来呈现效果：

    <div class="row">
        <!-- default按钮 -->
        <button type="button" class="btn btn-default btn-xs">
            Default &amp; Size=Mini
        </button>
        <button type="button" class="btn btn-default btn-sm">
            Default &amp; Size=Small
        </button>
        <button type="button" class="btn btn-default">Default</button>
        <button type="button" class="btn btn-default btn-lg">
            Default &amp; Size=Large
        </button>
    </div>

显示效果如下：

![][8]

### Bootstrap Form（表单）

表单常见于大多数业务应用程序里，因此统一的样式有助于提高用户体验，Bootstrap 提供了许多不同的 CSS 样式来美化表单。
 
使用 ASP.NET MVC 的 HTML.BeginForm 可以方便的创建一个表单，通过为<form>添加名为 **form-horizontal** 的 class 来创建一个 Bootstrap 水平显示表单。


    @using (Html.BeginForm("Login", "Account", FormMethod.Post, new { @class = "form-horizontal", role = "form" }))
    {
        <div class="form-group">
            @Html.LabelFor(m =>m.UserName, new { @class = "col-md-2 control-label" })
            <div class="col-md-10">
                @Html.TextBoxFor(m => m.UserName, new { @class = "form-control" })
                @Html.ValidationMessageFor(m => m.UserName)
            </div>
        </div>
        <div class="form-group">
            @Html.LabelFor(m => m.Password, new { @class = "col-md-2 control-label" })
            <div class="col-md-10">
                @Html.PasswordFor(m => m.Password, new { @class = "form-control" })
                @Html.ValidationMessageFor(m => m.Password)
            </div>
        </div>
        <div class="form-group">
            <div class="col-md-offset-2 col-md-10">
                <input type="submit" value="Log in" class="btn btn-default">
            </div>
        </div>
    }


上述代码中，使用 class 为 form-group 的<div>元素包裹了 2 个 Html 方法（Html.LabelFor、Html.TextboxFor），这能让 Bootstrap 验证样式应用在 form 元素上，当然你也可以使用 Bootstrap 栅格col- class 来指定 form 中元素的宽度，效果如下显示：

![][9]

Bootstrap 基础表单默认情况下是垂直显示内容，在 Html.BeginForm 帮助方法里移除 class 为 **form-horizontal** 和 class **col-** 后，显示的效果如下：

![][10]

内联表单表示所有的 form 元素一个接着一个水平排列， 只适用于视口（viewport）至少在 768px  宽度时（视口宽度再小的话就会使表单折叠）。 

**_记得_一定要添加 **&nbsp;label&nbsp;** 标签，如果你没有为每个输入控件设置&nbsp;label&nbsp; 标签，屏幕阅读器将无法正确识别。对于这些内联表单，你可以通过为label&nbsp;设置 &nbsp;.sr-only&nbsp;**** 类将其隐藏。详细代码如下：


    @using (Html.BeginForm("Login", "Account", FormMethod.Post, new { @class = "form-inline", role = "form" }))
    {
        <div class="form-group">
            @Html.LabelFor(m => m.UserName, new { @class = "sr-only" })
            @Html.TextBoxFor(m => m.UserName, new { @class = "form-control", placeholder = "Enter your username" })
            @Html.ValidationMessageFor(m => m.UserName)
        </div>
        <div class="form-group">
            @Html.LabelFor(m => m.Password, new { @class = "sr-only" })
            @Html.PasswordFor(m => m.Password, new { @class = "form-control", placeholder = "Enter your username" })
            @Html.ValidationMessageFor(m => m.Password)
        </div>
        <div class="form-group">
            <input type="submit" value="Log in" class="btn btn-default">
        </div>
    }

显示效果如下：

![][11]

### Bootstrap Image

在 Bootstrap 3.0 中，通过为图片添加 &nbsp;.img-responsive&nbsp;类可以让图片支持响应式布局。其实质是为图片设置了&nbsp;max-width: 100%;、&nbsp;height: auto;&nbsp;和&nbsp;display: block;&nbsp;属性，从而让图片在其父元素中更好的缩放。

通过为&nbsp;<img>&nbsp;元素添加以下相应的类，可以让图片呈现不同的形状。

* img-rounded
* img-circle
* img-thumbnail

请看如下代码：

    <div class="row">
        <h3>Our Team</h3>
        @foreach (var item in Model)
        {
            <div class="col-md-4">
                <img src="@Url.Content(" ~="" images="" employees="" "="" +="" item.employeeid="" ".png")"="" alt="@item.FirstName@item.LastName" class="img-circle img-responsive">
                <h3>
                    @item.FirstName @item.LastName <small>@item.Title</small>
                </h3>
                <p>@item.Notes</p>
            </div>
        }
    </div>

![][12]

## Bootstrap 验证样式

默认情况下 ASP.NET MVC 项目模板支持 unobtrusive 验证并且会自动添加需要的 JavaScript 库到项目里。然而默认的验证不使用 Bootstrap 指定的 CSS。当一个 input 元素验证失败，JQuery validation 插件会为元素添加 input-validation-error class（存在 Site.css 中）。那怎样不修改 JQuery Validation 插件而且使用 Bootstrap 内置的错误样式呢？

Bootstrap 提供了 class为has-error 中的样式（label 字体变为红色，input 元素加上红色边框）来显示错误:

    <div class="form-group has-error">
        @Html.LabelFor(m => m.UserName, new { @class = "col-md-2 control-label" })
        <div class="col-md-10">
            @Html.TextBoxFor(m => m.UserName, new { @class = "form-control" })
        </div>
    </div>

所以，我需要动态来为<div class=" from-group">元素动态绑定/移除 has-error。为了不修改 JQuery.validation 插件，我在 Scripts 文件夹中添加 jquery.validate.bootstrap 文件:

    $.validator.setDefaults({
        highlight: function (element) {
            $(element).closest('.form-group').addClass('has-error');
        },
        unhighlight: function (element) {
            $(element).closest('.form-group').removeClass('has-error');
        },
    });

这段脚本的通过调用 setDefaults 方法来修改默认的 JQuery validation 插件设置。看以看到我使用 highlight 和 unhighlight 方法来动态添加/移除 has-error class。

最后将它添加到打包文件中

    bundles.Add(new ScriptBundle("~/bundles/jqueryval").Include(
    "~/Scripts/jquery.validate.js",
    "~/Scripts/jquery.validate.unobtrusive.js",
    "~/Scripts/jquery.validate.bootstrap.js"));

 注：默认情况下，ASP.NET MVC **使用通配符***来将 **jquery.validate*** 文件打包到 jqueryval 文件中，如下所示：

     bundles.Add(new ScriptBundle("~/bundles/jqueryval").Include(
    
     "~/Scripts/jquery.validate*"));

 但是 jquery.validate.bootstrap.js 必须在 jquery validate 插件后加载，所以我们只能显式的指定文件顺序来打包，**因为默认情况下打包加载文件的顺序是按通配符代表的字母顺序排列的。**

## ASP.NET MVC 创建包含 Bootstrap 样式编辑模板

**编辑模板（Editor Template）指的是在 ASP.NET MVC 应用程序中，基于对象属性的数据类型通过 Razor 视图渲染后，自动产生表单 Input 元素。ASP.NET MVC 包含了若干的编辑模板，当然我们也可以实现扩展。编辑模板类似于局部视图，不同的是，局部视图通过 name 来渲染，而编辑模板通过类型来渲染。**

举个例子，@Html.EditorFor(model => model.Property)，如果 Property 类型为 string，那么@Html.Editor 会创建一个 Type=Text 的 Input 元素；如果 Property 类型为 Password，那么会创建一个 Type=Password 的 Input 元素。所以 EditorFor helper 是基于 model 属性的数据类型来渲染生成 HTML。

不过美中不足的是，默认产生的 HTML 如下所示：

![][13]

**可以看到 class="text-box single-line"，但先前提到过，Bootstrap Form 元素 class 必须是 form-control。**

所以，为了让 Editor helper 生成 class为form-control 的表单元素，我们需要创建一个自定义的编辑模板来重写旧的模板。你需要如下操作：

* 在 Shared 文件夹中创建名为 EditorTemplates（**注意要一样的名称**）的文件夹
* 添加名为 string.cshtml（**注意要一样的名称**）文件，并添加如下代码：

    @model string
    @Html.TextBox("", ViewData.TemplateInfo.FormattedModelValue, new
    {
        @class = "form-control"
    })

在上述代码中，我们调用 @Html.TextBox 方法，并且传递了一个**空的字符串**作为 textbox 的name。这将会让 model 的属性名作为生成的 textbox 的 name，并且 textbox 显示的内容是 model 的值，最后追加了名为 class的attribute，而且其值为"form-control"。

重新生成项目，发现新生成的 input 元素它的 class 已经改为"form-control"了。如下所示：

&nbsp;![][14]

ASP.NET MVC 能让开发者创建**_根据自定义_****_DataType_**的编辑模板，比如自动生成多行文本框并且规定行数为3行，也是同样的操作：

* 添加 MultilineText. Cshtml（注意名称相同）文件到 EditorTemplates 中
* 添加如下代码：

    @model string
    @Html.TextArea("", ViewData.TemplateInfo.FormattedModelValue.
    ToString(), new { @class = "form-control", rows = 3 })

* 为了让我们的 Model 的属性在渲染时采用 MultilineText.cshtml 编辑模板，我们需要为属性**指定** DataType attribute 为 MultilineText:

     [DataType(DataType.MultilineText)]
      public string Description { get; set; }

最终显示如下所示：
![][15]

## 小结

>  这篇文章为大家介绍了 Bootstrap 的响应式布局（grid），并且简单介绍了 Bootstrap 中的HTML 元素，包括 Table、Button、Form、Image…然后修改了 JQuery validate 插件默认的的设置，使其友好支持 Bootstrap 中的错误提示样式。最后探索了 ASP.NET MVC 中的编辑模板，能让产生的 input 元素自动包含 form-control 样式，谢谢大家支持。

[1]: images/Chapter2/1.png
[2]: https://msdn.microsoft.com/zh-cn/library/ms143221.aspx
[3]: http://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif
[4]: http://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif
[5]: images/Chapter2/5.png
[6]: images/Chapter2/6.png
[7]: images/Chapter2/7.png
[8]: images/Chapter2/8.png
[9]: images/Chapter2/9.png
[10]: images/Chapter2/10.png
[11]: images/Chapter2/11.png
[12]: images/Chapter2/12.png
[13]: images/Chapter2/13.png
[14]: images/Chapter2/14.png
[15]: images/Chapter2/15.png
