
# ASP.NET MVC 使用 Bootstrap 系列（4）——使用 JavaScript 插件  

> Bootstrap 的 JavaScript 插件是以 JQuery 为基础，提供了全新的功能并且还可以扩展现有的 Bootstrap 组件。通过添加 data attribute（data 属性）可以轻松的使用这些插件，当然你也可以使用编程方式的 API 来使用。

> 为了使用 Bootstrap 插件，我们需要添加 Bootstrap.js 或者 Bootstrap.min.js 文件到项目中。这两个文件包含了所有的 Bootstrap 插件，推荐引用 Bootstrap.min.js。当然你也可以包含指定的插件来定制化 Bootstrap.js，已达到更好的加载速度。

## Data 属性 VS 编程 API

Bootstrap 提供了完全通过 HTML 标记的方式来使用插件，这意味着，你可以不写任何 JavaScript 代码，事实上这也是 Bootstrap 推荐的使用方式。

举个例子，若要使 Alert 组件支持关闭功能，你可以通过添加 data-dismiss="alert"属性到按钮(Botton)或者链接（A）元素上，如下代码所示：

    <div class="alert alert-danger">
	    <button data-dismiss="alert" class="close" type="button">x</button>
	    <strong>警告</strong>10秒敌人到达
    </div>

当然，你也可以通过编程方式的 API 来实现同样的功能：

    <div class="alert alert-danger" id="myalert">
	    <button data-dismiss="alert" class="close" type="button" onclick="$('#myalert').alert('close')">x</button>
	    <strong>警告</strong>10秒敌人到达
    </div>

## 下拉菜单（dropdown.js）

使用 dropdown 插件（增强交互性），你可以将下拉菜单添加到绝大多数的 Bootstrap 组件上，比如navbar、tabs 等。**注：将下拉菜单触发器和下拉菜单都包裹在 .dropdown 里，或者另一个声明了 position: relative; 的元素中**。

如下是一个地域的菜单，通过 Razor 引擎动态绑定菜单元素:

    <div class="form-group">
            @Html.LabelFor(model => model.TerritoryId, new { @class = "control-label col-md-2" })
            @Html.HiddenFor(model => model.TerritoryId)
            <div class="col-md-10">
                <div id="territorydropdown" class="dropdown">
                    <button id="territorybutton" class="btn btn-sm btn-info" data-toggle="dropdown">@Model.Territory.TerritoryDescription</button>
                    <ul id="territories" class="dropdown-menu">
                        @foreach (var item in ViewBag.TerritoryId)
                        {
                            <li><a href="#" tabindex="-1" data-value="@item.Value">@item.Text</a></li>
                        }
                    </ul>
                </div>
            </div>
        </div>

注意：通过添加 data 属性：data-toggle="dropdown" 到 Button 或者 Anchor 上，可以切换 dropdown 下拉菜单，增加了交互性。其中菜单的元素设置 tabindex=-1，这样做是为了防止元素成为 tab 次序的一部分。

## 模态框（modal.js）

模态框以弹出框的形式可以为用户提供信息亦或者在此之中完成表单提交功能。Bootstrap 的模态框本质上有 3 部分组成：模态框的头、体和一组放置于底部的按钮。你可以在模态框的 Body 中添加任何标准的 HTML 标记，包括 Text 或者嵌入的 Youtube 视频。

一般来说，务必将模态框的 HTML 代码放在文档的最高层级内（也就是说，尽量作为 body 标签的直接子元素），以避免其他组件影响模态框的展现和/或功能。

如下即为一个标准的模态框，包含 Header、Body、Footer:

![][1]

将下面这段 HTML 标记放在视图的 Top 或者 Bottom：  

    <div class="modal fade" id="deleteConfirmationModal" tabindex="-1" role="dialog" aria-hidden="true">
    <div class="modal-dialog">
        <div class="modal-content">
            <div class="modal-header">
                <button type="button" class="close" data-dismiss="modal" aria-hidden="true">&times;</button>
                <h4 class="modal-title">删除</h4>
            </div>
            <div class="modal-body">
                <p>即将删除 '@Model.CompanyName'. </p>
                <p>
                    <strong>你确定要继续吗</strong>
                </p>
                @using (Html.BeginForm("Delete", "Customers", FormMethod.Post, new { @id = "delete-form", role = "form" }))
                {
                    @Html.HiddenFor(m => m.CustomerId)
                    @Html.AntiForgeryToken()
                }
            </div>
            <div class="modal-footer">
                <button type="button" class="btn btn-default" onclick="$('#delete-form').submit();">是.删除</button>
                <button type="button" class="btn btn-primary" data-dismiss="modal">取消</button>
            </div>
        </div>
    </div>
    </div>
注意：通过在 Button 上添加 data 属性：data-dismiss="modal"可以实现对模态框的关闭，当然你也可以使用编程方式 API 来完成：

    <button type="button" class="btn btn-primary" onclick="$('#deleteConfirmationModal').modal('hide')">取消</button>

为了展示模态框，我们可以不写任何 JavaScript 代码，通过添加 data-toggle="modal"属性到Button 或者 Anchor 元素上即可，同时，为了表示展示哪一个模态框，需要通过 data-target 来指定模态框的 Id。

    <a href="#" data-toggle="modal" data-target="#deleteConfirmationModal">Delete</a>

同样，也可以使用编程方式 API 来打开一个模态框：

    $(document).ready(function () {
    
    	$('#deleteConfirmationModal').modal('show');
    
    });

## 标签页（tab.js）

Tabs 可以使用在大的表单上，通过 Tabs 分割成若干部分显示局部信息，比如在 Northwind 数据库中存在 Customer 顾客信息，它包含了基本信息和地址，可以通过 Tabs 进行 Split，如下所示：

![][2]

要使用 Tabs 也是非常简单的：首先创建标准的无序列表
元素，需要为它的 class 设置为 nav nav-tabs 或者 nav nav-pills。其中
包含的元素即为 Tab 元素，需要设置其 data-toggle为tab 或者 pill 属性以及点击它指向的
内容:  

    <ul id="customertab" class="nav nav-tabs">
       <li class="active"><a href="#info" data-toggle="tab">Customer Info</a></li>
       <li><a href="#address" data-toggle="tab">Address</a></li>
    </ul>
为了设置 Tabs 的内容，需要创建一个父
元素并设置 class为tab-content，在父的 div 容器中为每一个 Tab 内容创建 div 元素，并设置class为tab-pane 和标识的 Id，通过添加 active 来激活哪一个 Tab 内容的显示。  

    <div class="tab-content well">
	    <div class="tab-pane active" id="info">
	    	Customer Info
	    </div>
	    <div class="tab-pane" id="address">
	    	Customer Address
	    </div>
    </div>

当然也可以通过编程方式的 API 来激活：

    $(document).ready(function () {
     $('.nav-tabs a[href="#address"]').tab('show');
    });

## 工具提示（tooltip.js）

Tooltip 能为用户提供额外的信息，Boostrap Tooltip 能被用在各种元素上，比如 Anchor：

    <a data-toggle="tooltip" data-placement="top" data-original-title="这是提示内容" href="#" >工具提示？</a>

你可以添加 data-toggle="tooltip"来使用 tooltip，当然你也可以设置内容的显示位置，通过添加 data-placement 属性来实现，Bootstrap 为我们提供了 4 种位置：  

- top
- bottom
- left
- right
 
最后，通过设置 data-original-title 设置了需要显示的 Title。

注意：为了性能的考虑，Tooltip 的 data-api 是可选的，这意味着你必须手动初始化 tooltip 插件：

    <script type="text/javascript">
	    $(document).ready(function () {
	    	$('[data-toggle="tooltip"]').tooltip();
	    });
    </script>

## 弹出框（popover.js）

弹出框和 Tooltip 类似，都可以为用户提供额外的信息，但弹出框可以展示更多的信息，比如允许我们展示一个 Header 和 Content，如下所示：  

    <a data-content="关于基础建设问题需要有具体的研讨和商量...." data-placement="bottom" title="" data-toggle="popover" href="#" data-original-title="转至百度">城市规划</a>

和 Tooltip 一样，为了性能的考虑，data-api 是可选的，这意味着你必须手动初始化 popover 插件:  

    <script type="text/javascript">
    	$(document).ready(function () {
     		$('[data-toggle="popover"]').popover();
    	});
    </script>

![][3]

## 手风琴组件（collapse.js）

手风琴组件有若干 panel groups 组成，每一个 panel group 依次包含了若干 header 和 content 元素,最常见的使用场景就是 FAQ，如下所示：

![][4]

为了使用手风琴组件，需要对 Panel Heading 中的设置 data-toggle="collapse"和点击它展开的容器(Div)Id，具体如下所示:  

    <div class="row">
        <div class="panel-group" id="accordion">
            <div class="panel panel-default">
                <div class="panel-heading">
                    <h4 class="panel-title">
                        <a data-toggle="collapse" data-parent="#accordion"
                           href="#questionOne">
                            问题 1：什么是 Microsoft MVP 奖励？
                        </a>
                    </h4>
                </div>
                <div id="questionOne" class="panel-collapse collapse in">
                    <div class="panel-body">
                        解答 1：Microsoft 最有价值专家 (MVP) 奖励是一项针对 Microsoft 技术社区杰出成员的年度奖励，根据 Microsoft 技术社区的成员在过去 12 个月内对 Microsoft 相关离线和在线技术社区所作贡献的大小而确定。
                    </div>
                </div>
            </div>
            <div class="panel panel-default">
                <div class="panel-heading">
                    <h4 class="panel-title">
                        <a data-toggle="collapse" data-parent="#accordion"
                           href="#questionTwo">
                            问题 2：MVP 奖励存在的意义何在？
                        </a>
                    </h4>
                </div>
                <div id="questionTwo" class="panel-collapse collapse">
                    <div class="panel-body">
                        解答 2：我们相信，技术社区可以促进自由且客观的知识交流，因此可以构建出一个可靠、独立、真实且可使每个人获益的专业知识来源。Microsoft MVP 奖励旨在表彰那些能积极与其他技术社区成员分享自身知识的全球杰出技术社区领导者。
                    </div>
                </div>
            </div>
        </div>
    </div>

## 旋转木马组件（carousel.js）

Carousel 它本质上是一个幻灯片，循环展示不同的元素,通常展示的是图片，就像旋转木马一样。你可以在许多网站上看到这种组件，要使用它也是非常方便的：

* 将 Carousel 组件被包含在一个 class 为 carousel 以及 data-ride 为"carousel"的
元素中。
* 在上述容器里添加一个有序列表，它将渲染成小圆点代表当前激活的幻灯片（显示在右下角）。
* 紧接着，添加一个 class为carousel-inner 的，这个容器包含了实际的幻灯片
* 然后，添加左右箭头能让用户自由滑动幻灯片
* 最后，设置滑动切换的时间间隔，通过设置 data- interval 来实现。当然也可以通过编程方式 API 来实现：$('#myCarousel').carousel({interval: 10000})

完成了基础之后，将下面HTML标识放在View中即可：  

    <div class="container-full">
    <div id="myCarousel" class="carousel" data-ride="carousel" data-interval="10000">
        <!-- 指示灯 -->
        <ol class="carousel-indicators">
            <li data-target="#myCarousel" data-slide-to="0" class="active"></li>
            <li data-target="#myCarousel" data-slide-to="1"></li>
            <li data-target="#myCarousel" data-slide-to="2"></li>
        </ol>
        <div class="carousel-inner">
            <div class="item active">
                <img src="@Url.Content("~/Images/slide1.jpg")" alt="First slide">
                <div class="container">
                    <div class="carousel-caption">
                        <h1>Unto all said together great in image.</h1>
                        <p>Won't saw light to void brought fifth was brought god abundantly for you waters life seasons he after replenish beast. Creepeth beginning.</p>
                        <p><a class="btn btn-lg btn-primary" href="#" role="button">Read more</a></p>
                    </div>
                </div>
            </div>
            <div class="item">
                <img src="@Url.Content("~/Images/slide2.jpg")" alt="Second slide">
                <div class="container">
                    <div class="carousel-caption">
                        
                        <h1>Fowl, land him sixth moving.</h1>
                        <p>Morning wherein which. Fourth man life saying own creeping. Said sixth given.</p>
                        <p><a class="btn btn-lg btn-primary" href="#" role="button">Learn more</a></p>
                    </div>
                </div>
            </div>
            <div class="item">
                <img src="@Url.Content("~/Images/slide3.jpg")" alt="Third slide">
                <div class="container">
                    <div class="carousel-caption">
                        <h1>Far far away, behind the word mountains.</h1>
                        <p>A small river named Duden flows by their place and supplies it with the necessary.</p>
                        <p><a class="btn btn-lg btn-primary" href="#" role="button">See all</a></p>
                    </div>
                </div>
            </div>
        </div>
        <a class="left carousel-control" href="#myCarousel" data-slide="prev"><span class="glyphicon glyphicon-chevron-left"></span></a>
        <a class="right carousel-control" href="#myCarousel" data-slide="next"><span class="glyphicon glyphicon-chevron-right"></span></a>
    </div>
    </div>
展示的效果如下：

![][7]

## 小结

> 在这篇博客中介绍了常见的 Bootstrap 插件，通过使用数据属性和编程方式的 API 来使用这些插件，更多插件访问： 获取。

[1]: images/Chapter4/1.png
[2]: images/Chapter4/2.png
[3]: images/Chapter4/3.png
[4]: images/Chapter4/4.png
[7]: images/Chapter4/7.png