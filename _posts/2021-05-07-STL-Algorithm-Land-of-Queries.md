---
layout: post
toc: true
title: "STL Algorithm: Land of Queries"
date: 2021-05-07
category: blog
tag: 
- C++
author: ingerchao
---



## 数值算法

count: 返回容器中等于指定目标的元素数量；前两个参数是查找范围，第三个参数为目标值

count_if: 返回容器中符合条件的元素数量；前两个参数为查找范围，第三个参数为判断条件，是一个返回值为 bool 的函数，也可以是 lambda 函数。

```c++
    vector<int> vec = {1, 2, 3, 3, 5, 5, 3, 6, 7, 9, 10};
    int target1 = 3, target2 = 4;
    cout << "target 1: " << target1 << " count: "<< count(vec.begin(), vec.end(), target1) << endl;
    cout << "target 2: " << target2 << " count: "<< count(vec.begin(), vec.end(), target2) << endl;
    cout << "number divisible by 3: " << count_if(vec.begin(), vec.end(), [](int i){return i % 3 == 0;}) << endl ;
```

```
target 1: 3 count: 3
target 2: 4 count: 0
number divisible by 3: 5
```

