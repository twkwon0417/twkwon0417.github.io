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

```cpp
void bitReverse(vector<complex<double> >& x) {
    size_t N = x.size();

    // N이 bit 수 계산
    size_t log_N = 0;
    while ((1 << log_N) < N) {
        log_N++;
    }

    // 모든 인덱스 i에 대해 뒤집힌 인덱스(rev)를 구하고 스왑
    for (size_t i = 0; i < N; ++i) {
        size_t rev = 0;

        // i의 비트를 하나씩 읽어서 rev의 반대편에 심어줘
        for (size_t j = 0; j < log_N; ++j) {
            // i의 j번째 비트가 1인지 확인
            if ((i >> j) & 1) {
                // 1이라면, rev의 대칭되는 위치(log_N - 1 - j)를 1로 세팅
                rev |= (1 << (log_N - 1 - j));
            }
        }

        // 반 바퀴만 돌면돼, 한 바퀴 다 돌면 다시 원상복귀 시킬꺼야
        if (i < rev) {
            swap(x[i], x[rev]);
        }
    }
}

vector<complex<double> > computeDFT(const vector<complex<double> >& x) {
    size_t N = x.size();

    // 1. 비트 반전으로 데이터 미리 섞기 (바닥까지 쪼개진 상태)
    bitReverse(x);

    // layer의 수 만큼 반복
    for (size_t len = 2; len <= N; len <<= 1) {

        // 현재 블록 크기(len)에 맞는 한 칸짜리 회전 각도 계산
        double angle = -2.0 * PI / len;
        complex<double> wlen(cos(angle), sin(angle));

        // 전체 배열을 블록 단위(len)로 건너뛰며 순회
        for (size_t i = 0; i < N; i += len) {
            complex<double> w(1.0, 0.0);

            // 하나의 블록 안에서의 나비 연산 수행
            for (size_t j = 0; j < len / 2; ++j) {
                // x[]에서 이전 layer의 값을 재활용해
                complex<double> u = x[i + j];
                complex<double> v = x[i + j + len / 2] * w;

                x[i + j] = u + v;
                x[i + j + len / 2] = u - v;

                w *= wlen;
            }
        }
    }
}
```
