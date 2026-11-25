---
id: 8051-timer-registers-practical
title: Configuring 8051 Timer Registers to Generate a Time Delay
subject: embedded-systems
unit: 1
type: practical
practical: 4
syllabus: [UITIV-006, UDSIV-006]
status: draft
sources:
  - "Muhammad Ali Mazidi and Janice Mazidi, The 8051 Microcontroller and Embedded Systems, Pearson, Ch. 9 (verify)"
  - "Intel, MCS-51 Microcontroller Family User's Manual, timer/counters (verify)"
---

## Aim

To configure the timer control registers (TMOD, TCON) of the 8051 and use Timer 0 in mode 1 to generate an exact 50 ms delay, producing a 10 Hz square wave on pin P1.0.

<!-- Covers syllabus Unit 1 practicals 5 (configure timer control registers) and 6 (generate a specific delay). -->

## Concepts used

- [[control-and-status-registers]] (TMOD, TCON, TF0, TR0)
- [[8051-debugging-practical]] (Keil stopwatch)

## Setup

### Tools

- **Keil µVision 5 (C51)** simulator, device AT89C51, Xtal **11.0592 MHz**.
- To see the wave: **Debug → Logic Analyzer**, add signal `P1.0` (type `PORT1.0`), run, and read the period. In **Proteus**, connect a virtual **Oscilloscope** to P1.0.

### Calculating the count

1. Timer clock = crystal ÷ 12 = 11.0592 MHz ÷ 12 = **921.6 kHz**, so one count takes 1 / 921.6 kHz ≈ **1.085 µs**.
2. Counts for 50 ms = 0.050 × 921,600 = **46,080**.
3. In mode 1 the timer counts up and sets TF0 when it rolls over from FFFFH to 0000H (65,536). So start from 65,536 − 46,080 = **19,456 = 4C00H**.
4. Load **TH0 = 4CH**, **TL0 = 00H**.

Some books show 4BFDH. They divide 50 ms by the rounded value 1.085 µs and get 46,083 counts; 4C00H is the exact value.

## Code

```c
#include <reg51.h>

sbit WAVE = P1^0;                   // square wave output pin

void delay_50ms(void)
{
    TMOD = (TMOD & 0xF0) | 0x01;    // Timer 0, mode 1 (16-bit); leave Timer 1 bits unchanged
    TH0 = 0x4C;                     // start count 4C00H = 19456
    TL0 = 0x00;
    TR0 = 1;                        // start Timer 0           (control bit in TCON)
    while (TF0 == 0);               // wait for overflow flag   (status bit in TCON)
    TR0 = 0;                        // stop Timer 0
    TF0 = 0;                        // clear the flag for the next delay
}

void main(void)
{
    while (1)
    {
        WAVE = ~WAVE;               // toggle P1.0
        delay_50ms();               // 50 ms high, 50 ms low: period 100 ms, 10 Hz
    }
}
```

## Output

```output
Time between toggles of P1.0 : 50.02 ms   (553,176 clock cycles at 11.0592 MHz)
Square wave on P1.0          : 50 ms high, 50 ms low, period 100 ms, frequency about 10 Hz
```

The extra 0.02 ms (216 clock cycles) is the time taken by the instructions that reload TH0/TL0, start the timer and toggle the pin.

<!-- For Prathamesh: the 50.02 ms was measured by compiling the same program with SDCC and timing the toggles in
     the s51 simulator at 11.0592 MHz. Keil's reload overhead may differ by a few microseconds. Add a Logic Analyzer
     or Proteus oscilloscope screenshot to ../assets/img/embedded-systems/, with alt text. -->

## Explanation

### Which registers control Timer 0?

==Timer 0 is controlled by TMOD (mode), TCON (run and overflow bits), and TH0/TL0 (the count).==

| Register | Bits used here | Role |
|---|---|---|
| ==**TMOD**== (89H) | Lower nibble 0001: GATE = 0, C/T = 0, M1 M0 = 01 | Timer (not counter), mode 1, 16-bit |
| ==**TCON**== (88H) | TR0 (bit 4), TF0 (bit 5) | TR0 starts/stops; TF0 shows overflow |
| ==**TH0, TL0**== | Whole bytes | High and low bytes of the 16-bit count |

### What are the four timer modes?

| M1 M0 | Mode | Use |
|---|---|---|
| 00 | ==Mode 0: 13-bit timer== | Old compatibility mode |
| 01 | ==Mode 1: 16-bit timer== | Delays up to 71 ms at 11.0592 MHz |
| 10 | ==Mode 2: 8-bit auto-reload== | Baud rates, fixed short intervals |
| 11 | ==Mode 3: split timer== | Timer 0 becomes two 8-bit timers |

The longest mode 1 delay is 65,536 counts × 1.085 µs ≈ 71.1 ms. Longer delays repeat a shorter one (practical 10).

### How does the delay function work, step by step?

1. ==**Set the mode**== in TMOD without disturbing Timer 1.
2. ==**Load the start count**== 4C00H into TH0 and TL0.
3. ==**Start the timer**== with TR0 = 1. It now counts up by one every 1.085 µs, independently of the CPU.
4. ==**Poll the flag**==: wait until TF0 becomes 1 (after 46,080 counts).
5. ==**Stop and clear**==: TR0 = 0, then TF0 = 0, ready for the next call.

### Why is a timer delay better than a counting loop?

- ==The time depends only on the crystal==, not on the compiler or optimisation settings.
- ==It is calculated exactly== (46,080 counts), not found by trial.
- ==It can run in the background==: with an interrupt instead of polling, the CPU can do other work while the timer counts (Unit 3 and Unit 4 topics).

## Try it

Change the program to give a **25 ms** delay (20 Hz wave). Calculate the new TH0 and TL0.

> [!answer]- Answer
> Counts = 0.025 × 921,600 = 23,040. Start value = 65,536 − 23,040 = 42,496 = **A600H**, so TH0 = 0xA6, TL0 = 0x00. P1.0 now toggles every 25 ms: period 50 ms, 20 Hz.

## Viva questions

1. **What is the timer clock frequency for an 11.0592 MHz crystal?** 11.0592 MHz ÷ 12 = 921.6 kHz; one count ≈ 1.085 µs.
2. **What value is loaded in TMOD for Timer 0, mode 1?** 01H (lower nibble 0001).
3. **What does TF0 indicate?** That Timer 0 has overflowed (rolled from FFFFH to 0000H).
4. **Why must TF0 be cleared after each delay?** It stays 1 until cleared; otherwise the next wait ends immediately.
5. **What is the maximum delay in mode 1 at 11.0592 MHz?** About 71.1 ms (65,536 × 1.085 µs).
6. **Why do we write `TMOD = (TMOD & 0xF0) | 0x01`?** To set Timer 0's bits without changing Timer 1's, which may be in use.
7. **What is the difference between a timer and a counter?** A timer counts internal machine cycles; a counter counts pulses on an external pin (T0 or T1). The C/T bit selects it.
8. **What is mode 2 used for?** 8-bit auto-reload, mainly for generating serial baud rates with Timer 1.

## Sources

- Muhammad Ali Mazidi and Janice Mazidi, *The 8051 Microcontroller and Embedded Systems*, Pearson, Ch. 9 (timer programming) (verify).
- Intel, *MCS-51 Microcontroller Family User's Manual*, timer/counters (verify).
