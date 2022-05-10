---
layout: post
title: "Introduction to Multithreading for Video Games"
date:   2022-05-09 19:26:01 +0900
tag: notes
---

<iframe width="560" height="315" src="https://www.youtube.com/embed/M1e9nmmD3II" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

![Untitled](/images/introducemultithreading/Untitled.png)

멀티스레딩이 어려운 이유 순서가 뒤죽박죽인 것을 3, 4개 섞인것을 잘 따라가지 못한다.

이렇게 가는 것이 코어수가 얼마가 나오든 사람들이 이해할 수 있는 방법이라는 내용

![Untitled](/images/introducemultithreading/Untitled%201.png)

CPU의 역사와 같이 비교해서 보면 좋다

90년대 초반 cpu하나일때

이때도 멀티스레딩을 하긴했다

Blocking 되는 불편함을 감추기 위해 멀티스레딩을 했음

대표적인 예 네트워킹, 파일 IO

![Untitled](/images/introducemultithreading/Untitled%202.png)

context switch 오버헤드가 존재.

![Untitled](/images/introducemultithreading/Untitled%203.png)

당시 게임 에서도 multithreading을 하긴 함 

위처럼 음악, 네트워킹 같은것들 worker threading으로 따로 뺐다.

왜 multithreading을 안했냐 

1. 성능이 안나옴
2. 그냥 안할수있으면 안하는게 제일 좋음
3. 2000년 중반까지 무어의 법칙덕에 그냥 cpu가 알아서 빨라졌으니 할 이유가 없었음

![Untitled](/images/introducemultithreading/Untitled%204.png)

cpu 속도를 높이는 것에 한계에 도달함 자체 성능을 높일 순 없으니 cpu에 코어를 두개를 달기 시작함

2005년 5월에 최초의 듀얼코어 출시 

이때부터 본격적으로 멀티스레딩을 하기 시작함

![Untitled](/images/introducemultithreading/Untitled%205.png)

기존엔진구조를 확 바꿀 순없으니

update와 render부분을 분리함.

![Untitled](/images/introducemultithreading/Untitled%206.png)

2008년 i7 나오면서 하드웨어 4개 하이퍼 스레딩 포함 8개 달려서 나옴 

옛날방식으로 돌리면 코어 두개만 먹고 25%먹고 욕 되게 처먹는 게임들 있었다 

PS3의  CPU구조가 괴랄스러웠다 

CPU 1 (PPE: Power Processor Element)

7개가 인텔과 호환되는 CPU가 아님 SPU라고 해서 (SPE Synergistic Processing Elements) 알수없는 어셈블리로 해야했음 

update가 render로 가르는게 불가능했음

XBOX360과 비교했을때 PS3로 나온 게임들이 엄청나게 구린게 많았다

PS3하나를 위해서 엔진을 뒤집어야했으니 무리가 많았음

코어수가 정말 많아질때도 쓸 수 있는 아키텍처를 만들기위해 새로운 스레드 방식을 고안함

![Untitled](/images/introducemultithreading/Untitled%207.png)

프로세스, action기반으로 스레드를 나누기보다 데이터를 위주로 나누자는 생각을 하게됨 

물체가 800개인데 코어가 8개라면 하나당 100개씩 던져주자

job Queue, Task system이라고 부름

OS에서는 Barrier lock이란 용어도 사용 

![Untitled](/images/introducemultithreading/Untitled%208.png)

![Untitled](/images/introducemultithreading/Untitled%209.png)

장점 race condition이 없다

사람이 추적이 가능함 

오버헤드가 굉장히 적다 context switch가 적다

해당 시스템에서 조금 더 발전하는 방법 

- job strealing : 먼저끝난 스레드가 다른 스레드 일을 가져와서 처리
- dependency graph

현재 돌아가는 기본적인 알고리즘 

parallelism의 기본적인 방식

C# parallel for , C++에 들어올 parallel for, async 모두 이방식으로 돌고있다.