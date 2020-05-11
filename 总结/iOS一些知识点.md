analyze:静态分析工具，用来分析可能存在的内存泄漏的点。对有标记的行进行分析，如果确认存在那么就直接改正，如果无法确认，先放在那里。

再用instrument的leaks，来进行内存泄漏的分析。如果有爆红的记录，那么就是存在内存泄漏，就需要根据提示具体定位代码后进行改正。

使用真机调试 最好使用 release包测试

Time Profile:时间分析工具用来检测应用CPU的使用情况.可以看到应用程序中各个方法正在消耗CPU时间.使用大量CPU不一定是个问题, 主要用来减少CPU的压力。(能够复用的对象就不要重复创建)

Core Animation: 得到FPS，当低于45时，卡顿比较明显         (主要用来减轻GPU的压力)。

Color Blended Layers: 查看图层混合，如果图层混合就会显示红色，如果上层有透明度，那么上层颜色和下层颜色会发生图层混合，这个过程会消耗一定的GPU资源。

Color Hits Green and Misses Red   光栅化。系统分配一定的缓存，如果操作缓存会引发离屏渲染，最好不要开启。一般在图像内容不变的情况下才使用光栅化，如果是cell上，绘制频繁，会引发离屏渲染。

（1）系统给光栅化缓存分配了一个固定的大小，因此不能过度使用，如果超出了缓存也会造成离屏渲染。
（2）缓存的时间为100ms，因此如果在100ms内没有使用缓存的对象，则会从缓存中清除。



Color Misaligned Images(图片大小)： 如果图片大小压缩/拉伸，那么图片显黄色。那GPU会有一定的资源消耗

Color Offscreen-Rendered Yellow:

离屏渲染Off-Screen Rendering 指的是GPU在当前屏幕缓冲区以外新开辟一个缓冲区进行渲染操作。还有另外一种屏幕渲染方式-当前屏幕渲染On-Screen Rendering ，指的是GPU的渲染操作是在当前用于显示的屏幕缓冲区中进行。 离屏渲染会先在屏幕外创建新缓冲区，离屏渲染结束后，再从离屏切到当前屏幕， 把离屏的渲染结果显示到当前屏幕上，这个上下文切换的过程是非常消耗性能的，实际开发中尽可能避免离屏渲染。

触发离屏渲染Offscreen rendering的行为：

（1）drawRect:方法

（2）layer.shadow

（3）layer.allowsGroupOpacity or layer.allowsEdgeAntialiasing

（4）layer.shouldRasterize

（5）layer.mask

（6）layer.masksToBounds && layer.cornerRadius

Leaks:主要是用来检验内存泄漏（对象使用完没有释放）。代理强引用，block，timer等

Zombies:它会用一个僵尸来替换默认的dealloc实现，也就是在引用计数降到0时，该僵尸实现会将该对象转换成僵尸对象。僵尸对象的作用是在你向它发送消息时，就不会向之前那样Crash或者产生 一个难以理解的行为，而是放出一个错误消息，它会显示一段日志并自动跳入调试器， 因此我们就可以找到具体或者大概是哪个对象被错误的释放了。主要用来检测过度释放(对象)。启用instrument的zombies,如果程序发生EXC_BAD_ACCESS错误，那么就会直接定位到过度释放的对象。



XCode可以手动设置zombie Objects，勾选后，会导致内存占用的增长，同时会影响Leaks工具的调试，这是因为设置NSZombieEnabled会用僵尸对象来代替已释放对象。



//创建表

create table if not exists 表名(key1 text,key2 integer, key3 blob);

//插入

insert into 表名(key1, key2, key3) values (value1, value2, value3); 

//更新

update 表名 set key1=value1, key2=value2, key3=value3 where key4=value4 and key5=value5;

//删除

delete from 表名 where key1=value1 and key2=value2;

//根据条件选择数据并降序排列(也可以判断存在)

select * from 表名 where key1=value1 and key2=value2 ORDER BY key3 DESC;



NSUserDefault

NSCoding 归档

文件管理系统,存储文件到沙盒中



内存管理采用了引用计数机制，retainCount 当引用计数为0时释放内存

MRC:是手动管理内存

如果对一个对象使用了alloc、[mutable]copy、retain，new, 那么你必须使用相应的release或者autorelease。

由我们自己来管理对象的生命周期，负责对象的创建和销毁

ARC: 自动管理内存

内存泄漏：对象或变量使用完，没有被释放，一直占用着内存直到程序结束。内存被大量占据后，会影响程序的运行速度，最后被系统杀死。

栈内存的变量：在遇到最近的大括号后自动释放

堆内存的变量：要手动释放

常量区: 常量占用内存，只读状态，决不可修改！

静态存储区:

  1 只初始化一次

  2 如果初始没给值，默认值为0

  3 只有程序退出才释放（永远存在）

  4 只要程序不退出，就存储上一次所赋的值

strong ：强引用，ARC中使用，与MRC中retain类似，使用之后，计数器+1。

weak ：弱引用 ，ARC中使用，如果指向的对象被释放了，其指向nil，可以有效的避免野指针，其引用计数为1。

readwrite : 可读可写特性，需要生成getter方法和setter方法时使用。

readonly : 只读特性，只会生成getter方法 不会生成setter方法，不希望属性在类外改变。

assign ：赋值特性，不涉及引用计数，弱引用，setter方法将传入参数赋值给实例变量，仅设置变量时使用。

retain ：表示持有特性，setter方法将传入参数先保留，再赋值，传入参数的retaincount会+1。

copy ：表示拷贝特性，setter方法将传入对象复制一份，需要完全一份新的变量时。

nonatomic ：非原子操作，不加同步，多线程访问可提高性能，但是线程不安全的。决定编译器生成的setter getter是否是原子操作。

atomic ：原子操作，同步的，表示多线程安全，与nonatomic相反。

MRC的文件可以添加编译选项-fno-objc-arc

ARC的文件可以添加编译选项 -fobjc-arc

多线程:

提高了资源的使用效率，现在市面上多数的CPU都是多核的，多核的CPU运算多线程更为出色;

使用线程可以把耗时的任务放到其他线程进行处理，处理完成后再回到主线程进行UI操作。

常用的有三种: NSThread NSOperationQueue GCD。

  1、NSThread 是这三种范式里面相对轻量级的，但也是使用起来最负责的，

  你需要自己管理thread的生命周期，线程之间的同步。线程共享同一应用程序的部分内存空间，

  它们拥有对数据相同的访问权限。你得协调多个线程对同一数据的访问，

  一般做法是在访问之前加锁，这会导致一定的性能开销。

  2、NSOperationQueue 以面向对象的方式封装了用户需要执行的操作，

  我们只要聚焦于我们需要做的事情，而不必太操心线程的管理，同步等事情，

  因为NSOperation已经为我们封装了这些事情。 

  NSOperation 是一个抽象基类，我们必须使用它的子类。

  3、 GCD: iOS4 才开始支持，它提供了一些新的特性，以及运行库来支持多核并行编程，

  它的关注点更高：如何在多个cpu上提升效率。

  总结：

  \- NSThread是早期的多线程解决方案，实际上是把C语言的PThread线程管理代码封装成OC代码。

  \- GCD是取代NSThread的多线程技术，C语法+block。功能强大。

  \- NSOperationQueue是把GCD封装为OC语法，额外比GCD增加了几项新功能。

​    \* 最大线程并发数

​    \* 取消队列中的任务

​    \* 暂停队列中的任务

​    \* 可以调整队列中的任务执行顺序，通过优先级

​    \* 线程依赖

​    \* NSOperationQueue支持KVO。 这就意味着你可以观察任务的状态属性。

  但是NSOperationQueue的执行效率没有GCD高，所以一般情况下，我们使用GCD来完成多线程操作。

多线程目的是提高运行效率, 它能够利多核CPU,提高设备资源利用率。

NSThread：自己控制线程的生命周期

NSOperation

GCD

推出的时间 iOS4 目的是用来取代NSThread（ios2.0推出）的，是 C语言框架，它能够自动利用更多CPU的核数，并且会自动管理线程的生命周期。

  CGD的两个核心概念：任务， 队列

  任务：即为在block中执行的代码。

  队列：用来存放任务的。

  注意事项： 队列 != 线程。队列中存放的任务最后都要由线程来执行!。队列的原则:先进先出,后进后出(FIFO/ First In First Out)

2 队列又分为四种种：1 串行队列 2 并发队列 3 主队列 4 全局队列

  串行队列： 任务一个接一个的执行

  并发队列： 队列中的任务并发执行

  主队列： 跟主线程相关的队列，主队列里面的内容都会在主线程中执行（我们一般在主线程中刷新UI）

  全局队列： 一个特殊的并发队列。

'同步'执行任务:dispatch_sync，只能在'当前'线程中执行任务,不具备开启新线程的能力

 异步'执行任务:dispatch_async，可以在'新'的线程中执行任务,具备开启新线程的能力.



串行队列同步执行，既在当前线程中顺序执行

串行队列异步执行，开辟一条新的线程，在该线程中顺序执行

并行队列同步执行，不开辟线程，在当前线程中顺序执行

并行队列异步执行，开辟多个新的线程，并且线程会重用，无序执行

主队列异步执行，不开辟新的线程，顺序执行

主队列同步执行，会造成死锁





分别异步执行两个耗时操作;其次:等两次耗时操作都执行完毕后,再回到主线程执行操作？？

dispatch_group_t group = dispatch_group_create();

dispatch_queue_t queue = dispatch_get_global_queue(0, 0); // 全局并发队列

dispatch_group_async(group, queue, ^{// 异步执行操作1

// longTime1

});

dispatch_group_async(group, queue, ^{ // 异步执行操作2

// longTime2

});

dispatch_group_notify(group, dispatch_get_main_queue(), ^{

// 在主线程刷新数据
 // reload Data
 });







**NSOperation**

NSOperation: 抽象类,不能直接使用,需要使用其子类.

NSInvocationOperation(调用) 和 NSBlockOperation(块);



**NSOperation Queue**

存放NSOperation的集合类。不能说队列，不是严格的先进先出



NSOperation提供的方便操作

- 最大并发数
- 队列的暂停和继续
- 取消所有的操作
- 指定操作之间的依赖关系依赖关系，可以让异步任务同步执行.
- 将KVO用于NSOperation中，监听一个operation是否完成。
- 能够设置NSOperation的优先级，能够使同一个并行队列中的任务区分先后地执行
- 对NSOperation进行继承，在这之上添加成员变量与成员方法，提高整个代码的复用度





http tcp/ip socket 



http是应用层的协议，tcp是传输层的协议，ip是网络层的协议。

http:超文本传输协议，是应用层的协议，基于tcp的短连接。其特点是客户端发送的每次请求都需要服务器回送响应，在请求结束后，释放连接。



请求



请求行:请求的方法,请求的URI，HTTP版本号

请求头：

Host: 目标服务器的网络地址

Accept: 让服务端知道客户端所能接收的数据类型

Content-Type: body中的数据类型

Accept-Language: 客户端的语言环境

Accept-Encoding:客户端支持的数据压缩格式

User-Agent:客户端的软件环境

Connection: keep-alive（用来告诉服务端这是一个持久连接）

Content-Length:body的长度

Cookie:用户登录信息

请求体:

真正需要发给服务端的数据

响应的状态行:包含HTTP版本号、状态码、状态码对应的英文名

响应头

响应实体

响应:s

一、什么是TCP连接的三次握手

　　第一次握手：客户端发送syn包(syn=j)到服务器，并进入SYN_SEND状态，等待服务器确认;

　　第二次握手：服务器收到syn包，必须确认客户的SYN(ack=j+1)，同时自己也发送一个SYN包(syn=k)，即SYN+ACK包，此时服务器进入SYN_RECV状态;

　　第三次握手：客户端收到服务器的SYN+ACK包，向服务器发送确认包ACK(ack=k+1)，此包发送完毕，客户端和服务器进入ESTABLISHED状态，完成三次握手。

　　握手过程中传送的包里不包含数据，三次握手完毕后，客户端与服务器才正式开始传送数据。

　　理想状态下，TCP连接一旦建立，在通信双方中的任何一方主动关闭连接之前，TCP 连接都将被一直保持下去。

　　断开连接时服务器和客户端均可以主动发起断开TCP连接的请求，断开过程需要经过“四次握手”(过程就不细写了，就是服务器和客户端交互，最终确定断开)

　　二、利用Socket建立网络连接的步骤

socket层只是在TCP/UDP传输层上做的一个抽象接口层

　　建立Socket连接至少需要一对套接字，其中一个运行于客户端，称为ClientSocket ，另一个运行于服务器端，称为ServerSocket 。

　　套接字之间的连接过程分为三个步骤：服务器监听，客户端请求，连接确认。

　　1、服务器监听：服务器端套接字并不定位具体的客户端套接字，而是处于等待连接的状态，实时监控网络状态，等待客户端的连接请求。

　　2、客户端请求：指客户端的套接字提出连接请求，要连接的目标是服务器端的套接字。

　　为此，客户端的套接字必须首先描述它要连接的服务器的套接字，指出服务器端套接字的地址和端口号，然后就向服务器端套接字提出连接请求。

　　3、连接确认：当服务器端套接字监听到或者说接收到客户端套接字的连接请求时，就响应客户端套接字的请求，建立一个新的线程，把服务器端套接字的描述发给客户端，一旦客户端确认了此描述，双方就正式建立连接。



//什么是runloop？

  //1 runloop可以理解为一个对象，它管理它所要处理的所有事件和消息

  //2 其内部可以简单理解为一个do while 循环，直到接到退出的消息

  //3 每一个线程都对应一个runloop

  //4 一个runloop包含若干个mode,一个mode若干个source,observe,timer

   