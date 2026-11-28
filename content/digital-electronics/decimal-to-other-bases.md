---
id: decimal-to-other-bases
title: Decimal to Binary, Octal and Hexadecimal
subject: digital-electronics
unit: 1
type: theory
syllabus: [UITII-004, UDSII-005]
status: draft
sources:
  - "M. Morris Mano, Digital Logic and Computer Design, Ch. 1 (verify)"
  - "Floyd, Digital Fundamentals, Ch. 2 (verify)"
  - "Python documentation, Floating Point Arithmetic: Issues and Limitations (docs.python.org/3/tutorial/floatingpoint.html)"
---

## Why should you care?

Your home Wi-Fi router has the address `192.168.1.1`. You type decimal numbers, but the router stores each part in binary: 192 becomes `11000000`. Subnet masks, firewall rules and network classes all make sense only once you can see that binary.

Now open Python and type `0.1 + 0.2`. The answer is `0.30000000000000004`, not `0.3`. The reason is in this lecture: some simple decimal fractions **cannot be written exactly in binary**, so the computer stores a close approximation.

In the last lecture you went from any base **to** decimal. Today you go the other way: from decimal **to** binary, octal and hexadecimal.

## In one line

To convert a decimal integer to base r, divide repeatedly by r and read the remainders from bottom to top; to convert a decimal fraction, multiply repeatedly by r and read the integer parts from top to bottom.

## Explained

### Q1. How do you convert a decimal integer to another base? ⭐

==Use the repeated division method: divide by the new base again and again, and collect the remainders.==

**Steps:**
1. ==Divide the number by the base r== (2, 8 or 16).
2. ==Write down the remainder;== it is always between 0 and r − 1.
3. ==Divide the quotient again by r,== and repeat.
4. ==Stop when the quotient becomes 0.==
5. ==Read the remainders from bottom to top.== The first remainder is the LSB; the last is the MSB.

For hexadecimal, ==write remainders 10–15 as A–F.==

### Q2. Worked example: decimal to binary [2M]

Convert $(25)_{10}$ to binary.

| Division | Quotient | Remainder |
|---|---|---|
| 25 ÷ 2 | 12 | 1 (LSB) |
| 12 ÷ 2 | 6 | 0 |
| 6 ÷ 2 | 3 | 0 |
| 3 ÷ 2 | 1 | 1 |
| 1 ÷ 2 | 0 | 1 (MSB) |

Reading remainders bottom to top: $(25)_{10} = (11001)_2$

**Check:** $16 + 8 + 0 + 0 + 1 = 25$. Correct.

The router example: dividing 192 by 2 repeatedly gives remainders 0, 0, 0, 0, 0, 0, 1, 1, so $(192)_{10} = (11000000)_2$.

### Q3. Worked example: decimal to octal [2M]

Convert $(175)_{10}$ to octal.

| Division | Quotient | Remainder |
|---|---|---|
| 175 ÷ 8 | 21 | 7 |
| 21 ÷ 8 | 2 | 5 |
| 2 ÷ 8 | 0 | 2 |

Bottom to top: $(175)_{10} = (257)_8$

**Check:** $2 \times 64 + 5 \times 8 + 7 = 128 + 40 + 7 = 175$. Correct. (This is the same number we converted the other way in the last lecture.)

### Q4. Worked example: decimal to hexadecimal [2M]

Convert $(687)_{10}$ to hexadecimal.

| Division | Quotient | Remainder |
|---|---|---|
| 687 ÷ 16 | 42 | 15 → F |
| 42 ÷ 16 | 2 | 10 → A |
| 2 ÷ 16 | 0 | 2 |

Bottom to top: $(687)_{10} = (2AF)_{16}$

**Check:** $2 \times 256 + 10 \times 16 + 15 = 512 + 160 + 15 = 687$. Correct.

**Tip for the division:** $687 \div 16$: $16 \times 42 = 672$, and $687 - 672 = 15$.

### Q5. How do you convert a decimal fraction to another base? ⭐

==Use the repeated multiplication method: multiply the fraction by the new base, and collect the integer parts.==

**Steps:**
1. ==Multiply the fraction by the base r.==
2. ==Write down the integer part== (0 to r − 1) of the result.
3. ==Keep only the fraction part== and multiply it again.
4. ==Stop when the fraction becomes 0,== or when you have enough digits (usually 4–5).
5. ==Read the integer parts from top to bottom.== The first one goes right after the point.

Some fractions never reach 0. Then the answer is an **approximation**, and you write how many places you kept.

### Q6. Worked example: fraction to binary

**(a) Convert $(0.625)_{10}$ to binary.**

| Multiplication | Result | Integer part |
|---|---|---|
| 0.625 × 2 | 1.25 | 1 (first) |
| 0.25 × 2 | 0.5 | 0 |
| 0.5 × 2 | 1.0 | 1 (last) |

Fraction is now 0, so stop. Top to bottom: $(0.625)_{10} = (0.101)_2$

**(b) Convert $(0.3)_{10}$ to binary (5 places).**

| Multiplication | Result | Integer part |
|---|---|---|
| 0.3 × 2 | 0.6 | 0 |
| 0.6 × 2 | 1.2 | 1 |
| 0.2 × 2 | 0.4 | 0 |
| 0.4 × 2 | 0.8 | 0 |
| 0.8 × 2 | 1.6 | 1 |

$(0.3)_{10} \approx (0.01001)_2$. The pattern 1001 repeats forever, because 0.6 appears again. This is why `0.1 + 0.2` is not exactly 0.3 in Python.

### Q7. Worked example: a mixed number to all three bases

==Convert the integer and fraction parts separately, then join them with the point.==

Convert $(10.5)_{10}$:

| Target | Integer part 10 | Fraction part 0.5 | Answer |
|---|---|---|---|
| Binary | 10 → 1010 | 0.5 × 2 = **1**.0 → .1 | $(1010.1)_2$ |
| Octal | 10 → 12 | 0.5 × 8 = **4**.0 → .4 | $(12.4)_8$ |
| Hex | 10 → A | 0.5 × 16 = **8**.0 → .8 | $(A.8)_{16}$ |

And $(25.625)_{10} = (11001.101)_2$, using the answers of Q2 and Q6(a).

### Q8. Is there a faster way to convert small numbers to binary? ⭐

==Yes: the subtraction (powers of 2) method.== It is quicker in your head for numbers up to about 255.

1. ==List the powers of 2:== 128, 64, 32, 16, 8, 4, 2, 1.
2. ==Take the largest power that fits,== subtract it, and put a 1 in its place.
3. ==Repeat with the remainder;== put 0 under every power you skip.

**Example:** $(45)_{10}$. $45 = 32 + 8 + 4 + 1$.

| 32 | 16 | 8 | 4 | 2 | 1 |
|---|---|---|---|---|---|
| 1 | 0 | 1 | 1 | 0 | 1 |

$(45)_{10} = (101101)_2$

### Q9. Convert $(156)_{10}$ to binary, octal and hexadecimal. [2M]

| Base | Repeated division (quotient, remainder) | Read bottom to top |
|---|---|---|
| 2 | 78 r0, 39 r0, 19 r1, 9 r1, 4 r1, 2 r0, 1 r0, 0 r1 | $(10011100)_2$ |
| 8 | 19 r4, 2 r3, 0 r2 | $(234)_8$ |
| 16 | 9 r12 (C), 0 r9 | $(9C)_{16}$ |

**Check:** $9 \times 16 + 12 = 156$. Correct.

### Q10. Convert $(0.8125)_{10}$ to binary, octal and hexadecimal. [2M]

==Multiply the fraction by the base repeatedly; read integer parts top to bottom.==

| Base | Multiplications | Answer |
|---|---|---|
| 2 | 0.8125×2 = **1**.625; 0.625×2 = **1**.25; 0.25×2 = **0**.5; 0.5×2 = **1**.0 | $(0.1101)_2$ |
| 8 | 0.8125×8 = **6**.5; 0.5×8 = **4**.0 | $(0.64)_8$ |
| 16 | 0.8125×16 = **13**.0 → D | $(0.D)_{16}$ |

### Q11. Why can $(0.1)_{10}$ not be stored exactly in binary? [2M]

- ==**Repeated multiplication never reaches 0:**== 0.1 → 0.2 → 0.4 → 0.8 → 1.6 → 1.2 → 0.4 … the pattern repeats.
- ==**So the binary fraction is endless:**== $(0.1)_{10} = (0.000110011\ldots)_2$.
- ==**A computer keeps a fixed number of bits,**== so it stores a value very close to 0.1, not 0.1 itself.
- This is why `0.1 + 0.2` gives `0.30000000000000004` in Python.

### Q12. Convert $(43.375)_{10}$ to binary. ⭐

==Convert the integer and fraction parts separately, then join them.==

**Integer part 43** (divide by 2):

| 43÷2 | 21÷2 | 10÷2 | 5÷2 | 2÷2 | 1÷2 |
|---|---|---|---|---|---|
| r1 | r1 | r0 | r1 | r0 | r1 |

Bottom to top: 101011.

**Fraction part 0.375** (multiply by 2): 0.375×2 = **0**.75; 0.75×2 = **1**.5; 0.5×2 = **1**.0 → .011

$$(43.375)_{10} = (101011.011)_2$$

**Check:** $32 + 8 + 2 + 1 = 43$ and $0.25 + 0.125 = 0.375$. Correct.

### Q13. Convert $(250.75)_{10}$ to binary, octal and hexadecimal.

**Integer part 250:**
- Binary: remainders 0, 1, 0, 1, 1, 1, 1, 1 (LSB first) → 11111010
- Octal: 250÷8 = 31 r2, 31÷8 = 3 r7, 3÷8 = 0 r3 → 372
- Hex: 250÷16 = 15 r10 (A), 15÷16 = 0 r15 (F) → FA

**Fraction part 0.75:**
- ×2: **1**.5, **1**.0 → .11
- ×8: **6**.0 → .6
- ×16: **12**.0 → .C

$$(250.75)_{10} = (11111010.11)_2 = (372.6)_8 = (FA.C)_{16}$$

## Analogy and real-life example

### Analogy: making change

You must pay ₹687 using only ₹256, ₹16 and ₹1 coins, using the biggest coins first. Two ₹256 coins (512) leave 175. Ten ₹16 coins (160) leave 15. Fifteen ₹1 coins finish it. The counts **2, 10, 15** are the hex digits **2, A, F**. Repeated division is just a careful way of counting coins of each size.

### Real system: floating-point numbers in Python and NumPy

A `float` in Python stores numbers in binary. Since 0.1 and 0.2 become endless binary fractions (like 0.3 in Q6), they are cut off after 52 binary places. The tiny error shows up in `0.1 + 0.2`. Data scientists meet this every day: never test floats with `==`; use `math.isclose()` or `numpy.isclose()` instead. Banking software avoids the problem by storing money in decimal formats (Python's `decimal` module) or in whole paise.

## Diagram

```mermaid
flowchart TB
  accTitle: Decimal to other bases
  accDescr: A decimal number is split into an integer part and a fraction part. The integer part is divided repeatedly by the base and the remainders are read bottom to top. The fraction part is multiplied repeatedly by the base and the integer parts are read top to bottom. The two results are joined with a point.
  N[Decimal number] --> I[Integer part]
  N --> F[Fraction part]
  I --> D["Divide by r repeatedly; read remainders bottom to top"]
  F --> M["Multiply by r repeatedly; read integer parts top to bottom"]
  D --> R[Join with the point]
  M --> R
```

## Key points

> [!important] Key points
> - Integer part: divide by the base repeatedly; remainders read bottom to top (first remainder = LSB).
> - Fraction part: multiply by the base repeatedly; integer parts read top to bottom (first = just after the point).
> - Convert integer and fraction parts separately, then join them.
> - In hexadecimal, remainders 10–15 are written A–F.
> - Some decimal fractions (e.g. 0.1, 0.3) give endless binary fractions; keep a fixed number of places.
> - Shortcut for binary: break the number into powers of 2 (45 = 32 + 8 + 4 + 1 = 101101).
> - Always check by converting the answer back to decimal.

## Common mistakes

> [!warning] Common mistake
> **Reading the remainders top to bottom.** For the integer part, the first remainder is the LSB, so the remainders are read from the **bottom** up. Reading them downwards gives $(10011)_2$ for 25, which is 19.

> [!warning] Common mistake
> **Reading the fraction digits bottom to top.** For the fraction part it is the opposite: the **first** integer part goes right after the point.

> [!warning] Common mistake
> **Writing "15" as a hex digit.** A remainder of 15 is one hex digit, F. Writing it as "15" gives two digits and a wrong number.

## Sources

- M. Morris Mano, *Digital Logic and Computer Design*, Prentice Hall, Ch. 1, conversion of decimal numbers (verify).
- Floyd, *Digital Fundamentals*, Pearson, Ch. 2, decimal-to-binary conversion (verify).
- Python documentation, *Floating Point Arithmetic: Issues and Limitations* (docs.python.org/3/tutorial/floatingpoint.html).
- University syllabus, Sem 2 Digital Electronics: BSc IT UITII-004 and BSc DS UDSII-005 (same units), Unit 1 (BOS 05/06/2026).
