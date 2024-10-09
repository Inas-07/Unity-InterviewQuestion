上次更新：上个月

### Unity 协程和线程,进程的区别
- 其实很简单，首先需要理解进程和线程是怎么一回事：进程是Windows系统中的一个基本概念，他包含着运行一个程序所需要的基本资源。一个正在运行的应用程序在操作系统中被视为一个进程，进程可以包括一个或多个线程。　　
- 进程和线程的区别通过以上，一目了然。　　
- 再谈谈线程和协程的区别。一般应用一个应用程序只使用线程这一“资源”。
- 需要明确，<i><b>Unity只使用了一个线程</b></i>，但是，我们需要”同时做很多事“，那Unity作为单线程，该如何去做，协程，就来了，协程是一种”伪线程“。
协同程序（coroutine）.,即协作式程序，一系列互相依赖的协程间依次使用CPU，每次只有一个协程工作，而其他协成处于休眠状态。协程实际上是在一个线程中，只不过每个协程对CPU进行分时，协程可以访问和使用unity的所有方法和component。
- <i><b>同一时间只能执行某个协程</b></i>，协程适合对某个任务进行分时处理。　　　　
- 控制代码在特定的时间执行。
- 协程不是线程，也不是异步执行，跟Update一样，在主线程中执行。　　　　
- 不用考虑同步和锁的问题。　　　　
- 协程是一个分部组件，遇到条件（yield return）会挂起,直到条件满足才会被唤起执行后面的语句。

### 本地坐标系 世界坐标系
- 世界坐标系：世界坐标是指物体在场景中的坐标，当某个物体没有父物体时，它的position即为世界坐标的position，rotation同理；本地坐标是物体相对于它的父物体的坐标而言，这个相对坐标是以父物体本身为坐标轴进行计算的，与世界坐标没有必然联系。而对于没有父物体的物体，可以认为不存在本地坐标这种说法。
- 本地坐标系：当某个物体有父物体时，它的inspector栏transform中的position实际是localposition，即本地坐标。
- 使用TransformPoint方法将本地坐标系转为世界坐标系

### Unity调整旋转
- 矩阵旋转：
    - 优点：旋转轴可以是任意向量
    - 缺点：<b><i>消耗时间和内存</i></b>。旋转其实只需要知道一个向量+一个角度(共4个信息值)，但矩阵却用了16个元素、
- 欧拉角旋转
    - 优点：容易理解，形象直观；表示更方便，只需要三个值(分别对应x、y、z轴的旋转角度)
    - 缺点：欧拉角这种方法是要按照一个固定的坐标轴的顺序旋转的，因此不同的顺序会造成不同结果；欧拉角旋转会造成万向锁现象，这种现象的发生就是由于上述固定的坐标轴旋转顺序造成的。理论上，欧拉角旋转可以靠这种顺序让一个物体旋转到任何一个想要的方向，但如果在旋转中不幸让某些坐标轴重合，就会发生万向锁现象，这时就会丢失一个方向上的旋转能力(两个旋转轴(环)重叠)，也就是说在这种状态下，我们无论怎么旋转(还是按照原先的旋转顺序)，都不可能得到某些想要的结果，除非打破原先的旋转顺序或者同时旋转三个轴。
    - <b><i>由于万向锁的存在，欧拉旋转无法实现球面平滑插值</i></b>。
    - 万向锁的简单解决办法：构造一个不同的旋转层级顺序，但是万向锁总是会在某一个顺序发生，调整旋转顺序不是根本解决办法。（Unity使用的是Z-X-Y顺规，即旋转顺序为z轴、x轴、y轴，虽然某些情况下会出现万向锁，但是这种顺规出现万向锁的概率最小）
    - <i>万向锁解决办法：将欧拉角转换为四元数，对四元数进行Slerp插值</i>，再将这一系列四元数转换为对应的欧拉角，然后作用于需要进行旋转的对象。这种做法缺点在于消耗内存，但是可以使物体任意旋转，灵活度高。
    - 使用欧拉旋转出现旋转路径偏移的根本原因：在万向锁情况下对欧拉角的插值不是线性的。(突变)
    - 静态欧拉角：其旋转轴使用的是静止不同的参考系。
    - 动态欧拉角：使用object本身的坐标系，因而会随着object旋转而旋转。(局部坐标系会随着对象的旋转而旋转)
- 四元数旋转
    - 优点：可以避免万向锁；只需要一个4维的四元数就可以执行绕任意过原点的向量的旋转，方便快捷，在某些实现下比旋转矩阵效率更高；而且四元数旋转可以提供平滑插值。
    - 缺点：比欧拉旋转稍微复杂了一点，因为多了一个维度，理解更困难，不直观。

### UGUI底层
想详细了解unity UGUI的渲染流程，但是感觉没有专业文献，如何解决这种难题？ - 柴柴爱Coding的回答 - 知乎  
https://www.zhihu.com/question/312113725/answer/2378626636

在游戏运行过程中是如何显示一个UI，并捕获用户的输入和操作从而驱动应用程序作出反应的呢？其中涉及到显示、事件检测、事件调度、事件处理。

* 显示：简单来说在计算机中所有的形状都是由点和面形成的，由点构成三角面，三角面构成我们所看到的UI。
* 捕获：通过<b><i>射线检测</i></b>，从摄像机出发出射线穿过当前Pointer或者Touch所在位置，获得碰撞的GameObject
* 调度：发生在_**EventSystem的Update**_中，找到当前的Module并处理，处理中包含了对移动点击拖动事件的处理。
* 处理：对于发生的事件，会由ExecuteEvents中的Execute方法找到合适的方法处理。



### NGUI与UGUI [还是得实际用一下]
1. NGUI与UGUI的区别
UGUI是官方4.6以后推出的一套UI组件，相对于之前的GUI倒是改头换面了，可视化操作，便捷、省心。NGUI则是unity一直以来最强的UI插件（没有之一），甚至还是官方新UI的导师。
    1. ugui的ui根目录为canvas（画布），ngui则是uiroot。在命名上官方似乎更贴合想象力。
    2. 在屏幕自适应方面，ugui为render mode。ngui则为scaling style。
    3. anchor（锚点）的使用方式差不多，都是用来固定位置，在可视化方面，ugui的花瓣锚点真不太好调。
    4. ngui灵活性不是一般的高，随意创建一个sprite，加了boxcollider，它就可以是按钮、滑动条……
    5. ugui的sprite的切图功能真心不错。ngui使用图集不能直接拖拉（毕竟是三方插件）略不方便。
    6. ngui的tween动画功能很省心，无需额外定义代码，使用封装好的脚本就可以实现一些简单动画，叠加脚本甚至能实现相对复杂的动画效果。

![nguiugui](img/nguiUgui.webp)

总结：
- NGUI与UGUI的区别
- uGUI的Canvas 有世界坐标和屏幕坐标
- uGUI的Image可以使用material
- UGUI通过Mask来裁剪，而NGUI通过Panel的Clip
- NGUI的渲染前后顺序是通过Widget的Depth，而UGUI渲染顺序根据Hierarchy的顺序，越下面渲染在顶层.
- UGUI 不需要绑定Colliders，UI可以自动拦截事件
- UGUI的Anchor是相对父对象，没有提供高级选项，个人感觉uGUI的Anchor操作起来比NGUI更方便
- UGUI没有Atlas一说，使用Sprite Packer
- UGUI的Navigation在Scene中能可视化
- UGUI的事件需要实现事件系统的接口，但写起来也算简单

2. 各自的优缺点
    1. NGUI还保留着图集，需要进行图集的维护。而UGUI没有图集的概念，可以充分利用资源，避免重复资 源。
    2. UGUI出现了锚点的概念，更方便屏幕自适应。
    3. NGUI支持图文混排，UGUI暂未发现支持此功能。
    4. UGUI没有 UIWrap 来循环 scrollview 内容。
    5. UGUI暂时没有Tween组件。

3. 使用选择总结
UGUI由于是Unity原生支持的，所以使用上会更加的人性化。并且伴随着版本升级功能会越来越强，逐渐将成为主流ui方案。NGUI是UGUI出现之前的产物，通过MeshRenderer来实现类似CanvasRenderer的功能，概念上有点蹩脚。作为一个插件虽然已最大努力让UI开发工作变得简单，但相比能够让UnityEditor做出相应修改的UGUI（如RectTransform的出现），其易用性是没法比的。综合来说，新的项目建议使用UGUI，学习成本不高，工具流更有助于提高开发效率。

### UI事件渗透
http://www.xuanyusong.com/archives/4241

【当需要“渗透”的时候，UI逻辑本身可能就比较复杂了，代码也会变多一些】

### DoTween
相关的git：https://github.com/jeffreylanters/unity-tweens

先来看一下我画的DoTween的类图结构：
![](img/dotween.png)

只要明白上面类图中的结构，我认为就差不多搞懂了DoTween的大体实现方式了。首先在Unity中之所以可以使用例如：transform.DoMove(new vector3(1,1,1)，2)；这种的调用方式，是因为DoTween定义了大量与Transform、Rigibody、Material、Camera等相关的拓展函数，这些拓展函数就定义在了ShortcutExtensions类中。


### 碰撞器(Collider)和触发器(Trigger)的区别
- 碰撞器是触发器的载体，而触发器只是碰撞器身上的一个属性。
- 当Is Trigger=false时，碰撞器根据物理引擎引发碰撞，产生碰撞的效果，可以调用OnCollisionEnter/Stay/Exit函数；
- 当Is Trigger=true时，碰撞器被物理引擎所忽略，没有碰撞效果，可以调用OnTriggerEnter/Stay/Exit函数。
- 如果既要检测到物体的接触又不想让碰撞检测影响物体移动或要检测一个物件是否经过空间中的某个区域这时就可以用到触发器

### UGUI canvas的三种模式
【Unity3d游戏开发】浅谈UGUI中的Canvas以及三种画布渲染模式: https://www.cnblogs.com/msxh/p/6337338.html

1. **Screen Space-Overlay**: 画布会填满整个屏幕空间，并将画布下面的所有的UI元素置于屏幕的最上层（如此，显示这个UI画布时，3D场景内的所有东西都会被遮挡住）
2. **Screen Space-Camera**: 画布会被放置到摄影机前方，画布看起来绘制在一个与摄影机固定距离的平面上；【摄像机与画布之间的场景内物体也会被渲染出来。黑神话悟空的物品栏就是一个例子】
3. **World Space**: 画布被视为与场景中其他普通游戏对象性质相同的类似于一张面片（Plane）的游戏物体。它有一个单独的参数Event Camera，用来指定接受事件的摄像机。这种模式可以用来实现跟随人物移动的血条或者名称。
【打比方：血量条UI将是敌人GameObject的一部分，UI通过Event Camera来渲染，UI的大小也会取决于此时Camera与敌人之间的相对位置等因素。也因此，这是一种画布不与屏幕对应的渲染模式】

## 从这继续看

### UnityMonoBehaivoir生命周期

![](img/Center.png)

MonoBehaviour是Unity中所有脚本的基类，如果你使用js的话，脚本会自动继承MonoBehaviour。如果使用C#的话，你需要显式继承MonoBehaviour。
在我们使用MonoBehaviour的时候，尤其需要注意的是它有哪些可重写函数，这些可重写函数会在游戏中发生某些事件的时候被调用。我们在Unity中最常用到的几个可重写函数是这几个：
- Awake：当一个脚本实例被载入时Awake被调用。我们大多在这个类中完成成员变量的初始化
- OnEnable：当对象变为可用或激活状态时此函数被调用。
- Start：仅在Update函数第一次被调用前调用。因为它是在Awake之后被调用的，我们可以把一些需要依赖Awake的变量放在Start里面初始化。 同时我们还大多在这个类中执行StartCoroutine进行一些协程的触发。要注意在用C#写脚本时，必须使用StartCoroutine开始一个协程，但是如果使用的是JavaScript，则不需要这么做。
- Update：当MonoBehaviour启用时，其Update在每一帧被调用。
- FixedUpdate：当MonoBehaviour启用时，其 FixedUpdate 在每一固定帧被调用。
- LateUpdat 方法 :是在所有Update函数调用后被调用。这可用于调整脚本执行顺序。例如:当物体在Update里移动时，跟随物体的相机可以在LateUpdate里实现
- OnGUI 方法:  渲染和处理GUI事件时调用 这里不是 没帧都调用哦
- OnDisable：当对象变为不可用或非激活状态时此函数被调用。
- OnDestroy：当MonoBehaviour将被销毁时，这个函数被调用。
- 【协程看图上，似乎是在Update之后，LateUpdate之前被调用】


## Unity GC
【Unity3d游戏开发】浅谈Unity中的GC以及优化:https://blog.csdn.net/worisaa/article/details/64121436

 Unity GC垃圾回收 : https://blog.csdn.net/wangjiangrong/article/details/89011349

***

## 10000个monobehavior，每个各自执行update，和放到一个update里执行，哪个效率更高？为什么？
 https://blog.unity.com/engine-platform/10000-update-calls

【结论】
放到一个update里效率更高。
这里要引出并回答下面的问题：
- 各个Monobehavior的Update（如果有）是如何被调用的？
  - 在往GameObject添加Monobehavior Component时，unity会检查它是否有Update等诸如此类的方法；若有，Unity会将其添加到自己的一个list中。
  - 对update：每一帧中，Unity就要遍历这个list。这一过程除了要调用各个Update外，还要做下面的检查
    1. 先遍历所有Behavior【占用约15%】
    2. CallMethodIfAvailable: 检查MB对应的GO是否还活着（没有被Destory）【占用约22%】
    3. ScriptingInvocationNoArgs: 准备调用方法；ScriptingInvocationNoArgs实例表示一个native side stuff对managed side method的调用【占用约21%】
    4. scripting_method_invoke: 检查传入的参数是否正确【9%】
    5. 调用IL2CPP虚拟机的Runtime::Invoke: 首先检查是否有Update方法【10.2%】
    6. 对被调用的方法的签名（参数列表），生成一个RuntimeInvoker function，并调用它
    7. Update方法被调用【例子中，占用0.4%】

所以：放到一个Update里运行效率更高，因为能省去非常多的检查开销。这也为怎么做优化提供思路，但也意味着我们要自己做必要的检查，这也是费功夫的事。

### 为什么instantiate会耗性能？
第一次执行GameObject.Instantiate的时候有明显卡顿，该怎么解决？

原文链接：https://blog.csdn.net/qq_35957011/article/details/121148527

Instantiate的卡顿与三部分开销相关：
1. 相关资源加载
2. 脚本组件的序列化
3. 构造函数的执行

绝大部分原因均是相关资源加载导致。所以，我们的建议如下：

1. 通过 Profiler 查看 Instantiate 具体的CPU分配情况；
2. 如果是资源加载导致的性能瓶颈，则一方面通过<b><i>简化资源</i></b>来缓解CPU耗时压力，另一方面通过 AssetBundle <b><i>依赖关系打包</i></b>将资源预先加载，即<u>将此处 Instantiate 的总体耗时拆分，平摊到之前帧进行执行</u>（比如切换场景处等），从而让 Instantiate 实例化操作的局部耗时更加平滑；
3. 如果是脚本组件序列化导致的性能瓶颈，则可尝试减少脚本中的序列化信息；
4. 如果是构造函数的执行导致的性能瓶颈，一般只能在策略上进行规避，比如降低 Instantiate 的调用频率等。

回答问题：
如果只是第一次加载会很卡，后面加载还算快，那说明AB中有可共用的部分。那么就可以将其拆分开来加载。

### AssestBundle基础:
https://blog.csdn.net/m0_48554728/article/details/123562033 |【我自己找的，Unity文档的机翻】

https://docs.unity3d.com/550/Documentation/Manual/AssetBundleCompression.html | 【Unity文档】

### AB包的压缩格式有哪些？你知道哪些压缩算法？
Unity3D:资源包的压缩(Asset Bundle Compression)

https://blog.csdn.net/zgl159040290/article/details/52806507

* 无压缩：加载最快，大小最大
* LZ4: 块压缩；压缩率不如LZMA，但是解压速度非常快，以至于可以在运行时再解压（这也是为什么MD都用LZ4）
* LZMA：流压缩，标准压缩格式；文件大小最小，解压和加载时间最长

`WWW.LoadFromCacheOrDownload`: As data arrives from the (network) socket, Unity will decompress it (from `LZMA`) and recompress it in `LZ4` format. This recompression occurs during the download streaming, which means the cache compression begins as soon as enough of the data is downloaded, and continues incrementally until the download is complete. After that, data is read from the cached bundle by decompressing chunks on-the-fly when needed.

### Uielement
一个新的用于UnityEditor的UI框架,实现类似于Web前端
Unity 2019.1的UIElements功能介绍
https://www.linecg.com/news_8740.html

## 动态合批静态合拼

https://gwb.tencent.com/community/detail/114323 | 【非常好的文章】

### 静态批处理（static batching）：

`Static` 的含义：如果我们的游戏场景中有一些共享同一材质的模型存在，并且**这些模型一直都不会移动、旋转和缩放**。我们可以将这些模型设置为Static。

对于每种共享材质（sharedMaterial），Unity在打包时，会将各个模型的Vertex buffer与Index buffer，合并为1个大、新的Vertex buffer与Index buffer下，并记录每个模型原本的Index Buffer在新的大Index buffer中的位置`[start, end]`

其作用有：
1. 优化掉了各个单独模型的局部坐标到世界坐标的转换的计算；
2. 虽然不减少DrawCall次数，但因为（后面这段建议背诵）子模型有材质共享，多次DrawCall调用中没有了**渲染状态**的切换，渲染API会缓存绘制命令，进而实现**渲染优化**。
3. [debuff] 应用打包之后文件体积增大，运行时所占用的内存也增大。
    - 不开启Static batching，GameObject共享的模型会在应用程序包内或者内存中只存在一份，绘制的时候提交模型顶点信息，然后设置每一个GameObjec的材质信息，分别调用渲染API绘制。
    - 开启Static batching，在Unity执行Build的时候，场景中所有引用相同模型的GameObject都必须将模型顶点信息复制，并经过计算变化到最终在世界空间中，存储在最终生成的Vertex buffer中。这就导致了打包的体积及运行时内存的占用增大。

### 动态批处理（dynamic batching）:
`Dynamic`的含义：如果我们开启了Dynamic batching，Unity会自动地将所有**共享同一材质**的动态GameObject在一个Draw call内绘制。

在进行场景绘制之前将所有的共享同一材质的模型的顶点信息变换到世界空间中，然后通过_**一次**_Draw call绘制多个模型，达到合批的目的。

#### 一些注意点：
* 模型顶点变换的操作是由CPU完成的，所以这会带来一些CPU的性能消耗（每一帧都有，因为每一帧都要计算）。
* Dynamic batching只能处理一些小模型：
  - 计算的模型顶点数量不宜太多，否则CPU串行计算耗费的时间太长会造成场景渲染卡顿
  - Unity限制能进行Dynamic batching的模型最高能有900个**顶点属性**（区别于900个顶点）；顶点属性包含position, normal, UV这些

#### 不是什么时候都能做Dynamic batching的：
1. Unity的文档中说，GameObject之间如果有镜像变换不能进行合批。例如，"GameObject A with +1 scale and GameObject B with –1 scale cannot be batched together"。
2. 使用Multi-pass Shader的物体会禁用Dynamic batching，因为Multi-pass Shader通常会导致一个物体要连续绘制多次，并切换渲染状态（这是核心问题，静态batching就是希望能减少渲染状态切换次数。
3. 后面还有三点，没看明白（缺了些相关知识）；

### GPU Instancing

一句话概括：特别适用于场景中重复出现的、完全一样的Mesh（有相同拓扑、相同material）；

渲染时，这些重复出现的Mesh要有相同的Mesh拓扑结构以及相同的材质，但可以有不同的参数（比如，颜色与scale）

文档：https://docs.unity3d.com/2019.4/Documentation/Manual/GPUInstancing.html

***

### DrawCall、Batch、SetPassCall的区别
https://zhuanlan.zhihu.com/p/366779113

### 还有一些面试题
https://blog.csdn.net/qq_35957011/article/details/121148527

### 图像（材质）压缩

`Dithering`是什么？是一种图像的压缩方式（技巧）：https://shihn.ca/posts/2020/dithering/

![](img/img_compression.png)