---
layout: post
title: "Semaphore, Mutex, Critical section, SpinLock, Event"
date: 2013-03-22 17:06
comments: true
categories: 
---

####Critical Section

In concurrent programming, a critical section is a piece of code that accesses a shared resource (data structure or device) that must not be concurrently accessed by more than one thread of execution. A critical section will usually terminate in fixed time, and a thread, task, or process will have to wait for a fixed time to enter it (aka bounded waiting). Some synchronization mechanism is required at the entry and exit of the critical section to ensure exclusive use, for example a semaphore. (From wiki)

����������**�ٽ���**.������Ӳ���ٽ���Դ����������ٽ���Դ������̱߳��뻥��ض������з��ʡ����ڲ���Ҫ����ϵͳ��(Mutex�������ڿ����ͬ����������ں˶���)�����Ƚ�������Ч�ʻ�ߺܶࣨWindows�£��������еĲ���ϵͳ**����**��**�߳�**��ͬһ�����������VxWorksֻ��**Task**,�����ֻ����£�Mutex��Critical Section�ǵȼ۵ġ�

#####Critical Section Usage
**Example Code For Critical Sections with POSIX pthread library**


	/* Sample C/C++, Unix/Linux */
	#include <pthread.h>
	 
	/* This is the critical section object (statically allocated). */
	static pthread_mutex_t cs_mutex = PTHREAD_MUTEX_INITIALIZER;
	 
	void f()
	{
	    /* Enter the critical section -- other threads are locked out */
	    pthread_mutex_lock( &cs_mutex );
	 
	    /* Do some thread-safe processing! */
	 
	    /*Leave the critical section -- other threads can now pthread_mutex_lock()  */
	    pthread_mutex_unlock( &cs_mutex );
	}
	 
	int main()
	{
	    f();
	 
	    return 0;
	}


**Example Code For Critical Sections with Win32 API**

	
	/* Sample C/C++, Windows, link to kernel32.dll */
	#include <windows.h>
	 
	static CRITICAL_SECTION cs; /* This is the critical section object -- once initialized,
	                               it cannot be moved in memory */
	                            /* If you program in OOP, declare this as a non-static member in your class */ 
	void f()
	{
	    /* Enter the critical section -- other threads are locked out */
	    EnterCriticalSection(&cs);
	 
	    /* Do some thread-safe processing! */
	 
	    /* Leave the critical section -- other threads can now EnterCriticalSection() */
	    LeaveCriticalSection(&cs);
	}
	 
	int main()
	{
	    /* Initialize the critical section before entering multi-threaded context. */
	    InitializeCriticalSection(&cs);
	 
	    f(); 
	 
	    /* Release system object when all finished -- usually at the end of the cleanup code */
	    DeleteCriticalSection(&cs);
	 
	    return 0;
	}

##### Critical Section �� Mutex ������
Critical Section����һ�����Ķ����޷���֪�����ٽ������߳�������������������ٽ������̹߳��ˣ�û���ͷ��ٽ���Դ��ϵͳ�޷���֪������û�а취�ͷŸ��ٽ���Դ���� Mutex ���г�ʱѡ��ģ����һ�� Mutex ռ����Դ̫�ã���ô��ֱ�ӱ��ͷš����ҿ����������������� Mutex���󣬵��� Critical Section�Ͳ�����

##### Critical Section properties
Typically, critical sections prevent process and thread migration between processors and the preemption of processes and threads by interrupts and other processes and threads.

Critical sections often allow nesting. Nesting allows multiple critical sections to be entered and exited at little cost.
If the scheduler interrupts the current process or thread in a critical section, the scheduler will either allow the currently executing process or thread to run to completion of the critical section, or it will schedule the process or thread for another complete quantum. The scheduler will not migrate the process or thread to another processor, and it will not schedule another process or thread to run while the current process or thread is in a critical section.

Similarly, if an interrupt occurs in a critical section, the interrupt's information is recorded for future processing, and execution is returned to the process or thread in the critical section. Once the critical section is exited, and in some cases the scheduled quantum completes, the pending interrupt will be executed. The concept of scheduling quantum applies to "Round Robin" and similar scheduling policies.

Since critical sections may execute only on the processor on which they are entered, synchronization is only required within the executing processor. This allows critical sections to be entered and exited at almost zero cost. No interprocessor synchronization is required, only instruction stream synchronization. Most processors provide the required amount of synchronization by the simple act of interrupting the current execution state. This allows critical sections in most cases to be nothing more than a per processor count of critical sections entered.

Performance enhancements include executing pending interrupts at the exit of all critical sections and allowing the scheduler to run at the exit of all critical sections. Furthermore, pending interrupts may be transferred to other processors for execution.


##### Critical Section ʹ��ע������
Critical sections should not be used as a long-lived locking primitive. They should be short enough that the critical section will be entered, executed, and exited without any interrupts occurring, neither from hardware much less the scheduler.

##### Critical Section under Linux is *futex*

The 'fast' Windows equal of critical selection in Linux would be a futex, which stands for fast user space mutex. The difference between a futex and a mutex is that with a futex, the kernel only becomes involved when arbitration is required, so you save the overhead of talking to the kernel each time the atomic counter is modified. A futex can also be shared amongst processes, using the means you would employ to share a mutex.

Unfortunately, futexes can be very tricky to implement (PDF).

####Mutex ("mutual exclusion" lock)

Mutex��critical sectionһ����������֤ͬʱֻ��һ���߳̽���ĳ����ͨ������ʵ�ֶ�ĳ��һ��Դ�ķ��ʿ��ơ�Mutex�����趨time out�����Բ���critical sectionһ�����ȡ����һ��ӵ��Mutex���߳��ڷ���֮ǰû�е���ReleaseMutex()����ô���Mutex�ͱ������ˣ����ǵ������̵߳ȴ�(WaitForSingleObject��)���Mutexʱ�����ܷ��أ����õ�һ��WAIT_ABANDONED_0����ֵ��

A mutex (which stands for "mutual exclusion" lock) is a locking or synchronization object that allows multiple threads to synchronize access to shared resources. It is often used to ensure that shared variables are always seen by other threads in a consistent state.

In Windows, the mutexes are both named and un-named. The named mutex is shared between the threads of different process.

��MS Windows��API��CreateMutex(), OpenMutex(), ReleaseMutex(), WaitForSingleObject()��WaitForMultipleObjects()����MFC���CMutex��������ͬ�����ܡ�

In Linux, the mutexes are shared only between the threads of the same process. To achieve the same functionality in Linux, a System V semaphore can be used (����ο���ƪ����).

֧��POSIX���ϵͳ(Linux/Unix)����pthread_mutex_lock()��pthread_mutex_unlock()��

��˵CriticalSection is a user-mode component implemented by the Win32 subsystem, while Mutex is a kernel-mode component. 
Practially, CriticalSection is much faster when there's no actual blocking (due to reduction in user-kernel mode switches), and probably slower when there is blocking (due to more complex implementation). Additionally, since a Mutex is represented by a HANDLE, you can wait on a mutex with a timeout, or with several other handles. Neither option is available with a Critical Section. Mutexes can be named and shared between processes, while CriticalSections are restricted to the threads of a single process.

 

####Semaphore

�ص����м�����ͬʱ������N���߳̿��Խ���һ������Windows�ϵ�API��CreateSemaphore(), OpenSemaphore(), ReleaseSemaphore(), WaitForSingleObject()��WaitForMultipleObjects()��������MFC��CSemaphore�ࡣ

VxWorksû��Mutex, ���ṩ����Semaphore: binary, counting, mutex. ���API��semBCreate, semMCreate, semCCreate, semDelete, semTake, semGive, semFlush

��VxWork�ϣ�Mutex�ᴥ��prority inheritance. If a higher priority task is waiting for a semaphore 
taken a low priority task and the low priority task, its priority will be temporarily changed to the high priority task which is waiting.

####Binary semaphore

���е�ϵͳ��Binary semaphore��Mutex��û�в���ġ����е�ϵͳ�ϣ���Ҫ�Ĳ�����mutexһ��Ҫ�ɻ�����Ľ������ͷš���semaphore���������������ͷţ���ʱ��semaphoreʵ�ʾ��Ǹ�ԭ�ӵı�������ҿ��Լӻ���������semaphore�������ڽ��̼�ͬ����Semaphore��ͬ������������ϵͳ��֧�ֵģ���Mutex�ܷ������������ͷ���δ������˽���mutexֻ���ڱ���critical section����semaphore�����ڱ���ĳ����������ͬ����

####Event

Event��������ʵ��observerģʽ������һ��Event��Ȼ����WaitForSingleObject()����ȴ������̵߳���(set)���Event���Ҿ����õ�һ�����������û�̬����һ��Event�ľ����Ȼ��ͨ��DeviceIoControl()����������������ȥ���������յ��ⲿ���ߴ��������ݰ��͵������Event��Windows��API��CreateEvent(), OpenEvent(), SetEvent(), WaitForSingleObject(), WaitForMultipleObjects()��MFC������CEvent�ࡣ

####Spin lock

spin lock��һ���ں�̬���spin lock��semaphore����Ҫ������spin lock��busy waiting����semaphore��sleep�����ڿ���sleep�Ľ�����˵��busy waiting��Ȼû�����塣���ڵ�CPU��ϵͳ��busy waiting��Ȼ��û���壨û��CPU�����ͷ���������ˣ�ֻ�ж�CPU���ں�̬�ǽ��̿ռ䣬�Ż��õ�spin lock��Linux kernel��spin lock�ڷ�SMP������£�ֻ�ǹ�irq��û�б�Ĳ���������ȷ���öγ�������в��ᱻ��ϡ���ʵҲ��������mutex�����ã����л��� critical section�ķ��ʡ�����mutex���ܱ����жϵĴ�ϣ�Ҳ�������жϴ�������б����á���spin lockҲһ��û�б�Ҫ���ڿ���sleep�Ľ��̿ռ䡣
