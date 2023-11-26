---
layout: post
title:  "Tomcat Takeover Blue Team Challenge WriteUp"
date:   2023-11-26 19:26:01 +0900
tag: ctf
---



이러저러한 이유로 WireShark를 공부해서 이제 실습느낌으로 추천받아서 풀었는데 

이런 문제는 처음 푸는거라 처음엔 도움도 꽤나 받았고.. 때려맞춘것도 많이 크고 -_-;

[https://cyberdefenders.org/blueteam-ctf-challenges/135#nav-questions](https://cyberdefenders.org/blueteam-ctf-challenges/135#nav-questions)

1. Given the suspicious activity detected on the web server, the pcap analysis shows a series of requests across various ports, suggesting a potential scanning behavior. Can you identify the source IP address responsible for initiating these requests on our server?

웹서버에 이상한 활동이 감지되었을때 pcap 분석은 여러 포드들에 요청과정과, 조사하는 행동을 제공하는 것을 보여준다. 우리 서버에 이러한 요청을 시작하는 sourece IP를 알 수 있니?  

간단하게는 우리서버에 개짓을한 ip를 찾아라다.

![Untitled](/images/tomcatctf/Untitled.png)

시작화면  

어떻게 해야할질 모르겟는데

일단 웹서버라는것에 착안을 해 HTTP로 filter를 걸어봤다

![Untitled](/images/tomcatctf/Untitled%201.png)

그러고 내리다보면이제 404 not found가 엄청 나오는데

![Untitled](/images/tomcatctf/Untitled%202.png)

일단 알려준 사람이 전체흐름을 볼때는 404나 이런 문제가 있어보이는 부분을 좀 유의해서 보면 도움된다고 알려주더라 

이 404를 받는놈이 나쁜짓을 하는 놈이지않을까?

14.0.0.120

1. Based on the identified IP address associated with the attacker, can you ascertain the city from which the attacker's activities originated?

얘 도시알아내는건데 실제 덤프와는 상관이없이 ip로 검색 때려맞추면 나온다.

Guangzhou

1. From the pcap analysis, multiple open ports were detected as a result of the attacker's activitie scan. Which of these ports provides access to the web server admin panel?

admin panel로 접근하는 포트가 뭐냐

pcap을 보면 Web Server는 언제나 8080으로만 통신을 한다.

8080

1. Following the discovery of open ports on our server, it appears that the attacker attempted to enumerate and uncover directories and files on our web server. Which tools can you identify from the analysis that assisted the attacker in this enumeration process?

계속 질문에서 보면 상황설명을 해주기에 패킷을 직접 보지않아도 어느정도 상황 이해가 갈 수 있는 상황이긴하다.

이제 웹서버에있는 디렉토리를 계속 접근하려고 요청을 이래저래 날린다고 함.

이중에 얘가 쓰는 프로세스가 뭔지?

이놈과의 연결을 follow → http stream 으로 한번 보자.

![Untitled](/images/tomcatctf/Untitled%203.png)

![Untitled](/images/tomcatctf/Untitled%204.png)

![Untitled](/images/tomcatctf/Untitled%205.png)

404가 나오기전의 패킷을 보면

계속 뭔가 url 접근을 시도하는데 없다고 하고있는거다

그래서 대충보니 딕셔너리 어택을 하고있는것을 알 수 있었다.

여기서 질문의 프로세스는 이를 시도하는 User-Agent의 프로세스를 물은것이다.

가장 일반적인걸로는 이제 웹브라우저가 가장 대표적이고 아니면 패킷만 날릴 수 있는 프로세스라면 다 가능한데 이 프로세스 이름을 물은것이다.

나는 진짜 이짓을 하는 프로세스가 api등으로 프로세스명을 받아와서 그대로 패킷에 내용을 담아 보내는 그런 과정이 있는 줄 알았다.

뭐 아무튼..위에 사진을 보면 알 수 있지만 gobuster 이다. (뭔지몰라서 찾아보니 칼리리눅스의 툴이고 실제로 딕셔너리 어택을 하는 툴이더라)

그리고 아래부분에서 보면 요청에서 401로 바뀐다.

![Untitled](/images/tomcatctf/Untitled%206.png)

1. Subsequent to their efforts to enumerate directories on our web server, the attacker made numerous requests trying to identify administrative interfaces. Which specific directory associated with the admin panel was the attacker able to uncover?

아 위에 적혀있네 /manager

1. Upon accessing the admin panel, the attacker made attempts to brute-force the login credentials. From the data, can you identify the correct username and password combination that the attacker successfully used for authorization?

brute force attack해서 admin로그인을 했는데 올바른 username 패스워드가 뭐냐?

위에서보면 404 → 401 Unauthorized가 나왔다 그럼 이 401이 안나온 순간을 보면되겠네?

그리고 filter를 추가로 걸었따

http && ((ip.src == 14.0.0.120) || (ip.dst == 14.0.0.120))

보면 어느순간 401이 여러번뜨는 케이스가있는데

![Untitled](/images/tomcatctf/Untitled%207.png)

이렇게 Authorization에보면 Credentials에 값이 나온다.

![Untitled](/images/tomcatctf/Untitled%208.png)

OK를 받는순간의 패킷은 이것.

1. Once inside the admin panel, the attacker attempted to upload a file with the intent of establishing a reverse shell. Can you identify the name of this malicious file from the captured data?

리버스쉘을 심는데 파일 이름이 뭘까?

![Untitled](/images/tomcatctf/Untitled%209.png)

그 바로 밑의 패킷 중에 POST 요청이있는데 여길뒤지면 이걸 발견할 수 있다.

이렇게 찾을 수도있고

![Untitled](/images/tomcatctf/Untitled%2010.png)

file → export → http로 파일들 전체 list를 보고 찾아볼 수도있다.

참고로 파일은 \n 6개로 이루어져있다.

1. Upon successfully establishing a reverse shell on our server, the attacker aimed to ensure persistence on the compromised machine. From the analysis, can you determine the specific command they are scheduled to run to maintain their presence?

스케줄 명령어가 뭐냐?

재밌게도 http로는 더 알 수 있는게 없었는데

file 로 찾거나 tcp로 뒤지다보니 찾을 수 있었다. 

![Untitled](/images/tomcatctf/Untitled%2011.png)

이 문제까지왔을땐 패킷 캡처 끝자락이라 별로 전체 눌러보는것도 별 문제는 없었다.