---
layout: post
title: NSRunLoop
date: 2016-02-15 15:32:24.000000000 +09:00
---

# 前言



不知道大家有没有想过这个问题，一个应用开始运行以后放在那里，如果不对它进行任何操作，这个应用就像静止了一样，不会自发的有任何动作发生，但是如果我们点击界面上的一个按钮，这个时候就会有对应的按钮响应事件发生。给我们的感觉就像应用一直处于随时待命的状态，在没人操作的时候它一直在休息，在让它干活的时候，它就能立刻响应。其实，这就是run loop的功劳。

本篇文章很有参考价值，因此转载到本博客中，希望好文章对大家都有所帮助！

# 一、线程与run loop

## 1.1 线程任务的类型

再来说说线程。有些线程执行的任务是一条直线，起点到终点；而另一些线程要干的活则是一个圆，不断循环，直到通过某种方式将它终止。直线线程如简单的Hello World，运行打印完,它的生命周期便结束了，像昙花一现那样；圆类型的如操作系统，一直运行直到你关机。在IOS中，圆型的线程就是通过run loop不停的循环实现的。

## 1.2 线程与run loop的关系

Run loop，正如其名，loop表示某种循环，和run放在一起就表示一直在运行着的循环。实际上，run loop和线程是紧密相连的，可以这样说run loop是为了线程而生，没有线程，它就没有存在的必要。Run loops是线程的基础架构部分，Cocoa和CoreFundation都提供了run loop对象方便配置和管理线程的run loop（以下都已Cocoa为例）。每个线程，包括程序的主线程（main thread）都有与之相应的run loop对象。

### 1.2.1 主线程的run loop默认是启动的。

iOS的应用程序里面，程序启动后会有一个如下的main()函数：

<div id="crayon-56e5af776a8ec046726454" class="crayon-syntax crayon-theme-familiar-copy crayon-font-consolas crayon-os-mac print-yes notranslate" data-settings=" minimize scroll-always disable-anim" style=" margin-top: 12px; margin-bottom: 12px; margin-right: 24px; font-size: 15px !important; line-height: 20px !important;">

<div class="crayon-main" style="">

<table class="crayon-table">

<tbody>

<tr class="crayon-row">

<td class="crayon-nums " data-settings="show">

<div class="crayon-nums-content" style="font-size: 15px !important; line-height: 20px !important;">

<div class="crayon-num" data-line="crayon-56e5af776a8ec046726454-1">1</div>

<div class="crayon-num crayon-striped-num" data-line="crayon-56e5af776a8ec046726454-2">2</div>

<div class="crayon-num" data-line="crayon-56e5af776a8ec046726454-3">3</div>

<div class="crayon-num crayon-striped-num" data-line="crayon-56e5af776a8ec046726454-4">4</div>

<div class="crayon-num" data-line="crayon-56e5af776a8ec046726454-5">5</div>

<div class="crayon-num crayon-striped-num" data-line="crayon-56e5af776a8ec046726454-6">6</div>

<div class="crayon-num" data-line="crayon-56e5af776a8ec046726454-7">7</div>

</div>

</td>

<td class="crayon-code">

<div class="crayon-pre" style="font-size: 15px !important; line-height: 20px !important; -moz-tab-size:4; -o-tab-size:4; -webkit-tab-size:4; tab-size:4;">

<div class="crayon-line crayon-striped-line" id="crayon-56e5af776a8ec046726454-2"><span class="crayon-t">int</span> <span class="crayon-h"></span> <span class="crayon-e">main</span><span class="crayon-sy">(</span><span class="crayon-t">int</span> <span class="crayon-h"></span> <span class="crayon-v">argc</span><span class="crayon-sy">,</span><span class="crayon-t">char</span> <span class="crayon-h"></span> <span class="crayon-v">*argv</span><span class="crayon-sy">[</span><span class="crayon-sy">]</span><span class="crayon-sy">)</span> <span class="crayon-h"></span> <span class="crayon-sy">{</span></div>

<div class="crayon-line" id="crayon-56e5af776a8ec046726454-3"><span class="crayon-h">    </span><span class="crayon-sy">@</span><span class="crayon-v">autoreleasepool</span> <span class="crayon-h"></span> <span class="crayon-sy">{</span></div>

<div class="crayon-line crayon-striped-line" id="crayon-56e5af776a8ec046726454-4"><span class="crayon-h">      </span><span class="crayon-st">return</span> <span class="crayon-h"></span> <span class="crayon-t">UIApplicationMain</span><span class="crayon-sy">(</span><span class="crayon-v">argc</span><span class="crayon-sy">,</span> <span class="crayon-h"></span> <span class="crayon-v">argv</span><span class="crayon-sy">,</span> <span class="crayon-h"></span> <span class="crayon-t">nil</span><span class="crayon-sy">,</span> <span class="crayon-h"></span> <span class="crayon-t">NSStringFromClass</span><span class="crayon-sy">(</span><span class="crayon-sy">[</span><span class="crayon-e">appDelegate</span> <span class="crayon-t">class</span><span class="crayon-sy">]</span><span class="crayon-sy">)</span><span class="crayon-sy">)</span><span class="crayon-sy">;</span></div>

<div class="crayon-line" id="crayon-56e5af776a8ec046726454-5"><span class="crayon-h">  </span> <span class="crayon-sy">}</span></div>

<div class="crayon-line crayon-striped-line" id="crayon-56e5af776a8ec046726454-6"><span class="crayon-sy">}</span></div>

</div>

</td>

</tr>

</tbody>

</table>

</div>

</div>

重点是UIApplicationMain()函数，这个方法会为main thread设置一个NSRunLoop对象，这就解释了本文开始说的为什么我们的应用可以在无人操作的时候休息，需要让它干活的时候又能立马响应。

<div><script type="text/javascript">var cpro_id = "u2513605";</script></div>

### 1.2.2 对其它线程来说，run loop默认是没有启动的

对其它线程来说，run loop默认是没有启动的，如果你需要更多的线程交互则可以手动配置和启动，如果线程只是去执行一个长时间的已确定的任务则不需要。

### 1.2.3 获取线程的run loop

在任何一个Cocoa程序的线程中，都可以通过：

<div id="crayon-56e5af776a8fa822219774" class="crayon-syntax crayon-theme-familiar-copy crayon-font-consolas crayon-os-mac print-yes notranslate" data-settings=" minimize scroll-always disable-anim" style=" margin-top: 12px; margin-bottom: 12px; margin-right: 24px; font-size: 15px !important; line-height: 20px !important;">

<div class="crayon-main" style="">

<table class="crayon-table">

<tbody>

<tr class="crayon-row">

<td class="crayon-nums " data-settings="show">

<div class="crayon-nums-content" style="font-size: 15px !important; line-height: 20px !important;">

<div class="crayon-num" data-line="crayon-56e5af776a8fa822219774-1">1</div>

<div class="crayon-num crayon-striped-num" data-line="crayon-56e5af776a8fa822219774-2">2</div>

<div class="crayon-num" data-line="crayon-56e5af776a8fa822219774-3">3</div>

</div>

</td>

<td class="crayon-code">

<div class="crayon-pre" style="font-size: 15px !important; line-height: 20px !important; -moz-tab-size:4; -o-tab-size:4; -webkit-tab-size:4; tab-size:4;">

<div class="crayon-line crayon-striped-line" id="crayon-56e5af776a8fa822219774-2"><span class="crayon-t">NSRunLoop</span><span class="crayon-h">  </span> <span class="crayon-v">*runloop</span> <span class="crayon-h"></span> <span class="crayon-o">=</span> <span class="crayon-h"></span> <span class="crayon-sy">[</span><span class="crayon-t">NSRunLoopcurrentRunLoop</span><span class="crayon-sy">]</span><span class="crayon-sy">;</span></div>

</div>

</td>

</tr>

</tbody>

</table>

</div>

</div>

来获取到当前线程的run loop。

## 1.3 关于run loop的几点说明

### 1.3.1 Cocoa中的NSRunLoop类并不是线程安全的

我们不能再一个线程中去操作另外一个线程的run loop对象，那很可能会造成意想不到的后果。不过幸运的是CoreFundation中的不透明类CFRunLoopRef是线程安全的，而且两种类型的run loop完全可以混合使用。Cocoa中的NSRunLoop类可以通过实例方法：

<div id="crayon-56e5af776a900837648371" class="crayon-syntax crayon-theme-familiar-copy crayon-font-consolas crayon-os-mac print-yes notranslate" data-settings=" minimize scroll-always disable-anim" style=" margin-top: 12px; margin-bottom: 12px; margin-right: 24px; font-size: 15px !important; line-height: 20px !important;">

<div class="crayon-main" style="">

<table class="crayon-table">

<tbody>

<tr class="crayon-row">

<td class="crayon-nums " data-settings="show">

<div class="crayon-nums-content" style="font-size: 15px !important; line-height: 20px !important;">

<div class="crayon-num" data-line="crayon-56e5af776a900837648371-1">1</div>

<div class="crayon-num crayon-striped-num" data-line="crayon-56e5af776a900837648371-2">2</div>

<div class="crayon-num" data-line="crayon-56e5af776a900837648371-3">3</div>

</div>

</td>

<td class="crayon-code">

<div class="crayon-pre" style="font-size: 15px !important; line-height: 20px !important; -moz-tab-size:4; -o-tab-size:4; -webkit-tab-size:4; tab-size:4;">

<div class="crayon-line crayon-striped-line" id="crayon-56e5af776a900837648371-2"><span class="crayon-o">-</span> <span class="crayon-h"></span> <span class="crayon-sy">(</span><span class="crayon-t">CFRunLoopRef</span><span class="crayon-sy">)</span><span class="crayon-v">getCFRunLoop</span><span class="crayon-sy">;</span></div>

</div>

</td>

</tr>

</tbody>

</table>

</div>

</div>

获取对应的CFRunLoopRef类，来达到线程安全的目的。

### 1.3.2 Run loop的管理并不完全是自动的。

我们仍必须设计线程代码以在适当的时候启动run loop并正确响应输入事件，当然前提是线程中需要用到run loop。而且，我们还需要使用while/for语句来驱动run loop能够循环运行，下面的代码就成功驱动了一个run loop：

<div id="crayon-56e5af776a905374496234" class="crayon-syntax crayon-theme-familiar-copy crayon-font-consolas crayon-os-mac print-yes notranslate" data-settings=" minimize scroll-always disable-anim" style=" margin-top: 12px; margin-bottom: 12px; margin-right: 24px; font-size: 15px !important; line-height: 20px !important;">

<div class="crayon-main" style="">

<table class="crayon-table">

<tbody>

<tr class="crayon-row">

<td class="crayon-nums " data-settings="show">

<div class="crayon-nums-content" style="font-size: 15px !important; line-height: 20px !important;">

<div class="crayon-num" data-line="crayon-56e5af776a905374496234-1">1</div>

<div class="crayon-num crayon-striped-num" data-line="crayon-56e5af776a905374496234-2">2</div>

<div class="crayon-num" data-line="crayon-56e5af776a905374496234-3">3</div>

<div class="crayon-num crayon-striped-num" data-line="crayon-56e5af776a905374496234-4">4</div>

<div class="crayon-num" data-line="crayon-56e5af776a905374496234-5">5</div>

<div class="crayon-num crayon-striped-num" data-line="crayon-56e5af776a905374496234-6">6</div>

<div class="crayon-num" data-line="crayon-56e5af776a905374496234-7">7</div>

</div>

</td>

<td class="crayon-code">

<div class="crayon-pre" style="font-size: 15px !important; line-height: 20px !important; -moz-tab-size:4; -o-tab-size:4; -webkit-tab-size:4; tab-size:4;">

<div class="crayon-line crayon-striped-line" id="crayon-56e5af776a905374496234-2"><span class="crayon-t">BOOL</span> <span class="crayon-h"></span> <span class="crayon-v">isRunning</span> <span class="crayon-h"></span> <span class="crayon-o">=</span> <span class="crayon-h"></span> <span class="crayon-t">NO</span><span class="crayon-sy">;</span></div>

<div class="crayon-line crayon-striped-line" id="crayon-56e5af776a905374496234-4"><span class="crayon-st">do</span><span class="crayon-h">  </span><span class="crayon-sy">{</span></div>

<div class="crayon-line" id="crayon-56e5af776a905374496234-5"><span class="crayon-h">    </span><span class="crayon-v">isRunning</span> <span class="crayon-h"></span> <span class="crayon-o">=</span> <span class="crayon-h"></span> <span class="crayon-sy">[</span><span class="crayon-sy">[</span><span class="crayon-t">NSRunLoopcurrentRunLoop</span><span class="crayon-sy">]</span> <span class="crayon-e ">runMode</span><span class="crayon-v">:NSDefaultRunLoopModebeforeDate</span><span class="crayon-o">:</span><span class="crayon-sy">[</span><span class="crayon-t">NSDatedistantFuture</span><span class="crayon-sy">]</span><span class="crayon-sy">]</span><span class="crayon-sy">;</span></div>

<div class="crayon-line crayon-striped-line" id="crayon-56e5af776a905374496234-6"><span class="crayon-sy">}</span> <span class="crayon-h"></span> <span class="crayon-st">while</span> <span class="crayon-h"></span> <span class="crayon-sy">(</span><span class="crayon-v">isRunning</span><span class="crayon-sy">)</span><span class="crayon-sy">;</span></div>

</div>

</td>

</tr>

</tbody>

</table>

</div>

</div>

### 1.3.3 Run loop同时也负责autorelease pool的创建和释放

在使用手动的内存管理方式的项目中，会经常用到很多自动释放的对象，如果这些对象不能够被即时释放掉，会造成内存占用量急剧增大。Run loop就为我们做了这样的工作，每当一个运行循环结束的时候，它都会释放一次autorelease pool，同时pool中的所有自动释放类型变量都会被释放掉。

### 1.3.4 Run loop的优点

一个run loop就是一个事件处理循环，用来不停的监听和处理输入事件并将其分配到对应的目标上进行处理。如果仅仅是想实现这个功能，你可能会想一个简单的while循环不就可以实现了吗，用得着费老大劲来做个那么复杂的机制？显然，苹果的架构设计师不是吃干饭的，你想到的他们早就想过了。

首先，NSRunLoop是一种更加高明的消息处理模式，他就高明在对消息处理过程进行了更好的抽象和封装，这样才能是的你不用处理一些很琐碎很低层次的具体消息的处理，在NSRunLoop中每一个消息就被打包在input source或者是timer source（见后文）中了。

其次，也是很重要的一点，使用run loop可以使你的线程在有工作的时候工作，没有工作的时候休眠，这可以大大节省系统资源。

# 二、Run loop相关知识点

## 2.1输入事件来源

Run loop接收输入事件来自两种不同的来源：输入源（input source）和定时源（timer source）。两种源都使用程序的某一特定的处理例程来处理到达的事件。图-1显示了run loop的概念结构以及各种源。

需要说明的是，当你创建输入源，你需要将其分配给run loop中的一个或多个模式（什么是模式，下文将会讲到）。模式只会在特定事件影响监听的源。大多数情况下，run loop运行在默认模式下，但是你也可以使其运行在自定义模式。若某一源在当前模式下不被监听，那么任何其生成的消息只在run loop运行在其关联的模式下才会被传递。

![image](http://www.henishuo.com/wp-content/uploads/2016/02/20130703215237531.png)

图-1 Runloop的结构和输入源类型

### 2.1.1输入源（input source）

传递异步事件，通常消息来自于其他线程或程序。输入源传递异步消息给相应的处理例程，并调用runUntilDate:方法来退出(在线程里面相关的NSRunLoop对象调用)。

#### 2.1.1.1基于端口的输入源

基于端口的输入源由内核自动发送。

Cocoa和Core Foundation内置支持使用端口相关的对象和函数来创建的基于端口的源。例如，在Cocoa里面你从来不需要直接创建输入源。你只要简单的创建端口对象，并使用NSPort的方法把该端口添加到run loop。端口对象会自己处理创建和配置输入源。

在Core Foundation，你必须人工创建端口和它的run loop源。我们可以使用端口相关的函数（CFMachPortRef，CFMessagePortRef，CFSocketRef）来创建合适的对象。下面的例子展示了如何创建一个基于端口的输入源，将其添加到run loop并启动：

<div id="crayon-56e5af776a90d431605583" class="crayon-syntax crayon-theme-familiar-copy crayon-font-consolas crayon-os-mac print-yes notranslate" data-settings=" minimize scroll-always disable-anim" style=" margin-top: 12px; margin-bottom: 12px; margin-right: 24px; font-size: 15px !important; line-height: 20px !important;">

<div class="crayon-main" style="">

<table class="crayon-table">

<tbody>

<tr class="crayon-row">

<td class="crayon-nums " data-settings="show">

<div class="crayon-nums-content" style="font-size: 15px !important; line-height: 20px !important;">

<div class="crayon-num" data-line="crayon-56e5af776a90d431605583-1">1</div>

<div class="crayon-num crayon-striped-num" data-line="crayon-56e5af776a90d431605583-2">2</div>

<div class="crayon-num" data-line="crayon-56e5af776a90d431605583-3">3</div>

<div class="crayon-num crayon-striped-num" data-line="crayon-56e5af776a90d431605583-4">4</div>

<div class="crayon-num" data-line="crayon-56e5af776a90d431605583-5">5</div>

<div class="crayon-num crayon-striped-num" data-line="crayon-56e5af776a90d431605583-6">6</div>

<div class="crayon-num" data-line="crayon-56e5af776a90d431605583-7">7</div>

<div class="crayon-num crayon-striped-num" data-line="crayon-56e5af776a90d431605583-8">8</div>

<div class="crayon-num" data-line="crayon-56e5af776a90d431605583-9">9</div>

<div class="crayon-num crayon-striped-num" data-line="crayon-56e5af776a90d431605583-10">10</div>

<div class="crayon-num" data-line="crayon-56e5af776a90d431605583-11">11</div>

<div class="crayon-num crayon-striped-num" data-line="crayon-56e5af776a90d431605583-12">12</div>

<div class="crayon-num" data-line="crayon-56e5af776a90d431605583-13">13</div>

<div class="crayon-num crayon-striped-num" data-line="crayon-56e5af776a90d431605583-14">14</div>

<div class="crayon-num" data-line="crayon-56e5af776a90d431605583-15">15</div>

<div class="crayon-num crayon-striped-num" data-line="crayon-56e5af776a90d431605583-16">16</div>

<div class="crayon-num" data-line="crayon-56e5af776a90d431605583-17">17</div>

<div class="crayon-num crayon-striped-num" data-line="crayon-56e5af776a90d431605583-18">18</div>

<div class="crayon-num" data-line="crayon-56e5af776a90d431605583-19">19</div>

</div>

</td>

<td class="crayon-code">

<div class="crayon-pre" style="font-size: 15px !important; line-height: 20px !important; -moz-tab-size:4; -o-tab-size:4; -webkit-tab-size:4; tab-size:4;">

<div class="crayon-line crayon-striped-line" id="crayon-56e5af776a90d431605583-2"><span class="crayon-e">voidcreatePortSource</span><span class="crayon-sy">(</span><span class="crayon-sy">)</span> <span class="crayon-h"></span> <span class="crayon-sy">{</span></div>

<div class="crayon-line" id="crayon-56e5af776a90d431605583-3"><span class="crayon-h">    </span><span class="crayon-t">CFMessagePortRef</span> <span class="crayon-h"></span> <span class="crayon-v">port</span> <span class="crayon-h"></span> <span class="crayon-o">=</span> <span class="crayon-h"></span> <span class="crayon-t">CFMessagePortCreateLocal</span><span class="crayon-sy">(</span><span class="crayon-v">kCFAllocatorDefault</span><span class="crayon-sy">,</span><span class="crayon-t">CFSTR</span><span class="crayon-sy">(</span><span class="crayon-s">"com.someport"</span><span class="crayon-sy">)</span><span class="crayon-sy">,</span><span class="crayon-v">myCallbackFunc</span><span class="crayon-sy">,</span> <span class="crayon-h"></span> <span class="crayon-t">NULL</span><span class="crayon-sy">,</span><span class="crayon-t">NULL</span><span class="crayon-sy">)</span><span class="crayon-sy">;</span></div>

<div class="crayon-line" id="crayon-56e5af776a90d431605583-5"><span class="crayon-h">    </span><span class="crayon-t">CFRunLoopSourceRef</span> <span class="crayon-h"></span> <span class="crayon-v">source</span> <span class="crayon-h"></span> <span class="crayon-o">=</span> <span class="crayon-h"></span> <span class="crayon-t">CFMessagePortCreateRunLoopSource</span><span class="crayon-sy">(</span><span class="crayon-v">kCFAllocatorDefault</span><span class="crayon-sy">,</span> <span class="crayon-h"></span> <span class="crayon-v">port</span><span class="crayon-sy">,</span><span class="crayon-cn">0</span><span class="crayon-sy">)</span><span class="crayon-sy">;</span></div>

<div class="crayon-line" id="crayon-56e5af776a90d431605583-7"><span class="crayon-h">    </span><span class="crayon-t">CFRunLoopAddSource</span><span class="crayon-sy">(</span><span class="crayon-t">CFRunLoopGetCurrent</span><span class="crayon-sy">(</span><span class="crayon-sy">)</span><span class="crayon-sy">,</span> <span class="crayon-h"></span> <span class="crayon-v">source</span><span class="crayon-sy">,</span><span class="crayon-v">kCFRunLoopCommonModes</span><span class="crayon-sy">)</span><span class="crayon-sy">;</span></div>

<div class="crayon-line" id="crayon-56e5af776a90d431605583-9"><span class="crayon-h">    </span><span class="crayon-st">while</span> <span class="crayon-h"></span> <span class="crayon-sy">(</span><span class="crayon-v">pageStillLoading</span><span class="crayon-sy">)</span> <span class="crayon-h"></span> <span class="crayon-sy">{</span></div>

<div class="crayon-line crayon-striped-line" id="crayon-56e5af776a90d431605583-10"><span class="crayon-h">        </span><span class="crayon-t">NSAutoreleasePool</span> <span class="crayon-h"></span> <span class="crayon-v">*pool</span> <span class="crayon-h"></span> <span class="crayon-o">=</span> <span class="crayon-h"></span> <span class="crayon-sy">[</span><span class="crayon-sy">[</span><span class="crayon-t">NSAutoreleasePoolalloc</span><span class="crayon-sy">]</span> <span class="crayon-e ">init</span><span class="crayon-sy">]</span><span class="crayon-sy">;</span></div>

<div class="crayon-line" id="crayon-56e5af776a90d431605583-11"><span class="crayon-h">        </span><span class="crayon-t">CFRunLoopRun</span><span class="crayon-sy">(</span><span class="crayon-sy">)</span><span class="crayon-sy">;</span></div>

<div class="crayon-line crayon-striped-line" id="crayon-56e5af776a90d431605583-12"><span class="crayon-h">        </span><span class="crayon-sy">[</span><span class="crayon-e">pool</span> <span class="crayon-r">release</span><span class="crayon-sy">]</span><span class="crayon-sy">;</span></div>

<div class="crayon-line" id="crayon-56e5af776a90d431605583-13"><span class="crayon-h">    </span><span class="crayon-sy">}</span></div>

<div class="crayon-line" id="crayon-56e5af776a90d431605583-15"><span class="crayon-h">    </span><span class="crayon-t">CFRunLoopRemoveSource</span><span class="crayon-sy">(</span><span class="crayon-t">CFRunLoopGetCurrent</span><span class="crayon-sy">(</span><span class="crayon-sy">)</span><span class="crayon-sy">,</span> <span class="crayon-h"></span> <span class="crayon-v">source</span><span class="crayon-sy">,</span><span class="crayon-v">kCFRunLoopDefaultMode</span><span class="crayon-sy">)</span><span class="crayon-sy">;</span></div>

<div class="crayon-line" id="crayon-56e5af776a90d431605583-17"><span class="crayon-h">    </span><span class="crayon-t">CFRelease</span><span class="crayon-sy">(</span><span class="crayon-v">source</span><span class="crayon-sy">)</span><span class="crayon-sy">;</span></div>

<div class="crayon-line crayon-striped-line" id="crayon-56e5af776a90d431605583-18"><span class="crayon-sy">}</span></div>

</div>

</td>

</tr>

</tbody>

</table>

</div>

</div>

#### 2.1.1.2自定义输入源

自定义的输入源需要人工从其他线程发送。

为了创建自定义输入源，必须使用Core Foundation里面的CFRunLoopSourceRef类型相关的函数来创建。你可以使用回调函数来配置自定义输入源。Core Fundation会在配置源的不同地方调用回调函数，处理输入事件，在源从run loop移除的时候清理它。

除了定义在事件到达时自定义输入源的行为，你也必须定义消息传递机制。源的这部分运行在单独的线程里面，并负责在数据等待处理的时候传递数据给源并通知它处理数据。消息传递机制的定义取决于你，但最好不要过于复杂。创建并启动自定义输入源的示例如下：

<div id="crayon-56e5af776a913994535262" class="crayon-syntax crayon-theme-familiar-copy crayon-font-consolas crayon-os-mac print-yes notranslate" data-settings=" minimize scroll-always disable-anim" style=" margin-top: 12px; margin-bottom: 12px; margin-right: 24px; font-size: 15px !important; line-height: 20px !important;">

<div class="crayon-main" style="">

<table class="crayon-table">

<tbody>

<tr class="crayon-row">

<td class="crayon-nums " data-settings="show">

<div class="crayon-nums-content" style="font-size: 15px !important; line-height: 20px !important;">

<div class="crayon-num" data-line="crayon-56e5af776a913994535262-1">1</div>

<div class="crayon-num crayon-striped-num" data-line="crayon-56e5af776a913994535262-2">2</div>

<div class="crayon-num" data-line="crayon-56e5af776a913994535262-3">3</div>

<div class="crayon-num crayon-striped-num" data-line="crayon-56e5af776a913994535262-4">4</div>

<div class="crayon-num" data-line="crayon-56e5af776a913994535262-5">5</div>

<div class="crayon-num crayon-striped-num" data-line="crayon-56e5af776a913994535262-6">6</div>

<div class="crayon-num" data-line="crayon-56e5af776a913994535262-7">7</div>

<div class="crayon-num crayon-striped-num" data-line="crayon-56e5af776a913994535262-8">8</div>

<div class="crayon-num" data-line="crayon-56e5af776a913994535262-9">9</div>

<div class="crayon-num crayon-striped-num" data-line="crayon-56e5af776a913994535262-10">10</div>

<div class="crayon-num" data-line="crayon-56e5af776a913994535262-11">11</div>

<div class="crayon-num crayon-striped-num" data-line="crayon-56e5af776a913994535262-12">12</div>

<div class="crayon-num" data-line="crayon-56e5af776a913994535262-13">13</div>

<div class="crayon-num crayon-striped-num" data-line="crayon-56e5af776a913994535262-14">14</div>

<div class="crayon-num" data-line="crayon-56e5af776a913994535262-15">15</div>

<div class="crayon-num crayon-striped-num" data-line="crayon-56e5af776a913994535262-16">16</div>

<div class="crayon-num" data-line="crayon-56e5af776a913994535262-17">17</div>

<div class="crayon-num crayon-striped-num" data-line="crayon-56e5af776a913994535262-18">18</div>

<div class="crayon-num" data-line="crayon-56e5af776a913994535262-19">19</div>

</div>

</td>

<td class="crayon-code">

<div class="crayon-pre" style="font-size: 15px !important; line-height: 20px !important; -moz-tab-size:4; -o-tab-size:4; -webkit-tab-size:4; tab-size:4;">

<div class="crayon-line crayon-striped-line" id="crayon-56e5af776a913994535262-2"><span class="crayon-e">voidcreateCustomSource</span><span class="crayon-sy">(</span><span class="crayon-sy">)</span> <span class="crayon-h"></span> <span class="crayon-sy">{</span></div>

<div class="crayon-line" id="crayon-56e5af776a913994535262-3"><span class="crayon-h">    </span><span class="crayon-t">CFRunLoopSourceContext</span> <span class="crayon-h"></span> <span class="crayon-v">context</span> <span class="crayon-h"></span> <span class="crayon-o">=</span> <span class="crayon-h"></span> <span class="crayon-sy">{</span><span class="crayon-cn">0</span><span class="crayon-sy">,</span><span class="crayon-t">NULL</span><span class="crayon-sy">,</span> <span class="crayon-h"></span> <span class="crayon-t">NULL</span><span class="crayon-sy">,</span><span class="crayon-t">NULL</span><span class="crayon-sy">,</span> <span class="crayon-h"></span> <span class="crayon-t">NULL</span><span class="crayon-sy">,</span><span class="crayon-t">NULL</span><span class="crayon-sy">,</span> <span class="crayon-h"></span> <span class="crayon-t">NULL</span><span class="crayon-sy">,</span><span class="crayon-t">NULL</span><span class="crayon-sy">,</span> <span class="crayon-h"></span> <span class="crayon-t">NULL</span><span class="crayon-sy">,</span><span class="crayon-t">NULL</span><span class="crayon-sy">}</span><span class="crayon-sy">;</span></div>

<div class="crayon-line" id="crayon-56e5af776a913994535262-5"><span class="crayon-h">    </span><span class="crayon-t">CFRunLoopSourceRef</span> <span class="crayon-h"></span> <span class="crayon-v">source</span> <span class="crayon-h"></span> <span class="crayon-o">=</span><span class="crayon-t">CFRunLoopSourceCreate</span><span class="crayon-sy">(</span><span class="crayon-v">kCFAllocatorDefault</span><span class="crayon-sy">,</span><span class="crayon-cn">0</span><span class="crayon-sy">,</span> <span class="crayon-h"></span> <span class="crayon-o">&</span><span class="crayon-v">context</span><span class="crayon-sy">)</span><span class="crayon-sy">;</span></div>

<div class="crayon-line" id="crayon-56e5af776a913994535262-7"><span class="crayon-h">    </span><span class="crayon-t">CFRunLoopAddSource</span><span class="crayon-sy">(</span><span class="crayon-t">CFRunLoopGetCurrent</span><span class="crayon-sy">(</span><span class="crayon-sy">)</span><span class="crayon-sy">,</span> <span class="crayon-h"></span> <span class="crayon-v">source</span><span class="crayon-sy">,</span><span class="crayon-v">kCFRunLoopDefaultMode</span><span class="crayon-sy">)</span><span class="crayon-sy">;</span></div>

<div class="crayon-line" id="crayon-56e5af776a913994535262-9"><span class="crayon-h">    </span><span class="crayon-st">while</span> <span class="crayon-h"></span> <span class="crayon-sy">(</span><span class="crayon-v">pageStillLoading</span><span class="crayon-sy">)</span> <span class="crayon-h"></span> <span class="crayon-sy">{</span></div>

<div class="crayon-line crayon-striped-line" id="crayon-56e5af776a913994535262-10"><span class="crayon-h">        </span><span class="crayon-t">NSAutoreleasePool</span> <span class="crayon-h"></span> <span class="crayon-v">*pool</span> <span class="crayon-h"></span> <span class="crayon-o">=</span> <span class="crayon-h"></span> <span class="crayon-sy">[</span><span class="crayon-sy">[</span><span class="crayon-t">NSAutoreleasePoolalloc</span><span class="crayon-sy">]</span> <span class="crayon-e ">init</span><span class="crayon-sy">]</span><span class="crayon-sy">;</span></div>

<div class="crayon-line" id="crayon-56e5af776a913994535262-11"><span class="crayon-h">        </span><span class="crayon-t">CFRunLoopRun</span><span class="crayon-sy">(</span><span class="crayon-sy">)</span><span class="crayon-sy">;</span></div>

<div class="crayon-line crayon-striped-line" id="crayon-56e5af776a913994535262-12"><span class="crayon-h">        </span><span class="crayon-sy">[</span><span class="crayon-e">pool</span> <span class="crayon-r">release</span><span class="crayon-sy">]</span><span class="crayon-sy">;</span></div>

<div class="crayon-line" id="crayon-56e5af776a913994535262-13"><span class="crayon-h">    </span><span class="crayon-sy">}</span></div>

<div class="crayon-line" id="crayon-56e5af776a913994535262-15"><span class="crayon-h">    </span><span class="crayon-t">CFRunLoopRemoveSource</span><span class="crayon-sy">(</span><span class="crayon-t">CFRunLoopGetCurrent</span><span class="crayon-sy">(</span><span class="crayon-sy">)</span><span class="crayon-sy">,</span> <span class="crayon-h"></span> <span class="crayon-v">source</span><span class="crayon-sy">,</span><span class="crayon-v">kCFRunLoopDefaultMode</span><span class="crayon-sy">)</span><span class="crayon-sy">;</span></div>

<div class="crayon-line" id="crayon-56e5af776a913994535262-17"><span class="crayon-h">    </span><span class="crayon-t">CFRelease</span><span class="crayon-sy">(</span><span class="crayon-v">source</span><span class="crayon-sy">)</span><span class="crayon-sy">;</span></div>

<div class="crayon-line crayon-striped-line" id="crayon-56e5af776a913994535262-18"><span class="crayon-sy">}</span></div>

</div>

</td>

</tr>

</tbody>

</table>

</div>

</div>

#### 2.1.1.3 Cocoa上的Selector源

除了基于端口的源，Cocoa定义了自定义输入源，允许你在任何线程执行selector方法。和基于端口的源一样，执行selector请求会在目标线程上序列化，减缓许多在线程上允许多个方法容易引起的同步问题。不像基于端口的源，一个selector执行完后会自动从run loop里面移除。

当在其他线程上面执行selector时，目标线程须有一个活动的run loop。对于你创建的线程，这意味着线程在你显式的启动run loop之前是不会执行selector方法的，而是一直处于休眠状态。

NSObject类提供了类似如下的selector方法：

<div id="crayon-56e5af776a919501908720" class="crayon-syntax crayon-theme-familiar-copy crayon-font-consolas crayon-os-mac print-yes notranslate" data-settings=" minimize scroll-always disable-anim" style=" margin-top: 12px; margin-bottom: 12px; margin-right: 24px; font-size: 15px !important; line-height: 20px !important;">

<div class="crayon-main" style="">

<table class="crayon-table">

<tbody>

<tr class="crayon-row">

<td class="crayon-nums " data-settings="show">

<div class="crayon-nums-content" style="font-size: 15px !important; line-height: 20px !important;">

<div class="crayon-num" data-line="crayon-56e5af776a919501908720-1">1</div>

<div class="crayon-num crayon-striped-num" data-line="crayon-56e5af776a919501908720-2">2</div>

<div class="crayon-num" data-line="crayon-56e5af776a919501908720-3">3</div>

</div>

</td>

<td class="crayon-code">

<div class="crayon-pre" style="font-size: 15px !important; line-height: 20px !important; -moz-tab-size:4; -o-tab-size:4; -webkit-tab-size:4; tab-size:4;">

<div class="crayon-line crayon-striped-line" id="crayon-56e5af776a919501908720-2"><span class="crayon-o">-</span> <span class="crayon-h"></span> <span class="crayon-sy">(</span><span class="crayon-t">void</span><span class="crayon-sy">)</span><span class="crayon-e ">performSelectorOnMainThread</span><span class="crayon-o">:</span><span class="crayon-sy">(</span><span class="crayon-t">SEL</span><span class="crayon-sy">)</span><span class="crayon-v">aSelector</span> <span class="crayon-e ">withObject</span><span class="crayon-o">:</span><span class="crayon-sy">(</span><span class="crayon-t">id</span><span class="crayon-sy">)</span><span class="crayon-e ">argwaitUntilDone</span><span class="crayon-o">:</span><span class="crayon-sy">(</span><span class="crayon-t">BOOL</span><span class="crayon-sy">)</span><span class="crayon-v">wait</span> <span class="crayon-e ">modes</span><span class="crayon-o">:</span><span class="crayon-sy">(</span><span class="crayon-t">NSArray</span> <span class="crayon-h"></span> <span class="crayon-t ">*</span><span class="crayon-sy">)</span><span class="crayon-v">array</span><span class="crayon-sy">;</span></div>

</div>

</td>

</tr>

</tbody>

</table>

</div>

</div>

### 2.1.2 定时源（timer source）

定时源在预设的时间点同步方式传递消息，这些消息都会发生在特定时间或者重复的时间间隔。定时源则直接传递消息给处理例程，不会立即退出run loop。

需要注意的是，尽管定时器可以产生基于时间的通知，但它并不是实时机制。和输入源一样，定时器也和你的run loop的特定模式相关。如果定时器所在的模式当前未被run loop监视，那么定时器将不会开始直到run loop运行在相应的模式下。类似的，如果定时器在run loop处理某一事件期间开始，定时器会一直等待直到下次run loop开始相应的处理程序。如果run loop不再运行，那定时器也将永远不启动。

创建定时器源有两种方法，

方法一：

<div id="crayon-56e5af776a91e847963456" class="crayon-syntax crayon-theme-familiar-copy crayon-font-consolas crayon-os-mac print-yes notranslate" data-settings=" minimize scroll-always disable-anim" style=" margin-top: 12px; margin-bottom: 12px; margin-right: 24px; font-size: 15px !important; line-height: 20px !important;">

<div class="crayon-main" style="">

<table class="crayon-table">

<tbody>

<tr class="crayon-row">

<td class="crayon-nums " data-settings="show">

<div class="crayon-nums-content" style="font-size: 15px !important; line-height: 20px !important;">

<div class="crayon-num" data-line="crayon-56e5af776a91e847963456-1">1</div>

<div class="crayon-num crayon-striped-num" data-line="crayon-56e5af776a91e847963456-2">2</div>

<div class="crayon-num" data-line="crayon-56e5af776a91e847963456-3">3</div>

<div class="crayon-num crayon-striped-num" data-line="crayon-56e5af776a91e847963456-4">4</div>

<div class="crayon-num" data-line="crayon-56e5af776a91e847963456-5">5</div>

<div class="crayon-num crayon-striped-num" data-line="crayon-56e5af776a91e847963456-6">6</div>

<div class="crayon-num" data-line="crayon-56e5af776a91e847963456-7">7</div>

<div class="crayon-num crayon-striped-num" data-line="crayon-56e5af776a91e847963456-8">8</div>

<div class="crayon-num" data-line="crayon-56e5af776a91e847963456-9">9</div>

</div>

</td>

<td class="crayon-code">

<div class="crayon-pre" style="font-size: 15px !important; line-height: 20px !important; -moz-tab-size:4; -o-tab-size:4; -webkit-tab-size:4; tab-size:4;">

<div class="crayon-line crayon-striped-line" id="crayon-56e5af776a91e847963456-2"><span class="crayon-t">NSTimer</span> <span class="crayon-h"></span> <span class="crayon-v">*timer</span> <span class="crayon-h"></span> <span class="crayon-o">=</span> <span class="crayon-h"></span> <span class="crayon-sy">[</span><span class="crayon-t">NSTimer</span> <span class="crayon-e ">scheduledTimerWithTimeInterval</span><span class="crayon-o">:</span><span class="crayon-cn">4.0</span></div>

<div class="crayon-line" id="crayon-56e5af776a91e847963456-3"><span class="crayon-e ">                                                  target</span><span class="crayon-v">:self</span></div>

<div class="crayon-line crayon-striped-line" id="crayon-56e5af776a91e847963456-4"><span class="crayon-e ">                                                selector</span><span class="crayon-o">:</span><span class="crayon-st">@selector</span><span class="crayon-sy">(</span><span class="crayon-e ">backgroundThreadFire</span><span class="crayon-o">:</span><span class="crayon-sy">)</span></div>

<div class="crayon-line" id="crayon-56e5af776a91e847963456-5"><span class="crayon-e ">                                                userInfo</span><span class="crayon-v">:nil</span></div>

<div class="crayon-line crayon-striped-line" id="crayon-56e5af776a91e847963456-6"><span class="crayon-e ">                                                 repeats</span><span class="crayon-v">:YES</span><span class="crayon-sy">]</span><span class="crayon-sy">;</span></div>

<div class="crayon-line crayon-striped-line" id="crayon-56e5af776a91e847963456-8"><span class="crayon-sy">[</span><span class="crayon-sy">[</span><span class="crayon-t">NSRunLoop</span> <span class="crayon-h"></span> <span class="crayon-v">currentRunLoop</span><span class="crayon-sy">]</span> <span class="crayon-e ">addTimer</span><span class="crayon-v">:timerforMode</span><span class="crayon-v">:NSDefaultRunLoopMode</span><span class="crayon-sy">]</span><span class="crayon-sy">;</span></div>

</div>

</td>

</tr>

</tbody>

</table>

</div>

</div>

方法二：

<div id="crayon-56e5af776a922331614314" class="crayon-syntax crayon-theme-familiar-copy crayon-font-consolas crayon-os-mac print-yes notranslate" data-settings=" minimize scroll-always disable-anim" style=" margin-top: 12px; margin-bottom: 12px; margin-right: 24px; font-size: 15px !important; line-height: 20px !important;">

<div class="crayon-main" style="">

<table class="crayon-table">

<tbody>

<tr class="crayon-row">

<td class="crayon-nums " data-settings="show">

<div class="crayon-nums-content" style="font-size: 15px !important; line-height: 20px !important;">

<div class="crayon-num" data-line="crayon-56e5af776a922331614314-1">1</div>

<div class="crayon-num crayon-striped-num" data-line="crayon-56e5af776a922331614314-2">2</div>

<div class="crayon-num" data-line="crayon-56e5af776a922331614314-3">3</div>

<div class="crayon-num crayon-striped-num" data-line="crayon-56e5af776a922331614314-4">4</div>

<div class="crayon-num" data-line="crayon-56e5af776a922331614314-5">5</div>

<div class="crayon-num crayon-striped-num" data-line="crayon-56e5af776a922331614314-6">6</div>

<div class="crayon-num" data-line="crayon-56e5af776a922331614314-7">7</div>

</div>

</td>

<td class="crayon-code">

<div class="crayon-pre" style="font-size: 15px !important; line-height: 20px !important; -moz-tab-size:4; -o-tab-size:4; -webkit-tab-size:4; tab-size:4;">

<div class="crayon-line crayon-striped-line" id="crayon-56e5af776a922331614314-2"><span class="crayon-sy">[</span><span class="crayon-t">NSTimer</span> <span class="crayon-e ">scheduledTimerWithTimeInterval</span><span class="crayon-o">:</span><span class="crayon-cn">10</span></div>

<div class="crayon-line" id="crayon-56e5af776a922331614314-3"><span class="crayon-e ">                                 target</span><span class="crayon-v">:self</span></div>

<div class="crayon-line crayon-striped-line" id="crayon-56e5af776a922331614314-4"><span class="crayon-e ">                               selector</span><span class="crayon-o">:</span><span class="crayon-st">@selector</span><span class="crayon-sy">(</span><span class="crayon-e ">backgroundThreadFire</span><span class="crayon-o">:</span><span class="crayon-sy">)</span></div>

<div class="crayon-line" id="crayon-56e5af776a922331614314-5"><span class="crayon-e ">                               userInfo</span><span class="crayon-v">:nil</span></div>

<div class="crayon-line crayon-striped-line" id="crayon-56e5af776a922331614314-6"><span class="crayon-e ">                                repeats</span><span class="crayon-v">:YES</span><span class="crayon-sy">]</span><span class="crayon-sy">;</span></div>

</div>

</td>

</tr>

</tbody>

</table>

</div>

</div>

## 2.2 RunLoop观察者

源是在合适的同步或异步事件发生时触发，而run loop观察者则是在run loop本身运行的特定时候触发。你可以使用run loop观察者来为处理某一特定事件或是进入休眠的线程做准备。你可以将run loop观察者和以下事件关联：

1.  Runloop入口
2.  Runloop何时处理一个定时器
3.  Runloop何时处理一个输入源
4.  Runloop何时进入睡眠状态
5.  Runloop何时被唤醒，但在唤醒之前要处理的事件
6.  Runloop终止

和定时器类似，在创建的时候你可以指定run loop观察者可以只用一次或循环使用。若只用一次，那么在它启动后，会把它自己从run loop里面移除，而循环的观察者则不会。定义观察者并把它添加到run loop，只能使用Core Fundation。下面的例子演示了如何创建run loop的观察者：

<div id="crayon-56e5af776a928242797628" class="crayon-syntax crayon-theme-familiar-copy crayon-font-consolas crayon-os-mac print-yes notranslate" data-settings=" minimize scroll-always disable-anim" style=" margin-top: 12px; margin-bottom: 12px; margin-right: 24px; font-size: 15px !important; line-height: 20px !important;">

<div class="crayon-main" style="">

<table class="crayon-table">

<tbody>

<tr class="crayon-row">

<td class="crayon-nums " data-settings="show">

<div class="crayon-nums-content" style="font-size: 15px !important; line-height: 20px !important;">

<div class="crayon-num" data-line="crayon-56e5af776a928242797628-1">1</div>

<div class="crayon-num crayon-striped-num" data-line="crayon-56e5af776a928242797628-2">2</div>

<div class="crayon-num" data-line="crayon-56e5af776a928242797628-3">3</div>

<div class="crayon-num crayon-striped-num" data-line="crayon-56e5af776a928242797628-4">4</div>

<div class="crayon-num" data-line="crayon-56e5af776a928242797628-5">5</div>

<div class="crayon-num crayon-striped-num" data-line="crayon-56e5af776a928242797628-6">6</div>

<div class="crayon-num" data-line="crayon-56e5af776a928242797628-7">7</div>

<div class="crayon-num crayon-striped-num" data-line="crayon-56e5af776a928242797628-8">8</div>

<div class="crayon-num" data-line="crayon-56e5af776a928242797628-9">9</div>

<div class="crayon-num crayon-striped-num" data-line="crayon-56e5af776a928242797628-10">10</div>

<div class="crayon-num" data-line="crayon-56e5af776a928242797628-11">11</div>

<div class="crayon-num crayon-striped-num" data-line="crayon-56e5af776a928242797628-12">12</div>

<div class="crayon-num" data-line="crayon-56e5af776a928242797628-13">13</div>

<div class="crayon-num crayon-striped-num" data-line="crayon-56e5af776a928242797628-14">14</div>

<div class="crayon-num" data-line="crayon-56e5af776a928242797628-15">15</div>

<div class="crayon-num crayon-striped-num" data-line="crayon-56e5af776a928242797628-16">16</div>

<div class="crayon-num" data-line="crayon-56e5af776a928242797628-17">17</div>

<div class="crayon-num crayon-striped-num" data-line="crayon-56e5af776a928242797628-18">18</div>

</div>

</td>

<td class="crayon-code">

<div class="crayon-pre" style="font-size: 15px !important; line-height: 20px !important; -moz-tab-size:4; -o-tab-size:4; -webkit-tab-size:4; tab-size:4;">

<div class="crayon-line crayon-striped-line" id="crayon-56e5af776a928242797628-2"><span class="crayon-o">-</span> <span class="crayon-h"></span> <span class="crayon-sy">(</span><span class="crayon-t">void</span><span class="crayon-sy">)</span><span class="crayon-v">addObserverToCurrentRunloop</span> <span class="crayon-h"></span> <span class="crayon-sy">{</span></div>

<div class="crayon-line" id="crayon-56e5af776a928242797628-3"><span class="crayon-h">    </span><span class="crayon-c">// The application uses garbage collection, so noautorelease pool is needed.</span></div>

<div class="crayon-line" id="crayon-56e5af776a928242797628-5"><span class="crayon-h">    </span><span class="crayon-t">NSRunLoop</span> <span class="crayon-h"></span> <span class="crayon-v">*myRunLoop</span> <span class="crayon-h"></span> <span class="crayon-o">=</span> <span class="crayon-h"></span> <span class="crayon-sy">[</span><span class="crayon-t">NSRunLoop</span> <span class="crayon-h"></span> <span class="crayon-v">currentRunLoop</span><span class="crayon-sy">]</span><span class="crayon-sy">;</span></div>

<div class="crayon-line crayon-striped-line" id="crayon-56e5af776a928242797628-6"><span class="crayon-h">  </span> <span class="crayon-c">// Create a run loop observer and attach it to the runloop.</span></div>

<div class="crayon-line crayon-striped-line" id="crayon-56e5af776a928242797628-8"><span class="crayon-h">    </span><span class="crayon-t">CFRunLoopObserverContext</span><span class="crayon-h">  </span><span class="crayon-v">context</span> <span class="crayon-h"></span> <span class="crayon-o">=</span> <span class="crayon-h"></span> <span class="crayon-sy">{</span><span class="crayon-cn">0</span><span class="crayon-sy">,</span><span class="crayon-r">self</span><span class="crayon-sy">,</span> <span class="crayon-h"></span> <span class="crayon-t">NULL</span><span class="crayon-sy">,</span><span class="crayon-t">NULL</span><span class="crayon-sy">,</span> <span class="crayon-h"></span> <span class="crayon-t">NULL</span><span class="crayon-sy">}</span><span class="crayon-sy">;</span></div>

<div class="crayon-line" id="crayon-56e5af776a928242797628-9"><span class="crayon-h">  </span> <span class="crayon-t">CFRunLoopObserverRef</span><span class="crayon-h">    </span><span class="crayon-v">observer</span> <span class="crayon-h"></span> <span class="crayon-o">=</span><span class="crayon-t">CFRunLoopObserverCreate</span><span class="crayon-sy">(</span><span class="crayon-v">kCFAllocatorDefault</span><span class="crayon-sy">,</span></div>

<div class="crayon-line" id="crayon-56e5af776a928242797628-11"><span class="crayon-h">                                                              </span><span class="crayon-v">kCFRunLoopBeforeTimers</span><span class="crayon-sy">,</span><span class="crayon-t">YES</span><span class="crayon-sy">,</span> <span class="crayon-h"></span> <span class="crayon-cn">0</span><span class="crayon-sy">,</span> <span class="crayon-h"></span> <span class="crayon-o">&</span><span class="crayon-v">myRunLoopObserver</span><span class="crayon-sy">,</span> <span class="crayon-h"></span> <span class="crayon-o">&</span><span class="crayon-v">context</span><span class="crayon-sy">)</span><span class="crayon-sy">;</span></div>

<div class="crayon-line" id="crayon-56e5af776a928242797628-13"><span class="crayon-h">    </span><span class="crayon-st">if</span> <span class="crayon-h"></span> <span class="crayon-sy">(</span><span class="crayon-v">observer</span><span class="crayon-sy">)</span> <span class="crayon-h"></span> <span class="crayon-sy">{</span></div>

<div class="crayon-line crayon-striped-line" id="crayon-56e5af776a928242797628-14"><span class="crayon-h">        </span><span class="crayon-t">CFRunLoopRef</span><span class="crayon-h">    </span><span class="crayon-v">cfLoop</span> <span class="crayon-h"></span> <span class="crayon-o">=</span> <span class="crayon-h"></span> <span class="crayon-sy">[</span><span class="crayon-v">myRunLoopgetCFRunLoop</span><span class="crayon-sy">]</span><span class="crayon-sy">;</span></div>

<div class="crayon-line" id="crayon-56e5af776a928242797628-15"><span class="crayon-h">      </span> <span class="crayon-t">CFRunLoopAddObserver</span><span class="crayon-sy">(</span><span class="crayon-v">cfLoop</span><span class="crayon-sy">,</span> <span class="crayon-h"></span> <span class="crayon-v">observer</span><span class="crayon-sy">,</span> <span class="crayon-h"></span> <span class="crayon-v">kCFRunLoopDefaultMode</span><span class="crayon-sy">)</span><span class="crayon-sy">;</span></div>

<div class="crayon-line crayon-striped-line" id="crayon-56e5af776a928242797628-16"><span class="crayon-h">    </span><span class="crayon-sy">}</span></div>

<div class="crayon-line" id="crayon-56e5af776a928242797628-17"><span class="crayon-sy">}</span></div>

</div>

</td>

</tr>

</tbody>

</table>

</div>

</div>

其中，kCFRunLoopBeforeTimers表示选择监听定时器触发前处理事件，后面的YES表示循环监听。

## 2.3 RunLoop的事件队列

每次运行run loop，你线程的run loop对会自动处理之前未处理的消息，并通知相关的观察者。具体的顺序如下：

1.  通知观察者run loop已经启动
2.  通知观察者任何即将要开始的定时器
3.  通知观察者任何即将启动的非基于端口的源
4.  启动任何准备好的非基于端口的源
5.  如果基于端口的源准备好并处于等待状态，立即启动；并进入步骤9。
6.  通知观察者线程进入休眠
7.  将线程置于休眠直到任一下面的事件发生：
    *   某一事件到达基于端口的源
    *   定时器启动
    *   Run loop设置的时间已经超时
    *   run loop被显式唤醒
8.  通知观察者线程将被唤醒。
9.  处理未处理的事件
    *   如果用户定义的定时器启动，处理定时器事件并重启run loop。进入步骤2
    *   如果输入源启动，传递相应的消息
    *   如果run loop被显式唤醒而且时间还没超时，重启run loop。进入步骤2
10.  通知观察者run loop结束。

因为定时器和输入源的观察者是在相应的事件发生之前传递消息，所以通知的时间和实际事件发生的时间之间可能存在误差。如果需要精确时间控制，你可以使用休眠和唤醒通知来帮助你校对实际发生事件的时间。

因为当你运行run loop时定时器和其它周期性事件经常需要被传递，撤销run loop也会终止消息传递。典型的例子就是鼠标路径追踪。因为你的代码直接获取到消息而不是经由程序传递，因此活跃的定时器不会开始直到鼠标追踪结束并将控制权交给程序。

Run loop可以由run loop对象显式唤醒。其它消息也可以唤醒run loop。例如，添加新的非基于端口的源会唤醒run loop从而可以立即处理输入源而不需要等待其他事件发生后再处理。

从这个事件队列中可以看出：

*   ①如果是事件到达，消息会被传递给相应的处理程序来处理， runloop处理完当次事件后，run loop会退出，而不管之前预定的时间到了没有。你可以重新启动run loop来等待下一事件。
*   ②如果线程中有需要处理的源，但是响应的事件没有到来的时候，线程就会休眠等待相应事件的发生。这就是为什么run loop可以做到让线程有工作的时候忙于工作，而没工作的时候处于休眠状态。

## 2.4 什么时候使用run loop

仅当在为你的程序创建辅助线程的时候，你才需要显式运行一个run loop。Run loop是程序主线程基础设施的关键部分。所以，Cocoa和Carbon程序提供了代码运行主程序的循环并自动启动run loop。IOS程序中UIApplication的run方法（或Mac OS X中的NSApplication）作为程序启动步骤的一部分，它在程序正常启动的时候就会启动程序的主循环。类似的，RunApplicationEventLoop函数为Carbon程序启动主循环。如果你使用xcode提供的模板创建你的程序，那你永远不需要自己去显式的调用这些例程。

对于辅助线程，你需要判断一个run loop是否是必须的。如果是必须的，那么你要自己配置并启动它。你不需要在任何情况下都去启动一个线程的run loop。比如，你使用线程来处理一个预先定义的长时间运行的任务时，你应该避免启动run loop。Run loop在你要和线程有更多的交互时才需要，比如以下情况：

*   使用端口或自定义输入源来和其他线程通信
*   使用线程的定时器
*   Cocoa中使用任何performSelector…的方法
*   使线程周期性工作

如果你决定在程序中使用run loop，那么它的配置和启动都很简单。和所有线程编程一样，你需要计划好在辅助线程退出线程的情形。让线程自然退出往往比强制关闭它更好。





[jekyll-docs]: http://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
