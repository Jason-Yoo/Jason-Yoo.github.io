---
title: c++多线程并发笔记
typora-root-url: ../../source
typora-copy-images-to: ../images/c++_thread.assets
mathjax: true
categories: c++
tags:
- c++
---

#### 互斥量：Mutex类  

lock unlock 成对使用，Lock是阻塞式的，有一个地方lock, 第二个地方的lock会一直卡在lock里，直到lock成功；

Lock_guard类模版（推荐使用），直接取代lock和unlock；Lock_guard的构造函数里执行了lock，Lock_guard的析构函数里执行了unlock；局部变量超出作用域名时，生命周期结束并执行析构函数；

<!--more-->

加 { } 可以提前结束里面的局部变量，提前执行循环里所定义局部变量的析构函数；

锁住的代码的多少称为锁的粒度，粒度一般用粗细来描述，锁住的越少，执行的效率越高； 

##### 死锁：两个线程争夺资源的过程中，两个互斥量嵌套使用，且使用的顺序不一致；

产生条件：必须有两个互斥量（这里比作金锁，银锁），保护不同的数据共享块；

线程A和B，线程A先锁金锁，后锁银锁；线程B先锁银锁，后锁金锁；

当线程A，金锁lock 成功后，开始进行银锁的lock,这时候，线程后台切换到了线程B，导致银锁lock , 线程B开始执行金锁lock，卡在了金锁lock，这时候线程管理又切换到线程A，线程A开在了银锁lock，这样就造成了死锁，两个线程都卡在了lock函数里，无法解开；

##### 死锁的一般解决方案：

保证互斥量的调用顺序一致；

std::lock函数模版:一次锁住两个或者两个以上的互斥量，要么锁住两个，要么两个都没锁住 ，只锁住一个的时候会主动放开，不会造成死锁问题；（至少两个）

#### Unique_lock取代Lock_guard：

第二个参数（和Lock_guard一样）：adopt_lock 起到标记作用，标记互斥量已经锁过了，不需要构造函数中再lock；

 try_to_lock：不能之前lock，尝试用mutex的lock去锁定，但如果没有锁定成功，我也会立即返回，不会阻塞在哪里等待，没有拿到锁可以干其他的事情，不会使线程阻塞在那里傻等；

defer_lock：不能之前lock，初始化了一个没加锁的mutex;

成员：lock，unlock，try_lock（）；

release（）： 返回所管理的mutex对象指针，并释放所有权；

##### 所有权传递问题：

Unique_lock是在管理mutex指针对象，只和一个mutex绑定到一起，拥有mutex的所有权，可以讲所有权转移给其她的unique_lock，可以移动所有权但不能复制， 

### 单例设计模式：

单例类：整个项目中，有某个或者某些特殊的类，属于该类的对象，我只能创建一个，多了创建不了；

方法：把构造函数私有化；

```c++
std::mutex resource_mutex;
class mycas{  //单例类
private:
				mycas() {}//私有化了构造函数 
private:
  static mycas *m_Instance;//静态成员变量
public:
  static mycas *getInstance()
  {
    if(m_Instance == NULL )//双重锁定，解决多个线程创建单例类时的临界不安全问题，提高效率
    {
      std::unique_lock<std::mutex>mymutex(resource_mutex);//自动加锁
      if(m_Instance == NULL )
      {
        m_Instance = new mycas();
        static cGarhuishou cl; //为了释放m_Instance所指的内存空间
      } 
    }
    else  
      return  m_Instance;
  }
  class cGarhuishou //类中套类，用来释放new的mycas
  {
    public:
    ~cGarhuishou() //类的析构函数
    {
       if（mycas::m_Instance）//如果类对象指针不为空
       {
         delete mycas::m_Instance;
         mycas::m_Instance = NULL;
       }     
    }
  };
};
//类静态变量初始化
mycas *mycas::m_Instance = NULL;
int main()
{
  mycas *p_A = mycas::getInstance();//创建一个单例类对象，且只能创建一次
  mycas *p_B = mycas::getInstance();//其实是与p_A指向的空间相同的；
  // mycas p_A  的方式是无法创建的；
}

```

#### 单例设计模式共享数据问题分析与解决

在自己或者其他线程中需要创建使用单例类对象，我们可能会面临getInstance（）这种成员函数要互斥；

两个线程都要创建单例类等时候，存在调用getInstance（）时判断条件由于线程自动切换产生歧义，导致创建了两个单例类，这个时候采用互斥量可以解决；

std::call_once(); 功能：保证函数只被调用一次，具备互斥量的能力，且效率上比互斥量消耗的资源更少(据说)，需要与一个标记结合使用，这个标记std::once_flag;

####  条件变量 condition_variable、wait、notify_one、notify_all

std::condition_variable是个类，他会生成一个条件变量对象，但要与互斥量配合使用;

```c++
condition_variable my_condvar;
std::unique_lock<std::mutex>mymutex(resource_mutex);//自动加锁
//如果第二个参数返回false ，则wait()将解锁互斥量，并堵塞到本行；
//堵塞到什么时候为止呢？堵塞到其他某个线程调用notify_one（）成员函数为止；
//如果第二个参数什么都不填，第一次调用默认条件不成立，那么默认就是false；
//当其他线程调用notify_one（）成员函数时，那么这里的my_condvar.wait将会被唤醒（原本是堵塞的状态）
//被唤醒后，他会不断的重新尝试获取互斥量锁，如果获取不到还是会卡在这个位置，如果获取到就继续往下执行
  //往下执行时，会先判断wait的第二个参数返回值，如果还是返回false，则就解锁并且再次进入睡眠状态等待唤醒
	//往下执行时，如果第二个参数返回值为true,那么就开始执行下面的流程，此时互斥量还是被锁的状态 
	//如果wait没有第二个参数，则wait返回，流程走下来，互斥量处于被锁；
my_condvar.wait(mymutex,[this] {//一个lambda表达式就是一个可调用对象（函数）
	if(！somethings.empty())//如果消息队列不为空 
		return true;
  return false;
});

my_condvar.notify_one //不一定就是唤醒一次 wait就执行一次，因为线程切换原因，可能唤醒了好几次只有一次有用；
my_condvar.notify_all//唤醒多个wait
```

##### async、future、packaged_task、promise

std::async 是个函数模版，用来启动一个异步任务，启动起来以后，他返回一个std::future 对象，它是一个类模版 

什么叫启动一个异步任务？就是自动创建一个线程，并开始执行线程入口函数，返回的std::future 对象里就含有线程返回的结果 ，可以通过调用std::future对象的成员函数get，可以获取线程返回的结果，

future提供一种访问异步操作结果的机制，就是说线程的执行结果你可能没有办法马上拿到，但是将来线程执行完毕后，你就可以得到结果，它保存在future的成员函数里；

如果get函数拿不到那个线程的返回值，就会一直卡在那个位置，直到能够拿到，且只能调用一次；

**std::packaged_task：打包任务，把任务包装起来。** 类模板，它的模板参数是各种可调用对象，通过packaged_task把各种可调用对象包装起来，方便将来作为线程入口函数来调用。

```c++
int main() {
	cout << "main" << "threadid = " << std::this_thread::get_id() << endl;
	//我们把函数mythread通过packaged_task包装起来
  //参数是一个int，返回值类型是int
  std::packaged_task<int(int)> mypt(mythread);
	std::thread t1(std::ref(mypt), 1);
	t1.join();
	std::future<int> result = mypt.get_future(); 
	//std::future对象里包含有线程入口函数的返回结果，这里result保存mythread返回的结果。
	cout << result.get() << endl;
	return 0;
}
```

std::promise，类模板 我们能够在某个线程中给它赋值，然后我们可以在其他线程中，把这个值取出来

总结：通过promise保存一个值，在将来某个时刻我们通过吧一个future绑定到这个promise上，来得到绑定的值

注意：使用thread时，必须 join() 或者 detach() 否则程序会报异常

**小结：**

我们学习这些东西的目的并不是，要把他们都用到实际开发中。

相反，如果我们能够用最少的东西写出一个稳定的，高效的多线程程序，更值得赞赏。

我们为了成长必须阅读一些高手写的代码，从而实现自己代码的积累；

##### future其他成员函数、shared_future、atomic

**一、std::future 的成员函数**

**std::future_status status** = result.**wait_for**(std::chrono::seconds(几秒));

卡住当前流程，等待std::async()的异步任务运行一段时间，然后返回其状态std::future_status。如果std::async()的参数是std::launch::deferred（延迟执行），则不会卡住主流程。

std::future_status是枚举类型，表示异步任务的执行状态。类型的取值有 

 std::future_status::timeout
		std::future_status::ready
		std::future_status::deferred

因为get()函数的设计是一个移动语义，相当于将result中的值移动到了a中，再次get就报告了异常。

**二、std::shared_future：也是个类模板**

std::future的 get() 成员函数是转移数据

std::shared_future 的 get()成员函数是复制数据

**三、std::atomic原子操作**

3.1 原子操作概念引出范例：
		 互斥量：多线程编程中 用于保护共享数据：先锁住， 操作共享数据， 解锁。

有两个线程，对一个变量进行操作，一个线程读这个变量的值，一个线程往这个变量中写值。

即使是一个简单变量的读取和写入操作，如果不加锁，也有可能会导致读写值混乱（一条C语句会被拆成3、4条汇编语句来执行，所以仍然有可能混乱）；

大家可以把原子操作理解成一种：不需要用到互斥量加锁（无锁）技术的多线程并发编程方式。

原子操作：在多线程中不会被打断的程序执行片段。

从效率上来说，原子操作要比互斥量的方式效率要高。

互斥量的加锁一般是针对一个代码段，而原子操作针对的一般都是一个变量。

原子操作，一般都是指“不可分割的操作”；也就是说这种操作状态要么是完成的，要么是没完成的，不可能出现半完成状态。

原子操作实质上是：不允许在进行原子对象操作时进行CPU的上下文切换。

std::atomic来代表原子操作，是个类模板。其实std::atomic是用来封装某个类型的值的；

一般atomic原子操作，针对++，–，+=，-=，&=，|=，^=是支持的，其他操作不一定支持。

二、std::async深入理解

2.1 std::async参数详述，async 用来创建一个异步任务

延迟调用参数 std::launch::deferred【延迟调用】，std::launch::async【强制创建一个线程】

std::async()我们一般不叫创建线程（他能够创建线程），我们一般叫它创建一个异步任务。

std::async和std::thread最明显的不同，就是 async 有时候并不创建新线程。

①如果用std::launch::deferred 来调用async？

延迟到调用 get() 或者 wait() 时执行，如果不调用就不会执行

②如果用std::launch::async来调用async？

强制这个异步任务在新线程上执行，这意味着，系统必须要创建出新线程来运行入口函数。

③如果同时用 std::launch::async | std::launch::deferred

这里这个或者关系意味着async的行为可能是 std::launch::async 创建新线程立即执行， 也可能是 std::launch::deferred 没有创建新线程并且延迟到调用get()执行，由**系统根据实际情况来决定采取**哪种方案

④不带额外参数 std::async(mythread)，只给async 一个入口函数名，此时的系统给的默认值是 std::launch::async | std::launch::deferred 和 ③ 一样，有系统自行决定异步还是同步运行。

2.2 std::async和std::thread()区别：

std::thread()如果系统资源紧张可能出现创建线程失败的情况，如果创建线程失败那么程序就可能崩溃，而且不容易拿到函数返回值（不是拿不到）
std::async()创建异步任务。可能创建线程也可能不创建线程，并且容易拿到线程入口函数的返回值；

由于系统资源限制：

①如果用std::thread创建的线程太多，则可能创建失败，系统报告异常，崩溃。

②如果用std::async，一般就不会报异常，因为如果系统资源紧张，无法创建新线程的时候，async不加额外参数的调用方式就不会创建新线程。而是在后续调用get()请求结果时执行在这个调用get()的线程上。

如果你强制async一定要创建新线程就要使用 std::launch::async 标记。承受的代价是，系统资源紧张时可能崩溃。

③根据经验，一个程序中线程数量 不宜超过100~200 。

2.3 async不确定性问题的解决

不加额外参数的async调用时让系统自行决定，是否创建新线程。

std::future result = std::async(mythread);

问题焦点在于这个写法，任务到底有没有被推迟执行。

通过wait_for返回状态来判断：

```c++
std::future_status status = result.wait_for(std::chrono::seconds(6));
//std::future_status status = result.wait_for(6s);
	if (status == std::future_status::timeout) {
		//超时：表示线程还没有执行完
		cout << "超时了，线程还没有执行完" << endl;
	}
	else if (status == std::future_status::ready) {
		//表示线程成功放回
		cout << "线程执行成功，返回" << endl;
		cout << result.get() << endl;
	}
	else if (status == std::future_status::deferred) {
		cout << "线程延迟执行" << endl;
		cout << result.get() << endl;
	}
```

**递归独占互斥量 std::recursive_mutex**

std::mutex 独占式互斥量

std::recursive_mutex：允许在同一个线程中同一个互斥量多次被 lock() ，（但是递归加锁的次数是有限制的，太多可能会报异常），效率要比mutex低。

如果你真的用了 recursive_mutex 要考虑代码是否有优化空间，如果能调用一次 lock()就不要调用多次

**带超时的互斥量 std::timed_mutex 和 std::recursive_timed_mutex**

*5.1 std::timed_mutex：是待超时的独占互斥量*

- try_lock_for()：

等待一段时间，如果拿到了锁，或者超时了未拿到锁，就继续执行（有选择执行）如下：

```c++
std::chrono::milliseconds timeout(100);
if (my_mymutex.try_lock_for(timeout)){
    //......拿到锁返回ture
}
else{
    std::chrono::milliseconds sleeptime(100);
    std::this_thread::sleep_for(sleeptime);
}
```

- try_lock_until()：

参数是一个未来的时间点，在这个未来的时间没到的时间内，如果拿到了锁头，流程就走下来，如果时间到了没拿到锁，流程也可以走下来。

```c++
std::chrono::milliseconds timeout(100);
if (my_mymutex.try_lock_until(chrono::steady_clock::now() + timeout)){
    //......拿到锁返回ture
}
else{
    std::chrono::milliseconds sleeptime(100);
    std::this_thread::sleep_for(sleeptime);
}
```

两者的区别就是一个参数是时间段，一个参数是时间点

std::recursive_timed_mutex：是待超时的递归独占互斥量；

虚假唤醒：

notify_one或者notify_all唤醒wait()后，实际有些线程可能不满足唤醒的条件，就会造成虚假唤醒，可以在wait中再次进行判断解决虚假唤醒。

解决：wait中要有第二个参数（lambda），并且这个lambda中要正确判断所处理的公共数据是否存在。

**浅谈线程池：**
场景设想：服务器程序， 每来一个客户端，就创建一个新线程为这个客户提供服务。

问题：

1、2万个玩家，不可能给每个玩家创建一个新线程，此程序写法在这种场景下不通。

2、程序稳定性问题：编写代码中，“时不时地突然”创建一个线程，这种写法，一般情况下不会出错，但是不稳定的；

线程池：把一堆线程弄到一起，统一管理。这种统一管理调度，循环利用的方式，就叫做线程池。

实现方式：程序启动时，一次性创建好一定数量的线程。这种方式让人更放心，觉得程序代码更稳定。

