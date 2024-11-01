---
layout: post
title:  "폭증하는 카카오톡 트래픽에 대처하는 방법"
date:   2024-09-18 19:26:01 +0900
tag: notes
---

<iframe width="560" height="315" src="https://www.youtube.com/embed/U905BeDQ_BA?si=7q41Bl3I4pmG309J" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

![image.png](/images/kakaotalkdisaster/image.png)

메시징 서버 종류

- 메시지 전송
- 접속하기위한 로그인 요청
- 채팅방 진입
- 메시지 동기화 등

![image.png](/images/kakaotalkdisaster/image%201.png)

낮 최대 62만 건

![image.png](/images/kakaotalkdisaster/image%202.png)

신년 인사 등 트래픽이 폭증하는 케이스가 간혹 존재

![image.png](/images/kakaotalkdisaster/image%203.png)

이를 대비하기 위한 두가지 시스템 구축

# 카톡의 백그라운드 로그인 방식

![image.png](/images/kakaotalkdisaster/image%204.png)

채팅방에 있는 다른 사용자에게 메시지를 보내면

서버가 메세지를 다른 사용자에게 전달

근데 사용자가 서버에 접속 중이지 않을경우

안드로이드 fcm을 통해 푸시를 전송하게됨

메세지가 오면 클라단에선 서버에 로그인 요청을 먼저하면서 TCP연결을 함

이경우 사용자가 바로 사용할때 메세지를 바로 확인하고 전송이 가능한 상태가 됨 

# 대표적인 장애 사례 : 2016년 경주 지진

지진으로 인해 카톡이 2시간 정도 장애 발생

![image.png](/images/kakaotalkdisaster/image%205.png)

전국민에게 알림문자가 발송되면서 휴대폰이 깨어나게됨 → 카톡 클라가 백그라운드 로그인을 수행하게됨

→ 전국민이 카톡 서버에 접속 시도 

평상시케이스

- 수동으로 카톡 켤때
- 메세지 받거나
- 문자 받거나

근데 이례적으로 로그인 요청이 많아지는 상황이 발생

![image.png](/images/kakaotalkdisaster/image%206.png)

이로 인해서 서버가 로그인 요청만을 처리하게되고 연결을 맺고 사용하던 케이스에서도 메세지를 보낼때 실패처리가 되면서 연결이 끊어짐 → 다시 로그인 시도 

모든 요청이 로그인이 됨

![image.png](/images/kakaotalkdisaster/image%207.png)

19시 44분 지진 발생 

놀란 사용자들이 메세지를 보냄 → 메세지 TPS가 급격하게 올라감 + 백그라운드 로그인 요청도 엄청 들어옴 → 처리시간이 급격하게 높아지고 이후 TPS가 급격하게 낮아진걸로 봤을때, 서버가 로그인 처리만하느라 메시지 전송 처리를 못하게됨.

![image.png](/images/kakaotalkdisaster/image%208.png)

자연재해는 예측하는것이 불가능하기에, 자체적으로 트래픽 폭증을 인지하고 대응하는것이 필요.

이 중 아이디어로 활성화 된 스레드 수를 이용하여 부하레벨을 정함

![image.png](/images/kakaotalkdisaster/image%209.png)

이후의 변화

![image.png](/images/kakaotalkdisaster/image%2010.png)

![image.png](/images/kakaotalkdisaster/image%2011.png)

진짜 필요한 로그인만 처리하고 붙어있는 세션의 메시지 전송을 처리

# 17년 11월 15일 포항 지진

![image.png](/images/kakaotalkdisaster/image%2012.png)

![image.png](/images/kakaotalkdisaster/image%2013.png)

위의 초록색이 로그인 지표, 빨간 색은 백그라운드 로그인 지표 

메세지가 오면서 백그라운드 로그인 지표가 급격하게 올라감 그러나 백그라운드 차단으로 인해 급격하게 내려감

그러고 이후로 메세지 TPS가 뛰었지만 문제없이 처리한것을 확인

![image.png](/images/kakaotalkdisaster/image%2014.png)

피크시에 백그라운드 로그인 비율이 96프로까지 올라감 (서버가 로그인 처리만 하고있는 상황)

# 20년 1월 1일 신년 장애

![image.png](/images/kakaotalkdisaster/image%2015.png)

새로 구축한 시스템에서 10ms걸리던게 자정되면서 600ms까지 걸려 장애발생 → 전체 서버 장애 발생 → 클라이언트 요청에 타임아웃 발생 → 클라연결을 끊음 (기존 붙어있던 세션이 50%가 날아감)

새해 인사로 인한 카카오톡 메세지가 몰리는데 이는 백그라운드 로그인이 아니기에 막을 방법이 없어 이전 대응 구축이 효과가 없었음

![image.png](/images/kakaotalkdisaster/image%2016.png)

추가로 50%의 떨어져나간 사용자들도 다시 로그인 요청을 하면서 트래픽이 폭증하게 되고, 일부 서버는 메세지 전송이 불가해짐

그러면서 약 두시간 동안 장애 발생

로그인 트래픽의 무거운 요청이 그대로 유입되었기 때문 

# 교통관리 시스템 구축

스레드를 요청 별로 할당해서 아무리 많이 들어와도 한계가 있도록 너무 많은 요청이 와도 다른 처리는 할 수 있도록  

![image.png](/images/kakaotalkdisaster/image%2017.png)

매니저라는 객체가 존재

이 객체가 각 요청에 대해서 할당될 스레드 수를 정함 

스레드가 특정 요청을 처리하고 있을때 각각의 스레드 count가 1씩 상승 이 스레드 카운트가 한계점에 도달시 오버플로우 발생으로 실패 처리

# 트래픽 폭증 사례 : 카타르 월드컵

가나 전때 조규성이 동점골을 넣었을때 트래픽이 가장 심했음

![image.png](/images/kakaotalkdisaster/image%2018.png)

메세지 전송이 급증하였는데 위의 트래픽에 제한이 있었기에 다른 요청도 골고루 처리하여 문제가 없었음  

![image.png](/images/kakaotalkdisaster/image%2019.png)

![image.png](/images/kakaotalkdisaster/image%2020.png)

일부 메세지 전송이 실패했으나 전체 서비스 장애로 이루어지지는 않게 방지

![image.png](/images/kakaotalkdisaster/image%2021.png)