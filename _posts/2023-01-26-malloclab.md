---
layout: post
title: "malloc lab 2. 구현"
date:   2023-01-25 19:26:01 +0900
tag: csapp
---

64bit 기준으로 했다.

대부분이 아마 32bit 기준으로 malloc lab이 작성되어있다. 

아마 makefile보면 대부분 -m32 옵션이 붙어있을거다.

64bit일때 조심해야하는건 다음과같다

1. 기준이 WORD 4byte가아닌 DWORD 8byte가 된다.
2. pointer 의 자료형 크기가 8byte가된다.
3. malloc 반환 주소의 align은 16byte가된다. 

해당 랩에서 가장 힘들었던것은 아무래도 malloclab자체 처음 세팅하는데 가장 시간이 오래걸렸다.

몇년동안 버전이 계속 바뀌다보니 갖가지 프로젝트들이 다 갈라파고스마냥 갈라져있어서 어떤게 적합한지 도저히 못찾았는데.. github에 있는것들은 대부분 완성된 파일들이고.. 
그래서 딱 시작할수있는 template 레포를 하나 완성해서 만들었다.


https://github.com/EeeUnS/malloclab


## 결과 창에 나오는 것이 의미하는 것들

[https://www.clear.rice.edu/comp321/html/laboratories/lab09/](https://www.clear.rice.edu/comp321/html/laboratories/lab09/)

사실 전체코드를 읽어보고 분석해봐도 된다.

## aslr 해제

테스트를  위해서 주소를 막 출력해볼텐데 주소가 실행마다 다르게 된다. 이는 보안정책인 aslr때문인데 밑의 방법을 따라하면 실행마다 주솟값 출력이 동일해서 디버깅할때 꽤나 편하게된다.

[https://comsec.tistory.com/91](https://comsec.tistory.com/91)

# 구현

내가 사용한 정책은 다음과 같다.

- segrated list를 짰다.
- 헤더 추가 8바이트가 붙는다.
- 푸터는 따로 없다.
- free block은 싱글 리스트이다
- 요구 메모리에 딱맞게 할당하는것이아닌. 2의 승수에 맞게 메모리를 맞춰서 할당했다.
    
    예를들어 7byte 할당해줘 하면 8바이트를 할당해준다. 17바이트 요구시엔 32바이트 
    
- free된 블록들은 stack 처럼 작동해서 리스트 맨앞에 놓고 요구시에 바로 반환한다.
- freed 된 블록들 주소에는 해제 블록 포인터를 기입해서 바로 접근할 수 있게한다.

![Untitled](/images/malloclab/Untitled.png)

여기의 explicit list 상황과 완전히 동일하다. 

- 여러개를 만들어놓는다 (고정메모리풀) 이때 기준은 1024byte로 해당 2승수로 나눈 갯수만큼 블럭들 갯수를 미리 할당한다. 이 1024byte를 넘어서면 블럭 수를 그냥 1개로 본다.

## 만난 버그

```
euns@LAPTOP-JLAHCORI:~/final$ ./mdriver
Team Name:EUnS
Member 1 [:yunselee:euns312510@gmail.com](mailto::yunselee:euns312510@gmail.com)
Using default tracefiles in ./traces/
Alloc mem brk 0x7fff77dcd820 old brk 0x7fff77dcd010 size 2064
Alloc mem brk 0x7fff77dce030 old brk 0x7fff77dcd820 size 2064
Alloc mem brk 0x7fff77dce4b8 old brk 0x7fff77dce030 size 1160
Alloc mem brk 0x7fff77dcf4c8 old brk 0x7fff77dce4b8 size 4112
Alloc mem brk 0x7fff77dd04d8 old brk 0x7fff77dcf4c8 size 4112
ERROR [trace 1, line 5]: Payload (0x7fff77dcd028:0x7fff77dcd81f) lies outside heap (0x7fff77dcd010:0x7fff77dcd00f)
Terminated with 1 errors
```

이런 에러가 있었는데 나의 경우 

1. 새로운 trace file을 실행할때 static 값들 초기화를 안 시켜줌. 이는 init함수에서 처리해준다.
2. 할당 범위를 넘어서서 공간을 만든 경우가 있었음.

pool size 로 1024바이트와 그를 넘었을때 블록 갯수를 지정해준것이다.

이를 바꿔가며 테스트해봤는데 +-10%정도 차이가 계속 나긴한다

```c
#define POOL_SIZE  1024
#define DEFAULT_BLOCK_NUM 1
```

# 결과

[https://github.com/EeeUnS/mymalloc](https://github.com/EeeUnS/mymalloc)

-O2 옵션

```cpp
Team Name:EUnS
Member 1 :yunselee:euns312510@gmail.com
Using default tracefiles in ./traces/
Measuring performance with gettimeofday().

Results for mm malloc:
trace  valid  util     ops      secs   Kops
 0       yes   94%    5694  0.000038 149842
 1       yes   89%    5848  0.000038 152292
 2       yes   95%    6648  0.000046 146110
 3       yes   95%    5380  0.000036 147397
 4       yes   49%   14400  0.000069 208092
 5       yes   74%    4800  0.000061  78689
 6       yes   75%    4800  0.000067  72180
 7       yes   96%   12000  0.000074 161290
 8       yes   89%   24000  0.000112 214095
 9       yes   29%   14401  0.000098 147551
10       yes   45%   14401  0.000043 334130
11       yes   60%      12  0.000000 120000
12       yes   92%      12  0.000000 120000
Total          76%  112396  0.000682 164683

Perf index = 45 (util) + 40 (thru) = 85/100
```

```cpp
Team Name:EUnS
Member 1 :yunselee:euns312510@gmail.com
Using default tracefiles in ./traces/
Measuring performance with gettimeofday().

Results for mm malloc:
trace  valid  util     ops      secs   Kops
 0       yes   94%    5694  0.000197  28948
 1       yes   89%    5848  0.000150  38935
 2       yes   95%    6648  0.000175  38054
 3       yes   95%    5380  0.000138  39127
 4       yes   49%   14400  0.000361  39878
 5       yes   74%    4800  0.000191  25118
 6       yes   75%    4800  0.000198  24206
 7       yes   96%   12000  0.000272  44166
 8       yes   89%   24000  0.000424  56604
 9       yes   29%   14401  0.000230  62722
10       yes   45%   14401  0.000146  98976
11       yes   60%      12  0.000000  24000
12       yes   92%      12  0.000000  30000
Total          76%  112396  0.002481  45297

Perf index = 45 (util) + 40 (thru) = 85/100
```

속도면에 점수가 40점 만점으로 그 위로 점수 할당이 안이루어지고있다..

그래서 공간 활용도가 떨어진다고 만점이 안나오는 상황이다.

대충 검색해서 본 다른 사람들 점수랑은  sec면에서 10배정도까지 빠른거같은데 다 같은 만점이 나와서..

메모리 활용면에서는 성능좋은 균형 트리써서 (red block tree등) 공간분할하고 적절하게 메모리 합치는게 아마 메모리자체는 가장 컴팩트하고 효율좋게 쓸 수 있을 것이다. (거기들어가는 부하는 모르겠지만..)

재미있었던 점하나는 외부로 노출되지않는 함수들을 static 으로 바꿔주고 변수들에 const 마저 좀 붙여줬더니 디버그 버전 성능이 20%정도 향상했다 -_-;