# STM32-ADC-TRNG

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

A simple hardware-based true random number generator (TRNG) using the ADC of STM32 microcontrollers. It leverages analog noise read from floating ADC pins as an entropy source to generate unpredictable random numbers.

## 📝 Overview

Applications such as cryptography, simulations, and games often require unpredictable random numbers. Most software-based pseudo-random number generators (PRNGs) will always return the same sequence if initialized with the same seed.

This project uses the principle that when STM32 ADC pins are in a floating state (not connected externally), they pick up ambient thermal and electrical noise. This noise is truly random, so it can be converted into digital values to generate high-quality random seeds.

## ✨ Key Features

- **Hardware-based randomness:** Uses physical noise as an entropy source rather than a software algorithm (TRNG)  
- **Minimal dependencies:** No special libraries required beyond the STM32 HAL  
- **Lightweight and simple code:** Very short and intuitive, easy to integrate into any project  
- **Efficient conversion via DMA:** ADC readings are handled by DMA, minimizing CPU load  
- **Easy customization:** Bitwise operations allow generating random numbers in the desired format (8-bit, 16-bit, 32-bit)  

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
