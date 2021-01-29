---
layout: post
title:  "biginteger "
date:   2021-01-02 02:26:01 +0900
tag: etc
---


# 덧셈 연산과 뺄셈 연산의 부호

빅인테저를 직접 짜고있던 와중에 +연산과 -연산을 어떻게 처리할 지 골치였다. 실질적으로 더하는 연산과 빼는 연산은 연산이 들어오는것이아닌 복합적으로 값의 부호까지 고려해서 연산해야한다. 따라서 이를 따로 나눠주고 각 케이스에 맞춰서 서브 프로시저를 부르는 식으로 짜주었다. 그런데 이제 결과값의 부호를 어떻게 처리할지 문제가 생겼다.

각 케이스를 다음의 표로 나누었다

|부호| 연산자| 부호 |결과값 부호|
|:--:|:--:|:--:|:--:|
|+|+|+|+|
|+|-|+|모름|
|+|+|-|모름|
|+|-|-|+|
|-|+|-|-|
|-|-|-|모름|
|-|+|+|모름|
|-|-|+|-|


이중 실질적인 빼기 연산을 하는 것들은 결과 부호를 모르는 경우와 동일하다. 여기에 첫번째값과 두번째값의 절댓값 크기에 따라서 결과값 부호를 나누면 다음과 같다.

|부호| 연산자| 부호 | 큰쪽(절댓값) |결과값 부호|
|:--:|:--:|:--:|:--:|:--:|
|+|-|+|>|+|
|+|-|+|<|-|
|+|+|-|>|+|
|+|+|-|<|-|
|-|-|-|>|-|
|-|-|-|<|+|
|-|+|+|>|-|
|-|+|+|<|+|


이중에 공통점으로 묶을 수 있는것이 존재한다.

항상 연산자가 + 일때는 결과값 부호는 항상 큰쪽의 부호를 따르고 + 일때는 뒤쪽이 큰쪽일 경우에만 부호를 뒤집은 것을 따른다.  또한 연산자에 따라서 부호의 앞뒤 부호가 같고 다름이 차이가 난다. 이때에 연산자를 따로 기억할 필요없이 계산하는 값의 부호와 절댓값 크기비교만으로 결과값 부호를 도출해 낼 수 있다.



|부호| 연산자| 부호 | 큰쪽(절댓값) |결과값 부호|
|:--:|:--:|:--:|:--:|:--:|
|+|-|+|>|+|
|+|-|+|<|-|
|-|-|-|>|-|
|-|-|-|<|+|
|+|+|-|>|+|
|+|+|-|<|-|
|-|+|+|>|-|
|-|+|+|<|+|

이때 리턴값을 구할때 큰값을 기준으로 생성한다면 큰값을 따라서 부호가 정해졌으므로 부호가 같고 뒤의 값이 큰 경우에만 부호를 뒤집어주면 된다.


# 곱셈

결과 값의 부호는 if로 나눌 필요없이 다음으로 할 수 있다.

```c++
returnValue.mSign = (mSign == a.mSign);
```

