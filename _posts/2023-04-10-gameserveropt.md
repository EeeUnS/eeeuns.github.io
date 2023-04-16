---
layout: post
title: "socket + C/C++기반의)실시간 게임서버 최적화 전략"
date:   2023-04-10 19:26:01 +0900
tag: note
---

어렵다

<iframe width="560" height="315" src="https://www.youtube.com/embed/LBo_rKN_e-I" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

전통적인 온라인 게임 

TCPIP Native 작업

최적화 작업

![Untitled](/images/gameserveropt/Untitled.png)

# 서버가 하는 일

![Untitled](/images/gameserveropt/Untitled%201.png)

결과만 저장해주는 것 

DB의 미들웨어 개념

![Untitled](/images/gameserveropt/Untitled%202.png)

능동적인 : 게임서버안에 메모리에 실시간으로 모든 플레이어의 상태를 추가

이론적으로 적대 크래시나지않고 전원공급이 무한하고 하드웨어도 고장나지않으면 DB가 없어도 말이 됨  

렌더링 루프와 다르게 클라에서 게임루프가 30프레임 돌아간다고하면 서버도 적어도 30프레임 돌아가주는 다수의 클라이언트들을 게임서버 안에서 시뮬레이션 하고있는 상태

![Untitled](/images/gameserveropt/Untitled%203.png)

소극적으로 능동적 : 충돌처리 전투판정

클라에서 전투판정 이동판정 다하고 서버에 결과만 보내주는 식

서버 : 패킷중계, 데이터 저장

위조된 패킷, 정보 (해킹)에 취약함

![Untitled](/images/gameserveropt/Untitled%204.png)

내가 만든식 

1000명 접속시 서버안에 천명분에 해당하는 뭔가 만들어지고 1000명을 시뮬레이션

이론상 달리기 시작햇을때 패킷을 두번만 보냄 달렸을때 멈췄을때

시뮬레이션 주체 : 클라이언트, 서버, 나를 바라보고있는  n 개의 클라이언트

해킹에 강함, 서버에서 할 수 있는게 많음

단점 : 부하가 큼.

플레이어를 바라보는 다른 클라이언트들도 추가적인 부하가 들어감

![Untitled](/images/gameserveropt/Untitled%205.png)

![Untitled](/images/gameserveropt/Untitled%206.png)

처리량 : 시간당 패킷 요청 처리

응답성 : 패킷 하나의 처리 시간

![Untitled](/images/gameserveropt/Untitled%207.png)

서버의 응답성이 떨어지지않게하려면 플레이어 A가 보낸 패킷 작업을 처리한다고 서버가 멈칫거리는 동안 플레이어 B의 패킷을 처리못하고 기다리고있으면 반응성이 떨어진다고 느낌

최대한 유저의 요청에대해서 빠르게 반응해야하고 어떤 작업을하는 것 때문에 플레이어 나한테 반응을 못해주는게 응답성이 떨어지는 상황 

플레이어 a가 업무를 요청했는데 1분간 연산하고 있다 서버가 1분간 멈추면 난리가 날거니 비동기로 빼야함 

PC의 os에 드라이버 쪽보면 레벨이 존재

IRQ라고 interupt request level이라고 우선순위가 존재 

드라이버는 일반적으로 실행하는 유저모드 어플리케이션보다는 우선순위가 높지만 

마우스를 움직이는데 인터럽트 발생해서 인터럽트 핸들러가 호출될거고 근데 여기서 핸들러 처리한다고 컴퓨터가 멈출순 없으니 

보통 인터럽트 핸들러에서는 최소한의 작업만하고 DPC란 형태로 빼놓음 

긴급한 인터럽트들을 처리하고 나서 유휴시간이 발생하면 DPC의 작업들을 처리하기 시작 

서버도 동일

async 작업으로 뺌

스레드 풀로 많이 구현해서 작업을 병렬로 처리하도록 설정 

아니면 기본적인 코드 최적화 

![Untitled](/images/gameserveropt/Untitled%208.png)

대표적인 케이스 로그인 로그아웃 

온라인 게임 초창기에 문제가 됐던 것들을 한 2000년대 중반 정도까지는 게임업계에서 대부분 해결을 해서 알아서 잘하고있었는데 지금은 오히려 퇴보하는 듯함 

지금와서 옛날 방식으로 소켓 연결을하는 PC나 콘솔을 리얼타임 서버 연결해서 돌아가는 mmo를 만든다고하면 상당수 업체들이 2000년대 초반까지 했던 삽질을 다시 반복할거라 봄

웹서버 연결해서 하는 게임들은 이런 문제들을 아예 고려하지않음

로그인을 예시로

로그인을 하면 로그인 정보, 패스워드 일치여부 등을 DB에서 가져옴 

DB가 옛날엔 정말 느렸는데 

옛날에 서버가 이런처리를 정말 쌩 싱글스레드로 처리하면 그 순간에 날아오는 다른 패킷들을 하나도 처리를 못함 10초씩 걸리면 10초씩 멈춰있는걸로 보임

로그인은 긴급하게 할필요없으니 별도의 스레드로 빼거나 DB자체도 비동기 억세스할수있는 api들이 있으니 그런것들을 사용하도록하고 pooling을하든 이벤트 드리븐으로 하든 그때가서 작업 마무리 하는식으로 처리하면 가능

아이템 흭득 또한 중요하지만 0.1ms를 다투어 처리할만큼 중요하진 않으니  

먹으면 먹은거고 사용한거만 확실하게 처리하면 되니 db에서 제거하거나 db에서 추가하고 그결과를 받아서 메모리에 캐싱하는 거는 비동기로 빼야함

DB 의존적인 것들 

![Untitled](/images/gameserveropt/Untitled%209.png)

![Untitled](/images/gameserveropt/Untitled%2010.png)

![Untitled](/images/gameserveropt/Untitled%2011.png)

밀리는 문제가 생김 

![Untitled](/images/gameserveropt/Untitled%2012.png)

1000개를 나눠서 쑤셔넣기 

걸리는 총시간은 동일 

어느 한시점에도 응답성을 떨어뜨리지 않고 그 작업을 다 완료할 수 있음 

별도의 멀티스레드를 사용하지않아 실수할 가능성이 적음 

다른 캐릭터를 불러온다던가 할때 

사실상 수동 스케줄링 

![Untitled](/images/gameserveropt/Untitled%2013.png)

현재 스레드가 아닌 다른 스레드에 맡기는 것 

![Untitled](/images/gameserveropt/Untitled%2014.png)

동기화 문제 race condition 문제 

너무 적극적으로 사용하는건 그닥 좋진 않음 

![Untitled](/images/gameserveropt/Untitled%2015.png)

같은 종류의 작업을 다수의 스레드가 달려들어서 빠르게 처리하는 것 

처리량 때문에 느려지는 경우 

투입한 코어/스레드 갯수만큼  줘서 처리

![Untitled](/images/gameserveropt/Untitled%2016.png)

![Untitled](/images/gameserveropt/Untitled%2017.png)

수학함수 별로없으면 의미 없음 

까놓고 말해 메모리 힙 할당 해제를 안하게 짜는게 가장 중요 

보통 해제는 두배이상 느림

커스텀 메모리 풀 이용하면 보통 5~10배 빨라짐 

의도치않게 호출하는 STL을 제거하면 빨라짐 

STL걷어내고 상당부분 직접 자료구조 짜면 빨라짐 

![Untitled](/images/gameserveropt/Untitled%2018.png)

현재 가볍고 빠른 lock 많이 제공해줘서 좋음.

![Untitled](/images/gameserveropt/Untitled%2019.png)

2000년대 초반에서 18년도까지 

예전에서 윈도우 프로그래밍을 한다고하면 iocp를 썼냐 안썼냐가 화두였음

iocp로짜려면 책을 봄

책을보는데 심플한 예제가있는데 예제가 나쁜건아닌데 

첫번째로 이해를 위해 쉽게 짜여져있고 

두번째로 iocp의 성능을 최대한 뽑아내는 식으로 짜여져있음 

그거의 문제는 그거를 실제 게임서버에서 쓰려고하면 난리가 남 

iocp스레드가 패킷을 받자마자 게임 메인프로시저로 들어감 게임 패킷 처리를 해버림 

그렇게하면 문제가 iocp 스레드 모두가 같은 코드로직을 들어갈 수 있음 

게임 메인 프로시저는 오만가지 기능이 들어감 다수의 스레드가 이게임의 모든것을 할 수 있는 함수를 동시 진입을 하게되면 게임의 모든 리소스에 락을 다걸어야함

그냥 만들면 모든 리소스에 락을 다 걸어야하고 성능측정하면 싱글스레드보다 느려지고 느려지기만하면 다행이고 락을걸다 실수를 하면 버그가 생기는데 버그를 찾을 수가 없음 못잡고 망하는 케이스를 워낙 많이 봄 

이 방식은 깔끔하게 버림

iocp를 쓰되 iocp에 pending된 스레드들은 철저히 io 작업만 함 

게임서버의 로직을 관장하는 패킷 핸들러쪽으로 들어가질 않음 

패킷 보냄, 남은거 다시 보냄, 수신, 사이즈 제대로 왔는지 완성 체크 , 완성되면 메세지 단위로 핸들러를 호출할수 있는 애한테 보내는 역할만함 

1개의 메인 스레드는 실제 게임 로직을 처리함. 컨트롤 스레드라 봐도되고 패킷 핸들러 스레드라고 불러도 되는데 메인스레드라 부름 

n개의 보조 스레드 풀은 패킷관리,iocp에 등록되어있는 io 스레드들도 하나의 보조 스레드 풀이고 충돌처리, db랑 통신하는 스레드 여러개의 스레드들이나 단일 스레드 나 혹은 복수의 스레드로 구성되어 있는 스레드 풀을 가지고 있음 

클라이언트가 하는걸 서버가 그대로 하려면 여러가지 문제점이 있지만 똑같은 결과를 내려면 코드가 다르면 안됨. 최대한 같은 코드를 사용하는데 dll레벨에서 대부분의 바이너리를 같은걸 씀 

dll을 로드하는 exe만 다르지 dll은 그대로 공유함

네트워크, 그래픽스도 동일 opengl directX 쓰는 레이어만 빼고 로드하는 식

코드를 비슷하게 짜는건 처음부터 불가능

같은거 공유하거나 다르게 짜거나 

A를 개조해서 A’를 만들어서 둘의 결과를 같게 만드는건 말이 안됨

그럼에도 스레드는 남용하지말자

simd를 사용하는것도 극단적으로 성능이 30나오던게 60나오지는 않음 근데 40나오던게 45나오는 정도 그런데 서버에서는 여러명의 유저가 붙기때문에 1클럭이라도 빨라야 함

![Untitled](/images/gameserveropt/Untitled%2020.png)

![Untitled](/images/gameserveropt/Untitled%2021.png)

엄청 dll이 많음

수학, 자료구조, 파일시스템 (개별 파일 오픈, 패키징되어있으면 데이터 긁어오고 쓸수있는 체계)

Renderer(DirectX 9/11/12 : 똑같은 인터페이스 노출, openGL)/ Geomaetry, Collision 을 합쳐서 그래픽스 엔진이라 부름 

서버에선 렌더링안하니 렌더러 dll 로딩안함

Geomertry.dll 별도의 공간분할, 충돌, 피킹 처리 안함 

Collision : 별도의 충돌처리 얘도 자체적인 스레드 풀로 동작

옛날 윈도우즈 95,98 쓰던 시기는 서버는 NT사용, 클라이언트는 윈도우 95 98을 사용해서  그 당시엔 iocp 지원이 안됐음 커널이 달라서 

그래서 서버는 iocp를 만들고, 클라이언트 용 네트워크 라이브러리는 이벤트 셀렉트 방식 같은 여러가지 다른 방식의 api를 사용해서 만들어 썼는데 xp이후로는 그럴필요가 없음 

그래서 옛날에는 클라이언트/서버 네트워크 라이브러리를 분리해서 만들어 썼음 

지금은 클라/서버 무조건 iocp로 만든 단 하나의 네트워크 라이브러리를 사용하고 있음 

얘는 서버 클라 겸용

다수의 네트워크 io 처리 가능, 유지관리 가능 

서버는 db억세스를 위해 db미들웨어 비슷한거 만들어서 붙여놓음  

다수의 스레드로 처리하고 비동기적으로 처리가능 

지형 데이터, 캐릭터 데이터 공유 

![Untitled](/images/gameserveropt/Untitled%2022.png)

동기화로 생기는 버그를 가능한 피해가는 것 

락걸면서 떨어지는 성능 방지

![Untitled](/images/gameserveropt/Untitled%2023.png)

Game loop

가운데에 메인스레드 흐름이 있음 

컨트롤 스레드가 매 프레임 마다 게임 루프를 돌면서 진행을 하고있음 

모든 플레이어 들의 상태를 알고 있음

그래서 매 프레임마다 플레이어들의 상태를 변화시킴

다수의 플레이어들을 빠르게 처리해야해서 비동기식 스레드 풀로 돌아감 

복셀데이터는 따로 압축해서 보내서 처리 안그러면 네트워크가 마비됨

On Event

항상 시행하지 않지만 비정기적으로 어떤 이벤트가 발생해서 처리하는 애들 

패킷 핸들러

- 로그인 요청
- 아이템 요청

패킷이 쌓여있는지 안쌓여있는지 핸들러는 모름 

번개표시는 다른 스레드들로부터 통지를 받아서 움직임

복셀 처리는 

자동복구되는 지형의 경우 복구에도 상당한 비용이 들어가서 메인스레드에서 하지않음 

데이터의 복사본을 가지고 다른 스레드가 복구를 완료한 다음에 메인스레드가 가져가는 식 

![Untitled](/images/gameserveropt/Untitled%2024.png)

게임특성과 상관없이 소켓 연결하는 서버는 다 같이 적용될 수 있음 

4byte 헤더가 있고 뒤에 바디가 따라옴

복잡한경우는 바디안에서 처리 

하위의 io스레드의 경우 패킷 도착시 4byte가 먼저 모였는지를 봄 

4byte가 모이면 총 길이가 몇바이트인지 판단 

버퍼안에 그만큼 쌓여있는지 보고 길이만큼 초과하면 바디부분을 짤라서 메인스레드가 지정한 버퍼쪽에 넣어줌 

더블버퍼링 사용 중 

![Untitled](/images/gameserveropt/Untitled%2025.png)

스레드 네개는 iocp에 물려있는 io스레드들 

CConnection은 class 이름 : 각 유저들

그림은 지금 12명 붙어있었다고 볼수있음

각 커넥션은 리시브 버퍼 센드 버퍼 하나씩 들고있음 

이벤트가 발생한 족족 스레드 4개중 하나에 시그널이 되서 해당 커넥팅에 대해서 처리를 함 

버퍼가 두개인데

io스레드 들은  write only buffer만 접근 가능 

패킷 수집시 쭉 쓰고 통지를함 (메인스레드가 즉각 반응하는건아님, 통지)

메인스레드는 자기가 접근할 수 있는 read only buffer가 존재 해당 버퍼의 패킷을 쭉 처리함 

다 처리하면 버퍼를 스위칭함 

전제는 front buffer(read only)를 처리하는 동안 io스레드로부터 back 버퍼(write only)에 패킷이 도착했다는 통지를 한번이상 받아야하는 전제가 필요

장점 : 락 비용이 거의 없음 

io스레드들이 떼거지로 패킷 핸들로 들어가게 되면 동기화가 매우까다로워짐

이런 게임도 있지만 유지보수하기 굉장히 힘들다고 생각함 

io 스레드들이 거의 독립적으로 제 성능 발휘 가능 

# 충돌처리

![Untitled](/images/gameserveropt/Untitled%2026.png)

충돌처리는 스킵..

![Untitled](/images/gameserveropt/Untitled%2027.png)

처리를 쪼개서 함

![Untitled](/images/gameserveropt/Untitled%2028.png)

![Untitled](/images/gameserveropt/Untitled%2029.png)

![Untitled](/images/gameserveropt/Untitled%2030.png)

![Untitled](/images/gameserveropt/Untitled%2031.png)

![Untitled](/images/gameserveropt/Untitled%2032.png)

![Untitled](/images/gameserveropt/Untitled%2033.png)

![Untitled](/images/gameserveropt/Untitled%2034.png)

요청 → 서버에서 시뮬 → 결과 통지 

![Untitled](/images/gameserveropt/Untitled%2035.png)

![Untitled](/images/gameserveropt/Untitled%2036.png)

![Untitled](/images/gameserveropt/Untitled%2037.png)

![Untitled](/images/gameserveropt/Untitled%2038.png)

![Untitled](/images/gameserveropt/Untitled%2039.png)

![Untitled](/images/gameserveropt/Untitled%2040.png)

![Untitled](/images/gameserveropt/Untitled%2041.png)

![Untitled](/images/gameserveropt/Untitled%2042.png)

![Untitled](/images/gameserveropt/Untitled%2043.png)

![Untitled](/images/gameserveropt/Untitled%2044.png)

![Untitled](/images/gameserveropt/Untitled%2045.png)

![Untitled](/images/gameserveropt/Untitled%2046.png)

async io

![Untitled](/images/gameserveropt/Untitled%2047.png)

![Untitled](/images/gameserveropt/Untitled%2048.png)

![Untitled](/images/gameserveropt/Untitled%2049.png)

![Untitled](/images/gameserveropt/Untitled%2050.png)

![Untitled](/images/gameserveropt/Untitled%2051.png)

![Untitled](/images/gameserveropt/Untitled%2052.png)

![Untitled](/images/gameserveropt/Untitled%2053.png)

![Untitled](/images/gameserveropt/Untitled%2054.png)

![Untitled](/images/gameserveropt/Untitled%2055.png)

오브젝트 단위로 ref count로 참조

![Untitled](/images/gameserveropt/Untitled%2056.png)

![Untitled](/images/gameserveropt/Untitled%2057.png)

![Untitled](/images/gameserveropt/Untitled%2058.png)

![Untitled](/images/gameserveropt/Untitled%2059.png)

![Untitled](/images/gameserveropt/Untitled%2060.png)