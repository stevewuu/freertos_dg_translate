## 关于FreeRTOS内核

FreeRTOS内核是由亚马逊维护的开源软件。

FreeRTOS内核非常适合深度嵌入的实时应用程序微控制器或小型微处理器。这种类型的应用程序通常包括两个硬件的混合和软实时性要求。

软实时要求是那些规定时间限制的要求，但是违反最后期限的要求不适用使系统无用。例如，对按键的响应太慢可能会使系统看起来像恼人的没有反应，而实际上使其无法使用

严格的实时要求是那些规定了时间限制和违反最终期限的结果在系统绝对失败。例如，驾驶员的安全气囊有可能造成更大的伤害如果它对速度传感器输入的响应速度太慢，反应良好FreeRTOS内核是嵌入式的实时内核（或实时调度程序）
可以构建应用程序以满足严格的实时要求。它允许组织应用程序作为独立执行线程的集合。在只有一个核心的处理器上，只有一个单线程可以在任何时候执行。内核决定哪个线程应该执行通过检查由应用程序设计者分配给每个线程的优先级。在最简单的情况下，应用程序设计人员可以为实时实时的线程分配更高的优先级要求和优先级较低的线程执行软实时要求。这个会确保硬实时线程总是在软实时线程之前执行，但优先级高分配决定并不总是那么简单

如果你还没有完全理解前面的概念，不要担心。本向导详细地介绍了它们，并提供了许多示例来帮助您了解如何使用实时内核和FreeRTOS内核。

### 价值主张 

FreeRTOS内核的前所未有的全球性成功来自其引人注目的价值主张。 FreeRTOS内核是专业开发，严格的质量控制，强大的支持，不包含任何知识产权所有权模糊性，真正自由地在商业应用程序中使用，无需公开您的专有源代码。 您可以使用FreeRTOS内核将产品推向市场，无需支付任何费用，成千上万的人也可以做到这一点。 如果您希望在任何时候获得额外的备份，或者如果您的法律团队需要额外的书面保证或赔偿，那么有一个简单的低成本商业升级途径。 在您选择的任何时候，您都可以放心地选择走商业路线。

### 关于术语的说明

在FreeRTOS内核中，每个执行线程都被称为一个任务。 虽然在嵌入式社区中对术语没有共识，但是在一些应用领域中，线程可以有更具体的含义。

### 为什么要使用实时内核？

如果开发的系统很简单，在不使用内核的情况下，编写好的嵌入式软件有许多成熟的技术，那么这些技术可以提供最合适的解决方案。 在更复杂的情况下，使用内核是可取的，但交叉点发生的地方总是主观的。

任务优先级可以帮助确保应用程序满足其处理期限，但内核也可能带来其他不太明显的好处：

- 抽象出时间信息

内核负责执行时间并为应用程序提供一个与时间相关的API。这允许应用程序代码的结构更简单，并且整体代码尺寸更小。


- 可维护性/可扩展性

抽象计时细节可以减少模块之间的相互依赖关系，并使软件以受控和可预测的方式进化。另外，内核负责计时，所以
应用程序性能不易受底层硬件变化的影响。

- 模块化

任务是独立的模块，每个模块都应该有一个明确的目的。

- 团队发展

任务还应该有明确的界面，让团队更容易开发。

- 更轻松的测试

如果任务是界面清晰的独立模块，则可以单独进行测试。

- 代码重用

更大的模块化和更少的相互依赖性导致代码可以更少的努力重复使用。

- 提高效率

使用内核允许软件完全由事件驱动，所以没有处理时间通过轮询未发生的事件而浪费。 代码只在有必须完成的事情时执行。
计算效率节省是需要处理RTOS滴答中断，并将执行从一个任务切换到另一个任务。 但是，不使用RTOS的应用程序通常包含某种形式的刻度中断。

- 空闲时间

空闲任务是在调度程序启动时自动创建的。 只要没有希望执行的应用程序任务，它就会执行。 空闲任务可用于测量备用处理能力，执行后台检查，或者简单地将处理器置于低功耗模式。

- 能源管理

通过使用RTOS提高效率，处理器可以在低功耗模式下花费更多的时间。
每次空闲任务运行时，通过将处理器置于低功耗状态，可显着降低功耗。 FreeRTOS内核还有一个无滴答模式，可以让处理器进入低功耗模式并保持更长的时间。

- 灵活的中断处理

通过将处理推迟到由应用程序编写器创建的任务或FreeRTOS守护程序任务，中断处理程序可以保持很短的时间。

- 混合处理要求

简单的设计模式可以在应用程序中实现周期性，连续性和事件驱动处理的混合。 另外，通过选择适当的任务和中断优先级，可以满足硬和软实时要求。

### FreeRTOS内核功能

FreeRTOS内核具有以下标准功能：

- 抢先或合作运作
- 非常灵活的任务优先分配
- 灵活，快速，轻量级的任务通知机制
- 队列
- 二进制信号量
- 计算信号量
- 互斥体
- 递归互斥
- 软件定时器
- 事件组
- 勾选钩子功能
- 空闲挂钩功能
- 堆栈溢出检查
- 跟踪记录
- 任务运行时统计信息收集
- 可选的商业许可和支持
- 完整的中断嵌套模型（对于某些体系结构）
- 针对极低功耗应用的无风险功能
- 适当的软件管理的中断堆栈（这可以帮助节省RAM）

### 许可

根据MIT许可条款，FreeRTOS内核可供用户使用。

### 包括源文件和项目

源代码，预配置的项目文件以及所有示例的完整构建说明随附的zip文件中提供。 如果您没有收到书的副本，您可以从[http://www.FreeRTOS.org/Documentation/code]下载zip文件。 该zip文件可能不包含最新版本的FreeRTOS内核。
本书中的屏幕截图是在Microsoft Windows环境下使用FreeRTOS Windows端口执行示例时拍摄的。 使用FreeRTOS Windows端口的项目已预先配置为使用免费的Visual Studio Express版本构建，可以从http://www.microsoft.com/express下载。 尽管FreeRTOS Windows端口提供了一个方便的评估，测试和开发平台，但它并没有提供真正的实时行为。

