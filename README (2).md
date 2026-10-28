# PWM and Logic Analyzer

Generating PWM waveforms with a hardware timer on the **STM32 B-L475E-IOT01A** and observing them with a **Zeroplus LAP-C 16128** logic analyzer.

## Assignment

> Please read some information about PWM techniques and applications in Internet resources (such as <https://en.wikipedia.org/wiki/Pulse-width_modulation>, <https://electricalfundablog.com/pulse-width-modulation/>, ...) and find an application that you are interested in. And generate the PWM waveforms for the applications, which can be observed using a logic analyzer. Show some waveform observations in your Demo to TA.

## Chosen Application

<!-- TODO: describe the application you picked, e.g. LED brightness control, servo motor control, DC motor speed control -->

_TBD_

## Hardware

| Item | Detail |
|---|---|
| Board | STM32 B-L475E-IOT01A |
| MCU | STM32L475VGT6 |
| Logic analyzer | Zeroplus LAP-C 16128 |
| IDE | STM32CubeIDE / STM32CubeMX |

## Pin Assignment

| Signal | MCU pin | Board header | Logic analyzer |
|---|---|---|---|
| PWM output (TIM2_CH1, AF1) | PA5 | Arduino **D13** (also drives LED1) | Channel A5 |
| Ground | GND | GND | GND |

> **Note:** `PA5` is the STM32 GPIO name, `D13` is the Arduino header label on the board, and `A5` is the logic analyzer channel name. The board's Arduino header also has pins labeled `A0`–`A5` (analog inputs, e.g. board `A5` = `PC0`) — these are **not** the PWM pin.

## Clock Configuration

| Clock | Frequency |
|---|---|
| MSI (internal RC) | 4 MHz |
| PLL (M = /1, N = ×40, R = /2) | 80 MHz |
| SYSCLK / HCLK | 80 MHz |
| PCLK1 (APB1 prescaler /1) | 80 MHz |
| APB1 timer clock (TIM2) | 80 MHz |

$$
f_{\mathrm{SYSCLK}} = \frac{4\ \text{MHz}}{1} \times \frac{40}{2} = 80\ \text{MHz}
$$

TIM2 is on APB1. Because the APB1 prescaler is /1, the TIM2 kernel clock equals PCLK1 = 80 MHz.

## Timer Settings

$$
f_{\mathrm{CNT}} = \frac{f_{\mathrm{TIM}}}{PSC + 1}, \qquad
f_{\mathrm{PWM}} = \frac{f_{\mathrm{TIM}}}{(PSC + 1)(ARR + 1)}, \qquad
\text{Duty} = \frac{CCR}{ARR + 1}
$$

| Purpose | PSC | ARR | Pulse (CCR) | PWM frequency | Duty |
|---|---|---|---|---|---|
| Visual check (LED1 blinks) | 7999 | 9999 | 5000 | 1 Hz | 50 % |
| Logic analyzer measurement | 79 | 999 | 500 | 1 kHz | 50 % |

CubeMX setup:

- **TIM2** → Clock Source: Internal Clock
- **Channel1** → PWM Generation CH1, mapped to **PA5**
- Mode: PWM mode 1, polarity High

## Code

CubeMX only initializes the timer; the PWM output must be started explicitly:

```c
/* USER CODE BEGIN 2 */
HAL_TIM_PWM_Start(&htim2, TIM_CHANNEL_1);
/* USER CODE END 2 */
```

Change the duty cycle at runtime:

```c
__HAL_TIM_SET_COMPARE(&htim2, TIM_CHANNEL_1, pulse);  // 0 .. ARR+1
```

## Logic Analyzer Settings

| Setting | Value |
|---|---|
| Sample rate | 1 MHz |
| Memory depth | 128K samples (≈ 131 ms per capture) |
| Threshold | 1.5 V (3.3 V logic) |
| Trigger | None, or rising edge on the PWM channel |

At 1 MHz sampling, one 1 kHz PWM period spans 1000 samples, and a single capture contains about 131 periods.

## Observations

<!-- TODO: add logic analyzer screenshots and measured values -->

| Setting | Expected frequency | Measured frequency | Expected duty | Measured duty |
|---|---|---|---|---|
| PSC 79 / ARR 999 / CCR 500 | 1 kHz | | 50 % | |

## Troubleshooting Notes

| Symptom | Cause / fix |
|---|---|
| No output at all | `HAL_TIM_PWM_Start()` not called, or `Pulse = 0` |
| LED1 blinks but analyzer shows nothing | Probing the wrong header pin — PWM is on **D13**, not the board's `A5` |
| Analyzer stuck at "waiting" | Trigger condition never met; disable the trigger to test |
| Flat line even with a valid signal | Capture window too short for the PWM period, or view zoomed in too far |
| Every channel flat, even on 3V3 | No electrical contact: check lead connectors (female-to-female does not connect), test wires for continuity, confirm common GND |

## References

- [Pulse-width modulation — Wikipedia](https://en.wikipedia.org/wiki/Pulse-width_modulation)
- [Pulse Width Modulation — Electrical Fundamentals Blog](https://electricalfundablog.com/pulse-width-modulation/)
- STM32L475xx datasheet and RM0351 reference manual
- UM2153: B-L475E-IOT01A Discovery kit user manual
