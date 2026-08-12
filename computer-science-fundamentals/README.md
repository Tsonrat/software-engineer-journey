# Computer Science Fundamentals

以 **C++ 作為主要學習語言**，從較底層的程式設計概念開始，逐步學習 **Pointer、Memory、Struct、Data Structure 與 Algorithm**。

學習過程除了理解語法，更重要的是理解：

* 資料在記憶體中如何儲存
* 程式如何存取與操作資料
* 不同資料結構如何組織資料
* 不同演算法如何處理與搜尋資料
* 資料結構與演算法對時間、空間效率的影響

在理解底層概念後，會再使用 **Java** 進行對照與實作，將 Computer Science 基礎與目前實際使用的 Java 開發經驗結合。

---

## Learning Approach

每個主題主要按照以下方式學習：

```text
Theory
  ↓
理解概念與存在的原因

C++ Example
  ↓
觀察較底層的 Memory / Pointer / Data Structure

Java Comparison
  ↓
理解 Java 如何封裝或實作相同概念

Practice
  ↓
透過程式與題目實際應用
```

重點不是單純記住語法，而是理解：

> Why it works and how it works.

---

## Learning Path

目前預計按照以下順序學習：

```text
C++ Fundamentals
        ↓
Pointer
        ↓
Reference
        ↓
Pointer + Function
        ↓
Pointer + Array
        ↓
Struct
        ↓
Data Structures
        ↓
Algorithms
```

---

## Repository Structure

```text
computer-science-fundamentals/
│
├── README.md
│
├── 01-Pointer/
│   ├── 01-Pointer-Basics.md
│   ├── 02-Pointer-and-Function.md
│   ├── 03-Reference.md
│   └── 04-Pointer-and-Array.md
│
├── 02-Struct/
│
├── 03-Data-Structures/
│
└── 04-Algorithms/
```

Repository 內容會隨學習進度逐步增加。

---

## Topics

### 01. Pointer

從 C++ Pointer 開始理解變數、記憶體位址與資料存取方式。

主要內容：

* Memory Address
* `&` Address Operator
* `*` Dereference Operator
* Pointer Variable
* Pointer to Pointer
* Pointer and Function
* Pass by Value
* Pass by Pointer
* Reference
* Pass by Reference
* Pointer and Array
* Pointer Arithmetic

---

### 02. Struct

使用 C++ Struct 理解如何將多個資料組合成一個資料結構，並進一步學習 Struct 與 Pointer 的關係。

---

### 03. Data Structures

在理解 Pointer 與 Struct 後，開始學習常見資料結構。

預計包含：

* Array
* Linked List
* Stack
* Queue
* Hash Table
* Tree
* Graph

除了理解資料結構的使用方式，也會嘗試理解其底層資料如何組織。

---

### 04. Algorithms

學習如何使用不同方法處理、搜尋與操作資料。

預計包含：

* Searching
* Sorting
* Binary Search
* Recursion
* DFS / BFS
* Time Complexity
* Space Complexity

後續再逐步加入其他演算法主題。

---

## C++ and Java

本 Repository 主要使用 **C++ 理解底層概念**，再使用 **Java 進行對照**。

例如：

```cpp
// C++

struct Node {
    int value;
    Node* next;
};
```

Java：

```java
class Node {
    int value;
    Node next;
}
```

透過兩種語言的比較，理解：

```text
C++ Pointer / Memory
        ↓
理解底層資料關係
        ↓
Java Reference / Collections
        ↓
實際 Java 開發
```

最終目標不是只學會 C++ 或 Java 語法，而是建立可以跨程式語言使用的 **Computer Science Fundamentals**。
