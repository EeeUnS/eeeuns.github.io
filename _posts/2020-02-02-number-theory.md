---
layout: post
title:  "1 기초 number theory"
date:   2020-02-02 00:53:01 +0900
tag: CLRS
---

- [기초정수론](https://eeeuns.github.io/2020/02/01/number-theory/)
- [PrimalityTest](https://eeeuns.github.io/2020/02/01/primalitytest/)
- [CRT](https://eeeuns.github.io/2020/02/01/crt/)
- [carmichael function](https://eeeuns.github.io/2020/02/01/carmichael/)
- [RSA](https://eeeuns.github.io/2020/02/01/rsa/)









기초 정수론
===========

$$d,m,n,$$이 어떤 정수일 때, 다음이 성립한다.

1.  $$d$$가 $$m$$과 $$n$$의 공약수일때, $$m+n$$도 $$d$$의 배수이다.

2.  $$d$$가 $$m$$과 $$n$$의 공약수일때, $$m-n$$도 $$d$$의 배수이다.

이에 대한 증명은 간단하다 $$m=dq_1 , n = dq_2( q_1,q_2 \subset Z)$$라
하자. $$m+n = d(q_1 + q_2) , m-n=d(q_1-q_2)$$\

이 장을 이해하기위해서 약속 몇가지를 정의한다.

-   $$d$$가 $$n$$의 약수(인수)일 때 $$d \mid n$$으로 표시한다.

-   $$m$$과 $$n$$의 최대공약수는 $$\gcd(m,n)$$이라고 한다.

-   $$r$$이 $$a$$를 $$b$$로 나눈 나머지라면 $$r=a\bmod b$$이다.

이를써서 위 명제를 다시 적으면
$$d\mid n , d\mid m  \longrightarrow d \mid (m+n), d\mid (m-n)$$

$$a,b,z$$를 양의 정수라 하면, 다음이 성립한다.
$$ab\bmod z= [(a\bmod z)(b \bmod z)]\bmod z$$

$$w = ab\bmod z$$라 하자. 다음이 성립하는 $$q_1$$이 존재한다.

$$ab=q_1z+w \Longleftrightarrow w=ab-q_1 z$$

마찬가지로 $$x =a \bmod z, y=b\bmod z$$라 하면, 다음을 만족시키는 $$q_2$$와
$$q_3$$, $$q$$가 존재한다. $$a=q_2 z + x , b=q_3 z + y$$

$$\begin{aligned}
    w & = ab-q_1 z = (q_2z+x)(q_3z+y)-q_1z\\
    & =(q_2q_3z+q_2y+q_3x-q_1)z+xy\\
    & =qz+xy\end{aligned}$$

여기서 $$q=q_2q_3z+q_2y+q_3x-q_1$$이므로 $$xy=-qz+w$$ 즉 $$w$$는 $$xy$$를
$$z$$로 나눌 때의 나머지이다. 그러므로 $$w=xy \bmod z$$가 되고 이는 다음과
같이 나타낼 수 있다. $$ab\bmod z= [(a\bmod z)(b \bmod z)]\bmod z$$

이는 큰수를 인수분해해서 작은값으로 나눠서 큰수를 다루는 부담을
덜어주지만 지수승에 대해서도 응용이 가능하다. 이를 이용해서
$$a^{29}\bmod z$$를 계산하는 절차를 예시로 들어보겠다. $$a^{29}$$는 다음과
같은 순서로 계산한다.
$$a , a^{5}=a \cdot a^4, a^{13}=a^{5}\cdot a^{8}, a^{29}=a^{13}\cdot a^{16}$$
$$a^{29} \bmod z$$는 다음과 같은 순서로 계산한다.

$$a \bmod z , a^{5}\bmod z, a^{13}\bmod z, a^{29}\bmod z$$

$$\begin{aligned}
    a^2 \bmod z & = [(a\bmod z)(a\bmod z)]\bmod z \\
    a^4 \bmod z & = [(a^2\bmod z)(a^2\bmod z)]\bmod z \\
    a^8 \bmod z & = [(a^4\bmod z)(a^4\bmod z)]\bmod z \\
    a^{16} \bmod z & = [(a^8 \bmod z)(a^8\bmod z)]\bmod z \\
    a^5 \bmod z & = [(a \bmod z)(a^4\bmod z)]\bmod z \\
    a^{13} \bmod z & = [(a^5\bmod z)(a^8\bmod z)]\bmod z \\ 
    a^{29} \bmod z & = [(a^{13}\bmod z)(a^{16}\bmod z)]\bmod z\end{aligned}$$

유클리드 호제법(Euclidean algorithm)
====================================

$$a$$가 음이 아닌 정수이고, $$b$$가 양의 정수이며 $$r=a \bmod b$$이면 다음이
성립한다. $$\gcd(a,b) = \gcd(b,r)$$

수식이 익숙하지않은 분을 위해 풀어서 설명하자면, $$a$$가 음이 아닌
정수이고, $$b$$가 양의 정수이며, $$r$$이 $$a$$를 $$b$$로 나눈 나머지라면 $$a$$와
$$b$$의 최대공약수는 $$b$$와 $$r$$의 최대공약수와 같다.

$$a=bq +r (0 \le r\: <\: b , q$$ 는 어떤 정수)인데, $$c$$를 $$a$$와 $$b$$의
공약수라 하면, $$c$$는 $$bq$$의 약수인 것은 자명하다. $$a$$또한 $$c$$의
약수이므로 $$c$$는 $$a-bq\:(=r)$$의 약수이다. 따라서 $$c$$는 $$b$$와 $$r$$의
공약수입니다. 반대로 $$c'$$가 $$b$$와 $$r$$의 공약수이면, $$c'$$는 $$bq+r(=a)$$의
약수가 되고 따라서 $$a$$와 $$b$$의 공약수가 된다. 따라서 $$a$$와 $$b$$의 공약수
집합이 $$b$$와 $$r$$의 공약수 집합과 같으므로 $$\gcd(a,b) = \gcd(b,r)$$이
성립한다.

유클리드 알고리즘의 의의는 나머지 연산만을 이용해서 뺑뺑이 돌리면 어떻게
됬든지 간에 최대공약수를 기계적으로 구할수있다는 것에 있다.
$$\gcd(a,b) = \gcd(b,r)$$에서 b,r을 새로운 a,b로서 값을 넣어서 연속적으로
계산을 하면 언젠가 b가 0이 되는 순간이 오는데, 이때 a가 처음 a,b의 최대
공약수가 되는것이다.

$$\alpha > \beta$$일때, 다음이 성립한다.

$$\gcd(\alpha ,\beta) = \gcd(\alpha-\beta , \beta)$$

$$\alpha$$, $$\beta$$의 최대 공약수를 $$x$$라 하자.
$$\alpha = x \cdot a , \beta = x \cdot b$$ ($$a,b$$는 $$a>$$b이며 서로소인 두
정수)이며, $$\alpha -\beta = x(a-b)$$이다 $$a-b$$는 $$b$$와 서로소이며 두 값의
최대공약수는 여전히 $$x$$이다.

$$f(n) = 1+10+\cdots +10^n$$이라 하자.

$$ \gcd(f(x) , f(y)) = f(\gcd(x,y)) $$임을 보여라.

$$x > y$$라 하자.

$$\begin{aligned}
        f(x) - f(y) &= 10^x + 10^{x-1} + \cdots + 10^{y+1}\\
        &= 10^y(10^{x-y} + \cdots + 1)\\
        &= f(x-y) \cdot 10^y\\
        \gcd(f(x),f(y)) &= \gcd(f(x)-f(y),f(y)) \\
            &= \gcd(f(x-y) \cdot 10^{y},f(y))
    \end{aligned}$$

이때 $$10^{y}$$와 $$f(y)$$는 항상 서로소이므로 $$\gcd(f(x-y), f(y))$$가
성립한다. 따라서 유클리드 호제법을 전개했을때,
$$\gcd(f(x), f(y)) = \gcd(f(\gcd(x,y)),0)$$이 되고 이는$$f(\gcd(x,y))$$과
같다.

확장된 유클리드 알고리즘(Extended Euclidean algorithm)
======================================================

확장된 유클리드 알고리즘은 다음의 방정식에대해서 $$s$$와 $$t$$를 효율적으로
구하는 방법에대한 것이다.

$$a$$와 $$b$$가 음이 아니고 동시에 0이 아닌 정수라 하면 다음을 만족시키는
정수 $$s$$와 $$t$$가 존재한다.
$$\gcd(a,b) = s\cdot a + t\cdot b $$

선형 디오판토스 방정식이라고도 한다.

베주의 항등식

-------------

$$ax + by =\gcd(x, y)$$인 $$a$$, $$b$$가 존재한다.

집합 $$S = \left\lbrace m | m =ax+by> , x\in \mathbf{Z} , y \in  \right\rbrace$$를
생각해보면 ,이 집합 $$S$$는 $$S \subset \mathbf{Z}$$ ,
$$S \subset \varnothing$$ ( x, y를 원소로 가짐을 알 수 있다.) 이다. 또한,
자연수의 정렬성으로부터 최소가 되는 원소 $$d$$가 존재한다.

$$\alpha \in S \Rrightarrow \alpha = qd+r (0 \le r < d $$)라 하자.

만약 $$d \nmid \alpha$$ 일때, $$r > 0$$,
$$ r = \alpha - qd , (\alpha , d \in S)$$
$$\alpha = a_{1}x+b_{1}y , d=a_{2}x+b_{2}y$$라 하면.
$$r=(a_{1} - a_{2} q)x + (b_{1}-qb_{2})y \in S $$ $$0 < r < d$$인 $$r$$에 대해
$$d$$가 최소라는 가정이 모순이다.

$$\therefore r = 0 , d \mid \alpha (\forall \alpha \in S)$$,
$$ d \mid x, d \mid y \cdots$$ $$d$$는 $$x$$ , $$y$$의 공약수, $$\gcd(x, y)=k $$라
할때, $$d = akx^{''}+bky^{''}=k(ax^{''}+by^{''})$$ $$k \mid d$$에서 $$ k = d$$

활용
----

이미 증명되어있는 유클리드 알고리즘의 흐름을 통해서 예시로 이해 해보자.\
$$a=273$$ , $$b=110$$으로 하는 $$\gcd(273,110)$$을 구해보자.

$$r= 273\bmod  110 = 53 \cdot\cdots \mathit{1}$$

$$a=110 , b=53$$으로 지정

$$r= 110\:\bmod \: 53 = 4\cdot\cdots \mathit{2}$$

$$a=53 , b=4$$로 지정

$$r= 53\:\bmod \: 4 = 1 \cdot\cdots \mathit{3}$$

$$a=4 , b=1$$로 지정

$$r= 4\bmod  1 = 0\cdot\cdots \mathit{4}$$

$$r=0$$이므로 $$\gcd(273,110)$$은 최대공약수로 1을 가진다. 여기서
$$\mathit{4}$$ 식으로 되돌아가면 이는 다음과 같이 쓸 수 있다.

$$1=53 - 4\cdot13$$

계속 역순으로 뒤집어 올라가자 $$\mathit{3}$$

$$4=110 - 53\cdot2$$

이를 처음의 식에 대입하면

$$1=53 - (110 - 53\cdot2)\cdot13 =27\cdot53-13\cdot110 $$

$$\mathit{2}$$

$$53=273 - 110\cdot2$$

이 식을 다시 대입하면

$$1=27\cdot53-13\cdot110=27\cdot(273 - 110\cdot2)-13\cdot110=27\cdot273-67\cdot 110$$

따라서 $$s=27, t=-67$$로서 성립하는 값을 찾았다.

나머지 연산에서 곱셈에 대한 역원 (modular multiplicative inverse) [^1]
======================================================================

[Inverse] $$\gcd(n,\phi)=1$$인 두 정수 $$n>0, \phi>1$$가 있다고 하자.[^2]
$$n\cdot s\bmod  \phi =1 $$을 만족시키는 $$s$$를 $$n\bmod  \phi$$의
역원(inverse) 이라고 한다.

$$\gcd(n,\phi)=1$$임을 이용해, 확장된 유클리드 알고리즘을 이용하여
$$s'\cdot n + t \cdot \phi = 1$$이되는 $$s'$$과 $$t'$$을 구할수있다.
$$n\cdot s'= -t'\phi+1$$이 되고 $$\phi>1$$이므로 1이 나머지가 된다.
$$n\cdot s'\bmod  \phi =1$$에서 $$s= s'\bmod  \phi$$라 하면
$$0 \le s <\phi$$가 되며 또한 $$s  \ne 0$$이다.\
위 식을 변형하면, $$s'=q\cdot \phi +s$$ 가되며 이를 만족하는 정수 q가
존재한다.\
따라서

$$n\cdot s=ns'-\phi nq=-t'\phi +1 -\phi nq=\phi(-t'-nq)+1 $$

따라서 $$n\cdot s\bmod  \phi =1 $$이 된다.

오일러의 $$\phi$$함수(Euler’s phi (totient) function)
===================================================

[phi function] 양의 정수 $$n$$에 대해서

$$\phi (n)$$ : 1부터 n까지의 양의 정수 중에 n과 서로소인 것의 개수를
나타내는 함수.

$$\phi (n)$$은 다음의 성질이 있다.

-   <span>소수 $$p$$에 대해서 $$\phi (p)=p-1$$</span>

-   <span> m, n이 서로소인 양의 정수일 때, 다음이 성립한다.
    $$\phi (mn)=\phi (m)\phi (n)$$</span>

-   소수 $$p$$와 양의 정수 $$\alpha$$에 대해 다음이 성립한다.
    $$\phi(p^\alpha) = p^\alpha \left ( 1 - \frac{1}{p} \right )$$

첫번째 성질은 어찌보면 당연하다 $$p$$는 소수이니 자기 자신을 제외한 모든
수와 서로소이다 (여기서 1도 세야한다.)

두번째 성질은 두수의 곱 $$mn$$은 각각 $$m$$에대해서 나눠지는 수가 $$n$$개이고
n에 대해서 나눠지는 수가 $$m$$개 이며 $$mn$$으로 나눠지는 수가 한 개이므로
$$mn -\dfrac{mn}{m}-\dfrac{mn}{n}+\dfrac{mn}{mn} =mn -m -n +1=(m-1)(n-1)=\phi (m)\phi(n)$$가
된다.

세번째 성질 $$p^\alpha$$보다 같거나 작은 $$p$$의 배수가 되는것은 다음이
있다. $$p , 2p , 3p , ... , p^{\alpha-1}p$$ 따라서 총 $$p^{\alpha-1}$$개가
있고
$$ \phi(p^\alpha) =p^{\alpha} -  p^{\alpha-1} = p^\alpha \left ( 1 - \frac{1}{p} \right )$$

*If $$m_1, m_2, \ldots, m_k$$ are $$k$$ positive integers which are prime
each to each, then*

$$\phi(m_1 m_2 \ldots m_k) = \phi(m_1) \phi(m_2) \ldots \phi(m_k).$$

*If $$m = p_1^{\alpha_1} p_2^{\alpha_2} \ldots p_n^{\alpha_n}$$ where
$$p_1, p_2, \ldots, p_n$$ are different primes and $$\alpha_1,
\alpha_2, \ldots, \alpha_n$$ are positive integers, then*

$$\phi(m) = m \left ( 1-\frac{1}{p_1} \right )
            \left ( 1-\frac{1}{p_2} \right )
            \ldots
            \left ( 1-\frac{1}{p_n} \right ).$$

For,

$$\begin{aligned}
\phi(m) &= \phi(p_1^{\alpha_1}) \phi(p_2^{\alpha_2}) \ldots
             \phi(p_n^{\alpha_n}) \\
        &= p_1^{\alpha_1} \left ( 1-\frac{1}{p_1} \right )
             p_2^{\alpha_2} \left ( 1-\frac{1}{p_2} \right )
             \ldots
             p_n^{\alpha_n} \left ( 1-\frac{1}{p_n} \right ) \\
        &= m \left ( 1-\frac{1}{p_1} \right )
             \left ( 1-\frac{1}{p_2} \right )
             \ldots
             \left ( 1-\frac{1}{p_n} \right ).\end{aligned}$$

오일러 정리(Euler’s theorem)[^3]
================================

임의의 정수 a와 n이 서로소일 때, 다음이 성립한다.
$$a^{\phi(n)} \bmod n = 1$$

정수 n에 대해서 1부터 n까지의 양의 정수 중에 n과 서로소인 것의 집합을
생각해보자. 그러면 이는 집합

$$A = \left\lbrace r_1 ,r_2,r_3, \cdots ,r_{\phi(n)}\right\rbrace$$[^4]

으로 나타낼 수 있다. 이 집합은 A라하고 이 각 원소에 n과 서로소인 a를
곱한 집합을 B집합이라 하자.
$$B = \left\lbrace ar_1 ,ar_2,ar_3, \cdots ,ar_{\phi(n)}\right\rbrace$$ 확실한건 $$B$$에 있는
모든 원소는 $$n$$과 서로소인 것이다. 그럼 $$B$$집합의 각 원소를 $$\bmod n$$에
대해 계산한 것을 생각해보자. 이는 각 원소의 나머지가 a를 곱하기전 값과
같은지는 모르지만 $$\phi(n)$$개에 대해서 각각 일대일대응이 가능 한다는것을
알수있다. [^5] 따라서 $$A$$의 모든 원소를 곱한 값에 $$\bmod n$$을 한것과
$$B$$의 모든 원소를 곱한 값에 $$\bmod n$$을 한 값은 같다.
$$ar_1 \cdot ar_2 \cdot ar_3 \cdots ar_{\phi_{n}} \equiv r_1 \cdot r_2 \cdot r_3 \cdots r_{\phi_{n}} \pmod n$$
$$a^{\phi(n)}\bmod n= 1$$

페르마의 소정리 : 소수 $$p$$에 대해 다음이 성립한다.
$$a^{p-1} \equiv 1 \pmod{p}$$

$$\phi(p)  = p-1 $$이므로 오일러 정리에의해 성립합을 알 수 있다.

$$a^{p-1} \equiv 1 \bmod p.$$

[^1]: 역원: a와 연산자에 대해 연산결과가 항등원($$=1$$)이 되는 유일한 원소
    b를 a의 역원이라한다.

[^2]: 한 마디로 n과 $$\phi$$는 서로소이다.

[^3]: 페르마의 소정리는 오일러 정리에서의 특수한 경우이다.

[^4]: 이러한 집합을 기약잉여계 $$\mathbb Z_n^*$$라고 부른다. 또한 집합 A의
    원소의 갯수는 $$\phi(n)$$dl다.

[^5]: 실제 증명은 귀류법을 통해서
    증명할수있다.$$ar_i  \equiv ar_j \bmod n $$ 인
    $$1 \le i < j \le \phi(n)$$ 이 존재한다고 가정해보자.

