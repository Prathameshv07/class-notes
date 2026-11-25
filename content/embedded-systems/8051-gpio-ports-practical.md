---
id: 8051-gpio-ports-practical
title: Configuring 8051 General-Purpose I/O Ports
subject: embedded-systems
unit: 1
type: practical
practical: 5
syllabus: [UITIV-006, UDSIV-006]
status: draft
sources:
  - "Muhammad Ali Mazidi and Janice Mazidi, The 8051 Microcontroller and Embedded Systems, Pearson, Ch. 4 and Ch. 7 (verify)"
  - "Microchip (Atmel), AT89C51 datasheet, port descriptions"
---

## Aim

To configure the general-purpose I/O ports of the 8051 as inputs and outputs, by reading eight switches on Port 2 and showing their state on eight LEDs on Port 1 (byte-wise I/O), and by controlling one LED on P3.7 from one switch on P3.2 (bit-wise I/O).

## Concepts used

- [[8051-reprogrammable-system-practical]] (active-low LED wiring)
- [[memory-io-interrupt-map]] (ports as SFRs)

## Setup

### Tools

- **Keil µVision 5 (C51)** to write and build the program.
- **Proteus** to simulate the circuit.

### Circuit in Proteus

| Part | Connection |
|---|---|
| AT89C51 | EA (pin 31) to +5 V |
| 8 × SWITCH (or BUTTON) | Between P2.0 … P2.7 and ground |
| 8 × LED-RED + 330 Ω | +5 V → resistor → LED → P1.0 … P1.7 (active-low) |
| 1 × BUTTON | Between P3.2 and ground |
| 1 × LED-GREEN + 330 Ω | +5 V → resistor → LED → P3.7 |

No pull-up resistors are needed on P1, P2 and P3: they have internal pull-ups. (Port 0 has none; it would need external pull-up resistors.)

## Code

```c
#include <reg51.h>

sbit SW  = P3^2;          // single push button
sbit LED = P3^7;          // single indicator LED

void main(void)
{
    P2 = 0xFF;            // write 1s: makes all Port 2 pins inputs
    SW = 1;               // write 1: makes P3.2 an input

    while (1)
    {
        P1 = P2;          // byte I/O: copy 8 switches to 8 LEDs
        LED = SW;         // bit I/O: copy one switch to one LED
    }
}
```

## Output

```output
Switches on P2 pressed   P2 read   P1 written   LEDs ON
None                     0xFF      0xFF         none
P2.1 and P2.3            0xF5      0xF5         P1.1, P1.3
P2.0 and P2.7            0x7E      0x7E         P1.0, P1.7

Button on P3.2           P3.2      P3.7         Green LED
Pressed                  0         0            ON
Released                 1         1            OFF
```

A pressed switch connects its pin to ground (0); because the LEDs are active-low, a 0 also turns the matching LED on. So **each LED lights while its switch is pressed**.

<!-- For Prathamesh: verified with SDCC + the s51 simulator by driving the Port 2 and Port 3 pins
     (0xF5, 0x7E, P3.2 low/high) and reading P1 and P3.7. Add a Proteus screenshot, with alt text. -->

## Explanation

### How does an 8051 port pin become an input or an output?

==8051 port pins have no direction register: writing 1 to a pin makes it usable as an input; writing a value makes it an output.==

- ==**Output:**== write 0 or 1 to the port or pin; the pin goes low or high.
- ==**Input:**== first write 1. The output transistor then turns off and the weak internal pull-up holds the pin high, so an external switch can pull it low.
- ==**After reset, all ports hold FFH**==, so they start as inputs; writing 1 again in the program makes the intent clear.
- ==**Reading a port**== (`x = P2;`) reads the actual pin levels.

### What is the difference between byte I/O and bit I/O?

| | Byte I/O | Bit I/O |
|---|---|---|
| ==Accesses== | All 8 pins at once | One pin |
| C syntax | `P1 = P2;` | `sbit LED = P3^7;` then `LED = SW;` |
| ==Used for== | LED bars, 7-segment data, parallel data | Single switch, LED, relay, buzzer |

==The `sbit` keyword names one bit of a bit-addressable SFR==, such as P3^7 for pin 3.7.

### Why are switches connected to ground?

- The internal pull-up keeps the pin at **1** when the switch is open.
- Pressing the switch connects the pin to ground: the pin reads **0**.
- So a pressed switch reads 0 (**active-low input**). Connecting the switch to +5 V instead would not work, because the pin is already high.

### What are the special features of each port?

| Port | Pull-ups | Other use |
|---|---|---|
| ==P0== | None (open drain): needs external pull-ups | Low address/data bus for external memory |
| ==P1== | Internal | Plain I/O only |
| ==P2== | Internal | High address byte for external memory |
| ==P3== | Internal | RXD, TXD, INT0, INT1, T0, T1, WR, RD |

When a P3 pin is used for its special function (for example P3.0 as RXD), it is not free for general I/O.

## Try it

Change the program so that each LED is **on when its switch is open** and off when pressed.

> [!answer]- Answer
> Invert the input before writing it: `P1 = ~P2;` and `LED = ~SW;` (or `LED = !SW;`). Nothing in the circuit changes.

## Viva questions

1. **How do you make an 8051 port pin an input?** Write 1 to it first, then read it.
2. **Why does Port 0 need external pull-up resistors?** Its pins are open-drain; they can pull low but cannot drive high on their own.
3. **What is the value of all ports after reset?** FFH.
4. **What is `sbit`?** A Keil C51 keyword that names one bit of a bit-addressable SFR, e.g. `sbit LED = P1^0;`.
5. **Which ports are bit-addressable?** All four, P0 to P3.
6. **Why is a pressed switch read as 0?** It connects the pin to ground, overriding the internal pull-up.
7. **Name two alternate functions of Port 3.** P3.0 RXD and P3.1 TXD (serial); also INT0, INT1, T0, T1, WR, RD.
8. **What is the difference between byte I/O and bit I/O?** Byte I/O reads or writes all 8 pins of a port; bit I/O reads or writes one pin.

## Sources

- Muhammad Ali Mazidi and Janice Mazidi, *The 8051 Microcontroller and Embedded Systems*, Pearson, Ch. 4 (I/O port programming) and Ch. 7 (bit-wise I/O in C) (verify).
- Microchip (Atmel), *AT89C51* datasheet, port pin descriptions (internal pull-ups on P1–P3; Port 0 open drain).
