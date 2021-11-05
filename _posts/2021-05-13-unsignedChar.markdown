---
layout: post
title:  "mem- 함수의 unsigned char변환"
date: 2021-05-13 02:53:01 +0900
tag: c
---


mem 관련 함수 명세들을 보면은 얘들을 무조건 converted to unsigned char 로 바꿔서 쓰라는 명시가 있었다. 
근데 이 데이터들을 읽어서 출력하는게아닌데 굳이 char이아닌 unsigned라고 명시했는지 궁금해서 이것저것 찾아봤는데 그러다 찾은 재밌는것들

정리하면
1. char / unsigned char / signed char 은 모두 다른 타입
2. C 표준에서 char 이  unsigned인지 signed인지는 언급이없고 프로세서마다 다름
3. C 표준에서 1 바이트씩 접근하는 함수들은 unsigned char로 접근하라고 명시되어있음

3의 이유로 문제가 안생길수도있지만 문제가 생길수있는 경우가 다분하기에 일괄하여 unsigned로 통일


참고

- https://jacking.tistory.com/149
- https://dojang.io/mod/forum/discuss.php?d=1459
- https://kldp.org/node/75686
