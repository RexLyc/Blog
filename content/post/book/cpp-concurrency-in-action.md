---
title: "Cpp Concurrency in Action读书笔记"
date: 2024-04-14T00:41:02+08:00
categories:
- 计算机科学与技术
- C++
tags:
- C++
- 施工中
- 读书笔记
thumbnailImagePosition: left
draft: true

---
本文重点关注一下并发代码设计，无锁数据结构设计。内容主要来自于书籍《C++ Concurrency In Action》，以及一些网络课程，如CppCon、Mit OpenCourse。
<!--more-->

## 术语
1. interleaving：指来自不同线程的指令交错执行
2. discipline：约束，比如lock-free discipline

## 并发编程基础概念
1. 为什么并发？
    1. 关注点分离：让一个线程/进程，专注于一件事情
    2. 性能：任务并行（执行任务的不同部分）、数据并行（计算数据的不同部分）

## 基于锁的数据结构
第六章中给出了若干线程安全的数据结构的设计，总体上看设计思路包括
1. 设计优先，考虑需要进行支持的操作
2. 确保线程持有锁的时间最短，
    1. 使用更细粒度的锁
    2. 引入虚拟节点。例如队列，可以用头尾虚拟节点将操作分离
3. 使用锁和条件变量，提供wait和notify功能。考虑异常抛出时对其他请求锁的线程的唤醒。

下面摘抄几个例子

查询表
```cpp
template<typename Key,typename Value,typename Hash=std::hash<Key> >
class threadsafe_lookup_table
{
private:
     class bucket_type
     {
     private:
          typedef std::pair<Key,Value> bucket_value;
          typedef std::list<bucket_value> bucket_data;
          typedef typename bucket_data::iterator bucket_iterator;
          bucket_data data;
          mutable boost::shared_mutex mutex;     // 1
          bucket_iterator find_entry_for(Key const& key) const     // 2
          {
               return std::find_if(data.begin(),data.end(),
               [&](bucket_value const& item)
               {return item.first==key;});
          }
     public:
          Value value_for(Key const& key,Value const& default_value) const
          {
               boost::shared_lock<boost::shared_mutex> lock(mutex);     // 3
               bucket_iterator const found_entry=find_entry_for(key);
               return (found_entry==data.end())?
                    default_value:found_entry->second;
          }
          void add_or_update_mapping(Key const& key,Value const& value)
          {
               std::unique_lock<boost::shared_mutex> lock(mutex);     // 4
               bucket_iterator const found_entry=find_entry_for(key);
               if(found_entry==data.end())
               {
                    data.push_back(bucket_value(key,value));
               }
               else
               {
                    found_entry->second=value;
               }
          }
          void remove_mapping(Key const& key)
          {
               std::unique_lock<boost::shared_mutex> lock(mutex);     // 5
               bucket_iterator const found_entry=find_entry_for(key);
               if(found_entry!=data.end())
               {
                    data.erase(found_entry);
               }
          }
     };
     std::vector<std::unique_ptr<bucket_type> > buckets;     // 6
     Hash hasher;
     bucket_type& get_bucket(Key const& key) const     // 7
     {
          std::size_t const bucket_index=hasher(key)%buckets.size();
          return *buckets[bucket_index];
     }
public:
     typedef Key key_type;
     typedef Value mapped_type;
     typedef Hash hash_type;
     threadsafe_lookup_table(
          unsigned num_buckets=19,Hash const& hasher_=Hash()):
          buckets(num_buckets),hasher(hasher_)
     {
          for(unsigned i=0;i<num_buckets;++i)
          {
               buckets[i].reset(new bucket_type);
          }
     }
     threadsafe_lookup_table(threadsafe_lookup_table const& other)=delete;
     threadsafe_lookup_table& operator=(
          threadsafe_lookup_table const& other)=delete;
     Value value_for(Key const& key,
                                             Value const& default_value=Value()) const
     {
          return get_bucket(key).value_for(key,default_value);     // 8
     }
     void add_or_update_mapping(Key const& key,Value const& value)
     {
          get_bucket(key).add_or_update_mapping(key,value);     // 9
     }
     void remove_mapping(Key const& key)
     {
          get_bucket(key).remove_mapping(key);     // 10
     }
};
```

链表
```cpp
template<typename T>
class threadsafe_list
{
     struct node     // 1
     {
          std::mutex m;
          std::shared_ptr<T> data;
          std::unique_ptr<node> next;
          node():next() // 2
          {}
          node(T const& value):     // 3
               data(std::make_shared<T>(value))
          {}
     };
     node head;
public:
     threadsafe_list()
     {}
     ~threadsafe_list()
     {
          remove_if([](node const&){return true;});
     }
     threadsafe_list(threadsafe_list const& other)=delete;
     threadsafe_list& operator=(threadsafe_list const& other)=delete;
     void push_front(T const& value)
     {
          std::unique_ptr<node> new_node(new node(value));     // 4
          std::lock_guard<std::mutex> lk(head.m);
          new_node->next=std::move(head.next);     // 5
          head.next=std::move(new_node);     // 6
     }
     template<typename Function>
     void for_each(Function f)     // 7
     {
          node* current=&head;
          std::unique_lock<std::mutex> lk(head.m);     // 8
          while(node* const next=current->next.get())     // 9
          {
               std::unique_lock<std::mutex> next_lk(next->m);     // 10
               lk.unlock();     // 11
               f(*next->data);     // 12
               current=next;
               lk=std::move(next_lk);     // 13
          }
     }
     template<typename Predicate>
     std::shared_ptr<T> find_first_if(Predicate p)     // 14
     {
          node* current=&head;
          std::unique_lock<std::mutex> lk(head.m);
          while(node* const next=current->next.get())
          {
               std::unique_lock<std::mutex> next_lk(next->m);
               lk.unlock();
               if(p(*next->data))     // 15
               {
                     return next->data;     // 16
               }
               current=next;
               lk=std::move(next_lk);
          }
          return std::shared_ptr<T>();
     }
     template<typename Predicate>
     void remove_if(Predicate p)     // 17
     {
          node* current=&head;
          std::unique_lock<std::mutex> lk(head.m);
          while(node* const next=current->next.get())
          {
               std::unique_lock<std::mutex> next_lk(next->m);
               if(p(*next->data))     // 18
               {
                    std::unique_ptr<node> old_next=std::move(current->next);
                    current->next=std::move(next->next);
                    next_lk.unlock();
               }     // 20
               else
               {
                    lk.unlock();     // 21
                    current=next;
                    lk=std::move(next_lk);
               }
          }
     }
};
```

## 无锁数据结构
### 基本概念
首先问一个问题，为什么要无锁？
1. 减少算法、数据结构的阻塞、等待时间
2. 避免锁和阻塞的问题：忘记锁、忘记释放锁、死锁、上错锁、粗粒度锁很方便但是性能很糟糕，锁之间的组合性不好

做无锁编程需要衡量的两点
1. 我真的需要优化
2. 我的优化确实有效

无锁编程的基础工具：事务思维、```atomic<T>```
1. 所谓的事务思维，就是我们所熟悉的，ACID。原子性、一致性、隔离性、持久性。
2. 在C++中是```atomic<T>```，在C11中是```atomic_*```。这些原子变量提供的能力是
    1. 对他们的读写都是原子性的，无锁的
    2. 根据设计需要，可以控制这些原子读写之间的重排情况
    3. 支持CAS指令。对于C++，提供```compare_exchange_strong```、```compare_exchange_weak```和```exchange```。关于这三者的语义，可以参考[C++ 多线程：原子类型(std::atomic)](https://juejin.cn/post/7086226046931959838)。弱版本允许(spuriously 地)返回 false(即原子对象所封装的值与参数 expected 的物理内容相同，但却仍然返回 false)，即使在预期的实际情况与所包含的对象相比较时也是如此。对于某些循环算法来说，这可能是可接受的行为，并且可能会在某些平台上带来显著的性能提升。在这些虚假的失败中，函数返回false，而不修改预期。对于非循环算法来说， ```compare_exchange_strong``` 通常是首选。
    4. 允许原子化的类型是有限的，而且此时其内存布局可能发生变化。
        > 如果模板参数类型很大，甚至会使用锁来完成

无锁的三种级别：
1. wait-free：最强级别，no one ever waits。任何线程的操作都能在一个有界的操作时间内完成
2. lock-free：稍弱级别，至少一个线程能在给定操作时间内完成
3. obstruction-free：最弱的级别，在其他线程都暂停的情况下，才有一个线程能在给定操作时间内完成

> 这三个级别，在编写代码的时候，是可以起到一定的指导意义的。比如针对一个给定的数据规模，我们可以知道在使用K个线程上下时，算法是wait-free还是lock-free，还是obstruction-free。一般来说，当线程超过一定量级时，性能将会不可避免的从wait-free变为lock-free。

### 基本例子
1. DCL：double-checked lock。控制锁的粒度，我们可以用无锁方法先去读一下。在需要的时候再上锁。而且在这种情况下，我们还可以进一步优化原子变量的读取/写入次数。
    ```cpp
    // 以经典的，工厂方法中可能用到的延迟构造举例
    // 当然更好的方法是用static局部变量

    // 在类Widget中
    atomic<Widget*> Widget::pInstance{nullptr};
    Widget* Widget::getInstance() {
        // 可以尝试在进入时读取一次并保存
        // Widget *p = pInstance; // 已经完成初始化后，这样能更快
        if(pInstance == nullptr) {
            lock_guard<mutex> lock{mutex};
            if(pInstance == nullptr) {
                pInstance = new Widget();
            }
        }
        return pInstance;
    }

    ```
2. 生产者消费者场景:
    考虑一个有若干槽位的，邮箱一样的数据结构。可以向其中任意的槽输入数据，并由消费者消费。
    ![生产者的无锁设计](/images/book/cpp-concurrency/producer-lockfree.png)
    ![消费者的无锁设计](/images/book/cpp-concurrency/consumer-lockfree.png)
    上面的设计并没有很完整（缺少了必备的CAS部分），但从上面的设计可以看出，无论无锁数据结构的场景多么复杂。其实最终关心的事情都是同样的，就是Compare And Set。
   

## 重点推荐参考
1. [1.84.0版本boost，第20章Boost.Lockfree](https://www.boost.org/doc/libs/1_84_0/doc/html/lockfree.html)
1. [论文：Simple, Fast, and Practical Non-Blocking and Blocking Concurrent Queue Algorithms](https://www.cs.rochester.edu/~scott/papers/1996_PODC_queues.pdf)
2. [CppCon 2014: Herb Sutter "Lock-Free Programming (or, Juggling Razor Blades), Part I"](https://www.youtube.com/watch?v=c1gO9aB9nbs)
3. [CppCon 2014: Herb Sutter "Lock-Free Programming (or, Juggling Razor Blades), Part II"](https://www.youtube.com/watch?v=CmxkPChOcvw)
4. [17. Synchronization Without Locks](https://www.youtube.com/watch?v=5sZo3SrLrGA)