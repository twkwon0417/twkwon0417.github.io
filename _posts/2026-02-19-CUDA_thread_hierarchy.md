---
title: CUDA.1 Introduction
date: 2025-12-21 21:34:52 +9000
categories: [HPE, CUDA]
tags: [cuda, block, grid, sm]     # TAG names should always be lowercase
---

GPU에서 thread hierachy가 왜 이리 헷갈리는지 정리하고 넘어가자

Overview
==

Kernel 실행시, 새로운 Grid가 하나 생성되는 거야.  

- **Grid**: 커널이 실행되는 공간 (kernel 실행시 Gird가 생성)
- - **Block**: 여러개의 워프들의 그룹, 같은 block 내의 thread들은 Shared Memory 같이 쓸수 있다. Block은 쪼개지지 않고 SM 하나에 통째로 배정돼서 실행
- - - **Warp**: Thread 32개씩 묶어서 하드웨어적으로 동시실행
- - - - **Thread**: 개별 연산 수행, 블록 내 shared memory로 소통

#### Thread

- thread 별로 독립적인 register을 가짐, `threadIdx.x`와 같은 변수로 자기 자신의 고유 번호 식별

#### Warp

- Hardware가 실제로 스레드들을 묶어서 실행하는 단위 (32개)
- 32개의 thread가 동시에, 똑같은 instruction을 실행

#### Block

- 같은 block에 속한 thread들은 Shared Memory를 공유해
- `__syncthreads()`와 같이 block내 thread들 끼리 동기화 할수 있어
- 쪼개지지 않고 하나의 SM에 통째로 배정돼 실행 돼

#### Grid

- 하나의 kernel을 호출할때 생겅되는 전체 작업 공간
- Block들 끼리는 완전히 독립적이라서 서로 소통이나 동기화 불가
- Block들이 1번 부터 순서대로 실행될지, 무작위로 실행될지는 전적으로 GPU scheduler가 결정

Block이라는 단위가 필요한 이유
--

1. SM과의 1:1 매칭을 위해서
    - Block 없이 warp 단위로 matching 시키면, 중앙 스케쥴러, 수많은 warp들을 추적, 관리 해야한는데, 비용이 엄청 클꺼야
2. Shared Memory를 쓰기 위해서
    - Warp 단위로 하면 shared에 참여하는 범위가 작고, grid 단위는 너무 커서 global memory를 사용해야해
3. 확실한 동기화를 위해서
    - Grid 전체로 하기에는 deadlock의 위험도 있고 너무 느려
4. Transparent Scalability
    - Block들이 완전히 독립적이여서, 하드웨어 성능(SM의 갯수)에 맞춰서 유연하게 분배될 수 있는 확장성을 가지게 됨
    - Block은 SM의 메모리를 할당 받기 위한 큼직한 덩어리이고, Warp은 실제로 명령어를 싱행하기 위해 하드웨어가 잘게 쪼갠 단위. 

뭔가 이상한 conclusion
--

``` cuda
__global__ void addition(int* A, int* B, int* C, int totalElements) {
  int workIndex = threadIdx.x + blockDim.x * blockIdx.x;

  if(workIndex < totalElements) {
    C[workIndex] = A[workIndex] + B[workIndex];
  }
}
```

Grid는 CPU에서 kernel이 실행될때 생기기 때문에, workIndex는 무조건 0부터 시작함이 보장 된다.  
