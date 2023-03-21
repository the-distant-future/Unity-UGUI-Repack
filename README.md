# Unity-UGUI-Repack
## 特点  
1.使用纯代码编程，不使用Unity编辑器。  
2.对GameObject添加了大量扩展方法。调用时优雅。  
3.在一个static partical类中添加了大量静态方法。调用时using此类，然后函数式编程，进一步优雅。  
## 功能
1.矩形：创建矩形、坐标尺寸、设置文本、设置颜色。调用时非常简单，不需要关心Unity的内部结构。  
2.布局：水平布局、垂直布局、网格布局、目录树布局、选项卡布局。布局都允许拖动移动、拖动缩放、拖动排序、存档读档。  
3.功能：右键菜单、鼠标悬浮注释、控制台。  
4.文件：JSON序列化存档读档、错误日志、文件加密。  
5.流程：每帧执行、等到满足X时执行。调用时允许在静态函数中添加Update事件。  
## 准备工作
    using CMKZ;
    using static CMKZ.LocalStorage;
## 矩形
### 基本用法
    GameObject A = MainPanel.创建矩形("0 0 100 100").SetColor(128,128,128).SetText("测试");
现在，忘掉Unity的Canvas结构。想象这样的设定：  
Unity中自带一个名为MainPanel的透明矩形，全屏。  
每一个GameObject拥有【文本】【颜色】【滚动条】的属性。在设置颜色、添加文本、添加滚动条时，不需要创建额外的GameObject。  
在使用本库时，就可以视为Unity是这样的设定。  
注：  
MainPanel会在初次调用时自动创建Canvas。  
SetColor三个参数是RGB值，范围为0到255。第四参数表示透明度，默认为1（不透明），0.5f表示半透明。  
SetText会在文本过多时自动添加滚动条。且，嵌套滚动条不会有bug，会自动【内部滚动条到底后，外部滚动条滚动】。  
SetColor与SetText都是return this。即，A是【创建矩形】的结果。  
创建矩形的四个参数分别表示【左上宽高】，即【本物体左侧与父物体左侧的距离】【本物体顶部与父物体顶部的距离】【本物体宽度】【本物体高度】。  
创建矩形的四个参数可以使用百分比，也可以使用百分比与数值的加减混合。例如：  

    GameObject A = MainPanel.创建矩形("20+10% 40%-20 50% -100+5%").SetColor(128,128,128).SetText("测试");
百分比表示父物体宽度（高度）的百分比。左宽对应宽度，上高对应高度。  
### 嵌套矩形
    GameObject A = MainPanel.创建矩形("0 0 100 100").SetColor(128,128,128);
    GameObject B = A.创建矩形("20 20 40 40").SetColor(255,128,128).SetText("测试");
注：可以不SetColor。此处SetColor只是为了便于观察。  
### 其他函数
    A.AddText("XX");
    string X = A.GetText();
    A.SetEditable(true);//允许编辑。即，A变为输入框
    A.SetEditable(flase);//不允许编辑。可以用于双击重命名时，点击外界后进入不允许编辑状态
    TMP_InputField X = A.GetInputComponent();//用于添加OnValueChange之类的东西
## 布局
### 垂直布局
最简用法：

    public List<角色类> 敌人列表;
    public GameObject Parent;
    var A = Parent.AddMyComponent<VListLayout<角色类>>();
    A.Data = 敌人列表;
    A.GetText = t => t.名称;//此处表示每个矩形如何设置文本。t为此矩形对应的角色类数据。
    A.Draw();
可选用法：

    public List<角色类> 敌人列表;
    public GameObject Parent;
    var A = Parent.AddMyComponent<HListLayout<角色类>>();
    A.Data = 敌人列表;
    A.GetText = t => t.名称;
    A.OnClick = (t, P) => {
        //给列表中每个矩形添加一个单击事件
        //t为这个矩形对应的数据
        //P为这个矩形的GameObject
    }
    A.OnOverText = t => t.注释;//鼠标放在每个矩形上时，显示的注释面板的内容
    A.Menu["锁定"] = (t, P) => {
        //给列表中每个矩形添加一个右键菜单项，选项文本为“锁定”
    }
    A.Draw();
当Data内容变化后，可以再次执行Draw，会重新生成布局。  
也可以使用WaitFor来自动刷新：  

    int B = 敌人列表.Count;
    F();
    void F() {
        WaitFor(敌人列表.Count != B).Then(() => {
            B = 敌人列表.Count;
            A.Draw();
            F();
        });
    }
未来可能会考虑把WaitFor放到布局的定义之中，并添加一个开关，调用时选择是否自动刷新。
水平布局与网格布局同理，不再介绍。
### 目录树布局
目录树布局的用法也跟垂直布局一样，都是那些字段。但是区别在于：Data字段的类型必须是JSON。
JSON的用法例如：

    JSON A = new();
    A["武器"]["一阶武器"]["砍刀"] = "价格：10。基础伤害：5。重量：7。";
    A["武器"]["一阶武器"]["长剑"] = "价格：10。基础伤害：3。重量：2。";
    A["武器"]["二阶武器"]["光剑"] = "价格：50。基础伤害：10。重量：5。";
    A["防具"]["铠甲"] = "价格：100。减伤：20% 格挡：10。重量：20。";
键可以有任意多重，值只能是string。  
JSON定义为一个string到JSON的字典，在for与foreach时它的用法与字典一样。  
每个JSON有一个StringData字段，如果此字段不为null，那么意味着这个JSON是叶节点。否则意味着它是中间结点，拥有子节点。  
### 选项卡布局
选项卡布局在调用时非常简单：

    GameObject A = MainPanel.创建矩形("0 0 100 100");
    A.AddTab<测试面板>();
    A.AddTab<第二测试>();
    
    class 测试面板 : 面板 {
        public 测试面板(GameObject X) : base(X) {
            SetTitle("测试");//选项卡布局中本面板的标题
            X.创建矩形("0 0 10% 10%").SetColor(255,0,0);//测试的面板内容。可以换为其他内容
        }
    }
    class 第二测试 : 面板 {
        public 第二测试(GameObject X) : base(X) {
            SetTitle("第二");
            X.创建矩形("20% 20% 10% 10%").SetColor(0,0,255);
        }
    }
选项卡可以拖动排序、拖动移出（成为单独选项卡面板）、拖动移入合并。  
使用如下函数存档与读档：  

    SaveAllTab();
    LoadAddTab();
存档以Title为识别，因此两个面板不要有相同的Title。  
存档只保存Title的布局关系，不保存面板内容。读档时需要先创建面板，再读档，读档只修改面板的布局。  
读档时允许当前界面中缺少或多余面板。只处理当前界面与存档中都有的面板。    
## 流程
### 时刻执行
常规的时刻执行：  

    AddUpdate(() => {
        //时刻执行的内容
        //用于函数式编程，在静态函数中添加时刻执行的操作。
    });
绑定某物体的时刻执行：

    GameObject A;
    A.AddUpdate(() => {
        //时刻执行的内容。当物体A被销毁时，此内容也不再执行。
        //OnOverText基于此而实现：时刻检测鼠标是否在物体上，如果在，那么创建注释面板。如果物体销毁，那么不再检测。
    });
    
绑定标识符的时刻执行：

    UpdateDo["标识符1"] = () => {
        //时刻执行的内容。当此标识符被赋予新的事件时（或删除时），旧的事件不再执行。
        //WaitFor基于此而实现。每次WaitFor都创建一个新的标识符、时刻执行监听。
    }
执行一次：

    Do(() => {
        //执行的内容
        //本函数用于解决【多线程时，子线程无法操作UGUI】的bug。在子线程中操作UGUI时，可以使用Do。
    });
### 等到X时执行
WaitFor有很多应用，例如前面介绍过的数据改动时布局自动刷新，再例如双击重命名：  

    var A = MainPanel.创建矩形("0 0 100 100");
    A.OnDoubleClick(() => {
        A.SetEditable(true);
        WaitFor(() => Input.OnMouseButtonDown(0) && !A.IsMouseHere()).Then(() => {
            A.SetEditable(false);
        });
    });
游戏中可能会有技能延迟生效，也可以使用WaitFor。  
如果战斗结束，那么还未生效的延迟技能需要被取消。此时可以使用ExitOn：  

    int A = 当前回合;
    WaitFor(() => 当前回合 > A + 延迟回合数).Then(() => {
        //释放技能
    }).ExitOn(() => 当前回合 == 0);//战斗结束后回合会自动归零。此处检测回合归零，然后让未生效的技能在战斗结束后退出WaitFor。
### 其他函数
    SetTimer(5,()=>{}).ExitOn(()=>{});//计划五秒后执行XX。如果五秒内出现XXX，那么取消计划。
    执行X次(5,i=>{});//对For的简化。等价于for(int i=0;i<5;i++){}
    A.ForEach(t=>{});//A是一个数组或列表。是对Foreach的简化，等价于foreach(var t in A){}
## 文件
### 存档读档
    自定义类 X;
    FileWrite("C:/XX.txt",X);
    自定义类 A = FileRead<自定义类>("C:/XX.txt");
第一，X可以是任何类型，自动JSON序列化存档。  
第二，存档会被简单加密。  
第三，X中只可以包含string,int等基本类型，以及基于基本类型而定义的高级类型（比如List）。不要包含GameObject、Sprite之类的东西。  
第四，文件名任意，可以是txt后缀，也可以是其他后缀或者无后缀。  
### 错误日志
局部错误：

    try{
        //XX
    }catch(e){
        ErrorLog(e,"C:/Error.txt");
    }
全局错误：

    CatchAllError("C:/AllError.txt");
第一，输出的错误日志会包含【时间】【错误内容】【错误位置】。  
第二，错误日志本质是对文件AddText，而不是Set。这导致日志会越积累越多，如果有大量错误日志，那么要定期清理。  
第三，全局错误会自动捕获所有【未手动捕获】的异常。
## 功能
//未完待续









