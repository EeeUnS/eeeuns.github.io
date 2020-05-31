---
layout: post
title:  "DreamHack Memory Corrunption in C++"
date:   2020-05-31 21:53:01 +0900
categories: paper
---

# [System Exploitation Fundamental 코스](https://dreamhack.io/lecture/curriculums/2)

# 실습에대한 답이 있으므로 직접 해결하실분은 보지마세요

# Memory Corruption - C++


## Buffer Overflow in C++

```cpp
// g++ -o bof-1 bof-1.cpp
#include <iostream>
int main(void) {
  char buf[20];
  std::cin >> buf;
}
```

```cpp
// g++ -o bof-2 bof-2.cpp
#include <iostream>
int main(void) {
  std::string buf;
  std::cin >> buf;
}
```





```cpp
// g++ -o container-1 container-1.cpp
#include <algorithm>
#include <vector>
#include <iostream>
void f(const std::vector<int> &src) {
        std::vector<int> dest(5);
        std::copy(src.begin(), src.end(), dest.begin());
}
int main(void){
        int size = 0;;
        std::vector<int> v;
        std::cin >> size;
        v.resize(size);
        v.assign(size, 0x41414141);
        f(v);
}
        
```

```cpp
// g++ -o container-2 container-2.cpp
#include <algorithm>
#include <vector>
#include <iostream>
void f(const std::vector<int> &src) {
        std::vector<int> dest(src); //오버플로우 막기
        std::copy(src.begin(), src.end(), dest.begin());
}
int main(void){
        int size = 0;;
        std::vector<int> v;
        std::cin >> size;
        v.resize(size);
        v.assign(size, 0x41414141);
        f(v);
}
```





```cpp

```







