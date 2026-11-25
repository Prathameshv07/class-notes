---
id: led-binary-counter-practical
title: 8-Bit Binary Counter on LEDs
subject: embedded-systems
unit: 2
type: practical
practical: 8
syllabus: [UITIV-006, UDSIV-006]
status: draft
sources:
  - "Muhammad Ali Mazidi and Janice Mazidi, The 8051 Microcontroller and Embedded Systems, Pearson, Ch. 7 (verify)"
---

## Aim

To simulate an 8-bit binary up-counter (0 to 255) using eight LEDs on Port 1 of the 8051, displaying each count in binary for half a second.

## Concepts used

- [[led-interfacing-practical]] (active-low LEDs, resistor)
- [[8051-programming-environment-practical]] (`unsigned char` and its range)

## Setup

### Tools

- **Keil µVision 5 (C51)**: build `counter.hex`; you can also watch Port 1 in the simulator (Peripherals → I/O-Ports → Port 1).
- **Proteus**: AT89C51 with eight LED-RED + 330 Ω resistors on P1.0 … P1.7, wired active-low (the circuit from practical 2). Use a driver chip or 2.2 kΩ resistors on a real board, as up to eight LEDs are lit together.

Place the LED for **P1.7 on the left** and **P1.0 on the right**, so the LEDs read like a binary number.

## Code

```c
#include <reg51.h>

void delay_ms(unsigned int ms)    // about 1 ms per count in Keil (measured in practical 3)
{
    unsigned int i, j;
    for (i = 0; i < ms; i++)
        for (j = 0; j < 1275; j++);
}

void main(void)
{
    unsigned char count = 0;      // 8-bit counter: 0 to 255

    while (1)
    {
        P1 = ~count;              // invert: active-low LEDs show 1-bits as ON
        delay_ms(500);
        count++;                  // after 255, wraps round to 0
    }
}
```

## Output

```output
count   binary      P1 (= ~count)   LEDs ON (P1.7 ... P1.0)
0       0000 0000   0xFF            . . . . . . . .
1       0000 0001   0xFE            . . . . . . . ●
2       0000 0010   0xFD            . . . . . . ● .
3       0000 0011   0xFC            . . . . . . ● ●
4       0000 0100   0xFB            . . . . . ● . .
...
255     1111 1111   0x00            ● ● ● ● ● ● ● ●
0       0000 0000   0xFF            . . . . . . . .    (wraps round and starts again)
```

<!-- For Prathamesh: P1 = FF, FE, FD, FC, FB verified with SDCC + s51, and the 255 -> 0 wrap verified
     separately. A full cycle takes 256 x 0.5 s = 128 s in Keil (about 2 minutes). Add a Proteus screenshot. -->

## Explanation

### How does the counter work?

- ==The counter is an 8-bit variable== (`unsigned char count`), so it holds 0 to 255: exactly what 8 LEDs can show.
- ==The output is inverted before display== (`P1 = ~count;`), because the LEDs are active-low (0 = ON). Without `~`, the LEDs would show the opposite pattern.
- ==The count goes up by one each pass== (`count++`); the delay keeps each value visible for half a second.
- ==After 255 (1111 1111), adding 1 overflows== the 8-bit variable to 0, so the counter restarts by itself. No `if` is needed.

### How do you read the LEDs as a binary number?

==Each LED is one bit, with a place value that doubles from right to left.==

| LED | P1.7 | P1.6 | P1.5 | P1.4 | P1.3 | P1.2 | P1.1 | P1.0 |
|---|---|---|---|---|---|---|---|---|
| Place value | 128 | 64 | 32 | 16 | 8 | 4 | 2 | 1 |

Example: LEDs on at P1.3, P1.1 and P1.0 → 8 + 2 + 1 = **11**.

Notice the pattern: P1.0 changes every step, P1.1 every 2 steps, P1.2 every 4 steps, and P1.7 only every 128 steps. Each bit blinks at half the speed of the bit to its right.

### How long does a full count take?

256 counts × 0.5 s = **128 seconds** (about 2 minutes and 8 seconds) before the counter wraps round to 0.

## Try it

Change the program into a **down-counter** from 255 to 0. Which lines change?

> [!answer]- Answer
> Start with `unsigned char count = 255;` and use `count--;`. After 0, subtracting 1 wraps round to 255, so it keeps counting down forever.

## Viva questions

1. **Why is `unsigned char` used for the counter?** It holds 0 to 255, matching the 8 LEDs, and wraps round to 0 after 255.
2. **Why is `~count` written to the port?** The LEDs are active-low, so the bits must be inverted for a 1 to light an LED.
3. **What is the place value of the LED on P1.5?** 32.
4. **What happens after the count reaches 255?** It overflows and becomes 0.
5. **How long does one full cycle take with a 500 ms delay?** 256 × 0.5 s = 128 s.
6. **Which LED blinks fastest, and why?** P1.0, the least significant bit, because it changes on every count.
7. **How would you show the count on 4 LEDs only?** Mask the upper bits: `P1 = ~(count & 0x0F);`, giving a 0–15 counter.
8. **What is the decimal value when only P1.7 and P1.0 are on?** 128 + 1 = 129.

## Sources

- Muhammad Ali Mazidi and Janice Mazidi, *The 8051 Microcontroller and Embedded Systems*, Pearson, Ch. 7 (I/O and logic operations in 8051 C) (verify).
