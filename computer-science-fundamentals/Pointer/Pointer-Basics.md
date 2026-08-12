# Pointer Basics｜指標基礎

本章以 C++ 為基礎，理解 **Variable、Memory Address、Pointer 與 Dereference** 之間的關係。

學習 Pointer 時，最重要的不是背 `*`、`&` 的語法，而是先分清楚：

```text
值（Value）
位址（Address）
Pointer 保存的位址
Pointer 指向的資料
```

---

## 1. Variable and Memory Address｜變數與記憶體位址

在 C++ 中，每個變數除了保存資料之外，也會存在記憶體中的某個位置。

例如：

```cpp
int a = 10;
```

可以概念化成：

```text
變數名稱：a
值：      10
位址：    0x100   // 假設位址
```

使用：

```cpp
cout << a;
```

取得的是 `a` 的值：

```text
10
```

使用：

```cpp
cout << &a;
```

取得的是 `a` 的記憶體位址：

```text
0x100
```

因此可以先記成：

```text
a   → a 的值
&a  → a 的位址
```

---

## 2. Pointer｜指標

Pointer 是一種用來保存 **記憶體位址** 的變數。

例如：

```cpp
int a = 10;
int* p = &a;
```

其中：

```cpp
int* p
```

代表宣告一個可以指向 `int` 的 Pointer `p`。

而：

```cpp
&a
```

代表取得 `a` 的記憶體位址。

所以：

```cpp
int* p = &a;
```

可以理解成：

> 建立 Pointer `p`，並將 `a` 的記憶體位址保存到 `p`。

假設：

```text
&a = 0x100
```

那麼：

```text
a  = 10
&a = 0x100

p  = 0x100
```

關係可以表示成：

```text
p = 0x100
│
▼
0x100
┌──────────┐
│ a = 10   │
└──────────┘
```

因此：

```text
p == &a
```

這裡比較的是：

> `p` 保存的位址，就是 `a` 的位址。

---

## 3. Dereference Operator `*`｜解參考

Pointer `p` 保存的是位址。

如果想要取得該位址裡面的資料，可以使用：

```cpp
*p
```

例如：

```cpp
int a = 10;
int* p = &a;

cout << p << endl;
cout << *p << endl;
```

假設：

```text
&a = 0x100
```

那麼：

```text
p  = 0x100
*p = 10
```

`*p` 的思考方式：

```text
p
↓
0x100
↓
前往 0x100
↓
取得裡面的資料
↓
10
```

因此：

```text
p   → Pointer 保存的位址
*p  → Pointer 指向位置裡面的值
```

---

## 4. Modify Value Through Pointer｜透過 Pointer 修改資料

Pointer 不只能讀取資料，也可以修改所指向位置的資料。

例如：

```cpp
int a = 10;
int* p = &a;

*p = 50;
```

因為：

```text
p
→ a 的位址

*p
→ a 的值
```

所以：

```cpp
*p = 50;
```

可以理解成：

```text
透過 p 找到 a 的位址
        ↓
取得該位置的資料
        ↓
修改成 50
```

因此最後：

```text
a = 50
```

關係：

```text
p
│
▼
┌──────────┐
│ a = 50   │
└──────────┘
```

也就是：

> 修改 `*p`，實際上是在修改 `p` 所指向的變數。

---

## 5. Pointer Has Its Own Address｜Pointer 自己也有位址

Pointer 本身也是一個變數。

因此 `p` 除了可以保存 `a` 的位址之外，`p` 自己也會存在某個記憶體位置。

例如：

```cpp
int a = 10;
int* p = &a;
```

假設：

```text
&a = 0x100
&p = 0x200
```

那麼：

```text
a  = 10
&a = 0x100

p  = 0x100
&p = 0x200

*p = 10
```

記憶體關係：

```text
0x200                         0x100

┌─────────────┐              ┌──────────┐
│ p = 0x100   │ ───────────→ │ a = 10   │
└─────────────┘              └──────────┘
```

所以必須區分：

```text
p
→ p 裡面保存的值
→ a 的位址

&p
→ Pointer p 自己的位址

*p
→ p 指向位置裡面的值
→ a 的值
```

Pointer 並不是和 `a` 存在同一個位置。

比較準確的理解是：

> **Pointer 自己住在另一個記憶體位置，但它保存了目標變數的位置。**

---

## 6. `*` 在不同位置的意義

`*` 出現在「宣告」與「運算式」時，意義不同。

### 宣告 Pointer

```cpp
int* p;
```

這裡的 `*` 表示：

> `p` 是一個指向 `int` 的 Pointer。

### Dereference

```cpp
*p
```

這裡的 `*` 表示：

> 前往 `p` 保存的位址，取得該位置的資料。

因此：

```text
int* p
     ↑
宣告 Pointer


*p
↑
Dereference
```

不要只看到 `*` 就判斷意思，需要看它出現在什麼語法位置。

---

## 7. `&` 在不同位置的意義

`&` 也會因為使用位置不同而有不同意義。

### Address Operator

例如：

```cpp
int a = 10;
int* p = &a;
```

這裡：

```cpp
&a
```

表示：

> 取得 `a` 的記憶體位址。

因此：

```text
a   → 值
&a  → 位址
```

### Reference Declaration

之後也會看到：

```cpp
int& r = a;
```

這裡的 `&` 不是「取得位址」，而是表示：

> 宣告 `r` 為 Reference。

因此：

```text
&a       → 取得 a 的位址

int& r   → 宣告 Reference
```

同樣需要依照 `&` 出現的位置判斷意思。

---

## 8. Pointer to Pointer｜二級指標

既然 Pointer 自己也是變數，而且 Pointer 自己也有位址，就可以再建立另一個 Pointer 指向它。

例如：

```cpp
int a = 10;

int* p = &a;
int** pp = &p;
```

關係：

```text
pp
│
▼
p
│
▼
a
```

假設：

```text
a  位於 0x100
p  位於 0x200
pp 位於 0x300
```

那麼：

```text
a   = 10
&a  = 0x100

p   = 0x100
&p  = 0x200

pp  = 0x200
&pp = 0x300
```

記憶體關係：

```text
0x300              0x200              0x100

┌───────────┐      ┌───────────┐      ┌──────────┐
│pp = 0x200 │ ───→ │p = 0x100  │ ───→ │ a = 10   │
└───────────┘      └───────────┘      └──────────┘
```

也就是：

```text
pp → p → a
```

---

## 9. `pp`、`*pp`、`**pp`

假設：

```cpp
int a = 10;
int* p = &a;
int** pp = &p;
```

那麼：

```text
pp
→ p 的位址

*pp
→ p 保存的值
→ a 的位址

**pp
→ a 的值
```

可以一層一層理解：

```text
pp
↓
&p

*pp
↓
p
↓
&a

**pp
↓
*p
↓
a
```

因此：

```cpp
**pp = 50;
```

最後修改的是：

```text
a = 50
```

因為：

```text
pp
→ 找到 p

*pp
→ 取得 p 保存的 a 位址

**pp
→ 取得並修改 a 的值
```

---

## 10. Pointer Mental Model｜Pointer 思考方式

Pointer 最重要的是分清楚：

```text
變數的值
變數的位址
Pointer 保存的位址
Pointer 自己的位址
Pointer 指向的值
```

看到：

```cpp
int a = 10;
int* p = &a;
```

可以按照下面的順序思考：

```text
a
→ a 的值
→ 10


&a
→ a 的位址
→ 假設 0x100


p
→ p 保存的值
→ 0x100
→ 也就是 a 的位址


*p
→ 前往 p 保存的 0x100
→ 取得裡面的資料
→ 10


&p
→ Pointer p 自己的位址
```

因此最重要的一組關係：

```text
a     → a 的值

&a    → a 的位址

p     → p 保存的位址

*p    → p 指向的值

&p    → p 自己的位址
```

---

# Summary｜整理

假設：

```cpp
int a = 10;
int* p = &a;
```

可以整理成：

| Expression | Meaning |
| --- | --- |
| `a` | `a` 的值 |
| `&a` | `a` 的位址 |
| `p` | `p` 保存的位址，也就是 `&a` |
| `*p` | `p` 指向位置的值，也就是 `a` |
| `&p` | Pointer `p` 自己的位址 |

如果再建立：

```cpp
int** pp = &p;
```

則：

| Expression | Meaning |
| --- | --- |
| `pp` | `p` 的位址 |
| `*pp` | `p` 保存的值，也就是 `a` 的位址 |
| `**pp` | `a` 的值 |

核心概念：

> **Pointer 自己是一個變數，它存在自己的記憶體位置；Pointer 保存的資料則是另一個物件的記憶體位址。**

---

# Java Comparison｜Java 對照

Java 不提供 C++ 這種可以直接操作記憶體位址的 Pointer。

C++ 可以：

```cpp
int a = 10;
int* p = &a;

*p = 50;
```

直接處理：

```text
&a
p
*p
&p
```

也就是可以明確操作記憶體位址與 Dereference。

Java 則沒有對應的 Pointer 語法：

```text
&a
*p
&p
Pointer Arithmetic
```

但是 Java 的 **Object Reference** 與 Pointer 有部分相似的概念。

---

## Java Primitive Type

先看 Primitive：

```java
int a = 10;
int b = a;

b = 50;
```

結果：

```text
a = 10
b = 50
```

因為：

```java
int b = a;
```

是將 `a` 的值複製給 `b`。

可以理解成：

```text
a
┌──────┐
│  10  │
└──────┘

    複製值
      ↓

b
┌──────┐
│  10  │
└──────┘
```

兩者是不同的變數。

因此修改：

```java
b = 50;
```

不會影響 `a`。

---

## Java Object Reference

Object 的情況不同。

例如：

```java
ProjectEntity a = new ProjectEntity();
ProjectEntity b = a;
```

這裡並沒有建立第二個 `ProjectEntity` Object。

而是：

```text
a ─────┐
       │
       ▼
┌─────────────────┐
│  ProjectEntity  │
└─────────────────┘
       ▲
       │
b ─────┘
```

`a` 和 `b` 都 Reference 到同一個 Object。

因此：

```java
b.setProjectName("TEST");
```

修改的是它們共同 Reference 的 Object。

所以：

```java
System.out.println(a.getProjectName());
```

也會得到：

```text
TEST
```

---

## C++ Pointer vs Java Reference

可以先建立以下概念：

```text
C++ Pointer
→ 保存記憶體位址
→ 可以 Dereference
→ 可以直接處理 Pointer / Address
→ 可以進行 Pointer Arithmetic


Java Object Reference
→ Reference 到 Object
→ 多個 Reference 可以指向同一個 Object
→ 可以透過 Reference 修改 Object
→ 不提供直接記憶體位址操作


Java Primitive
→ Assignment 時複製值
→ 修改副本不影響原本變數
```

簡化比較：

| Concept | C++ | Java |
| --- | --- | --- |
| Primitive Value Copy | `int b = a` | `int b = a` |
| Pointer | `int* p` | 不提供 |
| Address Operator | `&a` | 不提供 |
| Dereference | `*p` | 不提供 |
| Object Reference | Reference / Pointer 等相關機制 | Object Reference |
| Direct Memory Address Operation | 可以 | 不提供 |

因此，學習 C++ Pointer 的目的之一，是直接理解：

```text
資料
↓
記憶體位置
↓
Reference / Pointer 關係
↓
不同變數如何存取同一份資料
```

再回頭看 Java：

```java
ProjectEntity project = new ProjectEntity();
```

就不只是知道「這是建立 Object」，也能進一步理解：

> `project` 是一個 Reference variable，它參考到建立出來的 `ProjectEntity` Object。

C++ 讓記憶體與 Pointer 的關係直接呈現在語法中；Java 則將大部分底層記憶體操作封裝起來，主要透過 Object Reference 操作物件。
