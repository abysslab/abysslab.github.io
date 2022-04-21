---
layout: post
author: "smlijun"
title: Fuzzing JavaScript Engine - Part 1 Fuzzer Research
published: false
---



Introduction
---
최근 브라우저에 대한 공부를 진행하면서 브라우저에 `Fuzzing`을 진행할 수 있는 방법에 대해서 연구를 진행해보기로 마음을 먹게 되었다. 그동안 자주 사용했던 fuzzer들은 `AFL`과 같은 바이너리 포맷의 input을 타겟으로하기 좋은 fuzzer들만 사용해보았다. AFL을 수정해서 `RDP`를 fuzzing하거나 `Harness`작성을 통해 Fuzzing을 수행해본 경험은 있지만 Javascript와 같은 Syntax가 존재하는 input을 mutation하는 방식은 거의 수행해본 경험이 없다. 따라서 이번글을 어떻게 브라우저를 Fuzzing 할 수 있는지, 특히 브라우저 역시 Fuzzing을 많이 당한? 프로그램중 하나기 때문에 새로운 Fuzzing 아이디어가 필요하다.


Challenge for JS mutation
---
일반적인 바이너리 포맷에 대한 input들은 AFL이 사용하는 `Havoc`, `filp`등과 같은 일종의 `brute force방식`으로도 충분히 유의미한 testcase들을 생성해 낼 수 있다. 하지만 javascript의 경우 seed data에서 단순히 1바이트가 변경되었다고해서 유의미하게 다른 javscript가 될 수 없다. 오히려 `javascript`의 `syntax`가 깨져버리는 상황만 발생할 가능성이 높다. 예를 들면 아래와 같은 경우가 있다.
```js
var name = prompt('test'); // seed
var name = prompt('t!st'); // correct but same with seed
bar name = prompt('test'); // incorrect
```
따라서 syntax가 맞는 testcase를 위한 새로운 mutator가 필요해진다. 따라서 현재 많이 알려진 `Domato`, `Fuzzilli`, 그리고 최근에 논문급에서 개발된 `freedom`을 사용해보고 내부 구조 공부를 선행해보고자 한다.


Domato
---
`Domato`는 사용자가 정의한 rule에 따라 문법적으로 유효한 testcase를 만들어주는 mutator이다. 따로 타겟 JS engine에 값을 입해주는 부분은 사용자가 따로 구현해야한다. 또한 말 그대로 "문법적"으로만 유효한 testcase를 만들어낸다. 따라서 문법적으로는 문제가 없더라도 `Semantic`적으로 문제가 있을 확률이 있다. domato에서는 이러한 문제를 해결해주기 위해서 `try-catch()`문을 통해 에러가 발생하여도 뒤의 코드가 정상적으로 실행되게 해준다. 하지만 이러한 방식도 `JIT compile`이 되는 경우 아예 다른 코드가 되게 되면서 여러 한계점이 존재한다.

Generate Testcase with Domato
---
`Domato`를 사용하는 방식은 간단하다. 먼저 `Domato`를 clone해준다.
```sh
git clone https://github.com/googleprojectzero/domato.git
```
단순히 기본적으로 제공되는 rule을 이용해서 testcase를 생성하는 방법은 아래와 같다.
```sh
python3 generator.py --output_dir <output directory> --no_of_files <number of output files>
```
아래는 domato를 이용하여 10000개의 testcase들을 생성해낸 결과이다.
![domato](aaaaaa)

Fuzzilli
---
Domato의 단점들을 보안한  `javascript`에 대한 `fuzzer`이다. 

Fuzzing V8 with Fuzzilli
---
먼저 ```Ubuntu 20.04```에서 진행하였다. 먼저 clang-4.0 이상 버전의 clang을 설치해준다.
```sh
sudo apt install clang
```
이후 swift를 설치해준다.
```sh
wget https://swift.org/builds/swift-5.3.3-release/ubuntu2004/swift-5.3.3-RELEASE/swift-5.3.3-RELEASE-ubuntu20.04.tar.gz
mkdir ~/swift
tar -xvf swift-5.3.3-RELEASE-ubuntu20.04.tar.gz -C ~/swift
export PATH=~/swift/swift-5.3.3-RELEASE-ubuntu20.04/usr/bin/:$PATH
```
이후 Github의 Fuzzilli 코드를 가져와서 빌드해준다.
```sh
git clone https://github.com/googleprojectzero/fuzzilli.git
cd fuzzilli
swift build -c release --enable-test-discovery
```
Fuzzilli은 여러 `javascript engine`에 적용이 가능한데 현재 공부하는 쪽이 `chromium`이기 떄문에 `V8`에 Fuzzilli를 적용하려고 한다.
아래와 같은 방식으로 `V8`의 소스코드를 다운로드 하고 fuzzing가능한 형태로 빌드 할 수 있다.
```sh
git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git
export PATH="$PATH:${HOME}/depot_tools"

mkdir ~/v8
cd ~/v8
fetch v8
cd v8

build/install-build-deps.sh
%PATH to Fuzzilli%/Targets/V8/fuzzbuild.sh
sudo sysctl -w 'kernel.core_pattern=|/bin/false'
```
아래와 같은 command로 fuzzer를 실행시킬 수 있다.
```sh
swift run --enable-test-discovery FuzzilliCli --profile=v8 ~/v8/v8/out/fuzzbuild/d8
```
![fuzzilli](https://raw.githubusercontent.com/abysslab/abysslab.github.io/main/img/fuzzilli_default.png){:style="display:block; margin-left:auto; margin-right:auto"}



Freedom
---


































Reference
---
- [Project Zero Post on Domato](https://googleprojectzero.blogspot.com/2017/09/the-great-dom-fuzz-off-of-2017.html)  

Related Paper
---
- [Favocado for Fuzzing the Binding Code of JavaScript Engines](https://www.ndss-symposium.org/wp-content/uploads/ndss2021_6A-2_24224_paper.pdf)
- [FreeDom for Engineering a State-of-the-Art DOM Fuzzer](https://dl.acm.org/doi/pdf/10.1145/3372297.3423340)

