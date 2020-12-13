---
layout: post
title:  "4 Carmichael function"
date:   2020-02-02 03:53:01 +0900
tag: math algorithm
---


- [기초정수론](https://eeeuns.github.io/2020/02/01/number-theory/)
- [PrimalityTest](https://eeeuns.github.io/2020/02/01/primalitytest/)
- [CRT](https://eeeuns.github.io/2020/02/01/crt/)
- [carmichael function](https://eeeuns.github.io/2020/02/01/carmichael/)
- [RSA](https://eeeuns.github.io/2020/02/01/rsa/)


===================



# 추가 정보

카마이클 함수는 두가지 [정의](https://mathworld.wolfram.com/CarmichaelFunction.html)가 있다.
따라서 이 함수를 사용하는 문서마다 정의를 명시해주어야한다.
ver 1. 

$$\lambda(n) = \operatorname{lcm}(\phi(p_1^{e_1}), \ldots, \phi(p_r^{e_r})).$$

ver 2. 


$$
\begin{aligned}
\lambda(2^{\alpha}) &= \phi(2^{\alpha}) \text{ if a = 0, 1, 2;} \\
\lambda(2^{\alpha}) &= \frac{1}{2}\phi(2^{\alpha})
\text{ if a > 2 ;} \\
\lambda(p^{\alpha}) &= \phi(p^{\alpha})
\text{ if p is an odd prime;} \\
\lambda(2^{\alpha} p_1^{\alpha_1} p_2^{\alpha_2} \cdots p_n^{\alpha_n}) 
&= \text{lcm}(
    \lambda(2^{\alpha}),
    \lambda(p_1^{\alpha_1}),
    \lambda(p_2^{\alpha_2}), \ldots, \lambda(p_n^{\alpha_n}
    )
)\end{aligned}
$$


[ref](http://www.gutenberg.org/files/13693/13693-pdf.pdf)

Then let us write

$$a^{p-1} = 1 + hp.  (1)$$


Raising each member of this equation to the $$p^{\text{th}}$$ power we may write the result in the form


$$a^{p(p-1)} = 1 + h_1p^2.  (2)$$


where $$h_1$$ is an integer. 

Hence

$$a^{p(p-1)} \equiv 1 \bmod p^2.  $$


By raising each member of (2) to the $$p^{\text{th}}$$


power we can readily show that


$$a^{p^2(p-1)} \equiv 1 \bmod p^3.  $$

It is now easy to see that we shall have in general

$$a^{p^{\alpha - 1}(p-1)} \equiv 1 \bmod p^{\alpha}.  $$

where $$\alpha$$ is a positive integer; that is,

$$a^{\phi(p^{\alpha})} \equiv 1 \bmod p^{\alpha}.$$

For the special case when $$p$$ is 2 this result can be extended. For this
case (1) becomes

$$ a = 1 + 2h. $$

Squaring we have

$$ a^2 = 1 + 4h(h+1). $$

Hence,

$$ a^2 = 1+8h_1, (3)$$

where $$h_1$$ is an integer. Therefore

$$ a^2 \equiv 1 \bmod 2^3. $$

Squaring (3) we have

$$ a^{2^2} = 1 + 2^4h_2; $$

or}

$$ a^{2^2} \equiv 1 \bmod 2^4. $$

It is now easy to see that we shall have in general

$$ a^{2^{\alpha-2}} \equiv 1 \bmod 2^{\alpha} $$

if $$\alpha > 2$$. That is,

$$a^{\frac{1}{2}\phi(2^{\alpha})} \equiv 1 \bmod 2^{\alpha} \text{ if } a > 2.$$


Now in terms of the $$\phi$$-function let us define a new function
$$\lambda(m)$$ as follows:

$$
\begin{aligned}
\lambda(2^{\alpha}) &= \phi(2^{\alpha}) \text{ if a = 0, 1, 2;} \\
\lambda(2^{\alpha}) &= \frac{1}{2}\phi(2^{\alpha})
                                               \text{ if a > 2;} \\
\lambda(p^{\alpha}) &= \phi(p^{\alpha})
                                   \text{ if p is an odd prime;} \\
\lambda(2^{\alpha} p_1^{\alpha_1} p_2^{\alpha_2} \cdots p_n^{\alpha_n}) 
&= \text{lcm}(
    \lambda(2^{\alpha}),
    \lambda(p_1^{\alpha_1}),
    \lambda(p_2^{\alpha_2}), \ldots, \lambda(p_n^{\alpha_n}
    )
)\end{aligned}
$$

$$2, p_1, p_2, \ldots, p_n$$ being different primes.

Denote by $$m$$ the number

$$m = 2^{\alpha}p_1^{\alpha_1}p_2^{\alpha_2} \cdots p_n^{\alpha_n}.$$

Let $$a$$ be any number prime to $$m$$. From our preceding results we have

$$\begin{aligned}
a^{\lambda(2^{\alpha})}     &\equiv 1 \bmod 2^{\alpha}, \\
a^{\lambda(p_1^{\alpha_1})} &\equiv 1 \bmod p_1^{\alpha_1},\\
a^{\lambda(p_2^{\alpha_2})} &\equiv 1 \bmod p_2^{\alpha_2}, \\
\ldots \\
a^{\lambda(p_n^{\alpha_n})} &\equiv 1 \bmod p_2^{\alpha_n}. \\\end{aligned}$$

Now any one of these congruences remains true if both of its members are
raised to the same positive integral power, whatever that power may be.
Then let us raise both members of the first congruence to the power
$$\frac{\lambda(m)}{\lambda(2^\alpha)}$$; both members of the second
congruence to the power $$\frac{\lambda(m)}{\lambda(p_1^{\alpha_1})}$$;
$$\ldots$$; both members of the last congruence to the power
$$\frac{\lambda(m)}{\lambda(p_n^{\alpha_n})}$$. Then we have

$$ a^{\lambda(m)} \equiv 1 \mod 2^\alpha, $$

$$ a^{\lambda(m)} \equiv 1 \mod p_1^{\alpha_1}, $$

$$ \ldots \ldots $$

$$ a^{\lambda(m)} \equiv 1 \mod p_n^{\alpha_n}. $$

From these congruences we have immediately

$$a^{\lambda(m)} \equiv 1 \mod m.$$

We may state this result in full in the following theorem:

*If $$a$$ and $$m$$ are any two relatively prime positive integers, the
congruence*

$$a^{\lambda(m)} \equiv 1 \mod m.$$

*is satisfied.*

As an excellent example to show the possible difference between the
exponent $$\lambda(m)$$ in this theorem and the exponent $$\phi(m)$$ in
Fermat’s general theorem, let us take

$$m = 2^6 \cdot 3^3 \cdot 5 \cdot 7 \cdot 13 \cdot 17 \cdot 19\cdot 37 \cdot 73. $$

Here

$$\lambda(m) = 2^4 \cdot 3^2, \quad \phi(m) = 2^{31} \cdot 3^{10}.$$

In a later chapter we shall show that there is no exponent $$\nu$$ less
than $$\lambda(m)$$ for which the congruence

$$a^\nu = 1 \mod m$$

is verified for every integer $$a$$ prime to $$m$$.

From our theorem, as stated above, Fermat’s general theorem follows as a
corollary, since $$\lambda(m)$$ is obviously a factor of $$\phi(m)$$,

$$\phi(m) = \phi(2^\alpha) \phi(p_1^{\alpha_1}) \ldots
               \phi(p_n^{\alpha_n}).$$

Solve 31.8-2(CLRS)
------------------

It is possible to strengthen Euler’s theorem slightly to the form

$$a^{\lambda(n)} \equiv 1 (\mod n)$$ for all $$a \in \mathbb Z_n^*$$,

where $$n = p_1^{e_1} \cdots p_r^{e_r}$$ and $$\lambda(n)$$ is defined by

$$\lambda(n) = \operatorname{lcm}(\phi(p_1^{e_1}), \ldots, \phi(p_r^{e_r})).$$

Prove that $$\lambda(n) \mid \phi(n)$$. A composite number $$n$$ is a
Carmichael number if $$\lambda(n) \mid n - 1$$. The smallest Carmichael
number is $$561 = 3 \cdot 11 \cdot 17$$; here,
$$\lambda(n) = \text{lcm}(2, 10, 16) = 80$$, which divides $$560$$. Prove
that Carmichael numbers must be both “square-free” (not divisible by the
square of any prime) and the product of at least three primes. (For this
reason, they are not very common.)

### ENG

1.  Prove that $$\lambda(n) \mid \phi(n)$$.

    $$n = p_1^{e_1} \cdots p_r^{e_r}$$

    $$\phi(n) = \phi(p_1^{e_1})* \ldots*\phi(p_r^{e_r})$$

    $$\operatorname{lcm}(\phi(p_1^{e_1}, \ldots, \phi(p_r^{e_r})) | (\phi(p_1^{e_1})* \ldots*\phi(p_r^{e_r}))$$

    $$\lambda(n) \mid \phi(n)$$

2.  Prove that Carmichael numbers must be both “square-free” (not
    divisible by the square of any prime)

    let Carmichael number $$n = p^\alpha m( \alpha \ge 2 ,  p \nmid m )$$
    $$a^{n-1} \equiv 1 \pmod{n} (\gcd(a,n) = 1)$$

    set $$a = p+1 $$ then $$(p+1)^{n} \equiv p+1 \pmod{n}$$

    and $$gcd(p^2,a) = 1$$

    $$(p+1)^{n} \equiv (p+1)^{p^2 p^{\alpha-2}} \equiv p+1 \pmod{p^2}$$

    but $$\gcd(p^2,a) = 1$$, $$ a \equiv 1 \pmod{p^2}$$

    $$p+1 \equiv 1 \pmod{p^2}$$ This is impossible

    <https://math.stackexchange.com/questions/1764812/carmichael-number-square-free>

3.  the product of at least three primes.

    Assume that $$n=pq$$, with $$p<q$$ two distinct primes, is a Carmichael
    number. Then we have
    $$q≡1 \pmod{q−1} )\rightarrow n \equiv pq \equiv p \pmod{q−1}  \rightarrow n−1 \equiv p−1 \pmod{q−1}$$
    Here $$0 < p−1 < q−1$$ , so $$n−1$$ is not divisible by $$q−1$$.

    <https://math.stackexchange.com/questions/432162/carmichael-proof-of-at-least-3-factors>

### KR

​2. Prove that Carmichael numbers must be both “square-free” (not
divisible by the square of any prime)

카마이클 수 $$n = p^\alpha m( \alpha \ge 2 ,  p \nmid m )$$라 하자 정의에
따라 다음이 성립한다. $$a^{n} \equiv a \pmod{n}$$.

$$ a \equiv 1 + p \pmod{p^\alpha}$$ 라 하자.

$$(p+1)^{n} \equiv p+1 \pmod{m}$$이 되는데. $$m$$의 인자인 $$p^2$$와에
대해서도 다음이 성립한다.
$$(p+1)^{n} \equiv (p+1)^{p^\alpha m} \equiv (p+1)^{p^2 p^{\alpha-2} m}  \equiv p+1 \pmod{p^2}$$
그러나 $$\gcd(a,n) = 1$$ 이라서 $$\gcd(a, p^2) = 1$$이다.
$$(p+1)^{n} \equiv p+1  \equiv 1 \pmod{p^2}$$이며 이는 모순이다.

​3. the product of at least three primes.

서로다른 두 소수의 곱이 카마이클 수가 될 수 없음을 보이는것으로
충분하다.

$$n=pq$$라 하자( $$p<q$$인 서로다른 소수)

$$q \equiv 1 \pmod{q - 1} )\rightarrow n \equiv pq \equiv p \pmod{q - 1}$$
$$\rightarrow n - 1 \equiv p - 1 \pmod{q - 1}$$

$$0 < p-1 < q - 1$$ , $$n - 1$$는 $$q - 1$$로 나눠질수 없다.
