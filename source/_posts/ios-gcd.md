title: "GCD相关知识"
date: 2015-04-21 16:28:55
tags: "GCD"
categories: ["iOS","Objective-C"]
---

### IOS GCD

> [GCD 深入理解（一）](http://www.cocoachina.com/industry/20140428/8248.html)
> [GCD 深入理解（二）](http://www.cocoachina.com/industry/20140515/8433.html)
> [iOS GCD使用指南](http://blog.csdn.net/zhangao0086/article/details/38904923)
> [GCD介绍](http://www.cnblogs.com/lovesmile/archive/2012/09/13/2683468.html)
> [多线程编程4 - GCD](http://blog.csdn.net/q199109106q/article/details/8566300)
> [iOS多线程编程之Grand Central Dispatch(GCD)介绍和使用](http://blog.csdn.net/totogo2010/article/details/8016129)
> [利用dispatch_once创建单例](http://bj007.blog.51cto.com/1701577/649413/)

GCD是Grand Central Dispatch的简称,它是基于C语言的。如果使用GCD，完全由系统管理线程，我们不需要编写线程代码。只需定义想要执行的任务,然后添加到适当的调度队列(dispatch queue)。GCD会负责创建线程和调度你的任务，系统直接提供线程管理

#### GCD 术语
要理解 GCD ，你要先熟悉与线程和并发相关的几个概念。这两者都可能模糊和微妙，所以在开始 GCD 之前先简要地回顾一下它们。
 
* __Serial vs. Concurrent 串行 vs. 并发__
这些术语描述当任务相对于其它任务被执行，任务串行执行就是每次只有一个任务被执行，任务并发执行就是在同一时间可以有多个任务被执行。

* __Synchronous vs. Asynchronous 同步 vs. 异步__
在 GCD 中，这些术语描述当一个函数相对于另一个任务完成，此任务是该函数要求 GCD 执行的。一个同步函数只在完成了它预定的任务后才返回。
一个异步函数，刚好相反，会立即返回，预定的任务会完成但不会等它完成。因此，一个异步函数不会阻塞当前线程去执行下一个函数。
注意——当你读到同步函数“阻塞（Block）”当前线程，或函数是一个“阻塞”函数或阻塞操作时，不要被搞糊涂了！动词“阻塞”描述了函数如何影响它所在的线程而与名词“代码块（Block）”没有关系。代码块描述了用 Objective-C 编写的一个匿名函数，它能定义一个任务并被提交到 GCD 。

* __Critical Section 临界区__
就是一段代码不能被并发执行，也就是，两个线程不能同时执行这段代码。这很常见，因为代码去操作一个共享资源，例如一个变量若能被并发进程访问，那么它很可能会变质（译者注：它的值不再可信）。
 
* __Race Condition 竞态条件__
这种状况是指基于特定序列或时机的事件的软件系统以不受控制的方式运行的行为，例如程序的并发任务执行的确切顺序。竞态条件可导致无法预测的行为，而不能通过代码检查立即发现。
 
* __Deadlock 死锁__
两个（有时更多）东西——在大多数情况下，是线程——所谓的死锁是指它们都卡住了，并等待对方完成或执行其它操作。第一个不能完成是因为它在等待第二个的完成。但第二个也不能完成，因为它在等待第一个的完成。
 
* __Thread Safe 线程安全__
线程安全的代码能在多线程或并发任务中被安全的调用，而不会导致任何问题（数据损坏，崩溃，等）。线程不安全的代码在某个时刻只能在一个上下文中运行。一个线程安全代码的例子是 NSDictionary 。你可以在同一时间在多个线程中使用它而不会有问题。另一方面，NSMutableDictionary 就不是线程安全的，应该保证一次只能有一个线程访问它。
 
* __Context Switch 上下文切换__
一个上下文切换指当你在单个进程里切换执行不同的线程时存储与恢复执行状态的过程。这个过程在编写多任务应用时很普遍，但会带来一些额外的开销。
 
* __Concurrency vs Parallelism 并发与并行__
并发和并行通常被一起提到，所以值得花些时间解释它们之间的区别。并发代码的不同部分可以“同步”执行。然而，该怎样发生或是否发生都取决于系统。多核设备通过并行来同时执行多个线程；然而，为了使单核设备也能实现这一点，它们必须先运行一个线程，执行一个上下文切换，然后运行另一个线程或进程。这通常发生地足够快以致给我们并发执行地错觉

* __Queues 队列__
GCD 提供有 dispatch queues 来处理代码块，这些队列管理你提供给 GCD 的任务并用 FIFO 顺序执行这些任务。这就保证了第一个被添加到队列里的任务会是队列中第一个开始的任务，而第二个被添加的任务将第二个开始，如此直到队列的终点。所有的调度队列（dispatch queues）自身都是线程安全的，你能从多个线程并行的访问它们。 GCD 的优点是显而易见的，即当你了解了调度队列如何为你自己代码的不同部分提供线程安全。关于这一点的关键是选择正确类型的调度队列和正确的调度函数来提交你的工作。

* __Concurrent Queues 并发队列__
在并发队列中的任务能得到的保证是它们会按照被添加的顺序开始执行，但这就是全部的保证了。任务可能以任意顺序完成，你不会知道何时开始运行下一个任务，或者任意时刻有多少 Block 在运行。再说一遍，这完全取决于 GCD 。

* __Queue Types 队列类型__
__首先__，系统提供给你一个叫做 主队列（main queue） 的特殊队列。和其它串行队列一样，这个队列中的任务一次只能执行一个。然而，它能保证所有的任务都在主线程执行，而主线程是唯一可用于更新 UI 的线程。这个队列就是用于发生消息给 UIView 或发送通知的。系统同时提供给你好几个并发队列。它们叫做 全局调度队列（Global Dispatch Queues） 。目前的四个全局队列有着不同的优先级：background、low、default 以及 high。要知道，Apple 的 API 也会使用这些队列，所以你添加的任何任务都不会是这些队列中唯一的任务。__最后__，你也可以创建自己的串行队列或并发队列。这就是说，至少有五个队列任你处置：主队列、四个全局调度队列，再加上任何你自己创建的队列。

----

#### GCD基本方法
##### 创建和管理dispatch queue
1. 获得全局并发Dispatch Queue (concurrent dispatch queue)
1.1 并发dispatch queue可以同时并行地执行多个任务,不过并发queue仍然按先进先出的顺序来启动任务。并发queue会在之前的任务完成之前就出列下一个任务并开始执行。并发queue同时执行的任务数量会根据应用和系统动态变化,各种因素包括:可用核数量、其它进程正在执行的工作数量、其它串行dispatch queue中优先任务的数量等.
1.2 虽然dispatch queue是引用计数的对象,但你不需要retain和release全局并发queue。因为这些queue对应用是全局的,retain和release调用会被忽略。你也不需要存储这三个queue的引用,每次都直接调用dispatch_get_global_queue获得queue就行了
1.3 系统给每个应用提供三个并发dispatch queue,整个应用内全局共享,三个queue的区别是优先级。你不需要显式地创建这些queue,使用dispatch_get_global_queue函数来获取这三个queue:
```objc
dispatch_queue_t  queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0); 
```
第一个参数用于指定优先级，分别使用`DISPATCH_QUEUE_PRIORITY_HIGH`和`DISPATCH_QUEUE_PRIORITY_LOW`两个常量来获取高和低优先级的两个queue；第二个参数目前未使用到，默认0即可
2. 创建串行Dispatch Queue (serial dispatch queue)
 2.1 应用的任务需要按特定顺序执行时,就需要使用串行Dispatch Queue,串行queue每次只能执行一个任务。你可以使用串行queue来替代锁,保护共享资源 或可变的数据结构。和锁不一样的是,串行queue确保任务按可预测的顺序执行。而且只要你异步地提交任务到串行queue,就永远不会产生死锁
 2.2 你必须显式地创建和管理所有你使用的串行queue,应用可以创建任意数量的串行queue,但不要为了同时执行更多任务而创建更多的串行queue。如果你需要并发地执行大量任务,应该把任务提交到全局并发queue
 2.3 利用`dispatch_queue_create`函数创建串行queue,两个参数分别是queue名和一组queue属性
```objc
dispatch_queue_t queue = dispatch_queue_create("cn.itcast.queue", NULL);
```
第一个参数是队列的名称，一般是使用倒序的全域名。虽然可以不给队列指定一个名称，但是有名称的队列可以让我们在遇到问题时更好调试；当第二个参数为nil时返回Serial Dispatch Queue，如上面那个例子，当指定为`DISPATCH_QUEUE_CONCURRENT`时返回Concurrent Dispatch Queue。
需要注意一点，如果是在OS X 10.8或iOS 6以及之后版本中使用，Dispatch Queue将会由ARC自动管理，如果是在此之前的版本，需要自己手动释放(`dispatch_release(queue)`)
3. 运行时获得公共Queue
GCD提供了函数让应用访问几个公共dispatch queue:
3.1 使用`dispatch_get_current_queue`函数作为调试用途,或者测试当前queue的标识。在block对象中调用这个函数会返回block提交到的queue(这个时候queue应该正在执行中)。在block对象之外调用这个函数会返回应用的默认并发queue。
3.2 使用`dispatch_get_main_queue`函数获得应用主线程关联的串行dispatch queue
3.3 使用`dispatch_get_global_queue`来获得共享的并发queue

##### 用 dispatch_async 处理后台任务
为了避免界面在处理耗时的操作时卡死，比如读取网络数据，IO,数据库读写等，我们会在另外一个线程中处理这些操作，然后通知主线程更新界面。
```objc
dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{  
    // 例:下载图片 
    dispatch_async(dispatch_get_main_queue(), ^{  
        // 例:显示图片  
    });  
}); 
```

##### 使用 dispatch_after 延后工作
dispatch_after能让我们添加进队列的任务延时执行:
```objc
//NSEC_PER_SEC:秒 NSEC_PER_MSEC:毫秒
dispatch_time_t popTime = dispatch_time(DISPATCH_TIME_NOW, (Int64)(10 * NSEC_PER_SEC))；
dispatch_after(popTime, dispatch_get_main_queue(), ^(void){ 
    //延迟10秒执行
});
```

##### 利用dispatch_once创建单例
 该函数接收一个`dispatch_once`用于检查该代码块是否已经被调度的谓词（是一个长整型，实际上作为BOOL使用）。它还接收一个希望在应用的生命周期内仅被调度一次的代码块，对于本例就用于shared实例的实例化。
dispatch_once不仅意味着代码仅会被运行一次，而且还是线程安全的，这就意味着你不需要使用诸如@synchronized之类的来防止使用多个线程或者队列时不同步的问题。
```objc
static AccountManager *sharedAccountManagerInstance = nil;  //需要实例化的类
static dispatch_once_t onceToken;
dispatch_once(&onceToken, ^{ 
    sharedAccountManagerInstance = [[self alloc] init];
}); 
```
`dispatch_once()`以线程安全的方式执行且仅执行其代码块一次。试图访问临界区（即传递给 dispatch_once 的代码）的不同的线程会在临界区已有一个线程的情况下被阻塞，直到临界区完成为止。

##### dispatch_barrier_async的使用
`dispatch_barrier_async`就如同它的名字一样，在队列执行的任务中增加“栅栏”，在增加“栅栏”之前已经开始执行的block将会继续执行，当`dispatch_barrier_async`开始执行的时候其他的block处于等待状态，`dispatch_barrier_async`的任务执行完后，其后的block才会执行。
简单点说，就是`dispatch_barrier_async`是在前面的任务执行结束后它才执行，而且它后面的任务等它执行完成之后才会执行
```objc
dispatch_queue_t queue = dispatch_queue_create("test.gcd", DISPATCH_QUEUE_CONCURRENT);  
dispatch_async(queue, ^{  
    [NSThread sleepForTimeInterval:2];  
    NSLog(@"dispatch_async1");  
});  
dispatch_async(queue, ^{  
    [NSThread sleepForTimeInterval:4];  
    NSLog(@"dispatch_async2");  
});  
dispatch_barrier_async(queue, ^{  
    NSLog(@"dispatch_barrier_async");  
    [NSThread sleepForTimeInterval:4];  
  
});  
dispatch_async(queue, ^{  
    [NSThread sleepForTimeInterval:1];  
    NSLog(@"dispatch_async3");  
});  
```

##### dispatch_sync
`dispatch_sync()` 同步地提交工作并在返回前等待它完成。使用 `dispatch_sync` 跟踪你的调度障碍工作，或者当你需要等待操作完成后才能使用 Block 处理过的数据。它干的事儿和`dispatch_async`相同，但是它会等待block中的代码执行完成并返回。结合__block类型修饰符，可以用来从执行中的block获取一个值。
```objc
__block NSArray *array;
dispatch_sync(self.concurrentPhotoQueue, ^{  
    array = [NSArray arrayWithArray:_photosArray];
}); 
```
但你需要很小心。想像如果你调用 dispatch_sync 并放在你已运行着的当前队列。这会导致死锁，因为调用会一直等待直到 Block 完成，但 Block 不能完成（它甚至不会开始！），直到当前已经存在的任务完成，而当前任务无法完成！这将迫使你自觉于你正从哪个队列调用——以及你正在传递进入哪个队列。

##### dispatch_group
Dispatch Group 会在整个组的任务都完成时通知你。这些任务可以是同步的，也可以是异步的，即便在不同的队列也行。而且在整个组的任务都完成时，Dispatch Group 可以用同步的或者异步的方式通知你。
```objc
 dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
// 异步下载图片  
dispatch_async(queue, ^{  
    // 创建一个组  
    dispatch_group_t group = dispatch_group_create();  
    __block UIImage *image1 = nil;  
    __block UIImage *image2 = nil;  
      
    // 关联一个任务到group  
    dispatch_group_async(group, dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{  
        // 下载第一张图片  
        image1 = [UIImage imageWithData: [NSData dataWithContentsOfURL:url1]];  
    });  
      
    dispatch_group_async(group, dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{  
        // 下载第二张图片  
        image2 = [UIImage imageWithData: [NSData dataWithContentsOfURL:url2]];
    });  
      
    // 等待组中的任务执行完毕,回到主线程执行block回调  
    dispatch_group_notify(group, dispatch_get_main_queue(), ^{  
        self.imageView1.image = image1;  
        self.imageView2.image = image2;  
    });  
});
```
除此之外，还可以使用`dispatch_group_wait`方法来实现此功能
```objc
dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);  
dispatch_group_t group = dispatch_group_create();  
dispatch_group_async(group, queue, ^{  
    [NSThread sleepForTimeInterval:1];  
    NSLog(@"group1");  
});  
dispatch_group_async(group, queue, ^{  
    [NSThread sleepForTimeInterval:2];  
    NSLog(@"group2");  
});  
dispatch_group_async(group, queue, ^{  
    [NSThread sleepForTimeInterval:3];  
    NSLog(@"group3");  
});  
dispatch_group_wait(group, DISPATCH_TIME_FOREVER);
NSLog(@"group finish");
```
需要注意的是，`dispatch_group_wait`实际上会使当前的线程处于等待的状态，也就是说如果是在主线程执行`dispatch_group_wait`，在上面的Block执行完之前，主线程会处于卡死的状态。可以注意到`dispatch_group_wait`的第二个参数是指定超时的时间，如果指定为`DISPATCH_TIME_FOREVER`则表示会永久等待，直到上面的Block全部执行完，除此之外，还可以指定为具体的等待时间，根据`dispatch_group_wait`的返回值来判断是上面block执行完了还是等待超时了。

##### dispatch_apply
dispatch_apply会将一个指定的block执行指定的次数。`dispatch_apply`表现得就像一个 for 循环，但它能并发地执行不同的迭代。这个函数是同步的，所以和普通的 for 循环一样，它只会在所有工作都完成后才会返回。
```objc
dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
//循环10次
dispatch_apply(10, queue, ^(size_t i) {  
    NSLog(@"%@",i); 
}); 
```

##### dispatch_suspend / dispatch_resume
某些情况下，我们可能会想让*Dispatch Queue*暂时停止一下，然后在某个时刻恢复处理，这时就可以使用`dispatch_suspend`以及`dispatch_resume`函数
```objc
//暂停
dispatch_suspend(globalQueue)
//恢复
dispatch_resume(globalQueue)
```
挂起和继续是异步的,而且只在执行block之间（比如在执行一个新的block之前或之后）生效。挂起一个queue不会导致正在执行的block停止。

##### dispatch_semaphore
信号量让你控制多个消费者对有限数量资源的访问。举例来说，如果你创建了一个有着两个资源的信号量，那同时最多只能有两个线程可以访问临界区。其他想使用资源的线程必须在一个FIFO队列里等待。
信号量在多线程开发中被广泛使用，当一个线程在进入一段关键代码之前，线程必须获取一个信号量，一旦该关键代码段完成了，那么该线程必须释放信号量。其它想进入该关键代码段的线程必须等待前面的线程释放信号量。
信号量的具体做法是：当信号计数大于0时，每条进来的线程使计数减1，直到变为0，变为0后其他的线程将进不来，处于等待状态；执行完任务的线程释放信号，使计数加1，如此循环下去。

```objc
dispatch_semaphore_t semaphore = dispatch_semaphore_create(0);
__block NSMutableData *result = [[NSMutableData alloc] init];
NSURLSession *session = [NSURLSession sharedSession];
NSURLSessionDataTask *dataTask = [session dataTaskWithURL:[NSURL URLWithString:url] completionHandler:^(NSData *data, NSURLResponse *response, NSError *error) {
    if (error == nil) {
        result = [NSMutableData dataWithData:data];
    }else{
        *e = error;
        NSLog(@"NSURLSessionDataTask Error: %@", [error localizedDescription]);
    }
    dispatch_semaphore_signal(semaphore);
}];
[dataTask resume];
dispatch_semaphore_wait(semaphore, DISPATCH_TIME_FOREVER);
```
`dispatch_semaphore_create`创建一个信号，并且将信号量指定为0，这样在`dispatch_semaphore_wait`处就会一直等待信号的发送，直到`dispatch_semaphore_signal`发送出信号后,程序才会进行下一步，这个时候在主程序已经获取到了需要的返回数据了。
