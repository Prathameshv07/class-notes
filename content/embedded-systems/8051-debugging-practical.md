---
id: 8051-debugging-practical
title: Debugging and Testing 8051 Programs in Keil
subject: embedded-systems
unit: 1
type: practical
practical: 3
syllabus: [UITIV-006, UDSIV-006]
status: draft
sources:
  - "Arm Keil, µVision User's Guide: debugging, breakpoints, Watch and Memory windows, Performance Analyzer (keil.com)"
  - "Muhammad Ali Mazidi and Janice Mazidi, The 8051 Microcontroller and Embedded Systems, Pearson, Ch. 7 (verify)"
---

## Aim

To debug and test an 8051 program in Keil µVision using breakpoints, single-stepping and the Watch window, by finding and fixing an overflow bug in a program that averages five students' marks, and to measure a software delay with the simulator's stopwatch.

## Concepts used

- [[8051-programming-environment-practical]] (creating and building a Keil project)
- [[embedded-firmware]]

## Setup

### Tool

**Keil µVision 5 (C51)** with its built-in simulator. No board or Proteus is needed.

### Debug tools you will use

| Tool | Where | What it does |
|---|---|---|
| ==Breakpoint== | Click the grey margin beside a line (red dot) | Program stops before that line runs |
| ==Step (F11) / Step Over (F10)== | Debug menu | Run one line at a time; Step Over runs a whole function as one step |
| ==Run (F5) / Run to Cursor (Ctrl+F10)== | Debug menu | Run until a breakpoint or the cursor line |
| ==Watch window== | View → Watch Windows → Watch 1 | Shows chosen variables live |
| ==Registers window== | Left pane in debug mode | Shows R0–R7, A, PSW, and the **sec** (time) counter |
| ==Memory window== | View → Memory Windows | Shows raw memory, e.g. `D:0x30` for internal RAM |

### Steps

1. Create project `marks_debug` (device AT89C51, Xtal 11.0592 MHz), add `main.c` with Program 1, build.
2. **Ctrl+F5** to start debugging. Add `sum`, `i` and `avg` to Watch 1 (right-click a variable → Add to Watch 1).
3. Put a breakpoint on the line `avg = sum / 5;`. Press **F5**. Read `sum` in the Watch window.
4. Step through the loop with **F11** and watch `sum` after each mark is added. Find where it goes wrong.
5. Fix the bug (Program 2), rebuild, debug again and compare.
6. **Part B:** use the **sec** counter to time `delay_ms(1)` (see Explanation).

## Code

**Program 1: with a bug**

```c
#include <reg51.h>

unsigned char marks[5] = {85, 90, 78, 92, 88};   // marks of 5 students

void main(void)
{
    unsigned char sum = 0;      // BUG: 8 bits can hold only 0 to 255
    unsigned char i, avg;

    for (i = 0; i < 5; i++)
        sum = sum + marks[i];   // total should be 433

    avg = sum / 5;              // average should be 86
    P1 = avg;                   // show the average on Port 1
    while (1);
}
```

**Program 2: fixed**

```c
#include <reg51.h>

unsigned char marks[5] = {85, 90, 78, 92, 88};

void main(void)
{
    unsigned int sum = 0;       // FIX: 16 bits hold 0 to 65535
    unsigned char i, avg;

    for (i = 0; i < 5; i++)
        sum = sum + marks[i];

    avg = sum / 5;
    P1 = avg;
    while (1);
}
```

## Output

```output
                     Program 1 (bug)          Program 2 (fixed)
Watch: sum           177  (0xB1)              433  (0x01B1)
Watch: avg           35   (0x23)              86   (0x56)
Port 1 (P1)          0x23                     0x56
```

Watching `sum` step by step in Program 1: 85 → 175 → 253 → **89** → 177. It "falls" from 253 to 89 when 92 is added, because 253 + 92 = 345, and 345 − 256 = 89.

<!-- For Prathamesh: final values verified by compiling both programs with SDCC and running them in the s51
     simulator (P1 = 0x23 with the bug, 0x56 when fixed). The step-by-step sum values are arithmetic.
     Part B's time depends on the compiler, so students record their own Keil reading in the journal. -->

## Explanation

### What is debugging, and why use a simulator?

==Debugging is finding and fixing the cause of wrong behaviour in a program.==

- ==A program can build with 0 errors and still be wrong==: the compiler checks grammar, not logic.
- ==The simulator lets you pause the program== and look inside: variables, registers, memory.
- ==Breakpoints stop the program at a chosen line==; single-stepping runs one line at a time.
- ==The Watch window shows variables changing== as you step.

On a real board the same idea needs a hardware debugger or emulator; on the simulator it is free.

### What was the bug?

- `sum` was an `unsigned char`, which holds only 0 to 255.
- The true total is 433, so the value **overflowed**: it wrapped round past 255 and kept only the remainder (433 − 256 = 177).
- The average became 177 / 5 = 35 instead of 86.
- The fix was to make `sum` an `unsigned int` (0 to 65535). `avg` can stay `unsigned char`, because an average of marks never exceeds 100.

Stepping showed exactly **where** it broke: the fourth addition, 253 + 92.

### Part B: how do you measure a delay with the stopwatch?

1. Add a `delay_ms` function (from practical 2) and a call `delay_ms(1);` in `main`.
2. Put breakpoints on the call line and on the line after it.
3. Run to the first breakpoint. Note the **sec** value in the Registers window (it shows simulated time).
4. Run (F5) to the second breakpoint. Note **sec** again.
5. **Time of `delay_ms(1)` = second reading − first reading.**

Record the result in your journal. It shows whether the loop count 1275 really gives about 1 ms in Keil. A different compiler, or different optimisation settings, gives a different time for the same C code; practical 10 shows why a hardware timer avoids this problem.

### What is a good debugging method?

1. ==**Reproduce:**== make the wrong result appear every time.
2. ==**Predict:**== work out what each variable should be (sum should be 433).
3. ==**Observe:**== step and watch until a value differs from the prediction.
4. ==**Fix the cause**==, not the symptom (change the data type, don't add 256 at the end).
5. ==**Re-test**== with the same inputs, and with extreme ones (all marks 100).

## Try it

Change all five marks to 100. Before running, predict `sum` and `avg` for Program 2. Then check in the Watch window. What would Program 1 have shown?

> [!answer]- Answer
> Program 2: sum = 500, avg = 100. Program 1: 500 − 256 = 244, so avg = 244 / 5 = 48 (integer division). Testing with extreme values is how such bugs are caught before students' results are wrong.

## Viva questions

1. **What is the difference between a syntax error and a logic error?** A syntax error stops the build; a logic error builds fine but gives a wrong result.
2. **What is a breakpoint?** A marked line where the debugger pauses the program before running it.
3. **What is the difference between Step (F11) and Step Over (F10)?** Step enters a called function line by line; Step Over runs the whole function as one step.
4. **What is the Watch window used for?** To see chosen variables' values while the program is paused or stepping.
5. **What is overflow?** When a result is too large for its data type, so the extra part is lost and the value wraps round.
6. **What is the range of `unsigned char` and `unsigned int` in Keil C51?** 0 to 255, and 0 to 65535.
7. **How do you measure execution time in the Keil simulator?** Read the sec counter in the Registers window at two breakpoints and subtract.
8. **Why is a simulator useful before testing on hardware?** It shows the inside of the program without any wiring, and bugs found there cost nothing.

## Sources

- Arm Keil, *µVision User's Guide*: debugging, breakpoints, Watch and Memory windows (keil.com).
- Muhammad Ali Mazidi and Janice Mazidi, *The 8051 Microcontroller and Embedded Systems*, Pearson, Ch. 7 (data types and time delays in 8051 C) (verify).
