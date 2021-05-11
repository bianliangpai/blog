## 概要
定位 rviz 做消息显示正常，但 rqt_rviz 会 crash ( coredump ) 的问题  

## 定位过程记录
1. 搜索报错关键字，在 ros 社区发现一个 long-standing 的[类似问题](https://github.com/ros-visualization/rqt_rviz/issues/6)，但报错位置不相同。无法判断是否属于同一问题。  
2. 根据运行时报错信息，可以知道报错位置是 rviz 使用的底层渲染引擎 OGRE 里。想要看一下报错位置的源码，发现 OGRE VERSION 定义与打印出来的版本不相同；查找源码，发现确实 tag 与 VERSION 不同。  
3. 由于单独运行 rviz 并不会挂死，所以想要在 OGRE 崩溃处加上崩溃时的调用堆栈打印，突然想到 gdb 就可以看 backtrace（好久不写代码，忘记了正确的调试姿势）。  
4. 复现 crash 问题，注意到 gdb 中的 backtrace 信息与 ros wiki 上写的已知问题中指明的调用位置不相同，且 wiki 中提到的问题主要是调用冲突，而本地环境定位到的语句是分配 buffer 失败主动抛出异常。  
5. 注意到 ros wiki 中提到 rqt 0.2.3 版本一切正常，问题现象中 rqt 0.2.4 开始出现。尝试使用 0.2.3 版本接收 lqr_graph_plugins 消息。将相关 python packages 放入单独的路径，并使用当时的解释器( python 2.7 )来运行。到最后也没有完全解决语法不兼容问题，小黑盒内源环境没有某些 python2 的 package。  
6. 观察 rqt_rviz 源码，注意到其代码量非常少，进而注意到整个 rviz 界面都被封装进了 VisualizationFrame : public QMainWindow 的 class 里。观察 rviz 源码，集成代码中确实也基本只用到了 VisualizationFrame。所以我完全可以写一个自己的 rqt_rviz，或者叫 qt_rviz，并把其他 plugin 脱离 rqt 的依赖。  
7. 通过 crash 堆栈逐层看代码，注意到在某一层使用到了 singleton 的模式；并借由 crash 的位置是一个类似释放锁的函数里，由此推断 crash 的原因大概率是没有处理好资源竞争。   
8. 基于非常想把事情做好的心理想法，又做了些实验。在抛出 exception 处加了些打印，注意到只有启动 rqt_rviz 后才会才会有这些打印，开始猜测资源冲突与 rqt 无关，纯粹是 rqt_rviz 的内部冲突。  
   于是尝试取消部分显示内容（将部分内容降频）后又进行了一些尝试，此时就可以持续显示比较长的一段时间。  
9. 观察另外一处挂死位置处源码，加上一些打印，发现有些变量的地址经常重复出现，怀疑 OGRE 里有使用内存池，并通过阅读源码验证想法。
   通过进一步加打印定位到出现 segment fault 处确实是调用了一个已经在另一线程里析构的对象。由此可以判断此类挂死是 rqt 和 rviz 同时使用时的运行冲突。  

## 复盘
1. 当时都已经想到了重新编译 OGRE 源码（替换系统环境下的 so），应该进一步想到可以与 vscode 等 IDE 结合 debug OGRE 库的。使用打桩的方式效率低下。

## 结论
暂时认为跟社区里的已知问题是同一问题。
