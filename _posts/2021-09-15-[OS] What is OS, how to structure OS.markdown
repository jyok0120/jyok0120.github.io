---
layout: post
title:  "[OS] What is OS, how to structure OS"
date:   2021-09-15 21:30:04 +0530
categories: 
---

> 이 글은 포스텍 박찬익 교수님의 CSED312 운영체제 수업의 강의 내용과 자료를 기반으로 하고 있습니다.

---  

## What is Operating System?
<br>
　지금까지 Basic Hw에 대해 알아봤으니, Operating System이 무엇인지 기본적인 개념을 알아보자.
<br></br>
(사진첨부)
<br>
<br>
 　Operating System은 User application과 hardware 사이에 존재하는 것으로, 위 그림에서 보이듯이 상황에 따라 interface를 이용해 상호작용하는 것이다.(???)
 <br>
 　application의 실행 중 exception이 발생하면 Exception handler의 OS code를 수행하고, I/O device의 drive( print/read from file )를 위해 interface를 사용한다.
 <br></br>
 　OS는 application 관점에서 보면 service provider이다. 다른 application의 execution을 support하는 등의 service를 OS에서 제공한다.
 <br>
 　Hardware의 관점에서 보면 OS는 resource manager이다. OS의 underline hardware( cpu code, memory, drive )를 조작해 user application이 가능한 효율적으로 사용될 수 있도록 한다.
 <br>
 <br>

 * Challenge : Efficiency vs User-friendliness  
    OS는 user program을 가능한 효율적으로 execute되게 할 수 있게 하면서 동시에 computer system을 쉽게 사용할 수 있어야 하는 challenge를 가지고 있다.

---

 ## How to structure OS?
 