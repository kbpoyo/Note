## 特权级别，Privilege Level

## 基本概念

引入LDT和TSS，能够从任务层面上进一步强化了分段机制，但是从安全保障的角度来看，只相当于构建了可靠的硬件设施。仅有设施是不够的，还需要规章制度，还要有人来执行，处理器也一样。为此，在分段机制的基础上，处理器引入了特权级，并由固件负责实施特权级保护。

特权级别，Privilege Level，是存在于 Descriptor （描述符）及 Segment Selector（段选择子，存储在段寄存器以及门描述符中） 中一个数值，当这些 Descriptor 或 Segment Selector 要进行某些操作，或者被别的对象访问时，该数值用于控制它们能够进行的操作或者限制它们的可访问性。

Intel Processor 具有 4 个特权级别（0特权最大，3特权最小），以及3个受保护的主要资源：内存、I/O端口和执行某些机器指令的能力。在任何给定的时间，[x86 CPU](https://www.zhihu.com/search?q=x86%20CPU&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22article%22%2C%22sourceId%22%3A%22590066420%22%7D)都以特定的特权级别运行，这决定了代码可以做什么和不能做什么。这些特权级别通常被描述为保护环，最里面的环对应于最高的特权。

![](https://pic4.zhimg.com/v2-ad753890c419d8487e19e81ae3b3350b_b.jpg)

- 通常，因为操作系统是为所有程序服务的，可靠性最高，而且必须对软硬件有完全的控制权，所以它的主体部分必须拥有特权级0，并处于整个环形结构的中心。也正是因为这样，操作系统的主体部分通常又被称做内核(Kernel、 Core)。
- 特权级1和2通常赋予那些可靠性不如内核的[系统服务程序](https://www.zhihu.com/search?q=%E7%B3%BB%E7%BB%9F%E6%9C%8D%E5%8A%A1%E7%A8%8B%E5%BA%8F&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22article%22%2C%22sourceId%22%3A%22590066420%22%7D)，比较典型的就是设备驱动程序。当然，在很多比较流行的操作系统中，驱动程序与内核的特权级别相同，都是0。
- 应用程序的可靠性被视为是最低的，而且通常不需要直接访问硬件和一些敏感的系统资源，调用设备驱动程序或者操作系统例程就能完成绝大多数工作，故赋予它们最低的特权级别3。

在操作系统概念中，用户态和内核态其实是CPU的特权级，所以模式的切换就是CPU特权级的切换，模式等同于特权级，不同的模式表示CPU处于不同的特权级下，因此CPU特权级的切换不能局限于用户态到内核态之间，理论上CPU可以在任何特权级之间互相切换。

大多数现代x86内核只使用两个特权级别：0和3。之所以只是用两个级别，是因为：

1.实用性：现代操作系统基本上是多级分页管理而不是分段管理，更多的是需要对单个页进行管理。而且CPU给的权限管理细度不够，典型的像Android（Android也有x86版本）需要对单个APP的权限进行更细致化管理（有的权限仅能在APP运行过程中使用）。而特权级无法做到如此细致的管理。

2.可移植性：Windows和Linux不仅仅只是给x86架构设计的系统，也要在arm等其他架构运行。（虽然windows很大程度已经和x86架构绑定）有的架构只有2个特权级，使用这些会对特定的架构产生耦合。

3.效率：NT内核的设计者曾经研究过[虚拟内存](https://www.zhihu.com/search?q=%E8%99%9A%E6%8B%9F%E5%86%85%E5%AD%98&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22article%22%2C%22sourceId%22%3A%22590066420%22%7D)管理系统，真正用了x86上的四个特权级。然而他们发现，ring2下的[控制台程序](https://www.zhihu.com/search?q=%E6%8E%A7%E5%88%B6%E5%8F%B0%E7%A8%8B%E5%BA%8F&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22article%22%2C%22sourceId%22%3A%22590066420%22%7D)安全情况和ring3无异，[ring1](https://www.zhihu.com/search?q=ring1&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22article%22%2C%22sourceId%22%3A%22590066420%22%7D)下的权限管理系统要经常调用ring0特权指令，在切换特权级花费的成本太大，不如直接放入ring0特权级划算。

## 描述符特权级（DPL，Descriptor Privilege Level）

实施特权级保护的第一步，是为所有可管理的对象赋予一个特权级，以决定谁能访问它们。每个 Descriptor 都具有描述符特权级（DPL，Descriptor Privilege Level）字段，Descriptor 总是指向它所“描述”的目标对象，代表着该对象，因此该字段（DPL）实际上是目标对象的特权级。

## 当前特权级（CPL，Current Privilege Level）

当处理器正在一个代码段中取指令和执行指令时，那个代码段的特权级叫做当前特权级(Current Privilege Level, CPL)。 正在执行的这个代码段，其选择子位于段寄存器CS中，其最低两位就是当前特权级的数值。

一般来说，操作系统是最先从BIOS那里接收处理器控制权的，进入保护模式的工作也是由它做的，而且，最重要的是，它还肩负着整个计算机系统的管理工作，所以，它必须工作在0特权级别上，当操作系统的代码正在执行时，当前特权级CPL就是0；

相反，普通的应用程序则工作在特权级别3上。应用程序编写时，不需要考虑GDT、LDT、分段、描述符这些东西，它们是在程序加载时，由操作系统负责创建的，应用程序的编写者只负责具体的功能就可以了。应用程序的加载和开始执行，也是由操作系统所主导的，而操作系统一定会将它放在特权级3上。当应用程序开始执行时，当前特权级CPL自然就会是3。

这实际上就是把一个任务分成特权级截然不同的两个部分，全局部分是特权级0的，而局部空间则是特权级3的。这种划分是有好处的，全局空间是为所有任务服务的，其重要性不言而喻。为了保证它的安全性，并能够访问所有软硬件资源，应该使它拥有最高的特权级别。当任务在自己的局部空间内执行时，当前特权级CPL是3；当它通过调用系统服务，进入操作系统内核，在全局空间执行时，当前特权级CPL就变成了0。

## 请求特权级（RPL，Request Privilege Level）

RPL的意思是请求特权级(Requested Privilege Level)。 我们知道，要将控制从一个代码段转移到另一个代码段，通常是使用jmp和call 指令，并在指令中提供目标代码段的选择子，以及[段内偏移量](https://www.zhihu.com/search?q=%E6%AE%B5%E5%86%85%E5%81%8F%E7%A7%BB%E9%87%8F&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22article%22%2C%22sourceId%22%3A%22590066420%22%7D)(入口点)。而为了访问内存中的数据，也必须先将段选择子加载到段寄存器DS、ES、FS或者GS中。不管是实施控制转移，还是访问数据段，这都可以看成是一一个请求，请求者提供一个段选择子，请求访问指定的段。从这个意义上来说，RPL也就是指请求者的特权级别(Requestor’s Privilege Level)。

引入请求特权级(RPL)的原因是处理器在遇到一条将选择子传送到段寄存器的指令时，无法区分真正的请求者是谁。但是，引入RPL本身并不能完全解决这个问题，这只是处理器和操作系统之间的一种协议，处理器负贵检查请求特权级RPL，判断它是否有权访问，但前提是提供了正确的RPL；内核或者操作系统负责鉴别请求者的身份，并有义务保证RPL的值和它的请求者身份相符，因为这是处理器无能为力的。

操作系统的编写者很清楚段选择子的来源，即真正的请求者是谁。当它提供一个服务例程时，3特权级别的用户程序给出的选择子在哪里是由它定的，在这种情况下，它所要做的，就是将该选择子的RPL字段设置为请求者的特权级(可以使用arpl 指令)。剩下的工作就看处理器了。每当处理器执行一个将[段选择子](https://www.zhihu.com/search?q=%E6%AE%B5%E9%80%89%E6%8B%A9%E5%AD%90&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22article%22%2C%22sourceId%22%3A%22590066420%22%7D)传送到段寄存器(DS、ES、FS、 GS)的指令，都会进行特权检查。

## 特权指令（Privileged Instructions）

不同特权级别的程序，所担负的职责以及在系统中扮演的角色是不一样的。计算机系统的脆弱性在于一条指令就能改变它的整体运行状态，比如停机指令hlt 和对控制寄存器CRO的写操作，像这样的指令只能由最高特权级别的程序来做。因此，那些只有在当前特权级 CPL 为 0 时才能执行的指令，称为特权指令(Privileged Instructions)。

典型的特权指令包括加载[全局描述符表](https://www.zhihu.com/search?q=%E5%85%A8%E5%B1%80%E6%8F%8F%E8%BF%B0%E7%AC%A6%E8%A1%A8&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22article%22%2C%22sourceId%22%3A%22590066420%22%7D)的指令 lgdt (它在实模式下也可执行)、加载[局部描述符表](https://www.zhihu.com/search?q=%E5%B1%80%E9%83%A8%E6%8F%8F%E8%BF%B0%E7%AC%A6%E8%A1%A8&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22article%22%2C%22sourceId%22%3A%22590066420%22%7D)的指令lldt、 加载[任务寄存器](https://www.zhihu.com/search?q=%E4%BB%BB%E5%8A%A1%E5%AF%84%E5%AD%98%E5%99%A8&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22article%22%2C%22sourceId%22%3A%22590066420%22%7D)的指令ltr、读写控制寄存器的mov指令、停机指令hlt 等十几条。

## 输出特权级（I/O Privilege Level）

除了那些特权级敏感的指令外，处理器还允许对各个特权级别所能执行的I/O操作进行控制。通常，这指的是[端口访问](https://www.zhihu.com/search?q=%E7%AB%AF%E5%8F%A3%E8%AE%BF%E9%97%AE&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22article%22%2C%22sourceId%22%3A%22590066420%22%7D)的许可权，因为对设备的访问都是通过端口进行的。

在处理器的[标志寄存器](https://www.zhihu.com/search?q=%E6%A0%87%E5%BF%97%E5%AF%84%E5%AD%98%E5%99%A8&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22article%22%2C%22sourceId%22%3A%22590066420%22%7D)EFLAGS中，位13、位12是IOPL位，也就是输入/输出特权级(I/O Privilege Level)，它代表着当前任务的I/O特权级别。某些指令，例如IN，OUT，CLI需要 I/O 特权，这些操作根据 IOPL 和 CPL 确定合法性。

处理器不限制0特权级程序的I/O访问，它总是允许的。但是，可以限制低特权级程序的I/O访问权限。这是很重要的，操作系统的功能之一是设备管理，它可能不希望应用程序拥有私自访问外设的能力。

## 存储位置

### DPL

存储于段描述符中：

![](https://pic3.zhimg.com/v2-5bd7e054fdbc7a400a132a807d56c7f2_b.jpg)

存储于段选择子（段选择子存储于各个段寄存器以及门描述符中-调用门、任务门、[中断门](https://www.zhihu.com/search?q=%E4%B8%AD%E6%96%AD%E9%97%A8&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22article%22%2C%22sourceId%22%3A%22590066420%22%7D)、陷阱门）中：

![](https://pic3.zhimg.com/v2-fa15860a6ebbc59ed5f81c49dfa5a29e_b.jpg)

RPL 是请求特权级，表示给出当前选择子的那个程序的特权级别，正是该程序要求访问这个内存段。

### CPL

存储于CS寄存器的段选择子中（CS中的RPL就是CPL）：

![](https://pic2.zhimg.com/v2-1629eea09f3376d44756fb97d3a0ed3d_b.jpg)

### IOPL

存储于EFLAGS中：

![](https://pic4.zhimg.com/v2-6bb9c91752974af730854457299521c7_b.jpg)

正在运行任务的当前特权级(CPL)必须小于或等于I/O特权级才能允许访问I/O地址空间。这个域只能在CPL为0时才能通过POPF以及IRET指令修改。

## 控制转移的方法

代码段的特权级检查是很严格的，一般来说，控制转移只允许发生在两个特权级相同的代码段间。但是为了使特权级低的程序调用特权级高的操作系统例程，处理器提供相应的解决方法：

### 方法一、依从代码段

对于高特权级的代码段，将其 TYPE 字段的 C 位设置为 1（即依从的代码段），这样特权级低的程序则可以调用进入高特权级的代码段

但是，特权级的检查依旧非常严格：必须保证 CPL 低于 DPL，即数值上 CPL >= DPL （值越大，特权级越低，所以是 >= 符号，没有写错）。比如说 DPL == 1 则只有 CPL == 1/2/3 可以调用。

另外，特权级不会发生变化，即：对于定义为依从的高特权级的代码段，当从特权级低的程序调用进入后，当前特权级依旧是特权级低的程序的特权级，而不使用高特权级代码段的特权级。比如从 CPL == 3 进入 DPL == 1 代码段，依旧 CPL == 3 而不是 CPL == 1，这也是被称为“依从”的原因。

### 方法二、调用门（Call Gate）

调用门将控制从较低权限代码转移到与当前特权级相同或较高权限的代码段，调用 Call Gate 指向的函数，只需要使用调用门选择子作为 CALL FAR / JMP FAR 的操作数。

使用 JMP FAR 不改变特权级，依旧使用 CPL 运行。使用 CALL FAR 将改变特权级（CPL = DPL）。另外，除了从高特权级别的例程(通常是操作系统例程)返回外，不允许从特权级高的代码段将控制转移到特权级低的代码段，因为操作系统不会引用可靠性比自已低的代码。

## 基本的特权级检查规则

首先，将控制直接转移到非依从的代码段，要求当前特权级CPL和请求特权级RPL都等于目标代码段描述符的DPL。即，在数值上，  
CPL=目标代码段描述符的DPL  
RPL=目标代码段描述符的DPL  
一个典型的例子就是使用jmp指令进行控制转移:  
jmp 0x0012:0x00002000  
因为两个代码段的特权级相同，故，转移后当前特权级不变。

其次，要将控制直接转移到依从的代码段，要求当前特权级CPL和请求特权级RPL都低于，或者和目标代码段描述符的DPL相同。即，在数值上,  
CPL≥目标代码段描述符的DPL  
RPL≥目标代码段描述符的DPL  
控制转移后，当前特权级保持不变。

第三，高特权级别的程序可以访问低特权级别的数据段，但低特权级别的程序不能访问高特权级别的数据段。访问数据段之前，肯定要对段寄存器DS、ES、FS和GS进行修改，比如  
mov fs,ax  
在这个时候，要求当前特权级CPL和请求特权级RPL都必须高于，或者和目标数据段描述符的DPL相同。即，在数值上，  
CPL≤目标数据段描述符的DPL  
RPL≤目标数据段描述符的DPL

最后，处理器要求，在任何时候，栈段的特权级别必须和当前特权级CPL相同。因此，随着程序的执行，要对段寄存器ss的内容进行修改时，必须进行特权级检查。以下就是一个修改段寄存器ss的例子:  
mov ss,ax，  
在对段寄存器ss进行修改时，要求当前特权级CPL和请求特权级RPL必须等于目标栈段描述符的DPL。即，在数值上，  
CPL=目标栈段描述符的DPL  
RPL=目标栈段描述符的DPL

0特权级是最高的特权级别，当一个系统的各个部分都位于0特权级时，各种特权级检查总能够获得通过，就像这种检查和检验并不存在一样。所以，处理器的设计者建议，如果不需要使用特权机制的话，可以将所有程序的特权级别都设置为0。