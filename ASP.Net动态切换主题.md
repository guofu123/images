# ASP.Net动态切换主题

>参考课堂讲解改编，大部分借用了老师的原话

## Step1:编写一个主页面
可以参考下面的布局设计，在这里的附件中提供了一个示例主界面，在MasterPage.Master中。
![主界面布局](https://img-blog.csdnimg.cn/20191007110520520.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzIwOTc5NQ==,size_16,color_FFFFFF,t_70)
## Step 2：编写主页面的布局
在aspx文件里面编写自己的布局，并使用class来添加样式。因为使用主题，暂时不用引入CSS文件。参考文件TestMaster.aspx。
## Step 3：编写主题
1. 每一个主题对应一个文件夹，主题的名字就是文件夹名字，主题文件夹里放置 CSS 文件和其它资源文件。建议建立子文件夹，分类存放图片、声音、视频等文件。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191007111750503.png)
2. 每一个主题都有一个对应的 CSS 文件，该文件应当与主题名称一致，并且放到主题文件夹下面（别放到子文件夹里）。
3. 在主题 CSS 文件里给主页面中各个 class 添加样式。不同主题所面对的 class 是一样的，只不过每一个 class 的样式不一样。

注：参考附件的主题编写，自己可以自行编写，设计不同的主题。
##  Step 4：添加动态切换主题的控件
 1. 在主页面的合适位置，添加一个 DropDownList，用它来提供各个主题的选择项。并且编写它的事件处理方法。
2. 请再移动到 selectTheme_SelectedIndexChanged 这里，阅读该方法的实现。
帮助：鼠标右键点击下面的 selectTheme_SelectedIndexChanged，选择 查看代码，即可跳转。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191007112051191.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzIwOTc5NQ==,size_16,color_FFFFFF,t_70)

## Step 5：编写 DropDownList 的事件响应方法
根据 ASP.NET 页面生命周期，当用户改变 DropDownList 切换主题的时候，该方法被调用，然而此时已经错过了能够设置主题的时机：PreInit。所以，这里只能将用户选择的主题先保存起来，再让页面重新加载一次。在下一次加载的时候，读取保存起来的主题名称，再切换主题。在这里，我们选择使用 Cookie 来保存用户的选择。
参考代码：（这里包含注释，附件的代码注释少）
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191007112337969.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzIwOTc5NQ==,size_16,color_FFFFFF,t_70)
## Step 6：创建一个自定义 Page 类
1. 为什么需要这个类：
设置主题 Theme 或者 StyleSheetTheme，都必须在 System.Web.UI.Page 的子类里进行。但是主页面的父类是 System.Web.UI.MasterPage，没法设置。而在每一个我们编写的页面里都去重复写设置主题的代码，显然是重复劳动，跟主页面一次搞定重复工作的思想也不一致。所以，可以借助继承，先在System.Web.UI.Page 下面写一个新的页面类，让它去配合主页面的工作，负责主题的动态切换。之后我们具体的aspx页面，再继承这个类就好了。

 2. 怎么添加这个类？
 如果创建的是 ASP.NET 网站，那么需要先创建一个 APP_Code 文件夹，在这个文件夹里创建类。注意，APP_Code 是 ASP.NET 的特殊用途文件夹，名字固定，更改后失效。只有在这个文件夹里创建类，才能够在其它 cs 文件里引用它。
如果创建的是 Web App 工程，那么可以在工程根目录或者一个子文件夹里创建一个类。但是特别注意，不能在APP_Code 文件夹里创建类，会直接报错。因为 APP_Code 是专门为 ASP.NET 网站预留的，在 Web App 工程里不需要，而且还起反作用。最坑爹的是，VS 居然有提供新建 APP_Code 文件夹的选项，搞的和真的是的。
 
3. 这个类要怎么写：
（1）首先给它添加一个父类，必须是 System.Web.UI.Page；然后给它添加一个 public 无参构造函数。构造函数里什么也不要写。注意，一定不要忘记 public 修饰，否则该构造函数就是 private 的，将没法直接用 new 去实例化它，会报错。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191007113326214.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzIwOTc5NQ==,size_16,color_FFFFFF,t_70)
 （2）之后，根据我们设置主题的方式，添加如下方法：
     1、如果要设置 Theme 属性，让主题后生效，那么要添加 Page_PreInit 方法，响应 PreInit 事件。相关代码 请阅读下面的 Step 7A。
    

### Step 7A：
(a). 这是一个事件处理方法，返回值必须是 void，两个参数的类型分别是 object 和 EventArgs，且次序不能颠倒。这个方法的名字是 Page_PreInit，不能乱改。它是由 Page 的 AutoEventWireup属性决定的。该方法不能是 private 修饰。
( b). 由于 PreInit 事件触发得非常早，我们甚至连控件也看不到。在这里唯一可以获取的就是用户的 http请求数据，而且还是原始数据。由于 Cookies 也是随着 http 数据包一起发送的，所以之前写入到Cookies 的主题名字还是在这里能够找到的。我们要做的就是取出之前存储的主题名字，然后根据这个名字设置主题。注意，Cookies 数据是可以被 篡改的，一定要做充足的检查。
参考代码如下：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191007113938345.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzIwOTc5NQ==,size_16,color_FFFFFF,t_70)
    
 2、如果要设置 StyleSheetTheme 属性，让主题先生效，那么要重写 StyleSheetTheme 的 get 方法。相关代码请阅读下面的 Step 7B。
   （a）StyleSheetTheme 属性比较奇特，你会发现它无法被设置（没有 set 方法）。所以正确的处理方式是重载它的 get 方法。注意，重载的时候，别忘了写上 override 修饰符。重载完这个属性之后，它会被自动调用，而且是在 PreInit 事件触发时调用，我们不需要再写一个 PreInit 事件的处理方法。
     (b)  由于 PreInit 事件触发得非常早，我们甚至连控件也看不到。在这里唯一可以获取的就是用户的 http请求数据，而且还是原始数据。由于 Cookies 也是随着 http 数据包一起发送的，所以之前写入到Cookies 的主题名字还是在这里能够找到的。我们要做的就是取出之前存储的主题名字，然后根据这个名字设置主题。注意，Cookies 数据是可以被篡改的，一定要做充足的检查。
     参考代码：
```csharp
 public override string StyleSheetTheme
    {
        get
        {
            // 这个方法的调用时机是在 PreInit 事件的处理方法里首先判断用户的 Cookies 里有没有 themeName 这一项
            if (this.Request.Cookies["themeName"] != null)
            {
                // 如果 themeName 存在，那么先取出来。注意，Cookies 里面能够找到themeName 这一项，不代表它的 Value 一定存在
                string themeName = this.Request.Cookies["themeName"].Value;
                // 进一步判断取到的 themeName 是不是真的有数据
                if (!string.IsNullOrEmpty(themeName))
                {
                    // 如果字符串有内容，那还要再看看这个主题是不是真的存在，也就是检查一下
                    // 对应的目录是不是存在
                    // 这里又要注意了（光 TM 注意了），"/App_Themes/" 是网站里的虚拟目录
                    // 需要先利用 Server.MapPath 把它转换为服务器操作系统的硬盘路径，
                    // 这样 System.IO 里面的类才能找到对应的文件或者目录。
                    string themePath = this.Server.MapPath("/App_Themes/" + themeName);
                    // 使用下面的方法，可以判断目录是否存在
                    if (System.IO.Directory.Exists(themePath))
                    {
                        // 到此为止，这个主题才算真能用
                        return themeName;
                    }
                }
            }

            // 上面进行了一系列检查，只要有一项不通过，就会走到这里。我们返回默认主题
            // 如果默认主题都不能用，啊.......还是去问问佛祖怎么办吧.......
            return "Blue";
            // 主题设置完毕了，此时尝试更换主题，会发现有效果了，但是 DropDownList 的
            // 状态不对，导致换不回去。那么，该去看 MasterPage.master.cs 了。
        }
    }
```
 
  4. 最后在具体的aspx页面，继承这个类。
  ![在这里插入图片描述](https://img-blog.csdnimg.cn/20191007114919757.png)
## Step 8：更新 DropDownList 的状态
解决这个问题的关键是，判断页面是不是通过 PostBack 提交过来的。如果是首次打开， 那么 DropDownList 的状态只能根据 Cookies 设置，自然不会出错。如果是用户提交过 来的，DropDownList 可能被改变，此时 Cookies 里的数据和提交过来的 DropDownList
 状态会不一致（Cookies 里的数据是过时的）。此时要根据 DropDownList 来设置， 由于它自带 ViewState，可以放手不管。
 在MasterPage.master.cs 的Page_Load函数设置处理处理方法。
```csharp
 sideInfo.Text += selectTheme.SelectedItem.Text + "<br/>";
  if (!IsPostBack)
        {
            # region UpdateSelection
            // 首先判断用户的 Cookies 里有没有 themeName 这一项
            if (this.Request.Cookies["themeName"] != null)
            {
                // 如果 themeName 存在，那么先取出来。注意，Cookies 里面能够找到
                // themeName 这一项，不代表它的 Value 一定存在
                string themeName = this.Request.Cookies["themeName"].Value;
                // 进一步判断取到的 themeName 是不是真的有数据
                if (!string.IsNullOrEmpty(themeName))
                {
                    // 由于 Cookies 数据不可靠，所以要先检查一下用户选的主题，在
                    // 列表里有没有。使用 DropDownList.Items.FindByText （这个
                    // 方法真够隐蔽的，第二次写居然没找到......）
                    ListItem anItem = this.selectTheme.Items.FindByText(themeName);
                    if (anItem != null)
                    {
                        // 如果这个主题真的有，那么可以设置了。注意，先把原来
                        // 选择的选项清除掉，然后再设置。否则报错，说同时选了多个
                        // （这个设计应该是微软偷懒了，按理说，更变选择，可以自动
                        //   清除原来的才对，不就是一个事件嘛）
                        this.selectTheme.SelectedItem.Selected = false;
                        anItem.Selected = true;
                        sideInfo.Text += "Selection changed<br />";
                    }
                }
            }
            # endregion
        }
          sideInfo.Text += selectTheme.SelectedItem.Text + "<br/>";
```
注：到此为止，动态切换主题就完成了。恭喜你走到了最后。最后的总结：如果感觉自己的思路没有错，但是结果却非常不正常，此时请保持冷静，去对照 ASP.NET 页面生命周期，一步一步看看，是不是忽略了一些过程。

## 总结
以上大部分的注释和说明采用老师的注释，不是笔者写，但是感觉老师写的很不错，因此发到网上，作为参考，可以提供学习。步骤比较繁琐，需要一步步的去做，若有疑问，可以留言，欢迎大家指出问题。

参考代码的链接：
笔者编写的代码：https://pan.baidu.com/s/17hVPFrN09odusl4AEU3KTA
提取码：8rlr

 
