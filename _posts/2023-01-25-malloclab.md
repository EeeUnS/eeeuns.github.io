---
layout: post
title: "malloc lab 1. 개념정리"
date:   2023-01-25 19:26:01 +0900
tag: csapp
---

[15-213: Introduction to Computer Systems / Schedule Fall 2015](http://www.cs.cmu.edu/afs/cs/academic/class/15213-f15/www/schedule.html)

[](http://www.cs.cmu.edu/afs/cs/academic/class/15213-f15/www/lectures/19-malloc-basic.pdf)

[](http://www.cs.cmu.edu/afs/cs/academic/class/15213-f15/www/lectures/20-malloc-advanced.pdf)

[Lecture 19: Dynamic Memory Allocation: Basic Concepts](https://scs.hosted.panopto.com/Panopto/Pages/Viewer.aspx?id=d69a8072-3d23-4604-8081-0edeba33bb52)

[Lecture 20: Dynamic Memory Allocation: Advanced Concepts](https://scs.hosted.panopto.com/Panopto/Pages/Viewer.aspx?id=3efacbed-aa6d-4d18-b354-24cdc453e1e8)

영강이긴해도 ppt가 잘되어있어서 그냥 ppt보고 쭉 따라가도되더라 책 읽는것보다 더 편했음.

![Untitled](/images/malloclabf3e/Untitled.png)

뭐 그렇다고 합니다.

malloc free같은 기초는 넘어가고

구현 이슈로 바로 넘어가자.

![Untitled](/images/malloclabf3e/Untitled%201.png)

내부 단편화, 외부 단편화 운영체제 공부하면 알게되는 지식

넘어넘어가자

malloc이 일반적으로 반환 포인터 앞에 사이즈를 저장하고 free list를 운용한다는 정도는 아는 상태에서 내용을 전개.

# 구현이슈

![Untitled](/images/malloclabf3e/Untitled%202.png)

1. free하는 블럭의 사이즈를 알아내는 방법
2. free한 블럭들 관리 방법
3. free한 블럭들 보다 작은 공간을 요구할때 어떻게 해야하나?
4. 사이즈에 맞는 블럭 할당을 위해 적합한 블럭 찾는 방법
5. 해제한 블럭 다시 삽입하는 방법

# 1. free하는 블럭의 사이즈를 알아내는 방법

![Untitled](/images/malloclabf3e/Untitled%203.png)

일반적으로 헤더에 사이즈를 집어넣는다.

# 2. free한 블럭들 관리 방법

![Untitled](/images/malloclabf3e/Untitled%204.png)

1. implicit list(묵시적 리스트) 사용. 헤더 사이즈 만큼 pointer이동하면 다음 블럭 헤더를 찾는식 
    1. 이러기 위해서 이 블럭이 사용중인지 확인할 필요가 있다. 헤더에 비트를 쪼개서 flag를 세운다
2. explicit list(명시적 리스트) : 그냥 포인터로 다음 freed 블럭을 가르킨다.
3. Segregated free list(분리가용리스트) : 사이즈별로 프리 리스트를 가진다
4. Blocks sorted by size() : 균형 트리를 이용해 적합한 사이즈 블럭을 빠르게 찾는 식.

그럼 이제 implicit list 의 세부 구현법에 대해서 먼저 알아보자.

# implicit

![Untitled](/images/malloclabf3e/Untitled%205.png)

블럭의 구조는 이렇게 된다.

![Untitled](/images/malloclabf3e/Untitled%206.png)

단방향 리스트같은 모습이 나온다.

다음 블럭 포인터 주소는 헤더의 저장된 사이즈를 보고 그 크기만큼 이동하면 된다.

이렇게하고 사이즈 저장하는 헤더에 추가적인 비트 할당을 해서 할당된 블럭인지 free 된 블럭인지 표시를 해서 freed 블럭을 찾는 방법이다.

이 구현법에서 freed 블럭을 찾는 방법은 크게 세가지가 있다.

![Untitled](/images/malloclabf3e/Untitled%207.png)

1. first fit

그냥 제일 처음 찾은 사이즈 만족하고 빈놈을 그대로 반환

항상 처음부터 탐색한다.

1. next fit

반환한 블럭 그 다음부터 탐색.

1. best fit

전체 다돌고 젤 적합한 놈 반환.

메모리 효율이 좋으나 제일 느리다.

# freed 블럭 병합 Coalescing

외부 단편화를 해결하기위해 freed 블럭이 메모리상에 붙어있으면 큰 블럭으로 바꿔줘야한다.

이에 대한 깔끔한 해결책으로 장수하시고 CS책에 안 나오는 곳이 없는 크누스 형님이 또 좋은 방법을 고안해 내셨다.

![Untitled](/images/malloclabf3e/Untitled%208.png)

Boundary	tags 방법이라고  헤더처럼 footer를 추가해서 free할때 해당 블럭 앞뒤의 footer header를 각각보고 상황에 따라 병합하는 거다. 이렇게하면 상수시간에 끝낼 수 있다.

![Untitled](/images/malloclabf3e/Untitled%209.png)

![Untitled](/images/malloclabf3e/Untitled%2010.png)

![Untitled](/images/malloclabf3e/Untitled%2011.png)

![Untitled](/images/malloclabf3e/Untitled%2012.png)

![Untitled](/images/malloclabf3e/Untitled%2013.png)

그림만봐도 이해가 가니 넘어가자.

![Untitled](/images/malloclabf3e/Untitled%2014.png)

끝으로 이런 의문을 남긴다.

위 케이스를 보면 footer의 사이즈가 직접적으로 사용되는건 footer가 해당하는 블럭이 freed 블럭일 때만이다 따라서 할당된 블럭의 footer를 줄여볼 여지가 있는거다.

앞 뒤 블럭의 할당 상황을 freed 블럭에 기록한다면 할당된 블럭에 footer를 남기기 위한 추가적인 메모리를 아낄 수 있다. 

freed 블럭에서는 footer를 가져야한다는 문제가 있다고는하지만 freed 블럭은 말그대로 현재 사용되지않는 메모리기에 메모리 관리자 측면에서 해당 부분을 사용해도 딱히 문제는 없다.(사용자입장에선 이게 이제 쓰레기값이 되는 것/ 근데 애초에 freed된 블럭이기에 여기에 접근한다는것 자체가 Use After Free이다. ) 그래서 footer자체만을 위한 블럭에 추가적인 메모리 할당을 없앨 수 있다.

이 방식은 explicit list에서도 비슷하게 사용할 수 있다.

# explicit

![Screen Shot 2023-01-29 at 4.48.51 PM.png](/images/malloclabf3e/Screen_Shot_2023-01-29_at_4.48.51_PM.png)

implicit의 별로인점은 freed블럭을 찾기위해 전체를 순회해야한다는 점 explicit은 다음 freed 블럭을 가르키는 포인터를 저장해서 바로바로 이동할 수 있게 하는 것이 차이이다.

![Screen Shot 2023-01-29 at 4.53.59 PM.png](/images/malloclabf3e/Screen_Shot_2023-01-29_at_4.53.59_PM.png)

ppt에서 설명하기로 포인터를 앞뒤로 두개를 저장한다. 

Q 그러면 저장에 필요한 용량보다 작은 사이즈를 할당달라고 할때 처음 블럭을 만들때 어떻게하나요?

→ 이 저장을 할 수 있는 최소 사이즈를 할당하는 거다.

현재에서 32bit기준으로 앞뒤 포인터 변수를 저장해야할 8바이트가 추가로 필요한다 malloc(2)를 호출하더라도 8바이트를 할당해서 반환하는거다(참고로 해당구조에서 헤더 푸터로 4바이트씩먹으니 이 블럭의 최종 할당 바이트는 16바이트가 된다.)

해당 구조에서 블럭이 나눠지는 경우의 구조도

![Screen Shot 2023-01-29 at 5.00.49 PM.png](/images/malloclabf3e/Screen_Shot_2023-01-29_at_5.00.49_PM.png)

회색 칠해진 부분은 사용자에게 할당하는 경우고 남은 흰블럭들에대해서 더블 링크드 리스트의 구조를 유지한다.

해당 블럭이 그대로 freed 되고 붙어있는 블럭을 합치는 작업을 한다면 다시 반대로 될 것이다.

![Screen Shot 2023-01-29 at 5.06.24 PM.png](/images/malloclabf3e/Screen_Shot_2023-01-29_at_5.06.24_PM.png)

explicit 방식으로 할 경우  새 블럭을 어디에 끼워넣을까하는 정책적인 고려가 필요하게 된다.

Address ordered policy에 대해서 의문을 품을 수 있는데 진짜 이 방식대로만 그대로 하는게 arrange했을때 얻을 수 있는 장점이 있다. 

탐색의 경우에 적절한 트리구조를 사용하면 search, 삽입에 빠른 속도를 가질 수 있다. 조금 특별한 방식을 취한게 segrated list

![Screen Shot 2023-01-29 at 5.21.07 PM.png](/images/malloclabf3e/Screen_Shot_2023-01-29_at_5.21.07_PM.png)

가장 일반적인 free 상황

![Screen Shot 2023-01-29 at 5.30.22 PM.png](/images/malloclabf3e/Screen_Shot_2023-01-29_at_5.30.22_PM.png)

![Screen Shot 2023-01-29 at 5.40.31 PM.png](/images/malloclabf3e/Screen_Shot_2023-01-29_at_5.40.31_PM.png)

![Screen Shot 2023-01-29 at 5.42.42 PM.png](/images/malloclabf3e/Screen_Shot_2023-01-29_at_5.42.42_PM.png)

free 될 때 해당 블럭 앞뒤로 인접한 블럭들이 freed 블럭이면 Coalescing을 고려해야한다

![Screen Shot 2023-01-29 at 5.52.02 PM.png](/images/malloclabf3e/Screen_Shot_2023-01-29_at_5.52.02_PM.png)

![Screen Shot 2023-01-29 at 5.54.44 PM.png](/images/malloclabf3e/Screen_Shot_2023-01-29_at_5.54.44_PM.png)

![Screen Shot 2023-01-29 at 6.00.21 PM.png](/images/malloclabf3e/Screen_Shot_2023-01-29_at_6.00.21_PM.png)

# 종장

LFH같은것도 있지만 넘어가자.

<iframe width="560" height="315" src="https://www.youtube.com/embed/wB74q02x_P0" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>