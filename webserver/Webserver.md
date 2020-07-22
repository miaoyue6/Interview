# Webserver

## 信号量

信号量是一种特殊的变量，只能取自然数并且只支持两种操作：等待（P）和信号（V）假设有信号量SV，对其的P、V操作如下：

*P：如果SV大于0，则减一；若SV等于0，那么将其挂起；

*V：如果其他进程因为等待信号SV而被挂起，那么将其唤醒；否则将SV加1；

信号量可以取值任何数，最常用的是二进制信号量，只有0和1两个数字；

初始化信号：int sem_init（sem_t *sem, int pshared, unsigned int value);其中sem是要初始化的信号量，pshared表示此信号量是在进程间共享还是线程间共享，value是信号量的初始值；

销毁信号：int sem_destory(sem_t* sem);其中sem是要销毁的信号量，只有用sem_init初始化的信号量才能用sem_destory（）销毁；

int sem_wait(sem_t *sem)；等待信号量，如果信号量的值大于0，将信号量的值减1，立即返回。如果信号量的值为0，那么线程阻塞。相当于P操作。成功返回0，失败返回-1；

int sem_post(sem_t *sem):释放信号量，让信号量的值加1，相当于V操作；

## 互斥量

互斥所，也成互斥量，可以保护关键代码，确保独占式访问。当进入关键代码段时，获得互斥锁将其枷锁；离开代码段，唤醒该互斥锁线程。

锁类型结构：ptread_mutex_t。通过对该结构的操作，能够判断自愿是否可以访问。

1.互斥所pthread的创建于销毁

有两种方法创建互斥锁，**静态方式**和**动态方式**。POSIX定义了一个宏**PTHREAD_MUTEX_INITIALIZER**来静态初始化互斥锁，方法如下：

**pthread_mutex_t** mutex=**PTHREAD_MUTEX_INITIALIZER**;

在LinuxThreads实现中，pthread_mutex_t是一个结构，而PTHREAD_MUTEX_INITIALIZER则是一个结构常量。

动态方式是采用pthread_mutex_init(pthread_mutex_t *mutex, const pthread_mutexattr_t*mutexattr);

其中*mutexattr*用于指定互斥锁属性（见下），如果为**NULL**则使用缺省属性。

**pthread_mutex_destroy()**用于注销一个互斥锁，API定义如下：

int **pthread_mutex_destroy(pthread_mutex_t** **mute*x**)**

销毁一个互斥锁即意味着释放它所占用的资源，且要求锁当前处于开放状态。由于在Linux中，互斥锁并不占用任何资源，因此LinuxThreads中的 pthread_mutex_destroy()除了检查锁状态以外（锁定状态则返回EBUSY）没有其他动作。

**2.\*互斥锁属性\***

互斥锁的属性在创建锁的时候指定，在LinuxThreads实现中仅有一个锁类型属性，不同的锁类型在试图对一个已经被锁定的互斥锁加锁时表现不同。当前（glibc2.2.3,linuxthreads0.9）有四个值可供选择：

\* PTHREAD_MUTEX_TIMED_NP，这是缺省值，也就是**普通锁**。*当一个线程加锁以后，其余请求锁的线程将形成一个等待队列，并在解锁后按优先级获得锁*。这种锁策略保证了资源分配的公平性。
\* PTHREAD_MUTEX_RECURSIVE_NP，**嵌套锁**，*允许同一个线程对同一个锁成功获得多次，并通过多次unlock解锁*。如果是不同线程请求，则在加锁线程解锁时重新竞争。
\* PTHREAD_MUTEX_ERRORCHECK_NP，**检错锁**，如果*同一个线程请求同一个锁，则返回EDEADLK*，否则与PTHREAD_MUTEX_TIMED_NP类型动作相同。这样保证当不允许多次加锁时不出现最简单情况下的死锁。
\* PTHREAD_MUTEX_ADAPTIVE_NP，**适应锁**，动作最简单的锁类型，仅**等待解锁后重新竞争**。

**3.\*其他锁操作\***

　　锁操作主要包括加锁pthread_mutex_lock()、解锁pthread_mutex_unlock()和测试加锁 pthread_mutex_trylock()三个，不论哪种类型的锁，都不可能被两个不同的线程同时得到，而必须等待解锁。对于普通锁和适应锁类型，解锁者可以是同进程内任何线程；而检错锁则必须由加锁者解锁才有效，否则返回EPERM；对于嵌套锁，文档和实现要求必须由加锁者解锁，但实验结果表明并没有这种限制，这个不同目前还没有得到解释。在同一进程中的线程，如果加锁后没有解锁，则任何其他线程都无法再获得锁。

　　int pthread_mutex_lock(pthread_mutex_t *mutex)
　　int pthread_mutex_unlock(pthread_mutex_t *mutex)
　　int pthread_mutex_trylock(pthread_mutex_t *mutex)

　　pthread_mutex_trylock()语义与pthread_mutex_lock()类似，不同的是在锁已经被占据时返回EBUSY而不是挂起等待;



# [pthread_join()](https://www.cnblogs.com/renzmin/p/12097963.html)

void pthread_exit(void *retval)

int pthread_join(pthread_t th, void **thread_return)

相关1：pthread_join是为了防止主线程没有给其他线程执行的时间就返回了而设计的，

​      pthread_join(thread_t th,void ** thread_return )是使主线程等待th线程运行结束再运行
相关2：有时候主线程创建子线程后，如果不使用pthread_join将自己阻塞，自己会先退出而程序结束，
​      这样子线程的运行可能无法执行完毕就**退出了，这也算是要使用pthread_join的一个场景吧。

相关3：pthread_join应该是用来回收线程资源的,当线程结束时调用,在一支程序中一直创建线程,而在
      线程结束时又没有用pthread_join则会造成资源不足,无法继续创建线程的情况.

相关4：pthread_join回收线程资源，在pthread_create后父进程就可调用此函数，不过会阻塞父进程直到子进程结束。
      pthread_join()不会阻塞其他子进程。
      可以设置线程属性自动回收资源，就不用调用pthread_join了

## 条件变量

条件变量提供了一种线程间的通讯机制

要保证变量正确被修改，条件变量也需要受到保护，此时用了互斥锁来保护。如果不使用两者搭配的话，当一个线程只使用互斥锁的话，当其他线程到这个程序的时候就会来判断是否上锁，若上锁了就等待解锁，这个时间里可能会有很多线程来判断，然后阻塞在这里，当调用锁的线程释放锁后，那么被阻塞的线程又会来抢夺这个资源，从而造成了资源时间空间的浪费。而当加上了条件变量的时候，被阻塞的线程就会变得有序，在一个队列里排队，当接受到激活信号之前不会反复检查是否该锁已经释放，只需要一个通知（激活信号）来告诉这个线程，你所需要的那个锁已经释放了，现在你可以用了，此时整个过程就变的有序，而不是因为抢夺来消耗资源了。二者的关系操作可以形象看作从一个无序的占用资源变成一个有序的节约资源的一个改变，所以条件变量和互斥锁二者使用能够保证条件变量的正确修改。
————————————————
版权声明：本文为CSDN博主「Jialuhu」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/qq_36573828/article/details/76708162

**函数原型：

int pthread_cond_wait(pthread_cond_t *cond,pthread_mutex_t *mutex)

两个动作：释放第二个参数指向的锁，解锁后，吧第一个参数cond的条件变量指向条件变量阻塞变量，知道条件信号被激活；



## linux高性能服务器-条件变量

条件变量提供了一种线程间的通讯机制：当某个共享数据达到某个值的时候，唤醒等待这个共享数据的线程。

函数的第一参数cond指向要操作的目标条件变量。

## 服务器编程基本框架

主要由I/O单元，逻辑单元和网络纯属单元组成，其中每个单元之间通过请求队列进行通信，从而协同完成任务。

齐总IO单元用于处理客户端链接，读写网络数据；

逻辑单元用于处理业务逻辑线程；

网络存储单元指本地数据库和文件。

## 五中I/O模型

*阻塞IO:调用者调用了某个函数，等待这个函数返回，期间什么也不做，不停地去检查这个函数有没有返回，必须等待这个函数返回才能进行下一个动作；

*非阻塞IO：非阻塞等待，每隔一段时间去检测IO事件是否就绪。没有就绪就可以做其他事。非阻塞IO执行系统调用总是立即返回，不管时间是否已经发生，若事件没有发生，若事件没有发生就返回-1，

*信号驱动：linux用套接口进行信号驱动IO，安装一个信号处理函数，进程继续运行并不阻塞，当IO事件就绪，进程收到SIGIO信号。然后处理函数

*IO复用：linux用select/poll实现IO复用模型，

## 同步I/O模拟practor模式

1.主线程往epoll内核事件表注册socket上的读就绪事件；

2.主线程调用epoll_wait等待socket上有数据可读；

当socket上有数据可读，epoll_wait通知主线程，主线程从socket循环读取数据，知道没有更多数据可读，然后然后将读取的数据封装成一个请求对象并插入请求队列。

3.睡眠在请求队列上某个工作线程被唤醒，他获得请求对象并处理客户请求，然后往epoll内核事件报中注册该socket上的写就绪事件；

4.主线程调用epoll_wait等待socket可写；

5.当socket上有数据可写，epoll_wait通知主线程。主线程往socket上写入服务器处理客户请求的结构；

## 静态成员变量

将类成员变量沈明为static，无论建立多少对象，都只有一个静态成员变量的拷贝，静态成员变量属于一个类，所有对象共享。静态变量在编译阶段就分配了空间，对象还没创建时就已经分配了空间，放在全局静态区。

## 静态成员函数

类成员函数声明为static，则为静态成员函数。

静态成员函数：

1.静态成员函数刻印之界访问静态成员变量，不能直接访问普通成员变量，但可以通过参数传递的方式访问。

2.普通成员函数可以访问普通成员变量，也可以访问静态成员变量；

## 线程池分析

线程池的设计模式为版同步/伴反应堆，其中反应堆具体位置proactor事件处理模式。

主线程为异步线程，负责监听文件描述符，接收socket新链接，若当前监听的socket发生了读写事件，然后将任务插入到请求对列。工作线程从请求对列中取出任务，完成读写数据的处理。



