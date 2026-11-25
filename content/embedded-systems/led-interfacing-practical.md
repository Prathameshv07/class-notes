---
id: led-interfacing-practical
title: Interfacing LEDs with the 8051
subject: embedded-systems
unit: 2
type: practical
practical: 7
syllabus: [UITIV-006, UDSIV-006]
status: draft
sources:
  - "Muhammad Ali Mazidi and Janice Mazidi, The 8051 Microcontroller and Embedded Systems, Pearson, Ch. 4 and Ch. 7 (verify)"
  - "Microchip (Atmel), AT89C51 datasheet, DC characteristics (port current limits)"
---

## Aim

To interface LEDs with the 8051 microcontroller: calculate the series resistor, wire the LEDs active-low, keep one LED on as a power indicator and blink another LED once per second using single-bit control.

## Concepts used

- [[8051-gpio-ports-practical]] (port pins as outputs, `sbit`)
- [[sensors-and-actuators]] (an LED is an actuator)

## Setup

### Tools

- **Keil µVision 5 (C51)** for the program and HEX file.
- **Proteus** for the circuit: AT89C51, LED-RED, LED-GREEN, RES.

### Calculating the series resistor

A red LED drops about **2 V** when lit; we want about **10 mA**:

$$R = \frac{V_{CC} - V_{LED}}{I_{LED}} = \frac{5 - 2}{0.010} = 300\ \Omega$$

The nearest standard value above it is **330 Ω**, giving (5 − 2) / 330 ≈ **9 mA**.

### Circuit

| Part | Connection |
|---|---|
| Red LED + 330 Ω | +5 V → resistor → LED → **P1.0** (blinking LED) |
| Green LED + 330 Ω | +5 V → resistor → LED → **P1.7** (power indicator) |
| AT89C51 | EA (pin 31) to +5 V |

With both LEDs lit, Port 1 sinks about 18 mA, slightly over the AT89C51's 15 mA per-port limit. Proteus does not mind, but on a real board use 470 Ω (about 6 mA each, 13 mA together) to stay inside it.

## Code

```c
#include <reg51.h>

sbit LED = P1^0;                  // blinking red LED
sbit PWR = P1^7;                  // green "power on" LED

void delay_ms(unsigned int ms)    // about 1 ms per count in Keil (measured in practical 3)
{
    unsigned int i, j;
    for (i = 0; i < ms; i++)
        for (j = 0; j < 1275; j++);
}

void main(void)
{
    PWR = 0;                      // green LED on and stays on (active-low)

    while (1)
    {
        LED = 0;                  // red LED on
        delay_ms(500);
        LED = 1;                  // red LED off
        delay_ms(500);
    }
}
```

## Output

```output
P1 = 0x7E  ->  green LED (P1.7) ON, red LED (P1.0) ON
P1 = 0x7F  ->  green LED (P1.7) ON, red LED (P1.0) OFF
... repeats: red LED blinks (on half the time), green LED stays on
```

<!-- For Prathamesh: the P1 sequence 7E/7F was verified with SDCC + s51. The blink period depends on the
     delay loop, which is about 1 s in Keil (500 ms on + 500 ms off) but much longer with SDCC;
     practical 10 uses this difference as its lesson. Add a Proteus screenshot, with alt text. -->

## Explanation

### What are the two ways to connect an LED?

| | Active-low (current sink) | Active-high (current source) |
|---|---|---|
| ==Wiring== | +5 V → resistor → LED → pin | Pin → resistor → LED → ground |
| ==LED on when pin is== | 0 | 1 |
| ==Works well on 8051?== | Yes: pins sink up to 10 mA | No: pins source only about 60 µA (weak pull-up) |

==On the 8051, LEDs are wired active-low because a port pin can sink far more current than it can source.==

### Why is a series resistor essential?

- ==**An LED has almost no resistance once it turns on**==; without a resistor the current would rise until the LED or the port pin is damaged.
- ==**The resistor sets the current**== using Ohm's law: R = (V<sub>CC</sub> − V<sub>LED</sub>) / I<sub>LED</sub>.
- ==**Different colours drop different voltages**==: about 2 V for red, 3 V for blue and white, so the resistor changes too.
- ==**Check the port limits**==: 10 mA per pin and 15 mA per port on the AT89C51, 71 mA for all pins together.

### How does the program control single LEDs?

- `sbit LED = P1^0;` names bit 0 of Port 1, so `LED = 0;` changes **only** P1.0.
- `PWR = 0;` turns on the green LED once; nothing changes it again, so it stays on.
- Inside the loop, only P1.0 toggles, every 500 ms (about 1 blink per second).
- Using `sbit` avoids disturbing the other pins of the port, which a byte write like `P1 = 0x00` would change.

## Try it

Add a third LED on P1.3 that blinks **twice as fast** as the red LED. Where in the loop do you toggle it?

> [!answer]- Answer
> Add `sbit FAST = P1^3;` and split each 500 ms delay into two 250 ms delays, toggling `FAST = ~FAST;` after each one. FAST then changes every 250 ms (2 blinks per second) while LED still changes every 500 ms.

## Viva questions

1. **Why is a resistor connected in series with an LED?** To limit the current and protect both the LED and the port pin.
2. **Calculate the resistor for a red LED (2 V) at 10 mA from 5 V.** (5 − 2) / 0.01 = 300 Ω; use 330 Ω.
3. **What is active-low connection?** The LED is between +5 V and the pin; it lights when the pin is 0.
4. **Why are LEDs usually active-low on the 8051?** Port pins sink current well but source very little.
5. **What is the maximum current per pin and per port of the AT89C51?** 10 mA per pin; 15 mA per port (P1–P3), 26 mA for P0.
6. **What does `sbit LED = P1^0;` do?** Names bit 0 of Port 1 as LED, so it can be set or cleared alone.
7. **Why does a blue LED need a different resistor from a red one?** It has a higher forward voltage (about 3 V), so less voltage is left across the resistor.
8. **What happens if an LED is connected without a resistor?** Excess current flows; the LED or the port pin can be damaged.

## Sources

- Muhammad Ali Mazidi and Janice Mazidi, *The 8051 Microcontroller and Embedded Systems*, Pearson, Ch. 4 (I/O ports) and Ch. 7 (bit-wise I/O in C) (verify).
- Microchip (Atmel), *AT89C51* datasheet, DC characteristics: IOL limits per pin and per port, IOH of ports 1–3.
