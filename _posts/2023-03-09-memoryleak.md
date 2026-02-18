---
layout: post
title: "서버야 아프지마 - 신입 개발자를 위한 윈도우 메모리릭 디버깅"
date:   2023-03-09 19:26:01 +0900
tag: notes
---


언젠가 필요할지도..

<iframe width="560" height="315" src="https://www.youtube.com/embed/HH5gW5ov-78" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>


![Untitled](/images/serveritai/Untitled.png)

![Untitled](/images/serveritai/Untitled%201.png)

![Untitled](/images/serveritai/Untitled%202.png)

메모리가 모자른 경우는 실제로 물리메모리가 부족한 경우이지만 많은 경우에 가상메모리가 모자란 상황이다.

# os 가상메모리

![Untitled](/images/serveritai/Untitled%203.png)

![Untitled](/images/serveritai/Untitled%204.png)

# 프로세스 가상 메모리

![Untitled](/images/serveritai/Untitled%205.png)

한 프로세스에 할당할 수 있는 논리적인 주소공간

![Untitled](/images/serveritai/Untitled%206.png)

(강연당시)마비노기는 32bit 프로세스여서 4GB까지 할당이 가능하다.

![Untitled](/images/serveritai/Untitled%207.png)

![Untitled](/images/serveritai/Untitled%208.png)

그런데 실제로는  커널모드에서 2GB를 사용하기에 유저모드에서는 2GB밖에안된다.

![Untitled](/images/serveritai/Untitled%209.png)

![Untitled](/images/serveritai/Untitled%2010.png)

![Untitled](/images/serveritai/Untitled%2011.png)

![Untitled](/images/serveritai/Untitled%2012.png)

![Untitled](/images/serveritai/Untitled%2013.png)

실제로 RAM에 올라가있는 메모리 크기

![Untitled](/images/serveritai/Untitled%2014.png)

페이징까지 이루어진상황(Disk + RAM)

![Untitled](/images/serveritai/Untitled%2015.png)

![Untitled](/images/serveritai/Untitled%2016.png)

![Untitled](/images/serveritai/Untitled%2017.png)

![Untitled](/images/serveritai/Untitled%2018.png)

![Untitled](/images/serveritai/Untitled%2019.png)

![Untitled](/images/serveritai/Untitled%2020.png)

![Untitled](/images/serveritai/Untitled%2021.png)

![Untitled](/images/serveritai/Untitled%2022.png)

스킬창과 관련된 코드를 모두 뒤지면 잡을 수 있다.

그런데 너무 많아서 시간이 오래걸린다.

따라서 

![Untitled](/images/serveritai/Untitled%2023.png)

어디서 메모리가 새는지 찾는게 중요

![Untitled](/images/serveritai/Untitled%2024.png)

여기서 어디서는 콜스택을 의미

**어느 콜스택에서 메모리가 새는지 찾는게 중요**

![Untitled](/images/serveritai/Untitled%2025.png)

1. 재현전 메모리 할당량을 스냅샷을 찍음

1. 그 후 스냅샷을 다시 떠서 2번과 비교

이걸 수행하는 툴 

![Untitled](/images/serveritai/Untitled%2026.png)

![Untitled](/images/serveritai/Untitled%2027.png)

![Untitled](/images/serveritai/Untitled%2028.png)

![Untitled](/images/serveritai/Untitled%2029.png)

![Untitled](/images/serveritai/Untitled%2030.png)

메모리 사용량 자체를 추적하는 용으로도 쓸 수 있음.

![Untitled](/images/serveritai/Untitled%2031.png)

최근엔 VS에서 메모리 스냅샷 기능을 지원해서 똑같이 할 수 있다.

![Untitled](/images/serveritai/Untitled%2032.png)

![Untitled](/images/serveritai/Untitled%2033.png)

![Untitled](/images/serveritai/Untitled%2034.png)

![Untitled](/images/serveritai/Untitled%2035.png)

![Untitled](/images/serveritai/Untitled%2036.png)

![Untitled](/images/serveritai/Untitled%2037.png)

![Untitled](/images/serveritai/Untitled%2038.png)

덤프를 활용

![Untitled](/images/serveritai/Untitled%2039.png)

![Untitled](/images/serveritai/Untitled%2040.png)

![Untitled](/images/serveritai/Untitled%2041.png)

![Untitled](/images/serveritai/Untitled%2042.png)

사이즈 별로 할당횟수를 통계를 내고 가장 많은 횟수가 릭이 날 확률이 높다.

![Untitled](/images/serveritai/Untitled%2043.png)

![Untitled](/images/serveritai/Untitled%2044.png)

![Untitled](/images/serveritai/Untitled%2045.png)

![Untitled](/images/serveritai/Untitled%2046.png)

48바이트짜리 메모리 할당을 모두 뒤진다

여기서 케이스가 갈리는데

![Untitled](/images/serveritai/Untitled%2047.png)

![Untitled](/images/serveritai/Untitled%2048.png)

![Untitled](/images/serveritai/Untitled%2049.png)

![Untitled](/images/serveritai/Untitled%2050.png)

주소를 집어넣고 캐스팅을 하면 정보를 알 수 있게된다.

![Untitled](/images/serveritai/Untitled%2051.png)

메모리 할당 해제 비용이 크기때문에 대부분은 메모리 풀을 사용한다.

![Untitled](/images/serveritai/Untitled%2052.png)

이때 2.9GB 누수가 스트링 누수였던것

누수된 스트링을 찾는 법

![Untitled](/images/serveritai/Untitled%2053.png)

할당받은 공간을 모두사용하면 그 위의 공간을 새로 할당받아(커밋) 사용

![Untitled](/images/serveritai/Untitled%2054.png)

메모리가 할당될수록 주소가 커짐

![Untitled](/images/serveritai/Untitled%2055.png)

![Untitled](/images/serveritai/Untitled%2056.png)

이 영역에 써져있는 메모리를 읽어보자.

![Untitled](/images/serveritai/Untitled%2057.png)

조진케이스

![Untitled](/images/serveritai/Untitled%2058.png)

![Untitled](/images/serveritai/Untitled%2059.png)

![Untitled](/images/serveritai/Untitled%2060.png)

누수 사이즈가 유니크한 경우 4000byte짜리 버퍼 or 380byte짜리 객체 였다하는 경우

해당 함수를 이용하여 같은 사이즈의 객체가 어디서 할당되는지 추적이 가능하다

![Untitled](/images/serveritai/Untitled%2061.png)

![Untitled](/images/serveritai/Untitled%2062.png)



---

추가로 같이 보면 좋은 자료

- 넷마블 기술블로그 : 워라밸 브레이커, 메모리릭을 찾아라
  - https://netmarble.engineering/break-the-memory-leak-1-4/
  - https://netmarble.engineering/break-the-memory-leak-2-4/
  - https://netmarble.engineering/break-the-memory-leak-3-4/
  - https://netmarble.engineering/break-the-memory-leak-4-4/