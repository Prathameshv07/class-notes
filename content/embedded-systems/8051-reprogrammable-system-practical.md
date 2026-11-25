---
id: 8051-reprogrammable-system-practical
title: A Reprogrammable Embedded System with the 8051
subject: embedded-systems
unit: 1
type: practical
practical: 2
syllabus: [UITIV-006, UDSIV-006]
status: draft
sources:
  - "Muhammad Ali Mazidi and Janice Mazidi, The 8051 Microcontroller and Embedded Systems, Pearson, Ch. 7 (verify)"
  - "Microchip (Atmel), AT89C51 datasheet (verify)"
  - "Labcenter Electronics, Proteus help: loading a program file into a microcontroller (verify)"
---

## Aim

To design and develop a reprogrammable embedded system using the 8051 microcontroller, by loading two different programs into the same LED circuit and observing that the behaviour changes without changing the hardware.

## Concepts used

- [[8051-programming-environment-practical]] (creating a project and HEX file in Keil)
- [[embedded-vs-general-purpose-systems]] (firmware)

## Setup

### Tools

- **Keil µVision 5 (C51)** to write the two programs and make their HEX files.
- **Proteus** (ISIS schematic capture) to simulate the circuit and load each HEX file into the 8051.
- If your lab has a real board (for example with an AT89S52), the same HEX files can be burnt into it with the lab's programmer; follow the board's manual.

### Circuit in Proteus

| Part (Proteus name) | Connection |
|---|---|
| AT89C51 | The microcontroller |
| 8 × LED-RED | Anode through resistor to +5 V, cathode to P1.0 … P1.7 |
| 8 × RES, 330 Ω | One in series with each LED |
| Pin 31 (EA) | To +5 V, so the 8051 runs from its internal flash |

Proteus supplies the clock and reset for the AT89C51 in simulation. On a real board you also need an 11.0592 MHz crystal with two 33 pF capacitors, and a reset circuit on pin 9.

### Steps

1. In Keil, make project `blink` with Program 1, tick **Create HEX File**, build → `blink.hex`.
2. Make a second project `runlight` with Program 2, build → `runlight.hex`.
3. In Proteus, draw the circuit. Double-click the AT89C51: set **Clock Frequency** to `11.0592MHz` and **Program File** to `blink.hex`. Run.
4. Stop. Change **Program File** to `runlight.hex`. Run again. Same circuit, new behaviour.

## Code

**Program 1: all LEDs blink together (`blink`)**

```c
#include <reg51.h>

void delay_ms(unsigned int ms)        // software delay: about 1 ms per count
{                                     // at 11.0592 MHz in Keil C51 (approximate)
    unsigned int i, j;
    for (i = 0; i < ms; i++)
        for (j = 0; j < 1275; j++);
}

void main(void)
{
    while (1)
    {
        P1 = 0x00;        // all pins low  -> all 8 LEDs ON  (active-low)
        delay_ms(500);
        P1 = 0xFF;        // all pins high -> all 8 LEDs OFF
        delay_ms(500);
    }
}
```

**Program 2: running light, one LED moves from P1.0 to P1.7 (`runlight`)**

```c
#include <reg51.h>

void delay_ms(unsigned int ms)        // same delay function as Program 1
{
    unsigned int i, j;
    for (i = 0; i < ms; i++)
        for (j = 0; j < 1275; j++);
}

void main(void)
{
    unsigned char led = 0x01;         // bit = 1 marks the LED we want ON

    while (1)
    {
        P1 = ~led;                    // invert: LEDs are active-low
        delay_ms(300);
        led = led << 1;               // move to the next LED
        if (led == 0)                 // after P1.7, the 1 falls off the left end
            led = 0x01;               // start again from P1.0
    }
}
```

## Output

```output
Program 1 (blink.hex):     P1 = 00 (all LEDs ON) -> FF (all OFF) -> 00 -> FF ...

Program 2 (runlight.hex):  P1 = FE -> FD -> FB -> F7 -> EF -> DF -> BF -> 7F -> FE ...
                           LED ON:  P1.0  P1.1  P1.2  P1.3  P1.4  P1.5  P1.6  P1.7  P1.0 ...
```

<!-- For Prathamesh: both P1 sequences were verified by compiling the same logic with SDCC and stopping the
     s51 simulator at each delay call. The 1275-count delay is the usual Keil C51 figure for about 1 ms at
     11.0592 MHz; it is compiler-dependent and not measured here (Practical 3 measures it with Keil's stopwatch).
     Add two Proteus screenshots (one per program) to ../assets/img/embedded-systems/, with alt text. -->

## Explanation

### What makes this system "reprogrammable"?

==A reprogrammable embedded system can change its function by loading new firmware, without changing its hardware.==

- The AT89C51 ==stores its program in **flash memory**==, which can be erased and written again many times.
- In this practical, the circuit never changed. Only the HEX file loaded into the chip changed, and the LEDs did something completely different.
- Older chips used ==**mask ROM** (program fixed at the factory)== or ==**OTP** (one-time programmable)== memory, which cannot be rewritten.
- Real products use this every day: firmware updates for phones, routers, TVs and cars.

### Why are the LEDs "active-low"?

Each LED's anode goes to +5 V and its cathode to a port pin.
- Pin = **0** (low): current flows from +5 V through the LED into the pin. **LED ON.**
- Pin = **1** (high): both ends are at about 5 V, so no current flows. **LED OFF.**

==On the 8051, LEDs are usually wired active-low: writing 0 to a pin turns its LED on.==

The 8051's ==port pins can sink (take in) current better== than they can source (give out) it, which is why this wiring is preferred. ==The 330 Ω resistor limits the current== to about 10 mA, so neither the LED nor the pin is damaged.

### How does Program 1 work?

- `P1 = 0x00` writes 0 to all eight pins, so all LEDs turn on.
- `delay_ms(500)` wastes about half a second, so the eye can see the change.
- `P1 = 0xFF` writes 1 to all pins, so all LEDs turn off.
- `while (1)` repeats this forever.

The delay works by making the CPU count in two nested loops. The inner loop of 1275 counts takes about 1 ms in Keil C51 at 11.0592 MHz; the outer loop repeats it `ms` times.

### How does Program 2 work?

- `led` holds a single 1-bit marking which LED should be on: `0000 0001` means P1.0.
- `~led` inverts every bit (`1111 1110` = 0xFE), because the LEDs are active-low.
- `led << 1` shifts the 1 one place left: P1.0 → P1.1 → … → P1.7.
- After P1.7, shifting once more pushes the 1 out of the 8-bit variable, so `led` becomes 0. The `if` resets it to `0x01`, and the light starts again from P1.0.

## Try it

Write Program 3, which makes the light run **backwards**, from P1.7 to P1.0. Load it into the same circuit. Which two lines of Program 2 must change?

> [!answer]- Answer
> Start with `led = 0x80;` (P1.7) and shift right with `led = led >> 1;`. Reset with `if (led == 0) led = 0x80;`. Nothing in the circuit changes, which is the point of a reprogrammable system.

## Viva questions

1. **What is a reprogrammable embedded system?** One whose behaviour can be changed by loading new firmware into its memory, without changing the hardware.
2. **Which memory makes the AT89C51 reprogrammable?** Its on-chip flash memory, which can be erased and rewritten.
3. **What is the difference between mask ROM, OTP and flash?** Mask ROM is fixed at the factory; OTP can be written once by the user; flash can be erased and written many times.
4. **Why is pin 31 (EA) connected to +5 V?** EA high makes the 8051 fetch its program from internal ROM/flash; EA low makes it use external program memory.
5. **What does "active-low" mean?** The device turns on when the pin is 0 (low).
6. **Why is a 330 Ω resistor used with each LED?** To limit current to a safe value for the LED and the port pin.
7. **What does `led << 1` do?** Shifts all bits one place left, moving the ON position to the next LED.
8. **Name two real products that are updated by reprogramming.** Smartphones (OS updates) and Wi-Fi routers (firmware updates); also cars with over-the-air updates.

## Sources

- Muhammad Ali Mazidi and Janice Mazidi, *The 8051 Microcontroller and Embedded Systems*, Pearson, Ch. 7 (time delays and I/O programming in C) (verify).
- Microchip (Atmel), *AT89C51 8-bit Microcontroller with 4K Bytes Flash* datasheet (verify).
- Labcenter Electronics, *Proteus* help on microcontroller simulation and program files (verify).
