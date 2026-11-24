---
id: embedded-systems
title: Embedded System
type: index
class: Sem 4 BSc IT and BSc DS
codes: "IT: UITIV-005 (theory), UITIV-006 (practical); DS: UDSIV-005 (theory), UDSIV-006 (practical)"
---

Topic list in lecture order, mapped from the university syllabus (BOS 05/06/2026).
IT and DS share this syllabus, so one set of notes serves both classes.
Faded links are not written yet.

**Reference books (from syllabus):** Raj Kamal, *Embedded Systems: Architecture, Programming and Design*; Mazidi & Mazidi, *The 8051 Microcontroller and Embedded Systems*; Deshmukh, *Microcontrollers: Theory and Applications*; Vahid & Givargis, *Embedded Systems Design*.
**Additional:** NPTEL Embedded Systems, 8051 datasheets, Keil µVision, Proteus.

## Start here

0. [[embedded-systems-intro]]

## Theory

### Module 1 · Embedded Systems Fundamentals and Hardware

**Unit 1 · Introduction to Embedded Systems and Hardware (8 h)**
1. [[embedded-vs-general-purpose-systems]]
2. [[embedded-systems-history-and-classification]]
3. [[embedded-systems-applications-and-purpose]]
4. [[microprocessor-vs-microcontroller]]
5. [[risc-vs-cisc]]
6. [[endianness]]: big endian vs little endian
7. [[asic-pld-cots]]
8. [[sensors-and-actuators]]
9. [[communication-interfaces]]
10. [[embedded-firmware]]
11. [[embedded-system-characteristics]]
12. [[quality-attributes]]: operational and non-operational

**Unit 2 · Embedded Hardware and Peripherals (7 h)**
1. [[application-specific-embedded-system]]: e.g. washing machine
2. [[domain-specific-embedded-system]]: e.g. automotive
3. [[memory-io-interrupt-map]]
4. [[processor-family-and-external-peripherals]]
5. [[memory-types]]: RAM, ROM, flash
6. [[memory-testing-and-crc]]
7. [[control-and-status-registers]]
8. [[device-drivers]]: incl. timer drivers
9. [[watchdog-timer]]

### Module 2 · Microcontroller Programming and System Design

**Unit 3 · 8051 Microcontroller and Programming (8 h)**
1. [[8051-family-overview]]
2. [[8051-architecture]]: hardware components
3. [[8051-pins-and-ports]]: I/O pins, ports and circuits
4. [[8051-external-memory]]
5. [[8051-c-data-types]]
6. [[8051-time-delay-programs]]
7. [[8051-io-programming]]
8. [[8051-logic-operations]]
9. [[8051-data-conversion]]

**Unit 4 · RTOS and Embedded System Design (7 h)**
1. [[8051-system-design]]: selection, design considerations, integration
2. [[embedded-program-structure]]: structure, infinite loop
3. [[compiling-linking-debugging]]
4. [[os-basics-and-types]]
5. [[rtos-characteristics-and-selection]]
6. [[embedded-development-environment]]: IDE, cross compilation, file types
7. [[simulators-emulators-debuggers]]
8. [[embedded-product-development-life-cycle]]
9. [[embedded-industry-trends]]: IoT, edge computing

## Practicals

Tools: Keil µVision (C51) for every practical; Proteus whenever a circuit is simulated (from practical 2).

**Unit 1 · 8051 Programming and Basic Interfacing**
1. [[8051-programming-environment-practical]]: architecture study; write, compile, execute
2. [[8051-reprogrammable-system-practical]]
3. [[8051-debugging-practical]]
4. [[8051-timer-registers-practical]]: configure TMOD/TCON, generate a delay
5. [[8051-gpio-ports-practical]]
6. [[8051-data-transfer-between-controllers-practical]]

**Unit 2 · Timing and Communication**
7. [[led-interfacing-practical]]
8. [[led-binary-counter-practical]]
9. [[led-patterns-practical]]: sequential, alternate, LED control applications
10. [[timer-led-blinking-practical]]: incl. timer delay vs software loop delay

**Unit 3 · Peripheral Interfacing and Signal Generation**
11. [[serial-communication-practical]]: configure UART, exchange data with a PC
12. [[seven-segment-display-practical]]: incl. 0–99 counter
13. [[dac-square-wave-practical]]: DAC interfacing, square wave
14. [[dac-waveforms-practical]]: triangular and sine using lookup tables

**Unit 4 · Embedded Applications**
15. [[stepper-motor-practical]]: clockwise and anticlockwise
16. [[traffic-signal-controller-practical]]
17. [[temperature-monitoring-practical]]
18. [[elevator-control-practical]]
19. [[proteus-keil-simulation-practical]]
