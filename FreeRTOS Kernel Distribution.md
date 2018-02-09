## FreeRTOS Kernel Distribution


FreeRTOS内核作为一个zip文件存档分发，其中包含所有官方FreeRTOS内核端口和大量预配置的演示应用程序。

### 了解FreeRTOS内核分配

FreeRTOS内核可以用大约20种不同的编译器构建，并且可以在30多种不同的处理器架构上运行。 编译器和处理器的每个支持组合被认为是一个单独的FreeRTOS端口。

### 构建FreeRTOS内核

你可以把FreeRTOS想象成一个库，它提供了多任务处理功能，否则就是裸机应用程序。
FreeRTOS是作为一组C源文件提供的。 一些源文件对于所有端口是通用的，而另外一些是特定于端口的。 将源文件构建为项目的一部分，以使FreeRTOS API可用于您的应用程序。 为了方便您，每个官方的FreeRTOS端口都提供了一个
演示应用程序 演示应用程序已预先配置为构建正确的源文件并包含正确的头文件。
演示应用程序应该开箱即用。 但是，自从演示发布以来，在构建工具中进行的更改可能会导致问题。 有关更多信息，请参阅本主题后面的演示应用程序。

### FreeRTOSConfig.h

FreeRTOS由一个名为FreeRTOSConfig.h的头文件配置。

FreeRTOSConfig.h用于定制FreeRTOS以用于特定的应用程序。 例如，FreeRTOSConfig.h包含诸如configUSEPREEMPTION的常量，其设置定义了是否将使用合作或抢先调度算法。 FreeRTOSConfig.h包含特定于应用程序的定义，所以它应该位于正在构建的应用程序的一部分的目录中，而不是位于包含FreeRTOS源代码的目录中。

为每个FreeRTOS端口提供演示应用程序，每个演示应用程序包含一个 FreeRTOSConfig.h文件。 因此，您从不需要从头开始创建FreeRTOSConfig.h文件。 相反，我们建议您先开始使用FreeRTOSConfig.h，然后再使用为FreeRTOS端口提供的演示应用程序所使用的FreeRTOSConfig.h。

### 官方FreeRTOS内核分发

FreeRTOS分布在一个zip文件中。 该zip文件包含所有FreeRTOS演示应用程序的所有FreeRTOS端口和项目文件的源代码。 它还包含一系列FreeRTOS +生态系统组件和一系列FreeRTOS +生态系统演示应用程序。

不要担心FreeRTOS发行版中的文件数量。 在任何一个应用程序中只需要少量的文件。

### FreeRTOS分发中的顶级目录

这里显示和说明FreeRTOS分配的一级和二级目录。

FreeRTOS

││

│├─ Source目录包含FreeRTOS源文件的源目录

││

│└─ Demo目录包含预配置和特定于端口的FreeRTOS演示项目

│

FreeRTOS-Plus

│

├─ Source目录包含某些FreeRTOS +生态系统组件源代码的源目录

│

└─Demo包含FreeRTOS +生态系统组件演示项目的演示目录

该zip文件只包含FreeRTOS源文件的一个副本，所有FreeRTOS演示项目以及所有FreeRTOS +演示项目。 您应该在FreeRTOS / Source目录中找到FreeRTOS源文件。 如果目录结构发生更改，则文件可能不会生成。

### 所有端口通用的FreeRTOS源文件

核心FreeRTOS源代码仅包含在所有FreeRTOS端口通用的两个C文件中。 这些被称为tasks.c和list.c. 它们直接位于FreeRTOS / Source目录中。 以下源文件位于相同的目录中：

queue.c

queue.c提供了队列和信号量服务。 queue.c几乎总是需要的。

•timers.c

timers.c提供软件定时器功能。 只有在要使用软件定时器的情况下，才需要将其包含在构建中。

•eventgroups.c

eventgroups.c提供了事件组功能。 只有在将要使用事件组的情况下，您才需要将其包含在构建中。

•croutine.c

croutine.c实现FreeRTOS协同例程功能。 只有在要使用联合例程时，才需要将其包含在构建中。 协同例程旨在用于非常小的微控制器。

他们现在很少使用，因此不能保持与其他FreeRTOS功能相同的水平。 本指南不包含协同例程。

FreeRTOS

│

└─Source

│

├─tasks.c FreeRTOS source file - always required

├─list.c FreeRTOS source file - always required

├─queue.c FreeRTOS source file - nearly always required

├─timers.c FreeRTOS source file - optional

├─eventgroups.c FreeRTOS source file - optional

└─croutine.c FreeRTOS source file - optional

人们认识到，文件名可能会导致名称空间冲突，因为许多项目已经包含具有相同名称的文件。 但是，现在更改文件的名称将会产生问题，因为这样会破坏与成千上万使用FreeRTOS的项目以及自动化工具和IDE插件的兼容性。

### 特定于端口的FreeRTOS源文件

特定于FreeRTOS端口的源文件包含在FreeRTOS / Source / portable目录中。

便携式目录按层次排列，首先由编译器，然后由处理器体系结构排列。

如果您在使用编译器“编译器”的架构“体系结构”的处理器上运行FreeRTOS，那么除了FreeRTOS核心源文件之外，您还必须构建位于FreeRTOS / Source / portable / [compiler] / [architecture]目录。

如第2章“堆内存管理”中所述，FreeRTOS也将堆内存分配视为便携层的一部分。使用FreeRTOS版本低于V9.0.0的项目必须包含堆内存管理器。从FreeRTOS V9.0.0开始，只有在FreeRTOSConfig.h中将 configSUPPORTDYNAMICALLOCATION设置为1时才需要堆内存管理器，或者如果configSUPPORTDYNAMICALLOCATION是未定义的。

FreeRTOS提供了五个示例堆分配方案。这五个方案被命名为heap1到heap5，分别由源文件heap1.c到heap5.c来实现。示例堆分配方案包含在FreeRTOS / Source / portable / MemMang目录中。如果你已经配置好了
FreeRTOS使用动态内存分配，那么有必要在你的项目中构建这五个源文件之一，除非你的应用程序提供了一个替代实现。

下图显示FreeRTOS目录树中的特定于端口的源文件。

FreeRTOS

│

└─Source

│

└─portable Directory containing all port-specific source files

│

├─MemMang Directory containing the 5 alternative heap allocation source files

│

├─[compiler 1] Directory containing port files specific to compiler 1

│ │

│ ├─[architecture 1] Contains files for the compiler 1 architecture 1 port

│ ├─[architecture 2] Contains files for the compiler 1 architecture 2 port

│ └─[architecture 3] Contains files for the compiler 1 architecture 3 port

│

└─[compiler 2] Directory containing port files specific to compiler 2

│

├─[architecture 1] Contains files for the compiler 2 architecture 1 port

├─[architecture 2] Contains files for the compiler 2 architecture 2 port

└─[etc.]

### 头文件

使用FreeRTOS API的源文件必须包含“FreeRTOS.h”，后面是包含正在使用的API函数的原型的头文件 - “task.h”，“queue.h”，“semphr.h” ，'timers.h'或'eventgroups.h'。

#### 演示应用程序 

每个FreeRTOS端口都至少包含一个演示应用程序，该应用程序应该不会生成错误或警告，尽管一些演示程序比其他演示程序要早，有时自演示版发布以来，构建工具的更改可能会导致问题。

Linux用户注意：FreeRTOS是在Windows主机上开发和测试的。 在Linux主机上构建演示项目时，偶尔会导致构建错误。 构建错误几乎总是与引用文件名称时使用的字母或文件路径中使用的斜杠字符的方向有关。 请使用FreeRTOS联系表格（http://www.FreeRTOS.org/contact）提醒我们任何此类错误。
演示应用程序有几个目的：

- 提供一个工作和预先配置项目的例子，包含正确的文件，并设置正确的编译器选项。
- 使用最少的设置或事先知识来进行“开箱即用”的实验。
- 演示FreeRTOS API如何使用。
- 作为可以创建实际应用程序的基础。

每个演示项目位于FreeRTOS / Demo目录下的唯一子目录中。子目录的名称表示演示项目所涉及的端口。

每个演示应用程序也由FreeRTOS.org网站上的网页描述。该网页包含以下信息：

- 如何在FreeRTOS目录结构中找到演示的项目文件。
- 项目配置使用哪些硬件。
- 如何设置运行演示的硬件。
- 如何构建演示。
- 演示预期如何运作。

所有演示项目都创建了常见演示任务的子集，其中的实现包含在FreeRTOS / Demo / Common / Minimal目录中。常见的演示任务纯粹是为了演示FreeRTOS API如何使用 - 它们没有实现任何特别有用的功能。
最近的演示项目也可以建立一个初学者的“闪烁”项目。 Blinky项目是非常基本的。通常他们只会创建两个任务和一个队列。

每个演示项目都包含一个名为main.c的文件。这包含main（）函数，从中创建所有演示应用程序任务。请参阅各个main.c文件中的注释以获取该演示的具体信息。

以下文件显示FreeRTOS / Demo目录层次结构。

FreeRTOS

│

└─Demo Directory containing all the demo projects
│

├─[Demo x] Contains the project ﬁle that builds demo 'x'

│

├─[Demo y] Contains the project ﬁle that builds demo 'y'

│

├─[Demo z] Contains the project ﬁle that builds demo 'z'

│

└─Common Contains ﬁles that are built by all the demo applications

### 创建一个FreeRTOS项目

每个FreeRTOS端口都至少有一个预先配置的演示应用程序，应该无任何错误或警告地构建。 建议通过调整其中一个现有项目来创建新项目; 这将允许项目包含正确的文件，安装正确的中断处理程序和设置正确的编译器选项。

从现有的演示项目开始一个新的应用程序：

1. 打开提供的演示项目，确保按照预期构建和执行。
2. 删除定义演示任务的源文件。 Demo / Common目录下的任何文件都可以从项目中删除。
3. 删除main（）中除了prvSetupHardware（）和vTaskStartScheduler（）之外的所有函数调用，如清单1所示。
4. 检查项目仍然建立。

遵循这些步骤将创建一个包含正确的FreeRTOS源文件的项目，但不定义任何功能。

    int main( void )

    {
    
    	/* Perform any hardware setup necessary. */ 
    
    	prvSetupHardware();
    	/* --- APPLICATION TASKS CAN BE CREATED HERE --- */
    	
    	/* Start the created tasks running. */ 
    	vTaskStartScheduler();
    	/* Execution will only reach here if there was insufficient heap to start the scheduler. */ 
    	
    	for( ;; );
    	return 0;
    
    }

### 从零开始创建一个新项目


如前所述，建议从现有演示项目创建新项目。如果这是不可取的，那么可以使用以下程序创建一个新项目：



1. 使用您选择的工具链，创建一个尚未包含任何FreeRTOS源文件的新项目。
2. 确保可以构建新项目，下载到目标硬件并执行。
3. 只有当您确定您已经有一个工作项目时，才能将表1中详述的FreeRTOS源文件添加到项目中。 
4. 将提供给正在使用的端口的演示项目使用的FreeRTOSConfig.h头文件复制到项目目录中。
5. 将以下目录添加到项目将搜索以查找标题文件的路径中：

	- FreeRTOS/Source/include
	- FreeRTOS/Source/portable/[compiler]/[architecture] (where [compiler] and [architecture] are correct for your chosen port)
	- 包含FreeRTOSCon fi.h.h头文件的目录


1. 从相关演示项目复制编译器设置。

2. 安装可能需要的任何FreeRTOS中断处理程序。作为参考，使用描述正在使用的端口的网页以及为正在使用的端口提供的演示项目。

>| File				            | Location                    |
>
>| tasks.c			            | FreeRTOS/Source             |
>
>| queue.c			            | FreeRTOS/Source             |
>
>| list.c				        | FreeRTOS/Source             |
>
>| timers.c			            | FreeRTOS/Source             |
>
>| eventgroups.c		            | FreeRTOS/Source             |
>
>| All C and assembler ﬁles		| FreeRTOS/Source/portable/[compiler]/ [architecture]
>
>| heapn.c	                    | FreeRTOS/Source/portable/MemMang, where n is either 1, 2, 3, 4 or 5. This ﬁle became optional from FreeRTOS V9.0.0.

<table>
<thead>
<tr>
  <th>File</th>
  <th>Location</th>
</tr>
</thead>
<tbody>
<tr>
  <td>tasks.c</td>
  <td>FreeRTOS/Source</td>
</tr>
<tr>
  <td>queue.c</td>
  <td>FreeRTOS/Source</td>
</tr>
<tr>
  <td>list.c</td>
  <td>FreeRTOS/Source</td>
</tr>
<tr>
  <td>timers.c</td>
  <td>FreeRTOS/Source</td>
</tr>
<tr>
  <td>eventgroups.c</td>
  <td>FreeRTOS/Source</td>
</tr>
<tr>
  <td>All C and assembler ﬁles</td>
  <td> FreeRTOS/Source/portable/[compiler]/[architecture]</td>
</tr>
<tr>
  <td>heapn.c</td>
  <td>FreeRTOS/Source/portable/MemMang, where n is either 1, 2, 3, 4 or 5. This ﬁle became optional from FreeRTOS V9.0.0.</td>
</tr>
</tbody>
</table>
使用FreeRTOS版本低于V9.0.0的项目必须构建一个heapn.c文件。 从FreeRTOS V9.0.0开始，只有在FreeRTOSConfig.h中将configSUPPORTDYNAMICALLOCATION设置为1或未定义 configSUPPORTDYNAMICALLOCATION时，才需要heapn.c文件。 有关更多信息，请参阅第2章，堆内存管理。

### 数据类型和编码风格指南

FreeRTOS的每个端口都有一个唯一的portmacro.h头文件，它包含两个特定于端口的数据类型（除其他外）的定义：TickTypet和BaseTypet。

下表列出了FreeRTOS使用的数据类型。

一些编译器使所有非限定字符变量无符号，而其他编译器使它们被签名。 出于这个原因，FreeRTOS源代码明确地限定每个使用“signed”或“unsigned”的字符，除非char用来保存一个ASCII字符，或者指向char的指针用来指向一个字符串。

纯int类型从不使用。

### 变量名称

变量前缀的类型为：'c'表示char，'s'表示int16t（short），'l'int32t（long），'x'表示BaseTypet和其他非标准类型（结构，任务句柄，队列 手柄等）。

如果一个变量是无符号的，那么它的前缀是“u”。 如果一个变量是一个指针，那么它也以“p”作为前缀。
例如，类型为uint8t的变量将以“uc”为前缀，并且指向char的类型指针变量将以“pc”为前缀。

### 函数名称

函数带有它们返回的类型和它们在其中定义的文件的前缀。 例如：
•cTaskPrioritySet（）返回一个void，并在task.c中定义。
•xQueueReceive（）返回一个BaseType_t类型的变量，并在queue.c中定义。
•pvTimerGetTimerID（）返回一个指向void的指针，并在timers.c中定义。
文件范围（私有）功能以“prv”作为前缀。

### 格式化

一个Tab键始终设置为等于四个空格。

### 宏名

大多数宏被写成大写字母，并以小写字母作为前缀，表示宏定义的位置。

下表列出了宏前缀

<table>
<thead>
<tr>
  <th>Prefix</th>
  <th>Location of macro definetion</th>
</tr>
</thead>
<tbody>
<tr>
  <td>port (for example, portMAXDELAY)</td>
  <td>portable.h or portmacro.h</td>
</tr>
<tr>
  <td>task (for example, taskENTERCRITICAL())</td>
  <td>task.h</td>
</tr>
<tr>
  <td>pd (for example, pdTRUE)</td>
  <td>projdefs.h</td>
</tr>
<tr>
  <td>config (for example, configUSEPREEMPTION)</td>
  <td>projdefs.h</td>
</tr>
<tr>
  <td>err (for example, errQUEUEFULL)</td>
  <td>FreeRTOS/Source</td>
</tr>
</tbody>
</table>
信号量API几乎完全是作为一组宏来编写的，但遵循函数命名约定而不是宏命名约定。
下表列出了FreeRTOS源代码中使用的宏。
<table>
<thead>
<tr>
  <th>Macro</th>
  <th>Value</th>
</tr>
</thead>
<tbody>
<tr>
  <td>pdTRUE</td>
  <td>1</td>
</tr>
<tr>
  <td>pdFALSE</td>
  <td>0</td>
</tr>
<tr>
  <td>pdPASS</td>
  <td>1</td>
</tr>
<tr>
  <td>pdFAIL</td>
  <td>0</td>
</tr>
</tbody>
</table>

### 过多类型转换解释

FreeRTOS源代码可以用许多不同的编译器编译，所有这些编译器在产生警告的方式和时间上都有所不同。 特别是，不同的编译器希望以不同的方式使用转换。 因此，FreeRTOS源代码包含比通常保证更多的类型转换。