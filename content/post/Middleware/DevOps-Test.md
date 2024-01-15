---
title: "DevOps：软件测试篇"
date: 2024-01-11T14:24:57+08:00
categories:
- 计算机科学与技术
- DevOps
tags:
- DevOps
- 施工中
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/DevOps.jpg
draft: true
---
软件测试是DevOps的重要一环，本篇记录一些必备的测试知识。
<!--more-->
## 测试类型
- 单元测试
- 集成测试
- 黑盒测试
- 白盒测试
- 压力测试

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
    gtest_main
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

GoogleTest还有很多[高级](https://google.github.io/googletest/advanced.html)的功能，因为内容繁杂，建议查阅文档，这里简单列举一下功能和大致用法，
| 测试功能 | 实现方式 | 示例代码（如有）| 备注 |
| --- | --- | --- | --- |
| 显式的标记成功/失败 | 宏 | ```SUCCESS() / FAILED() ``` | 失败会立刻结束测试 |
| 添加一次错误样例 | 宏 | ```ADD_FAILURE()``` | 添加一个错误，还会继续运行 |
| 检测异常抛出 | 宏 | ```EXPECT_THROW(statement,exception_type)``` | statement可以是任意表达式 |
| 提供更好的测试错误信息 | 可调用对象 | ```testing::AssertionResult``` | 对于PRED类测试，可以包装待测试函数返回更好的错误信息，而不是只有简单的true/false |
| 浮点数比较 | 可调用对象 | ```testing::FLoatLE``` | 放到EXPECT_PRED/THAT等中使用 |
| 更多的字符串比较工具 | 可调用对象 | ```testing::HasSubstr``` | 用于EXPECT_THAT等 |
| 类型断言 | 类模板 | ```testing::StaticAssertType<T1,T2>``` | 判断类型是否相等，注意受模板限制，只有真正实例化才能触发判断，如果待实例化内容被优化了，这条测试无法获得判断结果 |
| 调过测试 | 宏 | ```GTEST_SKIP()``` | 可以放到Fixture类SetUp，或者TEST/TEST_F等任意GTest流程函数中 |
| 打印任意变量 | 可调用对象 | ```testing::PrintToString(x)``` | 返回字符串 |
| 自定义任意变量的打印 | 函数模板重载 | ```AbslStringfy(Sink &sink, YourType t)``` | 定义为友元函数，或者在同一命名空间 |
| 死亡测试 | 宏 | ```ASSERT_DEATH、EXPECT_EXIT``` | 在指定条件下，程序应当立即结束，死亡测试负责这种情况 |
| 额外的日志 | 函数 | ```testing::Test::RecordProperty(key,value)``` | 打印到日志 |
| 全局SetUp/TearDown | 类继承 | ```testing::Environment``` | 继承并重写Env的SetUp/TearDown |


一些限制
1. 会产生fatal类的错误测试宏（如```FAIL_*```、```ASSERT_*```），无法使用在非void返回值的函数中，这是因为GoogleTest对在测试过程中未使用C++异常。如果一定有返回值需要处理，需要将```int foo(int x)```，转换为```void foo(int x,int *ret)```。或者不要使用fatal类的错误测试宏，改用```ADD_FAILURE_*```、```EXCEPT_*```。


具体测试工具
| 工具类型 | 工具名 |
| --- | --- |
| 二元测试宏 | EXPECT_EQ、EXPECT_NE、EXPECT_STREQ |


## 参考
[《BPF 之巅：洞悉Linux系统和应用性能》读书笔记（四）火焰图 - Grissom的文章 - 知乎](https://zhuanlan.zhihu.com/p/671121116)
