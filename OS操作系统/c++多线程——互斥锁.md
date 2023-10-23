C++中多线程的实现方式有多种，其中可以使用操作系统相关的线程API，如在Linux上，可以使用pthread库；也可以使用boost::thread库或者自从C++ 11开始支持的std::thread1。

pthread库是POSIX线程库，是一套线程API，它定义了一套标准的线程操作函数，可以在多种平台上使用。boost::thread库是一个跨平台的C++多线程库，它提供了一些高级的线程操作接口。std::thread是C++11标准中提供的多线程库，它提供了一些高级的线程操作接口1。

# 互斥锁
在线程里也有这么一把锁——互斥锁（mutex），互斥锁是一种简单的加锁的方法来控制对共享资源的访问，互斥锁只有两种状态,即上锁( lock )和解锁( unlock )。

互斥锁的操作流程如下：

1. 在访问共享资源后临界区域前，对互斥锁进行加锁；
2. 在访问完成后释放互斥锁导上的锁。在访问完成后释放互斥锁导上的锁；
3. 对互斥锁进行加锁后，任何其他试图再次对互斥锁加锁的线程将会被阻塞，直到锁被释放。对互斥锁进行加锁后，任何其他试图再次对互斥锁加锁的线程将会被阻塞，直到锁被释放。
函数接口：

```cpp
# include <pthread.h>
# include <time.h>
// 初始化一个互斥锁。
int pthread_mutex_init(pthread_mutex_t *mutex,
const pthread_mutexattr_t*attr);

// 对互斥锁上锁，若互斥锁已经上锁，则调用者一直阻塞，
// 直到互斥锁解锁后再上锁。
int pthread_mutex_lock(pthread_mutex_t *mutex);

// 调用该函数时，若互斥锁未加锁，则上锁，返回 0；
// 若互斥锁已加锁，则函数直接返回失败，即 EBUSY。
int pthread_mutex_trylock(pthread_mutex_t *mutex);

// 当线程试图获取一个已加锁的互斥量时，pthread_mutex_timedlock 互斥量
// 原语允许绑定线程阻塞时间。即非阻塞加锁互斥量。
int pthread_mutex_timedlock(pthread_mutex_t *restrict mutex,
const struct timespec*restrict abs_timeout);

// 对指定的互斥锁解锁。
int pthread_mutex_unlock(pthread_mutex_t *mutex);

// 销毁指定的一个互斥锁。互斥锁在使用完毕后，
// 必须要对互斥锁进行销毁，以释放资源。
int pthread_mutex_destroy(pthread_mutex_t *mutex);
```

## 样例使用
```cpp
/*

* mutexcnt.c - 加上互斥锁（mutex lock）的多线程同步计数器
* 两个线程并发的给共享变量自增，观察是否有BUG
 */

# include <stdio.h>
# include <stdlib.h>
# include <unistd.h>
# include <pthread.h>

void *thread(void*vargp);          /*Thread函数声明*/

/*共享的全局变量*/
volatile long long cnt = 0;               /*计数器*/

pthread_mutex_t count_mutex;        /*互斥锁声明*/

int main(int argc, char **argv)
{
    long long niters;                     /* 单线程的累加次数 */
    pthread_t tid1, tid2;

    /* 检查输入格式 */
    if (argc != 2) { 
 printf("输入格式: %s <次数>\n", argv[0]);
 exit(0);
    }
    niters = atol(argv[1]);
    
    /* 互斥锁初始化 */
    pthread_mutex_init(&count_mutex,NULL);    
    /* 创建两个线程去执行thread函数，参数为niters */
    pthread_create(&tid1, NULL, thread, &niters);
    pthread_create(&tid2, NULL, thread, &niters);
    /* 等待两个线程并发的执行结束 */
    pthread_join(tid1, NULL);
    pthread_join(tid2, NULL);

    /* 检查计数器的值是否正确 */
    if (cnt != (2 * niters))
 printf("发现BUG! cnt=%lld\n", cnt);
    else
 printf("结果正确。 cnt=%lld\n", cnt);
    exit(0);
}

/*线程函数*/
void *thread(void*vargp)
{
    long long i, niters = *((long long*)vargp);

    for (i = 0; i < niters; i++){
        pthread_mutex_lock(&count_mutex);   //进入区
        cnt++;  //临界区
        pthread_mutex_unlock(&count_mutex); //退出区
    }
       
    return NULL;
}
```
# 自旋锁
参考
<https://www.cnblogs.com/cxuanBlog/p/11679883.html>

自旋锁的定义：当一个线程尝试去获取某一把锁的时候，如果这个锁此时已经被别人获取(占用)，那么此线程就无法获取到这把锁，该线程将会等待，间隔一段时间后会再次尝试获取。这种采用循环加锁 -> 等待的机制被称为自旋锁(spinlock)。
## 代码应用
```cpp
/* 
 * spincnt.c - 加上自旋锁（spin lock）的多线程同步计数器
 * 两个线程并发的给共享变量自增，观察是否有BUG
 */

#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <pthread.h>

void *thread(void *vargp);          /* Thread函数声明 */

/* 共享的全局变量 */
volatile long long cnt = 0;               /* 计数器 */

pthread_spinlock_t count_spinlock;  /* 自旋锁声明 */

int main(int argc, char **argv) 
{
    long long niters;                     /* 单线程的累加次数 */
    pthread_t tid1, tid2;
    
    /* 检查输入格式 */
    if (argc != 2) { 
 printf("输入格式: %s <次数>\n", argv[0]);
 exit(0);
    }
    niters = atol(argv[1]);
    
    /* 互斥锁初始化 */
    pthread_spin_init(&count_spinlock,0);    
    /* 创建两个线程去执行thread函数，参数为niters */
    pthread_create(&tid1, NULL, thread, &niters);
    pthread_create(&tid2, NULL, thread, &niters);
    /* 等待两个线程并发的执行结束 */
    pthread_join(tid1, NULL);
    pthread_join(tid2, NULL);

    /* 检查计数器的值是否正确 */
    if (cnt != (2 * niters))
 printf("发现BUG! cnt=%lld\n", cnt);
    else
 printf("结果正确。 cnt=%lld\n", cnt);
    exit(0);
}

/* 线程函数 */
void *thread(void *vargp) 
{
    long long i, niters = *((long long *)vargp);

    for (i = 0; i < niters; i++){
        pthread_spin_lock(&count_spinlock);   //进入区
        cnt++;  //临界区
        pthread_spin_unlock(&count_spinlock); //退出区
    }
        
    return NULL;
}
```

# 读写锁
参考

<https://zhuanlan.zhihu.com/p/62363777>
<https://zhuanlan.zhihu.com/p/135983375>

概念：允许多个读出，但只允许一个写入的需求。

读写锁与互斥量类似，不过读写锁允许更改的并行性，也叫共享互斥锁。

互斥量要么是锁住状态，要么就是不加锁状态，而且一次只有一个线程可以对其加锁。
读写锁可以有3种状态：读模式下加锁状态、写模式加锁状态、不加锁状态。
![img](https://img2023.cnblogs.com/blog/2740326/202305/2740326-20230524151713374-484117895.png)

# 条件变量
- <https://zhuanlan.zhihu.com/p/136431212>

与互斥锁不同，条件变量是用来等待而不是用来上锁的。条件变量用来自动阻塞一个线程，直 到某特殊情况发生为止。通常条件变量和互斥锁同时使用。

条件变量使我们可以睡眠等待某种条件出现。条件变量是利用线程间共享的全局变量进行同步 的一种机制，主要包括两个动作：

>一个线程等待"条件变量的条件成立"而挂起
>
>另一个线程使 “条件成立”（给出条件成立信号）
- 好像就是生产者和消费者
  
**条件变量使用步骤：**

1. 初始化：init()或者pthread_cond_tcond=PTHREAD_COND_INITIALIER；属性置为NULL；

2. 等待条件成立：pthread_wait，pthread_timewait.wait()释放锁,并阻塞等待条件变量为真 timewait()设置等待时间,仍未signal,返回ETIMEOUT(加锁保证只有一个线程wait)；

3. 激活条件变量：pthread_cond_signal,pthread_cond_broadcast(激活所有等待线程)

4. 清除条件变量：destroy;无线程等待,否则返回EBUSY清除条件变量:destroy;无线程等待,否则返回EBUSY
```cpp
# include <pthread.h>
// 初始化条件变量
int pthread_cond_init(pthread_cond_t *cond,pthread_condattr_t*cond_attr);

// 阻塞等待
int pthread_cond_wait(pthread_cond_t *cond,pthread_mutex_t*mutex);

// 超时等待
int pthread_cond_timewait(pthread_cond_t *cond,pthread_mutex*mutex,const timespec *abstime);

// 解除所有线程的阻塞
int pthread_cond_destroy(pthread_cond_t *cond);

// 至少唤醒一个等待该条件的线程
int pthread_cond_signal(pthread_cond_t *cond);

// 唤醒等待该条件的所有线程
int pthread_cond_broadcast(pthread_cond_t *cond);  
```

## 参考代码
打印10次ABC
```cpp

void *print(void *thread_id)
{
    char id = *(char *)thread_id;

    for (int i = 0; i < 10; ++i)
    {
        pthread_mutex_lock(&mutex); //互斥锁
        while (id != current_thread)
        {
            pthread_cond_wait(&cond, &mutex); // 使当前线程进入等待状态，等待条件变量满足特定条件。
        }

        printf("current_thread%c tid = %lld\n", id,pthread_self());

        current_thread = (current_thread - 'A' + 1) % 3 + 'A';

        pthread_cond_broadcast(&cond); // 唤醒所有等待该条件变量的线程。
        pthread_mutex_unlock(&mutex);
    }

    pthread_exit(NULL);
}

```

# 信号量
- <https://zhuanlan.zhihu.com/p/138497069>


编程时可根据操作信号量值的结果判断是否对公共资源具有访问的权限，当信号量值大于 0 时，则可以访问，否则将阻塞。

PV 原语是对信号量的操作，一次 P 操作使信号量减１，一次 V 操作使信号量加１。

PV成对出现
