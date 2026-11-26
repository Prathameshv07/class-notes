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
  - "Microchip (Atmel), AT89C51 datasheet: DC characteristics, IOL limits"
  - "Labcenter Electronics, Proteus help: loading a program file into a microcontroller (verify)"
---

## Aim

To design and develop a reprogrammable embedded system using the 8051 microcontroller, by loading two different programs into the same LED circuit and observing that the behaviour changes without changing the hardware.

## Concepts used

- [[8051-programming-environment-practical]] (creating a project and HEX file in Keil)
- [[embedded-vs-general-purpose-systems]] (firmware)

## Before the lab

- **Install:** Keil µVision 5 (C51) and **Proteus 8** (ISIS schematic capture), as available in the lab.
- **Revise first:** practical 1 ([[8051-programming-environment-practical]]), especially creating a project and a HEX file, and the meaning of active-low (Explanation below).
- **Know these Proteus names:** **P** = Pick Devices, **Terminals Mode** (for POWER and GROUND), **Run** button (bottom left).
- **Have ready:** a folder with two sub-folders, `blink` and `runlight`, one Keil project in each.

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

## Steps

1. In Keil, create project `blink` in the `blink` folder, exactly as in practical 1 steps 1–6 (AT89C51, 11.0592 MHz, Create HEX File ticked).
   *You should see:* Target 1 → Source Group 1 in the Project pane.
2. Add `main.c` with **Program 1** and press **F7**.
   *You should see:* `"blink" - 0 Error(s), 0 Warning(s).` and `creating hex file from "blink"...`.
3. Create project `runlight` in the `runlight` folder the same way, with **Program 2**, and build.
   *You should see:* `"runlight" - 0 Error(s), 0 Warning(s).`
4. Open **Proteus 8**, click **File → New Project**, give it a name and click **Next** through the wizard (default schematic, no PCB, no firmware project), then **Finish**.
   *You should see:* an empty blue-grid schematic sheet.
5. Press **P**, search `AT89C51`, double-click it; search `LED-RED` and `RES` and double-click each; close the window.
   *You should see:* AT89C51, LED-RED and RES in the **Devices** list on the left.
6. Place the AT89C51, eight LEDs and eight resistors, and wire them as in the circuit table. Add a **POWER** terminal (Terminals Mode) for +5 V.
   *You should see:* each LED's anode reaching +5 V through a resistor, and each cathode reaching a P1 pin.
7. Double-click each resistor and set **Resistance** to `330`.
   *You should see:* the label under each resistor change from 10k to 330.
8. Double-click the AT89C51, set **Program File** to `blink.hex` and **Clock Frequency** to `11.0592MHz`, click **OK**.
   *You should see:* the dialog close; nothing moves yet.
9. Click **Run** (the ▶ button at the bottom left).
   *You should see:* all eight LEDs turn on and off together.
10. Click **Stop** (■). Double-click the AT89C51 and change **Program File** to `runlight.hex`, click **OK**.
    *You should see:* the LEDs go dark while stopped.
11. Click **Run** again.
    *You should see:* one LED moving from P1.0 to P1.7 and starting again. Same circuit, new behaviour: the system was reprogrammed.

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

The 8051's ==port pins can sink (take in) current better== than they can source (give out) it, which is why this wiring is preferred. ==The 330 Ω resistor limits the current== to about 9 mA per LED.

On real hardware, check the totals: the AT89C51 datasheet allows at most 10 mA per pin but only 15 mA for a whole port (P1 to P3). Eight LEDs at 9 mA each is about 72 mA, far over that limit. Proteus does not complain, but on a real board use a driver chip (lab trainer boards already have one) or larger resistors, such as 2.2 kΩ (about 1.4 mA each).

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

## Common errors and fixes

| Message or symptom | Cause | Fix |
|---|---|---|
| LEDs never light, or glow very dimly | The resistors are still Proteus's default **10k** | Set each resistor to 330 Ω (step 7) |
| One LED never lights | That LED is placed the wrong way round | Rotate or flip it: anode (triangle side) to the resistor, cathode (bar side) to the pin |
| Simulation will not start; the log says the program file cannot be loaded | Wrong path, or the HEX file was not created | Rebuild in Keil with Create HEX File ticked, and browse to the `.hex` again |
| `Simulation is not running in real time due to excessive CPU load.` | The PC cannot keep up with the simulated 11.0592 MHz | Harmless: the LEDs just change more slowly. Close other programs if it is very slow |
| `warning C206: 'delay_ms': missing function-prototype` and `error C267: 'delay_ms': requires ANSI-style prototype` | `delay_ms` is written **below** `main`, so the compiler meets the call first | Move the function above `main`, or add `void delay_ms(unsigned int ms);` before `main` |
| LEDs blink, but in the opposite way to what you expect | The LEDs are wired active-high (pin → LED → ground) | Wire them active-low as in the table, or invert the values in the program |

## Try it

Write Program 3, which makes the light run **backwards**, from P1.7 to P1.0. Load it into the same circuit. Which three lines of Program 2 must change?

> [!answer]- Answer
> Start with `led = 0x80;` (P1.7) and shift right with `led = led >> 1;`. Reset with `if (led == 0) led = 0x80;`. Nothing in the circuit changes, which is the point of a reprogrammable system.

## Viva questions

1. **What is a reprogrammable embedded system?** One whose behaviour can be changed by loading new firmware into its memory, without changing the hardware.
2. **Which memory makes the AT89C51 reprogrammable?** Its on-chip flash memory, which can be erased and rewritten.
3. **What is the difference between mask ROM, OTP and flash?** Mask ROM is fixed at the factory; OTP can be written once by the user; flash can be erased and written many times.
4. **Why is pin 31 (EA) connected to +5 V?** EA high makes the 8051 fetch its program from internal ROM/flash; EA low makes it use external program memory.
5. **What does "active-low" mean?** The device turns on when the pin is 0 (low).
6. **Why is a 330 Ω resistor used with each LED?** To limit the current through the LED and the port pin. With many LEDs on one port, the port's total limit (15 mA on the AT89C51) also matters.
7. **What does `led << 1` do?** Shifts all bits one place left, moving the ON position to the next LED.
8. **Name two real products that are updated by reprogramming.** Smartphones (OS updates) and Wi-Fi routers (firmware updates); also cars with over-the-air updates.

## Sources

- Muhammad Ali Mazidi and Janice Mazidi, *The 8051 Microcontroller and Embedded Systems*, Pearson, Ch. 7 (time delays and I/O programming in C) (verify).
- Microchip (Atmel), *AT89C51 8-bit Microcontroller with 4K Bytes Flash* datasheet, DC characteristics (current limits per pin and per port).
- Labcenter Electronics, *Proteus* help on microcontroller simulation and program files (verify).
