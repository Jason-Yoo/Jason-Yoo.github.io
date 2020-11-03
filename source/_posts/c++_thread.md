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

成员：lock,unlock,try_lock(（）；

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

##### future其他成员函数、shared_future、atomic

