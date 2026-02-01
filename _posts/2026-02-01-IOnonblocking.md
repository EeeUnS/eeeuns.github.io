# IO (non)blocking / (a)sync 오개념 잡기

unix쪽 전용 api를 제외하고는 모두 windows 기준으로 내용이 진행됩니다.

# IO blocking vs non-blocking

**blcoking이란 스레드가 대기모드로 들어갈 수 있는가에 대한 여부이다.**

**io에 대한 설정**

- 블로킹 mode : `read()/recv()` 같은 io 호출이 성공할때까지 blocking이 걸린다. 로직이 단순
- 논블로킹 mode : io recv send가 성공하든 안하든 호출이 즉시 반환, 실패시 `EAGAIN/EWOULDBLOCK` 에러 발생. 실패시에 루프가 비효율적일 수 있어 보통 I/O 멀티플렉싱과 함께 사용.

어쨌든 무조건 보내기 위해 시도한다고하면 아래와 같은 상황이 발생할 수 있다.

![img (1).png](/images/iononblocking/img_(1).png)

# Socket에서의 Blocking vs NonBlocking

![img.png](/images/iononblocking/img.png)
![Untitled](/images/iononblocking/Untitled.png)

1. **일반적으로 Send, Recv시에  blocking이 걸리지 않는다.**
2. 커널 내부의 특정 상황에 따라서 blocking이 걸릴 수 있다.
3. non blocking mode는 이때에 blocking에 걸리는것이아닌 함수로 에러를 반환한다. `EAGAIN/EWOULDBLOCK`

## send

- Blocking 상황 : 송신버퍼가 다 찼을때
- 연결된 소켓이 나가는 데이터를 쓰는 함수.
- 함수가 성공적으로 실행됐다고해서 상대방이 성공적으로 전달되어 수신을 보장하지않음

## recv

- blocking 상황 :  송신버퍼에 수신할 데이터가 없는경우

# blocking 발생하는 경우/특이케이스에 대한 작동방식

- Send TCP기준 : 송신 버퍼가 가득 찰시 send()에는 **블로킹이 발생**
- Recv TCP기준 : 1byte라도 있을시 recv 즉시 반환/or **blocking**
- Send UDP기준 : 수신(receive)버퍼가 가득 찰시 send()한거는 (항상)즉시반환, 하지만 송신측에서 버림
- Recv UDP기준 : 데이터그램이 최소 한개 있을시 즉시 리턴/or **bloking**

# Asynchronous  IO vs Synchronous IO

![Untitled](/images/iononblocking/Untitled%201.png)

- Synchronous IO : IO(send, recv) 처리 요청을 하고 호출 스레드가 결과를 직접 받아 처리. 제어 흐름이 직관적이나 대기 구간이 길면 비효율.
- Asynchronous  IO :  IO(send, recv) 처리 하는데 이에 대한 실제 처리 제어를 커널로 넘기고 이에 대한 이후 결과를 큐/콜백으로 통지하여 다른 스레드에서 받아서 처리 하도록 처리한 구조 : windows에서는 overlapped io라고도 한다.

가장 간단하게 구분 하는 방법은 해당 io 함수의 이후 처리를 해당 호출을 한 스레드에서 처리하는가?로 보면 될것같다.

# 오개념 잡기

Async? Sync? NonBlocking? Blocking?

![Untitled](/images/iononblocking/Untitled%202.png)

# 왜 이런 재앙이 시작되었는가?

모든 재앙이 시작된 글

[https://developer.ibm.com/articles/l-async/](https://developer.ibm.com/articles/l-async/)

1. select, poll은 일단 asynchronous이 아니다.
    1. timeout 세팅에따라 non-blocking도 가능하다;;
    2. [https://man7.org/linux/man-pages/man2/select.2.html](https://man7.org/linux/man-pages/man2/select.2.html)
    3. [https://man7.org/linux/man-pages/man2/select.2.html](https://man7.org/linux/man-pages/man2/select.2.html)
2. aio 는 재밋게도 사장된 기술이다 ;;
    1. epoll이 더 좋다고.. 
    2. 실제로 대부분 현존하는 리눅스 서버 코어는 epoll로 되어있다
    3. [https://jacking75.github.io/network_poxis_aio/](https://jacking75.github.io/network_poxis_aio/)
    4. io_uring은 또 심각한 문제점이 있다고;;;;
        1. [https://news.ycombinator.com/item?id=39416981](https://news.ycombinator.com/item?id=39416981)

**애초에 저런 4분면을 나눈것부터가 모든 문제의 시작이라고 봐야한다.**

io muiltiplexing과 read write는 애초에 같은 계층으로서 볼 수 없다

초기에 막 찾아보면서 혼란이 굉장히 많았는데 어느정도 정리가 되었다.

그냥 저 사진을 해석하려고 시작하는 그 시작점부터 잘못됐으니 혼란이 있을 수 밖에.

- **blocking은 함수 호출시에 thread가 blocking모드로 가는지로 따져서 구분해야한다.**
- Asynchronous  IO : io multiplexing에 따른 전체 프로세스 구조에 따라서 구분해야한다.

- blocking 여부는 코드를 짬에 있어 따질 필요가있는 필요한 부분이다.
    - 명확한 정답이 존재하며 해당 함수, 해당 소켓이 blocking이 걸리는지 여부는 따질 필요가있는 중요하다.
- Asynchronous  IO는 추상화된 개념에 가까워서 그렇게 중요하지않다.
    - multiplexing 함수 하나를 놓고 얘가 Asynchronous  IO냐 아니냐를 따지는게 크게 의미있지않음
    - 그것보단 전체 스레드 구조가 어떻게 이루어져있고 io, 패킷 흐름이 어떻게 되는지, 이 전체 뷰를 파악하는것이 더 중요하다고 판단한다.,

# 결론 : 솔직히 말해서 이런거 구분하는 의미가 없다고 생각함.

---

- [https://velog.io/@jyongk/TCP의-데이터-보장성-대해-파헤쳐보자-m0k4vchxup](https://velog.io/@jyongk/TCP%EC%9D%98-%EB%8D%B0%EC%9D%B4%ED%84%B0-%EB%B3%B4%EC%9E%A5%EC%84%B1-%EB%8C%80%ED%95%B4-%ED%8C%8C%ED%97%A4%EC%B3%90%EB%B3%B4%EC%9E%90-m0k4vchxup)
- [https://learn.microsoft.com/en-us/windows/win32/fileio/synchronous-and-asynchronous-i-o](https://learn.microsoft.com/en-us/windows/win32/fileio/synchronous-and-asynchronous-i-o)
- [https://man7.org/linux/man-pages/man2/select.2.html](https://man7.org/linux/man-pages/man2/select.2.html)