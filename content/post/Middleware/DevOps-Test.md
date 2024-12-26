---
title: "DevOps：软件测试篇"
date: 2024-01-11T14:24:57+08:00
categories:
- 计算机科学与技术
- DevOps
tags:
- DevOps
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/DevOps.jpg
---
软件测试是DevOps的重要一环，本篇记录一些必备的测试知识。
<!--more-->
## 测试类型
- 单元测试
- 集成测试
- 黑盒测试
- 白盒测试
- 压力测试

## 工具
测试工具的种类非常多，而且也没有放诸四海皆有效的一个通用完整工具。

### JMeter
具体参考[JMeter]({{<ref "/content/post/Tools/JMeter-all-in-one.md">}})

### Perf
Linux下内核级支持的一个工具，主要用于对当前CPU上的执行情况进行采样分析。但其数据可视化效果比较差，一般会和其他工具结合。下面列举一些性能测试场景的使用案例。
```bash
# 对全部进程采样60秒，频率99HZ，保留调用堆栈
perf record -F 99 -a -g -- sleep 60

# 查看全部支持的perf event
perf list
# 注意有些事件只有在sudo权限下才可被观测
sudo perf list

# -e指定采样的事件，对缺页异常进行30秒的采样
sudo perf record -e page-faults -a -g -- sleep 30 && perf script > memory.perf

# 采样块设备事件
sudo perf record -e block:block_rq_insert -a -g -- sleep 30 && perf script > block.perf
```

> WSL2对Perf的支持并不一定很好。Perf性能测试最好在原生Linux系统上进行。

### GPerfTools
Google推出的性能测试工具，Google Performance Tools。GPerfTools也是以源代码形式发布的测试工具。在使用前需要在开发环境中进行编译再使用。下面是基本的编译安装步骤。
```bash
# 编译安装gperftools
# 依赖libtool(编译用）、libunwind（一个堆栈工具）、graphviz&ghostscript（绘图、PDF）
apt install libtool graphviz ghostscript

# 选择合适的libunwind版本
wget https://github.com/libunwind/libunwind/archive/refs/tags/v1.8.0.tar.gz
# 省去解压缩步骤
autoreconf -i
./configure
make -j4
sudo make install

# 选择合适的gperftool版本
wget https://github.com/gperftools/gperftools/archive/refs/tags/gperftools-2.15.tar.gz
# 省去解压缩步骤
./autogen.sh
./configure
make -j4
sudo make install
```

使用中，主要有侵入和非侵入两种模式，以及[cpu](https://gperftools.github.io/gperftools/cpuprofile.html)、[malloc](https://gperftools.github.io/gperftools/tcmalloc.html)、[heap check](https://gperftools.github.io/gperftools/heap_checker.html)、[heap profile](https://gperftools.github.io/gperftools/heapprofile.html)等profiler。

非侵入是指不修改代码，只在运行前调整环境变量和加载环境，并进行测试。例如
```bash
# 生成pdf报告

```

### PProf
前面的Perf、GPerfTool都是Profiling工具，就是生成采样数据的工具。PProf是用来对数据做一定程度的可视化，一般生成文本，或者pdf文档。也可以用于执行数据转换（比如从gperftool到perf数据）

### FlameGraph
[FlameGraph](https://github.com/brendangregg/FlameGraph)是一个非常强大的，性能测试数据可视化工具。它主要提供了以下几种内容的数据可视化
  1. CPU：展示CPU调用堆栈深度，以及采样次数之间的对比信息
  1. Memory：展示对内存的释放分配信息
  1. Off-CPU：展示CPU受I/O等原因的阻塞信息，此时进程不在CPU上执行，因此名为Off-CPU
  1. Hot/Cold：将CPU、Off-CPU图进行结合，在同一个图内展示活跃部分和阻塞部分
  1. Differential：性能差分图。对于一些在持续优化的代码，使用这种图形能够快速对比代码优化工作在不同的函数上带来的时间减少、时间增长，以得知性能优化成果
注意火焰图的纵坐标是调用堆栈，代表了调用的父子信息。横坐标是采样的命中次数，以及按名称的排序，**不是运行时的顺序**。在横轴上越长的函数，代表其相对被调用的次数越多（运行时间越长），可能存在性能问题。

> 参考：开发者提供的[视频分享教程](https://www.youtube.com/watch?v=D53T1Ejig1Q)

FlameGraph本身并不提供采样工具，它需要其他工具获得的性能采样数据来进行绘制。一个基本的CPU火焰图使用流程如下。
```bash
# FlameGraph以git仓库形式发放
git clone git@github.com:brendangregg/FlameGraph.git
cd FlameGraph

# 先用perf工具对全部进程采样60秒，频率99HZ
perf record -F 99 -a -g -- sleep 60
# 此时会生成perf.data二进制数据，将其转为文本数据
perf script > output.perf
# 为了下一步的绘图，用FlameGraph的工具对其进行折叠
./stackcollapse-perf.pl output.perf > output.folded
# 绘制svg
./flamegraph.pl output.folded > output.svg
```

其他类型的火焰图的创建流程类似，只不过根据所创建的类型，在最后生成svg图片的参数上有所不同，例如
```bash
# 一次完成从perf script结果的折叠和绘图，绘制成为内存类型（缺页异常），数据来源见Perf章节
./stackcollapse-perf.pl < memory.perf | ./flamegraph.pl --color=mem \
    --title="Page Faults" --countname="pages" > memory.svg

# 对I/O导致Off-CPU的情况的采样，数据来源见Perf章节
./stackcollapse-perf.pl < block.perf | ./flamegraph.pl --color=io \
    --title="Block I/O Flame Graph" --countname="I/O" > block.svg

# 差分图，先对CPU采样做两次折叠
./stackcollapse-perf.pl out.stack1 > out.stack1.folded
./stackcollapse-perf.pl out.stack2 > out.stack2.folded
# 用difffolded计算区别，并调用输出绘图
./difffolded.pl out.stack1.folded out.stack2.folded| ./flamegraph.pl > diff.svg
```

FlameGraph在CPU、内存、Off-CPU、Differential等各方面都有不同的子类型。根据测试需要，对专门的图形进行研究。

## 测试框架
### GoogleTest
[GoogleTest](https://google.github.io/googletest/)是Google开源的C++使用的测试框架，源代码形式发布，使用CMake、Bazel等可以很方便的集成到项目环境中，下面是一个典型的接入GoogleTest的CMake工程文件
```CMakeLists
cmake_minimum_required(VERSION 3.14)
project(cpp_learn)

set(CMAKE_CXX_STANDARD 20)

# 使用FetchContent自动获得，googletest源码
include(FetchContent)
FetchContent_Declare(
    googletest
    URL https://github.com/google/googletest/archive/refs/tags/v1.14.0.zip
)
# 强制控制Google Test框架使用共享的C Runtime（CRT）库
# CACHE BOOL "" FORCE的意思是，为该选项强制定义一个缓存的变量，BOOL类型，备注说明为空
set(gtest_force_shared_crt ON CACHE BOOL "" FORCE)
FetchContent_MakeAvailable(googletest)

# 在传统的CMake中，用来说明要生成测试目标，使后续的add_test生效
enable_testing()

# 包含测试代码的测试文件
add_executable(
    hello_test
    hello_test.cpp
)

target_link_libraries(
    hello_test

# gtest部分
    gtest_main
# gmock部分，必须单独引入
    gmock_main
)

include(GoogleTest)
# gtest_disvoer_tests旨在替代add_test、gtest_add_tests
# 可以自动获取可执行文件中的测试样例，无需一个个在CMake中填写
gtest_discover_tests(hello_test)
```

基本测试语法示例
```cpp
// 在hello_test.cpp内

#include <gtest/gtest.h>

int Factorial(int n) {
    int result = 1;
    for(int i = 1; i <= n; ++i){
        result *= i;
    }
    return result;
}

// TEST(测试大组名称，测试小组名称)
TEST(FactorialTest, HandlesZeroInput) {
EXPECT_EQ(Factorial(0), 1);
}

// 同一个大组名称会被归类到一起
TEST(FactorialTest, HandlesPositiveInput) {
EXPECT_EQ(Factorial(3), 6);
// 任意测试宏后面都可以跟着一个标准IO输出流，用来输出必要信息
EXPECT_EQ(Factorial(8), 40320) << "Factorail(8)";
}


// 使用Fixture风格，对固定组织起来的数据进行测试
// 待测试数据结构
template <typename E>
class Queue {
 public:
  Queue();
  void Enqueue(const E& element);
  E* Dequeue();  // Returns NULL if the queue is empty.
  size_t size() const;
  ...
};

// 必须继承testing::Test，准备所需要测试的一系列固定数据结构
class QueueTest : public testing::Test {
 protected:
  void SetUp() override {
     // q0_ remains empty
     q1_.Enqueue(1);
     q2_.Enqueue(2);
     q2_.Enqueue(3);
  }

  // 必要时候需要编写TearDown释放资源
  // void TearDown() override {}

  Queue<int> q0_;
  Queue<int> q1_;
  Queue<int> q2_;
};

// TEST_F(测试类类型，测试小组名称)
// 每一个TEST_F开始前，GTest负责创建QueueTest实例，结束后释放
// SetUp、TearDown
TEST_F(QueueTest, DequeueWorks) {
  int* n = q0_.Dequeue();
  EXPECT_EQ(n, nullptr);

  n = q1_.Dequeue();
  ASSERT_NE(n, nullptr);
  EXPECT_EQ(*n, 1);
  EXPECT_EQ(q1_.size(), 0);
  delete n;

  n = q2_.Dequeue();
  ASSERT_NE(n, nullptr);
  EXPECT_EQ(*n, 2);
  EXPECT_EQ(q2_.size(), 1);
  delete n;
}
```

一般情况下，将测试源文件和gtest_main进行链接，成为一个可执行程序。如果非要手动在main函数中运行测试，示例如下
```cpp
#include <gtest/gtest.h>

TEST(XX,XX) {
    // 测试代码
}

int main(int argc, char **argv) {
    // 根据命令行初始化
    testing::InitGoogleTest(&argc, argv);
    // ...

    // RUN_ALL_TEST()是调用测试的入口，而且其返回值必须反馈
    return RUN_ALL_TESTS();
}
```

GoogleTest还有强大的[Mocking功能](https://google.github.io/googletest/gmock_for_dummies.html)，Mock Object是指那些在测试中使用的模拟对象。构成它们的代码一般是来自于真实的生产环境代码（区分于Fake Object），但是限制了调用场景，以满足可测试的需求。在以下场景出现时可以考虑使用Mock。
1. 希望调整软件原型设计
2. 测试流程依赖一些高成本、不稳定操作（网络、数据库）
3. 希望测试一些不容易复现的错误情况
4. 需要测试对外接口，但这种对两端的同时观测很困难

总之，使用Mock的目的是，模拟一些组件的工作。而使用Mock的方式也很容易，也是继承+宏的方式，例如。
```cpp
// 代码来自于官方文档

// 该例子是用于测试一个类似LOGO语言（小乌龟）的绘图API
// 我们想要测试的是，程序在绘制过程中是否能绘制出想要的结果
// 而调用真实API并做截图对比并不可取

// 更好的办法是封装绘图API，查看绘图原子操作的调用是否满足要求
// 比如画笔向上向下移动了多少

// 因此将待测试API进行一定程度的封装，封装为Turtle，而不是直接去调用系统API
// 而且也并不需要给出实现，纯虚就可以
class Turtle {
public:
  virtual ~Turtle() {}
  virtual void PenUp() = 0;
  virtual void PenDown() = 0;
  virtual void Forward(int distance) = 0;
  virtual void Turn(int degrees) = 0;
  virtual void GoTo(int x, int y) = 0;
  virtual int GetX() const = 0;
  virtual int GetY() const = 0;
};

// 编写Mock类型
class MockTurtle: public Turtle {
public:
  // MOCK宏要覆盖每一个需要测试的API
  MOCK_METHOD(void, PenUp, (), (override));
  MOCK_METHOD(void, PenDown, (), (override));
  MOCK_METHOD(void, Forward, (int distance), (override));
  MOCK_METHOD(void, Turn, (int degrees), (override));
  MOCK_METHOD(void, GoTo, (int x, int y), (override));
  MOCK_METHOD(int, GetX, (), (const, override));
  MOCK_METHOD(int, GetY, (), (const, override));
}


// 真正的测试代码
#include <gmock/gmock.h>
#include <gtest/gtest.h>

using ::testing::AtLeast;

TEST(PainterTest, CanDrawSomething) {
  // 创建Mock对象
  MockTurtle turtle;
  // 编写一系列对Mock对象行为的预期条件EXPECT_*
  EXPECT_CALL(turtle, PenDown())
      .Times(AtLeast(1));
  
  // 调用相关测试功能
  Painter painter(&turtle);
  EXPECT_TRUE(painter.DrawCircle(0, 0, 10));

  // 在测试结束，gMock会自动检查所有的EXPECT条件是否满足
}
```

Mock测试的难点在于，如何设置灵活、正确的EXPECT_预期条件。一个更完整的EXCPECT_CALL如下所示
```cpp
/**
 * EXPECT_CALL 的一般结构：指定期待的方法、参数（必须match），指定调用次数
 * 以及函数返回的结果（注意Mock将自动完成并给出这些结果）
 * EXPECT_CALL(mock_object, non-overloaded-method)
    .Times(cardinality)
    .WillOnce(action)
    .WillRepeatedly(action);
 */

using ::testing::Return;
// 使用Return，表示第一次返回是100，第二次是150，之后永远是200
EXPECT_CALL(turtle, GetX())
    .Times(3)
    .WillOnce(Return(100))
    .WillOnce(Return(150))
    .WillRepeatedly(Return(200));

EXPECT_CALL(turtle, GetX())
    .Times(1)
    .WillOnce(Return(500));

// WillXXX是用来给出Mock对象的返回值的，而且顺序和下面的EXPECT正好匹配
// 注意不同EXPECT语句之间是栈，同一个语句内还是按照从左到右的顺序
EXPECT_EQ(turtle.GetX(),500);
EXPECT_EQ(turtle.GetX(),100);
EXPECT_EQ(turtle.GetX(),150);
EXPECT_EQ(turtle.GetX(),200);

// 也支持对指定输入参数进行检查
// 调用Forward时的输入是100
EXPECT_CALL(turtle, Forward(100));
// 允许实参条件宽泛一些，_作为通配
EXPECT_CALL(turtle, GoTo(50, _));
// 进一步结合条件
using ::testing::Ge;
EXPECT_CALL(turtle, Forward(Ge(100)));
// 也可以使用变量
int n=100;
EXPECT_CALL(turtle, Get()).Times(4).WillRepeatedly(Return(n++));

// 在所有的CALL期待都配置完之后，对于同一个接口
// gMock会按照期望的定义顺序的相反顺序, 检查调用是否符合
// 而且gMock对返回值的处理也是相反的顺序
turtle.Get();
// ... 更多测试

// 默认情况下，不同Mock接口的调用也不检查顺序，除非指定
using ::testing::InSequence;
{
    InSequence seq;
    // 期望以下调用必须按顺序出现
    EXPECT_CALL(/*...*/);
    EXPECT_CALL(/*...*/);
}
```

在Mock功能基础上，可以Matcher、Cardinalities、Action进行[自定义的扩展](https://google.github.io/googletest/gmock_cook_book.html)。其中对Matcher的扩展是最常见的。当我们需要有一些通用的测试工具，直接封装到一个函数中，在报错时的可读性并不好。将其封装为一个Matcher是更好的办法。如下所示。
```cpp
// 自定义Matcher
MATCHER(IsDivisibleBy7, "") { return (arg % 7) == 0; }
// 自定义参数化Matcher，MATCHER_P ~ MATCHER_P10提供了至多10个参数
MATCHER_P(HasAbsoluteValue, value, "") { return abs(arg) == value; }

// 对mock_foo.Bar(x)调用的检查，参数x必须能被7整除
EXPECT_CALL(mock_foo, Bar(IsDivisibleBy7()));

EXPECT_THAT(-1, HasAbsoluteValue(2));

// 自定义同类型Matcher，以下是最小实现（四个成员必不可少）
using ::testing::Matcher;
class DivisibleBy7Matcher {
public:
  using is_gtest_matcher = void;
  bool MatchAndExplain(int n, std::ostream*) const {
    return (n % 7) == 0;
  }
  void DescribeTo(std::ostream* os) const {
    *os << "is divisible by 7";
  }

  void DescribeNegationTo(std::ostream* os) const {
    *os << "is not divisible by 7";
  }
};
Matcher<int> DivisibleBy7() {
  return DivisibleBy7Matcher();
}
EXPECT_CALL(foo, Bar(DivisibleBy7()));


// 自定义多态Matcher，只需要将MatchAndExplain实现为函数模板
class NotNullMatcher {
 public:
  using is_gtest_matcher = void;
  template <typename T>
  bool MatchAndExplain(T* p, std::ostream*) const {
    return p != nullptr;
  }

  void DescribeTo(std::ostream* os) const { *os << "is not NULL"; }

  void DescribeNegationTo(std::ostream* os) const { *os << "is NULL"; }
};

NotNullMatcher NotNull() {
  return NotNullMatcher();
}

EXPECT_CALL(foo, Bar(NotNull()));
```

GoogleTest还有很多[高级](https://google.github.io/googletest/advanced.html)的功能，因为内容繁杂，建议查阅文档，这里简单列举一下功能和大致用法，
| 测试功能 | 实现方式 | 示例代码（如有）| 备注 |
| --- | --- | --- | --- |
| 显式的标记成功/失败 | 宏 | SUCCESS() / FAILED()  | 失败会立刻结束测试 |
| 添加一次错误样例 | 宏 | ADD_FAILURE() | 添加一个错误，还会继续运行 |
| 检测异常抛出 | 宏 | EXPECT_THROW(statement,exception_type) | statement可以是任意表达式 |
| 提供更好的测试错误信息 | 函数 | testing::AssertionResult | 对于PRED类测试，可以包装待测试函数返回更好的错误信息，而不是只有简单的true/false |
| 浮点数比较 | 函数 | testing::FLoatLE | 放到EXPECT_PRED/THAT等中使用 |
| 更多的字符串比较工具 | 函数 | testing::HasSubstr | 用于EXPECT_THAT等 |
| 类型断言 | 类模板 | testing::StaticAssertType<T1,T2> | 判断类型是否相等，注意受模板限制，只有真正实例化才能触发判断，如果待实例化内容被优化了，这条测试无法获得判断结果 |
| 调过测试 | 宏 | GTEST_SKIP() | 可以放到Fixture类SetUp，或者TEST/TEST_F等任意GTest流程函数中 |
| 打印任意变量 | 函数 | testing::PrintToString(x) | 返回字符串 |
| 自定义任意变量的打印 | 函数模板重载 | AbslStringfy(Sink &sink, YourType t) | 定义为友元函数，或者在同一命名空间 |
| 死亡测试 | 宏 | ASSERT_DEATH、EXPECT_EXIT | 在指定条件下，程序应当立即结束，死亡测试负责这种情况 |
| 额外的日志 | 函数 | testing::Test::RecordProperty(key,value) | 打印到日志 |
| 全局SetUp/TearDown | 类继承 | testing::Environment | 继承并重写Env的SetUp/TearDown |
| 参数化测试 | 类继承、宏 | TEST_P、testing::TestWithParam<> | 用INSTANTIATE_TEST_SUITE_P给出一系列测试用的数据，用GetParam获取 |
| 多类型测试 | 类继承、宏 | 定义Fixture模板、TYPED_TEST_SUITE、TYPED_TEST | 适用于对于同一个接口的多种实现的测试 |
| 参数化多类型测试 | 类继承、宏 | Fixture模板、TYPED_TEST_SUTE_P、TYPED_TEST_P | 参数化 |
| 运行期注册测试用例 | 函数模板 | RegisterTest | 通过Fixture类名，测试组名、类型参数、值参数、Fixture工厂等创建测试 |
| 获取测试信息 | 函数 | testing::UnitTest::GetInstance()->XXX() | 获取测试用例的元信息 |
| 自定义测试事件 | 类继承 | testing::TestEventListener等 | 用于在每一个测试前后的事件中添加自定义业务 |
| 各类测试选项 | 命令行选项 | 例如--gtest_repeat=1000 | 重复测试、测试样例过滤、打乱测试顺序等 |

一些限制
1. 会产生fatal类的错误测试宏（如```FAIL_*```、```ASSERT_*```），无法使用在非void返回值的函数中，这是因为GoogleTest对在测试过程中未使用C++异常。如果一定有返回值需要处理，需要将```int foo(int x)```，转换为```void foo(int x,int *ret)```。或者不要使用fatal类的错误测试宏，改用```ADD_FAILURE_*```、```EXCEPT_*```。


## 值得关注
1. 大佬[Brendan Gregg's Homepage](https://www.brendangregg.com/index.html)。性能测试专家。参与开发了很多性能测试工具。而且其本人非常乐于在各种论坛分享，因此有很多教学视频资料可以在主页上找到。

## 参考
[《BPF 之巅：洞悉Linux系统和应用性能》读书笔记（四）火焰图 - Grissom的文章 - 知乎](https://zhuanlan.zhihu.com/p/671121116)

[GoogleTest官方Examples](https://github.com/google/googletest/tree/main/googletest/samples)

《性能之巅：洞悉系统、企业与云计算》，即《Systems Performance: Enterprise and the Cloud》