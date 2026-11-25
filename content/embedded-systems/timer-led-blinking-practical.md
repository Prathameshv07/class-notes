---
id: timer-led-blinking-practical
title: Timer-Based LED Blinking vs Software Loop Delay
subject: embedded-systems
unit: 2
type: practical
practical: 10
syllabus: [UITIV-006, UDSIV-006]
status: draft
sources:
  - "Muhammad Ali Mazidi and Janice Mazidi, The 8051 Microcontroller and Embedded Systems, Pearson, Ch. 9 (verify)"
  - "SDCC Compiler User Guide and ucsim s51 simulator (used to compare delays)"
---

## Aim

To blink an LED every 500 ms using Timer 0 of the 8051, and to compare this timer-based delay with a software loop delay in terms of accuracy, dependence on the compiler and use of the CPU.

<!-- Covers syllabus Unit 2 practicals 5 (timer-based LED blinking) and 6 (compare timer delay with software loop delay). -->

## Concepts used

- [[8051-timer-registers-practical]] (Timer 0, mode 1, 50 ms delay)
- [[8051-debugging-practical]] (measuring time with the stopwatch)

## Setup

### Tools

- **Keil µVision 5 (C51)**, AT89C51, Xtal 11.0592 MHz: measure each delay with the **sec** counter (practical 3) or the Logic Analyzer.
- **Proteus**: one LED with 330 Ω on P1.0, active-low.

### Steps

1. Build the timer program below and measure the time between LED toggles.
2. Build the software-delay version: replace `delay_500ms();` with `delay_ms(500);` from practical 2, and measure again.
3. Fill in the comparison table in your journal.

## Code

```c
#include <reg51.h>

sbit LED = P1^0;

void delay_50ms(void)                 // exact 50 ms with Timer 0 (practical 4)
{
    TMOD = (TMOD & 0xF0) | 0x01;      // Timer 0, mode 1
    TH0 = 0x4C;                       // 65536 - 46080 = 4C00H
    TL0 = 0x00;
    TR0 = 1;
    while (TF0 == 0);
    TR0 = 0;
    TF0 = 0;
}

void delay_500ms(void)                // 10 x 50 ms = 500 ms
{
    unsigned char k;
    for (k = 0; k < 10; k++)
        delay_50ms();
}

void main(void)
{
    while (1)
    {
        LED = ~LED;                   // toggle the LED
        delay_500ms();                // on 500 ms, off 500 ms: 1 blink per second
    }
}
```

## Output

```output
Timer-based delay, measured time between toggles:      500.2 ms   (set by the crystal, not the compiler)

Software loop delay_ms(500) with the 1275-count loop:
  Keil C51 (the value the 1275 was chosen for):        about 500 ms   <- measure and record in your journal
  SDCC (another 8051 C compiler), same C code:         4851 ms        (almost 10 times longer)
```

<!-- For Prathamesh: 500.2 ms (5,532,216 clock cycles) was measured with SDCC + s51 at 11.0592 MHz; the timer
     value depends only on the crystal, so Keil gives the same apart from a few microseconds of overhead.
     4851 ms is the SDCC + s51 measurement of the software loop (53,646,276 cycles). The Keil figure is not
     measured here: students record it with the stopwatch. Add a Logic Analyzer screenshot, with alt text. -->

## Explanation

### Why does the timer version give exactly 500 ms?

- ==**The timer counts machine cycles in hardware**==: one count every 1.085 µs at 11.0592 MHz, whatever the program is doing.
- ==**The count is calculated**==: 46,080 counts = 50 ms, repeated 10 times = 500 ms.
- ==**Only a few instructions add overhead**==: reloading TH0/TL0 and looping add about 0.2 ms in total.

==A timer delay depends only on the crystal frequency, so it is accurate and repeatable.==

### Why did the same software loop give such different times?

A software delay is just the CPU running empty loops. Its time depends on **how many machine instructions** the compiler produces for `for (j = 0; j < 1275; j++);`:

- Keil C51 produces a short, fast loop, so 1275 passes take about 1 ms.
- SDCC produced a longer loop for the same C line, so 1275 passes took about 9.7 ms.
- Changing Keil's **optimisation level** can also change the time.

So a loop count found by trial on one compiler is wrong on another. This is why practical 3 asked you to **measure** the delay instead of trusting the number 1275.

### How do the two methods compare?

| Point | Software loop delay | Timer-based delay |
|---|---|---|
| ==Accuracy== | Approximate; found by trial | Exact; calculated from the crystal |
| ==Depends on compiler and optimisation?== | Yes | No |
| ==Depends on crystal frequency?== | Yes | Yes (recalculate the count) |
| ==CPU use== | CPU is busy counting all the time | CPU only waits for TF0 (or is free, with an interrupt) |
| ==Hardware needed== | None | Uses Timer 0 |
| ==Best for== | Rough, short delays (switch debounce) | Precise timing: clocks, baud rates, waveforms |

### How can the CPU be freed completely?

In this practical the CPU still **polls** TF0 in a loop. With a **Timer 0 interrupt** (vector 000BH), the timer interrupts the CPU every 50 ms, a small ISR counts to 10 and toggles the LED, and the main loop is free for other work such as reading switches. This is how real products blink a status LED while doing their main job.

## Try it

Change the timer program to blink the LED **twice per second** (on 250 ms, off 250 ms). Which number changes?

> [!answer]- Answer
> In `delay_500ms()`, loop 5 times instead of 10 (5 × 50 ms = 250 ms), and rename it `delay_250ms()`. The timer values TH0 = 4CH, TL0 = 00H stay the same.

## Viva questions

1. **Why is a timer delay more accurate than a software loop?** The timer counts hardware machine cycles at a fixed rate; a loop's time depends on the instructions the compiler generates.
2. **Why did the same C delay loop take different times with Keil and SDCC?** Each compiler generates different machine code for the loop.
3. **How is a 500 ms delay made with a 16-bit timer that can count only about 71 ms?** By repeating a 50 ms timer delay 10 times.
4. **What is polling?** Checking a flag (here TF0) repeatedly in a loop until it changes.
5. **How does an interrupt improve timer-based blinking?** The CPU is free between timer overflows; an ISR handles the LED.
6. **What is the vector address of the Timer 0 interrupt?** 000BH.
7. **When is a software delay acceptable?** For short, non-critical delays, such as waiting 20 ms for a switch to stop bouncing.
8. **If the crystal is changed to 12 MHz, what must change in the timer program?** The count: 12 MHz / 12 = 1 MHz, so 50 ms = 50,000 counts; load 65,536 − 50,000 = 15,536 = 3CB0H.

## Sources

- Muhammad Ali Mazidi and Janice Mazidi, *The 8051 Microcontroller and Embedded Systems*, Pearson, Ch. 9 (timer programming in C) (verify).
- SDCC (Small Device C Compiler) user guide and the ucsim s51 simulator, used here to measure both delays.
