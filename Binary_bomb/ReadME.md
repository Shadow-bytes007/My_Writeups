# OpenSecurityTraining2 Arch1001: Binary Bomb Lab Write-Up

This write-up explores the **Binary Bomb Lab**, originally developed for Carnegie Mellon University’s architecture course by **Bryant & O'Hallaron**, and later adapted for **Intel x86-64** by **Xeno Kovah** as part of **Architecture 1001: x86-64 Assembly** at **OpenSecurityTraining2**.

The objective of the lab is to **determine the precise input required to prevent the bomb program from exploding**, using reverse engineering and debugging techniques.

---

## Table of Contents

- [Phase 1](#phase-1)
- [Phase 2](#phase-2)
- [Phase 3](#phase-3)
- [Phase 4](#phase-4)
- [Phase 5](#phase-5)
- [Phase 6](#phase-6)
- [Secret Phase](#secret-phase)

---

## Phase 1

The challenge was analyzed using **WinDBG** in normal mode, allowing access to symbol information via `bomb.pdb`.

A breakpoint was set at `main`, and execution was stepped until the first user input prompt appeared. An arbitrary string (`test`) was entered to observe execution flow.

Execution then enters `phase_1`. Disassembling the function using:

```text
uf phase_1
