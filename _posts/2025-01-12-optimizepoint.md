---
layout: post
title:  "자료구조의 최적화 주요 포인트"
date:   2025-01-12 19:26:01 +0900
tag: cs
---

회사에 들어가기전엔 DB는 그렇게 중점적으로 공부하지 않았고, 로직적 성능을 신경썼었다. 

막상 회사오니 서버 부하에 반은 DB부하고 반은 로직 부하였다.

내가 겪은 다양한 부하 케이스와 성능 최적화에 대해서 쭉 정리를 해볼까 한다.

웹 서버들 쪽 성능논하는 얘기를 보면 항상 구조 얘기만 있고, 서버 자체의 로직적인 퍼포먼스 얘기에 대해서는 크게 안보여서 아쉽다싶다.

![image.png](/images/optimizepoint/image.png)

![image.png](/images/optimizepoint/image2.gif)


1. 캐시 메모리
2. 램
3. 하드디스크( 및 SSD)
4. 네트워크 전송

분산 DB에 대해서는 지식이나 경험이 크게 없기에, 

앞에 얘기 나왔던 DB부하에 대해서 먼저 얘기를 해보면

- 특정 작업을 위해서 DB작업이 여러번 필요한경우 sp사용을 꾀할 수 있다.
sp 로직은 결국은 DB단에서 직접적으로 움직이는 로직이고, 코드에서 db호출이 여러번 되는것은 결국 DB접근을 반복적으로 한다는 것이고 왔다갔다하는 불필요 작업이 발생하는것이다.
두번째로 긴 쿼리를 통신하던 상황에서 이를 sp로 바꾸면 네트워크 비용의 감소 또한 꾀할 수 있다

이렇게 sp가 갈려서 관리 포인트가 갈린다는건 결론적으로 유지보수의 어려움이 증가한다.
- 인덱싱은 너무 기본이니 생략..

서버가 단순하고 DB상하차만을 할 수록 대부분 케이스에 주요 병목 지점이 DB부하가 된다.

DB부하가 너무 커서 감당이 불가능할때 캐시레이어를 박기시작한다.

그럼으로 stateless 서버 완성

disk 접근 부하를 메모리 접근 비용으로 줄이고, 캐시서버 사망시 롤백이라는 짊을 지는것으로 trade off를 한다.

로직 최적화를 꾀할때는 다양한 방법이 있고, 아이디어 영역이라고 생각하지만 크게 나누면 두 가지라고 본다.

1. 병목 로직 호출 자체가 없도록 한다.
2. 돌아가는 코드 자체를 그냥 빠르게 한다.

1번은 말그대로 기존 주요 병목 함수를 10번 호출하는 방식에서 1번만 호출하게 변경하는 방식이다. 

예를들면 기존 스펙에서 한개를 선택해서, 진행하는 로직을 스펙을 N번으로 바꾸면서 그냥 패킷만 N개 던지게 하거나 처리로직을 그냥 *N만 하게한다던가 이런 식의 작업을 하면서 부하가 생기면 그냥 그 작성을 새로하면 된다.

그리고 호출할 필요가 없으면 호출하지 않게 (early return등) 한다던가 하는 방식으로 해당 문제를 근본적으로 고칠 수도있지만, 

해당 로직 및 관련 스펙을 빠삭하게 파악하고 있어야 진행할 수 있어 작업이 꽤나 어렵다. (가능한지 부터 견적을 봐야함)

전체 코드 로직을 모르는 나로서는 2번 방식을 우선적으로 고려하게 되는데

이중 자료구조에 대해서 좀 깊게 풀어보고자 한다.

# 자료구조 (시간복잡도의 함정)

1. 배열
2. 해시테이블 (std::unorderd_~)
3. 트리구조 (std::map, set)

자료구조를 논할때 꼭 같이 논해야하는 것이 있다.

캐시메모리 효율과, 메모리 할당 해제 비용이다.

메모리 할당 해제비용은 O(1)이라고 퉁치고 넘어가기엔 너무 큰 비용으로 시간복잡도는 결국 n이 클 수록 이에 대한 영향이 크게 되지만, 많은 경우에,
이를 논하기 이전에 그냥 단일 연산자체가 병목지점을 만들 수 있다는걸 명심해야한다.

이 중 배열의 메모리 생성 해제 비용이 가장 싸며 캐시 메모리 접근 효율 또한 가장 좋다.

초기 메모리를 미리 예측하여 확보할 수도있다.

일반적으로 키밸류 방식 자료구조를 사용하게 되므로 뇌를 비우고는 그냥 해시테이블을 사용하는것을 권장한다.

순회시 정렬 보장이 필요할때 map, set 을 고려할 수 있는데,

트리 구조의 메모리 생성 해제 비용은 가히 최악이다.

시간복잡도를 따지기 이전에 그냥 기본 성능자체가 느리다.

tree구조 map set은 아래 이유가 아니면 마지막의 마지막 선택지로 남겨놔도 괜찮다고 생각한다.

1. 정렬을 보장
2. 사용 구조가 N이 정말 크고 중간 삽입 삭제가 정말 빈번히 발생
3. 위의 케이스에서 전역적으로 접근하는 변수인 경우

저기서 좀 더 빠른 최적화를 꽤 한다면, 배열사용을 고려해볼법하다.

만능 사용은 힘들지만, 배열의 효율은 배열에 들어갈 N과 원소로 들어갈 객체의 size가 작을 수록 효율이 커진다. 

가변 배열인 벡터 구조의 경우 처음 생성시에 사이즈를 미리 크게 할당할 수도 있어 추가적인 메모리 비용을 줄여 보일 수도있다.

그러곤 무식하게 보일 수 있지만, 그냥 전체 순회를 한다던가.

정렬이 필요하다면 차라리 필요시점에서 객체를 직접 정렬해서 사용하던가 하는 방식으로도 가능하다. 캐시 미스로 인한 RAM 접근 비용을 꼼꼼히 따져야한다.

경험적으론 리스트 자료구조를 사용해야겠다 판단이 든 케이스는 없었고, 벡터(배열) 메모리 할당을 위한 복사 성능 성능개선을 위해 리스트로 바꾼 케이스는 역으로 성능저하를 불러왔다.

**자료구조 선택은 들어가는 들어갈 자료의 크기, 삽입, 삭제, 순회, 단일 접근 비용의 빈도를 모두 고려해야한다.** (여기서 삽입 삭제 비용은 객체의 생성 해제 비용과도 직결된다.)

# 컴퓨터 구조

이 정도까지 고려해서 작업하는 경우는 나의 경우에 없다보니 경험적으로는 없다.

그래픽스로 가면 컴퓨터 구조까지 따지면서 성능을 따지면서 최적화를 하는 경우도 많이 보는데, 특히 하나의 함수를 몇천번씩 호출하다보니, 그 함수의 최적화 되지않은 코드로직이 크게 영향이 가게된다. 이런 경우에 컴파일 최적화의 결과물까지 신경써서 해야한다고한다.

- 캐시 메모리 접근을 조금 더 신경쓰기
    - false sharing 정도 고려, 구조체 크기 64byte 메모리에 맞추기
    - 16byte 밑의 객체는 함수 파라미터 전달시 포인터가아닌 단순 복사 전달
- if 로인한 성능 저하 고려
- SIMD 최적화

# 파일 입출력

단순 파일 파싱하는 일회성 코드작업을 하는데, 복잡한 파일 구조 엑셀 등을 작업하거나, 

복잡한 형식으로 파일을 읽어온다거나 할때 파일이 클수록 성능저하가 크게 오는데,

많은 경우에 코드 로직적으로 raw 구조로 읽고 필요 변수들을 직접 파싱하는 식으로 변경한다면 시간을 획기적으로 줄일 수 있다.

방식에 따라 50배 100배까지 시간 차이가 발생할 수 있는 부분이라, 

해당 파싱 동작시간이 업무에 영향이 가거나, 
이 문제로인해 해당 작업이 가능 불가능을 논해야할정도면 신경써볼법하다. 

과거에 엑셀 파일 파싱이 필요했는데, 흔히 검색해서 나오는 엑셀 파일 파싱하는 라이브러리를 가져와서 사용하려니 너무 느려서 처리가 불가능할정도라 csv파일로 뽑아내서 처리했다.