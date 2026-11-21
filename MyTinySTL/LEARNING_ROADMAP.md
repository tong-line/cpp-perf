# MyTinySTL 自学路线图

## 学习目标

通过阅读、调试和模仿实现 MyTinySTL，掌握：

- STL 容器、迭代器、算法和分配器的核心设计
- C++11 模板、类型萃取、移动语义与异常安全
- `vector`、`list`、`deque` 等顺序容器的内部实现
- 红黑树、哈希表和关联容器的实现原理
- 泛型代码的性能分析、测试与调试方法

## 周期安排

- **每天学习时间**：6 小时
- **每周学习**：6 天
- **建议总周期**：8 周
- **预计总投入**：约 288 小时

| 当前基础 | 建议周期 |
|---|---|
| 有较好的 C++ 基础 | 6 周 |
| 熟悉 STL，但没有实现过容器 | 8 周 |
| C++ 或模板基础较弱 | 10～12 周 |

## 每日学习节奏

每天 6 小时建议安排为：

| 时间 | 内容 |
|---|---|
| 1 小时 | 补齐理论与数据结构知识 |
| 2 小时 | 阅读源码、调试并绘制结构图 |
| 2 小时 | 模仿实现、添加测试或修改源码 |
| 30 分钟 | 整理笔记、记录问题 |
| 30 分钟 | 回顾当天最难的知识点 |

每周留出一天进行复盘，完成随机测试、知识总结和下周准备。

## 第 1 周：C++11 与 STL 使用基础

### 学习内容

- 指针、引用、`const` 与对象生命周期
- RAII
- 左值、右值、移动构造与移动赋值
- 模板参数、模板特化和函数重载
- `auto`、`decltype`
- 迭代器分类
- 标准容器的复杂度和迭代器失效规则

### 重点文件

- [Test/test.cpp](./Test/test.cpp)
- [Test/test.h](./Test/test.h)
- [MyTinySTL/functional.h](./MyTinySTL/functional.h)
- [MyTinySTL/util.h](./MyTinySTL/util.h)

### 实践任务

- 用标准 STL 编写一个词频统计程序
- 分别使用 `vector`、`list`、`deque`、`map` 和 `unordered_map`
- 总结每种容器的插入、删除、查找和内存布局
- 编写计数类，观察拷贝、移动和析构次数

### 本周产出

- 一份 STL 容器复杂度表
- 一份迭代器失效规则表
- 能口头解释拷贝、移动和 RAII

## 第 2 周：分配器、迭代器与类型萃取

### 推荐阅读顺序

1. [MyTinySTL/type_traits.h](./MyTinySTL/type_traits.h)
2. [MyTinySTL/construct.h](./MyTinySTL/construct.h)
3. [MyTinySTL/allocator.h](./MyTinySTL/allocator.h)
4. [MyTinySTL/alloc.h](./MyTinySTL/alloc.h)
5. [MyTinySTL/uninitialized.h](./MyTinySTL/uninitialized.h)
6. [MyTinySTL/iterator.h](./MyTinySTL/iterator.h)
7. [MyTinySTL/memory.h](./MyTinySTL/memory.h)
8. [MyTinySTL/util.h](./MyTinySTL/util.h)

### 学习重点

- 内存分配与对象构造为什么要分离
- placement new
- allocator 的职责
- POD/trivial 类型优化
- 迭代器五种关联类型
- `iterator_traits`
- `move`、`forward`、`pair`
- 未初始化内存上的复制和填充

### 实践任务

- 实现一个最小 allocator
- 实现 `uninitialized_copy`
- 实现支持前进迭代器的简单迭代器
- 使用类型计数器验证对象构造与析构次数

### 验收标准

能画出从“申请内存”到“构造对象”再到“析构并释放内存”的完整流程。

## 第 3 周：`vector`

### 重点文件

- [MyTinySTL/vector.h](./MyTinySTL/vector.h)
- [Test/vector_test.h](./Test/vector_test.h)

### 学习重点

- 连续内存管理
- `size`、`capacity` 与缓冲区的关系
- 扩容策略
- `push_back` 的移动与复制路径
- `insert` 和 `erase`
- `reserve` 与 `resize`
- 迭代器失效
- 强异常安全与基本异常安全

### 跟踪建议

第一次不要通读整个文件，而是分别跟踪：

1. 构造函数
2. `push_back`
3. `insert`
4. `erase`
5. `reserve`
6. 析构函数

### 实践任务

从零实现 `MiniVector<T>`，逐步增加：

1. `size`、`capacity`、`empty`
2. `push_back`、`pop_back`、`operator[]`
3. `reserve`、`resize`
4. 迭代器
5. `insert`、`erase`
6. 拷贝和移动操作

### 验收标准

能画出一次扩容中旧缓冲区到新缓冲区的对象迁移、销毁和释放过程。

## 第 4 周：`list`、`deque` 与容器适配器

### 重点文件

- [MyTinySTL/list.h](./MyTinySTL/list.h)
- [MyTinySTL/deque.h](./MyTinySTL/deque.h)
- [MyTinySTL/stack.h](./MyTinySTL/stack.h)
- [MyTinySTL/queue.h](./MyTinySTL/queue.h)
- [MyTinySTL/heap_algo.h](./MyTinySTL/heap_algo.h)

### 学习重点

- 双向链表的哨兵节点
- 链表迭代器
- splice 类操作
- `deque` 的分段存储
- 缓冲区与中控数组
- 两端扩容
- `stack`、`queue` 的适配器设计
- `priority_queue` 与堆算法

### 实践任务

- 实现简化版 `MiniList<T>`
- 画出 `deque` 的中控块、缓冲区和元素关系
- 实现二叉堆的 `push_heap`、`pop_heap`
- 用计时程序比较 `deque` 与 `vector` 两端插入性能

### 验收标准

能解释为什么 `deque` 提供两端插入优势，但并不是真正连续存储。

## 第 5 周：泛型算法与字符串

### 重点文件

- [MyTinySTL/algobase.h](./MyTinySTL/algobase.h)
- [MyTinySTL/algo.h](./MyTinySTL/algo.h)
- [MyTinySTL/algorithm.h](./MyTinySTL/algorithm.h)
- [MyTinySTL/numeric.h](./MyTinySTL/numeric.h)
- [MyTinySTL/basic_string.h](./MyTinySTL/basic_string.h)

### 学习重点

- 泛型算法如何只依赖迭代器能力
- 复制、比较、填充和移动
- `rotate`、`sort`、二分查找
- 谓词与仿函数
- 字符串存储和扩容
- 字符串查找算法

### 实践任务

手写以下算法：

- `copy_if`
- `equal`
- `lower_bound`
- `upper_bound`
- `rotate`
- `accumulate`

之后实现一个简化版动态字符串，支持：

- `append`
- `reserve`
- `resize`
- `operator[]`
- `find`

### 验收标准

能通过迭代器类型解释算法为什么对复杂度和适用容器有不同限制。

## 第 6 周：红黑树与有序关联容器

这是最难的一周，建议投入完整的 36 小时。

### 重点文件

- [MyTinySTL/rb_tree.h](./MyTinySTL/rb_tree.h)
- [MyTinySTL/set.h](./MyTinySTL/set.h)
- [MyTinySTL/map.h](./MyTinySTL/map.h)
- [Test/map_test.h](./Test/map_test.h)
- [Test/set_test.h](./Test/set_test.h)

### 推荐阅读顺序

1. 红黑树节点和颜色定义
2. 左旋、右旋
3. 查找节点
4. 插入节点
5. 插入后的修复
6. 删除节点
7. 删除后的修复
8. 迭代器实现
9. `set` / `map` 的封装

### 学习重点

- 红黑树五条性质
- 旋转不会破坏什么性质
- 插入过程如何处理红红冲突
- 删除过程中的双黑问题
- `set`、`multiset`、`map`、`multimap` 的共用逻辑
- 节点中提取 key/value 的仿函数

### 实践任务

- 单独实现普通二叉搜索树
- 为二叉搜索树加入旋转
- 实现红黑树插入及插入修复
- 实现删除及删除修复
- 随机生成操作，与 `std::map` 逐步对照结果

### 验收标准

能手画红黑树插入和删除的关键修复分支，并检查每次操作后的红黑树性质。

## 第 7 周：哈希表与 `unordered_*`

### 重点文件

- [MyTinySTL/hashtable.h](./MyTinySTL/hashtable.h)
- [MyTinySTL/unordered_set.h](./MyTinySTL/unordered_set.h)
- [MyTinySTL/unordered_map.h](./MyTinySTL/unordered_map.h)
- [Test/unordered_map_test.h](./Test/unordered_map_test.h)
- [Test/unordered_set_test.h](./Test/unordered_set_test.h)

### 学习重点

- 开链法
- bucket 数量与素数策略
- 哈希函数
- 相等谓词
- 负载因子
- rehash
- 迭代器失效
- 哈希冲突下的性能退化

### 实践任务

实现 `MiniUnorderedSet<int>`：

1. 使用 vector 保存 bucket
2. 使用链表处理冲突
3. 实现 `insert`、`find`、`erase`
4. 实现 rehash
5. 测试全部碰撞和随机数据
6. 与 `std::unordered_set` 对照

### 验收标准

能解释 rehash 前后的节点迁移过程、为什么平均复杂度是常数，以及最坏情况为何可能退化。

## 第 8 周：复习、测试与毕业项目

### 复习任务

- 重新梳理所有容器的内存模型
- 总结迭代器失效条件
- 对比有序与无序关联容器
- 对比 `allocator` 和 `operator new/delete`
- 对照标准 STL 的接口和行为

### 毕业项目

从下面选择一个：

1. **MiniVector**
   - 连续内存
   - 扩容
   - 移动语义
   - 强异常安全测试

2. **MiniMap**
   - 红黑树插入和删除
   - 双向迭代器
   - 与 `std::map` 随机对照

3. **MiniUnorderedMap**
   - bucket
   - 哈希冲突处理
   - rehash
   - 负载因子控制

### 项目要求

- 保留清晰的公共接口
- 添加拷贝、移动、析构测试
- 添加随机数据测试
- 能说明每个主要操作的时间复杂度
- 绘制至少一张内部内存结构图
- 使用标准 STL 对照运行结果

## 学习方法

### 推荐的源码阅读顺序

每个功能按以下顺序学习：

1. 看测试代码，明确行为
2. 看公开接口，忽略内部实现
3. 画出内存结构
4. 跟踪一个典型调用路径
5. 单步调试
6. 手写简化版本
7. 增加边界情况
8. 与标准 STL 对照

不要一开始从文件第一行连续读到最后一行。

### 调试建议

建议为每个容器准备以下对象：

```cpp
class Tracked
{
public:
  Tracked();
  Tracked(const Tracked&);
  Tracked(Tracked&&);
  Tracked& operator=(const Tracked&);
  Tracked& operator=(Tracked&&);
  ~Tracked();
};
```

通过静态计数观察：

- 构造次数
- 拷贝次数
- 移动次数
- 析构次数
- 是否存在对象泄漏

### 笔记模板

每个组件至少记录：

1. 解决什么问题
2. 对外接口
3. 内部数据结构
4. 核心不变量
5. 典型调用路径
6. 异常安全保证
7. 迭代器失效条件
8. 时间复杂度与空间复杂度
9. 可以优化的地方
10. 当前仍不理解的问题

## 构建与测试

在仓库根目录执行：

```bash
cmake -S . -B build \
  -DCMAKE_BUILD_TYPE=Release \
  -DCMAKE_EXPORT_COMPILE_COMMANDS=ON

cmake --build build --parallel
./bin/stltest
```

项目要求 CMake 3.10 或更高版本，以兼容当前 CMake 版本。

## 每周验收清单

- [ ] 能独立说明本周主要数据结构的内存布局
- [ ] 至少跟踪两个公开 API 的完整调用链
- [ ] 至少实现一个简化版组件
- [ ] 至少增加一个边界测试
- [ ] 通过完整测试套件
- [ ] 完成本周知识总结
- [ ] 列出下周需要解决的问题

## 最终完成标准

完成路线图后，应当能够：

- 不看提示画出 `vector`、`deque`、红黑树和哈希表的结构
- 解释各容器的扩容、迭代器失效和复杂度
- 写出一个可用的泛型顺序容器
- 理解 allocator 与对象构造的边界
- 理解 STL 算法、迭代器和容器的解耦方式
- 独立阅读其他模板库中的容器实现
