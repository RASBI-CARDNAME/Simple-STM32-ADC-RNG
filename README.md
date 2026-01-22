# 🎲 STM32 ADC 노이즈 기반 하드웨어 난수 생성기 (TRNG)

## 1. 프로젝트 개요 (Overview)
소프트웨어 기반의 의사 난수 생성기(PRNG)는 초기 시드(Seed) 값이 같으면 항상 동일한 패턴을 반복하는 한계가 있습니다.
본 프로젝트는 **STM32의 ADC 핀을 Floating 상태로 두었을 때 발생하는 열 잡음(Thermal Noise)과 전자기적 노이즈**를 엔트로피 원천으로 활용하여, 예측 불가능한 진성 난수(True Random Number)를 생성하는 펌웨어입니다.

*   **핵심 원리:** 외부 연결이 없는 ADC 핀에서 불규칙한 미세 전압 변화를 측정하여 난수화
*   **주요 특징:** DMA를 활용한 CPU 부하 최소화 및 XOR/Shuffling을 통한 난수 균일도 향상

---

## 2. 기술 스택 (Tech Stack)
*   **HW:** STM32F103C8T6 (Blue Pill)
*   **SW:** C, STM32 HAL Library
*   **Tools:** J-Link (SWD Debugging), Logic Analyzer (Timing Verification)

---

## 3. 동작 원리 (Algorithm)

### 1) 엔트로피 수집 (Entropy Collection)
*   **Floating Pin:** 외부 회로와 연결되지 않은(Open) ADC 채널 2개(예: IN0, IN9)를 설정합니다.
*   **Dual Channel Sampling:** 두 채널의 값을 읽어 주변 환경의 무작위 노이즈를 수집합니다.

### 2) 데이터 가공 (Post-Processing)
단순한 ADC 값은 편향(Bias)이 있을 수 있으므로, 두 채널의 값을 혼합하여 엔트로피를 높입니다.
```c
// 예시 알고리즘 (XOR Mixing)
uint16_t raw1 = adc_buffer[0];
uint16_t raw2 = adc_buffer[1];
uint16_t random_val = (raw1 ^ raw2) ^ (raw1 << 1) ^ (raw2 >> 1);

// 개선된 알고리즘
for (int i = 0; i < 1000 i++) {
    for (int k = 0; k < 16 k++) {
        HAL ADC Start(&hadc1);
        HAL_ADC_PollForConversion (&hadc1, 10);
        temp = ( (HAL_ADC_GetValue(&hadc1) & 0x01) <<k); // 1비트씩 쏙쏙 뽑아서 채움
    }
result[i] = temp;
temp = 0;
}

```

---

## 4. 트러블 슈팅 (Troubleshooting) 🛠️
### DMA 전송 완료 전 데이터 읽기 문제 (Race Condition)  
메인 루프에서 ADC 값을 읽으려 할 때, DMA 전송이 아직 완료되지 않아 이전 쓰레기 값을 읽거나 데이터가 꼬이는 동기화 문제(Race Condition) 발생 가능성을 파악함.  

### 해결 방법:  
*   HAL_ADC_Start_DMA 호출 후 바로 데이터를 읽지 않음.  
*   HAL_ADC_ConvCpltCallback (전송 완료 인터럽트 콜백) 함수 내에서 conversion_complete 플래그를 SET 하도록 구현.  
*   메인 루프에서는 이 플래그가 1이 될 때까지 대기(Blocking)하거나 상태를 체크하여, 데이터 무결성이 보장된 시점에만 난수 연산을 수행하도록 안전장치 마련.  

---

## 5. 활용 예시 (Use Cases)
* 보안: AES 암호화 통신을 위한 초기 벡터(IV) 및 세션 키 생성 시드
* 게임/시뮬레이션: 예측 불가능한 몬스터 패턴이나 아이템 드롭 확률 구현
* 통신: 네트워크 백오프(Back-off) 알고리즘의 랜덤 대기 시간 생성

---

## 📄 라이선스
MIT 라이선스 – 자세한 내용은 [`LICENSE`](LICENSE)를 참조하십시오.


