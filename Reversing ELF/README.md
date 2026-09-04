<img width="668" height="595" alt="image" src="https://github.com/user-attachments/assets/6dc5dc32-60b2-4bda-b508-d0c1584e6fb3" /># Reversing ELF - TryHackMe Write-up

- **Platform:** [TryHackMe](https://tryhackme.com/)
- **Room:** Reversing ELF
- **Difficulty:** Easy
- **Category:** Reverse Engineer (RE)
- **Tools Used:** Kali Linux

## Challenge Overview

Room for beginner Reverse Engineering CTF players

---


## Crack Me 1

<details>
<summary>Click here</summary>
<br>

> Let's start with a basic warmup, can you run the binary?

<br>

First, we need to downlaod the task file and then we need to know what kind of file it is :

```bash
file crackme1
```

<img width="1335" height="60" alt="image" src="https://github.com/user-attachments/assets/4026c269-7621-472b-813b-08abd4edeb2d" />

it is an ELF 64 bit. let execute and run this file use this command 

```bash
chmod +x crackme1
```

and then

```bash
./crackme1
```

<img width="382" height="92" alt="image" src="https://github.com/user-attachments/assets/0c9f8b15-5e6a-4497-aa85-94e3fbaec999" />



</details>

---



## Crack Me 2

<details>
<summary>Click here</summary>
<br>

> Find the super-secret password! and use it to obtain the flag

<br>

> What is the super secret password ?

<br>

<img width="401" height="47" alt="image" src="https://github.com/user-attachments/assets/84538144-9986-41d6-bbee-ada3b72cde2c" />

We saw that, this file need a password to get the flag. We can simply using the strings command

```bash
strings crackme2
```

<img width="583" height="437" alt="image" src="https://github.com/user-attachments/assets/515f433a-9fbf-4b74-97d9-d46dcc1bf089" />

<br>

We found the password !!

now let try use the password  and run the file.

<img width="462" height="66" alt="image" src="https://github.com/user-attachments/assets/54ca353f-2cac-485e-8c71-1e73fa9643a8" />


</details>

---

## Crack Me 3

<details>
<summary>Click here</summary>
<br>

> Use basic reverse engineering skills to obtain the flag


<img width="415" height="41" alt="image" src="https://github.com/user-attachments/assets/49d5fa92-2583-4304-a92d-df7395f1c034" />


seems like this file also need a password. let use strings command

```bash
strings crackme3
```

<img width="872" height="287" alt="image" src="https://github.com/user-attachments/assets/f3fe2249-91ce-4663-b1b1-8a953c4d881a" />


we found this base 64 encoded strings. we can simply use [cyberchef](https://gchq.github.io/CyberChef/) to decoded this.


<img width="1507" height="501" alt="image" src="https://github.com/user-attachments/assets/e9865690-3d2f-474e-80b2-db410e02dd23" />

</details>

---


## Crack Me 4

<details>
<summary>Click here</summary>
<br>

> Analyze and find the password for the binary?


<img width="447" height="57" alt="image" src="https://github.com/user-attachments/assets/4e246e04-9b03-4347-8787-e3af1c0a7fce" />


this time wee need to use ```strcmp``` to find the password because from that hint we know that this binary file use ```strcmp``` fucntion.

- strcmp stands for String Compare. It is a standard C library function used to check whether two strings of text are identical.

Let use gdb debugger tu debug this binary

```bash
gdb crackme4
```

<img width="658" height="297" alt="image" src="https://github.com/user-attachments/assets/124e8beb-9dfa-4162-8090-f6a4b3deb2a1" />

Next, list all the function inside this binary file

```bash
info functions
```

<img width="496" height="350" alt="image" src="https://github.com/user-attachments/assets/fd0901db-0c45-455f-9596-72ab5b9b3735" />

there are several function name that use to compare the password which is ```main```, ```get_pwd```, and ```compare_pwd```. but i decided to take ```strcmp@plt function```. We can assume that the password entered is compared to the correct password via ```strcmp()```.


Using gdb, let set a breakpoint at the memory address of this function.

```bash
b *0x0000000000400520
```

<img width="531" height="40" alt="image" src="https://github.com/user-attachments/assets/077890fe-1991-4a0a-8df9-80feb9d8e6f9" />


we do this because sets an intentional stopping trap right at the start of strcmp. 

and then we run dummy input

```bash
run test
```

<img width="680" height="92" alt="image" src="https://github.com/user-attachments/assets/6c16c59b-8441-4be8-86df-befda18b2a30" />

now let see the current state of register (Inspect the CPU storage slots)

```bash
info register
```

<img width="630" height="435" alt="image" src="https://github.com/user-attachments/assets/5141ee96-cd60-4a36-b723-a47c4dfc01a3" />

usually, whenever you pause execution at strcmp on a 64-bit Linux program, 

- $rdi holds input #1.
- $rsi holds input #2.

One register will hold your dummy test word, and the other will hold the target password.

now let examine string ```(x/s)``` those 2 register


<img width="355" height="70" alt="image" src="https://github.com/user-attachments/assets/e8ee6fd9-4dad-45b8-8372-30527b961695" />



we found the the password ! ( the flag)


</details>

---


## Crack Me 5

<details>
<summary>Click here</summary>
<br>

> What will be the input of the file to get output Good game ?

<img width="273" height="78" alt="image" src="https://github.com/user-attachments/assets/cc655bf5-0d5e-457f-ae72-ef6bfa3ee610" />

let use strings command to dig deeper.

```bash
strings crackme5
```


seems like it doesnt show any clue. Instead of guessing, I used GDB to pause the program right when it checks the input and read the answer directly from memory. ( this method same with crackme4 method)


we need to find the comparison function first.

```bash
info function
```

<img width="457" height="342" alt="image" src="https://github.com/user-attachments/assets/7f8bbb29-cedd-4399-b7ce-6a4c3baa1511" />


there are several fucntion that do the comparison check. let try with ```strncmp@plt```.

set the breakpoint and then run the dummy test


<img width="661" height="157" alt="image" src="https://github.com/user-attachments/assets/309d8f50-578b-4467-8d01-1b167ac1e270" />

Because this is a 64-bit program, the inputs being compared are sitting in the CPU registers ```($rdi and $rsi)```. Use ```x/s``` (examine string) to print them

<img width="485" height="72" alt="image" src="https://github.com/user-attachments/assets/2edc9650-d2fd-48b6-babd-0c494bfc35e3" />


we found the input !


</details>

---


## Crack Me 6

<details>
<summary>Click here</summary>
<br>

> Analyze the binary for the easy password

<img width="323" height="60" alt="image" src="https://github.com/user-attachments/assets/669c8ddc-8355-411f-a0b2-06986188e458" />


in this challenge. we cannot use method ```strcmp``` because the file uses a custom check. therefore let analyze this file use **ghidra**

import the file and analyze it

after you import, look at the left panel ( symbol tree ) and then find ```main``` fucntion. Every C program starts here. we need to check how input enters the program

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/e8e06f43-339f-404a-b676-2d03a4b3d610" />

at the right side is decompiler panel. it translate into readable code. 

we can seee there is fucntion ```compare_pwd```. double click it. it will redirect to that function.

<img width="677" height="377" alt="image" src="https://github.com/user-attachments/assets/04197042-8d2d-4b46-a665-f28a5774735b" />


now, the code says if ```iVar1 = my_secure_test``` , the program will show ```password OK```. otherwise it will print ```password not OK```. Let double click function ```my secure test```

<img width="575" height="555" alt="image" src="https://github.com/user-attachments/assets/4ad81d3a-1240-4d53-b531-70a9992211c1" />

Inside the decompiler window, you will see a series of simple if checks comparing your input array against specific characters or numbers

Arrange the character in order and that is your password (flag).

<img width="330" height="36" alt="image" src="https://github.com/user-attachments/assets/2d97bfd0-b035-4a17-946d-35c738e1918d" />



</details>

---


## Crack Me 7

<details>
<summary>Click here</summary>
<br>

>   Analyze the binary to get the flag

<img width="332" height="140" alt="image" src="https://github.com/user-attachments/assets/05128d16-85aa-4139-a9e9-7df4f7bce108" />

<br>

let use ghidra to analyze the code.


let start with the ```main``` fucntion to see how this program start.

<img width="1917" height="598" alt="image" src="https://github.com/user-attachments/assets/6fadbd4d-1c48-442c-9959-440e67dfc319" />


look at the right side (decompiler panel), and read the code. we can see that

<img width="670" height="600" alt="image" src="https://github.com/user-attachments/assets/8b6c419e-e622-4553-a605-6c6654951faa" />


if we input 0x7a69 , it will call the ```giveFlag()``` fucntion. let us try

<img width="255" height="152" alt="image" src="https://github.com/user-attachments/assets/5cbfdedd-5ac1-44d7-b1cb-f5e6f27427a9" />

seems like i forgot to change the hex decimal to decimal number. since the menu only read the input in interger.

so 0x7a69 = 31337


<img width="368" height="170" alt="image" src="https://github.com/user-attachments/assets/1b1f47b0-c84d-46d4-8896-bff633c7a31c" />




</details>

---


## Crack Me 8

<details>
<summary>Click here</summary>
<br>

> Analyze the binary and obtain the flag 


<img width="343" height="51" alt="image" src="https://github.com/user-attachments/assets/93bfc8f6-4ea0-4e26-8461-5fa3bc2e9d4c" />


lets open ghidra and analyze the code.

<img width="1897" height="652" alt="image" src="https://github.com/user-attachments/assets/43ea9873-065c-4dbe-903b-516772b0a297" />

and lets look at the decompiler panel

<img width="668" height="595" alt="image" src="https://github.com/user-attachments/assets/98909af0-4838-4396-9478-d6d98765c48c" />

we can see that, if the input is ```-0x35010ff3 ``` it will call function giveFlag(). 

but before the program run the comparison, the input will pass at ```atoi()``` function.

atoi stand for 

- a stands for ASCII (text characters / string).
- to stands for to (convert to).
- i stands for integer (a whole number).
- Combined: a-to-i $\rightarrow$ Convert ASCII string to Integer.

therefore, if i change ```-0x35010ff3``` into integer (decimal) which is ```-889262067```

<img width="547" height="65" alt="image" src="https://github.com/user-attachments/assets/d9ec1bf6-90d2-4224-a81c-f4ccf003d16b" />


it will show the flag.
</details>

---


