# Binary Bomb Lab Write-Up

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

By checking sscanf we geyt to know that **Two inputs are required.**
First Input -> rbp+64h
Second Input -> rbp+84h

Key observations:
- First input is masked using `AND 0xF`
```
mov     eax,dword ptr [rbp+64h]
and     eax,0Fh
```
`and     eax,0Fh` from this we get to know that our 1st input cannot exceed decimal 15 (0x0fh)

`0x0` is placed into `rbp+4` and `rbp+24`

```
cmp     dword ptr [rbp+64h],0Fh
je      bomb!phase_5+0xc4 (00007ff7`fb952524)
```
if we take this jump , it leads to another jump which leads to explode bomb by comparing  two immediates 0x0 and 0x0F
`NOTE : [rbp+64] should not be 0fh (15 decimal)`
So skip the 1st `je`

This takes us to a series of instructions that puts us in a loop and increments rbp+4 by 1 so that it can  be equal to 0x0F by incremeting, and we can jump to 00007ff7fb952535. However, there is some pointer arithmetic that modifies the value of rbp+64h(out 1st input):

```
movsxd  rax,dword ptr [rbp+64h]
lea     rcx,[bomb!n1+0x20 (00007ff7`fb95f1d0)]
mov     eax,dword ptr [rcx+rax*4]
mov     dword ptr [rbp+64h],eax
```

if our `[rbp+64]` value is modified to `0xf` before the `rbp+4` valure reaches `0xf` then the loop breaks and pushes to `explode bomb`.

`[rcx+rax*4]`
with command `dd 00007ff7fb95f1d0+X*4 L1` we can view the memory of those address 
`NOTE: RAX = OUR 1ST INPUT (rbp+64h) `
<img width="380" height="488" alt="image" src="https://github.com/user-attachments/assets/02fb2bde-6a80-4f4a-b877-fbe7804a6030" />

BY THIS WE CAN CHECK THAT , if `RAX` is more than 5 ,it leads to `0xf` before the `rbp+4` reaches `0xf`  

so if our 1st input is 5 , `RAX` takes 15 rotaion to get memory as 15 `0xf`, where `rbp+4` increments by 1 and it needs 15 rotation for `0xf` 

``
Input of 5 = C --> 3 --> 7 --> B --> D --> 9 --> 4 --> 8 --> 0 --> A --> 1 --> 2 --> E --> 6 --> F
``

we can see some more arithmetic performed on the immediate `0x0` in `rbp+24h` within the loop:

```
mov     ecx,dword ptr [rbp+24h]
add     ecx,eax
mov     eax,ecx
mov     dword ptr [rbp+24h],eax
```

`NOTE : rbp+24h value will be `cmp`  to our 2nd input `rbp+84h` ,and needs to be equal otherwise it jumps to an explode_bomb ` 
we know that the increment is  done on `RAX` , and those increments the `rbp+24h` by the value in `00007ff7fb95f1d0+X*4` for each loop

Therefore, if our input = `5` ( C + 3 + 7 + B + D + 9 + 4 + 8 + 0 + A + 1 + 2 + E + 6 + F )
by the end of the loop, `rbp+24h` will have a value of `0x73`  because of this instruction
``add     ecx,eax``

This means that our second input needs to be the decimal equivalent of hex 73, so it is decimal 115



After analyzing all possible paths, the only valid input is:

- First input: `5`
- Accumulated sum: `0x73` → `115` (decimal)

**Phase 5 Input**



5 115


---

## Phase 6

This phase requires **six unique integers**.
Firstly, there is a call to `read_six_numbers` , so it requires an  input six digits

```
cmp     dword ptr [rbp+rax*4+48h],1
jl      bomb!phase_6+0xa1 (00007ff6`47f62651)

```
this means the value must be in range `1–6`
<img width="831" height="54" alt="phase6 1" src="https://github.com/user-attachments/assets/dd35d143-c8c3-4ac0-80dc-047b763c4232" />
the 1st input should be less than 6 , to skip `explode bomb`

Next `eax` is incremented then `mov` to `[rbp+0E4h]` and it compares to 6 , jge leads to `explode bomb`

```
movsxd  rax,dword ptr [rbp+0C4h]
movsxd  rcx,dword ptr [rbp+0E4h]
mov     ecx,dword ptr [rbp+rcx*4+48h]
cmp     dword ptr [rbp+rax*4+48h],ecx
jne     bomb!phase_6+0xea (00007ff7fb95269a)
```
this comapres if input 1 == input2 
If we skip this jump it calls explode_bomb, so our first input and second input can’t be equal.

we enter a loop which increments `rbp+0E4h` by `0x`1 each loop. It needs to loop 5 times in order for rbp+0E4h’s value to be equal to `0x6`, so we can take the `JGE` instruction to `00007ff7fb95269c` and break out of the loop. This will require our first input to not be equal to any other input. This is because in the CMP/JNE instructions it uses `rbp+0E4h` which is incremented by 0x1 each time to access our inputs.

the next jump takes to the start which increments rbp+0C4h and this loop 6 times in order for the JGE instruction to jump to `00007ff7fb95269e` and break us out of the loop.

Next, we come to some more CMP instructions:
```
cmp     dword ptr [rbp+0C4h],6
jge     bomb!phase_6+0x166 (00007ff7fb952716)
```

[rbp+0C4h] (it is reset reset to 0x0) >= to 6  


rbp+8 (00007ff7fb95f050) is moved into RAX, and RAX is moved into rbp+28h.

```
movsxd  rax,dword ptr [rbp+0C4h]
mov     eax,dword ptr [rbp+rax*4+48h]
cmp     dword ptr [rbp+0E4h],eax
jge     bomb!phase_6+0x154 (00007ff7fb952704)
```

[rbp+0E4h] (reset to 0x1) >= first input

If it is not >= 0x1, we move `rbp+28h (00007ff7fb95f050`) into RAX, increment it by eight bytes to access `00007ff7fb95f040’s` memory, and store is back in `rbp+28h`.

It will then enter a loop which will increment rbp+0E4h (0x1) by 0x1 UNTIL rbp+0E4h is >= to our first input.

ONCE rbp+0E4h is >= to our first input, it jumps to a series of instructions which saves the current `rbp+28h` value into memory at `rbp+rax*8+78h`:
```
movsxd  rax,dword ptr [rbp+0C4h]
rcx,qword ptr [rbp+28h]
mov     qword ptr [rbp+rax*8+78h],rcx
```

It is important to note that during the **first loop iteration**, if `rbp+0E4h` (initialized to `0x1`) is **greater than or equal to** our first input (for example, `0x1`), the program immediately saves the address `00007ff7fb95f050` to the memory location `rbp + rax*8 + 78h`.

If, however, the input is `6`, execution cycles through memory and ultimately saves the **lowest address**, `00007ff7fb95f000`, to `rbp + rax*8 + 78h`.

Based on this behavior, we can assume the following input-to-address mapping:

- `00007ff7fb95f050` corresponds to input `1`
- `00007ff7fb95f040` corresponds to input `2`
- `00007ff7fb95f030` corresponds to input `3`
- `00007ff7fb95f020` corresponds to input `4`
- `00007ff7fb95f010` corresponds to input `5`
- `00007ff7fb95f000` corresponds to input `6`

```
Constraints:
- Values must be in the range `1–6`
- No duplicates are allowed
- Inputs reorder a linked list
- Resulting node values must be in **descending order**
```

 we need rbp+0C4h (reset to 0x1) to be >= to 5 so we can jump to 00007ff7fb9527d1 and exit:
```
cmp     dword ptr [rbp+0C4h],5
jge     bomb!phase_6+0x221 (00007ff7`fb9527d1)
```

However, in order to do so, we need to take the jump to 00007ff7fb9527c3 on EVERY loop, so rbp+0C4h can reach 5:
```
cmp     dword ptr [rcx],eax
jge     bomb!phase_6+0x213 (00007ff7fb9527c3)
Is “[rcx] >= [eax]”? In addition, if we don’t take this jump, it will call explode_bomb.
```

If we look at the instructions just before the jump to `00007ff7fb9527c3`, we can see that the values used in the `cmp` instruction come from the addresses stored at `rbp+78h` and higher. The value loaded into `RCX` for the comparison corresponds to the address saved at `rbp+78h`, while the value loaded into `EAX` corresponds to the address saved at `rbp+78h + 8`. On each iteration of the loop, both of these addresses move forward by **eight bytes**.

As mentioned earlier, we control which addresses are saved at `rbp+78h` and above. Because of this, to ensure the conditional jump is always taken, the value at the address stored in `rbp+78h` must be **greater than or equal to** the value at the address stored in `rbp+78h + 8`. Simply put, each input we choose must map to a value that is **greater than or equal to the value produced by the next input**.

For example, when examining the values stored at each address, we can see that `0x3A7` is the **largest value**.




0x3a7 corresponds to address 7ff7fb95f010, and as previously mentioned, 7ff7fb95f010 corresponds to an input of 5. So, the next highest value is 0x393 at address 7ff7fb95f020, which corresponds to an input of 4, and so on:


<img width="294" height="225" alt="image" src="https://github.com/user-attachments/assets/a7ccb5c4-2254-4e5e-beb5-164717fd45e0" />


Node values:

| Input | Value |
|------:|------:|
| 5 | 0x3A7 |
| 4 | 0x393 |
| 3 | 0x215 |
| 1 | 0x212 |
| 6 | 0x200 |
| 2 | 0x1C2 |



**Phase 6 Input**



5 4 3 1 6 2


---


## Final Result

🎉 **Bomb Successfully Defused**

---

## Credits

- **Bryant & O’Hallaron**, Carnegie Mellon University  
- **Xeno Kovah**, OpenSecurityTraining2  

This lab is an excellent exercise in **assembly analysis**, **debugger proficiency**, and **reverse engineering**
```text
