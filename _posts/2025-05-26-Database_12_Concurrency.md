---
title: Concurrency
date: 2025-05-26 20:52:43 +9000
categories: [Konkuk_3-1, Database]
tags: [concurrency, serializability, phantom, transaction, locking]     # TAG names should always be lowercase
---

Serializability
-
여러 트랜잭션이 동시에 실행될 때에도 “어떤 순서로 하나씩 순차적으로 실행”했을 때와 같은 결과를 보장

Serializable하다는 것은, 지금 이 스케줄(동시 실행 순서)이 “어떤” 직렬 실행 순서와 
결과가 완전히 동일함을 의미합니다.

이렇게 되면 “트랜잭션들이 실제로는 섞여 실행됐지만, 데이터베이스 일관성은 마치 
순차 실행했을 때와 똑같이 보장된다”는 장점이 있습니다.

즉, 실제로는 A와 B, C가 뒤섞여 실행되었더라도, “A→B→C” 또는 “B→C→A” 같은 어느 한 순차 실행(serial schedule)과 결과가 똑같다면, 그 스케줄은 직렬화 가능(serializable)하다고 부릅니다
