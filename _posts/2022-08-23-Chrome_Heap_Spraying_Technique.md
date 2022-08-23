---
layout: post
author: "nevul37"
title: Heap Spraying Technique for exploit Chrome UaF Notes
---
[reference1](https://paper.seebug.org/1876/)  
 └ [origin](https://mp.weixin.qq.com/s/tGwCwOQ8eAwm26fHXTCy5A)

Introduction  
---
국내에는 `Browser Exploit`에 대한 한국어 자료가 매우 적다. 1day를 공부하다 보면 RCE는 `Type Confusion을 이용한 Heap Overflow`나 `특정 객채에서 일어나는 UaF`가 대부분이다. 
이는 Sandbox Escaping도 마찬가지다. 특히 Sandbox Escaping은 자료가 극히 적은 편에 속하고 Javascript PoC도 공개되지 않은 경우가 상당하여,
어떤 방식으로 `Heap Spraying`을 하는지 이해하기에 어려움이 많다. 그렇기에 Chrome RCE/SBX 1day를 공부하면서 알게 된 `Heap Spraying Technique` 방법을 정리해보려고 한다.  
__※ 오개념이 있을 수 있습니다__

Issue 1062091
---
