---
layout: post
title:  "빌드 시간을 줄이기 위한 노력들"
date:   2024-09-03 19:26:01 +0900
tag: etc
---


- [C언어가 cpu에 작동하기까지](https://eeeuns.github.io/2021/10/27/understandingcomputer/)
- [C언어가 cpu에 작동하기까지 심화 : 컴파일](https://eeeuns.github.io/2024/08/09/understandingcomputercompileo/)
  - [빌드 시간을 줄이기 위한 노력들](https://eeeuns.github.io/2024/09/03/reducebuild/)
  - [전방선언 : 컴파일러가 필요한 정보](https://eeeuns.github.io/2025/11/24/forwarddeclaration/)



## 파일 수정 개별 빌드

이미 빌드가 다된 프로젝트에서 파일 하나 코드 한줄을 수정한다고 생각해보자.

이때 모든 파일을 전부다 컴파일새로해서 링킹까지해서 빌드를 하면 굉장히 시간이 오래걸릴거다.

**변경된 파일만 빌드를 하고 링킹을하는거다.**

5000 파일중 1파일 수정했다고 5000파일 전체를 새로 컴파일하는것보다는 기존에 있는 obj 파일은 냅두고, 1파일만 컴파일 + 링킹만하면된다.

이 방식도 생각보다 효과가 크지 않을 수 있는데 바로 헤더 파일이 수정되는 경우다.

헤더파일이 여러 파일에 include 되어있는 딸린 파일을 죄다 빌드해야하고(이게 헤더파일 안에서 include를 최대한 덜려는 큰 이유 중 하나), 심할경우 전체 빌드의 80퍼센트까지 새로 빌드해야할 수도, 최상위 stdafx.h 내부의 파일을 수정하는 경우 꼼짝없이 풀빌드를 해야할 수도 있다.

# **Visual Studio C++ Build Insights**

최근 Visual Studio에 Cpp에 재미있는기능이 많이 들어왔는데 그 중하나로 컴파일 시간을 분석할 수 있는 툴이 정식으로 들어왔다. (툴 자체는 19년부터 있었던걸로 추정되나 VS에 정식 기능으로 들어오기까지는 많은 시간이 걸렸다)

[https://learn.microsoft.com/ko-kr/cpp/build-insights/get-started-with-cpp-build-insights?view=msvc-170](https://learn.microsoft.com/ko-kr/cpp/build-insights/get-started-with-cpp-build-insights?view=msvc-170)

![a-screenshot-of-a-computer-description-automatica-1.png](/images/reducebuild/a-screenshot-of-a-computer-description-automatica-1.png)

![Screenshot-5.png](/images/reducebuild/Screenshot-5.png)

직접 사용해본 경험으로 굉장히 많은걸 파악할 수 있다.

- 헤더파일이 어떤 파일에 include 되고 토탈 걸린 컴파일 시간이 얼마나 걸리는지
- 함수 별로 함수가 얼마나 오래 걸리는지
- 특정 템플릿이 얼마나 걸리는지, 각 생성되는 각 타입들

등 직접적으로 파악 할 수 있다.



# 전방선언(**Forward declaration)**

[https://en.cppreference.com/w/cpp/language/class](https://en.cppreference.com/w/cpp/language/class)

전방선언은 사실 컴파일 속도만을 위한 이유는 아니라 순환참조를 직접적으로 해결하기 위한것도 이유가 되지만 컴파일 속도와도 관련이 많다.

이런 클래스가 존재한다 라는 지시문으로, 이렇게 하면 실제 클래스 전체가 필요하지 않고 간단하게만 표시하고 넘어가서 링킹때 찾아 연결한다.

1. 컴파일시 개별 파일의 크기 자체가 줄어들어 효과가 크다.
2. include 로 엮여있는 최상위파일이 변경될때 연관된 모든 파일이 컴파일을 새로해야하기에 이런 의존성이 제거되는게 이득

특히 헤더파일 안에서 헤더파일을 다시 include하는 상황을 제거할때 가장 효과적이다.

프로젝트가 커지면 전방선언을 떠나서 필요없는 include가 존재하는 케이스가 존재한데 이런걸 체크하여 없애주는 기능도 VS에는 최근에 들어왔다

- [https://devblogs.microsoft.com/visualstudio/visual-studio-17-8-now-available/#clean-up-and-sort-include-directives](https://devblogs.microsoft.com/visualstudio/visual-studio-17-8-now-available/#clean-up-and-sort-include-directives)


# stdafx.h

너무 많이 중복컴파일되는 헤더파일을 해결하기 위한 방법

중복되는 헤더파일들은 한곳에 뭉친다음 요거 **한번만 컴파일**하고 이 헤더파일 컴파일한걸 나머지 cpp파일에 복사 붙여넣어서 컴파일 하면 성능이 빨라지지 않을까 하는 아이디어에서 나온 방법

중복되는 헤더파일들을 stdafx.h에 인클루드한다

그러면 컴파일시에  

stdafx.h을 include한 stdafx.cpp 파일을 가장 먼저 빌드한다.

그 후 다른 파일들을 빌드하는데 이때 stafx.h에 include된 파일들은 새로 빌드하지 않고 이미 빌드한 파일을 위에 얹는 식이다.

stafx.h에 인클루트 한 파일을 여러번 빌드하지않으니 빌드시간을 아끼는셈,

아래의 유니티 빌드를 보면 소형 유니티 빌드 헤더파일 판 라고 생각할 법하지만 단점도 똑같이 공유한다.

단점은 **stdafx.h안에 include하는 파일이 수정될때 전체 빌드를 수행하게된다.**



# pimpl 패턴

헤더파일 크기 자체를 줄여서 해결하는 방법

이런 헤더 파일이 있다고 생각해보자

```jsx
Sample.h

class Sample
{
public :
~~
private
	void function()
	... 약 이천개의 함수
	
	int a;
	... 약 이천개의 멤버 변수
}
```

이 private의 코드들을 싹다 내부 class impl로 새로 정의하여 Sample.cpp에 정의하고 

헤더파일을 아래처럼 남겨놓는다면,

```jsx
Sample.h

class Sample
{
public :
~~
private
	impl *a;
}
```

위의 2천개의 함수, 변수들을 모두 빼버려 헤더파일 4천줄을 줄일 수가 있다.

그럼 요 파일을 include하는 모든 cpp 파일의 길이가 4천줄 줄게 되는것이다.

그럼 각 컴파일해야하는 cpp 크기 자체가 주니 전체적인 컴파일 속도의 향상을 가져온다 라는 아이디어이다.

단점은

- impl 이 파일을 **동적할당을 해야한다**는 것 이로인해 객체 접근시 메모리가 추가적으로 분리된다는것, 메모리 추가 접근을 통한 직접적인 성능하락이 발생한다
- impl 에서 또 변수를 간접 호출하게 되는 코드상의 불편함이 생긴다는것,
- 디버깅시에도 watch 창에서도 한칸 더 들어가서 봐야한다는것 모든 작업에서 뎁스가 하나씩 늘어나는거다.

성능적으로도, 이래저래 편리성측면에서도 많이 불편해진다.

# 유니티 빌드

중복을 없애기 위해 cpp 한 파일로 싸그리 모아서 요 파일 하나만 컴파일 하는 방법

![Untitled](/images/reducebuild/Untitled%201.png)

이러면 중복 라인에 대한 컴파일이 나올 수 없다

모든 작업 파일을 단 하나의 cpp 파일로 모은 다음 

**cpp 단 하나만 컴파일 하자**는게 본 취지.

![Untitled](/images/reducebuild/Untitled%202.png)

include는 앞서 말했지만 진짜 복사 붙여넣기다. cpp 파일을 최종적으로 빌드할 한 파일에 죄다 include하자.

visual studio에도 해당 기능이 시험적으로 들어와있다.

- [https://devblogs.microsoft.com/cppblog/support-for-unity-jumbo-files-in-visual-studio-2017-15-8-experimental/](https://devblogs.microsoft.com/cppblog/support-for-unity-jumbo-files-in-visual-studio-2017-15-8-experimental/)

단점 : 

1. 전역, static 함수 변수 define들이 마구마구 충돌이 날 가능성이 매우 높다.
2. 파일 수정시 리빌드와 동일한 시간이 걸린다.


# 빌드 툴

- ex) incredibuild

![Untitled](/images/reducebuild/Untitled.png)

가장 무식하고 강력한… 방법

기본 아이디어는 다음과 같다.

cpp 파일이 너무 많아서 컴파일이 오래걸리면 cpp 파일을 여러 컴퓨터에 나눠서 나눠서 컴파일하자!

100개의 cpp 파일이 있고 컴퓨터의 cpu 코어가 20개라고하자 그러면 100개의 cpp파일을

내컴퓨터, 4개의 컴퓨터에 20개씩 나눠서 컴파일한다음 합쳐서 최종적으로 내 컴퓨터에 모은다음 링킹을 하면 되겠구나라는 가장 간단한 생각에 나온 방식이다.

여러 컴퓨터를 때려박아 하드웨어로 해결하는 방법

하지만 컴파일을 위해서 각 파일을 여러 컴퓨터에 전송을 해야한다

이로인해 코어가 모잘라 여러 파일이 한번에 컴파일 되지못하는 문제는 줄지만 trade off관계로 네트워크 지연 및 비용, 각 컴퓨터의 부하를 주게된다.

이게 생각보다 단순 빌드보다 장점이 없을 수도있다





# 모듈

이런 고전적인 컴파일 시간의 문제를 해결하고자 모던한 언어에서는 module 시스템을 도입되어있다.

묶여있는 헤더파일이아닌, 함수, 클래스 하나하나만을 include하여 사용할 수 있도록해서 기존 헤더파일의 불필요한 부분을 최대한 덜어서 사용하자는 취지다.

Cpp도 모듈이 들어왔으나… 생각보다 널리 쓰이는것같지는 않다...


---
참고자료


- [https://learn.microsoft.com/en-us/cpp/cpp/modules-cpp?view=msvc-170](https://learn.microsoft.com/en-us/cpp/cpp/modules-cpp?view=msvc-170)
- [https://stackoverflow.com/questions/22693950/what-exactly-are-c-modules](https://stackoverflow.com/questions/22693950/what-exactly-are-c-modules)

- build insight
    - [https://devblogs.microsoft.com/cppblog/introducing-c-build-insights/](https://devblogs.microsoft.com/cppblog/introducing-c-build-insights/)
    - [https://devblogs.microsoft.com/cppblog/build-insights-now-available-in-visual-studio-2022/#getting-started](https://devblogs.microsoft.com/cppblog/build-insights-now-available-in-visual-studio-2022/#getting-started)
    - [https://devblogs.microsoft.com/cppblog/functions-view-for-build-insights-in-visual-studio-2022-17-8/](https://devblogs.microsoft.com/cppblog/functions-view-for-build-insights-in-visual-studio-2022-17-8/)
    - [https://devblogs.microsoft.com/cppblog/templates-view-for-build-insights-in-visual-studio-2/](https://devblogs.microsoft.com/cppblog/templates-view-for-build-insights-in-visual-studio-2/)

- stdafx
    - [https://learn.microsoft.com/en-us/cpp/build/creating-precompiled-header-files?view=msvc-170](https://learn.microsoft.com/en-us/cpp/build/creating-precompiled-header-files?view=msvc-170)
- 유니티 빌드
    - https://www.slideshare.net/slideshow/ndc2010-unity-build/4335591
    - [https://netmarble.engineering/unity-build-can-pump-up-build-speed/](https://netmarble.engineering/unity-build-can-pump-up-build-speed/)
    - [https://onqtam.github.io/programming/2018-07-07-unity-builds/](https://onqtam.github.io/programming/2018-07-07-unity-builds/)

- pimpl 패턴
    - [https://blog.popekim.com/ko/2013/05/21/pimpl-pattern-me-no-likey.html](https://blog.popekim.com/ko/2013/05/21/pimpl-pattern-me-no-likey.html)
- 모듈
    - [https://learn.microsoft.com/en-us/cpp/cpp/modules-cpp?view=msvc-170](https://learn.microsoft.com/en-us/cpp/cpp/modules-cpp?view=msvc-170)