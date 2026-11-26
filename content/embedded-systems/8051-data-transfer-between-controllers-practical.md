---
id: 8051-data-transfer-between-controllers-practical
title: Data Transfer Between Two 8051 Microcontrollers Using I/O Ports
subject: embedded-systems
unit: 1
type: practical
practical: 6
syllabus: [UITIV-006, UDSIV-006]
status: draft
sources:
  - "Muhammad Ali Mazidi and Janice Mazidi, The 8051 Microcontroller and Embedded Systems, Pearson, Ch. 4 and Ch. 7 (verify)"
  - "Labcenter Electronics, Proteus help: multiple microcontrollers in one design (verify)"
---

## Aim

To demonstrate parallel data transfer between two 8051 microcontrollers using general-purpose I/O ports, with a strobe (handshake) signal, where the sender transmits a count and the receiver displays each received byte on LEDs.

## Concepts used

- [[8051-gpio-ports-practical]] (input and output pins)
- [[communication-interfaces]] (parallel communication)

## Before the lab

- **Install:** Keil µVision 5 (C51) and Proteus 8.
- **Revise first:** practical 5 ([[8051-gpio-ports-practical]]): input pins must be written with 1 first; and parallel communication in [[communication-interfaces]].
- **Know this idea:** a **strobe** is a "data ready" signal on its own wire (Explanation below).
- **Have ready:** two folders, `sender` and `receiver`, one Keil project in each.

## Setup

### Tools

- **Keil µVision 5 (C51)**: two projects, `sender` and `receiver`, each with its own HEX file.
- **Proteus**: two AT89C51 chips in one design; load `sender.hex` into the first and `receiver.hex` into the second.

### Connections

| Sender (MCU-1) | Receiver (MCU-2) | Purpose |
|---|---|---|
| P1.0 … P1.7 | P1.0 … P1.7 | 8 data lines |
| P3.2 | P3.2 | Strobe: low means "data is ready" |
| GND | GND | Common ground |
| | P2.0 … P2.7 → 330 Ω → LEDs → +5 V | Show the received byte (active-low) |

On real boards, the two boards **must** share ground, or the receiver cannot read the sender's levels correctly.

## Steps

1. Create Keil project `sender` (as in practical 1 steps 1–6), add the **Sender** code and press **F7**.
   *You should see:* `"sender" - 0 Error(s), 0 Warning(s).` and `sender.hex` created.
2. Create project `receiver` the same way with the **Receiver** code, and build.
   *You should see:* `"receiver" - 0 Error(s), 0 Warning(s).` and `receiver.hex` created.
3. In Proteus, place **two** AT89C51 chips, eight LED-RED and eight 330 Ω resistors.
   *You should see:* U1 and U2 on the sheet.
4. Wire U1's P1.0–P1.7 to U2's P1.0–P1.7, and U1's P3.2 to U2's P3.2. (Tip: instead of long wires, put the same **net label**, such as D0 … D7 and STB, on both ends.)
   *You should see:* nine connections between the chips.
5. Wire the eight LEDs active-low on U2's P2.0–P2.7 (+5 V → 330 Ω → LED → pin).
   *You should see:* LEDs only on the receiver.
6. Double-click U1 and set **Program File** to `sender.hex`; double-click U2 and set it to `receiver.hex`. Set both clocks to 11.0592 MHz.
   *You should see:* each chip holding a different program.
7. Click **Run**.
   *You should see:* the receiver's LEDs count up in binary, one step about every half second.
8. Stop the simulation, right-click the strobe wire (P3.2 to P3.2), choose **Delete Wire**, then click **Run** again.
   *You should see:* the LEDs freeze: without the strobe the receiver never reads. Put the wire back.

## Code

**Sender (MCU-1)**

```c
#include <reg51.h>

sbit STB = P3^2;                  // strobe line to the receiver

void delay_ms(unsigned int ms)    // about 1 ms per count in Keil (see practical 3)
{
    unsigned int i, j;
    for (i = 0; i < ms; i++)
        for (j = 0; j < 1275; j++);
}

void main(void)
{
    unsigned char count = 0;
    STB = 1;                      // strobe idle (high)

    while (1)
    {
        P1 = count;               // 1. put the byte on the 8 data lines
        STB = 0;                  // 2. strobe low: "data ready"
        delay_ms(1);              //    hold it long enough for the receiver
        STB = 1;                  // 3. strobe back high
        count++;                  // next byte
        delay_ms(500);            // send one byte every half second
    }
}
```

**Receiver (MCU-2)**

```c
#include <reg51.h>

sbit STB = P3^2;                  // strobe from the sender

void main(void)
{
    P1 = 0xFF;                    // Port 1 as input (data lines)
    STB = 1;                      // P3.2 as input

    while (1)
    {
        while (STB == 1);         // 1. wait for the strobe to go low
        P2 = ~P1;                 // 2. read the byte, show it on active-low LEDs
        while (STB == 0);         // 3. wait for the strobe to end
    }
}
```

## Output

```output
Sender:   P1 = 00 (strobe pulse) -> 01 (strobe pulse) -> 02 (strobe pulse) -> ...
Receiver: data 05H + strobe        -> P2 = FAH  (LEDs P2.0 and P2.2 ON: binary 0000 0101)
          data changes to 06H,
          strobe stays high        -> P2 stays FAH (no new strobe, so no new read)
          strobe pulse             -> P2 = F9H  (LEDs P2.1 and P2.2 ON: binary 0000 0110)
```

The receiver's LEDs count up in binary, one step every half second, in step with the sender.

<!-- For Prathamesh: both programs verified separately with SDCC + s51: the sender's P1 values and strobe
     pulses were traced, and the receiver was driven with data and strobe on its pins (05H, 06H) to check
     that it reads only on a strobe. Two-chip timing in Proteus not tested here. Add a Proteus screenshot. -->

## Explanation

### What is parallel data transfer?

==Parallel transfer sends all 8 bits of a byte at the same time, one bit on each of 8 wires.==

- ==It is fast and simple==: one byte per transfer, no shifting bit by bit.
- ==It needs many pins and wires==: 8 data lines plus control lines.
- ==It suits short distances==, such as two chips on one board.

### Why is a strobe (handshake) needed?

Without it, the receiver cannot tell **when** the data lines hold a new, stable byte.

- ==**Data first, then strobe:**== the sender sets all 8 lines, then pulls the strobe low.
- ==**Read on strobe:**== the receiver reads only when the strobe is low, so it never catches half-changed data.
- ==**Wait for the end:**== the receiver waits for the strobe to return high, so it reads each byte exactly once.

This is a simple **one-way handshake**. A full handshake adds an **acknowledge** line from the receiver back to the sender, so a slow receiver can never miss a byte.

### Why does the receiver use `~P1`?

The receiver's LEDs are active-low: a 0 turns an LED on. Inverting the byte with `~` makes each **1** bit in the data light its LED, so the LEDs show the count directly in binary.

### How could this be improved?

- ==**Interrupt instead of polling:**== P3.2 is also INT0, so the strobe could trigger an interrupt and the receiver would be free to do other work.
- ==**Acknowledge line:**== the receiver answers on another pin after reading, for reliable transfer.
- ==**Serial transfer:**== the 8051's UART (practical 11) needs only 2 wires instead of 9.

## Common errors and fixes

| Message or symptom | Cause | Fix |
|---|---|---|
| Receiver LEDs never change | The strobe line is missing, or goes to a different pin on each chip | Connect P3.2 of U1 to P3.2 of U2 |
| Both chips behave the same | The same HEX file was loaded into both | U1 gets `sender.hex`, U2 gets `receiver.hex` |
| LEDs show the opposite pattern (0 shows all on) | `~` was left out in the receiver, or LEDs wired active-high | Use `P2 = ~P1;` with active-low LEDs |
| Receiver shows wrong or jumping values on a real board | No common ground between the two boards | Connect GND of both boards together |
| Receiver skips counts | Sender's strobe pulse is too short for the receiver to see | Keep `delay_ms(1)` between `STB = 0;` and `STB = 1;` |
| Net labels do not connect | Labels differ in spelling or case (`D0` vs `d0`) | Use exactly the same label text on both ends |

## Try it

Make the sender send only even numbers (0, 2, 4 …). Which one line changes? What pattern do the receiver's LEDs show?

> [!answer]- Answer
> Replace `count++;` with `count = count + 2;`. Bit 0 never becomes 1, so the LED on P2.0 never lights; the other LEDs count up in binary.

## Viva questions

1. **What is parallel data transfer?** Sending all bits of a byte at the same time on separate wires.
2. **Why is a strobe signal used?** To tell the receiver when the data lines hold a valid byte.
3. **Why must the two boards share a common ground?** Voltage levels are measured against ground; without a common reference the receiver reads wrong levels.
4. **What is handshaking?** An exchange of control signals (strobe, acknowledge) to coordinate data transfer.
5. **Why does the receiver write 0FFH to Port 1 first?** To make its Port 1 pins inputs.
6. **What is polling?** Repeatedly checking a pin or flag in a loop until it changes.
7. **How could the strobe be handled by an interrupt?** Connect it to INT0 (P3.2) and read the data in the INT0 service routine.
8. **Compare parallel and serial transfer between two controllers.** Parallel is faster but needs about 9–10 wires; serial (UART) needs only TX, RX and ground.

## Sources

- Muhammad Ali Mazidi and Janice Mazidi, *The 8051 Microcontroller and Embedded Systems*, Pearson, Ch. 4 (I/O ports) and Ch. 7 (I/O in C) (verify).
- Labcenter Electronics, *Proteus* help on simulating multiple microcontrollers (verify).
