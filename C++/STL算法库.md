# STL算法库

> STL算法库提供了一组通用的、可以应用于各种容器的函数模板，通过迭代器来访问元素。用于执行常见的数据操作，如排序、搜索、复制、修改容器中的元素等。

STL算法主要分布在三个头文件中：`<algorithm>`、`<numeric>`和`<functional>`。

`<algorithm>`头文件，包含了大量函数模板，用于操作序列容器（如`vector`、`list`、`deque`等）中的元素。算法大致可以分为以下几类：

- **非修正序列算法**：不对容器内容做直接修改，如`std::copy`、`std::find`、`std::equal`等。
- **修正序列算法**：会修改容器中的元素，如`std::transform`、`std::replace`、`std::remove_if`等。
- **排序算法**：用于对容器内的元素进行排序，如`std::sort`、`std::stable_sort`、`std::partial_sort`等。
- **搜索算法**：在容器中查找特定元素或子序列，如`std::find_if`、`std::search`、`std::binary_search`等。
- **集合算法**：在已排序的序列上执行操作，如`std::set_union`、`std::set_intersection`等。
- **其他**：还包括反转元素顺序的`std::reverse`、合并序列的`std::merge`等。

# 一、非修正序列算法

> 非修正序列算法是STL（Standard Template Library）中提供的一类算法，其特点是不修改所作用的容器内的元素次序和值。这些算法主要用于计算元素个数、查找元素等不改变容器状态的操作。

以下是一些非修正序列算法的使用示例。

## 1、count

`count`算法用于计算容器中等于给定值的元素个数。

```cpp
std::vector<int> vec = { 1,2,3,2,2,4,5 };

//count算法用于计算容器中给定范围内等于给定值的元素个数，需要提供用来划定范围的指针
size_t count_result = std::count(vec.begin(), vec.end(), 2);

std::cout << "数字2出现的次数：" << count_result << std::endl;
```

## 2、count_if

根据`谓词（一个返回bool值的函数或函数对象）`计算满足条件的元素数量。

```cpp
#include <iostream>
#include <algorithm>
#include <vector>

//谓词
bool isEven(int x) { return x % 2 == 0; }

int main()
{
	std::vector<int> vec = { 1, 2, 3, 4, 5, 6 };
	
	//count_if根据谓词计算符合条件的元素数量
	int evenCount = std::count_if(vec.begin(), vec.end(), isEven);
	
	std::cout << "偶数的数量： " << evenCount << std::endl;
}
```

# 二、修正序列算法

> 修正序列算法是STL（Standard Template Library）中用于修改容器内容的一系列算法。这些算法可以重新排列元素、添加或删除元素，或执行其他类型的修改。

下面是一些修正序列算法的使用示例及其注意事项。

## 1、 remove

将满足条件的所有元素移至容器末尾，并返回新逻辑末尾的迭代器，但不真正删除元素，需要手动擦除。

```cpp
std::vector<int> vec = { 1, 2, 3, 4, 5, 3, 6 };

//remove将满足条件的所有元素移至容器末尾，并返回排除被删除元素之外的新容器末尾的迭代器，但不真正删除元素，需要手动擦除
auto newEnd = std::remove(vec.begin(), vec.end(), 3);

vec.erase(newEnd, vec.end()); // 必须手动擦除被"移除"的元素(被移到后面的那部分)

for (int num : vec)
{
    std::cout << num << " ";
}
```

## 2、remove_if

与`remove`类似，但根据`谓词（一个返回bool值的函数或函数对象）`来决定移除哪些元素。

```cpp
#include <iostream>
#include <algorithm>
#include <vector>

bool isEven(int x) { return x % 2 == 0; }

int main()
{
    std::vector<int> vec = { 1, 2, 3, 4, 5, 6 };
	
	//remove_if与remove类似，但根据谓词来决定移除哪些元素
    auto newEnd = std::remove_if(vec.begin(), vec.end(), isEven);
    
    vec.erase(newEnd, vec.end());// 必须手动擦除被"移除"的元素(被移到后面的那部分)
    
    for (int num : vec)
    {
        std::cout << num << " ";
    } 
}
```

## 3、replace

将容器中满足特定值的所有元素替换为另一个值。

```cpp
    std::vector<int> vec = { 1, 2, 3, 2, 5 };
    
    //replace将容器中满足特定值的所有元素替换为另一个值。
    std::replace(vec.begin(), vec.end(), 2, 99);
    
    for (int num : vec)
    
    {
        std::cout << num << " ";
    }
```

## 4、reverse

**反转给定范围内元素的顺序**。

```cpp
std::vector<int> vec = { 1, 2, 3, 4, 5 };

//reverse用于反转给定范围内元素的顺序。
std::reverse(vec.begin(), vec.end());

for (int num : vec)
{
    std::cout << num << " ";
}
```

# 三、搜索和排序算法

## 1、find

`std::find`算法用于在容器中`查找等于给定值的第一个元素，并返回一个指向该位置的迭代器，如果一直没找到的话会返回end()。`，结合`std::distance算法`用于`计算两个迭代器中间的距离`，可以精准定位该位置。

```cpp
std::vector<int> vec = { 1, 12, 23, 15, 6 ,10, 15 };

//查找
auto position = std::find(vec.begin(), vec.end(), 15);

if (position != vec.end())
{
	//计算容器首位置到该位置的距离
    std::cout << "首次出现的位置: " << std::distance(vec.begin(), position) << std::endl;
}
```


## 2、find_if

**根据谓词在给定范围内查找满足特定条件的第一个元素，如果一直没找到的话会返回end()。**

```cpp
#include <iostream>
#include <vector>
#include <algorithm>

bool isEven(int x) { return x % 2 == 0; }

int main()
{
    std::vector<int> vec = { 1, 3, 5, 4, 6 };
	
	//find_if根据谓词在给定范围内查找满足特定条件的第一个元素。
    auto it = std::find_if(vec.begin(), vec.end(), isEven);
    
    if (it != vec.end())
    {
        std::cout << "第一个偶数的位置: " << std::distance(vec.begin(), it) << std::endl;
    }
    else
    {
        std::cout << "没有偶数" << std::endl;
    }    
}
```

## 3、sort

**默认升序对容器中的元素进行排序。**

```cpp
std::vector<int> vec = { 3, 1, 4, 1, 5, 9, 2, 6 };

//sort对容器中的元素进行排序
std::sort(vec.begin(), vec.end()); // 默认升序
```

**sort也支持使用谓词进行排序。**

```cpp
bool compare(int a, int b) {    return a > b; }// 降序排序

// 使用自定义比较函数（谓词）进行降序排序
std::sort(vec.begin(), vec.end(), compare);
```

# 四、函数对象

> 在C++中，函数对象（也称为functors或函数对象类）指的是可以像函数那样被调用的对象。它们实际上是类的实例，这些类重载了`()`函数调用运算符，使得它们的实例能够像函数一样被调用。

函数对象结合了函数的调用便利性和对象的灵活性，是泛型编程的强大工具。

## 1、实现函数对象

可以通过**结构体或类**来实现，重载`函数调用运算符()`。

```cpp
//使用结构体实现重载函数调用运算符()
struct Adder
{
	//要把操作的形参也定义好
    int operator()(int a, int b) const {//定义一个数字相加的函数对象
        return a + b;
    }
};

//使用类重载函数调用运算符()
class AscendingComparator
{
public:
	//要把操作的形参也定义好
    bool operator()(int a, int b) const {//定义一个升序比较的函数对象
        return a < b;
    }
};
```

## 2、使用函数对象

在STL算法（如`std::sort`, `std::find_if`）中作为比较函数或谓词，可以利用函数对象来定制算法的行为。

```cpp
#include <iostream>
#include <vector>
#include <algorithm>

//使用结构体实现重载函数调用运算符()
struct Adder
{
    int operator()(int a, int b) const {
        return a + b;
    }
};

//使用类重载函数调用运算符()
class AscendingComparator
{
public:
    bool operator()(int a, int b) const {//定义一个升序比较的函数对象
        return a < b;
    }
};

int main()
{
    //使用结构体函数对象实现相加的功能
    Adder test;
    std::cout << test(10, 20) << std::endl;
	
    //使用自定义的比较函数对象进行升序排序
    std::vector<int> vec = { 5, 3, 2, 8, 1, 4 };
    std::sort(vec.begin(), vec.end(), AscendingComparator());
	
	//遍历
    for (int num : vec) {
        std::cout << num << " ";
    }
    std::cout << std::endl;
    
    return 0;
}
```

上述代码中，虽然**看起来没有先实例化一个变量**，但是AscendingComparator()这一表达式在执行时会立即创建一个**匿名的临时对象**。在 C++ 中，这种临时对象的创建方式是合法且常见的，特别是在需要传递一个简单的、仅用于一次性使用的对象时。这种方式可以避免不必要的变量声明，使代码更加简洁，**C++中的lambda 表达式本质上也是一个匿名的临时函数对象**。