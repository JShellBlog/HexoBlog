
## 1. Base
并行编程困难程度在逐步的减化，如摩尔定律描述，硬件的性能提升能加快并行计算的能力，并且有越来越多的涉及并行技术的开源项目例如Linux kernel， database systems, message-passing sysyems。

### 1.1. 并行开销
涉及并行开销逐渐减小：
1. 脚本如python， bash
2. fork(), wait() 多进程
3. POSIX 多线程(pthread_create(),  pthread_join(),  pthread_exit())
4. POSIX Locking(pthread_mutex_lock(), pthread_mutex_unlock()), Reader-Writer Locking(pthread_rwlock_init(), pthread_rwlock_rdlock(), pthread_rwlock_wrlock(), pthread_rwlock_unlock())
5. Atomic operation

永远记住，进程内的通信和消息传递总是比共享内存的多线程执行要好。


### 1.2. 分隔和同步设计
编写并行软件时最重要的考虑是**如何进行分割**。正确的分割问题能够让解决办法简单、可扩展并且高性能，而不恰当的分割问题则会产生缓慢且复杂的解决方案。

## 2. RCU(Read-Copy Update)
RCU 主要用于**读多写少的情况**，例如文件查找等。
>写者修改对象的过程是：首先复制生成一个副本，然后更新这个副本，最后使用新的对象替换旧的对象。在写者执行复制更新的时候读者可以读数据。

>写者删除对象，必须等到所有访问被删除对象的读者访问结束，才能执行销毁操作。RCU的关键技术是怎么判断所有读者已经完成访问。等待所有读者访问结束的时间称为宽限期（grace period）。

>RCU的优点是读者没有任何同步开销：不需要获取任何锁，不需要执行原子指令，（在除了阿尔法以外的处理器上）不需要执行内存屏障。但是写者的同步开销比较大，写者需要延迟对象的释放，复制被修改的对象，写者之间必须使用锁互斥。

RCU 允许读操作可以与更新操作并发执行，这一点提升了程序的可扩展性。常规的互斥锁让并发线程互斥执行，并不关心该线程是读者还是写者，而读写锁在没有写者时允许并发的读者。

RCU 由三种基础机制构成，第一个机制用于插入，第二个用于删除，第三个用于让读者可以不受并发的插入和删除干扰。

### 2.1. 发布、订阅机制

```c
struct foo {
int a;
int b;
int c;
};
struct foo *gp = NULL;

/* . . . */
p = kmalloc(sizeof(*p), GFP_KERNEL);
p->a = 1;
p->b = 2;
p->c = 3;
gp = p;
```

`问题`
不幸的是，这块代码无法保证编译器和 CPU 会按照顺序执行最后四条赋值语句。如果对 gp 的赋值发生在初始化 p 的各字段之前，那么并发的读者会读到未初始化的值。

**rcu_assign_pointer()原语将内存屏障封装起来，让其拥有发布的语义。**
使用`rcu_assign_pointer(gp, p);` 替换最后一句，强制让编译器和 CPU 在为 p 的各字段赋值后再去为 gp 赋值。


不过，只保证更新者的执行顺序并不够，因为读者也需要保证读取顺序。下面的语句可能发生读取`p->a,p->b` 发生在 `p = gp` 之前。
```c
p = gp;
if (p != NULL) {
    do_something_with(p->a, p->b, p->c);
}
```

`rcu_dereference()`原语用了各种**内存屏障指令和编译器指令**来达到这一目的。
```c
rcu_read_lock();
p = rcu_dereference(gp);
if (p != NULL) {
    do_something_with(p->a, p->b, p->c);
}
rcu_read_unlock();
```

`rcu_dereference()`原语用一种“订阅”的办法获取指定指针的值。保证后续的解引用操作可以看见在对应的“发布”操作（`rcu_assign_pointer`）前进行的初始化。 rcu_read_lock()和 rcu_read_unlock()是肯定需要的：这对原语定义了 RCU 读端的临界区。

RCU 的发布与订阅原语   
类别 | 发布 | 取消发布 | 订阅 
:- | :- | :- | :- 
指针 | rcu_assign_pointer() | rcu_assign_pointer(…，NULL) | rcu_dereference() 
链表 | list_add_rcu() <br> list_add_tail_rcu() <br> list_replace_rcu() | list_del_rcu() | list_for_each_entry_rcu()
哈希 | hlist_add_after_rcu() | hlist_del_rcu() | hlist_for_each_entry_rcu()
链表 | hlist_add_before_rcu() <br> hlist_add_head_rcu() <br> Hlist_replace_rcu() | hlist_del_rcu() | hlist_for_each_entry_rcu()

### 2.2. 等待已有的 RCU 读者执行完毕
RCU 就是一种等待事物结束的方式，当然，有很多其他的方式可以用来等待事物结束，比如引用计数，读写锁，事件等等。 RCU的最伟大之处在于它可以等待（比如） 20000 种不同的事物，而无需显式地去跟踪它们中的每一个，也无需去担心对性能的影响，对扩展性的限制，复杂的死锁场景，还有内存泄漏带来的危害等等使用显式跟踪手段会出现的问题。

在 RCU 的例子中，被等待的事物称为“RCU 读端临界区”。 RCU 读端临界区从 rcu_read_lock()原语开始，到对应的 rcu_read_unlock()原语结束。

```c
1 struct foo {
2 struct list_head *list;
3 int a;
4 int b;
5 int c;
6 };
7 LIST_HEAD(head);
8
9 /* . . . */
10
11 p = search(head, key);深入理解并行编程
12 if (p == NULL) {
13 /* Take appropriate action, unlock, and
return. */
14 }
15 q = kmalloc(sizeof(*p), GFP_KERNEL);
16 *q = *p;
17 q->b = 2;
18 q->c = 3;
19 list_replace_rcu(&p->list, &q->list);
20 synchronize_rcu();
21 kfree(p);
```
第 16-19 行正如 RCU 其名（读——拷贝——更新）：在允许并发“读”的同时，第 16 行“拷贝”，第 17到 19 行“更新”。
`synchronize_rcu()`原语第一眼看上去有点神秘。毕竟它需要等待所有 RCU 读端临界区完成。

### 2.3. RCU 总结
基本RCU操作，
对于reader，RCU的操作包括：
1. rcu_read_lock，用来标识RCU read side临界区的开始。
2. rcu_dereference，该接口用来获取RCU protected pointer。reader要访问RCU保护的共享数据，当然要获取RCU protected pointer，然后通过该指针进行dereference的操作。
3. rcu_read_unlock，用来标识reader离开RCU read side临界区

对于writer，RCU的操作包括：  
1. rcu_assign_pointer。该接口被writer用来进行removal的操作，在witer完成新版本数据分配和更新之后，调用这个接口可以让RCU protected pointer指向RCU protected data。
2. synchronize_rcu。writer端的操作可以是同步的，也就是说，完成更新操作之后，可以调用该接口函数等待所有在旧版本数据上的reader线程离开临界区，一旦从该函数返回，说明旧的共享数据没有任何引用了，可以直接进行reclaimation的操作。
3. call_rcu。当然，某些情况下（例如在softirq context中），writer无法阻塞，这时候可以调用call_rcu接口函数，该函数仅仅是注册了callback就直接返回了，在适当的时机会调用callback函数，完成reclaimation的操作。这样的场景其实是分开removal和reclaimation的操作在两个不同的线程中：updater和reclaimer。

## Reference
[perfbook](https://mirrors.edge.kernel.org/pub/linux/kernel/people/paulmck/perfbook/perfbook.html)
[深入理解并行编程](https://book.douban.com/subject/27078711/)
[Introduction to RCU](http://www2.rdrop.com/users/paulmck/RCU/)
[RCU(1) 概述](http://www.wowotech.net/kernel_synchronization/461.html)
[Linux内核同步机制之（七）：RCU基础](http://www.wowotech.net/kernel_synchronization/rcu_fundamentals.html)
[Linux Documentation/RCU](https://github.com/torvalds/linux/tree/master/Documentation/RCU)
