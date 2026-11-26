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

## Before the lab

- **Install:** Keil µVision 5 (C51); Proteus is optional (to see the wave on an oscilloscope).
- **Revise first:** [[control-and-status-registers]] (TMOD, TCON, TF0, TR0), and practical 3's stopwatch method.
- **Work out first, on paper:** the timer count for 50 ms, following "Calculating the count" below. Check you get 4C00H before you type the code.
- **Have ready:** a calculator for decimal-to-hex conversion (Windows Calculator in Programmer mode works).

## Setup

### Tools

- **Keil µVision 5 (C51)** simulator, device AT89C51, Xtal **11.0592 MHz**.
- To measure the delay: the **sec** stopwatch with two breakpoints (practical 3), as in the Steps below. Optionally, **View → Analysis Windows → Logic Analyzer** in Keil, or a virtual **Oscilloscope** on P1.0 in Proteus, shows the square wave.

### Calculating the count

1. Timer clock = crystal ÷ 12 = 11.0592 MHz ÷ 12 = **921.6 kHz**, so one count takes 1 / 921.6 kHz ≈ **1.085 µs**.
2. Counts for 50 ms = 0.050 × 921,600 = **46,080**.
3. In mode 1 the timer counts up and sets TF0 when it rolls over from FFFFH to 0000H (65,536). So start from 65,536 − 46,080 = **19,456 = 4C00H**.
4. Load **TH0 = 4CH**, **TL0 = 00H**.

Some books show 4BFDH. They divide 50 ms by the rounded value 1.085 µs and get 46,083 counts; 4C00H is the exact value.

## Steps

1. Open **Keil µVision 5**.
   *You should see:* the µVision window, with an empty **Project** pane on the left.
2. Click **Project → New µVision Project**, make a new folder named `timer50`, type the project name `timer50` and click **Save**.
   *You should see:* the **Select Device for Target** window.
3. Type `AT89C51` in the search box, select it (under Microchip, or Atmel in older versions) and click **OK**.
   *You should see:* a question asking whether to copy `STARTUP.A51` to the project folder.
4. Click **No** (the C51 library already has start-up code).
   *You should see:* **Target 1 → Source Group 1** in the Project pane.
5. Click **Project → Options for Target 'Target 1'**, and on the **Target** tab set **Xtal (MHz)** to `11.0592`.
   *You should see:* 11.0592 in the Xtal box.
6. On the **Output** tab, tick **Create HEX File**, then click **OK**.
   *You should see:* the dialog close.
7. Add `main.c` with the code below and press **F7**.
   *You should see:* `"timer50" - 0 Error(s), 0 Warning(s).`
8. Press **Ctrl+F5**, then open **Peripherals → Timer → Timer 0**.
   *You should see:* the Timer 0 window showing mode, TH0, TL0, TR0 and TF0.
9. Put a breakpoint (F9) on the line `WAVE = ~WAVE;`.
   *You should see:* a red dot.
10. Press **F5** once and note **sec** in the Registers window.
    *You should see:* the program stop at the red dot.
11. Press **F5** again and note **sec** again.
    *You should see:* the difference is about **0.0500** s (50 ms), and P1.0 has changed state in Peripherals → I/O-Ports → Port 1.
12. Remove the breakpoint (F9 on the same line) and press **F5**; watch the Timer 0 window.
    *You should see:* TH0 and TL0 counting up from 4C00H, TF0 flicking to 1 at each overflow, and P1.0 toggling.

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

## Common errors and fixes

| Message or symptom | Cause | Fix |
|---|---|---|
| Program stops forever at `while (TF0 == 0);` | `TR0 = 1;` is missing, so the timer never runs | Start the timer before the wait loop |
| Delay is far too short after the first one | `TF0 = 0;` is missing, so the flag is still 1 next time | Clear TF0 after each overflow |
| Delay is wrong (about 71 ms, or very short) | TH0/TL0 not reloaded each time, or loaded the wrong way round | Load TH0 = 0x4C and TL0 = 0x00 inside the delay function, every call |
| Nothing happens on Timer 0; Timer 1 changes instead | `TMOD = 0x10` (that is Timer 1, mode 1) | Use `0x01` for Timer 0 in mode 1 |
| A build error on the line `TH0 = 4C;` | Hex numbers need the `0x` prefix | Write `TH0 = 0x4C;` |
| Measured time is about 46 ms, not 50 ms | Xtal in Options for Target is 12 MHz, not 11.0592 | Set Xtal to 11.0592 (step 5), or recalculate the count for 12 MHz |

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
