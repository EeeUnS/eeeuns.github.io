---
layout: post
title:  "github blog와 latex"
date:   2020-12-11 02:26:01 +0900
tag: paper
---

[2020 한국텍학회 제13차 정기총회 및 학술대회](http://www.ktug.org/xe/index.php?mid=KTUG_open_board&document_srl=239863)
발표자료를 posting 한것.

1. latex 수식을 포스팅하기 위한 여러가지 방안의 장단점을 따지고
2. 여러가지 방안의 수식을 쓰는 법에 대해서 정리하고 텍수식 적용 플러그인인 katex와 mathjax 비교
3. 대안으로서 github blog의 jekyll과 mathjax을 제안 

[해당 발표 repo](https://github.com/EeeUnS/github-blog-and-latex)

끝의 mathjax 적용은 최신에 맞춰서 ver 3.0으로 업데이트 한것을 기술


#  수식을 위한 BLOG

## blog platform

1. Naver blog
2. Tistory
3. Blogger

- 접근성 용이  
- 편의성 제공 <-> 제한된 기능

## 직접 만들기

1. 직접 세팅
2. 높은 자유도
3. 서버 유지 비용
4. 도메인 비용
5. 파일로서 직접 평생 소유
6. 어렵다

# 수식 쓰기

### Naver Blog에서 수식쓰기
에디터 자체에서 수식기능을 지원

![image](/assets/img/githubblog/naver1.png)

- Tex을 몰라도 사용가능
- 텍 문법을 어느정도 그대로 사용가능
- 네이버 카페에 글 공유가능
- 수식하나하나 칠때마다 한글과 비슷하게 수식기능을 따로따로 적용해야함(수식을 많이 쓸경우 불편)


![image](/assets/img/githubblog/naver2.png)


### Image

1. 수식만 이미지로 변환후 첨부 (많은 블로그 수식지원이 작동하는 방식)
2. 진짜 pdf를 그대로 이미지로 변환후 첨부하기


![image](/assets/img/githubblog/image1.png)

하지만 pdf 나온 파일은 컴퓨터로 보기엔 많은 공백이 생겨서 가독성에 문제가 많음.


![image](/assets/img/githubblog/image2.png)

- 여기에 대한 해결책으로 ebook 용으로 만든 템플릿을 조금 수정한 [텍 파일 template](https://github.com/EeeUnS/github-blog-and-latex/blob/master/ebook.tex) 제안.
- [해당 파일 원본](https://www.latextemplates.com/template/ebook)
- 사진 크기를 조정하는등 추가적인 수정은 필요
- 패키지 충돌 등 사용못하는 기능이 몇 있음


### Tex plugin
- 해결법 수식 -> tex로 자동 변환해주는 js 라이브러리 사용
- 수식 문법을 쓰면 자동으로 변환해서 수식으로 보여줌
- 실제 tex사용 환경과 문법이 미묘하게 다름

tistory, blogger와 직접 서비스 하는 사이트/블로그같은경우는 tex plugin을 사용가능. 현재 많이 쓰이는 tex plugin으로는 크게 두가지가있음.

- [mathjax](http://docs.mathjax.org/en/latest/input/tex/macros/)
- [katex](https://katex.org/docs/supported.html)

#### Mathjax vs Katex

|기능 | 압도적으로 많음 | 상대적으로 적음|
|:---:|:---:| :---:|
| 랜더링 시간 | 김([비교](https://www.intmath.com/cg5/katex-mathjax-comparison.php?processor=MathJax)) | 짧음|
| bus factor | [5(1)](https://github.com/mathjax/MathJax/graphs/contributors) | [10(1)](https://github.com/KaTeX/KaTeX/graphs/contributors)|

- 10번이상 commit한 사람으로 측정(실질적 영향력이 가장 큰 사람)

실 수식 표시의 차이

- Mathjax : 

![image](/assets/img/githubblog/mathjax.png)

- Katex : 

![image](/assets/img/githubblog/katex.png)


mathjax 는 마우스 우클릭시 여러가지 추가지원이 있으며 수식이 페이지 규격을 넘어서서 나타난다.

![image](/assets/img/githubblog/mathjax2.png)

![image](/assets/img/githubblog/mathjax3.png)

katex는 수식이 페이지 규격을 넘어서면 다음 줄에 이어서 나타난다.



![image](/assets/img/githubblog/katex2.png)






# Github blog

- 직접 블로그를 만들어서 쓰기위해 여러가지 스태틱 웹사이트 제너레이터가 존재한다.
  - Jekyll
  - Hugo
  - Mkdocs...
- 제한된 기능
- 편의성
- 직접 세팅
- 파일로서 직접 소유

여기에 서버로서 github blog를 사용 할 수 있다.

- 서버 유지 비용 X
- 도메인 비용 X
- 도메인 직접 설정가능

github blog과 jekyll로서 블로그 세팅을 하고 여기에 mathjax를 얹는 식으로 수식 지원을 하게 할 수 있다.


github blog 세팅은 굉장히 풀어져있는 자료정보가 많기에 각자 구글링에 맡기고 여기서는 mathjax 설정만 기술하도록 함.

참고 페이지

- [MathJax in Jekyll](https://quuxplusone.github.io/blog/2018/08/05/mathjax-in-jekyll/)
- [MathJax v3 in Jekyll](https://quuxplusone.github.io/blog/2020/08/19/mathjax-v3-in-jekyll/)

1. _includes 폴더 밑에 Mathjax.html 파일을 만들고 다음 내용을 넣는다.

```html
<script type="text/javascript">
  window.MathJax = {
    tex: {
      packages: ['base', 'ams']
    },
    loader: {
      load: ['ui/menu', '[tex]/ams']
    },
    startup: {
      ready() {
        MathJax.startup.defaultReady();
        const Macro = MathJax._.input.tex.Symbol.Macro;
        const MapHandler = MathJax._.input.tex.MapHandler.MapHandler;
        const Array = MathJax._.input.tex.ams.AmsMethods.default.Array;
        const env = new Macro('psmallmatrix', Array, [null, '(', ')', 'c', '.333em', '.2em', 'S', 1]);
        MapHandler.getMap('AMSmath-environment').add('psmallmatrix', env);
      }
    }
  };
  </script>
  <script type="text/javascript" id="MathJax-script" async
    src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-chtml.js">
  </script>
```
2. _layouts 폴더 밑의 default.html 맨밑에 Mathjax.html을 인클루드한다. 이때 body문 안에서 위에 지정하는것이아닌 제일 마지막에 인클루드를 해준다.

```html
  <body>
...
    {% include mathjax_support.html %}
  </body>

</html>
```

3. 마크다운을 kramdown로 세팅할 것. kramdown의 math blocks가 $$처리를 알아서 해준다.



# tex파일을 markdown으로

- pandoc
- 인터넷에 변환 사이트가 있음 그걸 이용하고 나머지는 수정. 


