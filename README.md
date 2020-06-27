# 面试相关准备

1. C++四个cast的用法

    C++的强制转换方式比C语言更加丰富，常见的有四个：

    1. const_cast

    2. static_cast

    3. dynamic_cast

    4. reinterpret_cast

    这四个的使用方式都一样：T t = XXX_cast\<T>(expressions)。

    1，const_cast这个操作符可以去掉变量const属性或者volatile属性的转换符，这样就可以更改const变量了。比如下面代码

    ~~~ c++
    string str = "hello";
    char *_const = str.substr(0,3).c_str();//c_str()返回const char*类型，直接赋值给char *显然出错，这句话编译不能通过
    char *non_const = const_cast<char *> (str.substr(0,3).c_str());//将const属性移除，可以通过编译了
    ~~~

    2，static_cast 这个操作符相当于C语言中的强制类型转换的替代品。多用于非多态类型的转换，比如说将int转化为double。但是不可以将两个无关的类型互相转化。（在编译时期进行转换）

    3，dynamic_cast操作符 可以安全的将父类转化为子类，子类转化为父类都是安全的。所以你可以用于安全的将基类转化为继承类，而且可以知道是否成功，如果强制转换的是指针类型，失败会返回NULL指针，如果强制转化的是引用类型，失败会抛出异常。dynamic_cast 转换符只能用于含有虚函数的类。用一个简单的代码例子可以看出

    ~~~ c++
    #include <iostream>
    using namespace std;
    class Animal {
    public:
        virtual void foo()
        {
        cout << "Animal::foo()" << endl;
        }
    };
    class dog : public Animal {
    public:
        void foo()
        {
        cout << "dog::foo()" << endl;
        }
    };
    class wolf : public Animal {
        public:
        void foo()
        {
        cout << "wolf::foo()" << endl;
        }
    };
    class nonrelated {
    public:
        virtual void foo()
        {
        cout << "nonrelated::foo()" << endl;
        }
    };
    int main()
    {
        Animal *ani;
        dog mydog;
        ani = &mydog;
        wolf *pb_dynamic = dynamic_cast<wolf *>(ani);
        wolf *pb_static = static_cast<wolf *>(ani); //可以转换成功
        cout << "pb_dynamic " << pb_dynamic << endl;//因为ani绑定到dog，所以pbno_dynamic转换失败，应该输出0
        cout << "pb_static " << pb_static << endl; //这个转换成功
        pb_static->foo(); //输出dog::foo()
        nonrelated *no = new nonrelated();
        wolf *pbno_dynamic = dynamic_cast<wolf *>(no);
        //wolf *pbno_static = static_cast<wolf *>(no);//static_cast 不能用于转换不相关的类
        cout << "pbno_dynamic " << pbno_dynamic << endl; //因为ani绑定到dog，所以pbno_dynamic转换失败，应该输出0
        delete no;
        return 0;
    }
    ~~~

    4， reinterpret_cast：重新解释（无理）转换。即要求编译器将两种无关联的类型作转换。

2. c++ 锁

    shared_mutex&unique_lock\
    shared_mutex 用来保护被多个线程访问的共享数据\
    如果一个线程已经获取了互斥锁，则其他线程都无法获取该锁。\
    如果一个线程已经获取了共享锁，则其他任何线程都无法获取互斥锁，但是可以获取共享锁。\
    在一个线程中，同时只能获得一个锁(共享或排他)\
    由于c++中的cout不是线程安全的函数，所以给cout输出加上了互斥锁。

    1. 线程之间的锁有：互斥锁、条件锁、自旋锁、读写锁、递归锁(std::recursive_mutex一般不用)。

        互斥锁是是一种sleep-waiting的锁\
        自旋锁是一种busy-waiting的锁

        互斥锁：\
            1. 定义互斥量 pthread_mutex_t mutex;\
            2. 初始化互斥量 pthread_mutex_init(&mutex, NULL)\
            3. 创建线程 pthread_create(&tid, NULL, task, (void *)"zhangfei");\
            4. join线程 pthread_join(tid, NULL);\
            5. 销毁互斥量

~~~ c++
            //函数内部需要先对互斥量加锁，最后解锁
            void *task(void *p)
            {
                //使用互斥量进行加锁
                pthread_mutex_lock(&mutex);
                buf[pos] = (char *)p;
                sleep(1);
                pos++;
                //用互斥量进行解锁
                pthread_mutex_unlock(&mutex);
            }
~~~

        条件锁：
            1. 条件锁一般与互斥锁一起用
                    出租车问题\
            2. 虚假唤醒\
                用while判断条件而不是if

        信号量
            pv操作

3. stl
    STL包含6大部件：容器、迭代器、算法、仿函数、适配器和空间配置器。
    [面试相关](https://blog.csdn.net/daaikuaichuan/article/details/80717222)

    1. ?为何map和set的插入删除效率比其他序列容器高，而且每次insert之后，以前保存的iterator不会失效？\
        因为存储的是结点，不需要内存拷贝和内存移动。

        因为插入操作只是结点指针换来换去，结点内存没有改变。而iterator就像指向结点的指针，内存  没变，指向内存的指针也不会变。

    2. 迭代器底层实现包含两个重要的部分：萃取技术和模板偏特化。

    3. 迭代器失效的问题？

        3.1 插入操作
            对于vector和string，如果容器内存被重新分配，iterators,pointers,references失效；如果没有重新分配，那么插入点之前的iterator有效，插入点之后的iterator失效；

            对于deque，如果插入点位于除front和back的其它位置，iterators,pointers,references失效；当我们插入元素到front和back时，deque的迭代器失效，但reference和pointers有效；

            对于list和forward_list，所有的iterator,pointer和refercnce有效。

        3.2 删除操作
            对于vector和string，删除点之前的iterators,pointers,references有效；off-the-end迭代器总是失效的；

            对于deque，如果删除点位于除front和back的其它位置，iterators,pointers,references失效；当我们插入元素到front和back时，off-the-end失效，其他的iterators,pointers,references有效；

            对于list和forward_list，所有的iterator,pointer和refercnce有效。

            对于关联容器map来说，如果某一个元素已经被删除，那么其对应的迭代器就失效了，不应该再被使用，否则会导致程序无定义的行为。

4. const和#define的区别

    const定义的只读变量在程序运行过程中只有一份拷贝(因为它是全局的只读变量，存放在静态区)，而#define定义的宏常量在内存中有若干个拷贝。

    #define宏是在预编译阶段进行替换，而const修饰的只读变量是在编译的时候确定其值。#define宏没有类型，而const修饰的只读变量具有特定的类型。

5. 函数重载指的是可以有多个同名的函数，因此对名称进行了重载。

    C++的多态性用一句话概括就是：在基类的函数前加上virtual关键字，在派生类中重写该函数

    a.编译时多态性：通过重载函数实现　　b 运行时多态性：通过虚函数实现
    
    覆写(override)  override 确保该函数为虚函数并覆写来自基类的虚函数

6. new和malloc

    new是操作符，malloc是函数

    new的对象会初始化，malloc不初始化

    new调用构造，malloc不调用

    new需要异常捕获，malloc判空

    如果内存申请不出来，那就exit，不要return

7. null和nullptr

    c里null = (void*)0   c++里 null = 0 但是，再函数重载的时候，null = 0可能出现问题

8. 野指针
    指针越界。指针未初始化。指针被delete之后，没有指向NULL

9. typedef和#define

    typedef 不只是简单的名称替换，还包含了一定的封装性
    ~~~c++
    typedef   char*   PSTR;
    int   mystrcmp(const   PSTR,   const   PSTR);
    const   PSTR实际上相当于const   char*吗？不是的，它实际上相当于char*   const。
    ~~~

10. linux下有用的命令，memseak, readelf(查看core dump),top,ps,kill,sed

11. 记住几个常用的gdb命令：

    l(list) ，显示源代码，并且可以看到对应的行号；
    b(break)x, x是行号，表示在对应的行号位置设置断点；
    p(print)x, x是变量名，表示打印变量x的值；
    r(run), 表示继续执行到断点的位置；
    n(next),表示执行下一步；
    c(continue),表示继续执行；
    q(quit)，表示退出gdb。

12. 内存泄漏检查

    windows下的_CrtDumpMemoryLeaks();\
    linux 的valgrind

13. malloc实现原理，小于128K的时候，调用brk，否则调用mmap，所以大空间可以单独释放，小空间必须按照顺序释放

14. static成员函数和成员变量
    * -静态成员变量属于整个类所有\
    * -静态成员变量的生命期不依赖于任何对象，为程序的生命周期\
    * -可以通过类名直接访问公有静态成员变量\
    * -所有对象共享类的静态成员变量\
    * -可以通过对象名访问公有静态成员变量\
    * <b>-静态成员变量需要在类外单独分配空间</b>\
    * -静态成员变量在程序内部位于全局数据区 (Type className::VarName = value)\
    * <b>静态成员函数属于整个类所有，没有this指针</b>
    * <b>静态成员函数只能直接访问静态成员变量和静态成员函数</b>

15. 多线程同步和互斥的实现方式:\
    同步有内核模式和用户模式\
    用户模式下有原子操作和临界区\
    内核模式下有:时间，信号量，互斥量

16. 虚函数表类似于一个static类内变量

17. 线程阻塞的常见情况
    1. 调用sleep()进入睡眠状态；

    2. 用wait()暂停了线程，除非收到notify()唤醒线程；

    3. 线程正在等待一些IO操作；

    4. 线程正在试图调用被锁起来了的对象。

18. 进程间通讯方式

    1. 管道(pipe ，fd[0,1])
    2. FIFO命名管道
    3. 消息队列
    4. 信号量
    5. 共享内存，速度最快
    6. socket

19. 工厂模式   单例模式   观察者模式   策略模式  <b>reactor模式</b>

20. (网络)[https://blog.csdn.net/daaikuaichuan/article/details/83475809]

    慢启动和拥塞避免
    快重传和快恢复

    https

21. <b>线程间通讯方式 </b>
    1. volatile
    2. wait notify()

22. 死锁的条件
    互斥  持有   不可抢占   循环等待

23. GET和POST的区别\
    GET提交的数据会放在URL之后，以?分割URL和传输数据，参数之间以&相连，如EditPosts.aspx?name=test1&id=123456；POST方法是把提交的数据放在HTTP包的Body中。

    GET一般传输数据大小不超过2k-4k（根据浏览器不同，限制不一样，但相差不大）；POST请求传输数据的大小根据php.ini 配置文件设定，也可以无限大。

    GET请求是可以缓存的；POST请求不可以缓存。

    GET请求页面后退时，不产生影响；POST请求页面后退时，会重新提交请求。

    get，参数url可见；post，url参数不可见。



27. mysql调优

    慢日志查询  缓存   当只需要一条数据时使用LIMIT 1    避免全表查询   为每张表设置一个id作为其主键   尽可能的使用not null   为搜索的字段建立索引

28. 聚集索引一个表只能有一个，而非聚集索引一个表可以存在多个。

30. const和static不能共同使用

C++编译器在实现const的成员函数的时候为了确保该函数不能修改类的实例的状态，会在函数中添加一个隐式的参数const this*。但当一个成员为static的时候，该函数是没有this指针的。也就是说此时const的用法和static是冲突的。我们也可以这样理解：两者的语意是矛盾的。static的作用是表示该函数只作用在类型的静态变量上，与类的实例没有关系；而const的作用是确保函数不能修改类的实例的状态，与类型的静态变量没有关系。因此不能同时用它们。

31. http 无连接 无状态 HTTP是媒体独立的

32. 守护进程创建
1)创建子进程，终止父进程
(2)在子进程中创建新会话
(3)改变工作目录
(4)重设文件创建掩码
(5)关闭文件描述符

33. 聚集索引 逻辑顺序和物理顺序相同 索引的叶子节点就是对应的数据节点 可以直接获取到对应的全部列的数据\
    非聚集索引分成普通索引，唯一索引，全文索引 非聚集索引的二次查询问题

    如何解决非聚集索引的二次查询问题\
    复合索引（覆盖索引）

34. MyISAM更适合读密集的表用标锁 聚簇的，而InnoDB更适合写密集的的表 InnoDB支持事务和行锁 非聚簇的。Postgre 多进程的，myssql多线程，PG有支持json

myisam是默认表类型不是事物安全的；innodb支持事物。

myisam不支持外键；Innodb支持外键。

myisam支持表级锁（不支持高并发，以读为主）；innodb支持行锁（共享锁，排它锁，意向锁），粒度更小，但是在执行不能确定扫描范围的sql语句时，innodb同样会锁全表。

执行大量select，myisam是最好的选择；执行大量的update和insert最好用innodb。

myisam在磁盘上存储上有三个文件.frm(存储表定义)  .myd（存储表数据）  .myi（存储表索引）；innodb磁盘上存储的是表空间数据文件和日志文件，innodb表大小只受限于操作系统大小。

myisam使用非聚集索引，索引和数据分开，只缓存索引；innodb使用聚集索引，索引和数据存在一个文件。

35. ACID 原子性(undo log))，一致，持久(redo log  log buffer 分为os buffer 直接刷进磁盘还是等待os buffer 一分钟刷一次) 持久性是指一个事务一旦被提交，它对数据库中数据的改变就是永久性的，接下来即使数据库发生故障也不应该对其有任何影响   隔离(锁)

    隔离级别: 脏读   不可重复读    虚读

36. 协程： 比线程更小的执行单元，自带cpu上下文，一个协程一个栈

37. 乐观锁
    每次获取数据的时候，都不会担心数据被修改，所以每次获取数据的时候都不会进行加锁（因为不担心数据被修改），但是在更新数据的时候需要判断该数据是否被别人修改过。如果数据被其他线程修改，则不进行数据更新，如果数据没有被其他线程修改，则进行数据更新。

    悲观锁
    每次获取数据的时候，都会担心数据被修改，所以每次获取数据的时候都会进行加锁，确保在自己使用的过程中数据不会被别人修改，使用完成后进行数据解锁。由于数据进行加锁，期间对该数据进行读写的其他线程都会进行等待。

    共享锁
    共享锁保证大家可以一起读，但只能一个人写

    行锁

    表锁

    、数据库隔离级别的实现
    read uncommitted（读未提交的数据）可以读到没有提交或者回滚的内容 （脏数据）。写数据时加上排他锁，直到事务结束， 读的时候不加锁。\
    read committed（读已提交的数据）事务1能读到其他已提交事务的修改，出现“不可重复读”的问题。写数据的时候加上排他锁， 直到事务结束，读的时候加上共享锁，读完数据立刻释放。\
    repeatable read（重复读-MySQL默认隔离级别）
      现象：能避免“丢失数据”和“脏数据”， “不可重复读”三个问题，事务1能读取到其他事务新插入读数据，出现“幻读”的问题。写数据的时候加上排他锁，直到事务结束， 读数据的时候加共享锁， 事务提交之后立刻释放该共享锁。\
    serializable（串行化）能避免“丢失数据”和“脏数据”， “不可重复读”，“幻读”四个问题。事务读数据则加表级共享锁，事务写数据则加表级排他锁。\

38. 分布式CAP定理
    一致 分区容错 可用

39. MongoDB文档存储一般用类似json的格式存储，存储的内容是文档型的。这样也就有机会对某些字段建立索引，实现关系数据库的某些功能。

40. neo4j 图数据库 遵循acid，节点，关系，属性

41. 访问者模式
    基本等于遍历

42. 单例模式(线程池，设备管理器)
    将构造函数私有化 \ 
    禁止赋值和拷贝\
    线程安全\
    用户通过接口获取实例：使用 static 类成员函数\
    全局只有一个实例：static 特性，同时禁止用户自己声明并定义实例（把构造函数设为 private）\

    懒汉模式(静态函数构造静态变量)   内存泄漏   线程安全的问题
    饿汉模式

~~~c++
class Singleton{
public:
    typedef std::shared_ptr<Singleton> Ptr;
    ~Singleton(){
        std::cout<<"destructor called!"<<std::endl;
    }
    Singleton(Singleton&)=delete;
    Singleton& operator=(const Singleton&)=delete;
    static Ptr get_instance(){

        // "double checked lock"
        if(m_instance_ptr==nullptr){
            std::lock_guard<std::mutex> lk(m_mutex);
            if(m_instance_ptr == nullptr){
              m_instance_ptr = std::shared_ptr<Singleton>(new Singleton);
            }
        }
        return m_instance_ptr;
    }


private:
    Singleton(){
        std::cout<<"constructor called!"<<std::endl;
    }
    static Ptr m_instance_ptr;
    static std::mutex m_mutex;
};

~~~

43. lock_guard && unique_lock 
    lock_guard 比较方便，自动解锁
    unique比较自由

44. 内存分配malloc流程\
        1. 获取分配区的锁，防止多线程冲突。
        2. 计算出实际需要分配的内存的chunk实际大小。
        3. 判断chunk的大小，如果小于max_fast（64Ｂ），则尝试去fast bins上取适合的chunk，如果有则分配结束。否则，下一步；
        4. 判断chunk大小是否小于512B，如果是，则从small bins上去查找chunk，如果有合适的，则分配结束。否则下一步；
        5. ptmalloc首先会遍历fast bins中的chunk，将相邻的chunk进行合并，并链接到unsorted bin中然后遍历 unsorted bins。如果unsorted bins上只有一个chunk并且大于待分配的chunk，则进行切割，并且剩余的chunk继续扔回unsorted bins；如果unsorted bins上有大小和待分配chunk相等的，则返回，并从unsorted bins删除；如果unsorted bins中的某一chunk大小 属于small bins的范围，则放入small bins的头部；如果unsorted bins中的某一chunk大小 属于large bins的范围，则找到合适的位置放入。若未分配成功，转入下一步；
        6. 从large bins中查找找到合适的chunk之后，然后进行切割，一部分分配给用户，剩下的放入unsorted bin中。
        7. 如果搜索fast bins和bins都没有找到合适的chunk，那么就需要操作top chunk来进行分配了. 当top chunk大小比用户所请求大小还大的时候，top chunk会分为两个部分：User chunk（用户请求大小）和Remainder chunk（剩余大小）。其中Remainder chunk成为新的top chunk。当top chunk大小小于用户所请求的大小时，top chunk就通过sbrk（main arena）或mmap（thread arena）系统调用来扩容。
        8. 到了这一步，说明 top chunk 也不能满足分配要求，所以，于是就有了两个选择: 如 果是主分配区，调用 sbrk()，增加 top chunk 大小；如果是非主分配区，调用 mmap 来分配一个新的 sub-heap，增加 top chunk 大小；或者使用 mmap()来直接分配。在 这里，需要依靠 chunk 的大小来决定到底使用哪种方法。判断所需分配的 chunk 大小是否大于等于 mmap 分配阈值，如果是的话，则转下一步，调用 mmap 分配， 否则跳到第 10 步，增加 top chunk 的大小。
        9. 使用 mmap 系统调用为程序的内存空间映射一块 chunk_size align 4kB 大小的空间。 然后将内存指针返回给用户。
        判断是否为第一次调用 malloc，若是主分配区，则需要进行一次初始化工作，分配 一块大小为(chunk_size + 128KB) align 4KB 大小的空间作为初始的 heap。若已经初 始化过了，主分配区则调用 sbrk()增加 heap 空间，分主分配区则在 top chunk 中切 割出一个 chunk，使之满足分配需求，并将内存指针返回给用户。
        简而言之： 获取分配区(arena)并加锁–> fast bin –> unsorted bin –> small bin –> large bin –> top chunk –> 扩展堆

45. unordered_map
    hashmap+桶 冲突的时候，如果小于一个数就链表，大于就map
    扩容用rsize，lower_bound找第一个大于这个数的素数。

46. 拥塞控制
    当出现超时重传和冗余ack的时候慢启动门限都要设置为当前发送窗口的一半
    不同的就是超时重传还得将拥塞窗口大小设为1，重新进入慢启动，而冗余ack则是将拥塞窗口设为慢启动门限大小并且进入拥塞避免

47. 覆盖索引的几个场景
    1. 无WHERE条件的查询优化：  没有WHERE条件时查询优化器无法通过索引检索数据
    2. 二次检索优化 
    3. 分页查询优化 (例如select tid,return_date from t1 order by inventory_id limit 50000,10;)
    4. 建了索引但是查询不走索引

48. 指令
    vmstat  top  netstat

49. 数据库调优
    slow log 分析
    紧急的: show processlist   show status like '%lock%'; # 查询锁状态
~~~
分布式数据库的数据一致性怎么保证
数据库死锁的场景，怎么解决，操作系统的死锁，怎么解决
数据库的ACID四种属性的底层实现原理
~~~



项目：
    客户端：
        线程一，二：
        使用的海康的rtsp协议，创建一个线程，通过ffmpeg地址读取摄像头，设置缓冲，udp，最大延迟，解码器解码视频流，使用av_read_frame阻塞读取视频帧，解码视频帧，像素转化，qimage读取。通过signal，前端页面播放
        线程三：
        接受来自伺服器的socket信号，截取视频，车牌识别，将出入与时间上传到远端服务器。
        线程四：
        接受来自远端服务器的价格信息或者刚来的信息。

    远端服务器：
        写了一个线程池，保持连接，用来处理客户端请求。为了防止粘包，还写了一个i。

    伺服器:
        用来接收传感器的zigee信号

    问题:线程池当时vector 用了clear，始终不为空

qt的sign和slots：
    为什么，回调函数不好吗？不好，第一，它们并不是类型安全，我们永远都不能确定调用者是否将通过正确的参数来调用“回调函数”；第二，回调函数与处理函数是紧耦合（strongly coupled）的，因为调用者必须知道应该在什么时候调用哪个回调函数。

一.PostgreSQL相对于MySQL的优势
PG是堆表，mysql是索引表
1、在SQL的标准实现上要比MySQL完善，而且功能实现比较严谨；\
2、存储过程的功能支持要比MySQL好，具备本地缓存执行计划的能力；\
3、对表连接支持较完整，优化器的功能较完整，支持的索引类型很多，复杂查询能力较强；\
4、PG主表采用堆表存放，MySQL采用索引组织表，能够支持比MySQL更大的数据量。\
5、PG的主备复制属于物理复制，相对于MySQL基于binlog的逻辑复制，数据的一致性更加可靠，复制性能更高，对主机性能的影响也更小。\
6、MySQL的存储引擎插件化机制，存在锁机制复杂影响并发的问题，而PG不存在。

二、MySQL相对于PG的优势：
1、innodb的基于回滚段实现的MVCC机制，相对PG新老数据一起存放的基于XID的MVCC机制，是占优的。新老数据一起存放，需要定时触 发VACUUM，会带来多余的IO和数据库对象加锁开销，引起数据库整体的并发能力下降。而且VACUUM清理不及时，还可能会引发数据膨胀；\
2、MySQL采用索引组织表，这种存储方式非常适合基于主键匹配的查询、删改操作，但是对表结构设计存在约束；\
3、MySQL的优化器较简单，系统表、运算符、数据类型的实现都很精简，非常适合简单的查询操作；\
4、MySQL分区表的实现要优于PG的基于继承表的分区实现，主要体现在分区个数达到上千上万后的处理性能差异较大。\
5、MySQL的存储引擎插件化机制，使得它的应用场景更加广泛，比如除了innodb适合事务处理场景外，myisam适合静态数据的查询场景。


下面是select的函数接口：\
int select (int n, fd_set *readfds, fd_set *writefds, fd_set *exceptfds, struct timeval *timeout);\
select 函数监视的du文件描述符分3类，分别是writefds、readfds、和exceptfds。调用后select函数会阻zhi塞，直到dao有描述副就绪（有数据 可读、可写、或者有except），或者超时（timeout指定等待时间，如果立即返回设为null即可），函数返回。当select函数返回后，可以 通过遍历fdset，来找到就绪的描述符。\
select目前几乎在所有的平台上支持，其良好跨平台支持也是它的一个优点。select的一 个缺点在于单个进程能够监视的文件描述符的数量存在最大限制，在Linux上一般为1024，可以通过修改宏定义甚至重新编译内核的方式提升这一限制，但 是这样也会造成效率的降低。\

poll：
int poll (struct pollfd *fds, unsigned int nfds, int timeout);\
不同与select使用三个位图来表示三个fdset的方式，poll使用一个 pollfd的指针实现。\
~~~c
struct pollfd {
int fd; /* file descriptor */
short events; /* requested events to watch */
short revents; /* returned events witnessed */
};
~~~
pollfd结构包含了要监视的event和发生的event，不再使用select“参数-值”传递的方式。同时，pollfd并没有最大数量限制（但是数量过大后性能也是会下降）。 和select函数一样，poll返回后，需要轮询pollfd来获取就绪的描述符。\
从上面看，select和poll都需要在返回后，通过遍历文件描述符来获取已经就绪的socket。事实上，同时连接的大量客户端在一时刻可能只有很少的处于就绪状态，因此随着监视的描述符数量的增长，其效率也会线性下降。\

epoll:\
epoll的接口如下：
~~~c
int epoll_create(int size)；
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event)；
typedef union epoll_data {
void *ptr;
int fd;
__uint32_t u32;
__uint64_t u64;
} epoll_data_t;
struct epoll_event {
__uint32_t events; /* Epoll events */
epoll_data_t data; /* User data variable */
};
int epoll_wait(int epfd, struct epoll_event * events, int maxevents, int timeout);
~~~
复制代码\
主要是epoll_create,epoll_ctl和epoll_wait三个函数。epoll_create函数创建epoll文件描述符，参数size并不是限制了epoll所能监听的描述符最大个数，只是对内核初始分配内部数据结构的一个建议。返回是epoll描述符。-1表示创建失败。epoll_ctl 控制对指定描述符fd执行op操作，event是与fd关联的监听事件。op操作有三种：添加EPOLL_CTL_ADD，删除EPOLL_CTL_DEL，修改EPOLL_CTL_MOD。分别添加、删除和修改对fd的监听事件。epoll_wait 等待epfd上的io事件，最多返回maxevents个事件。
在 select/poll中，进程只有在调用一定的方法后，内核才对所有监视的文件描述符进行扫描，而epoll事先通过epoll_ctl()来注册一 个文件描述符，一旦基于某个文件描述符就绪时，内核会采用类似callback的回调机制，迅速激活这个文件描述符，当进程调用epoll_wait() 时便得到通知。
epoll的优点主要是一下几个方面：
1. 监视的描述符数量不受限制，它所支持的FD上限是最大可以打开文件的数目，这个数字一般远大于2048,举个例子,在1GB内存的机器上大约是10万左 右，具体数目可以cat /proc/sys/fs/file-max察看,一般来说这个数目和系统内存关系很大。select的最大缺点就是进程打开的fd是有数量限制的。这对 于连接数量比较大的服务器来说根本不能满足。虽然也可以选择多进程的解决方案( Apache就是这样实现的)，不过虽然linux上面创建进程的代价比较小，但仍旧是不可忽视的，加上进程间数据同步远比不上线程间同步的高效，所以也 不是一种完美的方案。
2. IO的效率不会随着监视fd的数量的增长而下降。epoll不同于select和poll轮询的方式，而是通过每个fd定义的回调函数来实现的。只有就绪的fd才会执行回调函数。
3. 支持电平触发和边沿触发（只告诉进程哪些文件描述符刚刚变为就绪状态，它只说一遍，如果我们没有采取行动，那么它将不会再次告知，这种方式称为边缘触发）两种方式，理论上边缘触发的性能要更高一些，但是代码实现相当复杂。
4. mmap加速内核与用户空间的信息传递。epoll是通过内核于用户空间mmap同一块内存，避免了无畏的内存拷贝。


c++ 只能动态分配：  

      其实，编译器在为类对象分配栈空间时，会先检查类的析构函数的访问性（其实不光是析构函数，只要是非静态的函数，编译器都会进行检查）。如果类的析构函数在类外部无法访问，则编译器拒绝在栈空间上为类对象分配内存。因此，可以将析构函数设为private，这样就无法在栈上建立类对象了。但是为了子类可以继承，最好设置成protected。

只能静态分配：
    只有使用new运算符，对象才会被建立在堆上。因此只要限制new运算符就可以实现类对象只能建立在栈上。可以将new运算符设为私有。


opengl:
    去除空白 discard，alpha通道为0
    freetype
    当面加载完成之后，我们需要定义字体大小，这表示着我们要从字体面中生成多大的字形：
    一个FreeType面中包含了一个字形的集合。我们可以调用FT_Load_Char函数来将其中一个字形设置为激活字形。这里我们选择加载字符字形’X’：if (FT_Load_Char(face, 'X', FT_LOAD_RENDER))
    通过将FT_LOAD_RENDER设为加载标记之一，我们告诉FreeType去创建一个8位的灰度位图，我们可以通过face->glyph->bitmap来访问这个位图。
    在需要渲染字符时，我们可以加载一个字符字形，获取它的度量值，并生成一个纹理，但每一帧都这样做会非常没有效率。我们应将这些生成的数据储存在程序的某一个地方，在需要渲染字符的时候再去调用。我们会定义一个非常方便的结构体，并将这些结构体存储在一个地图中。OpenGL要求所有的纹理都是4字节对齐的，即纹理的大小永远是4字节的倍数。通常这并不会出现什么问题，因为大部分纹理的宽度都为4的倍数并/或每像素使用4个字节，但是现在我们每个像素只用了一个字节，它可以是任意的宽度。通过将纹理解压对齐参数设为1，这样才能确保不会有对齐问题（它可能会造成段错误）。
    
    我们将使用下面的顶点着色器来渲染字形：
    我们将位置和纹理纹理坐标的数据合起来存在一个vec4中。这个顶点着色器将位置坐标与一个投影矩阵相乘，并将纹理坐标传递给片段着色器：

    要渲染一个字符，我们从之前创建的Characters映射表中取出对应的字符结构体，并根据字符的度量值来计算四边形的维度。根据四边形的维度我们就能动态计算出6个描述四边形的顶点，并使用glBufferSubData函数更新VBO所管理内存的内容。

    顶点 几何 片段着色器


connect 非阻塞\
    1.fctnl\
    2.多进程，fork处理accept的socket\
    3.select  FD_SET(sock, &readfds);     //将新创建的socket添加到readfds中

select过程：\
    a. 从用户空间将fd_set拷贝到内核空间\
    b. 注册回调函数\
    c. 调用其对应的poll方法\
    d. poll方法会返回一个描述读写是否就绪的mask掩码，根据这个mask掩码给fd_set赋值。\
    e. 如果遍历完所有的fd都没有返回一个可读写的mask掩码，就会让select的进程进入休眠模式，直到发现可读写的资源后，重新唤醒等待队列上休眠的进程。如果在规定时间内都没有唤醒休眠进程，那么进程会被唤醒重新获得CPU，再去遍历一次fd。\
    f. 将fd_set从内核空间拷贝到用户空间\
缺点：两次拷贝耗时、轮询所有fd耗时，支持的文件描述符太小\
优点：跨平台支持

poll\
    优点：连接数（也就是文件描述符）没有限制（链表存储）\
    缺点：大量拷贝，水平触发（当报告了fd没有被处理，会重复报告，很耗性能）

epoll\
我们在使用ET的时候，必须采用非阻塞套接口，避免某文件句柄在阻塞读或阻塞写的时候将其他文件描述符的任务饿死
a. 当调用epoll_wait函数的时候，系统会创建一个epoll对象，每个对象有一个evenpoll类型的结构体与之对应，结构体成员结构如下。

rbn,代表将要通过epoll_ctl向epll对象中添加的事件。这些事情都是挂载在红黑树中。rdlist，里面存放的是将要发生的事件
\
b. 文件的fd状态发生改变，就会触发fd上的回调函数\
c. 回调函数将相应的fd加入到rdlist，导致rdlist不空，进程被唤醒，epoll_wait继续执行。\
d. 有一个事件转移函数——ep_events_transfer，它会将rdlist的数据拷贝到txlist上，并将rdlist的数据清空。\
e. ep_send_events函数，它扫描txlist的每个数据，调用关联fd对应的poll方法去取fd中较新的事件，将取得的事件和对应的fd发送到用户空间。如果fd是LT模式的话，会被txlist的该数据重新放回rdlist，等待下一次继续触发调用。

优点：

没有最大并发连接的限制
只有活跃可用的fd才会调用callback函数
内存拷贝是利用mmap()文件映射内存的方式加速与内核空间的消息传递，减少复制开销。（内核与用户空间共享一块内存）

只有存在大量的空闲连接和不活跃的连接的时候，使用epoll的效率才会比select/poll高

IO分两阶段：

1.数据准备阶段
2.内核空间复制回用户进程缓冲区阶段
一般来讲：阻塞IO模型、非阻塞IO模型、IO复用模型(select/poll/epoll)、信号驱动IO模型都属于同步IO，因为阶段2是阻塞的(尽管时间很短)。只有异步IO模型是符合POSIX异步IO操作含义的，不管在阶段1还是阶段2都可以干别的事。

TCP的四种拥塞控制算法
1.慢开始
2.拥塞控制
3.快重传
4.快恢复

哪些函数不能被定义为虚函数？

1：不能被继承的函数2：不能被重写的函数3：普通函数，友元函数，构造函数，内联成员函数，静态成员函数。

构造函数初始化列表：
    初始化列表中的变量直接初始化，而并非赋值。

    2）成员初始化的顺序只与声明的顺序有关，而跟初始化列表的顺序无关。例如在上面的代码中类A的构造函数初始化列表中，我们写成：z(x),y(x), x(a)，但是我们还是先初始化变量x，然后y，然后z，因为我们先声明的变量a，然后b，然后c

    3）成员之间可以相互初始化：a(12), b(a) 

    什么样的数据必须放到构造函数的初始化列表中呢？
    1）const修饰的成员变量。2)引用类型的成员变量。

queue如何实现的：
    用一个map，指向缓冲区

2.time_wait状态产生的原因

    1）为实现TCP全双工连接的可靠释放

    2）为使旧的数据包在网络因过期而消失

MVCC
我们认为他就是乐观锁的一整实现方式.每一行加两列，一个初试版本号，一个结束版本号
    1.MVCC手段只适用于Msyql隔离级别中的读已提交（Read committed）和可重复读（Repeatable Read

    2.Read uncimmitted由于存在脏读，即能读到未提交事务的数据行，所以不适用MVCC.


协程又可以称为用户线程,微线程，可以将其理解为单个进程或线程中的多个用户态线程，这些微线程在用户态进程控制和调度.协程的实现方式有很多种，包括

使用glibc中的ucontext库实现
利用汇编代码切换上下文
利用C语言语法中的switch-case的奇淫技巧实现(protothreads)
利用C语言的setjmp和longjmp实现

协程较于函数和线程的优点
相比于函数:协程避免了传统的函数调用栈，几乎可以无限地递归
相比与线程:协程没有内核态的上下文切换，近乎可以无限并发。协程在用户态进程显式的调度，可以把异步操作转换为同步操作，也意味着不需要加锁,避免了加锁过程中不必要的开销。

协程的任务调度是在用户态完成,需要代码里显式地将CPU交给其他协程,是协作式的

一组相互依赖的任务设计为协程。这样,当一个协程任务完成之后,可以手动的进行任务切换，把当前任务挂起(yield),切换到另一个协程区工作

11. 
针对listenfd，默认使用LT模式，ET模式并无实际意义，也无收益

针对connectfd，使用ET模式则需要注意到与LT模式的读写逻辑不同，比如读取数据，则需要在一个事件内读取完毕


Stevens在文章中一共比较了五种IO Model：
    blocking IO
    nonblocking IO
    IO multiplexing
    signal driven IO
    asynchronous IO

LT模式的优点在于：事件循环处理比较简单，无需关注应用层是否有缓冲或缓冲区是否满，只管上报事件。缺点是：可能经常上报，可能影响性能。

24. epoll  ET和LT

    LT水平触发(level-triggered)
    socket接收缓冲区不为空 有数据可读 读事件一直触发\
    socket发送缓冲区不满 可以继续写入数据 写事件一直触发

    ET边沿触发(edge-triggered)
    socket的接收缓冲区状态变化时触发读事件，即空的接收缓冲区刚接收到数据时触发读事件
    socket的发送缓冲区状态变化时触发写事件，即满的缓冲区刚空出空间时触发读事件

    nginx 采用边沿触发

    epoll使用RB-Tree红黑树去监听并维护所有文件描述符  还会再建立一个list链表，用于存储准备就绪的事件.

    <b>epoll高效的本质在于:</b>

    减少了用户态和内核态的文件句柄拷贝
    减少了对可读可写文件句柄的遍历
    mmap 加速了内核与用户空间的信息传递，epoll是通过内核与用户mmap同一块内存，避免了无谓的内存拷贝
    IO性能不会随着监听的文件描述的数量增长而下降
    使用红黑树存储fd，以及对应的回调函数，其插入，查找，删除的性能不错，相比于hash，不必预先分配很多的空间

    epoll相比于select并不是在所有情况下都要高效，例如在如果有少于1024个文件描述符监听，且大多数socket都是出于活跃繁忙的状态，这种情况下，select要比epoll更为高效，因为epoll会有更多次的系统调用，用户态和内核态会有更加频繁的切换。

    每个使用ET模式的文件描述符都应该是非阻塞的。 如果描述符是阻塞的，那么读或写操作将会因没有后续事件而一直处于阻塞状态 。

25. select核心是调用tcp的poll，不停查询

    每次调用select，都需要把fd集合从用户态拷贝到内核态，这个开销在fd很多时会很大；\
    同时每次调用select都需要在内核遍历传递进来的所有fd，这个开销在fd很多时也很大；\
    select对于超时值提供了更好的精度：微秒，而poll是毫秒。\
    poll基于链表，没有最大连接数限制;\

26. 非阻塞accept可以用select poll的方式实现


29. 下面简要分析一下epoll的工作过程：

(1) epoll_wait调用ep_poll，当rdlist为空（无就绪fd）时挂起当前进程，知道rdlist不空时进程才被唤醒。

(2) 文件fd状态改变（buffer由不可读变为可读或由不可写变为可写），导致相应fd上的回调函数ep_poll_callback()被调用。

(3) ep_poll_callback将相应fd对应epitem加入rdlist，导致rdlist不空，进程被唤醒，epoll_wait得以继续执行。

(4) ep_events_transfer函数将rdlist中的epitem拷贝到txlist中，并将rdlist清空。

(5) ep_send_events函数（很关键），它扫描txlist中的每个epitem，调用其关联fd对用的poll方法（图中蓝线）。此时对poll的调用仅仅是取得fd上较新的events（防止之前events被更新），之后将取得的events和相应的fd发送到用户空间（封装在struct epoll_event，从epoll_wait返回）。之后如果这个epitem对应的fd是LT模式监听且取得的events是用户所关心的，则将其重新加入回rdlist（图中蓝线），否则（ET模式）不在加入rdlist。

所以，ET下只有wait的时候调用回调函数的时候触发，
LT还有，send_event的时候也触发
