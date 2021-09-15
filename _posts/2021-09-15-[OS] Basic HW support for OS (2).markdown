---
layout: post
title:  "[OS] Basic H/W supports for OS (2)"
date:   2021-09-15 21:30:04 +0530
categories: 
---

> 이 글은 포스텍 박찬익 교수님의 CSED312 운영체제 수업의 강의 내용과 자료를 기반으로 하고 있습니다.

---  

## Dual Mode of CPU Execution
<br>
　모든 CPU는 최소 2가지 mode의 CPU execution을 제공한다. 이는, CPU가 execution 도중에 exception이 발생하면서 생길 문제를 미연에 방지하기 위함이다.
<br></br>
　예를 들어, 다음과 같은 상황을 고려하는 것이다.

1. CPU가 exception에 의해 exception handler로 jump하면서 user가 이를 modify할 수 있게 되는 것
2. Exception handler의 entry가 아닌 middle로 jump할 수 있게 되는 것.

결국, user가 exception handler에 접근할 수 있게 되는 unreliable한 상황들을 피하기 위해, privilege(권한)를 다르게 가지는 두 mode를 설정한다.

 * Kernel mode : Full privilege를 가지는 경우로, 모든 instruction들을 execute할 수 있다.
 * User mode : Limited privilege를 가지는 경우로, 특정 instruction을 execute할 수 없다.

위 두 CPU mode configuration은 x86 기준 EFLAGS라는 register에서 정해진다. CPU는 kernel mode일 때 해당 register를 조작해 mode를 전환할 수 있다.

---
## Privileged Operations in Kernel mode
<br>
　Kernel mode에서 CPU는 privileged operation들을 수행할 수 있다.
<br></br>  

  * Privileged instruction : Kernel mode에서만 할 수 있으며, memory read/write, I/O device access, network packet send/read 등이 있다.
  * Memory access control : Kernel mode에서만 memory를 할당하는 등의 조작을 가할 수 있다. 이를 통해 user code가 kernel을 overwritting하는 상황을 방지한다.

위 kernel mode의 특성인 memory access control에 의해 physical memory는 partition이 이루어진다.

<br>
(사진 첨부)
<br></br>
　위 그림에서 처럼 CPU는 higher privilege가 요구되는 exception handler등을 kernel memory에 install하고, 남는 영역인 user memory에 user code, data등을 load시킨다. User mode에선 kernel memory에 접근할 수가 없으며, Exception Control Flow에 의해서만 가능하다.

---
## CPU Mode Switch
<br>

(사진첨부)
<br></br>
　CPU는 power가 들어온 initial 상황에 kernel mode에서 시작한다. 모든 bootstrapping operation procedure과 operating system의 initialization( exception handler, exception table, etc)이 이루어진다. Initialization이 끝나면 Operating system은 cpu mode를 EFLAGS register를 통해 user mode로 설정하고 login application을 execute하며 이후 계속 user application 실행이 이어진다.
<br>
　User application을 실행하는 도중에 exception이 발생하면, ECF에 의해 automatically하게(hw feature가 변형됨) user mode에서 kernel mode로 바뀐다. Exception handler의 위치를 찾아 그 곳으로 jmp하며 exception handler의 마지막 단계에서 original location으로 돌아가기 위해 cpu는 kernel mode에서 user mode로 EFLAGS register를 설정해 돌아가게 된다.
<br>
　만약 user mode 상에서 exception이 발생하지 않으면 컴퓨터는 종료되기 전까지 kernel mode로 진입하지 않는다.

---
## Stack for Exception Handlers
<br>
　Stack은 cpu mode에 관계없이 user application call이나 exception handler call 모두 data의 저장과 프로그램의 실행을 위해서 필요하다. 그러나, 각 mode에서 application이 사용할 stack은 분리되어 각각 존재한다.
<br></br>
　Exception handler가 exception이 일어난 user program의 stack을 사용하지 못하는 이유는 exception이 발생했을 때 그 자체의 문제에 의해 의도치 않게 발생한 exception인지, 혹은 timer나 I/O device등을 사용하기 위해 발생한 exception인지에 따라 original location으로 돌아가는 게 달라지기 때문이다. 항상 돌아감을 보장할 수 없기 때문에 stack이 각각 보존되어야 한다.
<br></br>
(사진 첨부)
<br></br>
　User application은 code, data, stack을 모두 가지고 있다. Exception handler의 code, data는 kernel memory에 initialization step동안 load되고, code와 data는 다른 application과도 공유된다. 그러나 kernel stack은 공유되지 않고 각각 할당되기 때문에 각 application은 고유한 kernel stack을 kernel memory에 가지고 있다. ( 이 이유는 나중에 다룬다고 한다. )

---
## Kernel Stack for running OS code
<br>
　Kernel Stack은 Operating system code(ex. exception handler)를 running할 때만 사용된다. 
<br></br>
(사진첨부)
<br></br>
　위 예시를 보면, exception(system call)이 발생해 CPU state를 저장하고 syscall handler 호출, I/O driver를 호출하는 것등을 볼 수 있다.

