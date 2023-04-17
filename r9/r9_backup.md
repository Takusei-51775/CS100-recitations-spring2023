
---

# `static` 成员

---

## 回忆变量上的 `static` 关键词

`static` 变量只在声明语句第一次被执行时初始化，后续遇到声明语句将不会重复初始化。生命周期直到程序结束，但只在声明所在的作用域可以访问。

- 函数调用次数计数器（`static`变量在多次调用中保留状态）
```cpp
int how_many_times_called() {
  static std::size_t times_called = 0;
  return ++times_called;
}
// ...
std::cout << how_many_times_called() << std::endl;
std::cout << how_many_times_called() << std::endl;
std::cout << times_called << std::endl; // Error: times_called undefined.
```
---
## `static` 成员变量 

```cpp
class A {
public:
  static int something;
};
```

类似的想法：这个语句应该不会重复创建 `something`？
具体一些：在它所在的作用域（`class A` 里）只会创建一个 `something`。它不属于哪个特定的 `A` 类型对象，而是属于整个 `A` 类。
- 甚至就算还没有任何 `A` 类型对象被定义，它也已经存在了！
  
它的作用域属于 `class A`，因此：
- 在 `A` 类的成员函数中可以直接以 `something` 来访问它。
- 在 `A` 类以外，用 `A::something` 来访问它（因为 `public` 而不是 `private`）。
- 如果你有任何 `A` 类型的对象，比如叫`a1`和`a2`，那么也可以用 `a1.something` 或 `a2.something` 来访问 `A::something`。它们都是**同一个**变量。

---
## 特别注意：初始化 `static` 成员变量 
```cpp
class A {
public:
  static int something = 0; // Error: 
  // Cannot initialize a non-const static member
};
```
类内只允许声明，而 `static` 成员变量应当属于静态内存，应在全局作用域内初始化，而不能在 `A` 类中、`A` 的构造函数中、或者任何函数（比如 `main`）中被第一次初始化。

<div style="display: grid; grid-template-columns: 1fr 1fr;">
  <div>
(before C++17) 

在 `A` 类里声明，需要全局显式定义：

```cpp
class A {
public:
  static int something;
};
int A::something = 0; // No "static"!
```
  </div>
  <div>

(since C++17) 将 `static` 成员变量声明为 `inline` 即可在类内定义：
```cpp
class A {
public:
  inline static int something = 0; 
};
```

  </div>
</div>

---

## 示例：`static` 成员变量
<div style="display: grid; grid-template-columns: 1fr 1fr;">
  <div>

```cpp
class A {
public:
  inline static int something = 0;
  void increment() { something++; }
};

int main() {
  std::cout << A::something << std::endl;
  A a1; A a2;
  a1.increment();
  a2.increment();
  std::cout << A::something << std::endl;
  a1.something++;
  std::cout << A::something << std::endl;
  std::cout << a2.something << std::endl;
```
  </div>
  <div>
输出：

```cpp
0
2
3
3
```
  </div>
</div>

---

## `const static` 成员变量

我们经常会需要各种各样的常量：要不要给 `Dynarray` 一个默认 `DEFAULT_SIZE`？要不要给 `Student` 一个 `MIN_ENTRANCE_YEAR`？
这些常量只在对应的类内有意义！用宏定义或全局常量的方式会导致它们堆在一起，并且对外暴露了无意义的数据。

所有同一个类对象的公共数据→在类里只需要存在一遍的数据→静态成员！
```cpp 
class A {
  const static int CONSTANT_FOR_THE_CLASS = 100;
};
```
`const static` 变量可以在类内初始化。它其实是implicitly `inline` 的，不需要加 `inline` 限定词。

---

## `static` 成员函数
```cpp
class A {
public:
  static void doSomething();
};
```
理解了静态成员变量后，静态成员函数就好理解了：
- 属于整个 `A` 类，而不属于哪一个特定对象
- 可以使用 `A::doSomething()` 在类外无需对象地调用
- 也可以对实际对象调用**同一个**函数 `a1.doSomething()`, `a2.doSomething()`。

---

## `static` 成员函数
```cpp
class A {
public:
  static void doSomething();
};
```
额外的特性：
- 由于它不属于某个特定对象，在函数内便不可用 `this`。
- 同样，它不可以访问任何属于对象而不是 `A` 类整体的成员。也就是说，它不能访问非 `static` 的成员。
---

## `static` 有什么用呢？

假如你在开发访问手机摄像头、麦克风的接口。腾讯会议、微信视频、微软Skype纷纷找上你，希望用你的代码实现视频通话。
如果允许它们各自创建自己的对象来管理摄像头、麦克风，它们将无法知道有没有被其它软件占用、是否已经在录制状态……
- 核心问题：你的手机里的硬件只有一份，无法被复制，不能像在代码里一样随手创建新的对象。
- 需要它们被仅一个对象来管理。

---

## Singleton （单例）设计模式

一种设计模式：
- 只允许该类的对象被创建一次。
  - 我们需要禁止它随意被创建！
- 需要访问这个对象时，访问的都是同一个。
  - 我们需要让它变成一个 `static` 变量！


---

## Meyers' Singleton

Scott Meyers （Effective C++ 的作者 ） 提出了 C++ 中最简洁、完善的实现方式 Meyers' Singleton:

```cpp
class Singleton {
public:
  Singleton(const Singleton&) = delete;
  Singleton& operator=(const Singleton&) = delete;
  static Singleton& instance() {
    static Singleton inst;
    return inst;
  }
  // other members...
private:
  Singleton() {}
  ~Singleton() {}
  // other members...
};
```

---
## Meyers' Singleton

显式将 copy constructor 和 copy assignment operator 定义为了 `delete`，即不可拷贝
将默认构造函数定义为 `private`，使得它无法被在类外随意创建。

`instance()` 是一个 `static` 成员函数，这意味着没有该类的具体对象时也可以调用它。当它被调用时，会发生的事情是：
- 在第一次访问时，静态对象 `inst` 被第一次创建并初始化；之后的访问中它是同一个对象，可以保留这个对象的状态（比如其他成员变量的值）。
- 返回这个“仅一个的对象”的引用。
  
当你为它定义其他的成员时，便可以使用 `Singleton::instance().doSomething()`：先获得这个单例对象，再调用成员。


---



# 初识类型推导：`auto` 和 `decltype`