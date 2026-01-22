# STM32-ADC-TRNG
STM32F1의 ADC를 활용한 간단한 하드웨어 기반 진정한 난수 생성기(TRNG)로, ADC 핀에서 읽은 아날로그 노이즈를 엔트로피 원천으로 활용하여 예측 불가능한 난수를 생성합니다.

## 📝 개요
암호화, 시뮬레이션, 게임과 같은 애플리케이션은 종종 예측 불가능한 난수를 필요로 합니다. 대부분의 소프트웨어 기반 의사 난수 생성기(PRNG)는 동일한 시드로 초기화될 경우 항상 동일한 시퀀스를 반환합니다.

이 프로젝트는 STM32 ADC 핀이 플로팅 상태(외부 연결 없음)일 때 주변 열 및 전기 노이즈를 포착한다는 원리를 활용합니다. 이 노이즈는 진정한 난수이므로 디지털 값으로 변환하여 고품질 난수 시드를 생성할 수 있습니다.

# 🔧기술 스택 (Tech Stack)
- HW: STM32F103C8T6
- SW: C
- Tools: Logic Analyzer, J-link OB

## ⚙️ How It Works

1. **ADC channel setup:** Configure two or more GPIO pins as analog inputs without any external connection.  
2. **Start DMA transfer:** Call `HAL_ADC_Start_DMA()` to start ADC conversion on both channels and transfer the results to memory (the `seed` array).  
3. **Wait for conversion completion:** With `Continuous Conversion Mode` disabled, the ADC automatically stops after converting the two channels. When DMA transfer is complete, the `HAL_ADC_ConvCpltCallback` callback is triggered.  
4. **Completion signal:** The callback sets a flag indicating the conversion is complete in the main loop. (In this example, the most significant bit of `seed[0]` is used as the flag.)  
5. **Random number generation:** The main loop checks the flag to know new ADC values are available. Values from the two channels are combined using XOR, bit shifts, and other operations to produce the final random number. This process reduces statistical bias and improves randomness.

## 🚀 How to Use

### 1. CubeMX Setup

- **ADC1 configuration:**  
    - `Scan Conversion Mode`: `Enabled`  
    - `Continuous Conversion Mode`: `Disabled` (very important!)  
    - `Number Of Conversion`: `2`  
    - Configure two channels (e.g., `Channel 0`, `Channel 9`) with pins left unconnected.  
- **DMA configuration:**  
    - Add a DMA channel for ADC1 and set `Mode` to `Normal`.  
- **NVIC configuration:**  
    - Enable DMA interrupts.  

### 2. Apply Code

Add the files to the correct folders in your project:  
- ioc files → `root folder`  
- Source files → `Src`  

## 💡 Example Use Cases

- Seed values for pseudo-random number generators (PRNGs)  
- Initialization Vectors (IVs) for symmetric encryption algorithms like AES or ChaCha20  
- Item drop rates or random enemy behavior in games  
- Random back-off times in communication protocols  

## ⚠️ Caution

This code is a simple example intended to provide sufficient randomness for general applications. For systems requiring high-security levels, such as banking systems or national secrets, it is recommended to use the STM32 hardware RNG peripheral (if available) or certified cryptographic libraries (e.g., FIPS-140).

## 📄 License

This project is licensed under the [MIT License](LICENSE).
