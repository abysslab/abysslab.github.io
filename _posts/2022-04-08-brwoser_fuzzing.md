---
layout: post
author: "smlijun"
title: Fuzzing JavaScript Engine 
---

Introduction
---
최근 브라우저에 대한 공부를 진행하면서 브라우저에 `Fuzzing`을 진행할 수 있는 방법에 대해서 연구를 진행해보기로 마음을 먹게 되었다. 그동안 자주 사용했던 fuzzer들은 `AFL`과 같은 바이너리 포맷의 input을 타겟으로하기 좋은 fuzzer들만 사용해보았다. AFL을 수정해서 `RDP`를 fuzzing하거나 `Harness`작성을 통해 Fuzzing을 수행해본 경험은 있지만 Javascript와 같은 Syntax가 존재하는 input을 mutation하는 방식은 거의 수행해본 경험이 없다. 따라서 이번글을 어떻게 브라우저를 Fuzzing 할 수 있는지, 특히 브라우저 역시 Fuzzing을 많이 당한? 프로그램중 하나기 때문에 새로운 Fuzzing 아이디어가 필요하다.

Fuzzilli
---
현재 가장 널리 사용되는 `javascript`에 대한 `state-of-art fuzzer`이다. 

Fuzzing V8 with Fuzzilli
---







































Reference
---
- [Project Zero Post on blackbox javascript](https://googleprojectzero.blogspot.com/2021/09/fuzzing-closed-source-javascript.html)  

Related Paper
---
- [Favocado for Fuzzing the Binding Code of JavaScript Engines](https://www.ndss-symposium.org/wp-content/uploads/ndss2021_6A-2_24224_paper.pdf)
- [FreeDom for Engineering a State-of-the-Art DOM Fuzzer](https://dl.acm.org/doi/pdf/10.1145/3372297.3423340)

