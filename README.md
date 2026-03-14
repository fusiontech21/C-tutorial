# C Programming —  The Complete Guide

> *stop watching those shitty youtube tutorials trying to learn something bro. read this instead. it covers everything for real like actually everything.*

---

## Table of Contents

1. [Why C Though](#why-c-though)
2. [Setting Up](#setting-up)
3. [Your First Program](#your-first-program)
4. [Variables and Data Types](#variables-and-data-types)
5. [Operators](#operators)
6. [Control Flow](#control-flow)
7. [Functions](#functions)
8. [Arrays](#arrays)
9. [Strings](#strings)
10. [Pointers — yeah this is the one](#pointers)
11. [Memory Management](#memory-management)
12. [Structs, Unions, and Enums](#structs-unions-and-enums)
13. [File I/O](#file-io)
14. [The Preprocessor](#the-preprocessor)
15. [Header Files and Multi-File Projects](#header-files-and-multi-file-projects)
16. [Modern C — C99 through C23](#modern-c)
17. [Error Handling](#error-handling)
18. [Bit Manipulation Deep Dive](#bit-manipulation-deep-dive)
19. [Common Pitfalls](#common-pitfalls)
20. [Debugging](#debugging)
21. [Questions](#questions)
22. [Answers](#answers)

---

## Why C Though

fr though — your friend is out here doing "python for beginners day 3" and you're asking about C. different breed.

here's the actual answer: C is the language that *runs everything else*. python's interpreter? written in C. the linux kernel powering like 90% of the internet's servers? C. git, the thing you use to push to github? written in C. doom, minecraft's original engine, every OS you've ever touched — C.

when you write python or javascript you're sitting on top of like fifteen layers of abstraction hiding what's actually happening. when you write C you're one step above assembly. no garbage collector cleaning up your mess in the background, no virtual machine, no "automatic memory management". it's just you, your code, the compiler, and the CPU.

what that means in practice:
- you'll understand how memory *actually* works — not just "variables store data" but where, why, and what happens when you screw it up
- every other language will suddenly make more sense once you see what they're hiding
- pointers will make you feel lost for a week and then click and make you feel unstoppable
- you can write code that runs on a microcontroller with 2KB of RAM — smaller than your thumbnail

yeah it's harder than python. the compiler is going to yell at you nonstop at first. but when it clicks, it really clicks. let's go.

---

## Setting Up

you need three things: a compiler, an editor, a terminal. that's it.

### The Compiler

the compiler takes your C code and turns it into machine code the CPU can actually run. two main options: GCC and Clang. both are free, both are great, everything in this guide works on either.

**Linux:**
```bash
gcc --version
```
already installed 99% of the time. if not:
```bash
sudo apt install gcc    # ubuntu/debian
sudo dnf install gcc    # fedora
```

**Mac:**
```bash
xcode-select --install
```
this installs Clang, which on mac pretends to be gcc. it's fine, works identical for everything here.

**Windows:** three real options:
- **WSL** (Windows Subsystem for Linux) — runs a full linux terminal inside windows. what most people use, genuinely good
- **MinGW-w64** — GCC ported to windows natively
- **Visual Studio** — heavy, microsoft, but works well on windows

### The Editor

deadass just use **Zed**. it's fast as hell, written in Rust, has great C support out of the box, and doesn't feel like a corporate product. download it from zed.dev.

if for some reason you can't use Zed:
- VSCode with the C/C++ extension is solid and free
- Vim/Neovim if you want to feel like a wizard and spend a week learning keybinds
- CLion is a full IDE, free for students, overkill for starting out

### Compiling and Running

make a file called `hello.c`, then in your terminal:

```bash
gcc hello.c -o hello     # compile — -o names the output file
./hello                  # run (linux/mac)
hello.exe                # run (windows)
```

**always compile with these flags:**
```bash
gcc -Wall -Wextra -std=c17 -o hello hello.c
```
`-Wall` and `-Wextra` turn on extra warnings. `-std=c17` uses the 2017 C standard. these flags will catch bugs before they turn into hour-long debugging sessions. just make them a habit now.

---

## Your First Program

```c
#include <stdio.h>

int main(void) {
    printf("Hello, world!\n");
    return 0;
}
```

looks simple. let's actually break it down because "just trust the code" gets you nowhere in C.

**`#include <stdio.h>`**
before your code compiles, a preprocessor runs over it first and handles all lines starting with `#`. `stdio.h` is the standard input/output header — it contains declarations for functions like `printf` and `scanf`. without this the compiler literally doesn't know what `printf` is. it will refuse to compile. include it.

**`int main(void)`**
every C program starts at `main`. the `int` means it returns an integer to the operating system. `void` in the parentheses means it takes no arguments. writing `int main()` instead works fine in practice too.

**`printf("Hello, world!\n");`**
printf = "print formatted". the `\n` is a newline character — without it your terminal prompt appears on the same line as your output and it looks broken.

**`return 0;`**
returning 0 from main tells the OS "hey, everything worked fine". non-zero means something went wrong. shell scripts and CI pipelines check this. get used to returning 0 on success.

---

## Variables and Data Types

C is statically typed. every variable has a type set at compile time and it doesn't change. you can't do the python thing where something starts as an integer and becomes a string. the compiler will just refuse.

### The Basic Types

```c
int age = 17;                    // whole number, usually 4 bytes
float price = 9.99f;             // decimal, 4 bytes, ~7 digits precision
double pi = 3.14159265358979;    // better decimal, 8 bytes, ~15 digits precision
char letter = 'A';               // single character, 1 byte
```

stuff they don't tell you upfront:

**`int` size isn't guaranteed.** it's *at least* 16 bits but on basically every modern computer it's 32 bits. if you need exact sizes (common in systems work), use `<stdint.h>`:

```c
#include <stdint.h>

int8_t   tiny   = 127;                     // exactly 8 bits, signed
uint8_t  ubyte  = 255;                     // exactly 8 bits, unsigned
int16_t  small  = 32767;                   // exactly 16 bits
int32_t  normal = 2147483647;              // exactly 32 bits
int64_t  big    = 9223372036854775807LL;   // exactly 64 bits
uint64_t ubig   = 18446744073709551615ULL; // 64 bits unsigned
```

the `LL` suffix means "treat this as long long". `ULL` is "unsigned long long". without them the compiler might read a big number as a smaller type and silently overflow it.

**`char` is secretly an integer.** it stores the ASCII code of the character. `'A'` is 65, `'a'` is 97, `'0'` is 48, `'\n'` is 10. you can do math on chars:

```c
char c = 'a';
c = c - 32;   // 97 - 32 = 65 = 'A' — that's how you uppercase in C
```

**signed vs unsigned:**
```c
int x = -5;            // signed: negative allowed (-2.1B to +2.1B)
unsigned int y = 200;  // unsigned: no negatives, 0 to 4.2B
```

if you assign -1 to an unsigned int you get 4,294,967,295. it wraps around because of how binary works. this is called unsigned overflow and it's *defined* behavior in C. signed overflow however is undefined — more on that later.

**`size_t`** — this type shows up everywhere and confuses people. it's the unsigned type used for sizes and counts. guaranteed big enough to hold any memory size. use it when indexing arrays or measuring sizes:

```c
#include <stddef.h>  // or just use stdio.h

size_t length = 100;
printf("size: %zu\n", length);  // %zu for size_t
```

### Always Initialize Your Variables

```c
int x;        // DECLARED — but has garbage inside. whatever bytes were there before
int y = 10;   // DECLARED and INITIALIZED — safe
```

unlike python or java, C does NOT zero out variables for you (except globals and statics — explained later). `int x;` followed by reading `x` is undefined behavior. it could be 0, could be 847261, could literally be anything.

**always initialize your variables.** seriously, this one habit will save you from so many hours of debugging garbage values.

### Scope

variables live within the curly braces they were born in:

```c
int x = 10;   // global — visible everywhere in this file

int main(void) {
    int y = 20;  // local to main
    
    {
        int z = 30;  // only inside these braces
        printf("%d %d %d\n", x, y, z);  // fine here
    }
    
    printf("%d\n", z);  // COMPILER ERROR — z doesn't exist here
    return 0;
}
```

### Type Casting

C does some conversions automatically, sometimes silently dropping data:

```c
int x = 5;
double y = x;   // implicit, fine — no data lost

double d = 3.99;
int i = d;      // implicit — silently truncates to 3. no warning by default.
```

explicit cast:
```c
int a = 5, b = 2;
double result = (double)a / b;  // cast a to double BEFORE dividing
```

---

## Operators

### Arithmetic

```c
int a = 10, b = 3;
a + b   // 13
a - b   // 7
a * b   // 30
a / b   // 3  ← integer division, decimal gets THROWN AWAY
a % b   // 1  ← modulo (remainder after division)
```

integer division is where people trip up constantly. `10 / 3` in C is `3`. not `3.333`. to get the real result, at least one side needs to be a float:

```c
(double)a / b    // 3.333... — cast before dividing
10.0 / 3         // 3.333... — 10.0 is a double literal
```

useful modulo tricks:
```c
n % 2 == 0      // is n even?
n % 10          // last digit of n
index % size    // wrap around an array (circular buffer pattern)
```

### Assignment Shorthand

```c
x += 5;    // x = x + 5
x -= 3;    // x = x - 3
x *= 2;    // x = x * 2
x /= 4;    // x = x / 4
x %= 7;    // x = x % 7
x <<= 1;   // x = x << 1
x >>= 1;   // x = x >> 1
x &= 0xFF; // x = x & 0xFF
x |= 0x01; // x = x | 0x01
x ^= mask; // x = x ^ mask
```

### Increment and Decrement

```c
x++;   // post-increment: returns current value, THEN adds 1
++x;   // pre-increment: adds 1, THEN returns new value
x--;
--x;
```

used alone on their own line, identical. difference only shows up inside a bigger expression:

```c
int x = 5;
int a = x++;   // a = 5, then x = 6
int b = ++x;   // x = 7, then b = 7
```

### Comparison and Logic

```c
a == b   // equal (TWO equals signs)
a != b   // not equal
a > b    a < b    a >= b    a <= b

&&  // AND
||  // OR
!   // NOT
```

**the classic bug that bites everyone at least once:**
```c
if (x = 5) { }    // assigns 5 to x, always true — NOT a comparison
if (x == 5) { }   // actual comparison
```

`-Wall` catches this. short-circuit evaluation is your friend:
```c
if (ptr != NULL && ptr->value > 5) { ... }
// if ptr is NULL, second check NEVER RUNS — no crash
```

### Bitwise Operators

operate on individual bits. super important for hardware, networking, and systems code:

```c
&    // AND — 1 only if both bits are 1
|    // OR  — 1 if either bit is 1
^    // XOR — 1 if bits are different
~    // NOT — flips every bit
<<   // left shift  (multiply by 2^n)
>>   // right shift (divide by 2^n)
```

```c
5 << 1   // 10 — shift left = multiply by 2
5 << 3   // 40 — shift left 3 = multiply by 8
40 >> 2  // 10 — shift right 2 = divide by 4
```

the flag pattern — storing multiple booleans in one integer (you'll see this everywhere in real C):
```c
#define PERM_READ    (1 << 0)   // 001
#define PERM_WRITE   (1 << 1)   // 010
#define PERM_EXEC    (1 << 2)   // 100

int perms = 0;

perms |= PERM_READ;                  // set read
perms |= PERM_WRITE;                 // set write
perms &= ~PERM_EXEC;                 // clear exec
int can_read = perms & PERM_READ;    // check read
perms ^= PERM_WRITE;                 // toggle write
```

### sizeof

technically an operator, not a function. gives you the size in bytes of a type or variable at compile time:

```c
sizeof(int)        // 4 usually
sizeof(double)     // 8
sizeof(char)       // always 1
sizeof(void*)      // 4 on 32-bit, 8 on 64-bit

int arr[10];
sizeof(arr)                    // 40
sizeof(arr) / sizeof(arr[0])   // 10 — number of elements
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

curly braces are technically optional for single statements. use them anyway. apple had a major SSL security vulnerability called "goto fail" that happened partly because of a missing curly brace. just use them bro.

### Ternary

```c
int max = (a > b) ? a : b;
// same as if/else but in one line
```

fine for simple stuff. nesting ternaries inside each other is just making your future self suffer.

### switch

```c
switch (day) {
    case 1: printf("Monday\n");    break;
    case 2: printf("Tuesday\n");   break;
    case 3: printf("Wednesday\n"); break;
    default: printf("Weekend\n");  break;
}
```

**don't forget `break`**. without it, code falls through into the next case — sometimes useful, usually a bug. only works with integers/chars, not strings or floats.

### while / do-while / for

```c
// while — check first, might never run
while (i < 10) { i++; }

// do-while — run first, check after. always runs at least once
do {
    printf("Enter a number 1-10: ");
    scanf("%d", &input);
} while (input < 1 || input > 10);

// for — classic
for (int i = 0; i < 10; i++) {
    printf("%d\n", i);
}

// infinite loops
for (;;) { }    // most common style in C
while (1) { }   // also works
```

multiple variables in a for loop:
```c
for (int i = 0, j = 9; i < j; i++, j--) {
    printf("i=%d j=%d\n", i, j);
}
```

### break and continue

```c
for (int i = 0; i < 10; i++) {
    if (i == 5) break;         // exit loop entirely
    if (i % 2 == 0) continue;  // skip to next iteration
    printf("%d ", i);           // prints: 1 3
}
```

these only affect the innermost loop. for nested loops use a flag variable, restructure, or...

### goto

yeah it exists. yeah people say avoid it. the one time it's genuinely clean is escaping multiple nested loops:

```c
for (int i = 0; i < 100; i++) {
    for (int j = 0; j < 100; j++) {
        for (int k = 0; k < 100; k++) {
            if (found_it) goto done;
        }
    }
}
done:
    cleanup();
```

the linux kernel uses goto for cleanup all the time. use it when it's the clearest option, not as a default.

---

## Functions

named, reusable chunks of code. you've already seen `main` — let's write your own.

### Basic Function

```c
#include <stdio.h>

int add(int a, int b);   // prototype — tells compiler this function exists

int main(void) {
    printf("%d\n", add(3, 4));   // 7
    return 0;
}

int add(int a, int b) {   // definition — the actual code
    return a + b;
}
```

the prototype lets you call functions before defining them. without it every function would need to be above the one that calls it. fine for 20 lines, nightmare for anything real.

### void Functions

```c
void print_separator(void) {
    printf("-----------------------------\n");
}

void greet(const char *name) {
    printf("yo %s what's good\n", name);
}
```

### Parameters Are Copies — this matters a lot

when you call a function and pass a variable, C copies it. the function works on the copy:

```c
void try_to_change(int x) {
    x = 999;  // changes the local copy only
}

int n = 5;
try_to_change(n);
printf("%d\n", n);  // still 5. nothing changed.
```

to actually modify a variable from inside a function, pass a pointer to it (pointers section covers this).

### Multiple Return Values — kinda

C functions return one thing. to "return" multiple values, pass pointers as output parameters:

```c
void min_max(int arr[], int len, int *out_min, int *out_max) {
    *out_min = *out_max = arr[0];
    for (int i = 1; i < len; i++) {
        if (arr[i] < *out_min) *out_min = arr[i];
        if (arr[i] > *out_max) *out_max = arr[i];
    }
}

int mn, mx;
int data[] = {3, 1, 9, 2, 7};
min_max(data, 5, &mn, &mx);
printf("min=%d max=%d\n", mn, mx);  // min=1 max=9
```

### Recursion

functions can call themselves. needs a base case or it recurses forever and crashes (stack overflow):

```c
int factorial(int n) {
    if (n <= 1) return 1;           // base case — stop
    return n * factorial(n - 1);   // recursive step
}

// factorial(5) = 5 * 4 * 3 * 2 * 1 = 120
```

elegant but stack space is limited. deep recursion = crash. for huge inputs, use a loop instead.

### Static Local Variables

keeps its value between function calls, initialized only once:

```c
int make_id(void) {
    static int counter = 0;
    return ++counter;
}

make_id()  // → 1
make_id()  // → 2
make_id()  // → 3
```

### Variadic Functions — variable number of arguments

this is how `printf` takes different numbers of arguments each call:

```c
#include <stdarg.h>

int sum(int count, ...) {
    va_list args;
    va_start(args, count);
    int total = 0;
    for (int i = 0; i < count; i++) {
        total += va_arg(args, int);
    }
    va_end(args);
    return total;
}

sum(3, 10, 20, 30)   // 60
sum(5, 1, 2, 3, 4, 5) // 15
```

C can't figure out the types or count on its own — you have to pass that info (like how printf uses the format string to know what to expect).

### Function Pointers

functions have addresses. you can store them in pointers and call them dynamically:

```c
int add(int a, int b) { return a + b; }
int mul(int a, int b) { return a * b; }

int (*op)(int, int);  // pointer to a function taking 2 ints, returning int
op = add;
printf("%d\n", op(3, 4));  // 7
op = mul;
printf("%d\n", op(3, 4));  // 12
```

used as callbacks — passing a function as an argument to another function:

```c
void transform(int *arr, int len, int (*func)(int)) {
    for (int i = 0; i < len; i++) {
        arr[i] = func(arr[i]);
    }
}

int square(int x) { return x * x; }

int arr[] = {1, 2, 3, 4, 5};
transform(arr, 5, square);  // {1, 4, 9, 16, 25}
```

typedef to make the syntax less ugly:
```c
typedef int (*MathOp)(int, int);

MathOp op = add;
printf("%d\n", op(5, 3));
```

---

## Arrays

a fixed-size block of same-type elements stored one after another in memory.

### Declaring and Initializing

```c
int numbers[5];                          // 5 ints, uninitialized (garbage)
int scores[5] = {95, 87, 72, 91, 68};   // initialized
int zeroes[100] = {0};                   // all zeros — first is 0, rest default to 0
int auto_size[] = {1, 2, 3, 4, 5};      // compiler counts the elements (5)
```

### Accessing Elements

zero-indexed. first is [0], last is [size - 1]:

```c
int arr[5] = {10, 20, 30, 40, 50};
arr[0]        // 10
arr[4]        // 50
arr[2] = 99;  // modify
```

**C does NOT check bounds.** `arr[10]` on a 5-element array? compiler won't stop you. you'll be reading or writing memory that doesn't belong to that array. this is how you get mysterious crashes and security vulnerabilities. C trusts you to stay in bounds. stay in bounds.

### Getting Array Length

```c
int arr[] = {1, 2, 3, 4, 5};
int len = sizeof(arr) / sizeof(arr[0]);  // 20 / 4 = 5
```

**this ONLY works in the same scope where the array was declared.** pass the array to a function and it becomes a pointer — this trick breaks. always pass length separately:

```c
void print_arr(int arr[], int len) {
    for (int i = 0; i < len; i++) printf("%d ", arr[i]);
}
```

### Multidimensional Arrays

```c
int grid[2][3] = {
    {1, 2, 3},
    {4, 5, 6}
};

grid[1][2]  // 6 — row 1, column 2
```

stored in memory row by row (row-major). looping row by row is cache-friendly and fast. jumping by column kills cache performance.

### Variable-Length Arrays (C99)

C99 lets you use a runtime value as array size:

```c
int n;
scanf("%d", &n);
int arr[n];   // VLA — size set at runtime
arr[0] = 42;
```

lives on the stack, automatically freed when the function returns. don't make huge VLAs — the stack is limited. for large dynamic sizes, use `malloc` instead.

---

## Strings

strings in C are arrays of `char` ending with a null byte `\0` (ASCII 0). there is no string type. no automatic length tracking. no bounds checking. just bytes in memory with a zero at the end.

```c
char hello[] = "Hello";
// memory: H  e  l  l  o  \0
// index:  0  1  2  3  4   5
```

### Initialization

```c
char s1[] = "Hello";          // array, writable, \0 auto-added, size 6
char s2[20] = "Hello";        // array size 20, rest zero-filled
char s3[] = {'H','i','\0'};   // same as "Hi", manual
const char *s4 = "Hello";     // pointer to READ-ONLY string literal
```

that last one — `const char *s4` points to memory you can't modify. `s4[0] = 'h'` is undefined behavior, usually crashes. use an array if you need to modify the string.

### Reading Strings

```c
char name[50];
scanf("%s", name);       // reads until whitespace, no & needed, no bounds check — risky
scanf("%49s", name);     // reads at most 49 chars — safer

// reading whole lines (better):
fgets(name, sizeof(name), stdin);              // reads the whole line including spaces
name[strcspn(name, "\n")] = '\0';              // strip the newline fgets keeps
```

### String Functions

```c
#include <string.h>

strlen(s)              // length, NOT including \0
strcpy(dest, src)      // copy — dangerous if dest is too small
strncpy(dest, src, n)  // copy at most n chars — safer
strcat(dest, src)      // append — dangerous
strncat(dest, src, n)  // append at most n chars — safer
strcmp(s1, s2)         // 0 if equal, negative if s1<s2, positive if s1>s2
strncmp(s1, s2, n)     // compare first n chars
strchr(s, c)           // first occurrence of char c, returns pointer or NULL
strstr(s, sub)         // find substring, returns pointer or NULL
memset(s, c, n)        // set n bytes to value c
memcpy(dest, src, n)   // copy n bytes
memmove(dest, src, n)  // copy n bytes, safe when src and dest overlap
```

**never use `strcpy` on user input without knowing both sizes.** use `snprintf` for safe string building — it literally cannot overflow:

```c
char buf[64];
snprintf(buf, sizeof(buf), "Hello %s, you are %d years old", name, age);
// snprintf NEVER overflows. it stops at sizeof(buf).
```

### String Conversions

```c
#include <stdlib.h>

int x = atoi("42");          // string to int — no error checking
double d = atof("3.14");     // string to double — no error checking

// better: strtol has error checking
char *end;
long n = strtol("123abc", &end, 10);  // base 10
if (*end != '\0') printf("non-numeric stuff found: %s\n", end);
```

---

## Pointers

okay, this is the section. take your time. if you don't get it first read, read it again. no shame in that, pointers trip everyone up.

### What's Actually Happening

every variable lives at some memory address — just a number like `0x7fff5a8b2c10`. a pointer is a variable that stores one of those addresses.

```c
int x = 42;
int *p = &x;    // p stores the address of x
```

three operators to know:
- `&x` — **address-of**: gives you x's memory address
- `int *p` — declares p as "pointer to int" (holds an int's address)
- `*p` — **dereference**: "go to the address in p and look at what's there"

```c
printf("%d\n", x);          // 42 — value of x
printf("%p\n", (void*)p);   // 0x7fff... — the address
printf("%d\n", *p);         // 42 — value AT that address

*p = 100;
printf("%d\n", x);          // 100 — x changed because p points to x
```

### Why Pointers Exist

remember how function arguments are copies? pointers fix that:

```c
void actually_double(int *p) {
    *p = *p * 2;   // go to the address, modify what's there
}

int n = 5;
actually_double(&n);   // pass the ADDRESS
printf("%d\n", n);     // 10 — actually changed
```

this is exactly how `scanf` works — it needs to write INTO your variable:
```c
int x;
scanf("%d", &x);   // & gives scanf x's address so it can write there
```

### Null Pointers

pointer that points to nothing should be NULL:

```c
int *p = NULL;

if (p == NULL) {
    printf("not pointing to anything\n");
}

*p = 5;   // SEGFAULT — never dereference NULL
```

always check for NULL before dereferencing if there's any chance it could be NULL. tons of standard functions return NULL on failure.

### Pointer Arithmetic

adding to a pointer moves it by that many *elements*, not bytes:

```c
int arr[] = {10, 20, 30, 40, 50};
int *p = arr;     // points to arr[0]

*(p + 0)   // 10 — arr[0]
*(p + 1)   // 20 — arr[1]
*(p + 2)   // 30 — arr[2]

// these are identical:
arr[3]      // 40
*(arr + 3)  // 40
p[3]        // 40 — [] is literally just *(p + n) under the hood
```

`p + 1` adds `sizeof(int)` bytes (usually 4). the compiler scales automatically based on the pointer type.

### Pointer to Pointer

pointers can point to other pointers:

```c
int x = 5;
int *p = &x;
int **pp = &p;

printf("%d\n", **pp);  // 5 — dereference twice

**pp = 99;
printf("%d\n", x);     // 99
```

this comes up in: dynamic 2D arrays, linked lists, functions that need to modify a pointer itself, `char **argv`.

### const + Pointers — four combos

```c
int x = 5;

int *p = &x;               // can change p, can change *p
const int *p = &x;         // can change p, CANNOT change *p
int * const p = &x;        // CANNOT change p, can change *p
const int * const p = &x;  // can change neither

// read it right-to-left:
// "const int *p" → p is a [pointer to] [const int]
// "int * const p" → p is a [const pointer to] [int]
```

use `const int *` in function parameters to say "i promise i won't modify what you pass me":
```c
void print_string(const char *s) {
    printf("%s\n", s);  // read-only, won't modify s
}
```

---

## Memory Management

this is where C separates from every managed language. there is no garbage collector. you request memory, you free memory. that's on you.

### The Stack

local variables live here. fast, automatically managed. when a function returns, all its local variables are gone.

```c
void foo(void) {
    int x = 10;       // stack
    int arr[1000];    // stack — 4000 bytes
}   // both gone instantly when foo returns
```

stack is limited — usually 1-8MB. putting a huge array on the stack:
```c
void bad(void) {
    int massive[1000000];   // 4MB on stack — will crash (stack overflow)
}
```

for large data, use the heap.

### The Heap

large memory pool you manage yourself. request with `malloc`, release with `free`:

```c
#include <stdlib.h>

int *arr = malloc(100 * sizeof(int));   // allocate space for 100 ints

if (arr == NULL) {
    fprintf(stderr, "malloc failed\n");
    exit(1);
}

for (int i = 0; i < 100; i++) {
    arr[i] = i * i;
}

free(arr);     // release it when done
arr = NULL;    // prevent accidental use-after-free
```

### malloc / calloc / realloc / free

```c
// malloc — raw bytes, uninitialized (contains garbage)
int *p = malloc(10 * sizeof(int));

// calloc — allocates AND zero-initializes
int *p = calloc(10, sizeof(int));   // 10 ints, all 0

// realloc — resize an existing allocation
int *tmp = realloc(p, 20 * sizeof(int));
if (tmp == NULL) {
    free(p);    // realloc failed but p is still valid — free it
    return;
}
p = tmp;        // only update p if it succeeded

// free — give it back
free(p);
p = NULL;
```

**realloc gotcha:** if you do `p = realloc(p, size)` and it fails, you've lost your only reference to the original memory. always use a temp pointer.

### The Four Memory Crimes

these will genuinely ruin your program. and the worst part is the compiler won't even warn you about most of them.

**1. Memory Leak — forgetting to free:**
```c
void leaky(int n) {
    int *arr = malloc(n * sizeof(int));
    // ... use arr ...
    return;   // forgot free(arr) — that memory is gone until the process ends
}
// call this in a loop and your program eats RAM until it crashes
```

**2. Use-After-Free:**
```c
int *p = malloc(sizeof(int));
*p = 42;
free(p);
printf("%d\n", *p);   // undefined behavior — could print anything, could crash
```

**3. Double Free:**
```c
free(p);
free(p);   // undefined behavior — usually corrupts heap and crashes
```

**4. Buffer Overflow:**
```c
int *arr = malloc(5 * sizeof(int));
arr[5] = 99;    // one past the end — writing memory you don't own
arr[-1] = 0;    // before the start — also bad
```

### Catching Memory Bugs

compile with AddressSanitizer — it catches all four of the above:

```bash
gcc -fsanitize=address,undefined -g -o program program.c
./program
```

it'll tell you exactly what went wrong, on which line, with a full stack trace. run this on everything.

Valgrind for even deeper leak analysis:
```bash
valgrind --leak-check=full ./program
```

### Building a Dynamic Array

C has no built-in growable array (like python's list). here's the standard pattern:

```c
typedef struct {
    int    *data;
    size_t  size;      // current element count
    size_t  capacity;  // allocated slots
} IntArray;

IntArray array_new(void) {
    return (IntArray){
        .data = malloc(4 * sizeof(int)),
        .size = 0,
        .capacity = 4
    };
}

void array_push(IntArray *a, int val) {
    if (a->size == a->capacity) {
        a->capacity *= 2;
        int *tmp = realloc(a->data, a->capacity * sizeof(int));
        if (!tmp) { fprintf(stderr, "OOM\n"); exit(1); }
        a->data = tmp;
    }
    a->data[a->size++] = val;
}

void array_free(IntArray *a) {
    free(a->data);
    a->data = NULL;
    a->size = a->capacity = 0;
}
```

"double when full" is the standard strategy. python lists, C++ vectors, Java ArrayLists — all use this. gives O(1) amortized push time.

---

## Structs, Unions, and Enums

### Structs

group related variables together under one name:

```c
typedef struct {
    char  name[50];
    int   age;
    float gpa;
} Student;

Student s1 = {"Alice", 18, 3.9};
Student s2 = {.name = "Bob", .age = 19, .gpa = 3.5};  // designated initializers (C99)

printf("%s is %d\n", s1.name, s1.age);
s1.age++;   // just use dot notation
```

structs copy by value when assigned or passed to functions:
```c
Student copy = s1;    // completely separate copy
copy.age = 99;        // doesn't affect s1
```

to modify a struct inside a function, pass a pointer and use `->`:
```c
void birthday(Student *s) {
    s->age++;   // arrow -> is shorthand for (*s).age
}

birthday(&s1);
```

### Struct Padding — the thing nobody warns you about

```c
typedef struct {
    char a;   // 1 byte
    int  b;   // 4 bytes
    char c;   // 1 byte
} S;

printf("%zu\n", sizeof(S));   // probably 12, not 6
```

the compiler inserts padding to align fields on their natural boundaries (ints want addresses divisible by 4). reordering saves space:

```c
typedef struct {
    int  b;   // 4 bytes — put bigger types first
    char a;   // 1 byte
    char c;   // 1 byte
              // 2 bytes padding here
} S;          // 8 bytes — better
```

matters a lot when working with binary file formats or sending structs over a network.

### Linked Lists — the classic self-referencing struct

```c
typedef struct Node {
    int          data;
    struct Node *next;   // must say "struct Node" here, typedef isn't done yet
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
for (Node *cur = head; cur != NULL; cur = cur->next) {
    printf("%d ", cur->data);
}

// free every node
Node *cur = head;
while (cur != NULL) {
    Node *tmp = cur->next;   // save next BEFORE freeing current
    free(cur);
    cur = tmp;
}
```

### Unions

all members share the same memory. size = size of largest member. only one member valid at a time:

```c
union Data {
    int    i;
    float  f;
    double d;
};

union Data v;
v.i = 42;
printf("%d\n", v.i);   // 42

v.f = 3.14f;
printf("%f\n", v.f);   // 3.14
printf("%d\n", v.i);   // garbage — f and i share memory
```

tagged union pattern — a variable that can be multiple types:
```c
typedef enum { VAL_INT, VAL_FLOAT, VAL_STRING } ValType;

typedef struct {
    ValType type;
    union {
        int   i;
        float f;
        char  s[64];
    };
} Value;

Value v = {.type = VAL_INT, .i = 100};

if (v.type == VAL_INT) printf("int: %d\n", v.i);
```

this pattern is in JSON parsers, scripting languages, interpreters — anywhere one variable needs multiple possible types.

### Enums

named integer constants:

```c
typedef enum {
    MON = 0, TUE, WED, THU, FRI, SAT, SUN
} Weekday;

typedef enum {
    HTTP_OK           = 200,
    HTTP_NOT_FOUND    = 404,
    HTTP_SERVER_ERROR = 500
} HttpCode;

Weekday today = WED;
if (today >= MON && today <= FRI) printf("its a weekday grind\n");
```

under the hood enums are just ints. no type enforcement in C (unlike C++).

---

## File I/O

C handles files through the `FILE` type in `<stdio.h>`.

### Open, Use, Close

```c
FILE *fp = fopen("data.txt", "r");
if (fp == NULL) {
    perror("fopen");   // "fopen: No such file or directory"
    return 1;
}

// ... do stuff ...

fclose(fp);   // always close it. always.
```

| Mode | Meaning |
|------|---------|
| `"r"` | read (file must exist) |
| `"w"` | write (creates or truncates) |
| `"a"` | append (creates if needed) |
| `"r+"` | read + write |
| `"rb"` / `"wb"` | binary mode |

### Reading

```c
// character by character
int c;
while ((c = fgetc(fp)) != EOF) putchar(c);

// line by line
char line[256];
while (fgets(line, sizeof(line), fp) != NULL) {
    printf("%s", line);   // fgets keeps the \n
}

// formatted
int id; char name[50]; float score;
while (fscanf(fp, "%d %49s %f", &id, name, &score) == 3) {
    printf("%d %s %.1f\n", id, name, score);
}
```

### Writing

```c
FILE *fp = fopen("out.txt", "w");
fprintf(fp, "score: %d\n", 95);
fputs("another line\n", fp);
fputc('X', fp);
fclose(fp);
```

### Binary Files

```c
int data[] = {1, 2, 3, 4, 5};

// write
FILE *fp = fopen("data.bin", "wb");
fwrite(data, sizeof(int), 5, fp);
fclose(fp);

// read back
int back[5];
fp = fopen("data.bin", "rb");
fread(back, sizeof(int), 5, fp);
fclose(fp);
```

### Navigation

```c
fseek(fp, 0, SEEK_SET);    // go to start
fseek(fp, 0, SEEK_END);    // go to end
fseek(fp, -10, SEEK_CUR);  // go 10 bytes back from current position
long pos = ftell(fp);       // get current position
rewind(fp);                 // back to start (same as fseek SEEK_SET 0)

// get file size:
fseek(fp, 0, SEEK_END);
long size = ftell(fp);
rewind(fp);
```

---

## The Preprocessor

runs before the compiler. handles every line starting with `#`. basically doing smart find-and-replace before the compiler sees your code.

### #include

```c
#include <stdio.h>     // look in system headers
#include "myfile.h"    // look in current directory first
```

literally pastes the file contents at that line.

### #define

```c
#define PI        3.14159265358979
#define MAX_SIZE  1024
#define VERSION   "2.0.1"

// function-like macros
#define SQUARE(x)     ((x) * (x))
#define MAX(a, b)     ((a) > (b) ? (a) : (b))
#define ARRAY_LEN(a)  (sizeof(a) / sizeof((a)[0]))
```

**always parenthesize macro arguments and the full expression.** no parens:
```
SQUARE(1+2) → 1+2 * 1+2 = 5  ← WRONG
SQUARE(1+2) → (1+2) * (1+2) = 9  ← correct with parens
```

macros have no type checking, no scoping. they're text substitution. prefer `const` for constants and `inline` functions for simple calcs. macros shine when you need things that genuinely can't be functions (like `ARRAY_LEN` above — you can't pass a type to a function).

### Conditional Compilation

```c
#define DEBUG 1

#ifdef DEBUG
    printf("debug: x = %d\n", x);
#endif

#ifndef BUFFER_SIZE
    #define BUFFER_SIZE 512
#endif

// platform stuff
#if defined(_WIN32)
    #include <windows.h>
#elif defined(__linux__) || defined(__APPLE__)
    #include <unistd.h>
#endif
```

### Predefined Macros

```c
__FILE__      // current filename as a string
__LINE__      // current line number
__DATE__      // "Mar 14 2026"
__TIME__      // "12:34:56"
__func__      // current function name (C99)
```

debug logging macro:
```c
#define LOG(fmt, ...) \
    fprintf(stderr, "[%s:%d] " fmt "\n", __FILE__, __LINE__, ##__VA_ARGS__)

LOG("starting up");           // [main.c:15] starting up
LOG("x=%d y=%d", x, y);      // [main.c:20] x=5 y=10
```

---

## Header Files and Multi-File Projects

### Why Split Files

once you're past ~200 lines, one file gets impossible to navigate. splitting means:
- easier to find stuff
- only recompile what changed
- multiple people can work on different files
- reuse code across projects

### The Pattern

**vec2.h** — the public interface, declarations only:
```c
#ifndef VEC2_H
#define VEC2_H

typedef struct {
    double x, y;
} Vec2;

Vec2   vec2_add(Vec2 a, Vec2 b);
Vec2   vec2_scale(Vec2 v, double s);
double vec2_length(Vec2 v);
void   vec2_print(Vec2 v);

#endif
```

**vec2.c** — the actual implementation:
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

**main.c:**
```c
#include <stdio.h>
#include "vec2.h"

int main(void) {
    Vec2 a = {3.0, 4.0};
    printf("length: %.2f\n", vec2_length(a));  // 5.00
    return 0;
}
```

```bash
gcc -Wall -Wextra -std=c17 main.c vec2.c -o program -lm
```

### Include Guards

without these, double-including a header gives you "duplicate declaration" errors:

```c
// in every .h file:
#ifndef VEC2_H
#define VEC2_H

// ... all your declarations ...

#endif
```

or the simpler `#pragma once` — not technically in the standard but every compiler supports it and it's cleaner:
```c
#pragma once
// ... declarations ...
```

### Makefiles

stop retyping compile commands every time bro. use a Makefile:

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

`make` builds it, only recompiling changed files. `make clean` wipes compiled outputs.

---

## Modern C

C is not frozen in 1989. it gets updated: C99 (1999), C11 (2011), C17 (2017), C23 (2023). here's what was added that you actually need to know.

### C99 — the big one

**`//` comments** — yeah, single-line comments didn't exist in original C. only `/* */` existed.

**declare variables anywhere** — C89 required all declarations at the top of a block. C99 says declare wherever:
```c
// old C89 style
void foo(void) {
    int i;
    int result;
    i = 5;
    result = i * 2;
}

// C99 — declare when you need it
void foo(void) {
    for (int i = 0; i < 10; i++) {   // i declared right here
        int square = i * i;           // totally fine
    }
}
```

**`<stdbool.h>` — boolean type:**
```c
#include <stdbool.h>

bool found = false;
bool valid = true;

if (found) printf("found it\n");
// true = 1, false = 0 under the hood
```

**compound literals** — create anonymous struct/array values inline:
```c
typedef struct { int x, y; } Point;

void draw(Point p) { printf("(%d,%d)\n", p.x, p.y); }

draw((Point){3, 4});             // pass struct literal directly
draw((Point){.x=5, .y=7});      // with designated initializers

int *arr = (int[]){1, 2, 3, 4}; // anonymous array literal
```

**designated initializers** for arrays:
```c
int arr[10] = {[0]=1, [5]=10, [9]=99};  // other elements are 0
```

**flexible array members:**
```c
typedef struct {
    int length;
    int data[];   // must be last — size determined at alloc time
} IntBuf;

IntBuf *buf = malloc(sizeof(IntBuf) + 10 * sizeof(int));
buf->length = 10;
buf->data[0] = 42;
free(buf);
```

**`restrict` keyword** — tells the compiler two pointers don't overlap, enables better optimization:
```c
void copy(int * restrict dest, const int * restrict src, size_t n) {
    for (size_t i = 0; i < n; i++) dest[i] = src[i];
}
```

### C11 — worth knowing

**`static_assert`** — compile-time assertions:
```c
#include <assert.h>

static_assert(sizeof(int) == 4, "need 32-bit ints");
static_assert(sizeof(void*) == 8, "need 64-bit pointers");
// if condition is false: COMPILE ERROR with your message
// not a runtime crash. a compile error.
```

**`_Noreturn` / `noreturn`** — for functions that never return:
```c
#include <stdnoreturn.h>

noreturn void die(const char *msg) {
    fprintf(stderr, "fatal: %s\n", msg);
    exit(1);
}
```

**`_Generic`** — compile-time type dispatch:
```c
#define type_name(x) _Generic((x),  \
    int:    "int",                   \
    float:  "float",                 \
    double: "double",                \
    char*:  "string",                \
    default: "unknown"               \
)

printf("%s\n", type_name(42));      // int
printf("%s\n", type_name(3.14f));   // float
printf("%s\n", type_name("yo"));    // string
```

compiler picks the branch at compile time based on the type. the other branches don't exist in the binary.

**anonymous struct/union members (C11):**
```c
typedef struct {
    int type;
    union {           // anonymous — members accessed directly on parent struct
        int   i;
        float f;
        char  s[32];
    };
} Variant;

Variant v;
v.type = 0;
v.i = 42;   // direct access, no v.anon.i
```

**atomics (`<stdatomic.h>`)** — thread-safe operations:
```c
#include <stdatomic.h>

atomic_int counter = 0;

atomic_fetch_add(&counter, 1);    // counter++ thread-safely
int val = atomic_load(&counter);  // read thread-safely
atomic_store(&counter, 0);        // write thread-safely
```

**threads (`<threads.h>`):**
```c
#include <threads.h>

int worker(void *arg) {
    printf("thread running\n");
    return 0;
}

thrd_t t;
thrd_create(&t, worker, NULL);
thrd_join(t, NULL);
```

### C17 — mostly bugfixes

C17 was a cleanup/bugfix release for C11. main takeaway: `-std=c17` is the current stable standard to target. use it.

### C23 — the new stuff

**`nullptr`** — null pointer constant with its own type:
```c
int *p = nullptr;   // cleaner than NULL
```

**`#embed`** — embed file bytes at compile time:
```c
const unsigned char image[] = {
#embed "logo.png"   // raw bytes of the file baked into your binary
};
```

**`typeof`** — get the type of an expression:
```c
int x = 5;
typeof(x) y = 10;   // y is int

// clean swap macro using typeof:
#define SWAP(a, b) do {       \
    typeof(a) _tmp = (a);     \
    (a) = (b);                \
    (b) = _tmp;               \
} while(0)
```

**`[[nodiscard]]`, `[[deprecated]]`, `[[maybe_unused]]`:**
```c
[[nodiscard]] int important_result(void) { return 42; }
// compiler warns if you call this and throw away the return value

[[deprecated("use new_api() instead")]]
void old_api(void) { }
// compiler warns every time someone calls this

void func([[maybe_unused]] int debug_param) {
    // no unused parameter warning
}
```

**binary literals (now official in C23):**
```c
int flags = 0b10110100;   // was a GCC extension before, now standard
```

---

## Error Handling

C has no exceptions. you handle errors manually. few patterns for doing it.

### errno

many standard functions set `errno` from `<errno.h>` when they fail:

```c
#include <errno.h>
#include <string.h>

FILE *fp = fopen("missing.txt", "r");
if (fp == NULL) {
    printf("error %d: %s\n", errno, strerror(errno));
    perror("fopen");   // prints "fopen: No such file or directory"
}
```

common values: `ENOENT` (no such file), `EACCES` (permission denied), `ENOMEM` (out of memory), `EINVAL` (invalid argument).

**errno only means something immediately after a failure.** any subsequent function call might change it.

### Return Value Patterns

most C functions signal errors through their return value:

```c
// return NULL on failure
FILE *fp = fopen(...);
if (!fp) { handle_error(); }

// return -1 on failure (POSIX style)
int fd = open("file", O_RDONLY);
if (fd == -1) { perror("open"); }

// return 0 on success, positive on error (pthread style)
int err = pthread_create(&t, NULL, func, arg);
if (err) { fprintf(stderr, "%s\n", strerror(err)); }
```

### Custom Error Codes

for your own libraries:

```c
typedef enum {
    OK = 0,
    ERR_INVALID_ARG  = -1,
    ERR_OUT_OF_MEMORY = -2,
    ERR_NOT_FOUND    = -3,
} Error;

const char *err_str(Error e) {
    switch (e) {
        case OK:               return "ok";
        case ERR_INVALID_ARG:  return "invalid argument";
        case ERR_OUT_OF_MEMORY: return "out of memory";
        case ERR_NOT_FOUND:    return "not found";
        default:               return "unknown error";
    }
}
```

### assert

for programmer mistakes — things that should never happen. not for handling user input:

```c
#include <assert.h>

void process(int *data, int len) {
    assert(data != NULL);   // if this fires, it's a bug in the caller
    assert(len > 0);
}
```

false condition = program prints location and crashes. define `NDEBUG` to disable in release builds: `gcc -DNDEBUG ...`.

---

## Bit Manipulation Deep Dive

worth knowing if you'll touch hardware, networking, compression, or binary file formats.

### Number Representations

```c
int a = 255;        // decimal
int b = 0xFF;       // hex — same value
int c = 0377;       // octal — same value (leading 0 = octal, don't accidentally do this)
int d = 0b11111111; // binary — same value (C23 / GCC extension)
```

### Two's Complement

how negative numbers work in binary. to negate: flip all bits, add 1:
```
 5 in 8-bit: 00000101
flip bits:   11111010
add 1:       11111011  ← that's -5
```

this is why:
- signed overflow is undefined behavior — wrapping is NOT guaranteed
- `-1` as unsigned is all ones / max value
- right-shifting a negative number fills with 1s (sign extension)

### Bit Tricks

```c
// is n a power of 2?
bool is_pow2(unsigned n) {
    return n > 0 && (n & (n - 1)) == 0;
}
// power of 2 has exactly one bit: 0b1000
// n-1 flips the lower bits:       0b0111
// AND = 0 for any power of 2

// round up to next power of 2
unsigned next_pow2(unsigned n) {
    n--;
    n |= n >> 1;
    n |= n >> 2;
    n |= n >> 4;
    n |= n >> 8;
    n |= n >> 16;
    return n + 1;
}

// count set bits
int popcount(unsigned n) {
    int c = 0;
    while (n) { c += n & 1; n >>= 1; }
    return c;
}
// GCC built-in: __builtin_popcount(n)

// get/set/clear/toggle bit n
int  get_bit(int v, int n)    { return (v >> n) & 1; }
int  set_bit(int v, int n)    { return v |  (1 << n); }
int  clr_bit(int v, int n)    { return v & ~(1 << n); }
int  tog_bit(int v, int n)    { return v ^  (1 << n); }

// extract bits at position start, length count
int  extract(int v, int start, int count) {
    return (v >> start) & ((1 << count) - 1);
}

// XOR swap (no temp variable)
a ^= b;
b ^= a;
a ^= b;
```

---

## Common Pitfalls

these are the things that will catch you off guard. not because you're bad at this — because C is genuinely designed to let you shoot yourself in the foot without flinching.

**uninitialized variables:**
```c
int sum;
for (int i = 0; i < 5; i++) sum += i;  // sum starts as garbage
int sum = 0;  // fix — always initialize
```

**integer division when you want decimals:**
```c
double avg = total / count;          // both ints → truncated before converting
double avg = (double)total / count;  // cast BEFORE dividing
```

**signed/unsigned comparison:**
```c
int i = 0;
if (i < sizeof(arr)) { }           // warning — signed vs unsigned
if ((size_t)i < sizeof(arr)) { }   // correct
```

**modifying string literals:**
```c
char *s = "hello";
s[0] = 'H';     // segfault

char s[] = "hello";
s[0] = 'H';     // fine, it's a writable array
```

**scanf leaving newlines in the buffer:**
```c
int n;
char str[50];
scanf("%d", &n);
fgets(str, sizeof(str), stdin);   // reads the leftover '\n', not what you want
// fix:
scanf("%d", &n);
getchar();                         // consume the leftover newline
fgets(str, sizeof(str), stdin);   // NOW reads actual input
```

**off-by-one in loops:**
```c
char str[] = "hello";
for (int i = 0; i <= strlen(str); i++) { }  // i=5 accesses '\0', i=6 is out of bounds
for (int i = 0; i < strlen(str); i++) { }   // correct
// also: cache strlen, don't call it every iteration
size_t len = strlen(str);
for (size_t i = 0; i < len; i++) { }
```

**returning a pointer to a local variable:**
```c
int *bad(void) {
    int x = 42;
    return &x;     // x dies when function returns — dangling pointer
}

int *good(void) {
    int *p = malloc(sizeof(int));
    *p = 42;
    return p;      // heap memory persists — caller frees it
}
```

**signed integer overflow:**
```c
int x = INT_MAX;
x++;   // undefined behavior. in practice wraps to INT_MIN but you can't rely on it
       // signed overflow is NOT defined. use unsigned if you need wrapping.
```

**not null-terminating strings:**
```c
char buf[5];
strncpy(buf, "hello", 5);    // copies 5 chars but NO room for \0 — not terminated
buf[4] = '\0';               // manual fix
// or just use snprintf — always null-terminates
snprintf(buf, sizeof(buf), "%s", "hello");
```

**realloc gotcha:**
```c
p = realloc(p, new_size);    // if this fails, p is NULL and original is LEAKED
// correct:
void *tmp = realloc(p, new_size);
if (!tmp) { free(p); return NULL; }
p = tmp;
```

---

## Debugging

### GDB

compile with `-g` first:
```bash
gcc -g -Wall -o program program.c
gdb ./program
```

useful commands:
```
run                — start the program
break main         — breakpoint at main
break filename.c:42 — breakpoint at line 42
next               — next line (don't step into functions)
step               — next line (step INTO functions)
print x            — print variable x
print *ptr         — print value pointer points to
print arr[0]       — print array element
info locals        — print all local variables
backtrace          — show call stack (great after a crash)
continue           — run until next breakpoint
watch x            — pause when x changes value
quit               — exit
```

### AddressSanitizer + UBSan

best combo for catching memory bugs AND undefined behavior:

```bash
gcc -fsanitize=address,undefined -g -o program program.c
./program
```

catches: heap overflow, stack overflow, use-after-free, memory leaks, null dereference, signed overflow, invalid array index. run this on every project during development.

### Valgrind

slower but more thorough leak detection:
```bash
valgrind --leak-check=full --show-leak-kinds=all ./program
```

### printf Debugging

sometimes you just need to print stuff to figure out what's going on:

```c
#define DBG(fmt, ...) \
    fprintf(stderr, "[DBG %s:%d] " fmt "\n", __FILE__, __LINE__, ##__VA_ARGS__)

DBG("x = %d", x);
DBG("entering loop, n = %zu", n);
```

`stderr` keeps debug output separate from normal output so they don't mix when you redirect.

---

## Questions

try to answer these before looking at the answers section at the bottom. actually try. the ones that trip you up are the most useful.

---

**Q1:**
```c
int a = 5, b = 2;
printf("%d\n",   a / b);
printf("%.2f\n", (float)a / b);
printf("%.2f\n", (float)(a / b));
```
what does each line print and why?

---

**Q2:**
```c
char *s = "Hello";
s[0] = 'h';
```
what happens when you run this?

---

**Q3:**
```c
int arr[] = {10, 20, 30, 40, 50};
int *p = arr + 2;
printf("%d %d %d\n", *(p-1), *p, *(p+1));
```
what's the output?

---

**Q4:** find every bug:
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

**Q5:**
```c
int i = 0;
printf("%d %d %d\n", i++, i++, i++);
```
what does this print?

---

**Q6:** why doesn't this swap work? how do you fix it?
```c
void swap(int a, int b) {
    int temp = a;
    a = b;
    b = temp;
}

int x = 5, y = 10;
swap(x, y);
// x and y are unchanged after this
```

---

**Q7:**
```c
unsigned int x = 1;
printf("%u\n", x - 2);
```
what prints and why?

---

**Q8:** will this compile? what happens at runtime?
```c
int *get_num(void) {
    int x = 42;
    return &x;
}

int main(void) {
    int *p = get_num();
    printf("%d\n", *p);
    return 0;
}
```

---

**Q9:** how many bytes does this struct take? why?
```c
typedef struct {
    char   a;
    int    b;
    char   c;
    double d;
} S;
printf("%zu\n", sizeof(S));
```

---

**Q10:** what's the difference?
```c
const int *p;
int * const p;
```

---

**Q11:**
```c
char s1[] = "hello";
char s2[] = "hello";
printf("%d\n", s1 == s2);
printf("%d\n", strcmp(s1, s2) == 0);
```
what does each line print and why?

---

**Q12:**
```c
void count(void) {
    static int n = 0;
    printf("%d\n", ++n);
}
count(); count(); count();
```
what prints?

---

**Q13:** what are the two bugs in this realloc call?
```c
int *p = malloc(10 * sizeof(int));
p = realloc(p, 20 * sizeof(int));
if (p == NULL) {
    free(p);
}
```

---

**Q14:** what happens when you run this?
```c
char buf[10];
strcpy(buf, "hello");
strcat(buf, " world");
printf("%s\n", buf);
```

---

**Q15:** what does `_Generic` actually do at compile time vs runtime?
```c
#define type_name(x) _Generic((x), \
    int: "int",                     \
    float: "float",                 \
    double: "double",               \
    default: "other"                \
)
printf("%s\n", type_name(42));
printf("%s\n", type_name(3.14));
```

---

**Q16:** what's wrong with this string code?
```c
char buf[6];
strncpy(buf, "hello!", 6);
printf("%s\n", buf);
```

---

**Q17:** what does this print?
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

**Q18:** write a function that reads an entire file into a heap-allocated buffer. the caller is responsible for freeing it. handle all error cases.

---

---

## Answers

---

**A1:**
line 1 prints `2`. integer division — `5 / 2` is `2` in C, decimal thrown away.
line 2 prints `2.50`. casting `a` to float *before* dividing forces float division: `5.0 / 2 = 2.5`.
line 3 prints `2.00` — the trap. `(a / b)` happens *first* as integer division (= 2), THEN gets cast to float (= 2.0). the truncation already happened, the cast doesn't undo it. order of operations matters.

---

**A2:**
segfault, probably. string literals are stored in read-only memory. `s` is a pointer to a literal, not a writable array. writing to `s[0]` is undefined behavior — typically a crash. fix: `char s[] = "Hello";` — this copies the literal into a writable stack array.

---

**A3:**
`20 30 40`. `arr + 2` points to index 2 (value 30). `p - 1` points to index 1 (20). `p + 1` points to index 3 (40).

---

**A4:**
three bugs: (1) off-by-one — `i <= 10` writes `arr[10]` which is out of bounds (indices 0-9). fix: `i < 10`. (2) no NULL check after malloc — if malloc fails, `arr[i]` crashes immediately. (3) no `free(arr)` — memory leak.

---

**A5:**
undefined behavior. the C standard does not specify the order function arguments are evaluated. you might get `0 1 2`, `2 1 0`, or something else entirely — depends on the compiler and platform. modifying and reading the same variable multiple times in one expression without a sequence point between is always UB. never do this.

---

**A6:**
C passes arguments by value. `swap` gets copies of x and y. modifying the copies doesn't touch the originals. fix with pointers:
```c
void swap(int *a, int *b) {
    int temp = *a;
    *a = *b;
    *b = temp;
}
swap(&x, &y);
```

---

**A7:**
prints `4294967295` (or whatever `UINT_MAX` is on your system, usually that). `x` is unsigned. `1 - 2` mathematically = -1, but unsigned integers can't be negative — they wrap around. -1 in unsigned is `UINT_MAX`. this is defined behavior for unsigned (unlike signed overflow). use `%u` to print unsigned values correctly.

---

**A8:**
compiles, usually with a warning about returning address of a local variable. at runtime: `x` is a local variable on `get_num`'s stack frame. when the function returns, that stack frame is deallocated. `p` now points to invalid memory — a dangling pointer. dereferencing it is undefined behavior. it might print 42 if the memory hasn't been overwritten yet, or garbage, or crash. fix: either allocate on the heap (`malloc`) and return that, or take a pointer parameter to write into, or make `x` static.

---

**A9:**
probably 24 bytes, not 14. the compiler adds padding so each field starts at an aligned address. `a` (char, 1 byte), then 3 bytes padding so `b` (int) starts at offset 4, `b` (int, 4 bytes) at offset 4, `c` (char, 1 byte) at offset 8, then 7 bytes padding so `d` (double) starts at offset 16 (divisible by 8), `d` (double, 8 bytes) at offset 16. total: 24. reordering to put double first reduces this to 16.

---

**A10:**
completely different things. `const int *p` — pointer to const int — you CAN reassign `p` to point somewhere else, but CANNOT modify `*p`. `int * const p` — const pointer to int — you CANNOT reassign `p`, but CAN modify `*p`. read right-to-left: "p is a [pointer to] [const int]" vs "p is a [const pointer] to [int]".

---

**A11:**
first line prints `0`. second line prints `1`. `s1 == s2` compares memory addresses — s1 and s2 are two different arrays at different locations, so their addresses are different, and the comparison is false. `strcmp(s1, s2)` compares the actual character contents and returns 0 if identical — correct. never use `==` to compare strings in C.

---

**A12:**
prints 1, then 2, then 3. `n` is a static local variable — it persists between calls and is only initialized to 0 once. each call increments it and prints the new value.

---

**A13:**
two bugs. (1) if `realloc` fails it returns NULL but does NOT free the original allocation. `p = realloc(p, ...)` — if it returns NULL, you've lost the only reference to the original block. memory leaked. (2) `free(p)` where `p` is NULL is technically safe (freeing NULL is a no-op), but you're still leaking the original. fix: `void *tmp = realloc(p, new_size); if (!tmp) { free(p); return; } p = tmp;`

---

**A14:**
buffer overflow. `buf` is 10 bytes. after `strcpy(buf, "hello")` it holds 6 bytes (5 chars + null). `strcat(buf, " world")` tries to append 7 more bytes — that's 12 bytes total, 2 past the end of the buffer. undefined behavior, likely corrupts adjacent memory or crashes. fix: make `buf` bigger, or use `strncat`/`snprintf`.

---

**A15:**
purely compile-time. `_Generic` (C11) examines the type of the expression at compile time and selects one branch. the other branches don't exist in the compiled binary — they're not even compiled. `type_name(42)` → the compiler sees `int`, selects `"int"`, done. `type_name(3.14)` → 3.14 without a suffix is `double`, so selects `"double"`. this is zero-cost type dispatch.

---

**A16:**
the string "hello!" is 6 characters plus a null terminator = 7 bytes total. `strncpy(buf, "hello!", 6)` copies exactly 6 characters and fills the buffer completely — with NO room for `\0`. `buf` is not null-terminated. `printf("%s\n", buf)` reads past the end of the buffer until it hits a zero byte somewhere in memory — undefined behavior, could print garbage, could crash. fix: `char buf[7]` or copy fewer chars and manually set `buf[5] = '\0'`, or use `snprintf(buf, sizeof(buf), "%s", "hello!")`.

---

**A17:**
prints "nonzero", then "x is now: 0". `x = 0` in the condition is an assignment, not a comparison. it assigns 0 to x, then evaluates to 0, which is falsy, so the else branch runs. x is now 0. this is almost always a bug — you meant `if (x == 0)`. `-Wall` warns about this.

---

**A18:**
```c
#include <stdio.h>
#include <stdlib.h>

char *read_file(const char *path, size_t *out_size) {
    if (!path) return NULL;
    
    FILE *fp = fopen(path, "rb");
    if (!fp) return NULL;
    
    // get file size
    if (fseek(fp, 0, SEEK_END) != 0) { fclose(fp); return NULL; }
    long size = ftell(fp);
    if (size < 0) { fclose(fp); return NULL; }
    rewind(fp);
    
    // allocate buffer (+1 for null terminator, useful for text)
    char *buf = malloc((size_t)size + 1);
    if (!buf) { fclose(fp); return NULL; }
    
    // read all at once
    size_t read = fread(buf, 1, (size_t)size, fp);
    fclose(fp);
    
    if (read != (size_t)size) { free(buf); return NULL; }
    
    buf[size] = '\0';
    if (out_size) *out_size = (size_t)size;
    return buf;   // caller must free()
}

// usage:
// size_t len;
// char *contents = read_file("myfile.txt", &len);
// if (!contents) { perror("read_file"); exit(1); }
// printf("%s", contents);
// free(contents);
```

---

*that's actually everything. variables, pointers, memory, modern C all the way to C23, bit manipulation, debugging, the works.*

*no cap though — reading this is maybe 20% of learning C. the other 80% is building stuff, screwing it up, running asan, reading the error, fixing it, and doing it again. build a linked list. implement malloc (yes, actually do it). write a simple HTTP server. break things on purpose. use valgrind. understand why it broke.*

*C is honest with you in a way most languages aren't. when it crashes it's your fault, and figuring out why teaches you more than any tutorial. GL man*
