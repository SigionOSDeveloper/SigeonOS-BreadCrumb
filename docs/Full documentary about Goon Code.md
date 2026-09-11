# The Complete Goon Code Handbook

**A beginner-friendly, no-nonsense guide to writing Goon Code (`.gc`) programs that compile to `.hit` files and run on SigeonOS.**

<hr>
This was written by AI, expect idk.
<hr>


> **Read this first:** Goon Code is the little C-like language that ships with SigeonOS. You write it, you compile it with `gsc`, you get a `.hit` file, and that `.hit` file becomes an executable app — either a **console program** (text output) or a **windowed app** (draws to a window on the desktop). That's the whole game. Everything below is just filling in the details.

---

## Table of Contents

1. [What even is Goon Code?](#what-even-is-goon-code)
2. [The Two Flavours of Goon Code](#the-two-flavours-of-goon-code)
3. [Your First Program — Hello World](#your-first-program--hello-world)
4. [Compiling with `gsc`](#compiling-with-gsc)
5. [Running a `.hit` File](#running-a-hit-file)
6. [Basic Syntax Rules](#basic-syntax-rules)
7. [Variables and Types](#variables-and-types)
8. [Operators](#operators)
9. [Strings and Characters](#strings-and-characters)
10. [Control Flow — `if`, `else`, `while`](#control-flow--if-else-while)
11. [Functions](#functions)
12. [The Full Built-in Function List](#the-full-built-in-function-list)
13. [Console Programs](#console-programs)
14. [Windowed (GUI) Programs](#windowed-gui-programs)
15. [Drawing Reference — Colours and Coordinates](#drawing-reference--colours-and-coordinates)
16. [Filesystem Programming](#filesystem-programming)
17. [Network Programming](#network-programming)
18. [Permissions — What They Are and Why They Exist](#permissions--what-they-are-and-why-they-exist)
19. [Error Codes and What They Mean](#error-codes-and-what-they-mean)
20. [Full Worked Examples](#full-worked-examples)
21. [Common Mistakes and How to Fix Them](#common-mistakes-and-how-to-fix-them)
22. [Quick Reference Cheat Sheet](#quick-reference-cheat-sheet)

---

## What even is Goon Code?

Goon Code is a **small, C-like programming language** designed to run on SigeonOS. It is deliberately simple:

- It looks like C — curly braces, semicolons, `int main()`.
- It compiles down to a bytecode format stored in a `.hit` file.
- The SigeonOS kernel contains a tiny virtual machine (the "Sigeon VM") that loads and runs that bytecode.
- Programs can either **print text** to a terminal, or **draw graphics** inside a window, or both.

If you've ever used C, Go, or JavaScript, you already know 90% of the syntax. Goon Code is basically "C but only the fun parts and with training wheels."

**Things Goon Code is NOT:**

- It is **not** a full C compiler. No pointers, no `struct`, no `malloc`.
- It is **not** Python. There's no `for` loop right now (use `while`).
- It is **not** garbage collected. It has a fixed 8 KB string pool and fixed stacks.
- It is **not** fast. It runs inside a VM with a step budget per frame.

Keep those limits in mind and you'll have a great time.

---

## The Two Flavours of Goon Code

There are **two ways** to write Goon Code programs, and you need to know which one you're writing before you type a single character.

### Flavour 1 — Compiled Goon Code (the real one)

You write `.gc` source, run it through `gsc`, and get a `.hit` bytecode file. This is the full-featured path. It supports:

- Functions with typed parameters and return values
- `if` / `else` / `while`
- All the drawing, filesystem, and network built-ins
- Automatic permission tracking

**This is what you should use 99% of the time.**

### Flavour 2 — Interpreted Goon Code (the quick-and-dirty one)

Some `.hit` files carry a `GOON0001` magic header and hold raw Goon Code source inside. These are read line-by-line by a tiny interpreter in the kernel. This mode is very limited:

- Only `print()`, `print_num()`, `print_hex()`, and simple `int x = ...` assignments work in console mode
- Only `window()`, `text()`, `rect()`, `circle()`, `line()`, and `button()` work in window mode
- No `if`, no `while`, no functions, no return values

**Treat it as a legacy curiosity.** If you want a real program, compile it.

Everything in the rest of this document assumes you are writing **compiled Goon Code** — the real thing.

---

## Your First Program — Hello World

Create a file called `hello.gc` with this content:

```c
int main() {
    print("Hello, world!");
    return 0;
}
```

That's it. That's the whole program.

- `int main()` — every Goon Code program **must** have exactly one `main` function, it must return `int`, and it must take zero arguments. No exceptions. The OS looks for it first.
- `print(...)` — prints text to the serial console.
- `return 0;` — every function ends when it returns, or when the closing brace is reached.

---

## Compiling with `gsc`

Open a **Terminal** window on the SigeonOS desktop and run:

```
gsc hello.gc -o hello.hit
```

- `gsc` is the Goon Source Compiler.
- The first argument is your `.gc` source file.
- `-o <name>` tells it what to call the output `.hit` file.

If it succeeds, you'll see:

```
gsc: 0
```

If it fails, you'll see something like:

```
gsc: line 4: expected ';'
```

That tells you the exact line where the parser choked. Fix it and recompile.

### Wildcard compiling

You can compile every `.gc` in the current directory at once:

```
gsc *.gc -o *.hit
```

The `*` in the output name is replaced by the source file's basename (without the `.gc`). So `apple.gc` becomes `apple.hit`, `banana.gc` becomes `banana.hit`, and so on.

You can also mix and match — for example:

```
gsc *.gc -o build-*.hit
```

would produce `build-apple.hit`, `build-banana.hit`, etc.

---

## Running a `.hit` File

From the terminal, from inside the directory that contains the file:

```
run hello.hit
```

or equivalently:

```
./hello.hit
```

If it's a console program, the output appears right there in the terminal. If it opens a window, a window appears on the desktop. If it's broken, you get a "Hit file notice" alert saying the file is corrupted or unloadable.

You can also **double-click** a `.hit` file in Finder, or on the desktop, and it will launch.

To **pin** a `.hit` file to the dock, right-click it and choose **Pin to Dock**.

---

## Basic Syntax Rules

These are the rules. Memorise them. They are non-negotiable.

### Statements end with `;`

```c
print("one");
print("two");
print("three");
```

Every statement ends with a semicolon. If you forget one, you get a compiler error on the next line.

### Blocks are wrapped in `{ }`

```c
int main() {
    print("inside the block");
    return 0;
}
```

The opening brace starts a block; the closing brace ends it. Functions, `if` bodies, and `while` bodies all use braces.

### Comments

Two styles, both work:

```c
// This is a single-line comment.

/*
   This is a multi-line comment.
   It can span many lines.
*/
```

### Whitespace is free

You can indent however you want. Tabs, spaces, nothing — the compiler doesn't care. Indent for humans, not for the machine.

### Case matters

`int` is not `INT`. `print` is not `Print`. `main` is not `Main`. Be exact.

---

## Variables and Types

Goon Code has **five** built-in types. That's it. Five. No more.

| Type     | What it stores              | Example                    |
|----------|-----------------------------|----------------------------|
| `int`    | A signed 32-bit integer     | `int x = 42;`              |
| `uint`   | An unsigned 32-bit integer  | `uint y = 4000000000;`     |
| `bool`   | `true` or `false`           | `bool ready = true;`       |
| `char`   | A single character          | `char c = 'A';`            |
| `string` | Text (quoted with `"`)      | `string s = "hello";`      |

### Declaring a variable

```c
int score = 100;
string name = "Sigeon";
bool alive = true;
char grade = 'A';
```

### Declaring without initialising

```c
int counter;
```

`counter` exists but its value is undefined until you assign it. Assign before you read.

### Assigning later

```c
int score;
score = 10;
score = score + 5;   // now 15
```

### Why do I have to say the type?

Because Goon Code does not do type inference. You write the type. That's the deal.

### Numbers

Decimal and hex work:

```c
int a = 255;
int b = 0xFF;      // same as 255
```

There are no floats. No `1.5`. No `3.14`. Integers only.

---

## Operators

Goon Code has the usual C-style operators.

### Arithmetic

| Operator | Meaning             | Example          |
|----------|---------------------|------------------|
| `+`      | Add                 | `3 + 4` → `7`    |
| `-`      | Subtract            | `10 - 3` → `7`   |
| `*`      | Multiply            | `5 * 5` → `25`   |
| `/`      | Divide              | `10 / 3` → `3`   |
| `%`      | Modulo (remainder)  | `10 % 3` → `1`   |

**Important:** division by zero does **not** crash the program. `a / 0` evaluates to `0`, and `a % 0` evaluates to `0`. This is deliberate — it keeps broken programs from killing the whole OS.

### Comparison

| Operator | Meaning                |
|----------|------------------------|
| `==`     | Equal to               |
| `!=`     | Not equal to           |
| `<`      | Less than              |
| `<=`     | Less than or equal     |
| `>`      | Greater than           |
| `>=`     | Greater than or equal  |

All comparison operators produce a `bool`.

### Logical

| Operator | Meaning          |
|----------|------------------|
| `&&`     | Logical AND      |
| `||`     | Logical OR       |
| `!`      | Logical NOT      |

### Unary minus

```c
int x = -5;
```

### String concatenation with `+`

```c
string greeting = "Hello, " + "world!";
```

If either side is a string, `+` does concatenation, not addition.

### Assignment

Just `=`. There is no `+=`, `-=`, `++`, or `--`. Write it out longhand:

```c
counter = counter + 1;
```

---

## Strings and Characters

### String literals

Double quotes, escape sequences supported:

```c
string a = "normal text";
string b = "line one\nline two";
string c = "tab\there";
string d = "quote \"inside\"";
```

Supported escapes: `\n` (newline), `\t` (tab), `\r` (carriage return), `\"` (quote), `\\` (backslash).

### Character literals

Single quotes, one character:

```c
char letter = 'A';
char newline = '\n';
char quote = '\'';
```

### Getting the length of a string

```c
int len = str_len("hello");   // 5
```

### Converting a number to a string

```c
string s = to_string(42);     // "42"
```

### String equality

Use `==` and `!=`. The compiler emits string comparison opcodes.

```c
if (name == "Sigeon") {
    print("matches");
}
```

Strings are **immutable-ish**: you don't append to them in-place. Concatenation creates a new one in the string pool.

---

## Control Flow — `if`, `else`, `while`

### `if`

```c
if (score > 100) {
    print("High score!");
}
```

### `if` / `else`

```c
if (health > 0) {
    print("Alive");
} else {
    print("Dead");
}
```

### `if` / `else if` / `else`

Goon Code does **not** have `else if` as a keyword. Chain `if` inside `else`:

```c
if (score >= 90) {
    print("A");
} else {
    if (score >= 80) {
        print("B");
    } else {
        if (score >= 70) {
            print("C");
        } else {
            print("F");
        }
    }
}
```

Yes, it's verbose. Yes, that's the price of a simple compiler.

### `while`

```c
int i = 0;
while (i < 10) {
    print_num(i);
    i = i + 1;
}
```

There is **no** `for` loop in Goon Code. Use `while`.

There is **no** `break` or `continue`. Structure your loops so you don't need them.

---

## Functions

You can define your own functions. They can take parameters and return values.

### A simple function with no return value

```c
void greet() {
    print("Hello!");
}

int main() {
    greet();
    return 0;
}
```

### A function with parameters

```c
void greet_person(string name, int age) {
    print("Hello, ");
    print(name);
    print("! You are ");
    print_num(age);
    println(" years old.");
}

int main() {
    greet_person("Alice", 30);
    greet_person("Bob", 25);
    return 0;
}
```

### A function that returns a value

```c
int double_it(int x) {
    return x * 2;
}

int main() {
    print_num(double_it(21));   // prints 42
    return 0;
}
```

### Rules for functions

- Must have a return type — one of `int`, `uint`, `bool`, `char`, `string`, or `void`.
- `void` means "returns nothing."
- Non-`void` functions **must** return a value on every code path (the compiler assumes this and will warn you if not, but the safety net is limited).
- Parameters are typed and named.
- Up to 16 parameters.
- `main` must be `int main()` — no parameters, returns `int`.

### Calling order does not matter

The compiler scans all functions first, so you can call a function that appears later in the file. Nice.

---

## The Full Built-in Function List

These functions are provided by the OS. You do not define them. You just call them. The compiler automatically figures out which **permissions** you need (see the Permissions section below).

### Output / strings

| Function | Returns | Description |
|----------|---------|-------------|
| `print(x)` | `void` | Print `x` to the serial console. |
| `println(x)` | `void` | Print `x` followed by a newline. |
| `to_string(x)` | `string` | Convert an integer-ish value to a string. |
| `str_len(s)` | `int` | Length of a string. |

### Time / scheduling

| Function | Returns | Description |
|----------|---------|-------------|
| `ticks()` | `int` | Number of timer ticks since boot (100 ticks = 1 second). |
| `sleep(ms)` | `int` | Sleep for `ms` ticks (roughly milliseconds). Returns `1`. |

### Filesystem

| Function | Returns | Permission | Description |
|----------|---------|------------|-------------|
| `fs_read(path)` | `string` | FS read | Read a file's contents. Returns `""` if it doesn't exist. |
| `fs_write(path, data)` | `bool` | FS write | Write `data` to `path`. Creates the file if needed. |
| `fs_mkdir(path)` | `bool` | FS write | Create a directory. |
| `fs_delete(path)` | `bool` | FS delete | Delete a file or directory. |
| `fs_exists(path)` | `bool` | FS read | Does the path exist? |
| `fs_list(path)` | `string` | FS read | List the contents of a directory, newline-separated. |

### Network

| Function | Returns | Permission | Description |
|----------|---------|------------|-------------|
| `http_get(url)` | `string` | Network | Fetch an HTTP URL. Returns the body. |
| `dns_get(hostname)` | `string` | Network | Resolve a hostname to an IP as text. |

### Graphics (window / drawing)

| Function | Returns | Permission | Description |
|----------|---------|------------|-------------|
| `window(title, w, h)` | `int` | UI | Create a window. Returns a window handle. |
| `draw_pixel(color, x, y)` | `int` | UI | Plot a single pixel. |
| `draw_rect(color, x, y, w, h)` | `int` | UI | Draw a filled rectangle. |
| `draw_text(text, color, x, y)` | `int` | UI | Draw a string at a position. |
| `draw_circle(color, x, y, r)` | `int` | UI | Draw a filled circle. |
| `draw_line(color, x1, y1, x2, y2)` | `int` | UI | Draw a line. |

**Note the argument order for `draw_text`** — it's `(text, color, x, y)`, not `(text, x, y, color)`. Learn it.

---

## Console Programs

A console program is a Goon Code program that doesn't create a window. It just prints.

### Example — greeting script

```c
int main() {
    print("What is your name?\n");
    // (Note: this compiler does not have keyboard input built in,
    //  so we just greet a hard-coded name.)
    string name = "traveler";
    print("Hello, ");
    println(name);
    return 0;
}
```

### Example — counting

```c
int main() {
    int i = 1;
    while (i <= 10) {
        print_num(i);
        print(" ");
        i = i + 1;
    }
    println("");
    return 0;
}
```

Output:

```
1 2 3 4 5 6 7 8 9 10 
```

### Example — simple maths helper

```c
int square(int x) {
    return x * x;
}

int cube(int x) {
    return x * x * x;
}

int main() {
    int n = 5;
    print("n = ");       println(to_string(n));
    print("n^2 = ");     println(to_string(square(n)));
    print("n^3 = ");     println(to_string(cube(n)));
    return 0;
}
```

---

## Windowed (GUI) Programs

A GUI program creates a window and draws into it. The window behaves like any other SigeonOS window — you can drag it, minimise it, close it.

### The simplest possible window

```c
int main() {
    window("My First App", 400, 300);
    return 0;
}
```

That gives you an empty 400×300 window titled "My First App".

### Drawing into the window

```c
int main() {
    window("Drawing Demo", 400, 300);

    // A filled red rectangle at (50, 50), 100×80
    draw_rect(0xFF0000, 50, 50, 100, 80);

    // A filled blue circle at (250, 150) with radius 40
    draw_circle(0x0000FF, 250, 150, 40);

    // Text in white at (20, 240)
    draw_text("Hello, Goon Code!", 0xFFFFFF, 20, 240);

    return 0;
}
```

### Drawing a line

```c
int main() {
    window("Line Demo", 300, 300);
    draw_line(0x00FF00, 0, 0, 299, 299);   // diagonal from top-left to bottom-right
    return 0;
}
```

### Putting it together — animated bar

Since Goon Code has no direct frame loop other than `main`, you can use `sleep` to control animation within `main`:

```c
int main() {
    window("Loading Bar", 420, 120);

    int i = 0;
    while (i <= 300) {
        // Clear the bar area
        draw_rect(0x222222, 60, 50, 300, 20);
        // Draw the filled portion
        draw_rect(0x00CC44, 60, 50, i, 20);
        // Label
        draw_text("Loading...", 0xFFFFFF, 60, 30);
        // Wait a tick
        sleep(1);
        i = i + 3;
    }

    draw_text("Done!", 0xFFFFFF, 60, 90);
    return 0;
}
```

**How this works:** the OS repaints windows every frame. Your program draws, sleeps a tick, draws again, sleeps again. The result is animation.

> **Note:** since the VM runs with a per-frame step budget, long loops with `sleep(1)` are the friendly way to animate. Avoid tight infinite loops.

---

## Drawing Reference — Colours and Coordinates

### Colours

Colours are 24-bit RGB packed into an `int`:

```
0xRRGGBB
```

Examples:

| Colour  | Value       |
|---------|-------------|
| Black   | `0x000000`  |
| White   | `0xFFFFFF`  |
| Red     | `0xFF0000`  |
| Green   | `0x00FF00`  |
| Blue    | `0x0000FF`  |
| Yellow  | `0xFFFF00`  |
| Cyan    | `0x00FFFF`  |
| Magenta | `0xFF00FF`  |
| Orange  | `0xFF8800`  |
| Grey    | `0x808080`  |

### Coordinates

- `(0, 0)` is the **top-left** of the window's content area.
- `x` increases to the right.
- `y` increases downward.
- The content area starts *below* the title bar, so y=0 is right under the title.

### Window size limits

`window(title, w, h)` clamps your requested size to safe bounds:

- Minimum width: 160
- Minimum height: 100
- Maximum width: 900
- Maximum height: 650

If you ask for less, you get the minimum. If you ask for more, you get the maximum.

---

## Filesystem Programming

Filesystem paths are absolute, starting with `/`. The root holds standard directories:

```
/
├── Desktop/
├── Documents/
├── Downloads/
└── Applications/
```

### Reading a file

```c
int main() {
    string content = fs_read("/Documents/notes.txt");
    if (str_len(content) > 0) {
        println("File contents:");
        println(content);
    } else {
        println("File not found or empty.");
    }
    return 0;
}
```

### Writing a file

```c
int main() {
    bool ok = fs_write("/Documents/hello.txt", "This was written by Goon Code!");
    if (ok) {
        println("Write successful.");
    } else {
        println("Write failed.");
    }
    return 0;
}
```

### Creating a directory

```c
int main() {
    if (fs_mkdir("/Documents/MyFolder")) {
        println("Created.");
    } else {
        println("Could not create (maybe it already exists).");
    }
    return 0;
}
```

### Checking existence

```c
int main() {
    if (fs_exists("/Desktop/Sigeon.gsv")) {
        println("Yes, the image is there.");
    } else {
        println("No, it's missing.");
    }
    return 0;
}
```

### Listing a directory

```c
int main() {
    string listing = fs_list("/");
    println("Root contains:");
    println(listing);
    return 0;
}
```

`fs_list` returns names separated by `\n`.

### Deleting a file

```c
int main() {
    if (fs_delete("/Documents/old.txt")) {
        println("Deleted.");
    } else {
        println("Couldn't delete — maybe it doesn't exist.");
    }
    return 0;
}
```

Protected paths (the root, Desktop, Documents, Downloads, Applications themselves) cannot be deleted. That's a feature, not a bug.

---

## Network Programming

Goon Code can make HTTP requests and resolve DNS. Every network call is **blocking** — the program waits for the response.

### HTTP GET

```c
int main() {
    string body = http_get("http://example.com/");
    if (str_len(body) > 0) {
        println("Got response:");
        println(body);
    } else {
        println("No response (network down, DNS failed, or URL bad).");
    }
    return 0;
}
```

### DNS resolution

```c
int main() {
    string ip = dns_get("example.com");
    if (str_len(ip) > 0) {
        print("example.com resolves to ");
        println(ip);
    } else {
        println("Could not resolve.");
    }
    return 0;
}
```

### Realistic example — check if a host is up

```c
int main() {
    string ip = dns_get("example.com");
    if (str_len(ip) == 0) {
        println("Host unreachable — DNS failed.");
        return 1;
    }
    print("Resolved to ");
    println(ip);

    string page = http_get("http://example.com/");
    if (str_len(page) > 0) {
        println("Host is up.");
        return 0;
    } else {
        println("Host resolved but HTTP failed.");
        return 1;
    }
}
```

### Rules for network access

- Network calls need the **NETWORK** permission (auto-added by the compiler when you use `http_get` or `dns_get`).
- Only plain **HTTP** is supported. Not HTTPS. There is no TLS stack in this OS.
- DNS uses the resolver configured by DHCP, then falls back to Google DNS (8.8.8.8) and Cloudflare (1.1.1.1).
- Long responses are truncated into the kernel's response buffer.

---

## Permissions — What They Are and Why They Exist

SigeonOS is a **capability-based** system. When your program is compiled, the compiler walks the whole program and records which capability bits it needs. These bits are stored in the `.hit` header. When the program tries to use a capability it hasn't been granted yet, the OS shows a **permission prompt** — and only a mouse click decides.

### The permission bits

| Bit | Name      | Granted by calling… |
|-----|-----------|---------------------|
| 0   | FS read   | `fs_read`, `fs_exists`, `fs_list` |
| 1   | FS write  | `fs_write`, `fs_mkdir` |
| 2   | FS delete | `fs_delete` |
| 3   | Network   | `http_get`, `dns_get` |
| 4   | UI        | `window`, `draw_pixel`, `draw_rect`, `draw_text`, `draw_circle`, `draw_line` |
| 5   | Hardware  | (reserved) |

### What this means for you

- **You don't declare permissions.** The compiler figures it out.
- **You don't need to check them.** If a permission is denied, the built-in returns `0` or `""` gracefully.
- **You can't spoof them.** The header is validated before the program runs.

### The user experience

The first time a program calls a network function, the user sees:

> **Permission required**
> Application requests network access.
> Make network requests.
> `[ Allow ]` `[ Disallow ]`

The user must click **Allow** with the mouse. `Enter` and `Esc` are ignored — that's intentional, so you can't accidentally approve something by typing.

If the user clicks Disallow, subsequent calls to the same capability return failure values immediately.

---

## Error Codes and What They Mean

When a `.hit` file fails to load, the OS shows a **"Hit file notice"** with an error code. Here's the decoder ring.

| Code | Name                     | Meaning |
|------|--------------------------|---------|
| -1   | `ERR_HIT_INVALID_MAGIC`  | The file doesn't start with the right magic bytes. Not a `.hit` file, or corrupted. |
| -2   | `ERR_HIT_TOO_SMALL`      | The file is shorter than the minimum header size. |
| -3   | `ERR_HIT_NO_MEM`         | Too many apps loaded, or the file is too big for the blob pool. |
| -4   | `ERR_HIT_LOAD_FAIL`      | The header is present but invalid (wrong version, bad permission bits, malformed structure). |
| -5   | `ERR_HIT_INVALID_POINTER`| A string constant in the bytecode points outside the string pool. Recompile. |
| -6   | `ERR_HIT_INVALID_OPCODE` | The bytecode contains an unknown opcode, or a jump lands mid-instruction. Recompile. |

Almost always, codes -5 and -6 mean the `.hit` file was produced by an older/different version of `gsc`, or was hand-edited. **Recompile from source.**

If your program runs but hits a runtime fault, the VM stops just that app — the OS itself keeps running. This is the "application fault containment" feature, and it's why a buggy program can't take down your desktop.

---

## Full Worked Examples

### Example 1 — Fibonnaci printer

```c
int fib(int n) {
    if (n < 2) {
        return n;
    }
    return fib(n - 1) + fib(n - 2);
}

int main() {
    int i = 0;
    while (i < 15) {
        print("fib(");
        print_num(i);
        print(") = ");
        println(to_string(fib(i)));
        i = i + 1;
    }
    return 0;
}
```

### Example 2 — Write then read a file

```c
int main() {
    string path = "/Documents/goon-test.txt";
    string message = "Written by Goon Code at " + to_string(ticks()) + " ticks.";

    if (!fs_write(path, message)) {
        println("Write failed.");
        return 1;
    }
    println("Wrote the file.");

    string readback = fs_read(path);
    println("Read back:");
    println(readback);
    return 0;
}
```

### Example 3 — Draw a checkerboard

```c
int main() {
    window("Checkerboard", 400, 400);

    int cell = 40;
    int y = 0;
    while (y < 10) {
        int x = 0;
        while (x < 10) {
            int color;
            if (((x + y) % 2) == 0) {
                color = 0xFFFFFF;
            } else {
                color = 0x000000;
            }
            draw_rect(color, x * cell, y * cell, cell, cell);
            x = x + 1;
        }
        y = y + 1;
    }
    return 0;
}
```

### Example 4 — Simple HTTP page reader

```c
int main() {
    println("Fetching example.com ...");

    string body = http_get("http://example.com/");

    if (str_len(body) == 0) {
        println("Could not fetch. Check the network.");
        return 1;
    }

    println("Response body:");
    println(body);
    return 0;
}
```

### Example 5 — Mini dashboard

```c
int main() {
    window("Dashboard", 500, 320);

    while (1 == 1) {
        int now = ticks();

        // Background
        draw_rect(0x101820, 0, 0, 500, 320);

        // Title
        draw_text("SIGEON DASHBOARD", 0x00FFAA, 20, 20);

        // Uptime bar
        draw_text("Uptime (ticks):", 0xFFFFFF, 20, 60);
        draw_rect(0x222222, 20, 80, 460, 24);
        int progress = now % 460;
        draw_rect(0x00AAFF, 20, 80, progress, 24);

        // Numeric readout
        draw_text(to_string(now), 0xFFFFFF, 20, 120);

        sleep(1);
    }
    return 0;
}
```

### Example 6 — Sort an array (well, a fixed set of variables)

Goon Code has no arrays, so use a fixed set and swap values manually:

```c
int main() {
    int a = 9;
    int b = 3;
    int c = 7;

    // Sort a, b, c ascending via nested swaps
    if (a > b) { int t = a; a = b; b = t; }
    if (b > c) { int t = b; b = c; c = t; }
    if (a > b) { int t = a; a = b; b = t; }

    print_num(a); print(" ");
    print_num(b); print(" ");
    println(to_string(c));
    return 0;
}
```

Output:

```
3 7 9
```

---

## Common Mistakes and How to Fix Them

### 1. "expected ';'"

You forgot a semicolon. Add one. Yes, on that line. That one.

### 2. "unknown variable"

You either:

- Misspelled the variable name,
- Forgot to declare it, or
- Declared it inside a block and tried to use it outside that block.

Variables are scoped to the block they're declared in.

### 3. "type mismatch"

You're assigning something of the wrong type. For example, storing a `string` in an `int`:

```c
int x = "hello";   // wrong
int x = 5;         // right
```

Or passing the wrong type into a function whose parameter type doesn't match.

### 4. "wrong number of arguments"

You called a built-in with the wrong number of arguments. Check the built-in table above.

### 5. "unknown function"

You called something that isn't a built-in and isn't defined in your program. Check spelling, and remember that `main` doesn't call a function that doesn't exist.

### 6. "program must define int main()"

You either:

- Named it `Main`, `MAIN`, or `start`,
- Gave it parameters, or
- Forgot to define it entirely.

The correct signature is exactly:

```c
int main() { ... }
```

### 7. "argument type mismatch"

You passed, say, a `string` to a function that wants an `int`. Check both sides.

### 8. Program compiles but does nothing visible

- A windowed program opens a window. A console program prints to the terminal.
- If nothing appears, maybe your drawing is off-screen, or your `main` returns immediately.
- Try adding a `sleep(50)` at the end so you can see the window before it closes.

### 9. Division surprise

`7 / 2` is `3`, not `3.5`. That's integer division. If you need fractional maths, multiply by 100 and keep the scale in your head.

### 10. String comparison surprises

Use `==` and `!=` — they work on strings correctly. Do **not** try to compare string pointers or use `<` on strings. Only `==` and `!=` are meaningful for strings.

---

## Quick Reference Cheat Sheet

```c
// ---- Program skeleton ----
int main() {
    return 0;
}

// ---- Variables ----
int x = 5;
uint u = 100;
bool flag = true;
char c = 'A';
string s = "text";

// ---- Printing ----
print("hello");
println("hello with newline");
print_num(42);
print_hex(0xDEADBEEF);

// ---- Strings ----
string greeting = "Hello, " + "world!";
int len = str_len(greeting);
string s = to_string(123);

// ---- Control flow ----
if (x > 0) {
    // do this
} else {
    // do that
}

while (x < 10) {
    x = x + 1;
}

// ---- Functions ----
int add(int a, int b) {
    return a + b;
}

void say_hello() {
    println("hello");
}

// ---- Graphics ----
window("Title", 400, 300);
draw_pixel(0xFFFFFF, 10, 10);
draw_rect(0xFF0000, 20, 20, 100, 50);
draw_circle(0x00FF00, 200, 150, 40);
draw_line(0x0000FF, 0, 0, 400, 300);
draw_text("hi", 0xFFFFFF, 50, 50);

// ---- Filesystem ----
string data = fs_read("/Documents/file.txt");
bool ok = fs_write("/Documents/file.txt", "hello");
fs_mkdir("/Documents/NewFolder");
bool exists = fs_exists("/Documents/file.txt");
string listing = fs_list("/");
fs_delete("/Documents/file.txt");

// ---- Network ----
string body = http_get("http://example.com/");
string ip = dns_get("example.com");

// ---- Time ----
int now = ticks();
sleep(10);   // about 100 ms

// ---- Comments ----
// single-line
/* multi-line */
```

---

## Closing Words

Goon Code is **not** trying to be C. It's not trying to be Rust. It's not trying to be your next favourite language.

It's a small, blunt, friendly tool for making tiny apps on a hobby operating system. It gives you enough rope to draw a rectangle, write a file, fetch a URL, and print a hello. Everything else is up to you.

If you remember nothing else, remember these three things:

1. **`int main()` is the entry point.** Always.
2. **Every statement ends with `;` and every block uses `{ }`.**
3. **`gsc file.gc -o file.hit` compiles. `run file.hit` runs.**

Everything else is just details, and the details are all above.

Now go build something goofy.
