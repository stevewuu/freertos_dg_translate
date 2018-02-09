## 堆内存管理

从V9.0.0开始，FreeRTOS应用程序可以完全静态分配，这意味着不需要包含堆内存管理器。本节涵盖：

- FreeRTOS分配RAM时。
- FreeRTOS包含五个示例内存分配方案。
- 每种内存分配方案的用例。
F
### 先决条件

你需要很好的理解C编程才能使用FreeRTOS。 具体来说，你应该熟悉：

- 如何构建C项目，包括编译和链接阶段。
- 堆栈和堆的概念。
- 标准的C库malloc（）和free（）函数。

### 动态内存分配及其与FreeRTOS的关联

本指南介绍了诸如任务，队列，信号量和事件组等内核对象。 要使FreeRTOS尽可能易于使用，这些内核对象在编译时不是静态分配的，而是在运行时动态分配的。 每次创建内核对象时，FreeRTOS都会分配RAM，并且每次内核对象被删除时都会释放RAM。 此策略减少了设计和规划工作，简化了API，并最大限度地减少了RAM占用空间。

动态内存分配是一个C编程概念。 这不是FreeRTOS或多任务特有的概念。 这与FreeRTOS相关，因为内核对象是动态分配的，而通用编译器提供的动态内存分配方案并不总是适用于实时应用。

可以使用标准C库malloc（）和free（）函数来分配内存，但可能由于以下一个或多个原因而不适合或不适用：

- 它们并不总是可用于小型嵌入式系统。
- 它们的实现可能相对较大，占用有价值的代码空间。
- 它们很少是线程安全的。
- 他们不是确定性的。 执行这些功能所用的时间将与调用有所不同。
- 他们可能遭受分裂。 如果将堆中的空闲RAM分解为彼此分离的小块，则认为该堆被分割。 如果堆被分段，那么如果堆中没有单个空闲块足够大以包含块，那么分配块的尝试将失败，即使堆中所有单独的空闲块的总大小比 无法分配的块的大小。
- 他们可能会使链接器配置复杂化。
- 如果允许堆空间增长到其他变量使用的内存中，则它们可能成为难以调试的错误源。

### 动态内存分配选项

较早版本的FreeRTOS使用内存池分配方案，其中不同大小内存块的池在编译时预先分配，然后由内存分配函数返回。尽管这是实时系统中常用的方案，但却产生了很多支持请求。该方案因为无法有效地使用RAM而使其成为真正的小型嵌入式系统而被放弃。

FreeRTOS现在将内存分配作为便携层的一部分（而不是核心代码库的一部分）。这是对嵌入式系统动态内存分配和时序要求的认识。单个动态内存分配算法仅适用于一部分应用程序。此外，从核心代码库中移除动态内存分配，可以使应用程序编写者在适当时提供自己的特定实现。

当FreeRTOS需要RAM时，它调用pvPortMalloc（）而不是malloc（）。当RAM被释放时，内核调用vPortFree（）而不是free（）。 pvPortMalloc（）与标准C库malloc（）函数具有相同的原型。 vPortFree（）与标准C库free（）函数具有相同的原型。

pvPortMalloc（）和vPortFree（）是公共函数，所以它们也可以从应用程序代码中调用。

FreeRTOS提供了pvPortMalloc（）和vPortFree（）的五个示例实现，所有这些在这里都有记录。 FreeRTOS应用程序可以使用这些示例实现之一或提供自己的。

这五个例子在位于FreeRTOS / Source / portable / MemMang目录中的heap_1.c，heap_2.c，heap_3.c，heap_4.c和heap_5.c源文件中定义。

#### 内存分配示例

由于FreeRTOS应用程序可以完全静态分配，因此不需要包含堆内存管理器。

### Heap_1

小型专用嵌入式系统通常只在调度程序启动之前创建任务和其他内核对象。在应用程序开始执行任何实时功能之前，内存是由内核动态分配的，并且内存在应用程序的整个生命周期中都保持分配状态。这意味着所选择的分配方案不必考虑任何更复杂的存储器分配问题，例如确定性和分段。它可以考虑代码大小和简单性等属性。

Heap_1.c实现了一个非常基本的pvPortMalloc()。它没有实现vPortFree()。从不删除任务或其他内核对象的应用程序可以使用heap_1。

一些否则会禁止使用动态内存分配的商业关键和安全关键系统也可以使用heap_1。关键系统通常禁止动态内存分配，因为与非确定性，内存碎片和分配失败相关的不确定性，但heap_1总是确定性的，不能片段化内存。

当调用pvPortMalloc（）时，heap_1分配方案将一个简单的数组细分为更小的块。该数组被称为FreeRTOS堆。

数组的总大小（以字节为单位）由FreeRTOSConfig.h中的定义configTOTAL_HEAP_SIZE设置。以这种方式定义大型数组可能会使应用程序看起来消耗大量RAM，即使在从阵列分配任何内存之前。

每个创建的任务都需要一个任务控制块（TCB）和一个从堆中分配的堆栈。

下图显示了在创建任务时heap_1如何细分简单数组。 每次创建任务时，都会从heap_1数组中分配RAM。

![](https://github.com/stevewuu/freertos_dg_translate/blob/master/image/Heap-1.bmp)

- A在任何任务创建之前显示数组。 整个阵列是免费的。
- B显示创建一个任务后的数组。
- C显示创建三个任务后的数组。

### Heap_2

FreeRTOS发行版中包含Heap_2以实现向后兼容。不建议用于新设计。考虑使用heap_4，因为它提供了更多的功能。

Heap_2.c也可以通过细分一个由configTOTAL_HEAP_SIZE标注的数组来工作。它使用最适合的算法来分配内存。与heap_1不同，它允许释放内存。同样，数组是静态声明的，所以应用程序似乎消耗了大量的RAM，即使在阵列中已经分配了任何内存之前。

最佳拟合算法可确保pvPortMalloc（）使用与请求的字节数最接近的空闲内存块。例如，考虑以下情况：

- 堆包含三个空闲内存块，分别为5个字节，25个字节和100个字节。
- 调用pvPortMalloc（）来请求20个字节的RAM。

所请求的字节数所能容纳的最小空闲块是25字节的块，所以pvPortMalloc（）将25字节块分成一个20字节的块和一个5字节的块，然后返回一个指针20字节块。 （这是过于简单化了，因为heap_2在堆区域中存储块大小的信息，所以两个拆分块的总和实际上将小于25.）新的5字节块仍然可用于未来调用pvPortMalloc（） 。

与heap_4不同，heap_2不会将相邻的空闲块合并为一个较大的块。出于这个原因，它更容易分裂。但是，如果被分配和随后释放的块总是相同的大小，则碎片不是问题。 Heap_2适用于重复创建和删除任务的应用程序，只要分配给创建任务的堆栈的大小可以不变。

下图显示了正在创建和删除任务的RAM正在分配并从heap_2阵列中释放。 

![](https://github.com/stevewuu/freertos_dg_translate/blob/master/image/Heap-2.bmp)

该图显示了创建，删除和重新创建任务时最佳拟合算法的工作原理。

- A显示创建了三个任务后的阵列。 一个大的空闲块保留在数组的顶部。
- B显示了其中一个任务被删除后的阵列。 数组顶部的大型空闲块仍然存在。 现在还有两个较小的空闲块被分配给TCB和被删除的任务的堆栈。
- C显示创建另一个任务后的数组。 创建任务导致两个调用pvPortMalloc（）：一个分配一个新的TCB，一个分配任务堆栈。 任务使用xTaskCreate（）API函数创建，这在创建任务（p。27）中进行了介绍。 对pvPortMalloc（）的调用发生在xTaskCreate（）内部。

每个TCB的大小完全相同，所以最佳拟合算法确保先前分配给被删除任务的TCB的RAM块被重新用于分配新任务的TCB。

分配给新创建的任务的堆栈的大小与分配给先前删除的任务的堆栈的大小相同，因此最佳拟合算法确保先前分配给已删除任务的堆栈的RAM块被重用以分配堆栈 新任务。

数组顶部较大的未分配块保持不变。

Heap_2不是确定性的，但它比大多数malloc（）和free（）的标准库实现更快。

### Heap_3

Heap_3.c使用标准库malloc（）和free（）函数，所以堆的大小由链接器配置来定义。 configTOTAL_HEAP_SIZE设置不起作用。
Heap_3通过临时挂起FreeRTOS调度器使malloc（）和free（）线程安全。 有关线程安全性和调度程序挂起的信息，请参阅资源管理（p。151）一节。

### Heap_4
像heap_1和heap_2一样，heap_4将数组细分为更小的块。 该数组是静态声明和由configTOTAL_HEAP_SIZE维度，所以应用程序似乎消耗了大量的内存，甚至在从阵列分配任何内存之前。

Heap_4使用第一个拟合算法来分配内存。 与heap_2不同的是，它将相邻的空闲内存块组合（合并）成一个较大的块。 这最大限度地减少了内存碎片的风险。

- 第一个拟合算法确保pvPortMalloc（）使用足够大的第一个空闲内存块来容纳请求的字节数。 例如，考虑以下情况：
- 堆包含三块可用内存。 它们在数组中以5个字节，200个字节和100个字节的顺序出现。
调用pvPortMalloc（）来请求20个字节的RAM。

所请求的字节数量的第一个空闲块是200字节的块，所以pvPortMalloc（）把200字节的块分成一个20字节的块和一个180字节的块，然后返回一个指针 20字节块。 （这是一个过分简化，因为heap_4在堆区域内存储块大小的信息，所以两个拆分块的总和将小于200字节。）

新的180字节块仍然可用于未来调用pvPortMalloc（）。

Heap_4将相邻的空闲块组合（合并）成一个较大的块，从而最大限度地减少了碎片的风险。 Heap_4适用于重复分配和释放不同大小的RAM块的应用程序。

下图显示了正在分配并从heap_4阵列中释放的RAM。 它演示了heap_4与内存合并的第一个拟合算法是如何工作的，因为内存被分配和释放。

![](https://github.com/stevewuu/freertos_dg_translate/blob/master/image/Heap-4.bmp)

A显示了创建三个任务后的数组。 一个大的空闲块保留在数组的顶部。

B显示了其中一个任务被删除后的数组。 数组顶部的大型空闲块仍然存在。 还有一个空闲块，其中先前已经分配了TCB和已经被删除的任务的堆栈。 当TCB被删除时释放的内存和删除堆栈时的内存释放不会保留为两个单独的空闲块。 相反，他们被结合起来创造一个更大，单一的空闲块。

C显示FreeRTOS队列创建后的阵列。 使用xQueueCreate（）API函数创建队列，这在使用队列（p。67）中进行了介绍。 xQueueCreate（）调用pvPortMalloc（）来分配队列使用的RAM。 由于heap_4使用第一个拟合算法，pvPortMalloc（）从足够容纳队列的第一个空闲RAM块分配RAM。在图中，这是任务被删除时释放的RAM。队列不占用空闲块中的所有RAM，所以块被分成两部分。未使用的部分仍然可用于未来调用pvPortMalloc（）。

D显示了pvPortMalloc（）直接从应用程序代码中调用，而不是通过调用FreeRTOS API函数间接调用的数组。用户分配的块足够小以适应第一个空闲块，即分配给队列的内存与分配给下一个TCB的内存之间的块。当任务被删除时释放的内存现在被分成三个独立的块。第一个块保存队列。第二个块保存用户分配的内存。第三个是免费的。

E显示队列被删除后的数组，它自动释放分配给已删除队列的内存。现在在用户分配块的两侧都有空闲的内存。

F显示用户分配内存也被释放后的数组。已经被用户分配块使用的内存已经与任意一侧的空闲内存相结合，以创建一个更大的空闲块。
Heap_4不是确定性的，但比malloc（）和free（）的大多数标准库实现更快。

### 为Heap_4使用的数组设置起始地址
注意：本节包含高级信息。您不需要阅读本节以使用heap_4。

有时，应用程序编写者必须将heap_4使用的数组放在特定的内存地址。例如，FreeRTOS任务使用的堆栈是从堆中分配的，因此可能需要确保堆位于快速内存中，而不是外部存储器很慢。

默认情况下，heap_4使用的数组在Heap_4.c源文件中声明。其起始地址由链接器自动设置。但是，如果FreeRTOSConfig.h中的configAPPLICATION_ALLOCATED_HEAP编译时配置常量设置为1，则该数组必须由应用程序使用FreeRTOS来声明。如果数组声明为应用程序的一部分，那么应用程序的writer可以设置其起始地址。

如果在FreeRTOSConfig.h中将configAPPLICATION_ALLOCATED_HEAP设置为1，那么调用ucHeap并由configTOTAL_HEAP_SIZE设置进行标注的uint8_t数组必须在其中一个应用程序的源文件中声明。

将变量放在特定内存地址所需的语法取决于编译器。有关信息，请参阅您的编译器的文档。

两个编译器的例子如下。

下面是GCC编译器声明数组并将其放在名为.my_heap的内存分配中所需的语法：

    uint8_t ucHeap[ configTOTAL_HEAP_SIZE ] __attribute__ ( (section( ".my_heap" ) ) );

这是IAR编译器声明数组并将其放在绝对内存地址0x20000000所需的语法：
uint8_t ucHeap[ configTOTAL_HEAP_SIZE ] @ 0x20000000;

### Heap_5

heap_5用于分配和释放内存的算法与heap_4使用的相同。不同于heap_4，heap_5不限于从单个静态声明的数组中分配内存。 Heap_5可以从多个分开的内存空间分配内存。 当运行FreeRTOS的系统所提供的RAM在系统内存映射中不会显示为单个连续（无空间）块时，Heap_5会很有用。

Heap_5是唯一提供的内存分配方案，必须在调用pvPortMalloc（）之前明确地初始化。 它使用vPortDefineHeapRegions（）API函数进行初始化。 使用heap_5时，必须在创建任何内核对象（任务，队列，信号量等）之前调用vPortDefineHeapRegions（）。

### vPortDefineHeapRegions() API Function

vPortDefineHeapRegions（）用于指定每个单独的内存区域的起始地址和大小。 他们一起构成heap_5使用的总内存。

    void vPortDefineHeapRegions（const HeapRegion_t * const pxHeapRegions）;

每个单独的内存区域由HeapRegion_t类型的结构描述。 将所有可用内存区域的描述作为HeapRegion_t结构的数组传递到vPortDefineHeapRegions（）中。

    typedef struct HeapRegion
    {
    /* The start address of a block of memory that will be part of the heap.*/
    uint8_t *pucStartAddress;
    /* The size of the block of memory in bytes. */
    size_t xSizeInBytes;
    } HeapRegion_t;

下表列出了vPortDefineHeapRegions（）参数。

<table>
	<thead>
		<tr>
			<td>
				Parameter Name/ Returned Value
			</td>
			<td>
				Description
			</td>
		</tr>
	</thead>

	<tbody>
		<tr>
			<td>
				pxHeapRegions
			</td>
			<td>
				A pointer to the start of an array of HeapRegion_t structures. Each structure in the array describes the start address and length of a memory area that will be part of the heap when heap_5 is used. The HeapRegion_t structures in the array must be ordered by start address. The HeapRegion_t structure that describes the memory area with the lowest start address must be the first structure in the array, and the HeapRegion_t structure that describes the memory area with the highest start address must be the last structure in the array. The end of the array is marked by a HeapRegion_t structure that has its pucStartAddress member set
				to NULL.
			</td>
		</tr>
	</tbody>
</table>

举例来说，考虑下图中所示的假想的存储器映射，它包含三个独立的RAM块：RAM1，RAM2和RAM3。 可执行代码被放置在只读存储器中，但未显示。

![](https://github.com/stevewuu/freertos_dg_translate/blob/master/image/Heap-5.bmp)

以下代码显示了HeapRegion_t结构的数组。 他们一起描述了三个RAM块。

    /* Define the start address and size of the three RAM regions. */
    #define RAM1_START_ADDRESS ( ( uint8_t * ) 0x00010000 )
    #define RAM1_SIZE ( 65 * 1024 )
    #define RAM2_START_ADDRESS ( ( uint8_t * ) 0x00020000 )
    #define RAM2_SIZE ( 32 * 1024 )
    #define RAM3_START_ADDRESS ( ( uint8_t * ) 0x00030000 )
    #define RAM3_SIZE ( 32 * 1024 )
    /* Create an array of HeapRegion_t definitions, with an index for each of the three RAM
    regions, and terminating the array with a NULL address. The HeapRegion_t structures must
    appear in start address order, with the structure that contains the lowest start address
    appearing first. */
    const HeapRegion_t xHeapRegions[] =
    {
    { RAM1_START_ADDRESS, RAM1_SIZE },
    { RAM2_START_ADDRESS, RAM2_SIZE },
    { RAM3_START_ADDRESS, RAM3_SIZE },
    { NULL, 0 } /* Marks the end of the array. */
    };
    int main( void )
    {
    /* Initialize heap_5. */
    vPortDefineHeapRegions( xHeapRegions );
    /* Add application code here. */
    }
虽然代码正确地描述了RAM，但是这不是一个可用的例子，因为它将所有的RAM分配给堆，没有RAM供其他变量使用。在构建项目时，构建过程的链接阶段会为每个变量分配一个RAM地址。

可供链接器使用的RAM通常由链接器配置文件（例如链接器脚本）来描述。在上图的B中，假设链接描述文件包含RAM1上的信息，但不包括RAM2或RAM3上的信息。链接器因此将变量放在RAM1中，只留下地址0x0001nnnn以上的RAM1部分可供heap_5使用。 0x0001nnnn的实际值将取决于被链接的应用程序中包含的所有变量的组合大小。链接器已经将所有RAM2和RAM3留空，所以它们可以被heap_5使用。

如果使用前面的代码，分配给地址0x0001nnnn下的heap_5的RAM将与用于保存变量的RAM重叠。为了避免这种情况，xHeapRegions []数组中的第一个HeapRegion_t结构可以使用0x0001nnnn的起始地址而不是0x00010000。

这不是推荐的解决方案，因为：

- 起始地址可能不容易确定。
- 链接器使用的RAM数量可能会在将来的版本中更改，因此需要更新HeapRegion_t结构中使用的起始地址。
- 如果heap_5使用的RAM重叠，则构建工具将不知道，因此无法警告应用程序编写器。

以下代码演示了一个更方便和可维护的示例。它声明了一个名为ucHeap的数组。 ucHeap是一个正常的变量，所以它成为由链接器分配给RAM1的数据的一部分。 xHeapRegions数组中的第一个HeapRegion_t结构描述了ucHeap的起始地址和大小，所以ucHeap成为heap_5管理的内存的一部分。 可以增加ucHeap的大小，直到链接器使用的RAM消耗所有的RAM1，如上图中的C所示。

    /* Define the start address and size of the two RAM regions not used by the linker. */
    #define RAM2_START_ADDRESS ( ( uint8_t * ) 0x00020000 )
    #define RAM2_SIZE ( 32 * 1024 )
    #define RAM3_START_ADDRESS ( ( uint8_t * ) 0x00030000 )
    #define RAM3_SIZE ( 32 * 1024 )
    /* Declare an array that will be part of the heap used by heap_5. The array will be  placed in RAM1 by the linker. */
    #define RAM1_HEAP_SIZE ( 30 * 1024 )
    static uint8_t ucHeap[ RAM1_HEAP_SIZE ];
    /* Create an array of HeapRegion_t definitions. Whereas in previous code listing, the first entry described all of RAM1, so heap_5 will have used all of RAM1, this time the firstentry only describes the ucHeap array, so heap_5 will only use the part of RAM1 that contains the ucHeap array. The HeapRegion_t structures must still appear in start address order, with the structure that contains the lowest start address appearing first. */
    const HeapRegion_t xHeapRegions[] =
    {
    { ucHeap, RAM1_HEAP_SIZE },
    { RAM2_START_ADDRESS, RAM2_SIZE },
    { RAM3_START_ADDRESS, RAM3_SIZE },
    { NULL, 0 } /* Marks the end of the array. */
    };

在前面的代码中，一个HeapRegion_t结构数组描述了所有的RAM2，全部是RAM3，但是只是RAM1的一部分。
这里展示的技术的优点包括：

- 您不需要使用硬编码的起始地址。
- HeapRegion_t结构中使用的地址将由链接器自动设置，因此即使链接器在以后的版本中使用的RAM数量发生变化，它也会始终正确。
- 分配给heap_5的RAM不可能与链接器放入RAM1的数据重叠。
- 如果ucHeap太大，应用程序将不会链接。

#### 堆相关效用函数
###xPortGetFreeHeapSize() API Function

xPortGetFreeHeapSize（）API函数返回调用该函数时堆中的空闲字节数。 它可以用来优化堆大小。 例如，如果在创建所有内核对象后xPortGetFreeHeapSize（）返回2000，则configTOTAL_HEAP_SIZE的值可以减少2000。

使用heap_3时，xPortGetFreeHeapSize（）不可用。

xPortGetFreeHeapSize（）API函数原型


    size_t xPortGetFreeHeapSize( void );

下表列出了xPortGetFreeHeapSize（）返回值。

<table>
	<thead>
		<tr>
			<td>
				Parameter Name/ Returned Value
			</td>
			<td>
				Description
			</td>
		</tr>
	</thead>

	<tbody>
		<tr>
			<td>
				Returned value
			</td>
			<td>
				The number of bytes that remain unallocated in the heap at the time xPortGetFreeHeapSize() is called.
			</td>
		</tr>
	</tbody>
</table>

### xPortGetMinimumEverFreeHeapSize（）API函数

xPortGetMinimumEverFreeHeapSize（）API函数返回自FreeRTOS应用程序开始执行以来在堆中存在的最小未分配字节数。

由xPortGetMinimumEverFreeHeapSize（）返回的值表示应用程序已经耗尽堆空间的程度。 例如，如果xPortGetMinimumEverFreeHeapSize（）返回200，那么在应用程序开始执行之后的某个时间，它在200个字节的堆空间内运行。

只有在使用heap_4或heap_5时，xPortGetMinimumEverFreeHeapSize（）才可用。

下表列出了xPortGetMinimumEverFreeHeapSize（）返回值。

<table>
	<thead>
		<tr>
			<td>
				Parameter Name/ Returned Value
			</td>
			<td>
				Description
			</td>
		</tr>
	</thead>

	<tbody>
		<tr>
			<td>
				Returned value
			</td>
			<td>
				The minimum number of unallocated bytes that have existed in the heap since the FreeRTOS application started executing.
			</td>
		</tr>
	</tbody>
</table>



### Malloc失败钩子函数

可以直接从应用程序代码调用pvPortMalloc（）。 每当创建一个内核对象时，它在FreeRTOS源文件中也被调用。 内核对象包括任务，队列，信号量和事件组，所有这些都在后面描述。

就像标准库malloc（）函数一样，如果pvPortMalloc（）不能返回一块RAM，因为所请求大小的块不存在，那么它将返回NULL。 如果因为应用程序编写器正在创建内核对象而执行pvPortMalloc（），并且对pvPortMalloc（）的调用返回NULL，则不会创建内核对象。

如果对pvPortMalloc（）的调用返回NULL，则可以将所有示例堆分配方案配置为调用挂钩（或回调）函数。

如果FreeRTOSConfig.h中的configUSE_MALLOC_FAILED_HOOK设置为1，那么应用程序必须提供一个malloc失败的钩子函数，其名称和原型如下所示。 该功能可以以任何适合于应用程序的方式实现。

    void vApplicationMallocFailedHook( void );