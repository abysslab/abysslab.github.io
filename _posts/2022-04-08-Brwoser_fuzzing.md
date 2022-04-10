---
layout: post
author: "smlijun"
title: Fuzzing JavaScript Engine - Part 1 Fuzzilli
---

---

Introduction
---
최근 브라우저에 대한 공부를 진행하면서 브라우저에 `Fuzzing`을 진행할 수 있는 방법에 대해서 연구를 진행해보기로 마음을 먹게 되었다. 그동안 자주 사용했던 fuzzer들은 `AFL`과 같은 바이너리 포맷의 input을 타겟으로하기 좋은 fuzzer들만 사용해보았다. AFL을 수정해서 `RDP`를 fuzzing하거나 `Harness`작성을 통해 Fuzzing을 수행해본 경험은 있지만 Javascript와 같은 Syntax가 존재하는 input을 mutation하는 방식은 거의 수행해본 경험이 없다. 따라서 이번글을 어떻게 브라우저를 Fuzzing 할 수 있는지, 특히 브라우저 역시 Fuzzing을 많이 당한? 프로그램중 하나기 때문에 새로운 Fuzzing 아이디어가 필요하다.

Fuzzilli
---
현재 가장 널리 사용되는 `javascript`에 대한 `state-of-art fuzzer`이다. 

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






































Reference
---
- [Project Zero Post on blackbox javascript](https://googleprojectzero.blogspot.com/2021/09/fuzzing-closed-source-javascript.html)  

Related Paper
---
- [Favocado for Fuzzing the Binding Code of JavaScript Engines](https://www.ndss-symposium.org/wp-content/uploads/ndss2021_6A-2_24224_paper.pdf)
- [FreeDom for Engineering a State-of-the-Art DOM Fuzzer](https://dl.acm.org/doi/pdf/10.1145/3372297.3423340)

