---
title: Concurrency 4 (Optimistic Concurrency Control)
date: 2025-06-03 17:32:00 +9000
categories: [Konkuk_3-1, Database]
tags: [concurrency, optimistic, occ, validation, locking, tree]     # TAG names should always be lowercase
---

Validation-Based Protocol
==

Optimistic Concurrency Control (OCC)
--

희망찬 protocol, 일단 다 되겠지 생각하고 진행시켜, 우리는 **validate이랑 write연산이 Atomic하게 일어난가 가정할게~**

- Idea: Can we use **commit time as serialization order**?
- 다음과 같이 진행하자
  - Postpone writes to **end of transaction**
  - Keep track of data items read/written by transaction
  - Validation performed at commit time, detect any out of serialization order reads/writes
- Read에선 작동확률 good, Write에서는 validation fail 확률 up

- Execution of transactoin T<sub>i</sub> is done in three phases.
  - **Read and execution phase**: Transaction T<sub>i</sub>가 연산들을 자신의 **Local Memory (Not visible to other)** 
  - **Validation phase**: Transaction T<sub>i</sub> performs a **validation test** to 확인해, local memory에서 한 연산을 database에 저장해도 될까?, Serializability가 보장이 되나?
  - **Write phase**: Database에 적용하기 아니면 roll back, 이제서야 남들에게도 visible

- Three timestamps
  - **StartTS**: T<sub>i</sub>가 **execution을 시작한** 시간
  - **ValidationTS**: T<sub>i</sub>가 **validation phase로 진입한** 시간
  - **FinishTS**: T<sub>i</sub>가 **write phase를 끝낸** 시간

> 유효성 검증(validation) 시점을 트랜잭션의 **논리적 순서(TS)**로 삼는다. -> **Ensures Serializability** 

### Validation Test

![occ-case1.jpeg](../assets/Konkuk_3-1/Database/Post_18/occ-case1.jpeg)
이 외의 경우는 validation fail and T<sub>j</sub> is aborted

e.g.1 
![occ-eg.jpeg](../assets/Konkuk_3-1/Database/Post_18/occ-eg1.jpeg)

e.g.2
![occ-eg2.jpeg](../assets/Konkuk_3-1/Database/Post_18/occ-eg2.jpeg)
