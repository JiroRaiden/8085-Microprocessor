# 8085 Lab

**[Live demo →](https://8085-microprocessor.vercel.app)** · [Lab Bench](https://8085-microprocessor.vercel.app/lab-bench.html) · [Trainer Kit](https://8085-microprocessor.vercel.app/trainer-kit.html)

An Intel 8085 assembler, CPU emulator and trainer-kit simulator that runs entirely in the browser.
Built for microprocessor lab practice: write a program, run it instruction by instruction, watch the
registers and flags, and have your answer checked automatically.

Three files, no build step, no dependencies, no backend.

```
index.html         landing page
lab-bench.html     assembler + simulator + auto-graded lab assignments
trainer-kit.html   trainer-kit emulator (six-digit display + hex keypad)
```

---

## Running it

**Locally** — double-click `index.html`. That's it. Everything works from `file://`.

**Deployed** — see [Deploying](#deploying) at the bottom.

---

## Part 1 — The Lab Bench

`lab-bench.html` is where you write assembly and get it checked. The page is one screen with four
regions, top to bottom.

### The machine strip

The dark band across the top is the 8085 programmer's model, live:

| Zone | Shows |
|---|---|
| **Accumulator** | A in hex, decimal and binary |
| **Flag register** | the PSW byte laid out as it really is — `S Z 0 AC 0 P 1 CY`, with bits 5 and 3 always 0 and bit 1 always 1 |
| **Registers** | BC, DE and HL as pairs (both bytes plus the 16-bit value), SP, PC, and `(HL)` — the byte currently pointed at |
| **Execution** | instructions executed, T-states, elapsed time at 3.072 MHz, last `OUT` |

A register pair flashes when it changes, so during single-stepping you can see what each instruction
actually touched.

### Writing and running a program

1. Type your program into **Source program**. Origin is `2000H` unless you write your own `ORG`.
2. Press **Assemble**. Errors are reported with a line number; on success the **Hand-assembly
   listing** fills with the address / opcode / mnemonic table — the same table you write out by hand
   in the record book.
3. Press **Run** to execute to the end, or **Step** to execute one instruction at a time. The
   **Clock** slider sets the run speed; the slowest setting is slow enough to watch.
4. **Reset** clears the registers and sets PC back to the start. Memory is kept.

A program ends at `HLT` or at any `RST n` — the RST vectors at `0000H`–`0038H` are seeded with `HLT`,
the way a real kit's monitor sits there, so programs written for a trainer board stop correctly.

### Memory

The hex dump below the editor is editable. Click any cell and type two hex digits to change a byte —
that is how you put input data in place before running. Bytes your program wrote are highlighted, and
the byte at PC is outlined.

Use the **Base** field or the address buttons to jump around; **Follow PC** keeps the window on the
executing instruction.

### Assembler syntax

```asm
; a comment runs to the end of the line

COUNT   EQU  0AH             ; named constant

        ORG  2000H           ; load address

LOOP:   MVI  C, 05H          ; label, mnemonic, operands
        LXI  H, 2050H
        MOV  A, M
        JNZ  LOOP
        HLT
```

**Numbers**

| Form | Example | Note |
|---|---|---|
| Hex | `2050H`, `0FFH` | must start with a digit — write `0FFH`, not `FFH` |
| Hex (C style) | `0x2050` | |
| Binary | `10110010B` | |
| Decimal | `25` | the default if nothing marks it |
| Character | `'A'` | |
| Label arithmetic | `LOOP+2`, `TABLE-1` | |

**Directives**

| Directive | Does |
|---|---|
| `ORG 2000H` | set the load address |
| `DB 05H, 'A'` | place bytes (strings allowed) |
| `DW 2050H` | place a 16-bit word, low byte first |
| `DS 8` | reserve 8 bytes |
| `NAME EQU 0AH` | define a constant |
| `END` | stop assembling |

**Instruction set** — all of it. Data transfer (`MOV`, `MVI`, `LXI`, `LDA`, `STA`, `LHLD`, `SHLD`,
`LDAX`, `STAX`, `XCHG`), arithmetic (`ADD`, `ADC`, `ADI`, `ACI`, `SUB`, `SBB`, `SUI`, `SBI`, `INR`,
`DCR`, `INX`, `DCX`, `DAD`, `DAA`), logical (`ANA`, `ANI`, `ORA`, `ORI`, `XRA`, `XRI`, `CMP`, `CPI`,
`RLC`, `RRC`, `RAL`, `RAR`, `CMA`, `CMC`, `STC`), branching (`JMP` and all eight conditional jumps,
`CALL` and conditional calls, `RET` and conditional returns, `PCHL`, `RST`), and stack / machine
control (`PUSH`, `POP`, `XTHL`, `SPHL`, `IN`, `OUT`, `EI`, `DI`, `RIM`, `SIM`, `NOP`, `HLT`).

Flags follow the 8085A datasheet, including the ones people get wrong: `INR` and `DCR` modify every
flag *except* CY; `DAD` modifies only CY; `MOV`, `MVI`, `LDA`, `STA`, `LXI` and the jumps modify none.

### The four tabs

**Lab assignments** — 15 questions with automatic checking. Pick one, write your program, press
**Run & check my program**. It assembles your code, loads whatever data the question says is already
in memory, runs it, and compares every location the question asks about — memory addresses and
registers alike — showing expected vs found for each. A tick is stored against the ones you get
right. Each question also has a hint, a blank starting template with the right `ORG`, and a model
answer you can drop into the editor.

**Exam problems** — 15 classic problems with *randomised* input data. Same checking, but press
**New input data** to get fresh numbers and run again. This is the one that catches programs that
only work for the numbers you happened to test with.

**Instruction set** — every instruction with its opcode, byte count, T-states, which flags it
modifies, and what it does. The filter box searches all of it.

**Syntax & tips** — the quick reference, plus the mistakes that cost marks.

---

## Part 2 — The Trainer Kit

`trainer-kit.html` simulates the physical board: six-digit LED display, hex keypad, and the monitor
firmware. Use it to make the key sequences automatic before you sit down in the lab.

### The key sequences

These are the four you need. The kit's **Key sequences** tab has them too.

**Key a program in**

```
RESET → EX MEM → 2 0 0 0 → NEXT
```

The display now shows address `2000` and the byte stored there. Type two hex digits for the opcode,
press **NEXT** to store it and move to `2001`, and carry on. Press **RESET** when the last byte is in.

**Read a location back**

```
EX MEM → 2 0 5 2 → NEXT
```

This is how you read your answer after a run. **NEXT** steps forward through memory, **PREV** steps
back. Nothing is written unless you type new digits.

**Run the program**

```
RESET → GO → 2 0 0 0 → EXEC
```

The display blanks while the processor runs and comes back showing `HLT`. If it never comes back,
your program is stuck in a loop — press **RESET**.

**Single-step**

```
GO → 2 0 0 0 → SI    then SI, SI, SI …
```

Each **SI** executes one instruction and shows the next address with the opcode sitting there.

**Examine registers**

```
EX REG → A        (then NEXT to cycle through the rest)
```

The small legend under each hex key is its register name in this mode: `A`–`F` on keys A–F, `H` on 8,
`L` on 9, `SP` on 0, `PC` on 1.

### Keyboard shortcuts

| Key | Does |
|---|---|
| `0`–`9`, `A`–`F` | hex keys |
| `Enter` | NEXT |
| `Backspace` | PREV |
| `Esc` | RESET |
| `M` | EX MEM |
| `R` | EX REG |
| `G` | GO |
| `X` | EXEC |
| `S` | SI |

### Exercises

The **Exercise** dropdown holds 23 programs: 8 standard ones, then the 15 lab assignments. Each gives
you the address/opcode sheet to copy onto the kit, plus the data bytes to key in.

- **Check what I keyed in** compares every byte in memory against the sheet and names the first
  address that is wrong — so you practise fixing it with `EX MEM → address → NEXT` instead of
  starting over.
- **Check the result** verifies the output locations after you run it.
- **Load it for me** skips the typing when you only want to test the program.
- **Guide me** highlights the next key to press and walks you through an entire entry-and-run
  sequence, byte by byte.

### The other tabs

**Peek inside** shows registers, flags and memory while you work. The real kit shows you six digits
and nothing else, so treat this as training wheels and turn away from it before the exam.

**Make a code sheet** takes any assembly you paste in and produces the address/opcode table to key in
— useful when you are handed a program you have not seen before.

> **Key labels differ between kits.** Vinytics and Dyna boards say `EXMEM` / `EXREG`; ALS and ESA
> boards say `SUBST MEM` / `REG`, and the execute key may be marked `FILL`, `EXEC` or `GO` again. The
> *sequence* is identical everywhere: command key, address, confirm. Check the legend printed on your
> kit before you start.

---

## The 15 lab assignments

| # | Question | Origin |
|---|---|---|
| 1.1 | Place 7AH in D and 63H in C, add them into H, subtract 03H, result in E | 8000H |
| 1.2 | Fill 20 bytes from 9250H with natural numbers in increasing order | 8000H |
| 1.3 | AP series, 8 terms, common difference 2, from 9450H | 8000H |
| 1.4 | Store 24H in 80H successive locations from 9500H | 8000H |
| 2.1 | GP series, 8 terms, common ratio 2, from 9000H | 8000H |
| 2.2 | Store 5 bytes from 8220H, copy the block to 9040H in reverse | 8000H |
| 2.3 | 2's complement of the byte at 9100H, result in the accumulator | 8000H |
| 2.4 | Swap the bytes at 9200H and 9210H | 8000H |
| 2.5 | Count the 1s in the byte at 2050H, count to 2051H | 2000H |
| 2.6 | Fibonacci series, 10 terms, from 9300H | 2000H |
| 3.1 | Insert 66H and 77H into the string at 8600H, shifting it down two | 8000H |
| 3.2 | Add six bytes from 8200H into a 16-bit result at 8291H/8292H | 8000H |
| 3.3 | Mark each of eight bytes at 9020H even (01H) or odd (00H), from 8420H | 8000H |
| 4.1 | Multiply 04H by 06H by successive addition, result in D | 8000H |
| 4.2 | Multiply 80H by 81H, 16-bit result at 9290H/9291H | 8000H |

---

## How it works

**Assembler** — two passes. The first walks the source computing the size of each instruction and
recording label addresses; the second emits opcodes and resolves operands. Errors carry the source
line number.

**Emulator** — a fetch/decode/execute loop that decodes by opcode bit pattern rather than a 256-entry
table, so `MOV`, the ALU group, the conditional jumps/calls/returns and the register-pair
instructions are each handled once. Flags are computed the way the hardware does: subtraction runs as
addition of the two's complement, so the borrow and AC flags fall out correctly rather than being
special-cased.

**Grading** — each assignment declares the memory it starts with and the locations that must hold
particular values at the end. Submitting assembles your source into a cleared 64K memory, seeds the
RST vectors, applies the input data, runs to `HLT` with an instruction cap, then compares. Register
checks and memory checks work the same way.

---

