---
id: led-patterns-practical
title: Sequential, Alternate and Table-Driven LED Patterns
subject: embedded-systems
unit: 2
type: practical
practical: 9
syllabus: [UITIV-006, UDSIV-006]
status: draft
sources:
  - "Muhammad Ali Mazidi and Janice Mazidi, The 8051 Microcontroller and Embedded Systems, Pearson, Ch. 7 (verify)"
  - "Arm Keil, C51 User's Guide: the code memory type (verify)"
---

## Aim

To generate sequential and alternate LED patterns using the 8051's I/O ports, and to design an LED control application that plays a pattern stored in a lookup table, as a decorative light controller.

<!-- Covers syllabus Unit 2 practicals 3 (sequential patterns), 4 (alternate blinking) and 7 (LED control applications). -->

## Concepts used

- [[led-binary-counter-practical]] (active-low LEDs, binary place values)
- [[embedded-firmware]] (super loop, functions)

## Before the lab

- **Install:** Keil µVision 5 (C51) and Proteus 8.
- **Revise first:** practical 8 (active-low LEDs, P1.7 on the left), and the bit operators `<<`, `|` and `~`.
- **Work out first, on paper:** the binary of 0x55 and 0xAA, and draw which LEDs are on for each.
- **Have ready:** the eight-LED circuit from practical 8.

## Setup

### Tools

- **Keil µVision 5 (C51)**: build `patterns.hex`.
- **Proteus**: the eight-LED circuit from practical 8 (LEDs on P1.0 … P1.7, active-low, P1.7 on the left).

## Steps

1. Create Keil project `patterns` (as in practical 1 steps 1–6), add the code below and press **F7**.
   *You should see:* `"patterns" - 0 Error(s), 0 Warning(s).`
2. Press **Ctrl+F5**, open **Peripherals → I/O-Ports → Port 1**, press **F5**, and watch for a few seconds.
   *You should see:* P1 alternate 0x55 and 0xAA four times, then FE, FC, F8 … 00.
3. End the debug session.
   *You should see:* the editor layout again.
4. In Proteus, load `patterns.hex` into the AT89C51 of the practical 8 circuit (clock 11.0592 MHz) and click **Run**.
   *You should see:* odd and even LEDs taking turns, four times.
5. Keep watching.
   *You should see:* LEDs filling up one by one from the right until all eight are on.
6. Keep watching.
   *You should see:* two lights starting at both ends, meeting in the middle and moving out again; then everything repeats.

## Code

```c
#include <reg51.h>

// "meet in the middle" pattern: 1 = LED on (bit 7 = left LED)
unsigned char code meet[6] = {0x81, 0x42, 0x24, 0x18, 0x24, 0x42};

void delay_ms(unsigned int ms)    // about 1 ms per count in Keil (measured in practical 3)
{
    unsigned int i, j;
    for (i = 0; i < ms; i++)
        for (j = 0; j < 1275; j++);
}

void alternate(void)              // pattern 1: odd and even LEDs take turns
{
    unsigned char k;
    for (k = 0; k < 4; k++)
    {
        P1 = 0x55;                // LEDs P1.7, P1.5, P1.3, P1.1 ON (active-low)
        delay_ms(300);
        P1 = 0xAA;                // LEDs P1.6, P1.4, P1.2, P1.0 ON
        delay_ms(300);
    }
}

void fill_up(void)                // pattern 2: LEDs fill one by one from the right
{
    unsigned char led = 0x00, k;
    for (k = 0; k < 8; k++)
    {
        led = (led << 1) | 0x01;  // add one more 1 on the right
        P1 = ~led;                // invert for active-low LEDs
        delay_ms(300);
    }
}

void meet_in_middle(void)         // pattern 3: played from the lookup table
{
    unsigned char k;
    for (k = 0; k < 6; k++)
    {
        P1 = ~meet[k];
        delay_ms(300);
    }
}

void main(void)
{
    while (1)                     // super loop: play the three patterns forever
    {
        alternate();
        fill_up();
        meet_in_middle();
    }
}
```

## Output

```output
Pattern 1 (alternate):   P1 = 55, AA, 55, AA, 55, AA, 55, AA
                          ●.●.●.●.  then  .●.●.●.●  (four times)
Pattern 2 (fill up):     P1 = FE, FC, F8, F0, E0, C0, 80, 00
                          .......●  ......●●  ...  ●●●●●●●●
Pattern 3 (meet):        P1 = 7E, BD, DB, E7, DB, BD
                          ●......●  .●....●.  ..●..●..  ...●●...  ..●..●..  .●....●.
Then the whole sequence repeats.
```

(● = LED on, shown with P1.7 on the left.)

<!-- For Prathamesh: the full P1 sequence above was verified with SDCC + s51 (21 consecutive outputs).
     Add a Proteus screenshot or short screen recording of the patterns, with alt text. -->

## Explanation

### How are LED patterns made?

==An LED pattern is a sequence of bytes written to a port, one after another, with a delay between them.==

- ==**Each byte is one "frame"**==: its bits decide which LEDs are on.
- ==**The delay sets the speed**==: 300 ms per frame here.
- ==**Bit operations generate patterns**==: shifting (`<<`, `>>`), inverting (`~`) and OR-ing (`|`).
- ==**A lookup table stores patterns**== that are hard to calculate, such as "meet in the middle".

### How do the three patterns work?

- ==**Alternate:**== 0x55 (0101 0101) and 0xAA (1010 1010) are exact opposites, so the odd and even LEDs take turns.
- ==**Fill up:**== `led = (led << 1) | 0x01;` shifts the pattern left and adds a new 1 on the right: 0000 0001, 0000 0011, 0000 0111 … 1111 1111.
- ==**Meet in the middle:**== the table holds 1000 0001, 0100 0010, 0010 0100, 0001 1000 and back, so two lights move inwards and outwards.

Each pattern is a separate **function**, so the main loop reads like a playlist and new patterns are easy to add.

### Why is the table declared with `code`?

- ==Keil's `code` keyword stores the table in program memory (flash)== instead of RAM.
- The 8051 has only 128 bytes of RAM, but 4 KB or more of flash, so constant tables belong in flash.
- The table is read-only, which is correct for fixed patterns.

### Where is this used in real life?

Decorative festival light controllers (Diwali serial lights), shop sign boards, running-light indicators on vehicles, and LED bar-graph level meters all use the same idea: a sequence of bytes on a port, timed by a delay or a timer.

## Common errors and fixes

| Message or symptom | Cause | Fix |
|---|---|---|
| `error C249: 'DATA': SEGMENT TOO LARGE` | Large pattern tables declared **without** `code`, so they are put in the 128-byte RAM | Declare tables as `unsigned char code name[] = {...};` |
| `warning C206: 'alternate': missing function-prototype` | A pattern function is written below `main` | Write the functions above `main`, or add prototypes |
| Patterns look mirror-imaged | LEDs placed with P1.0 on the left | Put P1.7 on the left |
| Fill-up pattern lights from the wrong end, or all LEDs at once | `>>` used instead of `<<`, or `| 0x01` left out | Use `led = (led << 1) | 0x01;` |
| Meet-in-the-middle shows the inverse (lit LEDs dark) | `~` missing before `meet[k]` | Write `P1 = ~meet[k];` |
| Only the first pattern ever plays | A `while (1)` was put inside `alternate()` | Each pattern function must end; only `main` has the endless loop |

## Try it

Add a fourth pattern, **"Knight Rider"**: one LED moves from right to left and then back from left to right, without jumping. Write it as a lookup table.

> [!answer]- Answer
> `unsigned char code rider[14] = {0x01,0x02,0x04,0x08,0x10,0x20,0x40,0x80,0x40,0x20,0x10,0x08,0x04,0x02};` and play it like `meet_in_middle()`, writing `P1 = ~rider[k];` for k = 0 to 13. The end values 0x80 and 0x01 appear only once each, so the light does not pause at the ends.

## Viva questions

1. **What is a lookup table?** An array of precalculated values that the program reads in order instead of calculating them.
2. **Why is the table stored with the `code` keyword?** To keep it in flash program memory, saving the 8051's small RAM.
3. **What do 0x55 and 0xAA look like in binary?** 0101 0101 and 1010 1010: alternate bits, each the inverse of the other.
4. **What does `(led << 1) | 0x01` do?** Shifts the bits left by one and sets bit 0, adding one more lit LED.
5. **How do you change the speed of a pattern?** Change the delay between frames.
6. **Why is each pattern written as a separate function?** It keeps the main loop simple and makes patterns easy to reuse or add.
7. **Why is `~` used before writing the table values?** The table stores 1 = ON, but the LEDs are active-low.
8. **Give two real applications of LED patterns.** Festival serial-light controllers and LED level meters (bar graphs).

## Sources

- Muhammad Ali Mazidi and Janice Mazidi, *The 8051 Microcontroller and Embedded Systems*, Pearson, Ch. 7 (logic and shift operations in 8051 C) (verify).
- Arm Keil, *C51 User's Guide*, memory types: `code` (verify).
