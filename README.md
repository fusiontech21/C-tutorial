# C Programming — The Complete Guide (Actually Explained)

> *not just syntax. why it works, what's happening underneath, and what breaks when you get it wrong. no skipping.*

---

## Table of Contents

1. [Why C](#why-c)
2. [How C Code Becomes a Program](#how-c-becomes-a-program)
3. [Setting Up](#setting-up)
4. [Your First Program — every line](#your-first-program)
5. [Variables and Data Types — the full picture](#variables-and-data-types)
6. [Operators — all of them](#operators)
7. [Control Flow](#control-flow)
8. [Functions — how they actually work](#functions)
9. [Arrays — memory in a row](#arrays)
10. [Strings — just arrays with a zero at the end](#strings)
11. [Pointers — the real thing](#pointers)
12. [Memory Management — the heap](#memory-management)
13. [Structs, Unions, and Enums](#structs-unions-and-enums)
14. [File I/O](#file-io)
15. [The Preprocessor](#the-preprocessor)
16. [Header Files and Multi-File Projects](#header-files-and-multi-file-projects)
17. [Modern C — C99 through C23](#modern-c)
18. [Undefined Behavior — the scariest part](#undefined-behavior)
19. [Error Handling](#error-handling)
20. [Bit Manipulation Deep Dive](#bit-manipulation)
21. [Common Pitfalls](#common-pitfalls)
22. [Debugging](#debugging)
23. [Questions](#questions)
24. [Answers](#answers)

---

## Why C

real talk — why are you learning C when python exists and javascript is everywhere?

here's the honest answer: C is the language that *runs everything else*. python's interpreter is written in C. the linux kernel that powers literally 90% of the internet's servers is C. git, the thing you use to push code? C. sqlite, the most widely deployed database engine in the world? C. every operating system kernel, every browser engine, most compilers, game engines, embedded systems — C.

when you write python or javascript, you're sitting on top of fifteen layers of abstraction that hide what's actually happening. the language manages memory for you, handles types automatically, and abstracts away the hardware entirely. that's convenient, but it means you don't really understand what your code is doing.

when you write C, there's almost nothing between you and the machine. no garbage collector cleaning up behind you. no virtual machine. no runtime deciding what types things are. it's your code, the compiler, and the CPU — and that's it.

what that actually means:
- you learn how memory *actually* works. not just "variables store data" but physically where in RAM, how addressing works, what happens when you read the wrong location
- you understand *why* other languages make certain tradeoffs — garbage collectors exist because manual memory management is hard. reference counting, runtime type systems, bounds checking — all of these are solutions to problems you'll experience firsthand in C
- pointers will humble you for a week and then make everything click
- you can write software that runs on a chip with 2KB of RAM and no OS

yeah it's harder than python. the compiler will reject code you swear should work. segfaults will appear with no explanation. but once C clicks, every other language becomes clearer. let's go.

---

## How C Code Becomes a Program

before writing a single line, it's worth understanding what actually happens when you compile. this is stuff most tutorials skip, but it explains why certain errors show up and when.

your `.c` file goes through four distinct stages:

### Stage 1 — Preprocessing

the preprocessor runs first. it's a text-processing tool that handles every line starting with `#`. it knows nothing about C — it just does text manipulation:

- `#include <stdio.h>` → literally copy-pastes the content of `stdio.h` at that line
- `#define MAX 100` → find-and-replace every occurrence of `MAX` with `100`
- `#ifdef DEBUG ... #endif` → conditionally include or remove sections of code

the output of preprocessing is still C code, just with all the `#` lines resolved. you can see this output with `gcc -E file.c`.

### Stage 2 — Compilation

the compiler takes the preprocessed C code and converts it to assembly language — the human-readable representation of CPU instructions. this is where type checking happens, where syntax errors are caught, where optimizations are applied.

the output is assembly (`.s` files). you can see this with `gcc -S file.c`.

### Stage 3 — Assembly

the assembler converts assembly code to machine code — the raw binary instructions the CPU actually executes. the output is *object files* (`.o` files). each `.c` file produces one `.o` file.

object files contain compiled code but with *unresolved references* — calls to functions defined in other files (like `printf`) have placeholder addresses.

### Stage 4 — Linking

the linker combines all the `.o` files and resolves the unresolved references. it knows where `printf` is (in the C standard library), fills in the addresses, and produces the final executable.

why does this matter? because error messages tell you which stage failed:
- "implicit declaration of function" → compilation stage, you forgot an `#include`
- "undefined reference to function_name" → linking stage, you forgot to compile/link that file
- "segmentation fault" → runtime, nothing the compiler could catch

---

## Setting Up

### The Compiler

the compiler turns C code into machine code. GCC and Clang are the two main options — both are free, both work great, all the code in this guide compiles on either.

**Linux:**
```bash
gcc --version
```
probably already installed. if not: `sudo apt install gcc` (Ubuntu/Debian) or `sudo dnf install gcc` (Fedora).

**Mac:**
```bash
xcode-select --install
```
installs Clang, which on mac pretends to be gcc when you type `gcc`. it's fine, identical for our purposes.

**Windows:** use WSL (Windows Subsystem for Linux). it runs a full Linux environment inside Windows, gives you gcc, everything just works. or MinGW-w64 if you want native Windows gcc.

### The Editor

use **Zed** — zed.dev. it's fast, written in Rust, has solid C support, and feels like someone actually cared about the experience. open a `.c` file and it works.

VSCode with the C/C++ extension from Microsoft is also solid. Vim if you want to feel like a hacker. CLion if you want a full IDE.

### Compiling and Running

```bash
gcc hello.c -o hello      # compile — -o specifies the output filename
./hello                   # run on Linux/Mac
hello.exe                 # run on Windows
```

**always compile with these flags:**
```bash
gcc -Wall -Wextra -std=c17 -g -o hello hello.c
```

what each flag does:
- `-Wall` — enables a large set of warnings. without this, the compiler silently ignores a lot of potential bugs
- `-Wextra` — enables additional warnings beyond -Wall
- `-std=c17` — use the C17 standard (2017). without specifying, you might get old defaults
- `-g` — include debug information so tools like gdb can show you line numbers

during development, also add:
```bash
gcc -Wall -Wextra -std=c17 -g -fsanitize=address,undefined -o hello hello.c
```

`-fsanitize=address,undefined` catches memory bugs and undefined behavior at runtime. it's slow but absolutely invaluable for finding bugs. use it while developing, remove it for production builds.

---

## Your First Program

```c
#include <stdio.h>

int main(void) {
    printf("Hello, world!\n");
    return 0;
}
```

this is five lines but each one is doing something specific worth understanding.

### `#include <stdio.h>`

`stdio.h` stands for "standard input/output". it's a *header file* — a text file containing declarations (descriptions of what functions exist, what arguments they take, what they return).

`printf` is declared in `stdio.h`. without this include, the compiler doesn't know what `printf` is. you'd get something like "implicit declaration of function printf" — the compiler is saying "you're calling something but I have no idea what it looks like".

the angle brackets `<stdio.h>` mean "look for this in the system's include directories". quotes `"myfile.h"` mean "look in the current directory first, then system directories". for standard library headers, always use angle brackets.

### `int main(void)`

`main` is special — it's where program execution begins. every C program has exactly one `main`.

`int` before it means `main` returns an integer. this integer is the *exit code* — the value returned to the operating system when your program finishes. exit code 0 conventionally means success. any non-zero value signals some kind of failure. shell scripts, CI pipelines, and other programs that run yours can check this value.

`void` in the parameter list means "takes no arguments". you can also write `int main(int argc, char *argv[])` to receive command-line arguments — more on that later.

### `printf("Hello, world!\n")`

`printf` = "print formatted". it sends text to stdout (standard output — your terminal by default).

the argument is a *string literal* — a sequence of characters enclosed in double quotes. string literals in C are arrays of characters stored in read-only memory.

`\n` is an *escape sequence* — a way to represent characters that aren't printable or are hard to type. `\n` is newline (ASCII value 10). without it, your terminal prompt appears right after your output on the same line. other common escape sequences: `\t` (tab), `\\` (backslash), `\"` (double quote inside a string), `\0` (null character).

the semicolon `;` ends the statement. C ignores whitespace and newlines — the semicolon is how it knows the statement is done.

### `return 0`

returns 0 to the operating system — signals success. try returning 1 and then running `echo $?` right after — you'll see the exit code.

---

## Variables and Data Types

### What a Variable Actually Is

a variable is a named region of memory. when you write `int x = 5`, you're asking the system to reserve some bytes of memory, name that region `x`, and store the value 5 in those bytes.

every type has a size — a number of bytes it occupies. the size determines: how much memory it takes, what values fit in it, and how the CPU interprets the raw bits.

### The Basic Types

```c
int age = 25;                     // whole number
float temperature = 98.6f;        // decimal, ~7 digits precision
double pi = 3.14159265358979;     // decimal, ~15 digits precision
char letter = 'A';                // single character — actually just a small integer
```

**`int`** — the default integer type. on virtually all modern computers it's 32 bits (4 bytes). can hold values from -2,147,483,648 to 2,147,483,647. the exact size isn't guaranteed by the C standard (it's at least 16 bits), but in practice you can count on 32 bits on any modern system.

**`float` vs `double`** — both store decimal numbers, but they differ in precision. float is 32 bits with about 7 significant decimal digits. double is 64 bits with about 15. always prefer `double` unless you have a specific reason for float (memory constraints, GPU code, etc.). most math functions return double. mixing float and double causes accidental precision loss.

why not just have one decimal type? precision costs memory and computation. for embedded systems or graphics where you're crunching millions of numbers, float's lower precision is an acceptable tradeoff for the lower cost.

**`char`** — nominally a "character type" but it's really just a small integer (8 bits, -128 to 127 for signed, 0-255 for unsigned). it stores the ASCII code of the character. `'A'` is literally the integer 65. `'a'` is 97. `'0'` is 48. `'\n'` is 10. you can do arithmetic on chars and it works exactly like integer arithmetic.

```c
char c = 'A';
c = c + 1;   // c is now 'B' (66) — uppercase letter arithmetic
c = c + 32;  // c is now 'b' — lowercase. difference between cases is exactly 32 in ASCII
```

### Integer Type Variants

```c
short int x = 1000;     // at least 16 bits, usually 16
long int y = 100000L;   // at least 32 bits, often 64 on modern systems
long long z = 1LL << 40; // at least 64 bits, always 64 in practice

unsigned int u = 4294967295U;     // no negative numbers, 0 to ~4.2 billion
unsigned short s = 65535;
unsigned long long big = 18446744073709551615ULL;
```

the `U`, `L`, `LL`, `ULL` suffixes tell the compiler what type to treat the literal as. `1LL << 40` without `LL` would try to left-shift an `int`, which overflows. with `LL`, it's a `long long`, which handles it.

**why unsigned?** when your value can never logically be negative (array indices, sizes, memory addresses), use unsigned. it also doubles the positive range. the tradeoff: unsigned arithmetic wraps around silently. `0u - 1` gives 4,294,967,295 — that can surprise you.

### Exact-Width Types from `<stdint.h>`

the problem with `int` being "at least 16 bits" — you can't write portable code that depends on exact sizes. `<stdint.h>` (added in C99) solves this:

```c
#include <stdint.h>

int8_t   tiny   = 127;                      // exactly 8 bits, signed
uint8_t  ubyte  = 255;                      // exactly 8 bits, unsigned
int16_t  small  = 32767;                    // exactly 16 bits, signed
uint16_t ushort = 65535;                    // exactly 16 bits, unsigned
int32_t  normal = 2147483647;               // exactly 32 bits, signed
uint32_t u32    = 4294967295U;              // exactly 32 bits, unsigned
int64_t  big    = 9223372036854775807LL;    // exactly 64 bits, signed
uint64_t u64    = 18446744073709551615ULL;  // exactly 64 bits, unsigned
```

use these whenever you're doing anything that depends on exact sizes: binary file formats, network protocols, hardware registers, cryptography. for general integer math where you just need "a reasonably sized integer", `int` is fine.

also useful from `<stdint.h>`:
```c
intptr_t  p = (intptr_t)some_pointer;   // integer big enough to hold a pointer
uintptr_t up = (uintptr_t)ptr;          // unsigned version
size_t    s = sizeof(int);              // unsigned type for sizes and indices
ptrdiff_t d = ptr2 - ptr1;             // signed type for pointer differences
```

### `size_t` — the index type

`size_t` is the unsigned integer type that the system uses for object sizes and array indices. it's big enough to hold the size of any object in memory. on a 64-bit system it's 64 bits. on a 32-bit system it's 32 bits.

use `size_t` for:
- loop counters when iterating over arrays
- function return values that represent counts or sizes
- anything that `sizeof` returns (it returns `size_t`)

```c
size_t len = strlen(str);   // strlen returns size_t
for (size_t i = 0; i < len; i++) {
    printf("%c", str[i]);
}
printf("size: %zu\n", len);  // %zu is the format specifier for size_t
```

### How Numbers are Stored — Binary Basics

every value in C is ultimately stored as bits. an `int32_t` with value 5 is stored as:
```
00000000 00000000 00000000 00000101
```

**two's complement** — how negative integers are represented. to negate a number: flip all bits, then add 1.

```
5  = 00000000 00000000 00000000 00000101
    flip bits: 11111111 11111111 11111111 11111010
    add 1:     11111111 11111111 11111111 11111011 = -5
```

why two's complement? because addition works the same way for both positive and negative numbers. `5 + (-5)` in two's complement naturally gives zero with a carry-out that gets discarded. it's elegant hardware-wise.

consequence: `INT_MAX + 1` wraps to `INT_MIN` in two's complement binary. but in C, *signed integer overflow is undefined behavior* — the C standard explicitly doesn't guarantee what happens. in practice it wraps on almost every real system, but you can't rely on it. **never intentionally overflow a signed integer.**

**floating point** — floats and doubles use IEEE 754 format. a 32-bit float has: 1 sign bit, 8 exponent bits, 23 mantissa bits. this is why `0.1 + 0.2 != 0.3` in floating point — these numbers can't be represented exactly in binary, so they're approximated.

```c
printf("%.20f\n", 0.1);   // 0.10000000000000000555... not exactly 0.1
```

never compare floats with `==` directly:
```c
double a = 0.1 + 0.2;
if (a == 0.3) { ... }               // might fail
if (fabs(a - 0.3) < 1e-9) { ... }   // correct — check if they're close enough
```

### Declaration vs Initialization

```c
int x;        // declared — memory reserved, contents are GARBAGE
int y = 10;   // declared AND initialized — safe to use
```

this is critical. unlike python or Java, C does NOT zero-initialize local variables. when you declare `int x;`, the memory for x already existed as part of the stack frame. whatever bytes were there before — from a previous function call, from previous data — are still there. reading an uninitialized variable is undefined behavior. the value could be 0, could be -858993460 (a common "uninitialized" pattern debuggers use), could be anything.

**global and static variables are zero-initialized** — the OS zeroes out the data segment before your program starts. so global `int x;` starts as 0. local `int x;` starts as garbage.

```c
int global_x;   // 0 at program start — zero-initialized

void foo(void) {
    int local_x;        // GARBAGE — uninitialized
    static int static_x; // 0 — static local variables are zero-initialized
}
```

**always initialize local variables.** make it a habit. `int x = 0;` over `int x;`.

### Scope and Lifetime

**scope** — where in the code a variable is visible.

**lifetime** — how long the variable exists in memory.

```c
int global = 10;   // scope: entire file. lifetime: entire program.

void foo(void) {
    int local = 20;  // scope: this function. lifetime: while foo is executing.

    {
        int block = 30;  // scope: this block only. lifetime: while in the block.
        printf("%d %d %d\n", global, local, block);  // all visible
    }

    printf("%d\n", block);  // COMPILE ERROR — block is out of scope
}
```

when a function is called, space for its local variables is allocated on the *call stack*. when the function returns, that stack space is reclaimed — the variables are gone. this is why returning a pointer to a local variable is a serious bug.

### Type Conversions

**implicit conversion** — C automatically converts between types in certain situations:

```c
int x = 5;
double y = x;       // int to double — fine, no data lost

double d = 3.99;
int i = d;          // double to int — TRUNCATES to 3, no warning by default
                    // not rounding — truncation toward zero

unsigned int u = -1;  // -1 converted to unsigned — wraps to UINT_MAX
                      // this is valid/defined but usually a bug
```

**implicit conversion in expressions** — when you use different types in an expression, C promotes the "smaller" type to the "larger" one before the operation:

```c
int a = 5;
double b = 2.0;
double result = a / b;   // a is converted to double before dividing — 2.5

int x = 5, y = 2;
double r = x / y;    // WRONG — both ints, integer division happens first (gives 2),
                     // THEN 2 is converted to 2.0 — too late
double r = (double)x / y;   // correct — cast first, then divide
```

**explicit cast** — force a conversion:
```c
double d = 3.7;
int i = (int)d;     // explicit truncation — i = 3, clearly intentional
printf("%d\n", (int)(3.14 * 100));  // 314
```

**integer promotions** — char and short are automatically promoted to int in expressions. this is why `char + char` actually computes as `int + int`:

```c
char a = 200, b = 100;
int result = a + b;    // both promoted to int before addition
                       // 200 + 100 = 300 — fits in int, no overflow
```

---

## Operators

### Arithmetic

```c
int a = 10, b = 3;

a + b    // 13 — addition
a - b    // 7  — subtraction
a * b    // 30 — multiplication
a / b    // 3  — INTEGER DIVISION — decimal part truncated toward zero
a % b    // 1  — modulo — remainder after integer division
```

**integer division** is the big one. `10 / 3` in C is `3`, not `3.333`. the rule is simple: when both operands are integers, the result is an integer, and the fractional part is discarded (truncated toward zero, so `-7 / 2` is `-3` not `-4`).

to get decimal division, at least one operand must be a floating-point type:

```c
10.0 / 3      // 3.333... — 10.0 is a double literal
(double)10 / 3  // 3.333... — cast before dividing
10 / 3.0      // 3.333... — 3.0 is a double literal
```

**modulo** is the remainder after division. `10 % 3` = 1 because `10 = 3*3 + 1`. useful patterns:

```c
n % 2 == 0          // is n even? (remainder when divided by 2 is 0)
n % 10              // last digit of n
(n + 1) % size      // next index with wraparound (circular buffer)
n % k               // cycle through 0, 1, 2, ..., k-1
```

modulo with negative numbers: in C99+, the result has the sign of the dividend. `-7 % 3` is `-1` (not `2`). if you need a guaranteed non-negative result: `((n % k) + k) % k`.

**operator precedence** — `*`, `/`, `%` bind tighter than `+`, `-`. when in doubt, use parentheses. `2 + 3 * 4` is `14`, not `20`.

### Assignment and Shorthand

```c
x = 5;     // assignment — not comparison

x += 5;    // x = x + 5
x -= 3;    // x = x - 3
x *= 2;    // x = x * 2
x /= 4;    // x = x / 4
x %= 7;    // x = x % 7
```

these are all equivalent — just shorthand. `x += 5` is identical to `x = x + 5`.

assignment is an *expression* in C — it evaluates to the value being assigned. this enables patterns like:

```c
int c;
while ((c = fgetc(fp)) != EOF) {  // assign AND check in one expression
    putchar(c);
}
```

this is idiomatic C. `c = fgetc(fp)` reads a character AND assigns it to `c`. the `!= EOF` then checks the assigned value. the parentheses around `c = fgetc(fp)` are required — without them, the operator precedence would make it `c = (fgetc(fp) != EOF)` which assigns 0 or 1, not the character.

### Increment and Decrement

```c
x++;   // post-increment: evaluate to current value of x, THEN add 1
++x;   // pre-increment: add 1 to x, THEN evaluate to the new value
x--;   // post-decrement
--x;   // pre-decrement
```

when used as a standalone statement (`x++;` on its own line), pre and post are identical — the result of the expression is discarded. the difference only matters when the expression value is used:

```c
int x = 5;
int a = x++;   // a = 5 (old value), then x = 6
int b = ++x;   // x = 7 first, then b = 7

// x++ is implemented as:
// int temp = x;
// x = x + 1;
// the expression value is temp (old value)

// ++x is implemented as:
// x = x + 1;
// the expression value is x (new value)
```

**never do multiple increments in the same expression:**
```c
int i = 0;
printf("%d %d\n", i++, i++);   // UNDEFINED BEHAVIOR
// the order of argument evaluation is unspecified
```

### Comparison Operators

```c
a == b   // equal — TWO equals signs, not one
a != b   // not equal
a > b    // greater than
a < b    // less than
a >= b   // greater than or equal
a <= b   // less than or equal
```

these return `1` for true and `0` for false. C doesn't have a native boolean type before C99 (when `<stdbool.h>` was added). in C, *any non-zero value is true, zero is false*. this means:

```c
if (5) { ... }        // always executes — 5 is non-zero (truthy)
if (0) { ... }        // never executes — 0 is falsy
if (ptr) { ... }      // executes if ptr is non-NULL — idiomatic null check
if (!ptr) { ... }     // executes if ptr is NULL
```

**THE classic bug — `=` vs `==`:**
```c
if (x = 5) { ... }    // ASSIGNS 5 to x, then checks if 5 is non-zero (always true)
if (x == 5) { ... }   // COMPARES x to 5
```

`-Wall` catches this and warns you. some people write comparisons as `5 == x` (Yoda conditions) so if they accidentally write `5 = x`, the compiler rejects it (can't assign to a literal). your call, but at minimum use `-Wall`.

### Logical Operators

```c
&&   // AND — true only if both sides are non-zero
||   // OR  — true if at least one side is non-zero
!    // NOT — flips truth value. !0 = 1, !5 = 0, !!5 = 1
```

**short-circuit evaluation** — this is not just trivia, it's an important feature:

for `&&`: if the left side is false (zero), the right side is *never evaluated*.
for `||`: if the left side is true (non-zero), the right side is *never evaluated*.

```c
// safe null pointer check
if (ptr != NULL && ptr->value > 5) {
    // if ptr is NULL, second check never runs
    // so we can't crash dereferencing NULL here
}

// safe array access
if (i < len && arr[i] == target) {
    // if i >= len, arr[i] is never evaluated
    // so no out-of-bounds access
}

// common C idiom — open file only if path is not null
if (path && (fp = fopen(path, "r"))) {
    // both conditions true
}
```

### Bitwise Operators

these operate on individual bits. not just for tricks — they're fundamental for systems programming, flags, hardware interfaces, network protocols.

```c
&    // AND — each bit is 1 only if both corresponding bits are 1
|    // OR  — each bit is 1 if either corresponding bit is 1
^    // XOR — each bit is 1 if the corresponding bits are DIFFERENT
~    // NOT — flip every bit (unary operator)
<<   // left shift — shift all bits left by n positions
>>   // right shift — shift all bits right by n positions
```

concrete example with 8-bit numbers:

```
a = 0b10110100  (180 decimal)
b = 0b11001101  (205 decimal)

a & b = 0b10000100  (132) — only bits that are 1 in BOTH
a | b = 0b11111101  (253) — bits that are 1 in EITHER
a ^ b = 0b01111001  (121) — bits that are 1 in one but NOT both
~a    = 0b01001011  (75)  — every bit flipped

a << 2 = 0b11010000 (208) — shifted left 2 positions (multiply by 4)
a >> 1 = 0b01011010 (90)  — shifted right 1 position (divide by 2)
```

**left shift = multiply by 2ⁿ.** `x << 3` = `x * 8`. this is genuinely faster than multiplication on some hardware, and compilers often use it internally.

**right shift = divide by 2ⁿ (integer).** `x >> 2` = `x / 4`. for unsigned integers, this is well-defined. for signed negative integers, whether the sign bit is preserved (arithmetic shift) or zero-filled (logical shift) is implementation-defined — avoid right-shifting negative values.

**the flag pattern** — storing multiple boolean values in a single integer:

```c
// define each flag as a power of 2 (one bit set)
#define FLAG_READ    (1 << 0)   // 0b00000001 = 1
#define FLAG_WRITE   (1 << 1)   // 0b00000010 = 2
#define FLAG_EXEC    (1 << 2)   // 0b00000100 = 4
#define FLAG_HIDDEN  (1 << 3)   // 0b00001000 = 8

int permissions = 0;

// SET a flag — use OR
permissions |= FLAG_READ;    // turn on bit 0
permissions |= FLAG_WRITE;   // turn on bit 1
// permissions = 0b00000011 = 3

// CHECK a flag — use AND
if (permissions & FLAG_READ) {
    printf("can read\n");
}

// CLEAR a flag — use AND with NOT
permissions &= ~FLAG_WRITE;  // ~FLAG_WRITE = 0b11111101
                              // AND keeps all bits EXCEPT bit 1
// permissions = 0b00000001 = 1

// TOGGLE a flag — use XOR
permissions ^= FLAG_EXEC;    // flip bit 2
```

this pattern is everywhere in C: linux file permissions, network socket options, hardware control registers, window manager flags.

### The sizeof Operator

`sizeof` is a compile-time operator (not a function) that returns the size in bytes of a type or expression:

```c
sizeof(int)           // 4 on most systems
sizeof(double)        // 8
sizeof(char)          // always 1, by definition
sizeof(void*)         // 4 on 32-bit systems, 8 on 64-bit

int arr[10];
sizeof(arr)           // 40 (10 * sizeof(int))
sizeof(arr[0])        // 4
sizeof(arr) / sizeof(arr[0])  // 10 — number of elements

// sizeof doesn't evaluate its argument
int x = 5;
sizeof(x++)           // 4 — x is NOT incremented, sizeof never executes the expression
```

`sizeof` returns `size_t`. use `%zu` to print it:
```c
printf("int is %zu bytes\n", sizeof(int));
```

### The Ternary Operator

```c
int max = (a > b) ? a : b;
// equivalent to:
// int max;
// if (a > b) max = a;
// else max = b;
```

syntax: `condition ? value_if_true : value_if_false`. good for simple conditional assignments. don't nest them — it gets unreadable fast.

### The Comma Operator

the comma `,` is an actual operator — it evaluates both expressions left to right and returns the value of the right one:

```c
int x = (5, 10);  // x = 10 — 5 is evaluated and discarded
```

mainly used in for loop headers:
```c
for (int i = 0, j = 10; i < j; i++, j--) {
    // initialize multiple variables, update multiple variables
}
```

---

## Control Flow

### if / else if / else

```c
int score = 85;

if (score >= 90) {
    printf("A\n");
} else if (score >= 80) {
    printf("B\n");
} else if (score >= 70) {
    printf("C\n");
} else {
    printf("F\n");
}
```

the condition can be any expression — if it evaluates to non-zero, the block executes.

curly braces are technically optional for single statements. don't skip them. apple's "goto fail" SSL bug (2014) was a security vulnerability that allowed bypassing certificate verification. the bug was a duplicated `goto fail;` line without braces:

```c
if ((err = SSLHashSHA1.update(&hashCtx, &signedParams)) != 0)
    goto fail;
    goto fail;   // always executes — even when the if is false
```

the second `goto fail` was always reached. always. use braces.

### switch

```c
int day = 3;

switch (day) {
    case 1:
        printf("Monday\n");
        break;
    case 2:
        printf("Tuesday\n");
        break;
    case 3:
        printf("Wednesday\n");
        break;
    default:
        printf("Other\n");
        break;
}
```

**how switch actually works:** unlike other languages, switch doesn't do "match one case and run it". it *jumps* to the matching case and then runs everything until either a `break` or the end of the switch. this is called fall-through.

fall-through is sometimes intentional:
```c
switch (day) {
    case 1:
    case 2:
    case 3:
    case 4:
    case 5:
        printf("weekday\n");
        break;    // all five cases fall through to here
    case 6:
    case 7:
        printf("weekend\n");
        break;
}
```

switch only works with integer types (including char and enum). you can't switch on a float or a string.

**switch vs if-else:** switch compiles to a *jump table* when the cases are dense integers — O(1) lookup regardless of how many cases. a chain of if-else checks them one by one — O(n). for large numbers of cases on integer values, switch is faster.

### while

```c
int i = 0;
while (i < 10) {
    printf("%d\n", i);
    i++;
}
```

**what's actually happening:** the condition is evaluated. if non-zero, the body executes. then the condition is evaluated again. this repeats until the condition is zero. if the condition is false from the start, the body never runs.

the condition can be any expression:
```c
while (1) { ... }       // infinite loop — 1 is always non-zero
while (ptr) { ... }     // loop while ptr is non-NULL
while (fgets(buf, sizeof(buf), fp)) { ... }  // loop while fgets succeeds
```

### do-while

```c
int input;
do {
    printf("Enter a number between 1 and 10: ");
    scanf("%d", &input);
} while (input < 1 || input > 10);
```

body runs first, condition checked after. guaranteed to run at least once. essential for input validation where you need to get the input before you can check it.

### for

```c
for (int i = 0; i < 10; i++) {
    printf("%d\n", i);
}
```

three parts separated by semicolons:
1. **initialization** — runs once before the loop starts
2. **condition** — checked before each iteration, loop continues while non-zero
3. **update** — runs after each iteration

any part can be empty:
```c
int i = 0;
for (; i < 10; i++) { }   // skip init — i already exists

for (int i = 0; ; i++) {   // skip condition — implicit infinite loop
    if (i == 10) break;
}

for (;;) { }               // skip all three — classic infinite loop
```

multiple variables:
```c
for (int i = 0, j = 9; i < j; i++, j--) {
    printf("i=%d j=%d\n", i, j);
}
```

**what the compiler does with for:** `for (init; cond; update) { body }` is equivalent to:
```c
init;
while (cond) {
    body;
    update;
}
```

### break and continue

`break` exits the innermost loop or switch immediately:
```c
for (int i = 0; i < 100; i++) {
    if (arr[i] == target) {
        printf("found at %d\n", i);
        break;   // stop searching, we found it
    }
}
```

`continue` skips the rest of the current iteration and jumps to the update expression (for loop) or condition check (while/do-while):
```c
for (int i = 0; i < 10; i++) {
    if (i % 2 == 0) continue;   // skip even numbers
    printf("%d\n", i);           // prints 1, 3, 5, 7, 9
}
```

both only affect the *innermost* enclosing loop. to break out of nested loops:

### goto

`goto` jumps to a labeled statement anywhere in the current function:

```c
for (int i = 0; i < 100; i++) {
    for (int j = 0; j < 100; j++) {
        for (int k = 0; k < 100; k++) {
            if (found) goto cleanup;
        }
    }
}

cleanup:
    free(buffer);
    fclose(fp);
```

`goto` has a bad reputation from old-style "spaghetti code" where gotos jumped all over the place making programs impossible to follow. in C, there are two legitimate uses:
1. breaking out of nested loops (shown above)
2. cleanup/error handling patterns in functions that acquire multiple resources

the linux kernel uses goto extensively for cleanup. don't be afraid of it when it's genuinely the clearest option. just don't use it to jump backward or to jump into blocks.

---

## Functions

### What Functions Actually Are

a function is a named sequence of instructions at a specific memory address. when you call a function, the CPU:
1. pushes the return address onto the call stack
2. pushes function arguments (or puts them in registers)
3. jumps to the function's address
4. the function executes
5. the return value is put in a register (or on the stack for large values)
6. the function pops back to the return address and execution continues

understanding this makes several things obvious: why local variables only exist during a function call, why returning a pointer to a local variable is a bug, and why infinite recursion crashes (the stack overflows).

### Basic Function Anatomy

```c
#include <stdio.h>

// PROTOTYPE — declares the function exists before we define it
// tells the compiler: "there's a function called add, takes two ints, returns int"
int add(int a, int b);

int main(void) {
    int result = add(3, 4);   // call — jumps to add, gets result back
    printf("%d\n", result);   // 7
    return 0;
}

// DEFINITION — the actual code
int add(int a, int b) {
    return a + b;
}
```

**why prototypes?** C processes files top to bottom. without a prototype, if `main` calls `add` before `add` is defined, the compiler doesn't know what `add` looks like. prototypes tell the compiler in advance. in a multi-file project, prototypes go in header files so any file can use the function.

### Call Stack — How Function Calls Work in Memory

the call stack is a region of memory that grows and shrinks as functions are called and return. each function call creates a *stack frame* containing:
- the function's local variables
- the return address (where to go after the function returns)
- saved register values
- function parameters (sometimes)

```
main's stack frame:
  result = 7
  return address = OS

add's stack frame:  ← pushed when add is called
  a = 3
  b = 4
  return address = address in main after the call
```

when `add` returns, its stack frame is popped — that memory is reclaimed. the local variables `a` and `b` are gone.

this is why:
```c
int *bad_function(void) {
    int local = 42;
    return &local;   // local lives in bad_function's stack frame
}                    // stack frame is popped here — local is GONE
                     // the returned pointer points to now-invalid memory
```

### Parameters Are Copies

when you pass a variable to a function, C makes a *copy*. the function works on its own copy. the original is unchanged:

```c
void try_to_modify(int x) {
    x = 999;    // modifies the local copy only
}

int n = 5;
try_to_modify(n);
printf("%d\n", n);  // still 5 — the copy was modified, not n
```

this is called *pass by value*. it's a deliberate design choice — functions can't accidentally modify variables in the caller. to intentionally modify a variable, you pass its *address* (a pointer):

```c
void actually_modify(int *p) {
    *p = 999;    // dereference the pointer, modify the original
}

int n = 5;
actually_modify(&n);   // pass n's address
printf("%d\n", n);     // 999 — actually changed
```

### Return Values

a function can return one value. the return type must match what's declared:

```c
int max(int a, int b) {
    return a > b ? a : b;
}

double average(double *arr, int len) {
    double sum = 0;
    for (int i = 0; i < len; i++) sum += arr[i];
    return sum / len;
}

void print_separator(void) {
    printf("---\n");
    // no return needed in void function
    // return; is valid to exit early
}
```

to "return" multiple values, use output parameters (pointer parameters where you write the result):

```c
void minmax(int *arr, int len, int *out_min, int *out_max) {
    *out_min = *out_max = arr[0];
    for (int i = 1; i < len; i++) {
        if (arr[i] < *out_min) *out_min = arr[i];
        if (arr[i] > *out_max) *out_max = arr[i];
    }
}

int data[] = {3, 1, 4, 1, 5, 9, 2, 6};
int mn, mx;
minmax(data, 8, &mn, &mx);
printf("min=%d max=%d\n", mn, mx);
```

### Recursion

a function can call itself. every recursive function needs a *base case* (a condition that stops the recursion) and a *recursive case* (where it calls itself):

```c
int factorial(int n) {
    if (n <= 1) return 1;           // base case — stop recursing
    return n * factorial(n - 1);   // recursive case
}

// factorial(4):
//   = 4 * factorial(3)
//   = 4 * (3 * factorial(2))
//   = 4 * (3 * (2 * factorial(1)))
//   = 4 * (3 * (2 * 1))
//   = 4 * (3 * 2)
//   = 4 * 6
//   = 24
```

each recursive call creates a new stack frame. deep recursion can overflow the stack. `factorial(100000)` would create 100,000 stack frames — stack overflow. for deep recursion, use an iterative approach.

### static Local Variables

a `static` local variable retains its value between function calls. it's initialized once (at program start) and lives for the entire program:

```c
int make_id(void) {
    static int counter = 0;   // initialized once, NOT each call
    return ++counter;
}

make_id()  // returns 1
make_id()  // returns 2
make_id()  // returns 3
// counter's value persists between calls
```

useful for things like: counters, caches, one-time initialization flags.

### Function Pointers

functions have memory addresses. you can store those addresses in pointers and call functions through them:

```c
int add(int a, int b) { return a + b; }
int sub(int a, int b) { return a - b; }
int mul(int a, int b) { return a * b; }

// declare a pointer to a function taking two ints, returning int
int (*operation)(int, int);

operation = add;
printf("%d\n", operation(3, 4));   // 7

operation = mul;
printf("%d\n", operation(3, 4));   // 12
```

the real power is passing functions as arguments to other functions — *callbacks*:

```c
// apply a transformation to every element of an array
void transform(int *arr, int len, int (*func)(int)) {
    for (int i = 0; i < len; i++) {
        arr[i] = func(arr[i]);
    }
}

int double_it(int x) { return x * 2; }
int square(int x) { return x * x; }

int arr[] = {1, 2, 3, 4, 5};
transform(arr, 5, double_it);   // {2, 4, 6, 8, 10}
transform(arr, 5, square);      // {4, 16, 36, 64, 100}
```

the C standard library uses this pattern extensively. `qsort` takes a comparison function pointer so it can sort anything:

```c
int compare_ints(const void *a, const void *b) {
    int ia = *(const int *)a;
    int ib = *(const int *)b;
    return (ia > ib) - (ia < ib);   // returns -1, 0, or 1
}

int arr[] = {5, 2, 8, 1, 9, 3};
qsort(arr, 6, sizeof(int), compare_ints);
// arr = {1, 2, 3, 5, 8, 9}
```

`typedef` makes function pointer syntax less terrifying:
```c
typedef int (*BinaryOp)(int, int);

BinaryOp op = add;
printf("%d\n", op(5, 3));
```

### Variadic Functions

functions that accept a variable number of arguments, like `printf`:

```c
#include <stdarg.h>

// ... means "more arguments after this"
// count tells us how many there are (we can't figure it out otherwise)
int sum_all(int count, ...) {
    va_list args;
    va_start(args, count);   // initialize args to point after 'count'

    int total = 0;
    for (int i = 0; i < count; i++) {
        total += va_arg(args, int);   // get next argument, specifying its type
    }

    va_end(args);    // clean up
    return total;
}

sum_all(3, 10, 20, 30)   // 60
sum_all(5, 1, 2, 3, 4, 5) // 15
```

C has no way to automatically know how many arguments were passed or what their types are. you have to pass that information yourself (printf uses the format string, our sum_all uses the count parameter).

### Command Line Arguments

`main` can receive command-line arguments:

```c
int main(int argc, char *argv[]) {
    // argc = argument count (including the program name itself)
    // argv = array of argument strings
    // argv[0] = program name
    // argv[1] = first argument
    // ...
    // argv[argc-1] = last argument
    // argv[argc] = NULL (sentinel)

    printf("program: %s\n", argv[0]);

    for (int i = 1; i < argc; i++) {
        printf("arg %d: %s\n", i, argv[i]);
    }

    return 0;
}
```

```bash
./myprogram hello world 42
# argc = 4
# argv[0] = "./myprogram"
# argv[1] = "hello"
# argv[2] = "world"
# argv[3] = "42"
```

`argv[3]` is the string "42", not the integer 42. to convert: `int n = atoi(argv[3]);` or better `strtol(argv[3], NULL, 10)`.

---

## Arrays

### What an Array Actually Is in Memory

an array is a contiguous block of memory holding elements of the same type. "contiguous" means all the elements are right next to each other, with no gaps:

```
int arr[5] = {10, 20, 30, 40, 50};

Memory layout:
Address:  0x100  0x104  0x108  0x10C  0x110
Value:      10     20     30     40     50
Index:       0      1      2      3      4
```

each `int` is 4 bytes, so elements are 4 bytes apart. `arr[0]` is at address 0x100, `arr[1]` at 0x104, `arr[2]` at 0x108, etc.

**array indexing is pointer arithmetic.** `arr[i]` is exactly the same as `*(arr + i)`. the compiler sees `arr[2]` and computes: start address + (2 × sizeof(int)) = 0x100 + 8 = 0x108, then reads the int at that address.

### Declaring Arrays

```c
int numbers[5];                          // 5 ints, uninitialized (garbage)
int scores[5] = {95, 87, 72, 91, 68};  // initialized with values
int zeroes[100] = {0};                   // first element 0, rest default to 0
                                          // only the FIRST element is explicitly 0
                                          // others are zero-initialized by the rule
                                          // "if you partially initialize, rest is zero"
int auto_size[] = {1, 2, 3, 4, 5};     // compiler counts: size is 5

// designated initializers (C99):
int sparse[10] = {[0] = 1, [5] = 10, [9] = 99};
// all other elements are 0
```

### Accessing and Modifying Elements

```c
int arr[5] = {10, 20, 30, 40, 50};

arr[0]        // 10 — first element
arr[4]        // 50 — last element (index = size - 1)
arr[2] = 99;  // modify element at index 2

// accessing through a pointer (equivalent):
int *p = arr;    // arr decays to pointer to first element
*(p + 0)         // 10
*(p + 2)         // 30 (after modification: 99)
p[2]             // same as *(p + 2)
```

**C never checks array bounds.** accessing `arr[5]` on a 5-element array compiles silently and reads whatever memory happens to be at that address. writing `arr[5] = 0` writes to memory you don't own. this can corrupt other variables, corrupt heap metadata, enable security exploits, or crash — or it can appear to "work" and cause a bug far later. C trusts you completely with memory access.

### Getting Array Length

```c
int arr[] = {1, 2, 3, 4, 5};
int len = sizeof(arr) / sizeof(arr[0]);  // total bytes / bytes per element
```

`sizeof(arr)` gives the total bytes (20). `sizeof(arr[0])` gives bytes per element (4). dividing: 5 elements. clean. **but this only works in the scope where the array is declared.**

once you pass an array to a function, it *decays* to a pointer to its first element. at that point `sizeof` gives you the pointer size (8 bytes on 64-bit), not the array size. always pass length separately:

```c
void print_array(int *arr, int len) {
    // sizeof(arr) here is sizeof(int*) = 8 — NOT the array size
    for (int i = 0; i < len; i++) {
        printf("%d ", arr[i]);
    }
}

int data[] = {1, 2, 3, 4, 5};
print_array(data, 5);   // must pass 5 explicitly
// or:
print_array(data, sizeof(data) / sizeof(data[0]));  // compute it here where it works
```

a common macro:
```c
#define ARRAY_LEN(a) (sizeof(a) / sizeof((a)[0]))
// the (a)[0] with parentheses handles edge cases where a is a complex expression
```

### Multidimensional Arrays

a 2D array is an array of arrays:

```c
int matrix[3][4];   // 3 rows, 4 columns
```

in memory, it's stored *row by row* (row-major order):
```
matrix[0][0], matrix[0][1], matrix[0][2], matrix[0][3],
matrix[1][0], matrix[1][1], matrix[1][2], matrix[1][3],
matrix[2][0], matrix[2][1], matrix[2][2], matrix[2][3]
```

this is one contiguous block of 12 ints. `matrix[1][2]` accesses the element at row 1, column 2 — which is at offset `(1 * 4 + 2) * sizeof(int)` from the start.

```c
int grid[2][3] = {
    {1, 2, 3},
    {4, 5, 6}
};

grid[1][2]   // 6 — row 1, column 2

// iterate
for (int i = 0; i < 2; i++) {
    for (int j = 0; j < 3; j++) {
        printf("%d ", grid[i][j]);
    }
    printf("\n");
}
```

**cache performance:** accessing `grid[i][j]` row by row (incrementing j in the inner loop) is cache-friendly — you're moving through memory sequentially. accessing column by column (incrementing i in the inner loop) means jumping `4 * sizeof(int)` bytes on each step — worse cache performance.

### Variable-Length Arrays (C99)

C99 allows array sizes to be runtime variables:

```c
int n;
scanf("%d", &n);
int arr[n];    // VLA — size determined at runtime
arr[0] = 42;
```

VLAs are allocated on the stack. they're automatically freed when the function returns. limitations: can't initialize at declaration time, can't use `sizeof` at compile time, C11 made support *optional* (though gcc and clang still support them). for large or unknown-size allocations, use `malloc` instead.

---

## Strings

### What a String Actually Is

strings in C are just arrays of `char` with a null terminator byte (`\0`, ASCII value 0) at the end. there is no special "string" type. just bytes.

```c
char hello[] = "Hello";
```

in memory:
```
Index:    0     1     2     3     4     5
Value:   'H'   'e'   'l'   'l'   'o'  '\0'
Decimal:  72   101   108   108   111    0
```

the null terminator is automatically added by the compiler when you use a string literal. it's how all string functions know where the string ends — they scan until they find a `\0`.

**there's no length stored anywhere.** `strlen` works by scanning from the start until it hits `\0`, counting as it goes. it's O(n). calling it in a loop condition (`while (i < strlen(s))`) recomputes it every iteration — cache the result.

### String Initialization Options

```c
// 1. Array initialized from string literal — WRITABLE, \0 auto-added
char s1[] = "Hello";          // size 6 (5 + \0), writable

// 2. Array with explicit size
char s2[10] = "Hello";        // size 10, first 6 used ("Hello\0"), rest zero

// 3. Pointer to string literal — READ-ONLY
const char *s3 = "Hello";     // points to read-only memory

// 4. Manual initialization
char s4[6] = {'H', 'e', 'l', 'l', 'o', '\0'};  // same as s1
```

**the critical difference between 1 and 3:**

`char s1[] = "Hello"` — the string literal is copied into a writable array on the stack. you can modify `s1[0] = 'h'` freely.

`const char *s3 = "Hello"` — `s3` is a pointer to a string literal stored in the program's read-only data segment. trying to modify it (`s3[0] = 'h'`) is undefined behavior, usually a segfault. always use `const char *` for pointers to string literals.

### Reading Strings

`scanf` with `%s` reads until whitespace:
```c
char buf[50];
scanf("%s", buf);         // no & needed — buf already decays to pointer
                          // reads until whitespace, NO bounds check
scanf("%49s", buf);       // safer — reads at most 49 chars, leaves room for \0
```

`fgets` reads an entire line and respects the buffer size:
```c
char buf[100];
fgets(buf, sizeof(buf), stdin);   // reads at most 99 chars + \0
// fgets keeps the newline character '\n' at the end
// strip it:
buf[strcspn(buf, "\n")] = '\0';   // find '\n' position, replace with '\0'
```

`fgets` is generally safer than `scanf("%s")`. it won't overflow the buffer and it reads whole lines including spaces.

### String Functions — what they do and why

```c
#include <string.h>
```

**`strlen(s)`** — scan `s` until `\0`, return the count. does NOT include the null terminator. O(n).

**`strcpy(dest, src)`** — copy `src` into `dest` including the `\0`. **dangerous** — if `src` is longer than `dest`'s buffer, it writes past the end. classic buffer overflow. use `strncpy` or `snprintf` instead.

**`strncpy(dest, src, n)`** — copy at most `n` characters. GOTCHA: if `src` is exactly `n` characters long, no `\0` is written. always manually null-terminate after:
```c
strncpy(dest, src, sizeof(dest) - 1);
dest[sizeof(dest) - 1] = '\0';  // ensure null termination
```

**`strcat(dest, src)`** — append `src` to the end of `dest`. also dangerous for the same reason as `strcpy`.

**`strncat(dest, src, n)`** — append at most `n` characters. safer.

**`strcmp(s1, s2)`** — compare two strings character by character. returns 0 if equal, negative if s1 < s2 lexicographically, positive if s1 > s2. **never compare strings with `==`** — that compares pointers (addresses), not content.

```c
char a[] = "hello";
char b[] = "hello";

a == b             // WRONG — compares addresses (different arrays), likely false
strcmp(a, b) == 0  // CORRECT — compares content, true
```

**`strchr(s, c)`** — find first occurrence of character `c` in string `s`. returns a pointer to it, or NULL if not found.

**`strstr(s, sub)`** — find first occurrence of substring `sub` in string `s`. returns pointer to start of match, or NULL.

**`memset(ptr, value, n)`** — set `n` bytes starting at `ptr` to `value`. value is a byte (int that gets truncated to unsigned char):
```c
char buf[100];
memset(buf, 0, sizeof(buf));   // zero out buffer
memset(buf, 'X', 10);         // set first 10 bytes to 'X'
```

**`memcpy(dest, src, n)`** — copy `n` bytes from `src` to `dest`. faster than a loop for large copies. undefined behavior if src and dest overlap.

**`memmove(dest, src, n)`** — same as memcpy but safe when src and dest overlap. slightly slower. use when copying within the same buffer.

### Safe String Building — snprintf

the right way to build strings of unknown length:

```c
char buf[64];
snprintf(buf, sizeof(buf), "Hello %s, you are %d years old", name, age);
// snprintf ALWAYS null-terminates (unlike strncpy)
// NEVER writes more than sizeof(buf) bytes
// if the string would be longer, it's truncated
```

`snprintf` is like `printf` but writes to a buffer instead of stdout. it's the safest general-purpose string building tool in C.

### String Conversion Functions

```c
#include <stdlib.h>

atoi("42")      // string to int — "42" → 42. NO error checking.
atof("3.14")    // string to double. NO error checking.

// better: strtol has proper error handling
char *end;
long n = strtol("123abc", &end, 10);   // base 10
// n = 123
// end points to "abc" — where parsing stopped
if (*end != '\0') {
    printf("trailing non-numeric: %s\n", end);
}

// check for overflow:
errno = 0;
long val = strtol(str, &end, 10);
if (errno == ERANGE) {
    printf("overflow\n");
}
```

---

## Pointers

pointers are the most important and most misunderstood feature of C. take your time here. read it twice if needed.

### Memory Addresses — the foundation

every variable has an address — a number that identifies its location in RAM. on a 64-bit system, addresses are 64-bit numbers like `0x7fff5a8b2c10`.

a pointer is a variable that stores one of these addresses:

```c
int x = 42;
int *p = &x;    // p stores the address of x
```

three operators:
- `&x` — **address-of operator**: gives you x's address
- `int *p` — **pointer declaration**: p is a variable that holds an address of an int
- `*p` — **dereference operator**: go to the address in p, give me what's there

```c
printf("%d\n",   x);           // 42 — value of x
printf("%p\n",   (void*)p);    // 0x7fff... — address stored in p
printf("%d\n",   *p);          // 42 — value at the address p holds
printf("%zu\n",  sizeof(p));   // 8 — pointer is 8 bytes on 64-bit
```

since `*p` refers to the same memory as `x`, modifying one modifies the other:
```c
*p = 100;
printf("%d\n", x);   // 100
x = 200;
printf("%d\n", *p);  // 200
```

### Why Pointers Exist

three main reasons:

**1. Modifying variables from inside a function.** parameters are copies. pointers let you modify the original:
```c
void increment(int *n) {
    (*n)++;   // dereference n, then increment the value there
}

int count = 5;
increment(&count);
printf("%d\n", count);   // 6
```

**2. Efficient passing of large data.** copying a 1MB struct every function call is wasteful. passing a pointer (8 bytes) is cheap:
```c
void process(const LargeStruct *data) {  // 8 bytes passed, not 1MB
    // read data through pointer
}
```

**3. Dynamic memory allocation.** `malloc` returns a pointer to heap memory. without pointers you can't use the heap:
```c
int *arr = malloc(1000 * sizeof(int));   // pointer to 4000 bytes of heap memory
```

### Pointer Arithmetic — how indexing works

when you add an integer to a pointer, it advances by that many *elements*, not bytes:

```c
int arr[] = {10, 20, 30, 40, 50};
int *p = arr;   // p points to arr[0], address 0x100 (hypothetically)

p + 0   // address 0x100, value 10
p + 1   // address 0x104, value 20  (0x100 + 1*sizeof(int) = 0x100 + 4)
p + 2   // address 0x108, value 30
p + 3   // address 0x10C, value 40

*(p + 1)  // 20 — dereference
p[1]      // 20 — IDENTICAL TO *(p + 1), just different syntax
```

**array indexing is pointer arithmetic.** `arr[i]` is literally defined as `*(arr + i)`. they compile to the same machine code.

`p + 1` adds `sizeof(int)` bytes to the address. `p + n` adds `n * sizeof(int)` bytes. the compiler handles the scaling automatically based on the pointer type. this is why pointer type matters — `char *p; p + 1` moves 1 byte. `int *p; p + 1` moves 4 bytes. `double *p; p + 1` moves 8 bytes.

subtracting two pointers gives the number of elements between them:
```c
int *start = arr;
int *end = arr + 5;
ptrdiff_t n = end - start;   // 5 — number of elements, not bytes
```

### NULL Pointers

a pointer that doesn't point to anything valid should be set to NULL:

```c
int *p = NULL;   // NULL is defined as ((void*)0) or just 0

if (p == NULL) {
    printf("pointer is not initialized\n");
}

if (!p) {   // same thing — NULL is zero, zero is false
    printf("pointer is null\n");
}

*p = 5;   // CRASH (SIGSEGV) — dereferencing NULL
           // the OS catches this and kills your program
```

**why does dereferencing NULL crash?** address 0 is intentionally mapped to nothing by the OS. accessing it triggers a protection fault (segfault). this is actually a feature — it turns "use before initialize" bugs into immediate, detectable crashes rather than silent data corruption.

always check for NULL before dereferencing if there's any chance the pointer could be NULL. functions like `malloc`, `fopen`, `strchr` return NULL on failure.

### void Pointers

`void*` is a pointer with no type — it just holds an address without specifying what's there:

```c
void *ptr;
int x = 42;
ptr = &x;   // void* can hold any pointer type, no cast needed

// to use the value, cast to the correct type:
int *ip = (int *)ptr;
printf("%d\n", *ip);   // 42
```

`void*` is how C implements generic functions. `malloc` returns `void*` — it doesn't know what type you'll store there. `memcpy` takes `void*` parameters — it can copy any type. `qsort`'s comparison function receives `const void*` parameters.

### Pointer to Pointer

pointers can point to other pointers:

```c
int x = 5;
int *p = &x;
int **pp = &p;   // pointer to a pointer to int

printf("%d\n", x);    // 5 — the value
printf("%d\n", *p);   // 5 — value through one level of indirection
printf("%d\n", **pp); // 5 — value through two levels

**pp = 99;
printf("%d\n", x);    // 99 — modified through two levels of pointer
```

where this actually comes up:
- dynamic 2D arrays (`int **matrix`)
- functions that need to modify a pointer (like memory allocation wrappers)
- arrays of strings (`char **argv`)

```c
// function that allocates a buffer and gives you a pointer to it
void allocate(int **out, int size) {
    *out = malloc(size * sizeof(int));
}

int *data = NULL;
allocate(&data, 100);   // pass pointer to pointer
// data now points to allocated memory
```

### const and Pointers — four combinations

```c
int x = 5;

int *p = &x;                  // can change p (point elsewhere), can change *p
const int *p = &x;            // can change p, CANNOT change *p
int * const p = &x;           // CANNOT change p, can change *p
const int * const p = &x;     // cannot change either
```

read right-to-left through the type declaration:
- `const int *p` — "p is a [pointer to] [const int]"
- `int * const p` — "p is a [const] [pointer to] [int]"

use `const int *` in function parameters to promise you won't modify what the pointer points to:
```c
void print_string(const char *s) {  // won't modify the string
    while (*s) putchar(*s++);
}
```

this lets callers safely pass string literals (which are read-only):
```c
print_string("hello");  // fine — const char* can point to string literals
```

without `const`, the function signature would accept a pointer to modifiable chars, and passing a string literal would be technically wrong (even if the function doesn't actually modify it).

### Function Pointers in Depth

the syntax for declaring function pointers is notorious for being confusing:

```c
// a function:
int add(int a, int b) { return a + b; }

// a pointer to a function taking (int, int) and returning int:
int (*fp)(int, int);

// to read the declaration: start from the variable name, go right, then left
// fp is a (*fp) pointer to (int, int) function taking two ints, returning int
```

calling:
```c
fp = add;          // assign function address (no & needed, function name is already a pointer)
fp(3, 4)           // call through pointer — same as add(3, 4)
(*fp)(3, 4)        // also valid — explicitly dereference first
```

typedef cleans up the syntax:
```c
typedef int (*BinaryIntOp)(int, int);

BinaryIntOp operations[] = {add, sub, mul};
for (int i = 0; i < 3; i++) {
    printf("%d\n", operations[i](10, 3));
}
```

---

## Memory Management

### The Stack vs The Heap

**the stack** — fast, automatic, limited. local variables live here. when a function is called, space is pushed onto the stack. when it returns, space is popped. you never explicitly allocate or free stack memory — it's automatic.

```
| main's locals     |
| foo's locals      |  ← top of stack (grows downward)
| bar's locals      |
```

the stack is typically 1-8 MB. allocating too much on the stack (huge local arrays, deep recursion) overflows it — "stack overflow" is literally running out of stack space.

**the heap** — large, manual, flexible. you explicitly request memory from the heap with `malloc` and release it with `free`. the heap is limited only by your system's RAM. but you're responsible for tracking and releasing everything you allocate.

### malloc — Memory Allocation

```c
#include <stdlib.h>

// allocate space for n bytes
// returns pointer to allocated memory, or NULL if allocation fails
void *malloc(size_t size);

int *arr = malloc(10 * sizeof(int));   // allocate for 10 ints
```

what `malloc` does:
1. finds a free block of memory at least `size` bytes large in the heap
2. marks it as in-use (so no other call gets the same memory)
3. returns a pointer to it

the returned memory is **uninitialized** — contains whatever bytes were there before (from previous allocations that were freed). never read from freshly malloc'd memory without writing to it first.

**always check the return value:**
```c
int *arr = malloc(10 * sizeof(int));
if (arr == NULL) {
    fprintf(stderr, "malloc failed\n");
    exit(1);   // or handle the error gracefully
}
```

malloc returning NULL is rare in practice (your system is nearly out of memory) but it can happen. a NULL dereference crash is confusing to debug — a checked error is not.

### calloc — Zero-Initialized Allocation

```c
void *calloc(size_t n, size_t size);

int *arr = calloc(10, sizeof(int));   // 10 ints, all set to zero
```

calloc = count × size bytes, zero-initialized. prefer calloc when you need the memory zeroed (avoids manually calling memset). the two-argument form also protects against integer overflow in the size calculation (if `n * size` would overflow, `malloc(n * size)` might allocate a tiny buffer; calloc handles this correctly).

### realloc — Resize an Allocation

```c
void *realloc(void *ptr, size_t new_size);
```

resizes a previously allocated block. may return the same pointer (if the block can grow in place), or a new pointer (if it had to move the data to a larger location).

```c
int *arr = malloc(10 * sizeof(int));
// ... fill arr with 10 ints ...

// need more space:
int *bigger = realloc(arr, 20 * sizeof(int));
if (bigger == NULL) {
    // realloc FAILED — BUT arr is still valid and unchanged
    // if you had done arr = realloc(arr, ...) and it returned NULL,
    // you'd have lost arr — memory leak
    free(arr);
    return NULL;
}
arr = bigger;   // only update arr after confirming success
// arr now points to space for 20 ints, first 10 are the original values
```

**the realloc gotcha:** never do `ptr = realloc(ptr, size)` directly. if realloc fails, it returns NULL but does NOT free the original pointer. `ptr = realloc(ptr, size)` on failure sets ptr to NULL — you've lost your only reference to the original memory. leaked forever. always use a temp:

```c
void *tmp = realloc(ptr, new_size);
if (tmp == NULL) {
    free(ptr);     // still responsible for original
    return error;
}
ptr = tmp;
```

### free — Releasing Memory

```c
void free(void *ptr);
```

returns the memory pointed to by `ptr` back to the heap for future use. `ptr` must be a pointer returned by `malloc`, `calloc`, or `realloc` — you can't free partial allocations or stack memory.

```c
int *arr = malloc(100 * sizeof(int));
// ... use arr ...
free(arr);
arr = NULL;   // set to NULL after freeing — good practice
```

**setting to NULL after freeing:** prevents accidental use-after-free. if you dereference a NULL pointer, you get an immediate crash (detectable). if you dereference a freed (but non-NULL) pointer, you get undefined behavior — might appear to work, might corrupt data silently, might crash later. NULL is the safe sentinel.

### The Four Memory Crimes

**1. Memory Leak — allocate but never free:**
```c
void leaky_function(void) {
    int *data = malloc(1000 * sizeof(int));
    // ... use data ...
    return;   // forgot free(data)
}
// call this in a loop — 4KB lost per call, eventually run out of memory
```

**2. Use-After-Free — access memory after freeing it:**
```c
int *p = malloc(sizeof(int));
*p = 42;
free(p);
printf("%d\n", *p);   // undefined behavior — memory freed
p[0] = 5;             // might corrupt heap allocator's internal data structures
```

after `free(p)`, the allocator might immediately reuse that memory for another allocation. writing to it after freeing can corrupt completely unrelated data in your program.

**3. Double Free — free the same pointer twice:**
```c
free(p);
free(p);   // undefined behavior — often corrupts heap metadata, causes crash later
```

the heap allocator stores metadata (size, next free block, etc.) in the memory surrounding allocations. freeing the same pointer twice can corrupt these structures, causing crashes that appear to happen at completely unrelated points in the program — very hard to debug.

**4. Buffer Overflow — write past the end of an allocation:**
```c
int *arr = malloc(5 * sizeof(int));
arr[5] = 99;   // one past the end — writes over whatever follows in memory
arr[-1] = 0;   // before the start — also writes outside the allocation
```

the allocator stores metadata right before each allocation. writing `arr[-1]` or `arr[5]` can corrupt that metadata, causing heap corruption bugs.

### How to Find Memory Bugs

**AddressSanitizer** — compile with `-fsanitize=address,undefined`. it wraps all memory operations with checks. when you access memory you shouldn't, it immediately prints exactly what went wrong, what line it happened on, and a full call stack. use this during development:

```bash
gcc -fsanitize=address,undefined -g -o program program.c
./program
```

**Valgrind** — a tool that runs your program in a virtual machine and tracks every byte of memory:
```bash
valgrind --leak-check=full --show-leak-kinds=all ./program
```

slower than ASan but catches more subtle issues. both are invaluable. use ASan first (faster), use Valgrind for thorough leak checking.

### Dynamic Array Pattern

C has no built-in growable array. here's how to implement one:

```c
typedef struct {
    int    *data;
    size_t  size;      // current number of elements
    size_t  capacity;  // allocated space
} IntVec;

IntVec vec_create(void) {
    IntVec v;
    v.data = malloc(4 * sizeof(int));
    v.size = 0;
    v.capacity = 4;
    return v;
}

void vec_push(IntVec *v, int val) {
    if (v->size == v->capacity) {
        v->capacity *= 2;  // double capacity
        int *tmp = realloc(v->data, v->capacity * sizeof(int));
        if (tmp == NULL) { fprintf(stderr, "OOM\n"); exit(1); }
        v->data = tmp;
    }
    v->data[v->size++] = val;
}

void vec_free(IntVec *v) {
    free(v->data);
    v->data = NULL;
    v->size = v->capacity = 0;
}
```

"double when full" is the standard growth strategy. python lists, C++ vectors, java ArrayLists — all use this. the mathematical reason it's O(1) amortized: if you double each time, the total copies made over n pushes is at most 2n — a constant factor.

---

## Structs, Unions, and Enums

### Structs — Grouping Related Data

a struct bundles multiple variables under one name. it's a custom data type:

```c
typedef struct {
    char   name[50];
    int    age;
    double gpa;
    int    student_id;
} Student;

Student s1 = {"Alice", 18, 3.9, 1001};                  // positional
Student s2 = {.name = "Bob", .age = 19, .gpa = 3.5,
              .student_id = 1002};                        // designated (C99)

printf("%s: %.1f\n", s1.name, s1.gpa);   // Alice: 3.9
s1.age++;
```

**struct layout in memory** — members are laid out in the order declared, but the compiler inserts *padding bytes* between members to ensure alignment. integers typically need to start at addresses divisible by their size:

```c
typedef struct {
    char a;    // 1 byte at offset 0
               // 3 bytes padding (to align next int to 4-byte boundary)
    int  b;    // 4 bytes at offset 4
    char c;    // 1 byte at offset 8
               // 3 bytes padding (to make total size multiple of 4)
} S;

printf("%zu\n", sizeof(S));   // 12, not 6
```

order matters for size:
```c
typedef struct {
    int  b;    // 4 bytes at offset 0
    char a;    // 1 byte at offset 4
    char c;    // 1 byte at offset 5
               // 2 bytes padding
} S2;

printf("%zu\n", sizeof(S2));   // 8 — smaller, same data
```

rule for minimizing size: arrange members from largest to smallest. this matters when you have huge arrays of structs or when you're mapping binary file formats.

### Structs and Pointers

passing a struct by value copies the entire struct. for large structs this is expensive. pass by pointer:

```c
void birthday(Student *s) {
    s->age++;            // arrow operator: (*s).age
    // s->age is shorthand for (*s).age
}

Student st = {"Charlie", 20, 3.7, 1003};
birthday(&st);
printf("%d\n", st.age);   // 21
```

the arrow operator `->` dereferences the pointer and accesses the member. `s->age` is exactly equivalent to `(*s).age`.

### Nested Structs

```c
typedef struct {
    double x, y;
} Point;

typedef struct {
    Point  center;
    double radius;
    int    id;
} Circle;

Circle c = {{3.0, 4.0}, 5.0, 1};
printf("center: (%.1f, %.1f)\n", c.center.x, c.center.y);

Circle *cp = &c;
cp->center.x = 0.0;    // modify nested member through pointer
```

### Self-Referential Structs — Linked Lists and Trees

a struct can contain a pointer to a struct of the same type. this is how linked lists, trees, and graphs are built:

```c
typedef struct Node {
    int         data;
    struct Node *next;    // must say "struct Node", can't use typedef here
                          // because the typedef isn't complete until the end
} Node;

// build: 1 -> 2 -> 3 -> NULL
Node *head = malloc(sizeof(Node));
head->data = 1;
head->next = malloc(sizeof(Node));
head->next->data = 2;
head->next->next = malloc(sizeof(Node));
head->next->next->data = 3;
head->next->next->next = NULL;

// traverse
for (Node *curr = head; curr != NULL; curr = curr->next) {
    printf("%d ", curr->data);
}

// free — MUST save next before freeing current
Node *curr = head;
while (curr != NULL) {
    Node *next = curr->next;   // save before free
    free(curr);
    curr = next;
}
```

**why save next before freeing?** once you call `free(curr)`, `curr`'s memory is released and might be overwritten immediately. accessing `curr->next` after `free(curr)` is use-after-free — undefined behavior. save the next pointer first.

### Flexible Array Members (C99)

a struct with an unknown-size array at the end:

```c
typedef struct {
    size_t length;
    int    data[];   // flexible array member — must be last
} IntBuffer;

// allocate header + n ints in one allocation
size_t n = 10;
IntBuffer *buf = malloc(sizeof(IntBuffer) + n * sizeof(int));
buf->length = n;
buf->data[0] = 42;

free(buf);   // one free for everything — clean
```

useful for variable-length messages, buffers with metadata, protocol headers.

### Unions — Overlapping Memory

all members of a union share the same memory location. the size of a union equals the size of its largest member:

```c
union Data {
    int    i;
    float  f;
    double d;
    char   str[8];
};

union Data u;
printf("size: %zu\n", sizeof(u));   // 8 (size of largest member — double)

u.i = 42;
printf("%d\n", u.i);   // 42

u.f = 3.14f;
printf("%f\n", u.f);   // 3.14
printf("%d\n", u.i);   // GARBAGE — u.f overwrote the same bytes
```

unions exist for two main reasons:

**1. Tagged unions (discriminated unions)** — a value that can be one of several types:
```c
typedef enum { TYPE_INT, TYPE_FLOAT, TYPE_STRING } ValType;

typedef struct {
    ValType type;
    union {
        int    i;
        float  f;
        char   s[64];
    } data;
} Value;

Value v;
v.type = TYPE_INT;
v.data.i = 42;

if (v.type == TYPE_INT)   printf("int: %d\n",   v.data.i);
if (v.type == TYPE_FLOAT) printf("float: %f\n", v.data.f);
```

**2. Type punning** — reinterpreting the same bytes as a different type:
```c
union FloatBits {
    float    f;
    uint32_t bits;
};

union FloatBits fb;
fb.f = 3.14f;
printf("bits of 3.14f: 0x%08X\n", fb.bits);   // 0x4048F5C3
```

this is the correct way to do type punning in C (using a union). using pointer casts for the same thing violates strict aliasing rules and is technically undefined behavior.

### Enums — Named Integer Constants

enums give names to a sequence of integer constants:

```c
typedef enum {
    MON = 0,   // explicitly set — unnecessary since 0 is default
    TUE,       // 1
    WED,       // 2
    THU,       // 3
    FRI,       // 4
    SAT,       // 5
    SUN        // 6
} Weekday;

typedef enum {
    HTTP_OK           = 200,
    HTTP_CREATED      = 201,
    HTTP_NOT_FOUND    = 404,
    HTTP_SERVER_ERROR = 500
} HttpStatus;

Weekday today = WED;
if (today == WED || today == THU) printf("midweek\n");

HttpStatus status = HTTP_NOT_FOUND;
printf("status: %d\n", status);   // 404
```

enums are just ints. there's no type safety in C (unlike C++'s `enum class`). you can assign any integer to an enum variable and the compiler won't complain.

benefits over plain `#define` constants: enums show up in debuggers (you see `WED` not `2`), they're grouped together visually, and some compilers warn about missing cases in a switch.

---

## File I/O

C handles files through `FILE*` — an opaque type defined in `<stdio.h>` that represents an open file.

### Opening and Closing Files

```c
FILE *fp = fopen("data.txt", "r");   // open for reading
if (fp == NULL) {
    perror("fopen");   // prints: "fopen: No such file or directory" (from errno)
    return 1;
}

// ... use the file ...

fclose(fp);   // always close it. always.
```

**always check fopen's return value.** it returns NULL if the file doesn't exist, if permissions deny access, or if the OS is out of file descriptors. proceeding with a NULL file pointer causes a crash.

**always call fclose.** open file descriptors are a finite OS resource. more importantly, output may be buffered — `fclose` flushes the buffer and ensures all data is actually written to disk.

**mode strings:**

| Mode | Meaning |
|------|---------|
| `"r"` | read — file must exist |
| `"w"` | write — creates or truncates to zero |
| `"a"` | append — creates or writes to end |
| `"r+"` | read and write — file must exist |
| `"w+"` | read and write — creates or truncates |
| `"a+"` | read and append |
| `"rb"`, `"wb"` | binary mode (important on Windows — doesn't translate line endings) |

### Reading Text Files

**character by character:**
```c
int c;   // int, not char — EOF is -1, doesn't fit in char on some platforms
while ((c = fgetc(fp)) != EOF) {
    putchar(c);
}
```

**line by line:**
```c
char line[1024];
while (fgets(line, sizeof(line), fp) != NULL) {
    printf("%s", line);   // fgets keeps the '\n'
}
```

`fgets` reads at most `sizeof(line) - 1` characters plus a `\0`. it stops at newline (keeping it) or EOF. returns NULL at EOF or error.

**formatted data:**
```c
int id;
char name[50];
float score;

// returns number of items successfully read
// loop while it keeps reading the expected number of items (3)
while (fscanf(fp, "%d %49s %f", &id, name, &score) == 3) {
    printf("%d %s %.1f\n", id, name, score);
}
```

**whole file into memory:**
```c
char *read_entire_file(const char *path, size_t *out_size) {
    FILE *fp = fopen(path, "rb");
    if (!fp) return NULL;

    // get file size
    fseek(fp, 0, SEEK_END);
    long size = ftell(fp);
    rewind(fp);

    if (size < 0) { fclose(fp); return NULL; }

    char *buf = malloc((size_t)size + 1);  // +1 for null terminator
    if (!buf) { fclose(fp); return NULL; }

    size_t read = fread(buf, 1, (size_t)size, fp);
    fclose(fp);

    if (read != (size_t)size) { free(buf); return NULL; }

    buf[size] = '\0';
    if (out_size) *out_size = (size_t)size;
    return buf;   // caller must free
}
```

### Writing Text Files

```c
FILE *fp = fopen("output.txt", "w");
if (!fp) { perror("fopen"); return 1; }

fprintf(fp, "Hello %s! Score: %d\n", name, score);   // like printf to a file
fputs("another line\n", fp);                           // write string
fputc('X', fp);                                        // write single char

fclose(fp);
```

### Binary Files

for non-text data — images, audio, custom formats, serialized structs:

```c
struct Record {
    int id;
    float value;
    char name[32];
};

// write
struct Record records[5] = { ... };
FILE *fp = fopen("data.bin", "wb");
size_t written = fwrite(records, sizeof(struct Record), 5, fp);
// writes 5 records. each record is sizeof(struct Record) bytes.
fclose(fp);

// read back
struct Record loaded[5];
fp = fopen("data.bin", "rb");
size_t count = fread(loaded, sizeof(struct Record), 5, fp);
fclose(fp);
```

**warning:** binary files with structs have portability issues. struct padding, integer endianness, and floating-point format can differ between systems. for portable binary formats, use fixed-width types and explicit serialization.

### File Navigation

```c
fseek(fp, 0, SEEK_SET);     // go to start of file
fseek(fp, 0, SEEK_END);     // go to end
fseek(fp, -10, SEEK_CUR);   // go back 10 bytes from current position
fseek(fp, 100, SEEK_SET);   // go to byte 100

long pos = ftell(fp);        // get current position in bytes
rewind(fp);                  // equivalent to fseek(fp, 0, SEEK_SET)

int at_end = feof(fp);       // non-zero if at end of file
int error = ferror(fp);      // non-zero if an error occurred
```

### Error Handling with perror and errno

`perror` prints an error message based on the current value of `errno` (a global error code set by system calls):

```c
FILE *fp = fopen("nonexistent.txt", "r");
if (!fp) {
    perror("fopen");
    // prints: "fopen: No such file or directory"
    // format: "prefix: error description based on errno"
}
```

---

## The Preprocessor

the preprocessor runs before the compiler. it does purely textual manipulation — no type checking, no scoping, no real language features. every line starting with `#` is a preprocessor directive.

### #include — Paste File Contents

```c
#include <stdio.h>     // search system include directories
#include "myheader.h"  // search current directory first, then system
```

literally pastes the file contents at that location. if `stdio.h` is 500 lines, your file effectively becomes 500 lines longer before the compiler sees it.

### #define — Text Substitution

```c
#define PI        3.14159265358979
#define MAX_BUF   1024
#define GREETING  "Hello, World!"

// usage: every occurrence of PI is replaced with 3.14159265358979
double area = PI * r * r;
// becomes:
double area = 3.14159265358979 * r * r;
```

**function-like macros:**
```c
#define SQUARE(x)         ((x) * (x))
#define MAX(a, b)         ((a) > (b) ? (a) : (b))
#define ARRAY_LEN(arr)    (sizeof(arr) / sizeof((arr)[0]))
```

**always parenthesize everything in macros:**
```c
#define BAD_SQUARE(x)  x * x
#define GOOD_SQUARE(x) ((x) * (x))

BAD_SQUARE(1 + 2)   // expands to: 1 + 2 * 1 + 2 = 5  WRONG
GOOD_SQUARE(1 + 2)  // expands to: ((1 + 2) * (1 + 2)) = 9  CORRECT
```

the preprocessor doesn't know about operator precedence — it's pure text substitution. parentheses make the expansion correct regardless of context.

**macro gotchas:**

double evaluation — a macro argument that has side effects gets evaluated multiple times:
```c
MAX(i++, j++)    // expands to: ((i++) > (j++) ? (i++) : (j++))
                 // i or j is incremented TWICE — not what you want
```

this is why `inline` functions (C99) are safer for function-like macros — they evaluate arguments once:
```c
static inline int max(int a, int b) { return a > b ? a : b; }
```

### Conditional Compilation

include or exclude code at compile time:

```c
#ifdef DEBUG
    printf("debug: x = %d\n", x);
#endif

#ifndef BUFFER_SIZE
    #define BUFFER_SIZE 512
#endif

#if defined(__linux__)
    // Linux-specific code
#elif defined(__APPLE__)
    // macOS-specific code
#elif defined(_WIN32)
    // Windows-specific code
#else
    #error "Unsupported platform"
#endif
```

**compile with -D to define macros from command line:**
```bash
gcc -DDEBUG -o program program.c   # defines DEBUG
```

### Predefined Macros

```c
__FILE__      // current filename as a string: "main.c"
__LINE__      // current line number as int: 42
__DATE__      // compilation date: "Mar 14 2026"
__TIME__      // compilation time: "14:30:00"
__func__      // current function name (C99): "main"
__STDC__      // 1 if standard C
__STDC_VERSION__  // C standard version: 201710L for C17
```

useful for debug logging:
```c
#define LOG(fmt, ...) \
    fprintf(stderr, "[%s:%d %s] " fmt "\n", \
            __FILE__, __LINE__, __func__, ##__VA_ARGS__)

LOG("entering loop");
LOG("x = %d, y = %d", x, y);
// prints: [main.c:42 process_data] x = 5, y = 10
```

### Stringification and Token Pasting

```c
#define STRINGIFY(x)  #x    // turns x into a string literal
#define CONCAT(a, b)  a##b  // concatenates two tokens

STRINGIFY(hello)    // "hello"
STRINGIFY(42)       // "42"
CONCAT(foo, bar)    // foobar (as an identifier)

// useful for debugging:
#define PRINT_VAR(x) printf(#x " = %d\n", x)
int count = 42;
PRINT_VAR(count);   // prints: count = 42
```

---

## Header Files and Multi-File Projects

as programs grow, splitting across multiple `.c` files becomes necessary. header files (`.h`) are the glue.

### The Pattern

**vec2.h** — declarations. the public interface. what other files need to know:
```c
#ifndef VEC2_H
#define VEC2_H

// type definitions
typedef struct {
    double x, y;
} Vec2;

// function declarations (prototypes)
Vec2   vec2_add(Vec2 a, Vec2 b);
Vec2   vec2_scale(Vec2 v, double s);
double vec2_length(Vec2 v);
void   vec2_print(Vec2 v);

// constants
#define VEC2_ZERO  ((Vec2){0.0, 0.0})

#endif  // VEC2_H
```

**vec2.c** — definitions. the actual implementation:
```c
#include "vec2.h"
#include <stdio.h>
#include <math.h>

Vec2 vec2_add(Vec2 a, Vec2 b) {
    return (Vec2){a.x + b.x, a.y + b.y};
}

Vec2 vec2_scale(Vec2 v, double s) {
    return (Vec2){v.x * s, v.y * s};
}

double vec2_length(Vec2 v) {
    return sqrt(v.x*v.x + v.y*v.y);
}

void vec2_print(Vec2 v) {
    printf("(%.2f, %.2f)\n", v.x, v.y);
}
```

**main.c** — uses it:
```c
#include <stdio.h>
#include "vec2.h"

int main(void) {
    Vec2 a = {3.0, 4.0};
    Vec2 b = {1.0, 2.0};
    Vec2 sum = vec2_add(a, b);
    vec2_print(sum);                             // (4.00, 6.00)
    printf("length: %.2f\n", vec2_length(a));   // 5.00
    return 0;
}
```

compile:
```bash
gcc -Wall -Wextra -std=c17 main.c vec2.c -o program -lm
# -lm links the math library (needed for sqrt)
```

### Include Guards

without include guards, including the same header twice (common in large projects) causes duplicate declarations:

```c
#ifndef VEC2_H
#define VEC2_H

// ... contents ...

#endif
```

how it works: first time `vec2.h` is included, `VEC2_H` isn't defined, so the contents are processed. `VEC2_H` gets defined. second time, `VEC2_H` is already defined, so the `#ifndef` block is skipped entirely.

modern alternative (not in the C standard but universally supported):
```c
#pragma once
// contents...
```

simpler and can't have typos. use either one consistently.

### static and extern — Controlling Visibility

**`static` on a function or global variable** restricts its visibility to the current file. it won't be accessible from other `.c` files, even if they try to declare it with `extern`:

```c
// internal.c
static void helper(void) { ... }   // only visible in internal.c
static int counter = 0;            // only visible in internal.c

void public_function(void) {       // visible everywhere (no static)
    helper();
    counter++;
}
```

use `static` for implementation details that don't belong in the public API. it prevents name collisions across files and makes the module's interface clearer.

**`extern`** declares that a variable or function exists in another file:

```c
// config.c
int debug_level = 0;   // defined here

// main.c
extern int debug_level;  // declaration — "this exists somewhere else"
debug_level = 2;         // modify it
```

for functions, `extern` is usually implicit — function declarations are already treated as extern. you mostly use `extern` for global variables shared across files (though global mutable state is generally something to avoid).

### Makefiles

```makefile
CC     = gcc
CFLAGS = -Wall -Wextra -std=c17 -g
LIBS   = -lm

program: main.o vec2.o
	$(CC) main.o vec2.o -o program $(LIBS)

main.o: main.c vec2.h
	$(CC) $(CFLAGS) -c main.c

vec2.o: vec2.c vec2.h
	$(CC) $(CFLAGS) -c vec2.c

clean:
	rm -f *.o program

.PHONY: clean
```

`make` builds only what changed. if you modify `vec2.c`, only `vec2.o` is recompiled and the final link step reruns. `main.o` is untouched. for large projects this matters — you don't want to recompile 200 files because you changed one.

---

## Modern C

### C99 — the big update (1999)

**`//` single-line comments** — didn't exist in original C89. only `/* */` was available.

**declare variables anywhere** — C89 required all variable declarations at the top of a block before any other statements:
```c
// C89 — must declare at top
void foo(void) {
    int i, j, result;
    /* ... */
    for (i = 0; i < 10; i++) { }
}

// C99 — declare when you need them
void foo(void) {
    for (int i = 0; i < 10; i++) {   // i declared right here
        int square = i * i;           // fine
    }
}
```

**`<stdbool.h>` — boolean type:**
```c
#include <stdbool.h>
bool found = false;
bool valid = true;
// true = 1, false = 0 underneath
```

**compound literals** — create anonymous struct or array values inline:
```c
typedef struct { int x, y; } Point;

void draw_point(Point p) { printf("(%d,%d)\n", p.x, p.y); }

draw_point((Point){3, 4});        // pass a struct literal directly
draw_point((Point){.x=5, .y=7}); // with designated initializers

int *arr = (int[]){1, 2, 3, 4};  // anonymous array
```

**designated initializers** for arrays:
```c
int arr[10] = {[0] = 1, [5] = 10, [9] = 99};
// [0] = 1, [1-4] = 0, [5] = 10, [6-8] = 0, [9] = 99
```

**`restrict` keyword** — tells the compiler two pointers don't alias (point to overlapping memory). enables better optimization:
```c
void copy(int * restrict dest, const int * restrict src, size_t n) {
    for (size_t i = 0; i < n; i++) dest[i] = src[i];
}
// compiler can optimize this more aggressively knowing dest and src don't overlap
```

**`<stdint.h>` and `<inttypes.h>`** — exact-width types (covered earlier) and format macros:
```c
#include <inttypes.h>
int64_t val = 1234567890LL;
printf("%" PRId64 "\n", val);   // PRId64 expands to the right format specifier
```

**variable-length arrays** — array size can be a runtime variable (covered earlier).

**`__func__`** — predefined identifier with current function name.

### C11 additions (2011)

**`_Static_assert` / `static_assert`** — compile-time assertions:
```c
#include <assert.h>

static_assert(sizeof(int) == 4, "this code assumes 32-bit ints");
static_assert(sizeof(void*) == 8, "this code requires a 64-bit system");
// if false: COMPILE ERROR with your message
// not a runtime crash. a compiler error.
```

invaluable for catching platform assumption violations at compile time.

**`_Noreturn` / `noreturn`** — marks functions that never return:
```c
#include <stdnoreturn.h>

noreturn void fatal(const char *msg) {
    fprintf(stderr, "FATAL: %s\n", msg);
    exit(1);
    // compiler knows control never reaches here
    // enables better optimization and better warnings
}
```

**`_Generic`** — compile-time type-based selection. enables type-safe macros:
```c
#define print_val(x) _Generic((x),    \
    int:    printf("%d\n",    x),      \
    float:  printf("%f\n",    x),      \
    double: printf("%lf\n",   x),      \
    char*:  printf("%s\n",    x),      \
    default: printf("unknown\n")       \
)

print_val(42);       // prints int: 42
print_val(3.14);     // prints double: 3.140000
print_val("hello");  // prints string: hello
```

the compiler picks the matching branch based on the type of `x`. unchosen branches are eliminated entirely — they don't even need to compile.

**anonymous struct/union members (C11):**
```c
typedef struct {
    int type;
    union {         // anonymous — members accessed directly
        int   i;
        float f;
    };
} Value;

Value v;
v.type = 0;
v.i = 42;    // direct access, no v.unnamed_union.i
```

**`<threads.h>` — standard threading:**
```c
#include <threads.h>

int worker(void *arg) {
    printf("thread running\n");
    return thrd_success;
}

thrd_t t;
thrd_create(&t, worker, NULL);
thrd_join(t, NULL);
```

**`<stdatomic.h>` — atomic operations:**
```c
#include <stdatomic.h>

atomic_int counter = 0;

// these are safe to call from multiple threads simultaneously:
atomic_fetch_add(&counter, 1);
int val = atomic_load(&counter);
atomic_store(&counter, 0);
```

### C17 (2017)

C17 was a bugfix/clarification release. the main practical takeaway: `-std=c17` is the current stable target. use it.

### C23 (2023)

**`nullptr`** — dedicated null pointer constant with its own type:
```c
int *p = nullptr;   // cleaner than NULL, has type nullptr_t
```

**`#embed`** — embed file contents at compile time:
```c
const unsigned char image_data[] = {
#embed "logo.png"   // raw bytes of the PNG baked into the binary
};
const size_t image_size = sizeof(image_data);
```

**`typeof` / `typeof_unqual`:**
```c
int x = 5;
typeof(x) y = 10;   // y is the same type as x (int)

// safe swap macro — works for any type
#define SWAP(a, b) do {          \
    typeof(a) _tmp = (a);        \
    (a) = (b);                   \
    (b) = _tmp;                  \
} while(0)
```

**`[[nodiscard]]`, `[[deprecated]]`, `[[maybe_unused]]`** — standard attributes:
```c
[[nodiscard]] int important_result(void) { return 42; }
// compiler warns if caller ignores the return value

[[deprecated("use new_func() instead")]]
void old_func(void) { }

void process([[maybe_unused]] int debug_param) {
    // no warning about unused parameter
}
```

**binary integer literals (official):**
```c
int flags = 0b10110100;   // was a GCC extension, now C23 standard
```

---

## Undefined Behavior

this section gets its own chapter because it's genuinely the scariest and most misunderstood aspect of C.

### What Undefined Behavior Means

the C standard explicitly says that certain operations have "undefined behavior" (UB). this doesn't mean "implementation-specific behavior" or "usually crashes". it means: the standard makes absolutely no promise about what happens. the program can:
- produce the "expected" result
- produce a different but consistent result
- produce a random result
- crash
- corrupt memory silently
- format your hard drive (technically permitted under the standard)
- work perfectly in debug builds and fail in optimized builds

and here's what makes it genuinely dangerous: **the compiler can assume undefined behavior never happens.** optimizations routinely exploit this assumption. code that appears to do one thing in an unoptimized build does something completely different when compiled with `-O2`.

### Common Forms of Undefined Behavior

**1. Signed integer overflow:**
```c
int x = INT_MAX;   // 2147483647
int y = x + 1;     // UNDEFINED BEHAVIOR — not guaranteed to give -2147483648
```

the compiler, knowing that signed overflow is UB, can assume `x + 1 > x` is always true — eliminating bounds checks, loop conditions, and other code that depends on overflow happening.

**2. Out-of-bounds array access:**
```c
int arr[5];
arr[5] = 10;    // UB — might corrupt adjacent memory
arr[-1] = 0;    // UB — writes before the array
```

**3. Dereferencing a null or invalid pointer:**
```c
int *p = NULL;
*p = 5;            // UB — always a crash in practice, but UB per standard

int *q = (int*)0xDEADBEEF;  // random invalid address
*q = 5;            // UB — crash
```

**4. Use after free:**
```c
int *p = malloc(sizeof(int));
free(p);
*p = 42;    // UB — memory was freed, might be reused, might be corrupted
```

**5. Reading uninitialized variables:**
```c
int x;
printf("%d\n", x);   // UB — x's value is indeterminate
```

**6. Accessing a string beyond its terminator:**
```c
char s[] = "hi";   // s[0]='h', s[1]='i', s[2]='\0'
printf("%c\n", s[3]);   // UB — reads past the string
```

**7. Modifying a string literal:**
```c
char *s = "hello";
s[0] = 'H';   // UB — string literal is in read-only memory
```

**8. Violating strict aliasing:**
```c
int x = 42;
float *fp = (float *)&x;   // UB — accessing int through float pointer
float val = *fp;           // strictly aliasing violation
```

**9. Integer operations — shift amount out of range:**
```c
int x = 1;
int y = x << 32;   // UB if shifting by >= width of the type
```

**10. Data races in multi-threaded code:**
```c
// Thread 1: x = 1;
// Thread 2: y = x;
// without synchronization — UB
```

### The Real-World Impact — an Example

```c
int table[4] = {0, 1, 2, 3};

int f(int i) {
    if (table[i] == 0) return 0;
    return table[i];
}
```

compiled with `gcc -O2`, some compilers may transform this because: `table[i] == 0` and `table[i]` both read the same array element. without any modification between them, they must be the same value. so `if (table[i] == 0) return 0; return table[i]` becomes `return table[i]`. the bounds check is gone.

but if `i` is out of bounds (UB), the compiler was allowed to do this transformation because it assumed UB doesn't happen.

### How to Detect UB

use `-fsanitize=address,undefined` during development. it instruments your code to detect these at runtime:

```bash
gcc -fsanitize=address,undefined -g -o program program.c
./program
```

when UB occurs, it prints exactly what happened and on which line. use this on every project. remove it for production builds.

---

## Error Handling

C doesn't have exceptions. error handling is manual. here are the patterns.

### errno — the global error code

system calls and many standard library functions set `errno` (from `<errno.h>`) when they fail. check it immediately after a failure:

```c
#include <errno.h>
#include <string.h>

FILE *fp = fopen("missing.txt", "r");
if (fp == NULL) {
    // errno is set to ENOENT (no such file or directory)
    printf("error %d: %s\n", errno, strerror(errno));
    // strerror converts error code to readable message
    perror("fopen");   // prints "fopen: No such file or directory"
}
```

**important:** errno is only meaningful immediately after a failing function. any subsequent function call might change it. save it if you need it later:

```c
int saved_errno = errno;  // save before calling anything else
```

common errno values:
- `ENOENT` — no such file or directory
- `EACCES` — permission denied
- `ENOMEM` — out of memory
- `EINVAL` — invalid argument
- `EEXIST` — file already exists
- `ENOTSUP` — operation not supported
- `ETIMEDOUT` — connection timed out

### Return Value Patterns

most C functions signal errors through their return value:

```c
// NULL on failure (pointer-returning functions)
FILE *fp = fopen(path, "r");
if (!fp) { /* failed */ }

// negative on failure (POSIX style)
int fd = open(path, O_RDONLY);
if (fd < 0) { /* failed, check errno */ }

// 0 on success, non-zero on failure
int result = pthread_create(&thread, NULL, func, arg);
if (result != 0) { fprintf(stderr, "error: %s\n", strerror(result)); }

// special sentinel values (EOF = -1 for fgetc)
int c = fgetc(fp);
if (c == EOF) { /* end of file or error */ }
```

### Custom Error Codes

define an enum for your error codes:

```c
typedef enum {
    ERR_OK           =  0,
    ERR_INVALID_ARG  = -1,
    ERR_OUT_OF_MEM   = -2,
    ERR_NOT_FOUND    = -3,
    ERR_PERMISSION   = -4,
    ERR_IO           = -5,
} Error;

const char *error_string(Error e) {
    switch (e) {
        case ERR_OK:           return "success";
        case ERR_INVALID_ARG:  return "invalid argument";
        case ERR_OUT_OF_MEM:   return "out of memory";
        case ERR_NOT_FOUND:    return "not found";
        case ERR_PERMISSION:   return "permission denied";
        case ERR_IO:           return "I/O error";
        default:               return "unknown error";
    }
}

Error load_config(const char *path, Config *out) {
    if (!path || !out) return ERR_INVALID_ARG;

    FILE *fp = fopen(path, "r");
    if (!fp) return ERR_NOT_FOUND;

    // ... parse config ...
    fclose(fp);
    return ERR_OK;
}

Error err = load_config("config.txt", &cfg);
if (err != ERR_OK) {
    fprintf(stderr, "load_config failed: %s\n", error_string(err));
    return 1;
}
```

### assert — for programmer errors

`assert(condition)` checks a condition that should never be false. if it is, the program prints the failing expression, file, and line number, then calls `abort()`:

```c
#include <assert.h>

void process(int *data, int len) {
    assert(data != NULL);     // programmer error if NULL passed
    assert(len > 0);          // programmer error if non-positive length
    // ...
}
```

asserts are for *invariants* — things that should never happen in correct code. they're documentation that compiles and checks itself. define `NDEBUG` to disable them in release builds: `gcc -DNDEBUG ...`.

never use assert for user input validation — the user can trigger them legitimately. asserts are for conditions that indicate your code itself has a bug.

### Cleanup with goto

when a function acquires multiple resources and an error occurs partway through, goto provides clean cleanup:

```c
int load_and_process(const char *path) {
    FILE *fp = NULL;
    char *buf = NULL;
    int *data = NULL;
    int result = -1;

    fp = fopen(path, "r");
    if (!fp) goto cleanup;

    buf = malloc(BUFFER_SIZE);
    if (!buf) goto cleanup;

    data = malloc(MAX_ITEMS * sizeof(int));
    if (!data) goto cleanup;

    // ... do work ...
    result = 0;  // success

cleanup:
    free(data);   // free(NULL) is safe — always fine even if not allocated
    free(buf);
    if (fp) fclose(fp);
    return result;
}
```

this pattern is common in the linux kernel. `free(NULL)` is defined to be a no-op, so you can call it unconditionally.

---

## Bit Manipulation Deep Dive

### Two's Complement in Depth

all modern computers use two's complement for signed integers. knowing this explains a lot of C's behavior.

for an n-bit number, the range is `-2^(n-1)` to `2^(n-1) - 1`. for 32-bit int: -2,147,483,648 to 2,147,483,647.

the key property: negation = flip all bits + 1.

```
 5 in 32-bit: 00000000 00000000 00000000 00000101
flip all:     11111111 11111111 11111111 11111010
add 1:        11111111 11111111 11111111 11111011
= -5
```

consequences you'll hit:
- `INT_MIN` has no positive counterpart: `-INT_MIN == INT_MIN` (overflow)
- `-1` as unsigned int: all bits set = `UINT_MAX`
- signed right shift of negative fills with 1s (sign extension) on most machines, but is implementation-defined in C

### Useful Bit Tricks

```c
// check if n is a power of 2
bool is_power_of_two(unsigned n) {
    return n > 0 && (n & (n - 1)) == 0;
}
// explanation: powers of 2 have exactly one bit set
// 0b1000 & 0b0111 = 0 — AND is zero for any power of 2

// next power of 2 >= n
uint32_t next_pow2(uint32_t n) {
    n--;
    n |= n >> 1;
    n |= n >> 2;
    n |= n >> 4;
    n |= n >> 8;
    n |= n >> 16;
    return n + 1;
}

// count set bits (popcount / Hamming weight)
int count_bits(unsigned n) {
    int count = 0;
    while (n) {
        count += n & 1;
        n >>= 1;
    }
    return count;
}
// gcc/clang built-in (single CPU instruction): __builtin_popcount(n)

// get, set, clear, toggle a specific bit
int bit_get(int val, int pos)    { return (val >> pos) & 1; }
int bit_set(int val, int pos)    { return val |  (1 << pos); }
int bit_clear(int val, int pos)  { return val & ~(1 << pos); }
int bit_toggle(int val, int pos) { return val ^  (1 << pos); }

// extract a field: bits from position 'start', of length 'len'
int bit_extract(int val, int start, int len) {
    return (val >> start) & ((1 << len) - 1);
}

// rotate bits left (no built-in in C, but common pattern)
uint32_t rotate_left(uint32_t val, int n) {
    return (val << n) | (val >> (32 - n));
}

// XOR swap — swap two variables without a temp
a ^= b;
b ^= a;
a ^= b;
// works because: a^a = 0 and a^0 = a
// not actually useful today — compilers handle swap better

// check if two integers have opposite signs
bool opposite_signs(int a, int b) {
    return (a ^ b) < 0;   // XOR of opposite signs has the sign bit set
}

// absolute value without branching
int abs_val(int n) {
    int mask = n >> 31;         // all 1s if negative, all 0s if positive
    return (n + mask) ^ mask;   // if negative: two's complement negation; else: identity
}
```

### Bit Fields in Structs

structs can have members with specified bit widths:

```c
typedef struct {
    unsigned int red   : 5;   // 5 bits — 0-31
    unsigned int green : 6;   // 6 bits — 0-63  (green gets more bits because human eye)
    unsigned int blue  : 5;   // 5 bits — 0-31
} RGB565;                      // total: 16 bits = 2 bytes

RGB565 color;
color.red   = 31;   // max red
color.green = 63;   // max green
color.blue  = 0;    // no blue

printf("size: %zu\n", sizeof(RGB565));   // probably 4 (alignment)
```

bit fields are useful for hardware registers, packed protocol headers, and memory-constrained embedded systems. caveats: layout is implementation-defined, can't take the address of a bit field, padding between fields is possible.

---

## Common Pitfalls

### Uninitialized Variables

```c
int sum;
for (int i = 0; i < 5; i++) sum += i;   // sum starts as garbage + 0 + 1 + ...
// fix:
int sum = 0;
```

### Integer Division Truncation

```c
double avg = total / count;   // if both int, integer division happens first
double avg = (double)total / count;   // cast BEFORE dividing
```

### Off-by-One

```c
char str[] = "hello";  // indices 0-4 for chars, 5 is '\0'
for (int i = 0; i <= 5; i++) { }   // i=5 accesses '\0', i=6 is UB
for (int i = 0; i < 5; i++) { }    // correct

int arr[5];
for (int i = 0; i <= 5; i++) arr[i] = 0;    // arr[5] is out of bounds
for (int i = 0; i < 5; i++) arr[i] = 0;     // correct
```

### strlen in Loop Condition

```c
for (int i = 0; i < strlen(str); i++) { }   // strlen is O(n), called every iteration
                                              // loop becomes O(n^2)
size_t len = strlen(str);
for (size_t i = 0; i < len; i++) { }        // compute once
```

### Signed/Unsigned Comparison

```c
int i = -1;
if (i < sizeof(arr)) { }   // WARNING — sizeof returns size_t (unsigned)
                             // -1 converted to unsigned = huge number
                             // comparison is wrong

if ((size_t)i < sizeof(arr)) { }  // correct
if (i >= 0 && (size_t)i < sizeof(arr)) { }  // if i could be negative
```

### scanf Leaves Newlines in Buffer

```c
int n;
char buf[50];
scanf("%d", &n);            // reads "42\n" — consumes "42", leaves "\n"
fgets(buf, sizeof(buf), stdin);   // immediately reads the leftover "\n"

// fix: consume the newline:
scanf("%d", &n);
getchar();                  // consume '\n'
fgets(buf, sizeof(buf), stdin);   // now reads actual next line

// or use:
scanf(" %49[^\n]", buf);   // the leading space skips whitespace including '\n'
```

### Returning Pointer to Local

```c
int *wrong(void) {
    int local = 42;
    return &local;   // local is on the stack frame, which is destroyed on return
}
// using the returned pointer = undefined behavior

int *correct(void) {
    int *heap = malloc(sizeof(int));
    *heap = 42;
    return heap;   // heap memory persists, caller must free
}
```

### strncpy Doesn't Guarantee Null Termination

```c
char buf[5];
strncpy(buf, "hello", 5);   // copies 5 chars — NO room for '\0'
printf("%s\n", buf);         // reads past buf until it finds a '\0' — UB

// fix: always null-terminate manually, or use snprintf:
snprintf(buf, sizeof(buf), "%s", "hello");   // always null-terminates
```

### Modifying String Literals

```c
char *s = "hello";
s[0] = 'H';    // crash — string literal is read-only

char s[] = "hello";  // copy into writable array
s[0] = 'H';    // fine
```

### Signed Integer Overflow

```c
int x = INT_MAX;
x++;             // UNDEFINED BEHAVIOR for signed types
                 // unsigned overflow is defined (wraps), signed is not

unsigned int y = UINT_MAX;
y++;             // 0 — defined wraparound for unsigned
```

---

## Debugging

### GDB

compile with `-g` to include debug symbols:
```bash
gcc -g -Wall -std=c17 -o program program.c
gdb ./program
```

essential GDB commands:
```
run [args]           — start the program
break main           — set breakpoint at main function
break file.c:42      — set breakpoint at line 42 of file.c
break function_name  — set breakpoint at start of function
next (n)             — execute next line (don't step into functions)
step (s)             — execute next line (step INTO functions)
finish               — run until current function returns
continue (c)         — resume until next breakpoint
print x              — print value of variable x
print *ptr           — print value pointed to by ptr
print arr[0]         — print array element
print arr[0]@5       — print 5 elements starting at arr[0]
info locals          — print all local variables in current frame
backtrace (bt)       — show call stack (excellent after crashes)
up / down            — move up/down in the call stack to see other frames
watch x              — pause when x's value changes
display x            — print x's value on every step
set x = 5            — modify variable while debugging
x/10xw 0x7fff0000    — examine 10 hex words at address
quit                 — exit gdb
```

**after a crash:** run the program in gdb, let it crash, then type `backtrace` to see exactly where it crashed and what the call stack looked like. usually tells you immediately what went wrong.

### AddressSanitizer + UBSan

```bash
gcc -fsanitize=address,undefined -g -o program program.c
./program
```

detects at runtime: heap overflow, stack overflow, use-after-free, use-after-return, double free, memory leaks, null dereference, signed overflow, invalid array index, misaligned access.

when triggered, prints: the error type, the line it occurred on, the allocation site (for memory bugs), and a full stack trace. use this on every project during development.

### Valgrind

```bash
valgrind --leak-check=full --show-leak-kinds=all --track-origins=yes ./program
```

slower than ASan but more thorough for leak detection. `--track-origins` shows where uninitialized values came from.

### printf Debugging

sometimes the fastest way:

```c
#define DBG(fmt, ...) \
    fprintf(stderr, "[%s:%d] " fmt "\n", __FILE__, __LINE__, ##__VA_ARGS__)

DBG("entering loop with n=%d", n);
DBG("ptr = %p, *ptr = %d", (void*)ptr, *ptr);
```

write to `stderr` not `stdout` — keeps debug output separate from normal output, and `stderr` is unbuffered so it always appears even if the program crashes before stdout flushes.

### Checking for Memory Bugs Without Tools

read your code and ask:
1. for every `malloc`/`calloc`/`realloc` — is there a corresponding `free`? on every code path?
2. after every `free` — is the pointer set to NULL or never used again?
3. for every array access — can the index be out of bounds?
4. for every pointer dereference — can the pointer be NULL here?
5. for every function that returns a pointer — is the return value checked for NULL?

---

## Questions

try to answer these before scrolling to the answers.

---

**Q1:** what's the difference between these two declarations, and what's the danger in one of them?
```c
char s1[] = "Hello";
char *s2  = "Hello";
```

---

**Q2:** what does this print?
```c
int a = 5, b = 2;
printf("%d\n",    a / b);
printf("%.2f\n",  (float)a / b);
printf("%.2f\n",  (float)(a / b));
```

---

**Q3:** what's wrong here? what exactly happens at runtime?
```c
int *get_value(void) {
    int local = 42;
    return &local;
}

int main(void) {
    int *p = get_value();
    printf("%d\n", *p);
    return 0;
}
```

---

**Q4:** explain what pointer arithmetic actually does at the byte level.
```c
int arr[] = {10, 20, 30, 40, 50};
int *p = arr;
p++;
printf("%d\n", *p);
```
what is the numeric change in the address stored in p?

---

**Q5:** find every bug in this code:
```c
int main(void) {
    int *arr = malloc(10 * sizeof(int));
    for (int i = 0; i <= 10; i++) {
        arr[i] = i * 2;
    }
    printf("arr[5] = %d\n", arr[5]);
    return 0;
}
```

---

**Q6:** what does this print? why might it print something different on a different platform?
```c
printf("%zu\n", sizeof(int));
printf("%zu\n", sizeof(long));
printf("%zu\n", sizeof(void*));
```

---

**Q7:** what is the output?
```c
int x = 10;
if (x = 0) {
    printf("zero\n");
} else {
    printf("nonzero\n");
}
printf("x is now: %d\n", x);
```

---

**Q8:** what's wrong with this loop?
```c
for (int i = 0; i < strlen(str); i++) {
    process(str[i]);
}
```

---

**Q9:** explain why this swap function doesn't work, and fix it:
```c
void swap(int a, int b) {
    int temp = a;
    a = b;
    b = temp;
}
```

---

**Q10:** what is undefined behavior? give three specific examples from C code. why is it dangerous specifically with optimization enabled?

---

**Q11:** what does the following print, and why is the behavior of one of the lines technically undefined?
```c
unsigned int x = 0;
x--;
printf("%u\n", x);

int y = INT_MAX;
y++;
printf("%d\n", y);
```

---

**Q12:** what is the difference between the stack and the heap? for each one: what's stored there, who manages it, and what are the size limits?

---

**Q13:** what happens when you call `free(NULL)`?

---

**Q14:** what's wrong with this realloc usage?
```c
int *arr = malloc(10 * sizeof(int));
arr = realloc(arr, 20 * sizeof(int));
if (arr == NULL) {
    fprintf(stderr, "realloc failed\n");
}
```

---

**Q15:** explain struct padding. how many bytes does this struct take and why?
```c
typedef struct {
    char   a;
    int    b;
    char   c;
    double d;
} S;
```

---

**Q16:** what's the difference between `const int *p`, `int * const p`, and `const int * const p`?

---

**Q17:** what does `static` mean when applied to (a) a local variable inside a function, (b) a function at file scope, (c) a global variable at file scope?

---

**Q18:** what's the output?
```c
#include <stdio.h>

int counter = 0;

void increment(void) {
    static int local_count = 0;
    local_count++;
    counter++;
    printf("local=%d global=%d\n", local_count, counter);
}

int main(void) {
    increment();
    increment();
    increment();
    return 0;
}
```

---

**Q19:** why does this not compare strings correctly? how do you fix it?
```c
char s1[] = "hello";
char s2[] = "hello";

if (s1 == s2) {
    printf("equal\n");
} else {
    printf("not equal\n");
}
```

---

**Q20:** what is RAII and why doesn't standard C have it? how do C programmers handle the same problem?

---

**Q21:** write a function `char *str_repeat(const char *s, int n)` that returns a newly-allocated string containing `s` repeated `n` times. the caller is responsible for freeing. handle all error cases.

---

---

## Answers

---

**A1:**
`char s1[] = "Hello"` — creates an array on the stack. the string literal is copied into the array at program startup. the array is writable. `s1[0] = 'h'` works fine.

`char *s2 = "Hello"` — creates a pointer that points to the string literal in the program's read-only data segment. the pointer itself is writable (you can make s2 point elsewhere), but the characters it points to are not. `s2[0] = 'h'` is undefined behavior — typically a segfault because the OS protects read-only pages. always use `const char *` for pointers to string literals.

---

**A2:**
line 1: `2`. integer division — `5 / 2` = 2 with remainder 1, decimal discarded.

line 2: `2.50`. `(float)a` converts `a` to a float *before* the division. the division then happens as float/int, which promotes int to float, giving 2.5.

line 3: `2.00` — the trap. `(a / b)` is evaluated first as integer division (= 2), then the result 2 is cast to float (= 2.0). the fractional part was already lost before the cast. the order of operations kills you here.

---

**A3:**
`local` is a variable on `get_value`'s stack frame. when `get_value` returns, its stack frame is popped — the memory that `local` occupied is reclaimed and can be used by the next function call. the returned pointer `p` now points to this reclaimed memory. dereferencing `*p` is use-after-scope — undefined behavior. in practice, it might print 42 (the stack hasn't been overwritten yet), it might print garbage, or it might crash. the fix: either allocate `local` on the heap with `malloc` and return that pointer (caller frees it), declare `local` as `static` (it then lives for the entire program), or restructure to avoid returning a pointer to a local.

---

**A4:**
`p` starts pointing to `arr[0]`. `p++` is pointer arithmetic. since `p` is an `int*` and `sizeof(int) == 4`, `p++` adds 4 bytes to the address stored in p. if `arr[0]` was at address 0x100, `p` now holds 0x104 — pointing to `arr[1]`. dereferencing `*p` gives 20. pointer arithmetic always scales by `sizeof` the pointed-to type, automatically. this is why pointer type matters: a `char*` increment moves 1 byte, an `int*` moves 4, a `double*` moves 8.

---

**A5:**
three bugs:
1. **off-by-one overflow**: `i <= 10` writes `arr[10]` — but the array has indices 0-9. index 10 is out of bounds, undefined behavior (heap buffer overflow). fix: `i < 10`.
2. **missing NULL check after malloc**: if malloc fails (returns NULL), the loop immediately crashes dereferencing NULL. fix: `if (arr == NULL) { fprintf(stderr, "malloc failed\n"); return 1; }` after the malloc.
3. **memory leak**: `free(arr)` is never called. the 40 bytes allocated are never returned to the heap. fix: add `free(arr);` before `return 0`.

---

**A6:**
typical output on a 64-bit Linux/Mac system:
```
4
8
8
```

but it's platform-dependent:
- `sizeof(int)` is typically 4 on both 32-bit and 64-bit systems, but the C standard only guarantees at least 16 bits
- `sizeof(long)` is 4 on Windows (even 64-bit), 8 on Linux/Mac 64-bit
- `sizeof(void*)` is 4 on 32-bit systems, 8 on 64-bit systems — this tells you the pointer size, i.e., the address space width

this is why `<stdint.h>` exists — to write portable code that doesn't depend on these platform-specific sizes.

---

**A7:**
prints "nonzero", then "x is now: 0". `x = 0` inside the if condition is an assignment, not a comparison. it assigns 0 to x, then the condition evaluates to 0 (zero is false), so the `else` branch runs. x is now 0. `-Wall` would warn: "suggest parentheses around assignment used as truth value". the correct code would be `if (x == 0)`.

---

**A8:**
`strlen(str)` is called on every iteration. strlen scans the entire string to find `\0` — it's O(n). calling it n times makes the loop O(n²). for a 1000-character string, that's 1,000 strlen calls each scanning up to 1000 chars — potentially 1,000,000 operations instead of 1000. fix: cache the length: `size_t len = strlen(str); for (size_t i = 0; i < len; i++) { ... }`.

---

**A9:**
C passes arguments by value. `swap` receives copies of `a` and `b`. swapping the copies has no effect on the caller's variables. fix using pointers:

```c
void swap(int *a, int *b) {
    int temp = *a;
    *a = *b;
    *b = temp;
}

// call as:
int x = 5, y = 10;
swap(&x, &y);   // pass addresses
printf("%d %d\n", x, y);   // 10 5
```

---

**A10:**
undefined behavior (UB) means the C standard makes no promise about what the program does. it can produce any result or no result. it's dangerous specifically with optimization because the compiler is allowed to assume UB never occurs, and uses this assumption to eliminate "dead code". a bounds check that would prevent UB might be removed as "unnecessary" because UB can't happen (the compiler thinks). three examples:

```c
int x = INT_MAX;
x++;   // signed overflow — UB

int arr[5];
arr[5] = 0;   // out-of-bounds write — UB

int *p = NULL;
*p = 5;   // null dereference — UB
```

with `-O2`, code like `if (x + 1 < x)` might be completely eliminated because signed overflow is UB and the compiler assumes it never happens — so the condition "can never be true".

---

**A11:**
first line (`x--`): `x` is `unsigned int` starting at 0. subtracting 1 from an unsigned integer wraps around (defined behavior for unsigned). `0 - 1 = UINT_MAX`. prints `4294967295` (or whatever UINT_MAX is on your system, typically this for 32-bit unsigned int).

second line (`y++`): `y = INT_MAX + 1` is signed integer overflow — undefined behavior. the standard makes no promise. in practice on two's complement machines (all modern ones), it wraps to INT_MIN (-2147483648 for 32-bit). but you cannot rely on this. a compiler with `-O2` might remove checks that assume this overflow happens.

---

**A12:**
**stack:** stores local variables, function parameters, return addresses, saved register values. managed automatically — space is allocated when a function is called, freed when it returns. fixed size (typically 1-8 MB set by the OS). very fast (just moving a pointer). accessing beyond its limits causes a stack overflow.

**heap:** stores dynamically allocated memory (malloc/calloc/realloc). managed manually — you call malloc to allocate, free to release. limited only by available RAM (and virtual memory). slower than stack (allocator has to find free blocks, maintain metadata). leaks if you forget to free. enables objects that outlive function calls.

---

**A13:**
it's a no-op. `free(NULL)` is explicitly defined in the C standard to do nothing. this is intentional — it means you can safely call `free` on a pointer without first checking if it's NULL. common pattern:

```c
void cleanup(Resource *r) {
    free(r->buffer);   // fine even if buffer was never allocated (is NULL)
    free(r->name);
    free(r);
}
```

no need for `if (r->buffer) free(r->buffer)` — just call free directly.

---

**A14:**
two bugs. first: if `realloc` fails, it returns NULL but does NOT free the original pointer. by doing `arr = realloc(arr, ...)`, if it returns NULL, `arr` is now NULL — the original allocation is leaked. the only pointer to it is gone. second: after the check `if (arr == NULL)`, the error is printed but execution continues with a NULL `arr` — any subsequent use of `arr` is a null dereference crash. fix:

```c
int *tmp = realloc(arr, 20 * sizeof(int));
if (tmp == NULL) {
    fprintf(stderr, "realloc failed\n");
    free(arr);   // still responsible for original
    return -1;   // or handle the error
}
arr = tmp;   // only update arr on success
```

---

**A15:**
probably 24 bytes. the compiler inserts padding to ensure each member is aligned to its natural alignment:
- `char a` (1 byte) at offset 0
- 3 bytes padding — so `int b` (4 bytes) starts at offset 4 (divisible by 4)
- `int b` (4 bytes) at offset 4
- `char c` (1 byte) at offset 8
- 7 bytes padding — so `double d` (8 bytes) starts at offset 16 (divisible by 8)
- `double d` (8 bytes) at offset 16
- total struct size: 24. the struct size is also padded to be a multiple of its largest alignment requirement (8), so 24 works.

reordering to `double d; int b; char a; char c;` gives 16 bytes — put large types first to minimize padding.

---

**A16:**
`const int *p` — pointer to const int. you CAN change `p` to point to a different int. you CANNOT modify the int it points to through `p`. read right-to-left: "p is a [pointer to] [const int]".

`int * const p` — const pointer to int. you CANNOT change `p` to point elsewhere. you CAN modify the int it points to. "p is a [const pointer] to [int]".

`const int * const p` — const pointer to const int. you can change neither `p` itself nor the int it points to.

---

**A17:**
three completely different meanings:

(a) `static` local variable: the variable has static *storage duration* — it's initialized once (at program start) and retains its value between function calls. it exists for the entire lifetime of the program, not just when the function is running.

(b) `static` function: the function has *internal linkage* — it's only visible within the current translation unit (`.c` file). other files cannot call it even if they try to declare it. used to hide implementation details.

(c) `static` global variable: same as (b) — internal linkage. the variable is only visible within the current file. without `static`, a global variable has *external linkage* and can be accessed from other files with `extern`.

---

**A18:**
```
local=1 global=1
local=2 global=2
local=3 global=3
```

`local_count` is a static local variable. it's initialized to 0 once and retains its value between calls — it increments from 1 to 2 to 3. `counter` is a global variable, also shared across all calls — also goes 1, 2, 3. both start at 0, both increment by 1 each call, both reach 3.

---

**A19:**
`s1 == s2` compares the *addresses* of the two arrays, not their contents. `s1` and `s2` are two different arrays at two different memory locations. the addresses are different, so the comparison is false — it prints "not equal" even though both contain "hello". in C, an array name decays to a pointer to its first element. so `s1 == s2` is comparing two pointers. to compare string contents, use `strcmp`:

```c
if (strcmp(s1, s2) == 0) {
    printf("equal\n");
}
```

---

**A20:**
RAII (Resource Acquisition Is Initialization) is a C++ pattern where resources are tied to object lifetimes — acquired in the constructor, released in the destructor. since destructors run automatically when objects go out of scope, cleanup is guaranteed and exception-safe. C doesn't have destructors, so RAII isn't directly available.

C programmers handle this with:
1. **discipline** — manually tracking every resource, ensuring every code path calls cleanup
2. **goto cleanup** — jump to a cleanup block at the end of the function that frees everything
3. **wrapper patterns** — function pairs like `thing_create`/`thing_destroy` and strict conventions about calling them
4. **tools** — Valgrind and AddressSanitizer to catch leaks and use-after-free

```c
int process(const char *path) {
    FILE *fp = NULL;
    char *buf = NULL;
    int result = -1;

    fp = fopen(path, "r");
    if (!fp) goto cleanup;

    buf = malloc(BUFFER_SIZE);
    if (!buf) goto cleanup;

    // ... work ...
    result = 0;

cleanup:
    free(buf);        // free(NULL) is safe
    if (fp) fclose(fp);
    return result;
}
```

---

**A21:**
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

char *str_repeat(const char *s, int n) {
    if (s == NULL || n < 0) return NULL;
    if (n == 0) {
        char *empty = malloc(1);
        if (empty) empty[0] = '\0';
        return empty;
    }

    size_t slen = strlen(s);
    size_t total = slen * (size_t)n;

    // check for overflow — slen * n might overflow size_t
    if (slen != 0 && total / slen != (size_t)n) return NULL;

    char *result = malloc(total + 1);   // +1 for null terminator
    if (result == NULL) return NULL;

    char *ptr = result;
    for (int i = 0; i < n; i++) {
        memcpy(ptr, s, slen);   // copy each repetition
        ptr += slen;
    }
    *ptr = '\0';   // null-terminate

    return result;
}

// usage:
// char *repeated = str_repeat("ab", 4);
// if (repeated) {
//     printf("%s\n", repeated);   // "abababab"
//     free(repeated);
// }
```

---

*that's C. the full picture — not just the syntax but the memory model, the undefined behavior, the compiler pipeline, everything.*

*reading gets you maybe 20% there. the rest is building things. write a linked list. implement a hash table. build a small arena allocator. write something that reads and parses a binary file. make it crash, run it under asan, fix it. that's where it actually sticks.*

*GL bro*
