---
layout: post
title:  "3. 확률적 분석(Indicator random variables)"
date: 2021-07-16 02:53:01 +0900
tag: CLRS
---

- [1. 알고리즘 기초 지식](https://eeeuns.github.io/2021/07/13/alg/)
- [2. 분할정복(Divide and Conquer)](https://eeeuns.github.io/2021/07/15/divide-and-conquer/)
- [3. 확률적 분석(Indicator random variables)](https://eeeuns.github.io/2021/07/15/indicator-random-variables/)

# 평균 수행시간
    
- 입력 값의 분포에 대한 수행시간의 평균값.
- $$\Pr$$ : 확률
- x : 특정 입력값에 대해서 걸리는 수행시간
- $$\displaystyle E[X] = \sum_{x=1}^n x\Pr\{X=x\}$$



# 예시 : 고용 문제
- 매일 한명씩 지원자가 와서 면접을 본다. 
- 지원자가 현재 고용자보다 뛰어나면 해고하고 지원자를 고용한다.
- 이때 면접 비용 $$c_i$$와 고용 비용 $$c_h$$가 든다.
- 고용 비용이 면접 비용보다 훨씬 비싸다.
- 이때 면접과 고용에 드는 비용을 알고싶다.


```c++
HIRE-ASSISTANT(n)
    best = 0 // candidate 0 is a least-qualified dummy candidate
    for i = 1 to n
        interview candidate i
        if candidate i is better than candidate best
            best = i
            hire candidate i
```    

- 고용된 인원을 $$m$$이라 할때 총 비용은 $$O(c_in + c_hm)$$
- 최악의 경우 고용비용
    $$O(c_hn)$$
- 평균 고용 비용은?



# 지표확률변수



$$\begin{aligned}
    I\left\{ A \right\} =  
\begin{cases}
    1 &    (H {발생}) \\
    0 & ( \bar{H} {발생})
\end{cases}    \end{aligned}$$

$$
    \begin{aligned}
        E[X_A] &=  E[I \left\{ A \right\} ] \\
        &= 1 \times \Pr (A) + 0 \times \Pr(\bar{A}) \\
        &=\Pr (A)     
    \end{aligned}
$$

확률 변수를 0,1로 고정한 특별한 확률변수 횟수를 셀때 유용

- $$X$$ : 새로운 직원을 고용한 횟수에 대한 확률 변수.
- $$X_i$$ : $$i$$번째 지원자가 고용되었는지에 대한 지표 확률 변수.


$$
\begin{aligned}
X_i = I \left\{ {지원자 i가 고용됨} \right\} =
    \begin{cases}
        1 & 지원자 i 고용 \\
        0 & 지원자 i가 고용 안됨 
    \end{cases}    
\end{aligned}
$$  

$$X = X_1 + X_2 + \cdots + X_n$$


- $$\Pr\{지원자 $$i$$가 고용될 확률\}$$ ???
- $$E[X_i] = \dfrac{1}{i}$$


$$
    \begin{aligned}  
        E[X] &= E \left[  \sum_{i=1}^n X_i  \right] \\ 
        &= \sum_{i=1}^n E[X_i] \\ 
        &= \sum_{i=1}^n \dfrac{1}{i} \\ 
        &= \ln n + O(1)
    \end{aligned}
$$

결론
- 평균 고용 비용 : $$O(c_h \ln n)$$
- 총 평균 : $$O(c_in + c_n \ln n)$$
