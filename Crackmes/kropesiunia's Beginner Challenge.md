CrackME URL : https://crackmes.one/crackme/6705325e9b533b4c22bd0f9f

**Description**

Your task is to patch the file so that it always returns a successful password verification, regardless of the input, or to discover the correct password. This challenge is designed to help you practice basic reverse engineering skills. Good luck!

<img width="980" height="301" alt="image (3)" src="https://github.com/user-attachments/assets/af8d09e6-8c06-4950-83e7-c2418c0c4b68" />

1st  lets patch it !!!!!!!!!!!!!!!!!!!!!!!

Tool used - Binary Ninja

<img width="959" height="725" alt="image (4)" src="https://github.com/user-attachments/assets/bad1e7d3-e39f-417c-ba58-e9552f03a4a7" />

```
if Je Access Denied 
if JNE then Access Granted
```

**so patch JE to Nop (no operation with occur )**

  
<img width="962" height="628" alt="image (5)" src="https://github.com/user-attachments/assets/996920f0-18ee-484d-a560-74a1eb946d09" />

now lets debug and test this !!!!!!!!!!!!!  (before launching debug save it )



<img width="972" height="397" alt="image (6)" src="https://github.com/user-attachments/assets/dd7217d4-e89d-4a03-a556-df68976a91c1" />
