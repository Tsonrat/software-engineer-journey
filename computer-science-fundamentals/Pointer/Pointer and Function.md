# Pointer and Function｜指標與函式

本章延續 Pointer Basics，將 Pointer 實際應用到 Function。

主要理解：

```text
Pass by Value
→ Function 拿到值的副本

Pass by Pointer
→ Function 拿到外部變數的位址

Dereference
→ Function 可以透過位址修改外部變數
```

Pointer 在 Function 中的重要用途之一，就是：

> **讓 Function 可以存取並修改 Function 外部的資料。**

---

## 1. Pass by Value｜傳值

先看一般 Function：

```cpp
void add(int x) {
    x = x + 10;
}

int main() {
    int a = 20;

    add(a);

    cout << a;
}
```

最後：

```text
a = 20
```

雖然：

```cpp
add(a);
```

將 `a` 傳入 Function，但 `add()` 接收到的不是原本的 `a`。

而是：

> **將 `a` 的值複製一份給 Function Parameter `x`。**

可以理解成：

```text
main()

a = 20
│
│ 複製值
▼
add()

x = 20
```

因此：

```cpp
x = x + 10;
```

實際修改的是：

```text
Function 裡面的 x
```

變成：

```text
main()          add()

a = 20          x = 30
```

兩者是不同的變數。

所以 Function 結束後：

```cpp
cout << a;
```

仍然得到：

```text
20
```

---

## 2. Same Variable Name Does Not Mean Same Variable｜同名不代表同一個變數

即使 Function Parameter 也叫 `a`：

```cpp
void add(int a) {
    a = a + 10;
}

int main() {
    int a = 20;

    add(a);
}
```

兩個 `a` 仍然不是同一個變數。

可以概念化成：

```text
main() 的 a

位址：0x100
值：20


add() 的 a

位址：0x200
值：20
```

呼叫：

```cpp
add(a);
```

只是：

```text
main() a 的值 20
        │
        │ copy
        ▼
Function a = 20
```

所以：

```cpp
void add(int a) {
    a = 50;
}
```

修改的是：

```text
0x200 → 50
```

不是：

```text
0x100 → 20
```

因此外面的 `a` 不會改變。

---

## 3. Parameter Name Is Independent｜Parameter 名稱是獨立的

Function Parameter 不需要和外部變數使用相同名稱。

例如：

```cpp
void add(int number) {
    number = number + 10;
}

int main() {
    int a = 20;

    add(a);

    cout << a;
}
```

這裡：

```cpp
add(a);
```

可以理解成：

```text
a 的值
20
│
│ copy
▼
number = 20
```

所以：

```cpp
number = number + 10;
```

只會讓：

```text
number = 30
```

外面的：

```text
a = 20
```

仍然不變。

因此：

> **決定是不是同一份資料的不是變數名稱，而是資料如何傳入 Function。**

---

# 4. Pass by Pointer｜透過 Pointer 傳入

如果希望 Function 可以修改外部的變數，就可以將變數的 **位址** 傳入 Function。

例如：

```cpp
void add(int* p) {
    *p = *p + 10;
}

int main() {
    int a = 20;

    add(&a);

    cout << a;
}
```

這次結果：

```text
a = 30
```

---

## 5. `add(&a)` 到底傳了什麼？

首先：

```cpp
&a
```

代表：

> `a` 的記憶體位址。

假設：

```text
a = 20
&a = 0x100
```

呼叫：

```cpp
add(&a);
```

就是將：

```text
0x100
```

傳給 Function。

Function：

```cpp
void add(int* p)
```

使用 Pointer `p` 接收這個位址。

因此：

```text
p = 0x100
```

關係：

```text
main()                         add()

0x100                          Pointer p
┌──────────┐                   ┌───────────┐
│ a = 20   │ ◀──────────────── │ p = 0x100 │
└──────────┘                   └───────────┘
```

所以：

```cpp
*p
```

就是：

```text
前往 0x100
↓
找到 a
↓
取得 a 的值
↓
20
```

---

# 6. Modify External Variable｜修改 Function 外部變數

Function：

```cpp
void add(int* p) {
    *p = *p + 10;
}
```

假設：

```text
p = 0x100
```

而：

```text
0x100
```

就是 `a` 的位置。

所以：

```cpp
*p = *p + 10;
```

可以一步一步理解成：

```text
*p
↓
取得 a 的值
↓
20


*p + 10
↓
30


*p = 30
↓
將 a 所在位置的值修改成 30
```

因此概念上相當於：

```cpp
a = a + 10;
```

最後：

```text
a = 30
```

---

# 7. Pointer Parameter Is Still Another Variable｜Pointer Parameter 自己仍然是另一個變數

這裡有一個很重要的觀念。

假設：

```cpp
void test(int* p) {
}

int main() {
    int a = 10;

    test(&a);
}
```

`p` 和 `a` 並不是同一個變數。

假設：

```text
&a = 0x100
&p = 0x200
```

那麼：

```text
a 自己位於
0x100

p 自己位於
0x200

但是：

p 保存的值 = 0x100
```

所以：

```text
&p != &a
```

但：

```text
p == &a
```

記憶體關係：

```text
&p = 0x200                   &a = 0x100

┌────────────┐              ┌──────────┐
│ p = 0x100  │ ───────────→ │ a = 10   │
└────────────┘              └──────────┘
```

因此：

> **Pointer Parameter 自己是另一個變數，但它保存了外部變數的位址。**

這也是 Function 能透過 Pointer 找回外部變數的原因。

---

# 8. Pass by Value vs Pass by Pointer

兩種方式可以直接比較。

## Pass by Value

```cpp
void change(int x) {
    x = 50;
}

int main() {
    int a = 10;

    change(a);
}
```

關係：

```text
a = 10
│
│ copy
▼
x = 10

x = 50
```

最後：

```text
a = 10
```

---

## Pass by Pointer

```cpp
void change(int* p) {
    *p = 50;
}

int main() {
    int a = 10;

    change(&a);
}
```

關係：

```text
p
│
│ 保存 &a
▼
a = 10
```

執行：

```cpp
*p = 50;
```

最後：

```text
a = 50
```

---

## Comparison

| Function Parameter | 傳入內容 | 是否能修改外部 `a` |
| --- | --- | --- |
| `int x` | `a` 的值的副本 | No |
| `int* p` | `a` 的位址 | Yes，透過 `*p` |

核心差異：

```text
Pass by Value

a
│
│ 複製值
▼
x

a 和 x 是兩份資料


Pass by Pointer

p
│
│ 保存 a 的位址
▼
a

*p 可以操作原本的 a
```

---

# 9. Swap Example｜交換兩個變數

Pointer 一個經典的實際用途，就是讓 Function 修改多個外部變數。

例如交換：

```text
a = 10
b = 20
```

希望最後變成：

```text
a = 20
b = 10
```

可以寫：

```cpp
void swap(int* x, int* y) {

    int temp = *x;

    *x = *y;

    *y = temp;
}
```

呼叫：

```cpp
int main() {

    int a = 10;
    int b = 20;

    swap(&a, &b);

    cout << a << endl;
    cout << b << endl;
}
```

---

## 10. Swap Step by Step｜逐步理解 Swap

一開始：

```text
a = 10
b = 20
```

呼叫：

```cpp
swap(&a, &b);
```

因此：

```text
x → a
y → b
```

也就是：

```text
*x = 10
*y = 20
```

### Step 1

```cpp
int temp = *x;
```

取得 `x` 指向的值：

```text
*x = 10
```

所以：

```text
temp = 10

a = 10
b = 20
```

### Step 2

```cpp
*x = *y;
```

因為：

```text
*y = 20
```

所以將 `20` 寫入 `x` 指向的位置。

而 `x` 指向 `a`：

```text
temp = 10

a = 20
b = 20
```

### Step 3

```cpp
*y = temp;
```

將：

```text
temp = 10
```

寫入 `y` 指向的位置。

而 `y` 指向 `b`：

```text
temp = 10

a = 20
b = 10
```

最後：

```text
a = 20
b = 10
```

完成交換。

---

# 11. Why Use Pointer in Functions?｜為什麼 Function 需要 Pointer？

如果 Function 只是需要資料進行計算，不需要修改原本資料：

```cpp
void calculate(int x)
```

使用 Pass by Value 即可。

Function 可以自由修改自己的 `x`：

```cpp
x = x + 10;
x = x * 2;
```

但不會影響外部變數。

關係：

```text
外部資料
↓
複製
↓
Function 自己處理
```

但如果 Function 的目的就是修改外部資料，例如：

```text
修改數值
交換資料
操作 Array
修改 Struct
操作資料結構
```

Pointer 就可以讓 Function 取得原本資料的位置。

關係：

```text
外部資料
    ▲
    │
Pointer
    │
Function
```

所以 Pointer 的價值不是「故意把簡單事情變複雜」。

而是：

> **當 Function 需要操作原本那份資料時，可以透過記憶體位址找到並修改它。**

---

# Java Comparison｜Java 對照

Java Function Parameter 的觀念和 C++ 有一個非常重要的差異。

首先要記住：

> **Java 一律是 Pass by Value。**

但是 Java 傳入 Object 時，複製的「值」本身是一個 Object Reference。

這也是 Java 初學時很容易誤以為 Java 有 Pass by Reference 的原因。

---

## Java Primitive Parameter

例如：

```java
static void change(int x) {
    x = 50;
}

public static void main(String[] args) {

    int a = 10;

    change(a);

    System.out.println(a);
}
```

結果：

```text
10
```

因為：

```text
a = 10
│
│ copy value
▼
x = 10
```

Function 修改：

```java
x = 50;
```

只修改自己的 `x`。

所以：

```text
main()

a = 10
```

不會改變。

這和 C++：

```cpp
void change(int x)
```

的 Pass by Value 很接近。

---

## Java Object Parameter

Object 則比較特別。

例如：

```java
static void change(ProjectEntity project) {
    project.setProjectName("TEST");
}

public static void main(String[] args) {

    ProjectEntity project = new ProjectEntity();

    change(project);
}
```

Java 仍然是 Pass by Value。

但是這次複製的是：

```text
Object Reference 的值
```

可以概念化成：

```text
main()

project
   │
   │ Reference
   ▼
┌─────────────────┐
│ ProjectEntity   │
└─────────────────┘
   ▲
   │
   │ 複製 Reference 的值
   │
Function parameter
```

因此 Function Parameter 和外面的 `project`：

> 是兩個不同的 Reference Variable，但它們 Reference 到同一個 Object。

可以理解成：

```text
main project ─────┐
                  ▼
           ┌───────────────┐
           │ ProjectEntity │
           └───────────────┘
                  ▲
                  │
function project ─┘
```

所以：

```java
project.setProjectName("TEST");
```

修改的是共同 Reference 的 Object。

外面自然也會看到：

```text
projectName = TEST
```

---

## Important Difference｜重要差異

Java：

```java
static void change(ProjectEntity project) {
    project = new ProjectEntity();
}
```

這並不會讓外面的 `project` 改成新的 Object。

因為 Function Parameter 本身仍然只是 Reference Value 的副本。

可以理解成：

```text
一開始：

main project ───────→ Object A
                       ↑
function project ──────┘


執行：

project = new ProjectEntity();


變成：

main project ───────→ Object A

function project ───→ Object B
```

外面的 Reference 沒有被修改。

這證明：

> **Java 並不是 Pass by Reference，而是 Pass by Value。**

只是 Object 傳遞時：

> **被複製的 Value 是 Object Reference。**

---

# C++ and Java Comparison｜C++ 與 Java 對照

可以先建立以下關係：

```text
C++

void test(int x)
→ 複製 int 的值
→ 修改 x 不影響外部


void test(int* p)
→ 複製外部變數的位址到 Pointer
→ *p 可以修改外部資料


Java

void test(int x)
→ 複製 primitive value
→ 修改 x 不影響外部


void test(ProjectEntity project)
→ 複製 Object Reference 的值
→ 可以透過 Reference 修改同一個 Object
→ 但不能重新指定外面的 Reference
```

簡化比較：

| Situation | C++ | Java |
| --- | --- | --- |
| Primitive Parameter | Pass by Value | Pass by Value |
| Pointer Parameter | `int* p` | 無直接對應 |
| 修改外部 Primitive | Pointer / Reference | 無法直接透過一般 Parameter |
| Object Parameter | Pointer / Reference / Value 等方式 | Object Reference Value |
| Parameter Passing | Value / Pointer / Reference 等語法 | Pass by Value |

---

# Summary｜整理

Function Parameter：

```cpp
void test(int x)
```

代表：

```text
傳入值的副本
↓
Function 修改自己的 x
↓
不影響外部變數
```

Pointer Parameter：

```cpp
void test(int* p)
```

呼叫：

```cpp
test(&a);
```

代表：

```text
取得 a 的位址
↓
傳給 Pointer p
↓
p 保存 a 的位址
↓
*p 找到 a
↓
可以修改 a
```

因此：

```text
int x
→ Value Copy


int* p
→ Address Copy
→ *p 可以找到原本資料
```

最重要的核心概念：

> **Pass by Value 是讓 Function 拿到資料的副本；Pointer 則是把原本資料的位置告訴 Function，讓 Function 可以透過 Dereference 操作原本的資料。**

Java 則需要另外記住：

> **Java 永遠是 Pass by Value。Primitive 傳遞的是 Primitive Value；Object 傳遞的是 Reference Value 的副本。**
