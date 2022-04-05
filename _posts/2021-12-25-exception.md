---
layout: post
title:  "크리스마스날쓰는 예외처리를 어떻게 할 것이냐"
date:   2021-12-25 19:26:01 +0900
tag: etc
---

코드를 깔끔하게 짜는것과 예외처리에 대해서 최근 고민이 좀 많았다. 최근에 두가지 상황을 겪었다.

1. 최근에 코드를 짜는 과제들은 완벽한 프로그램을 원했었다. 이에 따라서 나도 happy path에서는 일어나지않는 예외들에대해서 어떻게 처리해야 깔끔한지 고민을 많이하게 됐다. 예를들어 io에서 error가 날때, 메모리 할당에 실패할때, 외부라이브러리 initialize 실패에 따른 null 반환, 이런것들도 과연 하나하나 에러처리를 해야하는가?란 의문을 많이 가졌다.
2. 웹개발을 하는데 나는 중간에 투입된 상황에서 짜여진 코드를 보는데 코드 전체가 함수 전체에 try를 걸어놓고 catch로 logging을 하는식으로 코드가 짜여져있더라. 이게 맞....나? 싶으면서도 이부분에서 catch에서 return 처리가 제대로 되어있지않아서 coding standard에서도 계속 에러가 났다. 그리고 코드를 짤때도 이부분을 exception 처리를 해야하는건 눈에 보이는데 어떻게 처리하고 return을 어떻게 해야할지 잘모르겠더라.


이 부분에 대해서 정리가 잘되지않아서 고민이 많았었는데, 어느정도 정리된것을 정리하고, 그외에 도움이 될까싶어서 pope kim의 exception 처리 영상을 보고 정리한것을 포스팅한다.

재미있는 글 : 
[asking_for_permission_instead_of_forgiveness_when_working_with_files](https://docs.quantifiedcode.com/python-anti-patterns/readability/asking_for_permission_instead_of_forgiveness_when_working_with_files.html)


# 일단 눈에 보이는 예외에 대해서는 if로 처리한다.
- ex : file로부터 입력을 받는데 당초에 file이 없는경우.

물론 그럼에도 exception이 필요한 경우는 있다.
- ex : file로부터 입력을 받는데 file이 존재했다. 그러나 그후에 파일이 사라졌다. 이는 exception 처리로 해결한다.

결론은 둘다쓰자.

- exception은 일단 exception의 의미를 생각해서라도 기본적으로 일어나서는 안되는 행동이라고 정해야한다. 
- happy path중에 exception은 일어나선 안된다. 
- 성능적으로도 exception은 인터럽트 들어오고 하드웨어와 os단에서 처리하는 부분이 존재해 기본적으로 일반적인 루틴보다 느리다.

# 인자값 null 처리

이는 코딩 스탠다드로 해결. 내가 현재 따르는 코딩스탠다드이다.

- [popeKim c++ coding standard](https://docs.popekim.com/ko/coding-standards/cpp)
- [관련 유튜브 플레이 리스트](https://www.youtube.com/playlist?list=PLW_uvsSPlijsp_NgPpVzb37u97pzPEola)


null이 들어오는것에 대해서는 다음과 같이 정리된다.

- 내가 짜는 함수에서 인자로 null은 들어오지않는다. 이렇게하면 함수내부에서 null처리를 안해도 된다.
- null이 들어오는 경우 매개변수명에 OrNull을 붙인다. 그럼 여기에 따른 적절한 처리를 한다.


# 그래서 exception 어떻게 할까?


<iframe width="846" height="480" src="https://www.youtube.com/embed/g7dzMgrWFic?list=PLW_uvsSPlijvMY-6Y-0I-bi4tlUFKEuFK" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

<iframe width="846" height="480" src="https://www.youtube.com/embed/YGOE5CEkX0o?list=PLW_uvsSPlijvMY-6Y-0I-bi4tlUFKEuFK" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

참고로 영상에 나온 블로그 글은 이것같다.
- [joel spolsky](https://www.joelonsoftware.com/2003/10/13/13/)


1. exception 처리는 기본적으로 힘들다.
   - exception같은거는 함수 signature만보곤 알 수가 없어서 문제가 생긴다(자바는 예외)
   - 내가 exception을 많이 쓰는 결과 회사에서 진짜 제대로 exception handling하는걸 못봤다. 제대로 할래도 놓치는게 많다.
2. exception이 나는것에 대해 생각하면서 코드짜는것은 중요한것같다
3. exception이 나는 경우는 다음이 있다.
   - 외부 dependency
     - DB사용  
     - 파일 사용 
     - 다른 웹호출 
     - 외부 라이브러리
   - 아니면 정말 코드에서 이상한 일을 할때
4. exception 처리는 여러방법이있다
   - exception을 당연히 예측을 하고 문제없이 처리해서 프로그램이 계속 돌게 해야하는 경우
   - logging을 한경우에 뻗게해야하는 경우도 있다
5. 중요한것은 exception을 어떻게 처리하는가가 중요한것이아닌 현재 프로그램의 상태가 고장나지않는것이 가장 중요하다.
6. exception은 안 쓸 수 있으면 쓰지마라

`옛날 C스타일의 함수를 보면 exception없이 error 코드 반환하고 마지막 error 가 뭐였는지 보는 방식이 좋다고 생각하는데 function signature를 보는것만으로도 error 코드가 오는걸 안다. 그 error 코드를 가지고 어떻게 처리해야될지 생각을 할 수 있다. 따라서 함수를 까보지않아도 무엇을 대비해야하는지 알 수 있게된다. 여기서 아이디어를 얻은것이 C#에서 예를들어 int.Parse가 있고 int.TryParse가 있는데 얘는 Parse의 경우 exception을 던지고 TryParse는 리턴이 Parse의 성공여부가 boolean으로 되어있고 인자가 outParmeter로 되어있다. (이 부분은 pope kim coding standard에서 out 인자명에 out-을 붙이는 식으로 해결한다) error 코드 비슷한 방향인데 딱 감이온다. 이 방향이 바람직한것같다.`

나는 여기서 착안해 미리미리 검사가가능한(if로 처리가능한) 에러 요소에 대해서는 error가 날수 있는 것들을 try 함수로 wrapping하는걸 선택했다.  


그리고 따로 대비가 안되는 실제로 try catch문으로서 해결해야하는 exception에대해서 이렇게 처리하기로 결정했다.

`거기서 내가 만든 서비스 안에서 익셉션이 나는경우 exception은 exception 마스킹을 하라고한다. 익셉션이 난건 이미 그자체가 큰 에러다. 모든게 깨져야한다. 더이상 정상적인 동작이 수행 불가능하다내 서비스 안에서 try catch를 하고 익셉션이 발생했다는 정보하나만 반환을 하게한다. 그리고 호출쪽에서는 그냥 뻗게하라고 하고 디버깅은 해야하니 익셉션 내부에서 난 경우 익셉션을 로깅은 해야한다. 요즘 각광받는 익셉션 처리방식인것같다.`

try는 딱 에러가 날수있는 부분만 감싸는걸로 처리. catch문에서는 일단 logging만으로 처리하고 에러반환 후 이를 라우터단으로 올려서 에러로 반환을 할 수 있도록 처리.

