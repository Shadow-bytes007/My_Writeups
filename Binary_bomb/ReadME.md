<img width="771" height="122" alt="phase3 1" src="https://github.com/user-attachments/assets/637ac114-380c-4853-8c27-24f59e14c913" /># Binary Bomb Lab Write-Up

This write-up explores the **Binary Bomb Lab**, originally developed for Carnegie Mellon University’s architecture course by **Bryant & O'Hallaron**, and later adapted for **Intel x86-64** by **Xeno Kovah** 

The objective of the lab is  **to prevent the bomb program from exploding**, using reverse engineering and debugging techniques.

---

## Table of Contents

- [Phase 1](#phase-1)
- [Phase 2](#phase-2)
- [Phase 3](#phase-3)
- [Phase 4](#phase-4)
- [Phase 5](#phase-5)
- [Phase 6](#phase-6)


---

## Phase 1

The challenge was analyzed using **WinDBG** in normal mode, allowing access to symbol information via `bomb.pdb`.

A breakpoint was set at `main`, and execution was stepped until the first user input prompt appeared. An arbitrary string (`test`) was entered to observe execution flow.

Execution then enters `phase_1`. Disassembling the function using:


uf phase_1


revealed two critical calls:

call bomb!ILT+810(strings_not_equal),
call bomb!ILT+945(explode_bomb)


The behavior mirrors `strcmp`: if the input string does not match the expected string, execution proceeds to `explode_bomb`.


![phase1](https://github.com/user-attachments/assets/d6018e0b-093f-4a11-a9be-76a184c0ef78)

Using the Microsoft x64 calling convention:
- **RCX** holds the first argument (my input)
- **RDX** holds the second argument (comparing str)

The compared string was identified using:

**da < address >**

The required input string is:



**I am just a renegade hockey mom.**


Providing this exact input successfully defuses **Phase 1**.

---

## Phase 2

Entering `phase_2` with an arbitrary input (e.g., `123`) results in termination inside `read_six_numbers`.
<img width="779" height="350" alt="phase2 1" src="https://github.com/user-attachments/assets/19571632-f592-4fd3-a909-cc382ee1228a" />

Disassembly shows:



call bomb!ILT+705(sscanf)
mov dword ptr [rbp+4], eax
cmp dword ptr [rbp+4], 6
jge ...


`sscanf` returns the number of successfully parsed fields. Since six integers are expected, the input must contain **six valid numbers**.

After satisfying this condition, the function enforces a sequence where each number must be double the previous one, implemented via bit-shift operations.

### Required Pattern

| Decimal | Hex |
|-------:|----:|
| 1 | 0x1 |
| 2 | 0x2 |
| 4 | 0x4 |
| 8 | 0x8 |
| 16 | 0x10 |
| 32 | 0x20 |


<img width="962" height="395" alt="phase2 3" src="https://github.com/user-attachments/assets/9ff573a3-ddf1-4045-a559-9559b609ae1a" />
<img width="355" height="107" alt="phase2 4" src="https://github.com/user-attachments/assets/5860a874-61aa-4edf-8dd2-17d507fad901" />

**Note:** In 5th loop , at cmp rip , rcx = 10 which is a 10h (10h = 16dec)

**Phase 2 Input**



**1 2 4 8 16 32**


---

## Phase 3

This phase requires **two inputs**.

<img width="771" height="122" alt="phase3 1" src="https://github.com/user-attachments/assets/c6ce0c43-ca40-4213-aff7-930d361fab3c" />


Key constraints identified via conditional jumps:

- First input [rbp+134h] must be **≤ 7** <img width="745" height="96" alt="phase3 2" src="https://github.com/user-attachments/assets/5a7178b7-a7df-42ba-a00e-e1e4bf7dfc22" />

- First input must also be **≤ 5** to bypass an early explosion <img width="732" height="73" alt="phase3 3" src="https://github.com/user-attachments/assets/5f93e99b-cb5b-4fa7-b774-62defc56e62b" />

- Second input [rbp+24h]must equal a computed value derived from a jump table
dd rbp+44h L1 = ffffffe6  
  ffffffe6 = 4294967270
  
Input :

1 4294967270

Here, `4294967270` (decimal) corresponds to `0xffffffe6`, matching the computed value stored during execution.

(or)

2 562


Here, `562` (decimal) corresponds to `0x232`, matching the computed value stored during execution.

---

## Phase 4

`sscanf` again requires **two inputs**.

<img width="749" height="360" alt="phase4 1" src="https://github.com/user-attachments/assets/3e42fac8-c69a-446b-9fbf-c0f32aef1187" />


Constraints:
- First input ≤ `0xE` (14)

<img width="730" height="142" alt="phase4 2" src="https://github.com/user-attachments/assets/e516d6f5-f480-48cb-b120-7a3a616aa83b" />

- Second input becomes  `10` because of mov 

<img width="695" height="94" alt="phase4 3" src="https://github.com/user-attachments/assets/8e3328d0-2b32-4b67-85cc-a68f70e2ffd7" />

- The return value of `func4` must equal `0xA`

<img width="609" height="64" alt="phase4 4" src="https://github.com/user-attachments/assets/559db025-322d-4bf1-a1ef-58aea9efb241" />
here at executing func4 

```In (rbp+4) dd rbp+4 L1 = 7 , so at addr 00007ff6`baf41fbe after executing instruction add rax = a ```

here after func4
```
00007ff6`baf423e9 894544            mov      dword ptr [rbp+44h],eax  

00007ff6`baf423ef 394544            cmp         dword ptr [rbp+44h],eax   == 10

and

00007ff6`baf423f7 394524          cmp     dword ptr [rbp+24h],eax  == 10

```

### func4 Analysis

`func4` is a recursive function resembling a binary search. The only value that produces the correct return value is:



3


**Phase 4 Input**



3 10


---

## Phase 5

Two inputs are required.

Key observations:
- First input is masked using `AND 0xF`
- A loop executes exactly **15 iterations**
- A lookup table drives the value transformation

After analyzing all possible paths, the only valid input is:

- First input: `5`
- Accumulated sum: `0x73` → `115` (decimal)

**Phase 5 Input**



5 115


---

## Phase 6

This phase requires **six unique integers**.

Constraints:
- Values must be in the range `1–6`
- No duplicates are allowed
- Inputs reorder a linked list
- Resulting node values must be in **descending order**

Node values:

| Input | Value |
|------:|------:|
| 5 | 0x3A7 |
| 4 | 0x393 |
| 3 | 0x215 |
| 1 | 0x212 |
| 6 | 0x200 |
| 2 | 0x1C2 |

Sorted descending:



5 4 3 1 6 2


**Phase 6 Input**



5 4 3 1 6 2


---


## Final Result

🎉 **Bomb Successfully Defused (Including Secret Phase)**

---

## Credits

- **Bryant & O’Hallaron**, Carnegie Mellon University  
- **Xeno Kovah**, OpenSecurityTraining2  

This lab is an excellent exercise in **assembly analysis**, **debugger proficiency**, and **reverse engineering**
```text
