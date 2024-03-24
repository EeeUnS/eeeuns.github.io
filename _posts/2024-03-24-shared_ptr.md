---
layout: post
title:  "shared_ptr"
date:   2024-03-24 19:26:01 +0900
tag: stl
---

# shared_ptr

[https://github.com/microsoft/STL/blob/main/stl/inc/memory](https://github.com/microsoft/STL/blob/main/stl/inc/memory)

![Untitled](/images/shared_ptr/Untitled.png)

생성자에서 해당 함수 호출

![Untitled](/images/shared_ptr/Untitled%201.png)

![Untitled](/images/shared_ptr/Untitled%202.png)

shared_ptr 은 _Ptr _base 상속 받아서 사용

_Ptr_base 클래스는  다음 멤버변수를 들고있다.

![Untitled](/images/shared_ptr/Untitled%203.png)

![Untitled](/images/shared_ptr/Untitled%204.png)

_Ref_count_base 값은 그냥 단순 변수 두개를 들고있다.

![Untitled](/images/shared_ptr/Untitled%205.png)

counting이 intelock계열을 이용해 atomic 하게 작동하는 함수임을 알 수 있다.

![Untitled](/images/shared_ptr/Untitled%206.png)

*_Ref_count가 실제로 사용.*

ptr을 추가로 가지고 있음.

전체 도식화

![Untitled](/images/shared_ptr/Untitled%207.png)
