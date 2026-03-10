---
title: In-place FFT
date: 2026-03-10 10:31:34 +9000
categories: [HPE, ALGORITHM]
tags: [fft, ft, dft, inpace]     # TAG names should always be lowercase
math: true
---

Bit-reversal
--

재귀 트리를 따라 내려가면서 배열이 어떻게 찢어지는지 보면, 특이한 패턴을 발견할수 있다.

N = 8인 배열이 재귀 함수를 타고 내려가는 과정을 보자

![in-place_eg.png](../assets/FFT/in-place_eg.png)

매 step마다 2로 나뉘어지고, 숫자표현이 이진수로 되어있으니 각 step 별로 짝홀 판단은 LSB로 시작해서 MSB에 결정됨을 쉽게 이해할수 있다. 

따라서 우선 배열들을 인접한 애들끼리 정렬한다음, 아래서 순서대로 연산을 진행한다 추가적인 메모리 할당 없이 원보 배열 하나에서 자리만 바꿔가며 연산하는 
반복문 기반의 In-Place FFT를 사용할수 있다. 
