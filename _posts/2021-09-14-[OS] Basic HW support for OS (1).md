---
layout: post
title:  "[OS] Basic H/W supports for OS (1)"
date:   2021-09-14 01:41:27 +0530
categories: 
---

> 이 글은 포스텍 박찬익 교수님의 CSED312 운영체제 수업의 강의 내용과 자료를 기반으로 하고 있습니다.

 1주차다 보니 Operating system에 대해 제대로 배우기 전, 기본적으로 알아야 할 HW 지식들을 우선 배운다.


 ## Bootstrapping OS kernel
 Bootstrapping. 우리가 흔히 Booting이라 알고 있는 말과 같은 뜻이다.
 Operating System(OS)를 사용하기 위해 컴퓨터의 전원을 킨 순간부터 OS kernel이 가동되기 까지의 과정을 알아보자.

 (이미지 삽입)

우리가 컴퓨터의 전원을 켜게 되면, HW(CPU)가 pre-define된 memory address에 접근하게 된다. ROM속에 존재하는 이 address 상에서 CPU는 Bootstrap code인 BIOS를 찾게 된다.

BIOS 프로그램이 돌아가기 시작하면, CPU는 또 다른 bootstrap code인 bootloader를 찾게 된다. 그후, 찾은 bootloader의 data와 instruction들에 대해 disk에서 physical memory로 load하게 된다. CPU는 bootloader로 jump하여 OS kernel의 탐색을 시작한다.

OS kernel을 찾게 되면 다시 이전처럼 disk에서 physical memory로 data와 instruction들을 load해온다. 그 후 CPU가 OS kernel로 jump해, OS kernel이 running되며 일련의 bootstrapping과정이 끝이 나는 것이다.

* 따라서 이 bootstrapping 과정은 BIOS가 bootloader를 load해오는 phase와 bootloader가 OS kernel을 load해오는 phase로 구성된다 볼 수 있다.

## Running User Application - Login Shell
 Bootstrapping 과정이 끝나면, OS kernel은 initialization step을 거친다.
 
 // 여기 수정 필요
 * 이에 대해 다루는 ppt 슬라이드는 없지만, 이후 2번째 시간에 교수님이 설명하시기론 이 initialization step을 통해 OS exception handler와 exception table등을 만든다고 한다.data structure도 initialize된다.

 Initialize를 끝내면, OS는 Login Shell이라는 이름의 user application을 찾게 된다.
 마찬가지로 disk에서 physical memory로 정보를 load한 뒤, CPU가 해당 위치로 jump해서 이 login application이 running되기 시작한다.

 Login application이 시작되면 우리가 소위 말하느 "Shell" 화면이 보이게 된다.

 (이미지 삽입)

 사용자는 Shell이라는 interface를 통해 command를 OS kernel에 전달할 수 있게 된다.
 
 예시 그림처럼 user command를 작성하면, Login Shell이 command를 OS에 request한다.
 그러면 OS는 다음과 같은 처리 절차를 진행한다.
 1. app code와 data를 disk에서 찾는다.
 2. (physical) memory를 할당한다.
 3. (physical) memory에 load한다.
 4. CPU가 해당 app을 run할 수 있도록 환경을 configure한다.

## Executable Image
 지금까지 user application의 실행까지의 과정을 보았으니, 이제 실제 code와 data가 어떻게 build 되는지 알아본다.

 (이미지 삽입)

 실제 code는 대부분 high level language(ex. C언어)로 작성되어 있다. 이 code가 compiler와 assembler를 거치면 우린 executable image를 얻을 수 있게 된다.
 Executable image란 CPU가 code에서 얻어낸 instruction들로 이루어진, disk에 위치한 file을 뜻한다. (즉, 실행파일인 셈이다.) // 확인 필요

 OS는 이 executable file을 이용해 excutable한 app을 찾고, memory를 할당한 뒤, disk로부터 load하여 CPU로 execute할 수 있게 된다.

 * Executable image는 user code외에 uninitialized data를 가지고 있어 CPU가 이를 execute하기 위해선 heap과 stack의 추가적인 memory 할당이 요구된다.

## Execute Code by CPU : Control Flow


