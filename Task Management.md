## 任务管理

本节介绍的概念对于理解如何使用FreeRTOS以及FreeRTOS应用程序的行为方式至关重要。 本节涵盖：

- FreeRTOS如何为应用程序内的每个任务分配处理时间。
- FreeRTOS如何选择在任何给定时间执行哪项任务。
- 每项任务的相对优先级如何影响系统行为。
- 任务可以存在的状态。

本节还将向您展示：

- 如何实施任务。
- 如何创建一个或多个任务实例。
- 如何使用任务参数。
- 如何更改已创建的任务的优先级。
- 如何删除任务。
- 如何使用任务实施定期处理。 有关更多信息，请参阅software_tier_management。
- 执行空闲任务时以及如何使用空闲任务。

### 任务函数

任务被实现为C函数。 关于它们唯一特别的是它们的原型，它必须返回void并取一个void指针参数，如下所示。

	void ATaskFunction（void * pvParameters）;

每项任务本身都是一个小程序。 它有一个入口点，通常会在无限循环内永远运行，并且不会退出。

FreeRTOS任务不得以任何方式从其实现功能中返回。 它们不得包含“返回”语句，并且不得允许执行超过该函数的结尾。 如果任务不再需要，应该明确删除。

单个任务功能定义可用于创建任意数量的任务。 每个创建的任务都是一个单独的执行实例，具有自己的堆栈以及任务本身内定义的任何自动（堆栈）变量的副本。

这里显示了一个典型任务的结构。

    void ATaskFunction( void *pvParameters )
    {
    /* Variables can be declared just as per a normal function. Each instance of a task created
    using this example function will have its own copy of the lVariableExample variable. This
    would not be true if the variable was declared static, in which case only one copy of the
    variable would exist, and this copy would be shared by each created instance of the task.
    (The prefixes added to variable names are described in Data Types and Coding Style Guide.)
    */
    int32_t lVariableExample = 0;
    /* A task will normally be implemented as an infinite loop. */
    for( ;; )
    {
    /* The code to implement the task functionality will go here. */
    }
    /* Should the task implementation ever break out of the above loop, then the task must be
    deleted before reaching the end of its implementing function. The NULL parameter passed to
    the vTaskDelete() API function indicates that the task to be deleted is the calling (this)
    task. The convention used to name API functions is described in Data Types and Coding
    Style Guide. */
    vTaskDelete( NULL );
    }

### 顶级任务状态（Top-level states）

应用程序可以由许多任务组成。 如果运行应用程序的处理器包含单个内核，则在任何给定时间只能执行一个任务。 这意味着任务可以存在于两个任务之一：运行和未运行。 请记住，这太简单了。 在本节的后面，您会看到Not Running状态实际上包含许多子状态。

当任务处于运行状态时，处理器正在执行任务的代码。 当任务处于未运行状态时，任务处于休眠状态，其状态已保存，因此在下一次调度程序决定进入运行状态时准备好继续执行任务。 当一个任务恢复执行时，它会在它最后一次离开运行状态之前执行它的指令。

下图显示了顶级任务状态和转换。

![](https://github.com/stevewuu/freertos_dg_translate/blob/master/image/Task-1.bmp)

据说已经从非运行状态转换到运行状态的任务被切换或交换。相反，从运行状态转换到未运行状态的任务据说已被切换出或换出。 FreeRTOS调度程序是唯一可以切换任务的实体。

### 创建任务

#### xTaskCreate() API Function

FreeRTOS V9.0.0还包含xTaskCreateStatic（）函数，该函数在编译时分配静态创建任务所需的内存。 任务使用FreeRTOS xTaskCreate（）API函数创建。

这可能是所有API函数中最复杂的。 由于任务是多任务系统的最基本组成部分，因此必须先掌握它们。 本指南中有许多xTaskCreate（）函数的示例。

有关使用的数据类型和命名约定的信息，请参阅FreeRTOS内核分发中的数据类型和编码风格指南。

以下显示xTaskCreate（）API函数原型。

    BaseType_t xTaskCreate( TaskFunction_t pvTaskCode, const char * const pcName, uint16_t usStackDepth, void *pvParameters, UBaseType_t uxPriority, TaskHandle_t *pxCreatedTask );

这里是xTaskCreate（）参数和返回值的列表。

- pvTaskCode：任务是简单的C函数，永远不会退出，因此通常被实现为无限循环。 pvTaskCode参数只是一个指向实现任务的函数的指针
（实际上，只是函数的名称）。
- pcName：任务的描述性名称。 FreeRTOS不会以任何方式使用此功能。它仅包含在内作为调试帮助。通过人类可读的名字识别任务比尝试更简单
通过手柄识别它。应用程序定义的常量configMAX_TASK_NAME_LEN定义了任务名称可以采用的最大长度，包括NULL终止符。提供比字符串长的字符串这个最大值将导致字符串被无声截断。
- usStackDepth：每个任务都有自己独特的堆栈，当内核由内核分配给任务时该任务已创建。 usStackDepth值告诉内核有多大堆栈。价值
指定堆栈可容纳的字数，而不是字节数。例如，如果堆栈是32位宽，usStackDepth以100的形式传入，则会分配400个字节的堆栈空间
（100 * 4字节）。堆栈深度乘以堆栈宽度不得超过最大值可以包含在uint16_t类型的变量中。空闲任务使用的堆栈大小为由应用程序定义的常量configMINIMAL_STACK_SIZE定义。这是唯一的方法FreeRTOS源代码使用configMINIMAL_STACK_SIZE设置。常量也用在里面
演示应用程序，以帮助演示在多处理器体系结构中移植。价值在正在使用的处理器架构的FreeRTOS演示应用程序中分配给此常量
是任何任务推荐的最小值。如果你的任务使用了很多堆栈空间，那么你必须分配更大的价值。没有简单的方法来确定任务所需的堆栈空间。有可能的来计算，但大多数用户会简单地分配他们认为合理的价值，然后使用FreeRTOS的特点是确保分配的空间确实足够，并且RAM不在
浪费了。有关如何查询任务所使用的最大堆栈空间的信息，请参阅故障排除部分中的堆栈溢出（第239页）。
- pvParameters：任务函数接受指向void（void *）的指针类型参数。分配的值pvParameters是传递给任务的值。本指南中有一些示例
参数如何使用。
- uxPriority：定义任务执行的优先级。优先级可以从中分配0（优先级最低）设置为（configMAX_PRIORITIES - 1），这是最高优先级。
configMAX_PRIORITIES是用户定义的常量，在任务优先级（p。33）中进行了介绍。 在上面传递一个uxPriority值 （configMAX_PRIORITIES - 1）将导致分配给该任务的优先级被静默地设置为最大合法值。
- pxCreatedTask：该参数可用于向正在创建的任务传递一个句柄。 然后，该句柄可用于在API调用中引用任务，例如，更改任务优先级或删除任务。 如果您的应用程序对任务句柄没有用处，则可将pxCreatedTask设置为NULL。

有两种可能的返回值：表示任务已成功创建的pdPASS和表示该任务尚未创建的pdPASS，因为FreeRTOS没有足够的可用内存空间来分配足够的RAM来保存任务数据结构和叠加。 有关更多信息，请参阅堆内存管理（p。14）。

#### 创建多任务（示例1）

这个例子演示了创建两个简单任务所需的步骤，然后开始执行它们。 这些任务只是周期性地打印出一个字符串，使用粗略的空循环来创建周期延迟。 两个任务都以相同的优先级创建，并且除了打印出的字符串外都是相同的。 以下代码显示了示例1中使用的第一个任务的实现。

    void vTask1( void *pvParameters )
    {
	    const char *pcTaskName = "Task 1 is running\r\n";
	    volatile uint32_t ul; /* volatile to ensure ul is not optimized away.*/
	    /* As per most tasks, this task is implemented in an infinite loop. */
	    for( ;; )
		    {
		    /* Print out the name of this task. */
		    vPrintString( pcTaskName );
		    /* Delay for a period. */
		    for( ul = 0; ul < mainDELAY_LOOP_COUNT; ul++ )
		    {
		    /* This loop is just a very crude delay implementation. There is nothing to do in
		    here. Later examples will replace this crude loop with a proper delay/sleep function. */
		    }
	    }
    }

以下代码显示了示例1中使用的第二个任务的实现。

    void vTask2( void *pvParameters )
    {
    	const char *pcTaskName = "Task 2 is running\r\n";
    	volatile uint32_t ul; /* volatile to ensure ul is not optimized away.*/
    	/* As per most tasks, this task is implemented in an infinite loop. */
    	for( ;; )
    	{
    		/* Print out the name of this task. */
    		vPrintString( pcTaskName );
    		/* Delay for a period. */
    		for( ul = 0; ul < mainDELAY_LOOP_COUNT; ul++ )
    		{
    		/* This loop is just a very crude delay implementation. There is nothing to do
    		in here. Later examples will replace this crude loop with a proper delay/sleep function.
    		*/
    		}
    	}
    }

main（）函数在启动调度程序之前创建任务。

    int main( void )
    {
	    /* 
		 * Create one of the two tasks. Note that a real application should check the return
	     * value of the xTaskCreate() call to ensure the task was created successfully. 
		 */
	    xTaskCreate( vTask1, /* Pointer to the function that implements thetask. */ "Task 1",/
	    * Text name for the task. This is to facilitate debugging only. */ 1000, /* Stack depth -
	    small microcontrollers will use much less stack than this. */ NULL, /* This example does
	    not use the task parameter. */ 1, /* This task will run at priority 1. */ NULL ); /* This
	    example does not use the task handle. */
	    /* Create the other task in exactly the same way and at the same priority. */
	    xTaskCreate( vTask2, "Task 2", 1000, NULL, 1, NULL );
	    /* Start the scheduler so the tasks start executing. */
	    vTaskStartScheduler();
	    /* If all is well then main() will never reach here as the scheduler will now be
	    running the tasks. If main() does reach here then it is likely that there was insufficient
	    heap memory available for the idle task to be created. For more information, see Heap
	    Memory Management. */
	    for( ;; );
    }

输出显示在这里。

![](https://github.com/stevewuu/freertos_dg_translate/blob/master/image/Task-2.bmp)

这两项任务似乎同时执行。 但是，由于两个任务都在同一个处理器内核上执行，所以情况并非如此。 事实上，这两项任务正在迅速进入和退出Running状态。 两个任务都以相同的优先级运行，因此可以在同一个处理器内核上共享时间。 他们的执行模式如下图所示。

![](https://github.com/stevewuu/freertos_dg_translate/blob/master/image/Task-3.bmp)

图中底部的箭头显示时间t1以后的时间流逝。 彩色线条表示在每个时间点执行哪个任务（例如，任务1正在时间t1和时间t2之间执行）。
任何时候在运行状态下只能有一个任务存在，因此当一个任务进入运行状态（任务切换）时，另一个任务进入非运行状态（任务切换）。
这两个任务都是在启动调度程序之前从main（）中创建的。 也可以从另一个任务中创建一个任务。 例如，任务2可以从任务1中创建。
以下代码显示在调度程序启动后从另一个任务内创建的任务。

    void vTask1( void *pvParameters )
    {
    	const char *pcTaskName = "Task 1 is running\r\n";
    	volatile uint32_t ul; /* volatile to ensure ul is not optimized away.*/
    	/* If this task code is executing then the scheduler must already have been started.
    	Create the other task before entering the infinite loop. */
    	xTaskCreate( vTask2, "Task 2", 1000, NULL, 1, NULL );
    	for( ;; )
    	{
    		/* Print out the name of this task. */
    		vPrintString( pcTaskName );
    		/* Delay for a period. */
    		for( ul = 0; ul < mainDELAY_LOOP_COUNT; ul++ )
    		{
    		/* This loop is just a very crude delay implementation. There is nothing to do
    		in here. Later examples will replace this crude loop with a proper delay/sleep function.
    		*/
    		}
    	}
    }

#### 使用任务参数（示例2）

示例1中创建的两个任务几乎完全相同。 唯一的区别是他们打印出的文本字符串。 通过创建单个任务实现的两个实例可以删除此重复。 然后可以使用任务参数将每个任务应该输出的字符串传递给每个任务。

以下内容包含单个任务函数（vTaskFunction）的代码。 这个函数取代了例1中使用的两个任务函数（vTask1和vTask2）。任务参数被转换为char *以获取任务应该打印出的字符串。

    void vTaskFunction( void *pvParameters )
    {
    	char *pcTaskName;
    	volatile uint32_t ul; /* volatile to ensure ul is not optimized away.*/
    	/* The string to print out is passed in via the parameter. Cast this to a character
    	pointer. */
    	pcTaskName = ( char * ) pvParameters;
    	/* As per most tasks, this task is implemented in an infinite loop. */
    	for( ;; )
    	{
    		/* Print out the name of this task. */
    		vPrintString( pcTaskName );
    		/* Delay for a period. */
    		for( ul = 0; ul < mainDELAY_LOOP_COUNT; ul++ )
    		{
    		/* This loop is just a very crude delay implementation. There is nothing to do
    		in here. Later exercises will replace this crude loop with a delay/sleep function. */
    		}
    	}
    }

即使现在只有一个任务实现（vTaskFunction），也可以创建多个已定义任务的实例。 每个创建的实例将在FreeRTOS调度程序的控制下独立执行。

以下代码显示如何使用xTaskCreate（）函数的pvParameters参数将文本字符串传递到任务中。

    /* Define the strings that will be passed in as the task parameters. These are defined
    const and not on the stack to ensure they remain valid when the tasks are executing. */
    static const char *pcTextForTask1 = "Task 1 is running\r\n";
    static const char *pcTextForTask2 = "Task 2 is running\r\n";
    int main( void )
    {
	    /* Create one of the two tasks. */
	    xTaskCreate( vTaskFunction, /* Pointer to the function that implements the task.
	    */ "Task 1", /* Text name for the task. This is to facilitate debugging only. */
	    1000, /* Stack depth - small microcontrollers will use much less stack than this. */
	    (void*)pcTextForTask1, /* Pass the text to be printed into the task using the task
	    parameter. */ 1, /* This task will run at priority 1. */ NULL );
	    /* The task handle is not used in this example. */
	    /* Create the other task in exactly the same way. Note this time that multiple tasks
	    are being created from the SAME task implementation (vTaskFunction). Only the value passed
	    in the parameter is different. Two instances of the same task are being created. */
	    xTaskCreate( vTaskFunction, "Task 2", 1000, (void*)pcTextForTask2, 1, NULL );
	    /* Start the scheduler so the tasks start executing. */
	    vTaskStartScheduler();
	    /* If all is well then main() will never reach here as the scheduler will now be
	    running the tasks. If main() does reach here then it is likely that there was insufficient
	    heap memory available for the idle task to be created. For more information, see Heap
	    Memory Management. */
	    for( ;; );
    }

#### 任务优先级

xTaskCreate（）API函数的uxPriority参数为正在创建的任务分配一个初始优先级。您可以使用vTaskPrioritySet（）API函数在调度程序启动后更改优先级。

可用优先级的最大数量由FreeRTOSConfig.h中的应用程序定义的configMAX_PRIORITIES编译时配置常量设置。低数字优先级值表示低优先级任务，优先级0是可能的最低优先级。因此，可用优先级的范围是0到（configMAX_PRIORITIES - 1）。任何数量的任务可以共享相同的优先级，确保最大的设计灵活性。

FreeRTOS调度程序可以使用两种方法之一来决定哪个任务将处于运行状态。configMAX_PRIORITIES可以设置的最大值取决于使用的方法：

1.通用方法
该方法以C语言实现，可与所有FreeRTOS体系结构端口一起使用。
当使用泛型方法时，FreeRTOS不会限制configMAX_PRIORITIES可以设置的最大值。但是，我们建议您将configMAX_PRIORITIES值保持在所需的最小值，因为它的值越高，所消耗的RAM越多，并且最坏情况下的执行时间越长。
如果在FreeRTOSConfig.h中将configUSE_PORT_OPTIMISED_TASK_SELECTION设置为0，或者未定义configUSE_PORT_OPTIMISED_TASK_SELECTION，或者如果
泛型方法是为使用中的FreeRTOS端口提供的唯一方法。

2.架构优化的方法
此方法使用少量的汇编代码，并且比通用方法更快。 configMAX_PRIORITIES设置不影响最坏情况下的执行时间。
如果使用此方法，则configMAX_PRIORITIES不能大于32.与通用方法一样，应将configMAX_PRIORITIES保持在所需的最小值，因为其值越高，将消耗的RAM就越多。
如果FreeRTOSConfig.h中的configUSE_PORT_OPTIMISED_TASK_SELECTION设置为1，将使用此方法。
并非所有FreeRTOS端口都提供了架构优化方法。
FreeRTOS调度程序将始终确保能够运行的最高优先级任务是选择进入运行状态的任务。在具有相同优先级的多个任务能够运行的情况下，调度程序将依次将每个任务转换为运行状态并将其转移出运行状态。

#### 时间测量和嘀嗒中断

调度算法（第56页）部分描述了一个称为时间分片的可选功能。 时间分割已被用在迄今为止提出的例子中。 这是他们产出的产品中观察到的行为。 在这些例子中，两个任务都是以相同的优先级创建的，而且这两个任务总是能够运行。 因此，每个任务都针对一个时间片执行，在时间片开始时进入运行状态，并在时间片结束时退出运行状态。 在这个图中，t1和t2之间的时间等于一个时间片。

![](https://github.com/stevewuu/freertos_dg_translate/blob/master/image/Task-4.bmp)

为了能够选择下一个要运行的任务，调度程序本身必须在每个时间片结束时执行。时间片的结束并不是调度程序可以选择要运行的新任务的唯一位置。

调度程序还会选择一个新任务，在当前执行的任务进入阻塞状态之后立即运行，或者中断将较高优先级任务移入就绪状态。一个周期性的中断叫做tick中断，用于这个目的。时间片的长度由tick中断频率有效设置，该频率由FreeRTOSConfig.h中应用程序定义的configTICK_RATE_HZ编译时配置常量配置。例如，如果configTICK_RATE_HZ设置为100（Hz），则时间片将为10毫秒。两个滴答中断之间的时间称为滴答周期。一个时间片等于一个时间段。

下图显示了扩展的执行顺序以显示滴答中断执行情况。顶行显示调度程序执行的时间。细箭头显示从任务到滴答中断的执行顺序，然后从滴答中断返回到不同的任务。

configTICK_RATE_HZ的最佳值取决于正在开发的应用程序，但典型值为100。


![](https://github.com/stevewuu/freertos_dg_translate/blob/master/image/Task-5.bmp)


FreeRTOS API调用始终以时间间隔的倍数指定时间，这些时间通常简称为时钟。 pdMS_TO_TICKS（）宏将以毫秒为单位的时间转换为以时钟单位指定的时间。

可用的分辨率取决于定义的节拍频率。 如果打勾频率高于1 KHz（如果configTICK_RATE_HZ大于1000），则不能使用pdMS_TO_TICKS（）。 以下代码显示如何使用pdMS_TO_TICKS（）将指定为200毫秒的时间转换为以时钟单位指定的等效时间。

    /* pdMS_TO_TICKS() takes a time in milliseconds as its only parameter, and evaluates to the
    equivalent time in tick periods. This example shows xTimeInTicks being set to the number
    of tick periods that are equivalent to 200 milliseconds. */
    TickType_t xTimeInTicks = pdMS_TO_TICKS( 200 );


注意：我们不建议您直接在应用程序中指定时间单位。 而应使用pdMS_TO_TICKS（）宏以毫秒为单位指定时间。 这样可以确保在应用程序中指定的时间不会更改，如果更改了打勾频率。

“滴答计数”值是自调度程序启动以来发生的滴答中断总数，假设滴答计数未溢出。 用户应用程序在指定延迟期间时不必考虑溢出，因为时间一致性由FreeRTOS内部管理。

有关影响调度程序何时选择要运行的新任务以及何时执行刻度中断的配置常量的信息，请参阅调度算法。

#### 优先级实验（例3）

调度程序将始终确保能够运行的最高优先级任务是选择进入运行状态的任务。 在目前使用的示例中，已经以相同的优先级创建了两个任务，因此都依次进入和退出Running状态。 本示例显示了示例2中创建的两个任务之一的优先级发生更改时发生的情况。 这一次，第一项任务的优先级为1，第二项优先级为2。

这里显示了以不同优先级创建任务的示例代码。 实现这两个任务的单个功能没有改变。 它仍然只是周期性地打印出一个字符串，使用空循环来创建延迟。

    /* Define the strings that will be passed in as the task parameters. These are defined
    const and not on the stack to ensure they remain valid when the tasks are executing. */
    static const char *pcTextForTask1 = "Task 1 is running\r\n";
    static const char *pcTextForTask2 = "Task 2 is running\r\n";
    int main( void )
    {
    	/* Create the first task at priority 1. The priority is the second to last parameter.
    	*/
    	xTaskCreate( vTaskFunction, "Task 1", 1000, (void*)pcTextForTask1, 1, NULL );
    	/* Create the second task at priority 2, which is higher than a priority of 1. The
    	priority is the second to last parameter. */
    	xTaskCreate( vTaskFunction, "Task 2", 1000, (void*)pcTextForTask2, 2, NULL );
    	/* Start the scheduler so the tasks start executing. */
    	vTaskStartScheduler();
    	/* Will not reach here. */
    	return 0;
    }

这个例子产生的输出如下所示。 调度程序将始终选择能够运行的最高优先级任务。 任务2比任务1具有更高的优先级，并且始终能够运行。 因此，任务2是进入运行状态的唯一任务。 因为任务1从不进入运行状态，所以它从不打印出它的字符串。 据说任务1在任务2中缺乏处理时间。

![](https://github.com/stevewuu/freertos_dg_translate/blob/master/image/Task-6.bmp)

任务2总是能够运行，因为它永远不需要等待任何东西。 它要么绕着空循环旋转，要么打印到终端。

下图显示了上述示例代码的执行顺序。

![](https://github.com/stevewuu/freertos_dg_translate/blob/master/image/Task-7.bmp)

## 扩展非运行状态

到目前为止，创建的任务总是有处理来执行，并且从来不必等待任何事情。
因为他们不需要等待任何事情，他们总是能够进入运行状态。这种类型的连续处理任务的用处有限，因为只能以最低优先级创建任务。如果他们以任何其他优先级运行，他们将阻止任何优先级较低的任务运行。
为了使任务有用，必须将其重写为事件驱动。事件驱动的任务只有在触发事件的事件发生后才能执行（处理），并且在该事件发生之前无法进入运行状态。调度程序始终选择能够运行的最高优先级任务。无法运行的高优先级任务意味着调度程序无法选择它们
并且必须选择能够运行的较低优先级的任务。因此，使用事件驱动的任务意味着可以在不同优先级的情况下创建任务，而不需要优先级最高的任务来处理所有处理时间较低的优先级任务。

### 阻塞状态

据说等待事件的任务处于阻塞状态，该状态是未运行状态的子状态。

任务可以进入阻塞状态以等待两种类型的事件：

1.时间（时间相关）事件，其中事件是延迟期限到期或达到绝对时间。例如，任务可以进入阻塞状态等待10毫秒通过。

2.同步事件，其中事件来自另一个任务或中断。例如，任务可以进入阻塞状态以等待数据到达队列中。同步事件涵盖了广泛的事件类型。

FreeRTOS队列，二进制信号量，计数信号量，互斥量，递归互斥量，事件组，和直接任务通知都可以用来创建同步事件。

任务可以用超时阻塞同步事件，可以同时有效阻止两种类型的事件。例如，任务可以选择等待最多10毫秒的数据到达队列。如果任何数据在10毫秒内到达或10毫秒内没有数据到达，任务将离开阻塞状态。

### 挂起态

暂停也是Not Running的一个子状态。 调度程序不能使用挂起状态中的任务。 进入挂起状态的唯一方法是通过调用vTaskSuspend（）API函数。 悬浮状态的唯一出路是通过调用vTaskResume（）或xTaskResumeFromISR（）API函数。 大多数应用程序不使用暂停状态。

### 就绪态

处于未运行状态但未被阻止或暂停的任务被称为处于就绪状态。 他们能够运行，因此可以运行，但目前不处于运行状态。

### 完成状态转换图

下图展示了简化的状态图，以包含本节中描述的所有未运行子状态。 到目前为止，示例中创建的任务尚未使用“已阻止”或“已挂起”状态。 它们仅在就绪状态和运行状态之间转换，如此处的粗线所示。

![](https://github.com/stevewuu/freertos_dg_translate/blob/master/image/Task-8.bmp)

### 使用阻塞状态建立延时函数（例4）

迄今为止提出的例子中创建的所有任务都是定期的。他们推迟了一段时间并打印出了字符串，再次延迟之前，等等。使用空循环非常粗糙地产生延迟。该任务有效地轮询增量循环计数器，直到达到固定值。例3清楚地表明了这种方法的缺点。优先级较高的任务
在执行空循环时保持在运行状态，使任何处理时间的优先级较低的任务处于饥饿状态。

任何形式的轮询都有其他一些缺点，其中最重要的是效率低下。在轮询期间，任务并没有任何工作要做，但它仍然使用最大的处理时间，因此浪费了处理器周期。示例4通过将调用空循环替换为对vTaskDelay（）API函数的调用来纠正此行为。只有当FreeRTOSConfig.h中的INCLUDE_vTaskDelay设置为1时，vTaskDelay（）API函数才可用。

vTaskDelay（）将调用任务置于阻塞状态，以获得固定数量的节拍中断。任务在处于阻塞状态时不会使用任何处理时间，因此只有在实际工作完成时该任务才会使用处理时间。

此处显示vTaskDelay（）API函数原型。

    void vTaskDelay( TickType_t xTickToDelay );

下表列出了vTaskDelay（）参数。

<table>
<thead>
<tr>
<td>
Parameter_Name 
</td>
<td>
Description
</td>
</tr>
</thead>
<tbody>
<tr>
<td>
xTicksToDelay
</td>
<td>

在转换回就绪状态之前，调用任务将保留在阻塞状态的刻度线中断的数量。

例如，如果在滴答计数为10,000时称为vTaskDelay（100），那么它将立即进入阻塞状态，并保持在阻塞状态，直到滴答计数达到10,100。

宏pdMS_TO_TICKS（）可用于将以毫秒为单位的时间转换为滴答。 例如，调用vTaskDelay（pdMS_TO_TICKS（100））将导致调用任务保持在阻塞状态100毫秒。

</td>
</tr>
</tbody>
</table>

在以下代码中，空循环延迟之后的示例任务已被调用vTaskDelay（）所取代。 此代码显示新的任务定义。

    void vTaskFunction( void *pvParameters )
    {
    	char *pcTaskName;
    	const TickType_t xDelay250ms = pdMS_TO_TICKS( 250 );
    	/* The string to print out is passed in via the parameter. Cast this to a character
    	pointer. */
    	pcTaskName = ( char * ) pvParameters;
    	/* As per most tasks, this task is implemented in an infinite loop. */
    	for( ;; )
    	{
    		/* Print out the name of this task. */
    		vPrintString( pcTaskName );
    		/* Delay for a period. This time a call to vTaskDelay() is used which places the
    		task into the Blocked state until the delay period has expired. The parameter takes a
    		time specified in 'ticks', and the pdMS_TO_TICKS() macro is used (where the xDelay250ms
    		constant is declared) to convert 250 milliseconds into an equivalent time in ticks. */
    		vTaskDelay( xDelay250ms );
    	}
    }

即使这两项任务仍然以不同的优先级创建，两者现在都会运行。 输出确认了预期的行为。

![](https://github.com/stevewuu/freertos_dg_translate/blob/master/image/Task-9.bmp)

下图所示的执行顺序解释了两个任务运行的原因，即使它们是以不同的优先级创建的。 为简单起见，省略了调度程序本身的执行。

空闲任务是在启动调度程序时自动创建的，以确保总是有至少一个能够运行的任务（至少有一个任务处于就绪状态）。 有关此任务的更多信息，请参阅空闲任务和空闲任务挂钩。

该图显示了任务使用vTaskDelay（）代替NULL循环时的执行顺序。

![](https://github.com/stevewuu/freertos_dg_translate/blob/master/image/Task-10.bmp)

只有两项任务的实施已经改变，而不是其功能。 如果将此图与时间测量和嘀嗒中断（第33页）中的数字进行比较，您会发现以更有效的方式实现了此功能。

该图显示了任务在其整个延迟期间进入阻塞状态时的执行模式，因此只有在实际需要执行的工作时才使用处理器时间（在这种情况下，仅打印出消息）。

![](https://github.com/stevewuu/freertos_dg_translate/blob/master/image/Task-11.bmp)

每次任务离开阻塞状态时，它们都会在重新进入阻塞状态之前执行一小部分滴答周期。 大多数情况下，没有任何应用程序能够运行（没有应用程序任务处于“就绪”状态），因此也没有可以选择进入运行状态的应用程序任务。 在这种情况下，空闲任务将运行。 分配给空闲任务的处理时间量是系统中的备用处理能力的量度。 只需通过允许应用程序完全由事件驱动，使用RTOS就可以显着提高备用处理能力。
下图中的粗线显示了示例4中任务执行的转换，现在每个转换都处于阻塞状态，然后返回就绪状态。
粗体线表示由任务执行的状态转换。

![](https://github.com/stevewuu/freertos_dg_translate/blob/master/image/Task-8.bmp)

vTaskDelayUntil() 接口函数

vTaskDelayUntil（）与vTaskDelay（）类似。 vTaskDelay（）参数指定在调用vTaskDelay（）的任务和再次转换到阻塞状态之后的同一任务之间应该发生的滴答中断的数量。 任务保持在阻塞状态的时间长度由vTaskDelay（）参数指定，但任务离开阻塞状态的时间与调用vTaskDelay（）的时间相关。

vTaskDelayUntil（）的参数指定了调用任务应该从阻塞状态进入就绪状态的确切滴答计数值。 vTaskDelayUntil（）是API函数，当需要一个固定的执行周期时（您希望您的任务以固定频率周期性执行）应该使用该函数，因为调用任务被解除阻塞的时间是绝对的，而不是相对于何时 该函数被调用（与vTaskDelay（）的情况一样）。

此处显示vTaskDelayUntil（）API函数原型。

    void vTaskDelayUntil( TickType_t * pxPreviousWakeTime, TickType_t xTimeIncrement );

下表列出了vTaskDelayUntil（）参数

<table>
<thead>
<tr>
<td>
Parameter_Name 
</td>
<td>
Description
</td>
</tr>
</thead>
<tbody>
<tr>
<td>
pxPreviousWakeTime
</td>
<td>

该参数是在假定vTaskDelayUntil（）用于实现周期性执行并具有固定频率的任务的情况下命名的。 在这种情况下，pxPreviousWakeTime保留任务最后离开阻塞状态（被唤醒）的时间。 这个时间用作参考点来计算任务下一次离开阻塞状态的时间。
由pxPreviousWakeTime指向的变量在vTaskDelayUntil（）函数内自动更新。 它通常不会被应用程序代码修改，但必须在首次使用之前将其初始化为当前的滴答计数。

</td>
</tr>

<tr>
<td>
xTimeIncrement
</td>
<td>

该参数也是以vTaskDelayUntil（）被用来实现定期执行并具有固定频率的任务的假定命名的。 频率由xTimeIncrement值设置。
xTimeIncrement以滴答指定。 宏pdMS_TO_TICKS（）可用于将以毫秒为单位的时间转换为滴答。

</td>
</tr>

</tbody>
</table>

### 使用vTaskDelayUntil()转换例程任务

示例4中创建的两个任务是周期性任务，但使用vTaskDelay（）并不能保证它们运行的频率是固定的，因为任务离开阻塞状态的时间与它们调用vTaskDelay（）时的时间有关。 将任务转换为使用vTaskDelayUntil（）而不是vTaskDelay（）可解决此潜在问题。

以下代码显示了使用vTaskDelayUntil（）的示例任务的实现。

    void vTaskFunction( void *pvParameters )
    {
    	char *pcTaskName;
    	TickType_t xLastWakeTime;
    	/* The string to print out is passed in by the parameter. Cast this to a character
    	pointer. */
    	pcTaskName = ( char * ) pvParameters;
    	/* The xLastWakeTime variable needs to be initialized with the current tick count.
    	This is the only time the variable is written to explicitly. After this, xLastWakeTime is
    	automatically updated within vTaskDelayUntil(). */
    	xLastWakeTime = xTaskGetTickCount();
    	/* As per most tasks, this task is implemented in an infinite loop. */
    	for( ;; )
    	{
    		/* Print out the name of this task. */
    		vPrintString( pcTaskName );
    		/* This task should execute every 250 milliseconds exactly. As per the
    		vTaskDelay() function, time is measured in ticks, and the pdMS_TO_TICKS() macro is
    		used to convert milliseconds into ticks. xLastWakeTime is automatically updated within
    		vTaskDelayUntil(), so is not explicitly updated by the task. */
    		vTaskDelayUntil( &xLastWakeTime, pdMS_TO_TICKS( 250 ) );
    	}
    }

## 结合阻塞任务和非阻塞任务（例6）


早些时候的例子已经单独检查了轮询和阻塞任务的行为。 这个例子通过展示两个方案合并时的执行顺序来加强陈述的期望系统行为。

1.以优先级1创建两个任务。这些任务只是连续打印一个字符串。

这些任务从不进行任何可能导致它们进入阻塞状态的API函数调用，因此它们始终处于“就绪”或“正在运行”状态。 像这样的任务被称为连续处理任务，因为他们总是有工作要做（即使在这种情况下它是微不足道的工作）。

2.然后创建优先级为2的第三个任务，因此高于其他两个任务的优先级。 第三个任务也打印出一个字符串，但这次是周期性的，所以它使用vTaskDelayUntil（）API函数将自己置于每次打印迭代之间的Blocked状态。

以下代码显示了连续处理任务。

    void vContinuousProcessingTask( void *pvParameters )
    {
    	char *pcTaskName;
    	/* The string to print out is passed in by the parameter. Cast this to a character
    	pointer. */
    	pcTaskName = ( char * ) pvParameters;
    	/* As per most tasks, this task is implemented in an infinite loop. */
    	for( ;; )
    	{
    		/* Print out the name of this task. This task just does this repeatedly without
    		ever blocking or delaying. */
    		vPrintString( pcTaskName );
    	}
    }
    void vPeriodicTask( void *pvParameters )
    {
    	TickType_t xLastWakeTime;
    	const TickType_t xDelay3ms = pdMS_TO_TICKS( 3 );
    	/* The xLastWakeTime variable needs to be initialized with the current tick count. Note
    	that this is the only time the variable is explicitly written to. After this xLastWakeTime
    	is managed automatically by the vTaskDelayUntil() API function. */
    	xLastWakeTime = xTaskGetTickCount();
    	/* As per most tasks, this task is implemented in an infinite loop. */
    	for( ;; )
    	{
    		/* Print out the name of this task. */
    		vPrintString( "Periodic task is running\r\n" );
    		/* The task should execute every 3 milliseconds exactly. See the declaration of
    		xDelay3ms in this function. */
    		vTaskDelayUntil( &xLastWakeTime, xDelay3ms );
    	}
    }

下图显示了执行顺序。

![](https://github.com/stevewuu/freertos_dg_translate/blob/master/image/Task-12.bmp)

## 空闲任务和空闲任务挂钩

示例4中创建的任务大部分时间都处于阻止状态。在这种状态下，它们无法运行，所以它们不能被调度程序选中。

必须至少有一个任务可以进入运行状态。甚至在使用FreeRTOS的特殊低功耗特性时也是如此。如果应用程序创建的任何任务都无法执行，则FreeRTOS正在执行的微控制器将进入低功耗模式。

要确保至少有一个任务可以进入运行状态，调度程序会在调用vTaskStartScheduler（）时创建一个空闲任务。闲置的任务不仅仅是一个循环，所以像第一个例子中的任务一样，它总是能够运行。

空闲任务具有尽可能最低的优先级（优先级为零），以确保它永远不会阻止较高优先级的应用程序任务进入运行状态。但是，没有什么可以阻止您在空闲任务优先级上创建任务，并因此共享空闲任务优先级。 FreeRTOSConfig.h中的configIDLE_SHOULD_YIELD编译时配置常量可用于防止空闲任务消耗处理时间将更有效地分配给应用程序任务。有关configIDLE_SHOULD_YIELD的更多信息，请参阅调度算法。

以最低优先级运行确保一旦较高优先级的任务进入就绪状态，空闲任务就会从运行状态转换出来。 这可以在示例4的执行序列图中的时间tn处看到，其中空闲任务立即被换出以允许任务2在其离开阻塞状态后立即执行。 据说任务2抢占了空闲任务。 抢占发生在没有被抢占的任务知识的情况下自动发生。

注意：如果应用程序使用vTaskDelete（）API函数，那么空闲任务不会因处理时间而变得非常重要。 这是因为空闲任务负责在删除任务后清理内核资源。

### 空闲任务挂钩函数

您可以通过使用空闲挂钩（或空闲回叫）功能将特定于应用程序的功能直接添加到空闲任务中。 这是空闲任务循环每次迭代一次会自动调用的函数。

空闲任务挂钩的常见用途包括：

- 执行低优先级，后台或连续处理功能。
- 测量备用处理能力的数量。 （空闲任务只有在所有优先级较高的应用程序任务都无法执行时才会运行，因此，测量分配给空闲任务的处理时间量可清楚地表明剩余的处理时间。）
- 将处理器置于低功耗模式下，无需执行任何应用程序处理即可提供简单且自动的方法以节省功耗。 （您可以使用无空闲模式来节省更多电量。）

### 空闲任务挂钩函数的实现限制

空闲任务挂钩功能必须遵守以下规则。

1.空闲任务挂钩功能绝不能试图阻塞或挂起。

注意：以任何方式阻止空闲任务都可能导致没有可用任务进入运行状态的情况。

2.如果应用程序使用vTaskDelete（）API函数，则空闲任务挂钩必须在合理的时间段内始终返回其调用方。 这是因为空闲任务负责在删除任务后清理内核资源。 如果空闲任务永久保留在空闲挂钩功能中，则不能进行清理。

空闲任务挂钩功能必须具有此处显示的名称和原型。

    void vApplicationIdleHook( void );

### 定义空闲任务挂钩函数（例7）

在示例4中使用阻塞vTaskDelay（）API调用创建了大量空闲时间，即执行空闲任务时的时间，因为这两个应用程序任务都处于阻止状态。 例7通过增加一个空闲挂钩函数来利用这个空闲时间，这里显示了它的来源。

以下代码描述了一个非常简单的空闲挂钩功能。


    /* Declare a variable that will be incremented by the hook function.*/
    volatile uint32_t ulIdleCycleCount = 0UL;
    /* Idle hook functions MUST be called vApplicationIdleHook(), take no parameters, and
    return void. */
    void vApplicationIdleHook( void )
    {
    	/* This hook function does nothing but increment a counter. */
    	ulIdleCycleCount++;
    }
    To call the idle
    hook function, calledconfigUSE_IDLE_HOOK must be set to 1 in FreeRTOSConfig.h.

如下所示，实现已创建任务的函数稍作修改以打印出ulIdleCycleCount值。 以下代码显示如何输出ulIdleCycleCount值。

    void vTaskFunction( void *pvParameters )
    {
    	char *pcTaskName;
    	const TickType_t xDelay250ms = pdMS_TO_TICKS( 250 );
    	/* The string to print out is passed in by the parameter. Cast this to a character
    	pointer. */
    	pcTaskName = ( char * ) pvParameters;
    	/* As per most tasks, this task is implemented in an infinite loop. */
    	for( ;; )
    	{
    		/* Print out the name of this task AND the number of times ulIdleCycleCount has
    		been incremented. */
    		vPrintStringAndNumber( pcTaskName, ulIdleCycleCount );
    		/* Delay for a period of 250 milliseconds. */
    		vTaskDelay( xDelay250ms );
    	}
    }

这里显示了示例7生成的输出。 您会看到在应用程序任务的每次迭代之间调用空闲任务挂接函数约为400万次。 （迭代次数取决于执行演示的硬件速度。）

![](https://github.com/stevewuu/freertos_dg_translate/blob/master/image/Task-13.bmp)


## 更改任务优先级

### vTaskPrioritySet() API 函数

调度程序启动后，vTaskPrioritySet（）API函数可用于更改任何任务的优先级。 只有当FreeRTOSConfig.h中的INCLUDE_vTaskPrioritySet设置为1时，vTaskPrioritySet（）API函数才可用。

以下显示vTaskPrioritySet（）API函数原型。

    void vTaskPrioritySet( TaskHandle_t pxTask, UBaseType_t uxNewPriority);

下表列举vTaskPrioritySet()参数

<table>
<thead>
<tr>
<td>
Parameter_Name 
</td>
<td>
Description
</td>
</tr>
</thead>
<tbody>
<tr>
<td>
pxTask
</td>
<td>

正在修改优先级的任务的句柄（主题任务）。 有关获取任务句柄的信息，请参阅xTaskCreate（）API函数的pxCreatedTask参数。
任务可以通过传递NULL来代替有效的任务句柄来改变自己的优先级。

</td>
</tr>

<tr>
<td>
uxNewPriority
</td>
<td>

主题任务设置的优先级。
这会自动上限为（configMAX_PRIORITIES - 1）的最大可用优先级，其中configMAX_PRIORITIES是FreeRTOSConfig.h头文件中设置的编译时常量。

</td>
</tr>

</tbody>
</table>


### uxTaskPriorityGet() API Function

uxTaskPriorityGet（）API函数可用于查询任务的优先级。 只有在FreeRTOSConfig.h中将INCLUDE_uxTaskPriorityGet设置为1时，uxTaskPriorityGet（）API函数才可用。

以下显示了uxTaskPriorityGet（）API函数原型。

    UBaseType_t uxTaskPriorityGet( TaskHandle_t pxTask );

下表列出了uxTaskPriorityGet（）参数和返回值。

<table>
<thead>
<tr>
<td>
Parameter_Name/Return_value
</td>
<td>
Description
</td>
</tr>
</thead>
<tbody>
<tr>
<td>
pxTask
</td>
<td>

正在查询优先级的任务的句柄（主题任务）。 有关获取任务句柄的信息，请参阅xTaskCreate（）API函数的pxCreatedTask参数。
任务可以通过传递NULL来代替有效的任务句柄来查询自己的优先级。

</td>
</tr>

<tr>
<td>
Returned value
</td>
<td>

当前分配给正在查询的任务的优先级。

</td>
</tr>

</tbody>
</table>


### 更改任务优先级（例8）

调度程序将始终选择最高就绪状态任务作为进入运行状态的任务。

示例8通过使用vTaskPrioritySet（）API函数来演示这一点，以更改两个任务相对于彼此的优先级。

这里使用的示例代码以两个不同的优先级创建两个任务。这两个任务都不会进行任何可能导致其进入阻塞状态的API函数调用，因此它们都始终处于“就绪”状态或“正在运行”状态。因此，相对优先级最高的任务将始终是调度程序选择的处于“运行”状态的任务。

1.任务1（显示在下面的代码中）以最高优先级创建，因此保证首先运行。任务1打印出一串字符串，然后将任务2（显示第二个）的优先级提高到高于其自己的优先级。

2.任务2具有最高的相对优先级后，即开始运行（进入运行状态）。任何时候只有一个任务处于运行状态，因此当任务2处于运行状态时，任务1处于就绪状态。

3.任务2在将自己的优先级降低到低于任务1的优先级之前打印出消息。

4.当任务2将其优先级降低时，任务1再次成为最高优先级任务。任务1重新进入运行状态，强制任务2回到就绪状态。

    /*The implementation of Task 1*/
    void vTask1( void *pvParameters )
    {
    	UBaseType_t uxPriority;
    	/* This task will always run before Task 2 because it is created with the higher
    	priority. Task 1 and Task 2 never block, so both will always be in either the Running or
    	the Ready state. Query the priority at which this task is running. Passing in NULL means
    	"return the calling task's priority". */
    	uxPriority = uxTaskPriorityGet( NULL );
    	for( ;; )
    	{
    		/* Print out the name of this task. */
    		vPrintString( "Task 1 is running\r\n" );
    		/* Setting the Task 2 priority above the Task 1 priority will cause Task 2 to
    		immediately start running (Task 2 will have the higher priority of the two created tasks).
    		Note the use of the handle to Task 2 (xTask2Handle) in the call to vTaskPrioritySet(). The
    		code that follows shows how the handle was obtained. */
    		vPrintString( "About to raise the Task 2 priority\r\n" );
    		vTaskPrioritySet( xTask2Handle, ( uxPriority + 1 ) );
    		/* Task 1 will only run when it has a priority higher than Task 2. Therefore, for
    		this task to reach this point, Task 2 must already have executed and set its priority back
    		down to below the priority of this task. */
    	}
    }
.

    /*The implementation of Task 2 */
    void vTask2( void *pvParameters )
    {
    	UBaseType_t uxPriority;
    	/* Task 1 will always run before this task because Task 1 is created with the higher
    	priority. Task 1 and Task 2 never block so they will always be in either the Running or
    	the Ready state. Query the priority at which this task is running. Passing in NULL means
    	"return the calling task's priority". */
    	uxPriority = uxTaskPriorityGet( NULL );
    	for( ;; )
    	{
    	/* For this task to reach this point Task 1 must have already run and set the
    		priority of this task higher than its own. Print out the name of this task. */
    		vPrintString( "Task 2 is running\r\n" );
    		/* Set the priority of this task back down to its original value. Passing in NULL
    		as the task handle means "change the priority of the calling task". Setting the priority
    		below that of Task 1 will cause Task 1 to immediately start running again, preempting this
    		task. */
    		vPrintString( "About to lower the Task 2 priority\r\n" );
    		vTaskPrioritySet( NULL, ( uxPriority - 2 ) );
    	}
    }

每个任务都可以通过简单地使用NULL来查询和设置自己的优先级，而无需使用有效的任务句柄。 只有当任务引用除自身以外的任务时才需要任务句柄，例如当任务1更改任务2的优先级时。要允许任务1执行此任务，将创建任务2时获取并保存任务2句柄， 如代码注释中突出显示的那样。

    /* Declare a variable that is used to hold the handle of Task 2. */
    TaskHandle_t xTask2Handle = NULL;
    int main( void )
    {
	    /* Create the first task at priority 2. The task parameter is not used and set to NULL.
	    The task handle is also not used so is also set to NULL. */
	    TaskCreate( vTask1, "Task 1", 1000, NULL, 2, NULL );
	    /* The task is created at priority 2 ______^. */
	    /* Create the second task at priority 1, which is lower than the priority given to
	    Task 1. Again, the task parameter is not used so it is set to NULL, but this time the task
	    handle is required so the address of xTask2Handle is passed in the last parameter. */
	    xTaskCreate( vTask2, "Task 2", 1000, NULL, 1, &xTask2Handle );
	    /* The task handle is the last parameter _____^^^^^^^^^^^^^ */
	    /* Start the scheduler so the tasks start executing. */
	    vTaskStartScheduler();
	    /* If all is well, then main() will never reach here because the scheduler will now be
	    running the tasks. If main() does reach here, then it is likely there was insufficient
	    heap memory available for the idle task to be created. For information, see Heap Memory
	    Management. */
	    for( ;; );
    }

下图显示了任务执行的顺序。

![](https://github.com/stevewuu/freertos_dg_translate/blob/master/image/Task-14.bmp)

输出显示在这里。

![](https://github.com/stevewuu/freertos_dg_translate/blob/master/image/Task-15.bmp)

## 删除任务

### vTaskDelete() API Function

任务可以使用vTaskDelete（）API函数来删除自己或任何其他任务。 只有当FreeRTOSConfig.h中的INCLUDE_vTaskDelete设置为1时，vTaskDelete（）API函数才可用。
已删除的任务不再存在，无法再次进入运行状态。
释放分配给已被删除的任务的内存是空闲任务的责任。
因此，使用vTaskDelete（）API函数的应用程序不会完全停止所有处理时间的空闲任务，这一点很重要。
注意：只有内核分配给任务的内存将在任务被删除时自动释放。 任何实现该任务分配的内存或其他资源必须明确释放。
这里显示了vTaskDelete（）API函数的原型。

void vTaskDelete( TaskHandle_t pxTaskToDelete );

下表列出了vTaskDelete（）参数。

<table>
<thead>
<tr>
<td>
Parameter_Name
</td>
<td>
Description
</td>
</tr>
</thead>
<tbody>
<tr>
<td>
pxTaskToDelete
</td>
<td>

要删除的任务的句柄（主题任务）。 有关获取任务句柄的信息，请参阅xTaskCreate（）API函数的pxCreatedTask参数。 任务可以通过传递NULL来代替有效的任务句柄来删除自己。

</td>
</tr>


</tbody>
</table>

### 删除任务（例9）

这是一个非常简单的例子。
1.任务1由优先级为1的main（）创建。当它运行时，它创建优先级为2的任务2.任务2现在是最高优先级任务，因此它立即开始执行。 main（）的源代码显示在第一个代码清单中。任务1的源代码显示在第二个代码清单中。
2.任务2除了删除本身外什么也不做。它可以通过将NULL传递给vTaskDelete（）来删除自身，但为了演示目的，它使用自己的任务句柄。任务2的源代码显示在第三个代码清单中。
3.当任务2被删除时，任务1又是最高优先级的任务，所以继续执行，此时它调用vTaskDelay（）在短时间内阻塞。
4.空闲任务在任务1处于阻塞状态时执行，并释放分配给现在已删除的任务2的内存。
5.当任务1离开阻塞状态时，它再次成为最高优先级的就绪状态任务，因此抢占空闲任务。当它进入运行状态时，它会再次创建任务2，并继续进行。

    int main( void )
    {
	    /* Create the first task at priority 1. The task parameter is not used so is set to
	    NULL. The task handle is also not used so likewise is set to NULL. */
	    xTaskCreate( vTask1, "Task 1", 1000, NULL, 1, NULL );
	    /* The task is created at priority 1 ______^. */
	    /* Start the scheduler so the task starts executing. */
	    vTaskStartScheduler();
	    /* main() should never reach here as the scheduler has been started.*/
	    for( ;; );
    }

.

    TaskHandle_t xTask2Handle = NULL;
    void vTask1( void *pvParameters )
    {
    	const TickType_t xDelay100ms = pdMS_TO_TICKS( 100UL );
    	for( ;; )
    	{
    		/* Print out the name of this task. */
    		vPrintString( "Task 1 is running\r\n" );
    		/* Create task 2 at a higher priority. Again, the task parameter is not used so it
    		is set to NULL, but this time the task handle is required so the address of xTask2Handle
    		is passed as the last parameter. */
    		xTaskCreate( vTask2, "Task 2", 1000, NULL, 2, &xTask2Handle );
    		/* The task handle is the last parameter _____^^^^^^^^^^^^^ */
    		/* Task 2 has/had the higher priority, so for Task 1 to reach here, Task 2 must
    		have already executed and deleted itself. Delay for 100 milliseconds. */
    		vTaskDelay( xDelay100ms );
    	}
    }

.

    void vTask2( void *pvParameters )
    {
    	/* Task 2 does nothing but delete itself. To do this, it could call vTaskDelete() using
    	NULL as the parameter, but for demonstration purposes, it calls vTaskDelete(), passing its
    	own task handle. */
    	vPrintString( "Task 2 is running and about to delete itself\r\n" );
    	vTaskDelete( xTask2Handle );
    }

输出显示在这里。

![](https://github.com/stevewuu/freertos_dg_translate/blob/master/image/Task-16.bmp)

下图显示了执行顺序。

![](https://github.com/stevewuu/freertos_dg_translate/blob/master/image/Task-17.bmp)

## 调度算法

### 任务状态和事件的回顾

实际运行的任务（使用处理时间）处于运行状态。在单个核心处理器上，任何时候只能有一个处于运行状态的任务。

没有运行但未处于阻塞状态或暂停状态的任务处于就绪状态。处于就绪状态的任务可供调度程序选择为进入运行状态的任务。调度程序将始终选择最高优先级的就绪状态任务来进入运行状态。

任务可以在事件阻塞状态下等待，并在事件发生时自动移回就绪状态。时间事件发生在特定时间（例如，阻塞时间到期）并且通常用于实现定期或超时行为。当任务或中断服务例程使用任务通知，队列，事件组或多种信号类型之一发送信息时，会发生同步事件。它们通常用于发送同步活动信号，例如到达外设的数据。

### 配置调度算法

调度算法是决定哪个Ready状态任务转换到Running状态的软件例程。

到目前为止所显示的所有示例都使用相同的调度算法，但您可以使用configUSE_PREEMPTION和configUSE_TIME_SLICING配置常量来更改它。两个常量都在FreeRTOSConfig.h中定义。

第三个配置常量configUSE_TICKLESS_IDLE也会影响调度算法。

它的使用会导致滴答中断在很长一段时间内完全关闭。

configUSE_TICKLESS_IDLE是一个高级选项，专门用于必须将功耗降至最低的应用程序。本节中提供的描述假设configUSE_TICKLESS_IDLE被设置为0，如果常量未定义，这是默认设置。

在所有可能的配置中，FreeRTOS调度程序将确保共享优先级的任务依次进入运行状态。这种反转策略通常被称为循环调度。循环调度算法不保证时间在同等优先级的任务之间平均分配，只有等优先级的就绪状态任务依次进入运行状态。

### 带时间片的优先抢占式调度

下表中显示的配置将FreeRTOS调度程序设置为使用称为固定优先级抢占调度与时间切片的调度算法，该调度算法是大多数小型RTOS应用程序使用的调度算法，以及迄今为止提供的所有示例所用的算法。


<table>
<thead>
<tr>
<td>
Constant
</td>
<td>
Value
</td>
</tr>
</thead>
<tbody>
<tr>
<td>
configUSE_PREEMPTION
</td>
<td>
1
</td>
</tr>
<tr>
<td>
configUSE_TIME_SLICING
</td>
<td>
1
</td>
</tr>

</tbody>
</table>

这里是算法中使用的术语：

- **固定优先级任务**：被描述为固定优先级的调度算法不会改变分配给正在调度的任务的优先级，但也不会阻止任务本身改变其优先级或其他任务的优先级。
- **抢占状态**：如果具有较高优先级的任务进入就绪状态，抢先调度算法将立即抢占正在运行的状态任务。被抢占意味着被非意愿地移动（没有明确地让步或阻止）跑出状态并进入就绪状态以允许不同的任务进入跑步状态。
- **时间分片**：时间分片用于在优先级相同的任务之间共享处理时间，即使这些任务没有明确放弃或进入阻止状态。如果存在与Running任务具有相同优先级的其他Ready状态任务，则描述为使用时间片的调度算法将选择一个新任务，以在每个时间片结束时进入Running状态。时间片等于两个RTOS滴答中断之间的时间。

下图显示了当应用程序中的所有任务具有唯一优先级时，选择任务以进入运行状态的顺序。

![](https://github.com/stevewuu/freertos_dg_translate/blob/master/image/Task-18.bmp)

该图显示了执行模式，突出显示了假设应用程序中的任务优先级和占先情况，其中每个任务都被分配了一个唯一的优先级。

1.空闲任务

空闲任务以最低优先级运行，因此每次优先级较高的任务进入就绪状态时（例如，在时间t3，t5和t9），都会被抢先占用。

2.任务3

任务3是一个事件驱动的任务，它以较低的优先级执行，但高于空闲优先级。
它大部分时间都处于阻塞状态，等待其感兴趣的事件，每次事件发生时都从阻塞状态转换到就绪状态。所有FreeRTOS任务间通信机制（任务通知，队列，信号量，事件组等）均可用于以这种方式表示事件并解除任务。
事件发生在时间t3和t5以及t9和t12之间。在时间t3和t5发生的事件会立即处理，因为在这些时间任务3是能够运行的最高优先级任务。发生在时间t9和t12之间某处的事件直到t12才被处理，因为直到此时为止，任务1和任务2的优先级更高的任务仍在执行。仅在时间t12，任务1和任务2都处于阻塞状态，使任务3成为最高优先级
就绪状态任务。

3.任务2

任务2是周期性任务，其执行的优先级高于任务3的优先级，但低于任务1的优先级。任务的周期间隔意味着任务2希望在时间t1，t6和t9执行。在时间t6，任务3处于运行状态，但任务2具有较高的相对优先级，因此它抢占任务3并立即开始执行。任务2完成其处理并在时间t7重新进入阻塞状态，此时任务3可重新进入运行状态以完成其处理。任务3在时间t8处阻止。

4.任务1

任务1也是一个事件驱动的任务。它以最高优先级执行，因此它可以抢占系统中的任何其他任务。所示的唯一任务1事件发生在时间t10，此时任务1抢占任务2.任务2只有在任务1在时间t11重新进入阻塞状态后才能完成其处理。

下图显示了在两个任务以相同优先级运行的假设应用程序中执行模式，突出显示任务优先级和时间分片。

![](https://github.com/stevewuu/freertos_dg_translate/blob/master/image/Task-19.bmp)

1.空闲任务和任务2

空闲任务和任务2都是连续处理任务，并且都具有0（优先级最低）的优先级。当没有更高优先级的任务可以运行时，调度程序仅为优先级为0的任务分配处理时间，并通过时间分片共享分配给优先级为0的任务的时间。一个新的时间片开始于每个时间片中断，即时间t1，t2，t3，t4，t5，t8，t9，t10和t11。
空闲任务和任务2依次进入运行状态，这可能会导致两个任务处于同一时间片部分的运行状态，就像在时间t5和时间t8之间发生的那样。

2.任务1

任务1的优先级高于空闲优先级。任务1是一个事件驱动的任务，其大部分时间都处于阻塞状态，等待其感兴趣的事件，每次事件发生时都从阻塞状态转换到就绪状态。感兴趣的事件发生在时间t6，当任务1成为能够运行的最高优先级任务时，因此任务1通过时间片部分抢占空闲任务。事件处理在时间t7完成，此时任务1重新进入
被阻止的状态。
上图显示了应用程序编写器创建的任务与空闲任务共享处理时间。如果应用程序编写器创建的空闲优先级任务需要完成，您可能不想为空闲任务分配太多处理时间，但空闲任务不需要。您可以使用configIDLE_SHOULD_YIELD编译时配置常量来更改空闲任务的计划方式：

- 如果configIDLE_SHOULD_YIELD设置为0，则空闲任务将在整个时间片内保持运行状态，除非它被更高优先级的任务抢占。
- 如果configIDLE_SHOULD_YIELD设置为1，那么如果在就绪状态下有其他空闲优先级任务，则空闲任务将在其循环的每次迭代中放弃（自愿放弃其分配的时间片剩余部分）。


上图中显示的执行模式是当configIDLE_SHOULD_YIELD设置为0时将会观察到的执行模式。
下图中显示的执行模式是在将configIDLE_SHOULD_YIELD设置为1时的相同场景中可以观察到的执行模式。它显示了在应用程序中的两个任务共享优先级时选择任务以进入运行状态的顺序。

![](https://github.com/stevewuu/freertos_dg_translate/blob/master/image/Task-20.bmp)

该图还显示，当configIDLE_SHOULD_YIELD设置为0时，在空闲任务后选择进入运行状态的任务不会针对整个时间片执行，而是针对空闲任务产生的时间片剩余时间执行任务。

### 优先抢先调度（无时间片）

没有时间分片的优先抢先调度保持了前一节中描述的相同的任务选择和抢占算法，但它不使用时间分片来在相同优先级的任务之间共享处理时间。
下表列出了FreeRTOSConfig.h设置，它们将FreeRTOS调度程序配置为使用优先抢先调度而不用时间分片。

<table>
<thead>
<tr>
<td>
Constant
</td>
<td>
Value
</td>
</tr>
</thead>
<tbody>
<tr>
<td>
configUSE_PREEMPTION
</td>
<td>
1
</td>
</tr>
<tr>
<td>
configUSE_TIME_SLICING
</td>
<td>
0
</td>
</tr>

</tbody>
</table>

如果使用时间分片并且有多个能够运行的最高优先级的就绪状态任务，则调度程序将在每个RTOS滴答中断期间选择一个新任务以进入运行状态（一个滴答中断表示标记结束一个时间片）。如果没有使用时间分片，则只有在以下情况下，调度程序才会选择一个新任务进入运行状态：

- 更高优先级的任务进入就绪状态。
- 处于运行状态的任务进入阻塞或挂起状态。


如果不使用时间片，则任务上下文切换的次数会减少。因此，关闭时间分片会减少调度程序的处理开销。但是，关闭时间切割也会导致相同优先级的任务接收到非常不同的处理时间量。无时间片运行调度程序被认为是一种先进的技术。它只应由有经验的用户使用。
该图显示了当不使用时间分片时，同等优先级的任务如何能够接收到非常不同的处理时间量。

![](https://github.com/stevewuu/freertos_dg_translate/blob/master/image/Task-21.bmp)

在上图中，configIDLE_SHOULD_YIELD设置为0。

1.打勾中断

在时间t1，t2，t3，t4，t5，t8，t11，t12和t13发生嘀嗒中断。

2.任务1

任务1是高优先级，事件驱动的任务，其大部分时间处于阻塞状态，等待其感兴趣的事件。每当事件发生时，任务1从阻塞状态转换到就绪状态（随后，因为它是最高优先级的就绪状态任务，进入运行状态）。
上图显示任务1处理时间t6和t7之间的事件，然后再次处理时间t9和t10。

3.空闲任务和任务2

空闲任务和任务2都是连续处理任务，并且都具有0（空闲优先级）的优先级。连续处理任务不会进入阻止状态。
时间分片未被使用，因此处于运行状态的空闲优先级任务将保持运行状态，直到它被优先级更高的任务1抢占。
在上图中，空闲任务在时间t1开始运行并保持运行状态，直到它在时间t6被任务1抢占，这是在进入运行状态后多于四个完整的时间周期。
任务2在时间t7开始运行，这是任务1重新进入阻塞状态以等待另一个事件的时间。任务2保持运行状态，直到它在时间t9被任务1抢占为止，该时间小于进入运行状态后的一个时间周期。
在时间t10，空闲任务重新进入运行状态，尽管已经接收比任务2多四倍的处理时间。

### 合作计划

FreeRTOS也可以使用协作式调度。 下表列出了配置FreeRTOS调度程序以使用协作调度的FreeRTOSConfig.h设置。

<table>
<thead>
<tr>
<td>
Constant
</td>
<td>
Value
</td>
</tr>
</thead>
<tbody>
<tr>
<td>
configUSE_PREEMPTION
</td>
<td>
0
</td>
</tr>
<tr>
<td>
configUSE_TIME_SLICING
</td>
<td>
Any value
</td>
</tr>

</tbody>
</table>

使用协作式调度程序时，只有在运行状态任务进入阻塞状态或运行状态任务通过调用taskYIELD（）明确产生（手动请求重新调度）时才会发生上下文切换。 任务永远不会被抢占，所以时间分片无法使用。

下图显示了协作式调度程序的行为。 水平虚线显示任务何时处于就绪状态。

![](https://github.com/stevewuu/freertos_dg_translate/blob/master/image/Task-22.bmp)

1.任务1

任务1具有最高优先级。它从Blocked状态开始，等待信号量。
在时间t3，中断给出信号量，导致任务1离开阻塞状态并进入就绪状态。有关从中断给信号量的信息，请参阅中断管理（p。120）。
在时间t3，任务1是最高优先级的就绪状态任务，并且如果已使用抢先调度程序，则任务1将成为正在运行状态任务。但是，因为正在使用协作调度程序，所以任务1保持在就绪状态，直到时间t4，即运行状态任务调用taskYIELD（）时为止。

2.任务2

任务2的优先级在任务1和任务3的优先级之间。它在阻塞状态下启动，等待任务3在时间t2发送给它的消息。
在时间t2，任务2是最高优先级的就绪状态任务，如果已使用抢先调度程序，则任务2将成为正在运行状态任务。但是，因为正在使用协作调度程序，所以任务2保持就绪状态，直到正在运行状态任务进入阻塞状态或调用taskYIELD（）。
运行状态任务在时间t4调用taskYIELD（），但此时任务1是最高优先级的就绪状态任务，因此任务2实际上不会成为运行状态任务，直到任务1在时间t5重新进入阻塞状态。
在时间t6，任务2重新进入阻塞状态以等待下一个消息，此时任务3再次是最高优先级的就绪状态任务。

在多任务应用程序中，应用程序编写人员必须注意，资源不能同时被多个任务访问，因为同时访问会损坏资源。 考虑以下情况，其中访问的资源是UART（串行端口）。 两项任务正在撰写字符串到UART。 任务1正在写入“abcdefghijklmnop”，任务2正在写入“123456789”：

1.任务1处于运行状态并开始写入其字符串。 它向UART写入“abcdefg”，但在写入任何其他字符之前保留运行状态。

2.任务2进入运行状态并在离开运行状态之前将“123456789”写入UART。

3.任务1重新进入运行状态并将其字符串的其余字符写入UART。

在这种情况下，实际写入UART的是“abcdefg123456789hijklmnop”。任务1写入的字符串没有按照预期的顺序以完整的顺序写入UART。相反，它已被破坏，因为任务2写入UART的字符串出现在其中。

您可以避免使用协作调度程序的同时访问导致的问题。本指南稍后会介绍在任务之间安全共享资源的方法。 FreeRTOS提供的资源，例如队列和信号量，在任务之间共享是安全的。

- 使用抢先式调度程序时，运行状态任务可随时被抢占，包括与其他任务共享的资源处于不一致状态时。如UART示例所示，将资源置于不一致状态可能导致数据损坏。
- 使用协作式调度程序时，应用程序写入程序控制何时可以切换到其他任务。因此，应用程序编写器可确保在资源处于不一致状态时切换到其他任务。
- 在UART示例中，应用程序编写器可以确保任务1在其整个字符串已写入UART之前不会保持运行状态，并且这样做可以消除由于激活其他任务而导致字符串损坏的可能性。
使用合作调度程序时，系统的响应速度会降低。
- 使用抢先式调度程序时，调度程序将在任务成为最高优先级的就绪任务后立即开始运行任务。对于必须在规定的时间段内响应高优先级事件的实时系统来说，这是非常重要的。
- 使用协作式调度程序时，直到Running state任务进入阻塞状态或调用taskYIELD（），切换到已成为最高优先级的就绪状态任务的任务才会执行。



