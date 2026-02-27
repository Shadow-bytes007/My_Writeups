# 🧩 Crackme Write-Up

🔗 Crackme Link: https://crackmes.one/crackme/698a6214fb46458f1ef6cef5  
🛠 Tool Used: Ghidra (Static Analysis)

---

## 📌 Overview

For this challenge, I performed **static analysis** using Ghidra.  
The goal was to understand the password validation logic and determine the correct input without executing the binary.

```
Straight lets jump into main function
there we can see a function called Passwordcheck
```
<img width="857" height="188" alt="image" src="https://github.com/user-attachments/assets/10345571-c846-4b74-899e-2767b29eadd4" />




``` go to this fuction and check the decompile tab ```


<img width="1503" height="724" alt="image (1)" src="https://github.com/user-attachments/assets/1e00c91d-239b-46bb-99a3-59b475d828b0" />

So effectively, the program implements a whitelist of allowed ASCII ranges.

## ✅ Allowed Character Ranges

From reversing the nested conditions, the allowed ranges are:

| Range | Characters |
|--------|------------|
| `'0'–'4'` | 0 1 2 3 4 |
| `'H'–'N'` | H I J K L M N |
| `'t'–'y'` | t u v w x y |
| `'a'–'f'` | a b c d e f |
| `'!'–'&'` | ! " # $ % & |
| `';'–'?'` | ; < = > ? |
| `'j'–'m'` | j k l m |
| `'z'–'}'` | z { \| } |
| `'o'–'s'` | o p q r s |
| `'[' < c < 'a'` | \ ] ^ _ ` |

Each character in the password must belong to **one of these ranges**.
<img width="979" height="507" alt="image (2)" src="https://github.com/user-attachments/assets/7361da29-15ec-4e76-9ef5-80c7941fbdda" />

🏁 Conclusion

This crackme does not use hashing, encryption, or anti-debugging techniques.
Instead, it relies on:

Nested ASCII range validation

Logical obfuscation through chained conditions

By simplifying the nested conditions into readable range checks, we can determine valid inputs using static analysis alone.
