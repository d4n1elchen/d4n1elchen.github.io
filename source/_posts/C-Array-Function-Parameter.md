---
title: C Array Function Parameter 全攻略
date: 2020-05-30 22:03:39
tags: [c, c++]
categories: c
---

這是嘗試把 C 的 array function parameter 搞懂的一篇筆記。

## 前言

從很久以前剛開始學 C 的時候就對如何讓 function 接收 array parameter 有很大的疑問，此外 C++ 的 array paramter 又有不太一樣的行為，因此後來學 C++ 之後更亂了，這次花點時間嘗試把所有東西搞懂，如果有人發現我哪邊還是搞錯的話歡迎大家指正。

文中引用 C 語言標準中的章節如果沒特別註明以 C99 為主。

## C Array Type

### C Types
在講 array parameter 之前來複習一下 C 的 array type，C types (6.2.5) 可以分類成以下幾種

* Basic types
* Void type
* Enumerated type
* Derived types

其中 Derived types 包含

* Array type
* Struct type
* Union type
* Function type
* Pointer type

Arrary Type 定義如下

> An array type describes a **contiguously allocated nonempty set of objects** with a particular member object type, called the element type. Array types are characterized by their **element type** and by the **number of elements** in the array. An array type is said to be derived from its element type, and if its element type is **T**,the **array type** is sometimes called **"array of T"**. The construction of an array type from an element type is called "array type derivation".
 
array type 由 element type 和 number of elements 決定，這兩個都是必要資訊，不同的 element type 和 number of elements 決定不同的 array type，例如說 `int [3]` 和 `int [10]` 是不同的 array type。

有一些網路上的教學會說 array 就是一個指標，這句話對也不對，而且經常讓初學者有錯誤的觀念，我就是被搞混的那個，其實 array type 和 pointer type 在 C 裡面是兩個截然不同的東西。

array 是代表一塊連續且大小固定的記憶體空間，就像 `int` 代表一塊大小為 4 bytes 的記憶體空間，pointer 也是一塊記憶體，只是記憶體的內容是另外一塊記憶體的地址，如果在 32-bits 系統底下 pointer 的大小是 4 bytes，64-bits 底下大小是 8 bytes。

問題在於 array 可以用 pointer 相同的語法來存取記憶體，所以他們看起來很像，但實際上是不同的東西。此外 array type 可以 cast 成 pointer type，但有一些 array type 的性質會消失，這稱為 array to pointer decay (且無法將 pointer type 轉換成 array type)。

**^^^ 注意上面這觀念很重要，function array paramter 的行為和 array to pointer decay 有很大的關聯 ^^^**

#### Scalar vs Aggregate types
有關於 type 的分類還有另一個值得知道的是

* Scalar types
    * 包含 arithmetic types and pointer types
* Aggregate types
    * 包含 array types and struct types

簡單來說 scalar types 就是簡單的一個值，而 aggregate type 是多個值組合起來的。

#### Incomplete types
另外 C 語言將所有 type 分為 object types、function type、incomplete types，object types 表示定義明確的物件，function type 就是 function，incomplete types 表式大小不明確的物件，所有非 incomplete types 或 function type 的物件都是 object types。C 語言中的 incomplete type 只有以下三種

* void
* arrays of unspecified length
* structures and unions with unspecified content

以下幾個例子為 incomplete type
```c
int []
struct Point
```

以下幾個例子為 complete type
```c
int [10]
struct Point { 
    int x;
    int y;
}
```

以下幾種宣告必須要使用 complete type，後面會看到 array elements 不允許 incomplete type 的例子
* array elements
* members of structures or unions
* objects local to a function

incomplete type 被允許出現在以下幾種宣告
* **Pointers** to incomplete types
* **Functions returning** incomplete types
* Incomplete **function parameter** types
* `typedef` names for incomplete types


### 宣告一個 Array (6.7.5.2, 6.7.8)

array 有兩種常見的宣告方式，`incomplete array declaration with array initializer` 和 `explicit sized array declaration`

要宣告一個 array 你必須明確的指定他的大小，或是指定初始值讓編譯器幫你計算所需要的大小
```c
// incomplete array declaration with array initializer
int a[] = { 1, 2, 3 };
// explicit sized array declaration
int b[4];
```

你也可以為已知大小的 array 指定 initializer
```c
int b[4] = { 1, 2, 3, 4 };
```

`char []` 可以用 string literal 初始化
```c
char s[] = "Hello!";
```

剛才有提到不同 size 的 array 視為不同的 type，因此應該無法將 array assign 給不同 size 的 array，但事實上所有的 array assignment 都是不允許的，即便是相同 size
```c
int a[3] = { 1, 2, 3 };
int b[4] = { 1, 2, 3, 4 };
int c[4];

c = a; // error: array type 'int [4]' is not assignable
c = b; // error: array type 'int [4]' is not assignable
```

#### Variable Length Array
C99 支援可變長度的 array，這類 array 的長度是由一個變數而非常數決定，在 run time 時決定而非 compile time

```c
int i = 5;
int d[i];
```

 VLA 無法使用 initializer 初始化
```c
int e[i] = { 1, 2, 3 }; // error: variable-sized object may not be initialized
```

## Function Declaration with Array Parameter (6.7.5.3, 6.9.1)

### Function Declaration
先來談 function declarator，declarator 在 C 裡面表示宣告某一個物件 (變數) 的語法中指定物件名稱 (identifier) 和 derivated type 相關特性的語法，例如
```c
int i = 1
```
中 `i` 為 declarator，而 `int` 為 declaration-specifier，表示這行宣告要宣告物件是什麼類型，而 `=` 後面的稱為 initializer，用來指定初始值，這整行稱為 declaration，即宣告。

上面看到的是最簡單的 declarator，再更早我們有看到 array declarator `arr[N]`，而以下語法為 function declarator
```c
func(parameter-type-list)
```

其中 parameter-type-list 為
```c
declaration-specifiers declarator , declaration-specifiers declarator , ...
```

function declarator 前面加上 declaration-specifier 表示函式的 return type
```c
declaration-specifiers func(parameter-type-list)
```

只宣告 return type 和 parameter list 稱為 function prototype，例
```c
int func(int i);
```

function prototype 可以不用包含變數名稱
```c
int func(int);
```

若包含 code block 稱為 function definition
```c
int func(int i) {
    return i * 2;
}
```

### Array Parameter
如果想宣告 array parameter 語法如下
```c
void fn(int a[3]);
```

但是 6.7.5.3.7 提到
> A declaration of a parameter as "array of type" shall be adjusted to "qualified **pointer to type**"
 
qualified 等一下再談，我們可以看到 array type 如果被宣告在 parameter list 裡面的話會被轉換成 pointer type，這時候就會有 array to pointer decay 的問題。

array to pointer decay 最常碰到的一個問題就是 `sizeof` 的執行結果，我們知道如果對一個變數使用 `sizeof` 會得到該變數的記憶體大小 (in bytes)，剛剛有提到 array type 是一整塊的記憶體，所以 `sizeof` 會回傳整塊記憶體的大小，例如 `sizeof int [10]` 會得到 40，但 pointer 是一個儲存地址的記憶體，所以 `sizeof` 會得到儲存該地址的記憶體大小，例如 `sizeof int *` 在 64 位元的系統下會得到 8。

回到 function declaration，因為 array type 在 function declaration 時會被 decay 成 pointer，所以以下程式的執行結果印出的會是 8 而不是 24
```c
void fn(int a[3]) {
    printf("%lu\n", sizeof a);
}
```

事實上如果你對 array paramter 做 sizeof，編譯器會有 warning
```
warning: sizeof on array function parameter will return size of 'int *' instead of 'int [3]'
```

因為是 pointer type，所以可以帶入任意長度的 array
```c
int a[3] = { 1, 2, 3 };
int b[4] = { 1, 2, 3, 4 };

fn(a); // working, of course, printout = 8
fn(b); // still working, printout = 8
```

所以其實在 parameter list 中的 size 是沒有作用的，也可以直接省略不寫，或直接宣告成 pointer
```c
void fn2(int a[]) {
    printf("%lu\n", sizeof a);
}

void fn3(int *a) {
    printf("%lu\n", sizeof a);
}
```

以上宣告方式都是等價的

### Type Qualifier, `[static N]`, and `[*]`

array declarator 完整的定義如下
```c
D[type-qualifier-list(opt) assignment-expression(opt)]
D[static type-qualifier-list(opt) assignment-expression]
D[type-qualifier-list static assignment-expression]
D[type-qualifier-list(opt) *]
```

我們常見的 `arr[N]` 是不包含 type qualifier 的第一種，其他所有的可能都只能被用在 function declaration 的 parameter list 裡面 (6.7.5.2.1)
> The optional type qualifiers and the keyword static shall **appear only in a declaration of a function parameter with an array type**, and then **only in the outer most array type derivation**.

type qualifier 就是指 `const` `restrict` `volatile` 這三個關鍵字，代表該變數在記憶體中的特性，有興趣可以自己研究，改天嘗試看看寫一篇介紹。

第二、三個基本上是同一件事，也就是「在 function parameter 中的 array type 可以在大小之前加入 `static` 關鍵字，且可以加上 type qualifier」，兩句只是想表達 `static` 跟 type qualifier 可以交換順序。

最後一個與 VLA 有關，表示你可以這樣在 parameter 中宣告 VLA `fn(int a[*])` 不過這只能在 function prototype 中使用，function definition 中是不允許的，待會再詳細說明。

後半段 outer most array 的部分和超過一維的矩陣有關，最後會提到。

#### Type Qualifier
type qualifier 會作用在轉換過後的 pointer，例如以下兩個宣告是等價的
```c
void fn(int a[const]);
void fn(int *const a);
```

> 註：`int const *a` 和 `int *const a` 是不一樣的，前者為該 pointer 指向的空間為 const，後者表示該 pointer 所在的記憶體空間為 const，可以將前者讀做 "pointer to const" 後者讀做 "const pointer" 方便記憶。

#### `[static N]`
static 關鍵字後面一定要接一個數字，表示傳入的 array 至少要大於這個大小，這是由編譯器進行檢查的，如果傳入小於該大小的 array (包含 null) 會有 warning (而非 error)
```c
void fn(int arr[static 4]);

int a[3] = { 1, 2, 3 };
int b[4] = { 1, 2, 3, 4 };
fn(b); // working without warning or error
fn(a); // warning: array argument is too small; contains 3 elements, callee requires at least 4 [-Warray-bounds]
fn(NULL); // warning: null passed to a callee that requires a non-null argument [-Wnonnull]
```

#### `[*]`
這是給 VLA 用的，如果你的 array 是個 VLA 的話，可以在 function prototype 用 `[*]` 宣告，如果你不知道 VLA 用哪個變數宣告大小
```c
void fn(int, int [*]);
void fn(int n, int arr[n]);
```

到這你可能會想個問題，既然 array 會被 decay 成 pointer，那 VLA 有什麼意義?

沒錯，上述例子等價
```c
void fn(int n, int arr[]);
void fn(int n, int *arr);
```

因此這裡使用 VLA 是沒有意義的，但如果宣告的是一個二維的 array type 的話，就有差別了，在那之前先來看看二維陣列會有什麼樣的行為

### 2D Array

宣告一個二維陣列
```c
int arr[10][10];
```

首先並不存在「二維陣列」這種型別，上面這樣的宣告應該解讀為「以 "長度為 10 的 int 陣列"  element type，長度是 10 的陣列」所以他還是一個 array type，只是他的 element type 也是個 array type。

### 2D Array as Function Parameter

那麼如果將 2D array 做為參數會發生什麼事情呢
```c
void fn(int arr[10][10]);
```

首先最外層的 array type 會 decay 成指向陣列第一個元素的 pointer，也就是指向第一個 `int [10]` array 的 **pointer**，接下來因為他已經 pointer type 而不是 array type，所以並不會繼續 decay 下去，因此上面與以下宣告等價
```c
void fn(int (*arr)[10]);
```
且與以下宣告不等價
```c
void fn(int **arr);
```
> 註：`int *arr[N]` 和 `int (*arr)[N]` 不等價，前者等價 `int *(arr[N])`，是 `int *` 的 array，後者是指向 `int [N]` 的 pointer，可以將前者讀做 "pointer to array" 後者讀做 "array of pointer" 方便記憶。

回到 VLA，考慮以下例子
```c
void fn(int n, int m, int arr[n][m]);
```

`arr` 會被 decay 成 pointer，指向的是一個長度是 m 的 VLA。其 function prototype 可以這樣宣告
```c
void fn(int, int, int arr[*][*]);
```
且與以下宣告等價
```c
void fn(int, int, int arr[][*]);
void fn(int, int, int (*arr)[*]);
```
與以下宣告不等價
```c
void fn(int, int, int arr[][]); // illegel
void fn(int, int, int **arr);
```
其中第一個是不允許的，還記的前面提到的 incomplete type 嗎? `int []` 做為 incomplete type 不能出現在 element type 中，因此 `int [][]` 是不合法的 (除了帶有 initializer 的 array declaration 之外，帶有 initializer 的陣列宣告其結果為 complete type)。

除此之外，剛剛提到的 type-specifier 和 static 關鍵字也只能寫在最外層的中括號裡，因為其他的並不會被 cast 成 pointer，在裡面指定 type-specifier 和 static 並沒有意義，可以回去看剛剛提到的 (6.7.5.2.1) 的後半句。
```c
void fn(int arr[const 10][10]);
void fn(int arr[static 10][10]);

void fn(int arr[][const 10]); // error: type qualifier used in non-outermost array type derivation
void fn(int arr[][static 10]); // error: 'static' used in non-outermost array type derivation
```

## C++
如果是 C++ 可以用 pass by reference 來避免 array to pointer decay
```c
void fn(int (&arr)[10]) {
    std::cout << sizeof arr; // 40!
}
```

## 小結
因為 array parameter 的 array to pointer decay 的特性，你無法使用常見的 [用 sizeof 去計算 array 的長度](https://stackoverflow.com/questions/37538/how-do-i-determine-the-size-of-my-array-in-c) 的方法來得到 array 長度
```c
void fn(int arr[10]) {
    int size = sizeof(arr) / sizeof(arr[0]);
    printf("%lu\n", size); // get 2 instead of 10
}
```

如果你想這麼做的話你必須讓 callee 去做這件事情並把 size 作為參數傳入
```c
void fn(int size, int arr[]) {
    printf("%lu\n", size); // get 10
}

int arr[10];
int size = sizeof(arr) / sizeof(arr[0]);
fn(size, arr);
```

或是利用 C++ 的 pass by reference
```c
void fn(int (&arr)[10]) {
    int size = sizeof(arr) / sizeof(arr[0]);
    std::cout << size; // get 10
}
```

就是這個問題困擾了我許久，不知道該怎麼在 function 裡面得到長度的資訊，此外讓二維陣列做為參數傳入的方法也是不太好理解的東西。

以上！

## References
* [ISO/IEC 9899:1999 - N1256 (C99)](http://www.open-std.org/jtc1/sc22/wg14/www/docs/n1256.pdf)
* [What is a full declarator in C? - Stack Overflow](https://stackoverflow.com/questions/36071441/what-is-a-full-declarator-in-c)
* [Is the operand of `sizeof` evaluated with a VLA? - Stack Overflow](https://stackoverflow.com/questions/32985424/is-the-operand-of-sizeof-evaluated-with-a-vla)
* [pointer to array as function argument in C - Stack Overflow](https://stackoverflow.com/questions/41538342/pointer-to-array-as-function-argument-in-c)
* [Why do arrays in C decay to pointers? - Stack Overflow](https://stackoverflow.com/questions/33291624/why-do-arrays-in-c-decay-to-pointers)
* [Why use an asterisk “[*]” instead of an integer for a VLA array parameter of a function? - Stack Overflow](https://stackoverflow.com/questions/17371645/why-use-an-asterisk-instead-of-an-integer-for-a-vla-array-parameter-of-a-f)
* [C++ pass an array by reference - Stack Overflow](https://stackoverflow.com/questions/10007986/c-pass-an-array-by-reference)
* [6.11 Incomplete Types (Sun Studio 12: C User's Guide)](https://docs.oracle.com/cd/E19205-01/819-5265/bjals/index.html)
* [Array declaration#Array to pointer decay (cppreference.com)](https://en.cppreference.com/w/cpp/language/array#Array-to-pointer_decay)
* [Aggregate initialization (cppreference.com)](https://en.cppreference.com/w/cpp/language/aggregate_initialization)
* [Type (cppreference.com)](https://en.cppreference.com/w/c/language/type)
* [A nice, little known C feature: Static array indices in parameter declarations](https://hamberg.no/erlend/posts/2013-02-18-static-array-indices.html)