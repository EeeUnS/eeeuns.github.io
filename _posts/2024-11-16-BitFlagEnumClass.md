---
layout: post
title:  "BitFlagEnumClass"
date:   2024-11-16 19:26:01 +0900
tag: cpp
---

[https://devblogs.microsoft.com/cppblog/visual-studio-2008-enum-bit-flags-visualization/](https://devblogs.microsoft.com/cppblog/visual-studio-2008-enum-bit-flags-visualization/)

VS에서는 enum을 Bit Flag처럼 사용할시에 아래처럼 debug view 에서 비트에 해당하는 enum값을 보여주고있다.

![image.png](/images/bitflagenumclass/image.png)

굉장히 도입된지 오래된 기능인데, 

기존 C스타일 enum도 그렇지만

특히 enum class들은 사칙연산, 비트연산을 위해서는 정수형처럼 변환을 해서 사용해야하기에 

casting부분에서 굉장히 불편함이 많다.

개인적으로 cpp스타일의 캐스팅을 사용하긴 하지만, 정말 타이핑적으로 귀찮다고 생각하기에;;

그래서 다른 사람들은 막 실제로 typedef로 정수형인데 타입명만 바꿔서 사용하고 이런다.

이럴 경우 아래 당연히 ide는 정수형 타입으로 인식해 visualization은 지원 안되는 문제가 있다 

이런 번거로움을 하나 없애보자 class를 하나 만들어서 상속받으면 되지않을까?

는 생각을 했고

enum class도 class인데 상속이 가능하지않을까란 생각을 했지만, 문법적으로 상속은 지원이 불가능했다.

어차피 enum class도 타입이겠다 template을 이용하자고 생각했고, 코드도 사실 굉장히 간단하다.

[https://github.com/EeeUnS/BitFlagEnumClass](https://github.com/EeeUnS/BitFlagEnumClass)

사실상 casting하는 로직만 귀찮지않게 따로 빼놨을 뿐인 수준

큰 코드 베이스일수록 곳곳에 bit flag를 사용하는 enum들이 굉장히 산재해있고, 

얘들이 디버깅시에 창에 제대로 안뜨면 직접 대조를 해봐야하니 파악이 굉장히 어렵게한다.

```jsx
#include "BitFlagEnumClass.h"

enum class eBigflag : int
{
	zero = 0,
	nii = 0x1,
	niii = 0x2,
	niiii = 0x4
};

using eBigFlag = BitFlagEnumClass<eBigflag>;

int main()
{
	eBigFlag a = eBigflag::zero;
	a |= eBigflag::nii;
	a |= eBigflag::niii;

	return 0;
}
```

~, shift 연산까지 할정도로 extream하게 쓰진않기에 따로 구현은 하지않았다.

힙하게 using을 써봤지만 typedef로 해도 동일하다.

![image.png](/images/bitflagenumclass/image%201.png)

구조도 진짜 타입 하나만 멤버변수로 가지고있으니 구조도 굉장히 단순하고 고쳐쓰기도 용이 할거라 생각한다.