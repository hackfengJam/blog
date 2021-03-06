## 用户态与内核态

### 1. 用户态切换到内核态方式

用户态切换到的触发条件：申请外部资源

#### 1.1 用户态切换到内核态的3种方式

1、系统调用

这是用户进程主动要求切换到内核态的一种方式，用户进程通过系统调用申请操作系统提供的服务程序完成工作。
而系统调用的机制其核心还是使用了操作系统为用户特别开放的一个中断来实现，例如Linux的ine 80h中断。


例：

- 读写文件 open/read/write   
- 申请内存（堆内存） malloc   
    - brk (<128k)    
    - mmap（申请虚拟内存空间）   
    
注意：

这里 mmap 申请的是虚拟内存空间，并不是主存上的物理内存空间，
想要拿到真正的物理内存空间，还要在第一次访问时，会发现虚拟内存地址没有映射到物理内存地址，
于是触发 缺页中断（缺页异常）   
    

2、异常

当CPU在执行运行在用户态的程序时，发现了某些事件不可知的异常，这是会触发由当前运行进程切换到处理此。
异常的内核相关程序中，也就到了内核态，比如缺页异常。

3、外围设备的中断

当外围设备完成用户请求的操作之后，会向CPU发出相应的中断信号，这时CPU会暂停执行下一条将要执行的指令，转而去执行中断信号的处理程序，
如果先执行的指令是用户态下的程序，那么这个转换的过程自然也就发生了有用户态到内核态的切换。
比如硬盘读写操作完成，系统会切换到硬盘读写的中断处理程序中执行后续操作等。


#### 1.2 系统调用

> man syscalls

① 进程  

- exit  
- fork  

② 文件 

- chmod   
- chown  
 
③ 设备  

- read  
- write  
- io ctrl  
- mmap(磁盘)  

④ 信息  

- getcpu  
- get xxx  


⑤ 通信  

- 管道（pipe） 
- mmap（文件和内存的映射）  

### 2. 切换操作

从触发方式看，可以在认为存在前述3种不同的类型，但是从最终实际完成由用户态到内核态的切换操作上来说，涉及的关键步骤是完全一样的，没有任何区别，都相当于执行了一个中断响应的过程，因为系统调用实际上最终是中断机制实现的，而异常和中断处理机制基本上是一样的，用户态切换到内核态的步骤主要包括：

1、从当前进程的描述符中提取其内核栈的ss0及esp0信息。

2、使用ss0和esp0指向的内核栈将当前进程的cs,eip，eflags，ss,esp信息保存起来，这个过程也完成了由用户栈找到内核栈的切换过程，同时保存了被暂停执行的程序的下一条指令。

3、将先前由中断向量检索得到的中断处理程序的cs，eip信息装入相应的寄存器，开始执行中断处理程序，这时就转到了内核态的程序执行了。

### 3. 用户态和内核态区别

#### 3.1 系统态和用户态

在计算机系统中，通常运行着两类程序：系统程序和应用程序，为了保证系统程序不被应用程序有意或无意地破坏，为计算机设置了两种状态：

- 系统态(也称为管态或核心态)，操作系统在系统态运行——运行操作系统程序  
- 用户态(也称为目态)，应用程序只能在用户态运行——运行用户程序  

#### 3.2 特权指令和非特权指令

在实际运行过程中，处理机会在系统态和用户态间切换。相应地，现代多数操作系统将 CPU 的指令集分为特权指令和非特权指令两类。

1) 特权指令——在系统态时运行的指令

对内存空间的访问范围基本不受限制，不仅能访问用户存储空间，也能访问系统存储空间，
特权指令只允许操作系统使用，不允许应用程序使用，否则会引起系统混乱。
 

2) 非特权指令——在用户态时运行的指令

一般应用程序所使用的都是非特权指令，它只能完成一般性的操作和任务，不能对系统中的硬件和软件直接进行访问，其对内存的访问范围也局限于用户空间。

#### 3.3 UNIX 系统中用户态与核心态

UNIX 系统把进程的执行状态分为两种:

- 一种是用户态执行，表示进程正处于用户状态中执行；

- 另一种是核心态执行，表示一个应用进程执行系统调用后，或 I/O 中断、时钟中断后，进程便处于核心态执行。


这两种状态的主要差别在于：

- 处于用户态执行时，进程所能访问的内存空间和对象受到限制，其所占有的处理机是可被抢占的；

- 而处于核心态执行中的进程，则能访问所有的内存空间和对象，且所占用的处理机是不允许被抢占的。


注：

- 用户态切换到内核态的唯一途径——>中断/异常/陷入   

- 内核态切换到用户态的途径——>设置程序状态字  


注意一条特殊的指令——陷入指令（又称为访管指令，因为内核态也被称为管理态，访管就是访问管理态）。该指令给用户提供接口，用于调用操作系统的服务。

### 感谢

TODO
