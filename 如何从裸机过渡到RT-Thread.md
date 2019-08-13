# 如何从裸机过渡到 RT-Thread？

我们在学习单片机的时候，首先要做的事情就是搭建一个工程，然后点亮一个 led 灯；之后可能就是一些外设了（uart、看门狗、pwm、lcd、rtc、adc、flash 等等）。

那么现在要学习实时操作系统的时候，首先需要做什么？RT-Thread 文档中心给出了学习路线：上手、内核、Env、外设、组件、软件包等。对于一个完全没有接触过 RTOS 的人来说，这个自学路线是一个不错的选择，能较为全面的对 RT-Thread 进行学习；另一种就是跟着学习营一起学习，有明确的目标及答疑支持。

## 【1】一般学习路线

### 上手阶段

```
目的：运行一个 RT-Thread 工程。
```

在上手阶段，建议大家使用模拟仿真的方式进行即可，这里有一篇文章，已经不需要自己搭建工程，并且包含有一个 led 跑马灯的代码，最后还说明了如何运行这个 led 灯代码。[Keil 模拟器 STM32F103 上手指南](https://www.rt-thread.org/document/site/tutorial/quick-start/stm32f103-simulator/stm32f103-simulator/)。

运行完代码之后，此时你最想问的可能就是这些问题：

（1）跑马灯 led() 没有被 main 函数调用，在窗口敲命令就运行起来了？

（2）MSH_CMD_EXPORT 是怎么回事？屏蔽掉这句，在窗口就不能输入 led 命令了。

（3）led 灯没有被初始化吗？怎么只有短短几行代码，led 就初始化完成并且可以闪烁了？

这些问题这里暂不解答，在下面的 <u>内核学习</u> 中就会接触到答案。

### 内核学习

```
目的：学习 RT-Thread 操作系统内核。
```

RT-Thread 的内核也是与硬件无关的，所以也可以继续在 keil 上模拟仿真进行学习，使用的代码也是上手阶段的代码。

内核主要包含了：线程管理、ipc 及信号、时钟管理、内存管理几个方面。（中断管理是对于非 cortex m 的管理，所以若是使用 cortex m 的进行入门，则暂时可以不用深入了解中断管理）。

学习线程：掌握线程的创建。[内核基础](https://www.rt-thread.org/document/site/programming-manual/basic/basic/) 与 [线程管理](https://www.rt-thread.org/document/site/programming-manual/thread/thread/)。
时钟管理：掌握如何创建一个定时器，知道什么是 SOFT TIMER 与 HARD TIMER。[时钟管理](https://www.rt-thread.org/document/site/programming-manual/timer/timer/)。
IPC 及信号：掌握 IPC 的创建、发送 / 释放、接收 / 获取；信号这里熟悉下就行。线程间 [同步](https://www.rt-thread.org/document/site/programming-manual/ipc1/ipc1/) 与 [通信](https://www.rt-thread.org/document/site/programming-manual/ipc2/ipc2/)。
内存管理：内存池的创建与使用；内存堆的创建与使用。[内存管理](https://www.rt-thread.org/document/site/programming-manual/memory/memory/)。

### Env 工具的使用

```
目的：掌握 Env 的使用。
```

这为后面的使用外设、组件与软件包做基础，因为 RT-Thread 有功能可裁剪特性，Env 工具就是对 RT-Thread 进行裁剪的工具，当需要使用 UART1 外设时，就打开 UART1；当需要使用文件系统组件时，直接打开；当需要一个内核示例代码时，直接选择并下载；并且这些功能在不使用时都可以关闭。Env 工具使用非常简单，就是一个使用空格按键来选中某功能 或者取消某功能的工具。

掌握几个常用命令即可：

- `menuconfig`：进入配置界面。在该界面可以使用的按键有空格键、方向键、Esc 键，Enter 键。
- `pkgs --update`：下载选中的软件包。
- `scons --target=mdk5`：生成 mdk5 工程，使配置生效到工程中。

### 外设的使用

```
目标：会使用常用外设。如 PIN IIC UART SPI 等.
```

> 该阶段需要硬件平台做支撑。然后还需要自己做一个适配开发板的 BSP。

在裸机上使用外设时，可能会有一系列的初始化、配置以及如何去使用外设的这些教程，那么在学习 RT-Thread 时，就不需要考虑这么多了，你只需要知道你想怎么使用就可以，调用的外设函数都是统一的接口。这是因为在 RT-Thread 中，外设均对接到了 IO 设备驱动框架，所以有了统一的 device 接口，rt_device_init/open/read/write/control/close。当然还有一些特殊的外设设备，为例使用简单或者体现特性，它们有自己特殊的接口，如 pin 设备、pwm 设备等，但是在不同平台上使用这些外设的接口依旧是一样的。

首先需要理解 IO 设备驱动框架，举一个简单的例子，我们把框架看做衣服，那么 IO 设备驱动框架也就是 IO 设备的衣服。不论是哪个驱动、哪个平台的驱动，它们穿上衣服之后，开发者对它们的操作就变成了统一操作，应用函数不会随着硬件平台变化。

现在，传感器也有了传感器驱动框架，如果开发者将传感器驱动对接到框架上，那么就可以使用标准 device 接口对传感器进行操作了。

### 组件与软件包

```
目标：
    组件：会使用文件系统、网络等；
    软件包：会下载软件包并使用软件包，如温湿度传感器软件包、onenet 软件包等。
```

组件与软件包使用起来应该是最方便的，组件配置即用，软件包也是选中之后下载即可。

在组件应用方面，RT-Thread 也提供了诸多的文档。这里是 [组件文档](https://www.rt-thread.org/document/site/programming-manual/finsh/finsh/)。

软件包的配置与使用方法都写在了软件包中的 README 里，点击这里查看文档 [常用软件包介绍](https://www.rt-thread.org/document/site/application-note/packages/netutils/an0018-system-netutils/)，点击这里 [查找软件包](http://packages.rt-thread.org/)。

## 【2】通过做小项目学习 RT-Thread

> 这也是这个 repo 在做的事情，通过举办学习营指导初学者进行小项目的实现，跨入嵌入式实时操作系统领域，熟悉 RT-Thread 应用。

通过实现一个小项目的方式学习 RT-Thread 是非常好的选择。在做之前可能给出一些详细的指导说明，让初学者在引导之下完成小项目。这种方法最大的优点就是能引导初学者直入主题，使用 RT-Thread 进行实际的应用。

当然小项目是专题性以及针对性的，所以也不可能将 RT-Thread 所有的资源使用齐全，只能使用到与项目或硬件相关的资源。所以对项目之外的外设、组件、软件包感兴趣的伙伴，就可以再次通过上面【1】中学习路线中的方法进行学习，然后将这些应用到实践之中。

### 学习营

学习营一般涉及以下内容，会根据实际应用对内容进行增减调整。

- 内核：使用到 IPC、线程。

- Env：熟悉 Env 工具的使用。

- 组件：文件系统、网络。根据实际情况进行调整。

- 软件包：传感器、lcd、云平台如 onenet。


学习营的学习路线是：将 RT-Thread 在开发板上运行起来、使用 PIN 设备点灯、PIN 设备按键、IPC、组件...

