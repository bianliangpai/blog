# 过程记录

## 原始问题

最近在重构 rqt_launch ，尝试用上新探索出来的“大 View 嵌套小 View，大 Model 嵌套多个小 Model”的模式。重构到今天，遇到了一个某 sub view 与 model 数据通信的问题。  
这个问题的难点在于：我无法合理地在 subview 中需要的位置直接获取 model 的数据。我要么在 subview 里生成一个 model 的实例，进而以 function call 的形式调用相应接口；要么 model 主动将数据更新给 subview。这两种方案，前者耦合了 subview 的接口形式（但现在想想，其实 signal 的形式也是同样），后者需要在 subview 里用一个成员变量才存储这个数据。  

## GUI Architecture 演变思考

基于这个背景，我开始对 puremvc 的实现产生兴趣，想要借鉴一下成熟框架是如何解决类似的问题的？是否也是这种大View嵌套多个小View、大Model嵌套多个小Model的形式？答案是否定的。puremvc 的模式是从 model, view, controller 中提取 proxy, mediator, command 来填充业务逻辑，从而保证核心部分的可复用性[[1]](#参考)。  
除了这篇介绍 puremvc 的外，我还凑巧看到一篇介绍 GUI 框架演变的文章[[2]](#参考)。两者都提到了“可复用性”的概念，这使我意识到 mvc 中 controller 的出现原本竟是为了提取 view 中的公有部分。类似的，[[2]](#参考)中还提到 mvp 的出现是为了解决 mvc 中 view 难以测试的问题。  
这也就是说，主流 GUI Architecture 是一个在演进中不断变化的事务，并非是我一直固有的那种“经典”、“永恒”的固定套路。这一点是我在了解了 GUI Architecture 演进历史后得到的感悟。  

## 进一步理解 QT

在学习 mvc 历史时，我注意到 controller 的定义是“翻译用户的输入并依照用户的输入操作模型和视图”。看到这句话，我突然联想到在我当前使用的框架里，这一部分和显示的部分（view）是在同一个 class 里实现的。我突然意识到 qt 实现的这些 widgets 可能并不只是代表 view。  
简单搜索了一下，发现之前已经有人提出了类似的疑问[[3]](#参考)。  
Stackoverflow 上这个问题下面的第二个回答，答主提到 "Qt's MVC only applies to one data structure... If you want an MVC architecture for your whole program, Qt hasn't such a 'huge' model/view framework."，并且在答案的最后，答主给出了和我一样的答案—用自己定义的 View 和 Model 集成 Qt 预定义的 widgets。

## 重新认识 GUI

在思考[GUI Architecture 历史](#GUI Architecture 演变思考)时，我除了对已知的 puremvc 进行了进一步的了解，还尝试用 "best mvc framework" 这样的关键词来进一步探索。在这个过程中，我发现那些在前端界耳熟能详的框架们也是类似的套路，如 react, vue 等[[4]](#参考)。这令我颇有恍然大悟之感，可不就是吗，前端后台这一套也是典型的 GUI 应用啊。之前说到 GUI 就是 Qt 的思考模式很有问题。  
另一个例子[[5]](#参考)是在思考[QT](#进一步理解 QT)时，通过搜索引擎找到的。文中以“用鼠标改变三角形视角”为例，介绍了一个基于 OpenGL 的 MVC 样例程序。由于之前写 3d Viewer 的经历，这个例子让我立马联想到，其实之前的这个 3d Viewer 也算是 GUI 应用程序（也可以有 MVC 的这一套）。但当时竟然丝毫没有意识到这一点，有点感叹。  

# 参考
1. [《PureMVC--一款多平台MVC框架》](https://www.jianshu.com/p/47deaced9eb3)  
2. [《2020-3-8-MVC、MVP、MVVM模式演变简析》](https://cloud.tencent.com/developer/article/1641997)  
3. [Why Qt is misusing model/view terminology?](https://stackoverflow.com/questions/5543198/why-qt-is-misusing-model-view-terminology)  
4. [Top 10 JavaScript MVC Frameworks](https://www.bbconsult.co.uk/blog/top-10-javascript-mvc-frameworks)  
5. [OpenGL Windows GUI Application](http://www.songho.ca/opengl/gl_mvc.html)  
