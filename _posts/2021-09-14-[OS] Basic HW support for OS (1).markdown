---
layout: post
title:  "[OS] Basic H/W supports for OS (1)"
date:   2021-09-14 01:41:27 +0530
categories: 
---

> 이 글은 포스텍 박찬익 교수님의 CSED312 운영체제 수업의 강의 내용과 자료를 기반으로 하고 있습니다.

---  
<br>
 　1주차에선 Operating system에 대해 제대로 배우기 전, 기본적으로 알아야 할 HW 지식들을 우선 배운다.
<br><br>
 
 
 ## Bootstrapping OS kernel
 
 <br>
 　Bootstrapping은 우리가 흔히 Booting(시동)이라 알고 있는 말과 같은 뜻을 가진다.<br>
 이는 Operating System(OS)를 사용하기 위해 컴퓨터의 전원을 킨 순간부터 OS kernel이 가동되기 까지의 과정을 나타낸다.
<br><br>

 (이미지 삽입)

<br>
　우리가 컴퓨터의 전원을 켜게 되면, HW(CPU)는 pre-define된 memory address에 접근하게 된다. ROM속에 존재하는 이 address 상에서 CPU는 Bootstrap code인 BIOS를 찾게 된다.
<br><br>
　BIOS 프로그램이 돌아가기 시작하면, CPU는 또 다른 bootstrap code인 bootloader를 찾게 된다. 그후, disk에서 physical memory로 찾은 bootloader의 data와 instruction들에 대해 load를 하여, CPU는 가져온 bootloader로 jump하여 OS kernel의 탐색을 시작한다.
<br><br>
　OS kernel을 찾게 되면 다시 이전처럼 disk에서 physical memory로 data와 instruction들을 load해오고, CPU가 OS kernel로 jump하게 된다. OS kernel이 running되기 시작하며 여기까지가 Bootstrapping에 해당된다.
<br><br>
　이 bootstrapping 과정은 BIOS가 bootloader를 load해오는 phase와 bootloader가 OS kernel을 load해오는 phase로 구성된다 볼 수 있다.

---

## Running User Application - Login Shell
<br>
　Bootstrapping 과정이 끝나면, OS kernel은 initialization step을 거친다.
  <!--여기 수정 필요-->

 * 이에 대해 다루는 ppt 슬라이드는 없지만, 이후 2번째 시간에 교수님이 설명하시기론 이 initialization step을 통해 OS exception handler와 exception table등을 만든다고 한다.data structure도 이 시기에 initialize된다.
 <!---->

 <br>
 　Initializiation이 끝나면, OS는 Login Shell이라는 이름의 user application을 찾게 된다. 마찬가지로 disk에서 physical memory로 정보를 load한 뒤, CPU가 해당 위치로 jump해서 이 login application이 running되기 시작한다.
 <br><br>
 　Login application이 시작되면 우리가 소위 말하는 "Shell" 화면이 보이게 된다.
<br></br>
 (이미지 삽입)
<br></br>

 사용자는 Shell이라는 interface를 통해 command를 OS kernel에 전달할 수 있게 된다.
 <br></br>
 　예시 그림처럼 user command를 작성하면, Login Shell이 해당 command를 OS에 request하게 되고, 이를 OS는 다음과 같은 처리 절차로 진행한다.
 1. app code와 data를 disk에서 찾는다.
 2. (physical) memory를 할당한다.
 3. (physical) memory에 load한다.
 4. CPU가 해당 app을 run할 수 있도록 환경을 configure한다.

---

## Executable Image
<br>
 　지금까지 OS kernel에서 user application의 실행까지의 과정을 보았으니, 이제 실제 code와 data가 어떻게 build 되는지 알아보자.
<br></br>
 (이미지 삽입)
<br></br>
 　실제 code는 대부분 high level language(ex. C언어)로 작성되어 있다. 이 code가 compiler와 assembler를 거치면 우린 executable image를 얻을 수 있게 된다.
 Executable image란 CPU가 code에서 얻어낸 instruction들로 이루어진, disk에 위치한 file을 뜻한다. (즉, 실행파일인 셈이다.) <!-- 확인 필요 -->
<br></br>
 　OS는 이 executable file을 이용해 excutable한 app을 찾고, memory를 할당한 뒤, disk로부터 load하여 CPU로 execute할 수 있게 된다.
<br></br>
　* 참고 : Executable image는 user code외에 uninitialized data를 가지고 있어 CPU가 이를 execute하기 위해선 heap과 stack의 추가적인 memory 할당이 요구된다.

---
## Execute Code by CPU : Control Flow
<br>
 　CPU가 code를 execute하는 과정은 우리가 컴퓨터sw개론 때부터 많이 봐왔던 program counter로 이루어진다. Program counter에 의해 가리켜진 instruction을 fetch하고, opcode에 따라 execute하며, program counter가 증감되거나 다른 위치로 이동해 다음 instruction을 fetch하는 과정이 반복된다. 이런 흐름 자체를 control flow라고 일컫는 것이다.
<br></br>
 (사진 첨부)
<br></br>
　위 그림에서 처럼 일반적으로 +4씩 이동하고 jmp 등을 만나면 완전히 다른 곳으로 이동한다.

---
## Altering CPU control flow
<br>
　일반적으로 control flow는 pc가 4씩 증감되는 흐름을 가지지만, 가끔 완전히 다른 곳으로 이동하게 되는 경우가 있다.
<br></br>
　이를 가능하게 하는 건 2가지 mechanism이 있다.

 1. jump and branches
 2. call and return using stack

<br>
　CPU의 control flow는 code 상에 explicit하게 표현되는 program state에 따라 변할 수 있고, 이는 모두 program normal execution의 case이다.  
<br></br>
　그렇다면, normal이 있으니 normal이 아닌 예외적인 상황도 존재한다는 것을 눈치챘을 것이다. 
<br></br>
　CPU는 data가 disk나 network로부터 도착한 경우, timer가 expire된 경우, control-C를 누른 경우, 0으로 나눈 경우 등 program state가 아닌 system state에 의해 cpu execution과 독립적으로 예외적인 상황에 처하곤 한다. 이런 exception들이 발생하면, CPU는 exception handler code로 control flow가 바뀌는 Exception control flow(ECF)라는 일련의 mechanism이 동작하게 된다.
<br></br>
　지금부터 Exception control flow를 알기 위해 무슨 event가 exception을 일으키고, 어떻게 이를 handle하며, exception handler로 부터 원래 위치로 돌아오는지에 대해 알아보자.

---
## Exceptions
<br>
　Exception은 2가지 종류로 나누어진다. CPU의 instruction execution의 마지막 단계에서 발생하는 Synchronous exception과 processor 밖의 event에 의해 발생하는 Asynchronous exception이 있다.

* Synchronous : trap이라고도 하며 intentional하게 발생하는 경우와 unexpected하게 발생하는 경우가 있다. (ex. System call / divide by zero)
* Asynchronous : interrupt라고도 하며 주로 H/W device에 의해 발생한다.

이런 Exception들은 고유한 exception number를 가지며, 이를 interrupt vector라고도 한다.

---
## Exception handler
<br>
　Exception에 무엇이 있는지 알아봤으니, 이를 handle하는 과정인 Exception handler에 대해 알아보자.
<br></br>

* Exception handler는 미리 coding이 되고 physical memory 상에 load 되어 있다 가정하고 예시를 살펴보자.

<br>
(사진첨부)
<br></br>
　Unexpected synchronous exception인 divid by zero가 code execution 도중에 발생했다 가정하자. CPU는 exception이 발생한 직후 자동으로 현재 system state를 저장한다. ( return address나 CPU register의 값 등등 ) 이후, 어떤 exception이 발생했는지 인식하기 위해 number를 확인한다. 이번 경우 exception의 number는 0으로, Exceptional Control Flow mechanism은 이를 index로 이용해 exception table(a.k.a interrupt vector table)상에서 맞는 handler를 찾게 된다. 이후, exception handler의 위치로 CPU는 jump하게 된다.
<br></br>
　CPU가 exception handler code를 모두 수행하고 나면, 상황에 따라 exception이 발생했던 원래 위치로 돌아오거나 그대로 종료된다. divide by zero와 같은 exception은 돌아가면 안되기 때문에 return 과정이 존재하지 않는다.
<br></br>

* Asynchronous exception의 경우 주로 return하고, synchronous의 경우 주로 return하지 않는다. ( 절대적이진 않음 )