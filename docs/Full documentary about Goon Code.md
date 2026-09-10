# Goon Code: The Complete Guide

A from-scratch, C-like scripting language that runs natively inside SigeonOS — with its own compiler (`gsc`), bytecode container format (`.hit`), and two execution backends.

---

## Table of Contents

1. What is Goon Code?
2. The `.hit` Container Format
3. The Goon Language
4. Built-in Functions
5. Variables and Expressions
6. The `gsc` Compiler
7. Running Your App
8. Two Runtimes: Console vs. GUI
9. The Sigeon VM (Low-Level `.hit`)
10. Complete Example Programs
11. Error Codes and Debugging
12. Under the Hood
13. Limitations and Gotchas

---

## What is Goon Code?

Goon Code is SigeonOS's native application language. It's a deliberately small, C-like language that compiles down to a `.hit` file — the OS's executable format. There are two distinct flavors of `.hit` files:

| Flavor | Magic Header | Runtime | Best For |
|---|---|---|---|
| Goon Code | `GOON0001` | Source-interpreting runtime (`goon_run_hit`) | Drawing GUIs, printing text |
| Sigeon VM | `SIGEON` | Bytecode VM (`vm_execute`) | Low-level pixel/FS/network work |

Goon Code is the friendlier, higher-level path. You write C-like code, the OS compiles it, and it either:

- Prints to a terminal (console mode), or
- Opens its own window and draws shapes, text, and buttons (GUI mode).

The whole design is intentionally compact — the entire language runtime in `kernel.c` is a few hundred lines, because it validates the source and then interprets it statement-by-statement. No external toolchain, no assembler, no linker. You type code on the machine, run `gsc`, and hit `run`.

---

## The `.hit` Container Format

A Goon Code `.hit` file is refreshingly simple:

```
┌────────────────────────────────────────────┐
│  Header (8 bytes): "GOON0001"              │
├────────────────────────────────────────────┤
│  Source code (plain ASCII, N bytes)        │
│  ...                                        │
└────────────────────────────────────────────┘
```

- Header: exactly the 8 ASCII bytes `G O O N 0 0 0 1`.
- Payload: the raw source text of your program. No bytecode, no compression — the source ships as-is.

This means a `.hit` is human-readable with any hex editor, and the runtime can show you the source when something fails.

> The Sigeon VM format uses a different 14-byte header (`SIGEON` + 8 bytes of metadata) followed by actual bytecode opcodes. That's a separate, lower-level system covered later.

### How the OS recognizes a Goon `.hit`

```c
static const char goon_hit_magic[GOON_HIT_HEADER_LEN] =
    {'G','O','O','N','0','0','0','1'};
```

If the first 8 bytes of a file match this, the file is treated as Goon Code. Otherwise, `.hit` extension + `SIGEON` magic routes it to the bytecode VM.

---

## The Goon Language

Goon Code is a statement-per-line language. Every statement must end with a semicolon (`;`), except for structural lines (`{`, `}`, `#include`, `else`).

### Source file conventions

- Extension: `.gc` (Goon Code source)
- Compiled output: `.hit`
- Encoding: plain ASCII
- Comments: lines starting with `//` are ignored
- Preprocessor: only `#include <goon.h>` or `#include "goon.h"` is accepted (and is a no-op — the symbols are all built in)

### Statement rules

The compiler (`goon_compile_source`) enforces a handful of rules:

1. Balanced parentheses and quotes. Strings must be closed, `(` must match `)`.
2. Balanced braces. `{` and `}` must nest correctly.
3. Trailing semicolon required on executable statements.
4. Every statement must be "known". The validator checks against a whitelist of function names and keywords — arbitrary expressions aren't allowed at the top level.
5. Recognized statement forms:
   - Function calls: `print(...)`, `print_num(...)`, `print_hex(...)`, `window(...)`, `text(...)`, `rect(...)`, `circle(...)`, `line(...)`, `button(...)`
   - Variable declarations: `int name = expr;`
   - Control words: `if (...)`, `while (...)`, `for (...)`, `else`, `return;`, `return expr;`
   - Braces `{` / `}`
   - `#include` directives

Unknown statements produce a compile error with the line number.

---

## Built-in Functions

All built-in functions are called as statements. Arguments are comma-separated. String arguments must be double-quoted.

### Console functions (terminal output)

| Function | Args | Description |
|---|---|---|
| `print("text")` | string | Prints text to the terminal, one line per call |
| `print_num(expr)` | integer | Prints an integer value |
| `print_hex(expr)` | integer | Prints a 32-bit hex value to the serial debug console |

### GUI functions (window mode)

These require the program to declare a window via `window(...)` first.

| Function | Args | Description |
|---|---|---|
| `window("title", w, h)` | string, int, int | Declares the app window. Must appear once. Size clamped to 160–900 × 100–650. |
| `text("str", x, y)` | string, int, int | Draws text at window-relative `(x, y)` |
| `rect(color, x, y, w, h)` | int, int, int, int, int | Filled rectangle. `color` is a 24-bit RGB value. |
| `circle(color, x, y, r)` | int, int, int, int | Filled circle |
| `line(color, x1, y1, x2, y2)` | int, int, int, int, int | Straight line |
| `button("label", x, y, w, h)` | string, int, int, int, int | Clickable button (fires terminal print in this build) |

### Color format

Colors are 24-bit RGB integers written in decimal or hex:

```
0xFF0000   // red
0x00FF00   // green
0x0000FF   // blue
0xFFFFFF   // white
0x000000   // black
```

---

## Variables and Expressions

### Declaring a variable

```c
int score = 100;
int x = 0;
int radius = 20;
```

Rules:
- Only the `int` type exists.
- Value must be assigned at declaration (no uninitialized vars).
- Names are case-sensitive and limited to letters, digits, and `_`.

### Expressions

The expression evaluator supports `+`, `-`, `*`, `/` and integer literals (decimal or `0x` hex). Variables can be referenced by name.

```c
int a = 10;
int b = a * 2 + 5;   // b = 25
```

Expression evaluation is single-pass left-to-right — no operator precedence. Parenthesize explicitly if you need a specific order.

```c
int x = 2 + 3 * 4;   // evaluates left to right: (2 + 3) * 4 = 20
int y = 2 + (3 * 4); // still left to right: 2 + 3 then * 4 — same result, be careful
```

> If you need precedence, compute intermediate variables manually. This is a tiny runtime, not a full compiler.

---

## The `gsc` Compiler

`gsc` is the built-in Goon Source Compiler. It runs inside the SigeonOS terminal.

### Syntax

```
gsc <input.gc> -o <output.hit>
```

### Wildcards

Both input and output support the `*` wildcard for batch compilation:

```
gsc *.gc -o *.hit
```

This compiles every `.gc` file in the current directory to a `.hit` with the same stem. `hello.gc` → `hello.hit`.

### What it does

1. Reads the source `.gc` file.
2. Runs `goon_compile_source` to validate:
   - Balanced parens/quotes/braces
   - Line ending in `;` (where required)
   - Every statement is recognized
   - No stray `#include` lines besides `goon.h`
3. On success, creates the output `.hit` with the 8-byte `GOON0001` header followed by the source.
4. On failure, prints a diagnostic like `gsc: line 7: expected ';'`.

### Example session

```
> gsc hello.gc -o hello.hit
Compiled hello.gc -> hello.hit

> run hello.hit
Running: hello.hit
```

---

## Running Your App

### From the terminal

```
> run hello.hit
```

or, as a shortcut:

```
> ./hello.hit
```

The OS loads the `.hit`, checks the `GOON0001` magic, and dispatches to `goon_run_hit`.

### From the Finder / Desktop

Double-click the `.hit` file (or its 3-letter icon on the desktop) — the OS sees the `GOON0001` header and launches it the same way. The app gets a dock icon tied to its file identity, so pinning/reopening works per-file.

---

## Two Runtimes: Console vs. GUI

When a Goon `.hit` runs, `goon_run_hit` scans the source for a `window(...)` call:

- If `window(...)` is present → GUI mode. A real window opens, and drawing statements render inside it.
- If `window(...)` is absent → Console mode. The program opens a Terminal window and prints its output there.

Console mode is useful for scripts and utilities. GUI mode is what you want for apps.

### GUI mode specifics

- Window size is clamped to `[160, 900] × [100, 650]`.
- All drawing coordinates are relative to the window's content area.
- Drawing happens on every frame automatically — you don't need a render loop.
- Buttons currently log to the terminal when clicked (this build).

### Console mode specifics

- A terminal window opens if one isn't focused.
- `print("...")` appends lines to the terminal buffer.
- `print_num(...)` writes an integer as a decimal line.
- `print_hex(...)` writes to the serial debug port only.

---

## The Sigeon VM (Low-Level `.hit`)

If you need *raw* pixel access, filesystem access, or network calls, you use the Sigeon VM format instead. It's a stack-based bytecode VM, not source.

### Container layout

```
┌────────────────────────────────────────────┐
│  Header (14 bytes): "SIGEON" + 8 metadata  │
├────────────────────────────────────────────┤
│  Bytecode (opcodes, variable-length)       │
└────────────────────────────────────────────┘
```

### Opcodes

| Opcode | Name | Operand | Effect |
|---|---|---|---|
| 0 | NOP | — | No-op |
| 1 | PUSH | 4-byte int | Push immediate onto stack |
| 2 | POP | — | Discard top of stack |
| 3 | ADD | — | Pop 2, push sum |
| 4 | SUB | — | Pop 2, push difference |
| 5 | MUL | — | Pop 2, push product |
| 6 | DIV | — | Pop 2, push quotient |
| 7 | JMP | 4-byte addr | Unconditional jump |
| 8 | JZ | 4-byte addr | Jump if top is zero |
| 9 | JNZ | 4-byte addr | Jump if top is non-zero |
| 12 | LOAD | 4-byte hash | Push variable |
| 13 | STORE | 4-byte hash | Pop into variable |
| 14 | SYSCALL | 4-byte hash | Call a built-in |
| 15 | HALT | — | Stop the VM |
| 16 | CMP | — | Compare |
| 17–20 | LT/GT/EQ/NEQ | — | Comparisons |

### Sigeon VM system calls

Dispatch is by hash of the function name (or by index if you prefer). Available calls:

**Drawing**
`draw_pixel`, `draw_rect`, `draw_text`, `draw_text_center`, `draw_circle`, `draw_line`, `draw_rect_outline`

**Windows**
`win_new`, `win_close`, `win_redraw`, `win_center`, `win_title`

**Input**
`mouse_get`, `key_get`

**Time**
`ticks`, `sleep`

**Debug output**
`print`, `print_num`, `print_hex`

**Filesystem**
`fs_read`, `fs_write`, `fs_list`, `fs_mkdir`, `fs_delete`

**Network**
`http_get`, `dns_get`

This format is intentionally lower-level. Most people should use Goon Code source + `gsc` unless they need a specific syscall.

---

## Complete Example Programs

### 1. Hello World (console mode)

```c
#include <goon.h>

print("Hello, SigeonOS!");
print("Goon Code is running.");
```

Compile and run:

```
> gsc hello.gc -o hello.hit
> run hello.hit
```

### 2. Simple window with text

```c
#include <goon.h>

window("My App", 400, 300);
text("Welcome to Goon Code!", 40, 40);
text("Press close to exit.", 40, 60);
```

### 3. Drawing shapes

```c
#include <goon.h>

window("Shapes Demo", 500, 400);

// Background
rect(0xFFFFFF, 0, 0, 500, 400);

// Red circle
circle(0xFF0000, 150, 150, 60);

// Blue rectangle
rect(0x0000FF, 250, 90, 180, 120);

// Green line
line(0x00FF00, 20, 350, 480, 350);
```

### 4. Using variables

```c
#include <goon.h>

window("Math Demo", 400, 300);

int base = 100;
int offset = 40;
int total = base + offset;

rect(0x3366CC, 20, 20, total, 30);

int r = 30;
int cx = 200;
int cy = 150;
circle(0xFFAA00, cx, cy, r);

text("Done.", 20, 250);
```

### 5. Multiple colors grid

```c
#include <goon.h>

window("Color Grid", 480, 360);

rect(0xFF0000,  20,  20, 100, 100);
rect(0x00FF00, 140,  20, 100, 100);
rect(0x0000FF, 260,  20, 100, 100);
rect(0xFFFF00,  20, 140, 100, 100);
rect(0xFF00FF, 140, 140, 100, 100);
rect(0x00FFFF, 260, 140, 100, 100);
rect(0x000000,  20, 260, 100, 100);
rect(0xFFFFFF, 140, 260, 100, 100);
rect(0x888888, 260, 260, 100, 100);
```

### 6. Button demo

```c
#include <goon.h>

window("Buttons", 400, 250);

text("Click a button:", 30, 20);
button("Save",   30, 60, 100, 30);
button("Cancel", 150, 60, 100, 30);
button("Help",   270, 60, 100, 30);
text("Button presses log to the terminal.", 30, 120);
```

### 7. Batch compile example

With three files in the current folder — `a.gc`, `b.gc`, `c.gc` — compile them all at once:

```
> gsc *.gc -o *.hit
Compiled a.gc -> a.hit
Compiled b.gc -> b.hit
Compiled c.gc -> c.hit
```

---

## Error Codes and Debugging

### Compile-time errors

`gsc` reports errors as `gsc: line N: <reason>`. Common reasons:

| Message | Cause |
|---|---|
| `expected ';'` | Statement missing trailing `;` |
| `unbalanced parentheses or string` | Unclosed `(` or `"` on the line |
| `unclosed '{'` | A `{` block was never closed |
| `unexpected '}'` | More `}` than `{` |
| `unknown statement` | Line isn't a recognized call/keyword/assignment |
| `preprocessor: only <goon.h> is supported` | Stray `#define`/`#if` etc. |

### Runtime error codes

Defined near the top of the kernel:

| Constant | Value | Meaning |
|---|---|---|
| `ERR_HIT_INVALID_MAGIC` | −1 | File doesn't start with `GOON0001` or `SIGEON` |
| `ERR_HIT_TOO_SMALL` | −2 | File is under the minimum header size |
| `ERR_HIT_NO_MEM` | −3 | No free app slot (MAX_APPS reached) |
| `ERR_HIT_LOAD_FAIL` | −4 | Couldn't create the window |
| `ERR_HIT_INVALID_POINTER` | −5 | Syscall got a bad pointer (Sigeon VM only) |
| `ERR_HIT_INVALID_OPCODE` | −6 | Unknown bytecode (Sigeon VM only) |

If loading fails, the OS shows an alert dialog: "This file is unable to run. It's either corrupted or something. idk."

### Serial debug output

`print_hex(...)` writes hex values to the serial port at `0x3F8`. You can watch them with any serial monitor at the default baud rate (115200 8N1).

---

## Under the Hood

### The compile path

When you run `gsc hello.gc -o hello.hit`:

1. `split_first_arg` parses the command line.
2. The `-o` flag splits input from output.
3. If input has a `*`, it iterates every `.gc` file in the current directory and generates a matching output name.
4. For each file, `gsc_make_hit(dir, src_name, out_name, term)`:
   - Reads the source node.
   - Calls `goon_compile_source(source, len, err, err_max)`.
   - On success, allocates an output node and writes:
     - 8 bytes of `GOON0001` magic
     - the raw source (up to `FS_FILE_MAX - 8` bytes)

### The run path

When you run a `.hit`:

1. `open_file_in_app(fs_idx)` (or the terminal's `run`) checks the first 8 bytes for `GOON0001`.
2. If matched, `dock_add_dynamic_item(APP_HIT_EXEC, fs_idx, name)` registers the app.
3. `goon_run_hit(NULL_PTR, fs_idx)` is called:
   - If a matching window is already open, it's focused.
   - Otherwise, `goon_source_window_info` scans for `window(...)`.
   - GUI mode: `win_alloc()` creates a window, its size comes from `window(...)`.
   - Console mode: a Terminal is opened and `goon_execute_console` prints.
4. In GUI mode, `draw_app_goon(w, cx, cy, cw, ch)` runs every frame:
   - Rebuilds the variable table from `int` declarations.
   - Walks each line and dispatches to the drawing primitives.
   - Buttons are hit-tested by the mouse handler against `goon_buttons[][]`.

### Why source-interpretation, not compilation?

Two reasons:

- **Auditability.** A `.hit` is plain text — the OS can show you exactly what it's about to run.
- **Simplicity.** The whole runtime is a few hundred lines. No register allocation, no instruction encoding, no optimizer.

It's slower than bytecode, but for GUI apps that draw a few dozen primitives per frame, it's more than fast enough.

---

## Limitations and Gotchas

- Only one type: `int`. No floats, no strings-as-variables, no arrays.
- Expressions evaluate left-to-right with no precedence. Parenthesize deliberately.
- Statements are line-oriented. You can't put two statements on one line.
- No user-defined functions. There is no `func` keyword.
- Loops (`if`/`while`/`for`) are recognized by the validator but not actually executed by the current interpreter — they're reserved for a future release. Use repeated statements or the Sigeon VM for real control flow.
- `window(...)` must appear at most once. Multiple windows per `.hit` aren't supported.
- Window size clamps to 900×650. Larger sizes get silently reduced.
- Buttons currently log to a Terminal instead of triggering callbacks. Callback wiring is a future feature.
- Source size limit is `FS_FILE_MAX - 8` bytes (~504 bytes). Keep programs short.
- No includes beyond `goon.h`. All symbols are built in.
- No comments other than `//` full-line. `/* */` blocks are not recognized.
- Whitespace is trimmed aggressively. Don't rely on indentation.

---

## Quick Reference Card

```
SOURCE FILE          .gc
OUTPUT FILE          .hit
MAGIC                "GOON0001" (8 bytes)

COMPILE              gsc <file.gc> -o <file.hit>
BATCH COMPILE        gsc *.gc -o *.hit

RUN                  run <file.hit>
RUN (SHORTCUT)       ./<file.hit>

CONSOLE OUTPUT       print("text");
                     print_num(expr);
                     print_hex(expr);      // serial only

WINDOW               window("title", w, h);

DRAWING              text("str", x, y);
                     rect(color, x, y, w, h);
                     circle(color, x, y, r);
                     line(color, x1, y1, x2, y2);
                     button("label", x, y, w, h);

VARIABLES            int name = expr;
                     int a = 10;
                     int b = a + 5;

COLORS               0xRRGGBB
                     0xFF0000 red, 0x00FF00 green, 0x0000FF blue
                     0xFFFFFF white, 0x000000 black

ERRORS               gsc: line N: <reason>          (compile)
                     Alert popup with error code     (runtime)
```

---

*Goon Code — because sometimes you just want to draw a rectangle and call it a day.*

<sub>This was written by AI. I am not writing a full document</sub>
