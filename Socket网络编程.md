# Socket

## Socket基础知识

<img src="https://gitee.com/canqchen/cloudimg/raw/master/img/socket%20connection.jpg" alt="socket connection" style="zoom:50%;" />

* 客户端调用connect发送SYN包，客户端协议栈收到 ACK 之后，使得应用程序从 connect 调用返回，表示客户端到服务器端的单
  向连接建立成功
* `accept`发生在什么阶段
  * 三次握手之后，tcp连接会加入到accept队列。accept()会从队列中取一个连接返回，若队列为空，则阻塞
* send/recv(read/write)返回值大于0、等于0、小于0的区别
  * recv
    * 阻塞与非阻塞recv返回值没有区分，都是 <0：出错，=0：连接关闭，>0接收到数据大小
    * 特别：非阻塞模式下返回 值 <0时并且(errno == EINTR || errno == EWOULDBLOCK || errno == EAGAIN)的情况 下认为连接是正常的，继续接收。
    * 只是阻塞模式下recv会阻塞着接收数据，非阻塞模式下如果没有数据会返回，不会阻塞着读，因此需要 循环读取。
  * write：
    * 阻塞与非阻塞write返回值没有区分，都是 <0：出错，=0：连接关闭，>0发送数据大小
    * 特别：非阻塞模式下返回值 <0时并且 (errno == EINTR || errno == EWOULDBLOCK || errno == EAGAIN)的情况下认为连接是正常的， 继续发送。
    * 只是阻塞模式下write会阻塞着发送数据，非阻塞模式下如果暂时无法发送数据会返回，不会阻塞着 write，因此需要循环发送。
  * read：
    * 阻塞与非阻塞read返回值没有区分，都是 <0：出错，=0：连接关闭，>0接收到数据大小
    * 特别：非阻塞模式下返回 值 <0时并且(errno == EINTR || errno == EWOULDBLOCK || errno == EAGAIN)的情况 下认为连接是正常的，继续接收。
    * 只是阻塞模式下read会阻塞着接收数据，非阻塞模式下如果没有数据会返回，不会阻塞着读，因此需要 循环读取。
  * send：
    * 阻塞与非阻塞send返回值没有区分，都是 <0：出错，=0：连接关闭，>0发送数据大小
    * 特别：非阻塞模式下返回值 <0时并且 (errno == EINTR || errno == EWOULDBLOCK || errno == EAGAIN)的情况下认为连接是正常的， 继续发送
    * 只是阻塞模式下send会阻塞着发送数据，非阻塞模式下如果暂时无法发送数据会返回，不会阻塞着 send，因此需要循环发送
* **Socket的四元组、五元组、七元组**
  * 四元组：源IP地址、目的IP地址、源端口、目的端口
  * 五元组：源IP地址、目的IP地址、**协议号**、源端口、目的端口
  * 七元组：源IP地址、目的IP地址、**协议号**、源端口、目的端口，**服务类型以及接口索引**
* 信号SIGPIPE与EPIPE错误码
  * 在linux下写socket的程序的时候，**如果服务器尝试send到一个disconnected socket上，就会让底层抛出一个SIGPIPE信号**。 这个信号的缺省处理方法是退出进程，大多数时候这都不是我们期望的。也就是说，**当服务器繁忙，没有及时处理客户端断开连接的事件，就有可能出现在连接断开之后继续发送数据的情况，如果对方断开而本地继续写入的话，就会造成服务器进程意外退出**
  * 根据信号的默认处理规则SIGPIPE信号的默认执行动作是terminate(终止、退出),所以client会退出。若不想客户端退出可以把 SIGPIPE设为SIG_IGN 如：signal(SIGPIPE, SIG_IGN); 这时SIGPIPE交给了系统处理。 服务器采用了fork的话，要收集垃圾进程，防止僵尸进程的产生，可以这样处理： signal(SIGCHLD,SIG_IGN); 交给系统init去回收。 这里子进程就不会产生僵尸进程了
* **如何检测对端已经关闭socket**
  * 根据ERRNO和recv结果进行判断
    * 在UNIX/LINUX下，**非阻塞模式SOCKET可以采用recv+MSG_PEEK的方式进行判断**，其中MSG_PEEK保证了仅仅进行状态判断，而不影响数据接收
      对于主动关闭的SOCKET, recv返回-1，而且errno被置为9（#define EBADF  9  // Bad file number ）或104 (#define ECONNRESET 104 // Connection reset by peer)
    * 对于被动关闭的SOCKET，recv返回0，而且errno被置为11（#define EWOULDBLOCK EAGAIN // Operation would block ）**对正常的SOCKET, 如果有接收数据，则返回>0, 否则返回-1，而且errno被置为11**（#define EWOULDBLOCK EAGAIN // Operation would block ）因此对于简单的状态判断(不过多考虑异常情况)：recv返回>0，  正常

# IO多路复用

* `select`什么时候返回0？
  * `select`成功时返回就绪fd的总数；如果在超时时间内没有任何fd就绪，返回0；失败返回-1，并设置errno；如果在`select`等待期间，程序接收到信号，则返回-1，并设置errno为EINTR

## select，poll和epoll的区别及应用场景

* **对于select和poll来说，所有文件描述符都是在用户态被加入其文件描述符集合的，每次调用都需要将整个集合拷贝到内核态；epoll则将整个文件描述符集合维护在内核态，每次添加文件描述符的时候都需要执行一个系统调用。系统调用的开销是很大的，而且在有很多短期活跃连接的情况下，epoll可能会慢于select和poll由于这些大量的系统调用开销。**
* **`select`使用线性表描述文件描述符集合，文件描述符有上限；`poll`使用链表来描述；`epoll`底层通过红黑树来描述，并且维护一个ready list，将事件表中已经就绪的事件添加到这里，在使用`epoll_wait`调用时，仅观察这个list中有没有数据即可。**
* **`select`和`poll`的最大开销来自内核判断是否有文件描述符就绪这一过程：每次执行select或poll调用时，它们会采用遍历的方式，遍历整个文件描述符集合去判断各个文件描述符是否有活动；epoll则不需要去以这种方式检查，当有活动产生时，会自动触发epoll回调函数通知epoll文件描述符，然后内核将这些就绪的文件描述符放到之前提到的ready list中等待`epoll_wait`调用后被处理**
* **`select`和`poll`都只能工作在相对低效的LT模式下，而`epoll`同时支持LT和ET模式。**
* **综上，当监测的fd数量较小，且各个fd都很活跃的情况下，建议使用`select`和`poll`；当监听的fd数量较多，且单位时间仅部分fd活跃的情况下，使用epoll会明显提升性能。要求实时性的场景也可以使用`select`，因为`select`的`timeout`参数精度是微妙级，而其他两个是毫秒**
* **需要监控的描述符状态变化多，而且都是非常短暂的，也没有必要使用 `epoll`**。因为 `epoll` 中的所有描述符都存储在内核中，造成每次需要对描述符的状态改变都需要通过 `epoll_ctl`进行系统调用，**频繁系统调用降低效率**。并且 `epoll` 的描述符存储在内核，不容易调试

## LT（电平触发）和ET（边缘触发）的差别

* LT（电平触发）：类似`select`，LT会去遍历在epoll事件表中每个文件描述符，来观察是否有我们感兴趣的事件发生，如果有（触发了该文件描述符上的回调函数），`epoll_wait`就会以非阻塞的方式返回。**若该epoll事件没有被处理完**（**没有返回**`EWOULDBLOCK`），该事件还会被后续的`epoll_wait`再次触发
* ET（边缘触发）：ET在发现有我们感兴趣的事件发生后，立即返回，并且`sleep`这一事件的`epoll_wait`，不管该事件有没有结束
* **在使用ET模式时，必须要保证该文件描述符是非阻塞的**（**确保在没有数据可读时，该文件描述符不会一直阻塞，以避免由于一个文件句柄的阻塞读/阻塞写操作把处理多个文件描述符的任务饿死**）；**并且每次调用`read`和`write`的时候都必须等到它们返回`EWOULDBLOCK`**（**确保所有数据都已读完或写完，因为不再通知**）

## epoll读写事件触发的条件

* LT模式，EPOLLIN触发条件
  * 处于可读状态：1. socket内核接收缓冲区中字节数大于或等于其低水位标记SO_RCVLOWAT；2. 监听socket上有新的连接请求；3. 通信对方关闭连接，读操作返回0；4. socket异常/错误未处理
  * 从不可读状态变为可读状态
* LT模式，EPOLLOUT触发条件
  * 处于可写状态：1. socket内核发送缓冲区中字节数大于等于其低水位标记SO_SNDLOWAT；2. socket写操作被关闭，写操作触发SIGPIPE；3. socket上有未处理错误；4. socket使用非阻塞connect连接成功或失败之后
  * 从不可写状态变为可写状态
* ET模式，EPOLLIN触发条件
  * 从不可读状态变为可读状态
  * 内核接收到新发来的数据
* ET模式，EPOLLOUT触发条件
  * 从不可写状态变为可写状态
  * 只要同时注册了EPOLLIN和EPOLLOUT事件，当对端发数据来的时候，如果此时是可写状态，epoll会同时触发EPOLLIN和EPOLLOUT事件
  * 接受连接后，只要注册了EPOLLOUT事件，那么就会马上触发EPOLLOUT事件

## epoll重要事件类型说明

* **EPOLLPRI：带外数据**

* **EPOLLRDHUP**

  * 发生场景
    * 对端发送 FIN (对端调用close 或者 shutdown(SHUT_WR))
    * 本端调用 shutdown(SHUT_RD)。 当然，关闭 SHUT_RD 的场景很少
  * 可以作为一种**读关闭**的标志，注意**不能读的意思内核不能再往内核缓冲区中增加新的内容。已经在内核缓冲区中的内容，用户态依然能够读取到**

* **EPOLLHUP** 

  * 发生场景

    * 本端调用shutdown(SHUT_RDWR)。 不能是close，**close 之后，文件描述符已经失效**

    * 本端调用 shutdown(SHUT_WR)，对端调用 shutdown(SHUT_WR)

    * 对端发送 RST。发送 RST 的常见场景：

      1)  系统崩溃重启(进程崩溃，只要内核是正常工作都还能兜底，发送的是FIN，不是这里讨论的RST)，四元组消失。此时收到任何数据，都会响应 RST

      2）设置 linger 参数，l_onoff 为 1 开启，但是 l_linger = 0 超时参数为0。此时close() 将直接发送 RST

      3)  接收缓冲区中还有数据，直接 close()， 接收缓冲区中的内容丢弃，直接发送 RST

      4）已经调用 close ，close 会立马发送一个 FIN。注意：仅仅从 FIN 数据包上，无法断定对端是 close 还是仅仅 shutdown(SHUT_WR) 半关闭。**此时如果往对端发送数据，若对端已经 close()，对端会回复 RST**

  * **表示读写都关闭**

# 网络编程

## [I/O模型](https://github.com/CyC2018/CS-Notes/blob/master/notes/Socket.md)

## 时间处理模式

* Reactor的组成

  * Handle(**句柄或描述符**)：本质上表示一种资源，是由操作系统提供的；该资源用于表示一个个的事件，事件既可以来自于外部，也可以来自于内部；外部事件比如说客户端的连接请求，客户端发送过来的数据等；内部事件比如说操作系统产生的定时事件等。**它本质上就是一个文件描述符，Handle是事件产生的发源地**

  * Synchronous Event Demultiplexer(**同步事件分离器**)：它本身是一个**系统调用**，用于等待事件的发生(事件可能是一个，也可能是多个)。调用方在调用它的时候会被阻塞，一直阻塞到同步事件分离器上有事件产生为止。对于Linux来说，同步事件分离器指的就是常用的I/O多路复用机制，比如说select、poll、epoll等

  * Event Handler(**事件处理器**)：本身**由多个回调方法构成**，这些回调方法构成了与应用相关的对于某个事件的反馈机制

  * Concrete Event Handler(**具体事件处理器**)：是事件处理器的实现。它本身实现了事件处理器所提供的各种回调方法，从而实现了特定于业务的逻辑。它本质上就是我们所编写的一个个的处理器**实现**

  * Initiation Dispatcher(**初始分发器**)：实际上就是**Reactor角色**。它本身定义了一些规范，这些规范用于控制事件的调度方式，同时又**提供了应用进行事件处理器的注册、删除等设施**。它本身是整个**事件处理器的核心所在**，Initiation Dispatcher会通过Synchronous Event Demultiplexer来等待事件的发生。一旦事件发生，Initiation Dispatcher首先会**分离出每一个事件，然后调用事件处理器，最后调用相关的回调方法来处理这些事件**

    <img src="https://gitee.com/canqchen/cloudimg/raw/master/img/reactor.jpg" alt="reactor" style="zoom:50%;" />

* Reactor处理流程

  * 初始化Initiation Dispatcher，然后将若干个Concrete Event Handler注册到Initiation Dispatcher中。当应用向Initiation Dispatcher注册Concrete Event Handler时，会在注册的同时指定感兴趣的事件，即，应用会标识出该事件处理器希望Initiation Dispatcher在某些事件发生时向其发出通知，事件通过Handle来标识，而Concrete Event Handler又持有该Handle。这样，事件 -> Handle -> Concrete Event Handler 就关联起来了
  * Initiation Dispatcher 会要求每个事件处理器向其传递内部的Handle。该Handle向操作系统标识了事件处理器
  * 当所有的Concrete Event Handler都注册完毕后，应用会调用handle_events方法来启动Initiation Dispatcher的事件循环。这是，Initiation Dispatcher会将每个注册的Concrete Event Handler的Handle合并起来，并使用Synchronous Event Demultiplexer(同步事件分离器)同步阻塞的等待事件的发生
  * 当与某个事件源对应的Handle变为ready状态时(比如说，TCP socket变为等待读状态时)，Synchronous Event Demultiplexer就会通知Initiation Dispatcher
  * Initiation Dispatcher会触发事件处理器的回调方法，从而响应这个处于ready状态的Handle。当事件发生时，Initiation Dispatcher会将被事件源激活的Handle作为`key`来寻找并分发恰当的事件处理器回调方法
  * Initiation Dispatcher会回调事件处理器的handle_event(type)回调方法来执行特定于应用的功能(开发者自己所编写的功能)，从而相应这个事件。所发生的事件类型可以作为该方法参数并被该方法内部使用来执行额外的特定于服务的分离与分发

## 线程池

* 线程池，开多大，为什么运用线程池？

  * 多线程应用并非线程越多越好。需要根据系统运行的硬件环境以及应用本身的特点决定线程池的大小。一般来说，如果代码结构合理，线程数与cpu数量相适合即可。如果线程运行时可能出现阻塞现象，可相应增加池的大小、如果有必要可采用自适应算法来动态调整线程池的大小，以提高cpu的有效利用率和系统的整体性能
    * 线程池大小简单估算公式，设N为CPU个数
      - 对于CPU密集型的应用，线程池的大小设置为N+1
      - 对于I/O密集型的应用，线程池的大小设置为2N+1
    * 最佳线程数量 = （（线程等待时间 + 线程CPU时间）／ 线程CPU时间 ）* CPU个数
  * CPU密集型任务
    * 尽量使用较小的线程池，一般为CPU核心数+1。 因为CPU密集型任务使得CPU使用率很高，若开过多的线程数，会造成**CPU过度切换**
  * IO密集型任务
    * 可以使用稍大的线程池，一般为2*CPU核心数。 IO密集型任务CPU使用率并不高，因此可以**让CPU在等待IO的时候有其他线程去处理别的任务**，充分利用CPU时间
  * 混合型任务
    * 可以将任务分成IO密集型和CPU密集型任务，然后分别用不同的线程池去处理。 只要分完之后两个任务的执行时间相差不大，那么就会比串行执行来的高效
    * 因为如果划分之后两个任务执行时间有数据级的差距，那么拆分没有意义
    * 因为先执行完的任务就要等后执行完的任务，最终的时间仍然取决于后执行完的任务，而且还要加上任务拆分与合并的开销，得不偿失
  * 使用线程池目的：**提高服务器性能，以空间换时间。减少在创建和销毁线程上所花的时间以及申请系统资源的开销，分配系统资源的系统调用是很耗时的。程序运行时预先创建一个线程的集合，当用户请求到来时，可以直接从线程池取得一个执行实体，而无需动态调用pthread_create等函数来创建线程。执行实体用完后可以放回线程池，以供循环使用。**

* **C++11简单线程池实现代码**

  ```c++
  #include <queue>
  #include <thread>
  #include <mutex>
  #include <condition_variable>
  #include <functional>
  #include <memory>
  #include <cstddef>
  #inclued <stdexcept>
  
  class ThreadPool{
  public:
      ThreadPool(size_t threadNum = 8): mPool(std::make_share<Pool>()){
          assert(threadNum>0);
          try{
              for(size_t i = 0; i < threadNum; i++){
                  std::thread(
                      [pool=mPool]{
                          std::unique_lock<std::mutex> locker(pool->mtx);
                          while(!pool->close){	// 获得锁才能往下执行
                              if(!pool->tasks.empty()){
                                  auto task = pool->tasks.top(); 	// 从任务队列取任务执行
                                  pool->tasks.pop();		
                                  locker.unlock();	// 取完解锁，让池接收新任务，或者让其他线程处理其他任务
                                  task();			// 执行任务
                                  locker.lock();   // 为下次循环加锁
                              }
                              else
                                  pool->cond.wait(locker);  // 无任务，等待，通过竞争锁被唤醒
                          }
                      }
                  ).detach();   
          	}
          }
          catch(...){
                  {
                      std::lock_guard<std::mutex> locker(mPool->mtx);  
                      mPool->close = true;         // closed = true，关闭所有线程
                  }
                  mPool->cond.notify_all();    // 唤醒所有线程，执行关闭
                  throw std::runtime_error("ThreadPool Init Failed!");
          }
      }
      
      // 往线程池任务队列添加任务
      template<typename Func>
      void AddTask(Func && task){
          {
              std::lock_guard<std::mutex> lg(mPool->mtx);
              mPool->tasks.emplace(std::forward<Func>(task));  // forward完美转发右值引用
          }
          mPool->cond.notify_one();
      }
      
      ~ThreadPool(){
          if(mPool != nullptr){
              {
                  std::lock_guard<std::mutex> lg(mPool->mtx);
          		mPool->close = true;
              }
              mPool->cond.notify_all();
          }
          
      }
      
      ThreadPool(const ThreadPool&) = delete;
      ThreadPool& operator = (const ThreadPool&) = delete;
  private:
    struct Pool{
  		std::queue<std::function(void())> tasks;  // 任务队列
          std::mutex mtx;
      	std::condition_variable cond;
          bool close = false;
      };
      std::share_ptr<Pool> mPool;
  };
  ```

* 线程安全的概念

  * 简单理解，确保在多线程访问的时候，我们的程序还能按照我们预期的行为去执行，那么就是线程安全

* 并发编程中的四性

  * 原子性：即一个操作或者多个操作，要么全部执行并且执行的过程不会被任何因素打断，要么就都不执行。原子性就像数据库里面的事务一样，他们是一个团队，同生共死
  * 可见性：指当多个线程访问同一个变量时，一个线程修改了这个变量的值，其他线程能够立即看得到修改的值。**volatile可以保证可见性**，当一个变量被volatile修饰后，表示着线程本地内存无效，当一个线程修改共享变量后他会立即被更新到主内存中，当其他线程读取共享变量时，它会直接从主内存中读取。**volatile也可以保证线程可见性且提供了一定的有序性，但是无法保证原子性**
  * 有序性：即程序执行的顺序**按照代码的先后顺序执行**。为了效率，C++或Java允许编译器和处理器对指令进行重排序，当然重排序它不会影响单线程的运行结果，但是对多线程会有影响。volatile可以保证一定的有序性

* 非阻塞同步（乐观锁）

  * 随着硬件指令集的发展，出现了基于冲突检测的乐观并发策略，通俗地说，**就是先进行操作，如果没有其他线程争用共享数据，那操作就成功了**；**如果共享数据有争用，产生了冲突，那就再采用其他的补偿措施。（最常见的补偿错误就是不断地重试，直到成功为止）**，这种乐观的并发策略的许多实现都不需要把线程挂起，因此这种同步操作称为**非阻塞同步**
  * **非阻塞的实现CAS（Compare-and-Swap）**
    * CAS指令需要有3个操作数，分别是内存地址（在java中理解为变量的内存地址，用V表示）、旧的预期值（用A表示）和新值（用B表示）。CAS指令执行时，当且仅当V处的值符合旧预期值A时，处理器用B更新V处的值，否则它就不执行更新，但是无论是否更新了V处的值，都会返回V的旧值，上述的处理过程是一个原子操作
  * **CAS的ABA问题**
    * 因为CAS需要在操作值的时候检查下值有没有发生变化，如果没有发生变化则更新，但是一个值原来是A，变成了B，又变成了A，那么使用CAS进行检查时会发现它的值没有发生变化，但是实际上却变化了。CAS只关注了比较前后的值是否改变，而无法清楚在此过程中变量的变更明细，这就是所谓的ABA漏洞。
  * ABA问题的解决思路就是使用版本号(如MySQL的MVCC)。在变量前面追加版本号，每次变量更新的时候把版本号加一，那么A-B-A就变成了1A-2B-3C

# 系统性能优化

## 性能优化现象和判断

* 提出性能优化现象
  * 前台访问很慢，请帮忙分析优化
  * 用户对性能很不满意，再不解决就要投诉
  * 数据库负载很重，请帮忙分析一下
  * xxx功能打开需要1分钟，请帮忙分析一下
* 在接到这些性能优化的时候，运维工程师希望能够了解下面的信息以判断问题的类型：
  * 系统性问题：比如CPU利用率，SWAP利用率，或者IO过高导致的整体性能下降
  * 功能性问题：整体性能良好，个别功能时延很长
  * 新出现问题：什么时候开始的？之前系统有哪些变动？（升级或者管理的资源大量增加）
  * 不规律问题：有时候快，有时候慢，没有特定的规律
  * 性能快慢的衡量标准是什么？原来多少秒，现在多少秒，目标是多少秒？

## 性能分析目的

* Linux性能分析的目的
  * 找出系统性能瓶颈（包括硬件瓶颈和软件瓶颈）
  * 提供性能优化方案（升级硬件？改进系统结构？）
  * 达到合理的硬件和软件配置
  * 使系统资源使用达到最大的平衡
* 一般情况下系统良好运行的时候恰恰各项资源达到了一个平衡状态，任何一项资源的过度使用都会造成平衡体系破坏，从而造成系统负载极高或者响应迟缓
* 比如cpu过度使用会造成大量进程等待cpu资源，系统响应变慢，等待会造成进程数增加，进程增加又会造成内存使用增加，内存耗尽又会造成虚拟内存使用，使用虚拟内存又会造成磁盘IO增加和cpu开销增加

## 性能分析步骤

### 需要系统监控工具和性能分析工具

* 对资源使用状况进行长期的监控和数据采集（nagios、cacti、ganglia、zabbix）
* 使用常见的性能分析工具（vmstat、top、htop、iotop、free、iostat、ifstat等）
* 实战技能和经验积累

### 出现性能问题的可能原因

* 应用程序设计的缺陷和数据库查询的滥用最可能导致性能问题
* 性能瓶颈可能使因为程序差、内存不足、磁盘瓶颈，但最终表现出的结构就是cpu耗尽，系统负载极高，响应迟缓，甚至暂时失去响应
* 物理内存不够时会使用交换内存，使用swap会带来磁盘io和cpu的开销
* 可能造成cpu瓶颈的问题：频繁执行perl，php，java程序生成动态web；数据库查询大量where子句、order by、group by排序等
* 可能造成内存瓶颈的问题：高并发用户访问、系统进程多、java内存泄漏等
* 可能造成磁盘io瓶颈的问题：生成cache文件，数据库频繁更新，或者查询大表等

## 影响linux性能的因素

* cpu
  * cpu使操作系统稳定运行的根本，cpu的速度与性能在很大程度上决定了系统整体的性能，因此，cpu数量越多、主频越高，服务器性能就相对越好，但事实并非如此
  * 目前大vu分cpu在同一时间内只能运行一个进程，超线程的处理器可以在同一时间运行多个线程，因此可以利用处理器的超线程特性提高系统性能。另外，linux内核会把多核的处理器当作多个单独的cpu来识别，例如两个4核的cpu，在linux系统中会被当做8个单核cpu。但是从cpu性能角度来说，两个4核的cpu喝8个单核的cpu并不完全等价，根据权威部门得出的测试结论，前者的整体性能要比后者低25%~30%
  * 可能出现cpu性能瓶颈的应用有邮件服务器、动态web服务器等，对于这列应用，要把cpu的配置和性能放在主要位置
* 内存
  * 内存的大小也是影响linux性能的一个重要因素，内存太小，系统进程将被阻塞，应用也会变得缓慢，甚至失去响应；内存太大，导致资源浪费。linux系统采用了物理内存和虚拟内存两种方式，虚拟内存虽然可以缓解物理内存的不足，但是占用过多的虚拟内存，应用程序的性能将明显下降，要保证应用程序的高性能运行，物理内存一定要够大
  * 可能出现内存性能瓶颈的应用有redis内存数据库服务器、cache服务器、静态web服务器等，对于这类应用要把内存大小放在主要位置
* 磁盘IO性能
  * 磁盘的io性能直接影响应用程序的性能，在一个有频繁读写的应用中，如果磁盘io性能得不到满足，就会导致应用停滞。好在如今的磁盘都采用了很多方法来提高io性能，比如常见的磁盘RAID技术
  * 根据磁盘组合方式的不同，RAID可以分为RAID0、RAID1、RAID2、RAID3、RAID4、RAID5、RAID6、RAID7、RAID0+1、RAID10等级别，常用的RAID级别有RAID0、RAID1、RAID5、RAID0+1
  * RAID0：这种方式成本低，没有容错和数据修复功能，因而只能用在对数据安全性要求不高的环境中
  * RAID1：也就是磁盘镜像，通过把一个磁盘的数据镜像到另一个磁盘上，最大限度地保证磁盘数据的可靠性和可修复性，具有很高的数据冗余能力，但磁盘利用率只有50%，因而，成本最高，多用在保存重要数据的场合
  * RAID5：采用了磁盘分段加奇偶校验技术，从而提高了系统可靠性，RAID5读出效率很高，写入效率一般，至少需要3块盘。允许一块磁盘  故障，而不影响数据的可用性
  * RAID0+1：把RAID0和RAID1技术结合起来就成了RAID0+1，至少需要4个硬盘。此种方式的数据除了分布在多个盘上外，每个磁盘都有其镜像盘，提供全冗余能力，同时允许一个磁盘故障，而不影响数据可用性，并具有快速读、写能力
  * 通过了解各个RAID级别的性能，可以根据应用的不同特性，选择适合自身的RAID级别，从而保证应用程序在磁盘方面达到最优性能
  * 目前常用的磁盘类型有STAT、SAS、SSD磁盘，STAT、SAS使普通磁盘，读写效率一般，如果要保证高性能的写操作，可以采用SSD固态磁盘，读写速度可达600MB/s
* 网络带宽
  * 网络带宽也是影响性能的一个重要因素，低俗的、不稳定的网络将导致网络应用程序的访问阻塞，而稳定、高速的网络带宽，可以保证应用程序在网络上畅通无阻地运行。幸运地是，现在网络一般都是千兆带宽或光纤网络，带宽问题对应用程序性能造成的影响也在逐步降低
  * 组件网络时，如果局域网内有大量数据传输需求（hadoop大数据业务、数据库业务），可采用千兆、万兆网络接口，针对每个服务器，如果单网卡效率不够，可采用双网卡绑定技术，提高网卡数据传输带宽和性能

## 操作系统相关资源

### 系统安装优化

![sys_install_refine](https://gitee.com/canqchen/cloudimg/raw/master/img/sys_install_refine.jpg)

### 内核参数优化

![kernel_params](https://gitee.com/canqchen/cloudimg/raw/master/img/kernel_params.jpg)

### 文件系统优化

![file_sys_refine](https://gitee.com/canqchen/cloudimg/raw/master/img/file_sys_refine.jpg)

## 系统性能分析工具

* vmstat

  ![vmstat](https://gitee.com/canqchen/cloudimg/raw/master/img/vmstat.jpg)

* iostat

  ![iostat](https://gitee.com/canqchen/cloudimg/raw/master/img/iostat.jpg)

* sar

  ![sar](https://gitee.com/canqchen/cloudimg/raw/master/img/sar.jpg)

## 系统性能分析标准

![standard](https://gitee.com/canqchen/cloudimg/raw/master/img/standard.jpg)

