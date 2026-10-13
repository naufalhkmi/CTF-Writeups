<img width="823" height="187" alt="image" src="https://github.com/user-attachments/assets/46ce93e8-91d3-4bd3-b313-1f72399b678c" /># Breaking RSA - TryHackMe Write-up

- **Platform:** [TryHackMe](https://tryhackme.com/)
- **Room:** Breaking RSA
- **Difficulty:**  Medium
- **Category:** Cryptography
- **Tools Used:** 


Hop in and break poorly implemented RSA using Fermat's factorization algorithm.



---


## A brief overview of RSA 

<details>
<summary>Click here</summary>
<br>

RSA is a public-key cryptosystem. RSA uses two mathematically linked keys:

- A public key that anyone can see — used to encrypt.
- A private key that only you hold — used to decrypt.

  The security of RSA rests on one hard problem: integer factorization. 
  Multiplying two huge primes is easy; un-multiplying the product back into its primes is astronomically hard.


**THREE CORE NUMBERS :**

| Symbol | Name | Public? | What it is | 
| --- | --- |--- | --- |
| n | Modulus | Public | Product of two large primes: n = p × q |
| e | Public exponent | Public | A small constant, almost always 65537 |
| d | Private exponent | Public | The secret that unlocks messages |

Public key = ```(e, n)```
Private key = ```d```

<br>

**CONCEPT**

In RSA CTF challenges, you're given ```n``` (the modulus). Your goal is to factor ```n``` back into its two secret primes ```p``` and ```q```. Once you have ```p``` and ```q```, you can compute the private key ```d```, which lets you decrypt the flag.

1. You are given: n, e, and ciphertext c
2.  You don't know: p, q, d
3.  Your job: factor n → find p and q
4.  Once you have p, q: compute d = e⁻¹ mod lcm(p−1, q−1)
5.  Now decrypt: m = c^d mod n → that's the flag

   
</details>

---



## Recon and exploration
<details>
<summary>Click here</summary>
 <br> 

if you already understand concept of RSA. lets dive into this challenge. first let start with nmap scanning:

<img width="775" height="297" alt="image" src="https://github.com/user-attachments/assets/4be42c72-57e8-4d16-8ea3-e8eb8f180c01" />

there is 2 open port. i go to the port 80 , it doenst show any interesting clue. it just show "A jack of all trades is a master of none but oftentimes better than a master of one" . i look at view page source, it is also doenst show any interesting clue. 

Now, let us find any hidden directory on this server by use gobuster command :

```bash
gobuster dir -u http://10.49.158.234/ -w /usr/share/wordlists/dirb/common.txt
```

<img width="821" height="332" alt="image" src="https://github.com/user-attachments/assets/2c16fea6-0cdb-4e7c-adf8-da99c98d13cf" />

we found one!

i go to the directory and we will found :

<img width="1415" height="397" alt="image" src="https://github.com/user-attachments/assets/09965182-b3b9-4b5f-b0fa-83b225928036" />

look inside log.txt,

<img width="823" height="187" alt="image" src="https://github.com/user-attachments/assets/d314dff8-417a-427c-bb86-88ee9f56c3e5" />

it said that , the ssy keys that they generate use weak RSA. because the p and q value is close. well, this information we already got earlier.
so we can use Fermat's Factorization method.


</details>

---


## Fermat's Factorization Method

<details>
<summary>Click here</summary>
<br>
  
read the file that we have downloaded inside development directory just now (id_rsa.pub).

<img width="1901" height="87" alt="image" src="https://github.com/user-attachments/assets/74d9e756-2422-40ec-8ccb-0da93ee59deb" />

this is the public key ssh format. it is base64 encoded binary structure.

</details>

---


## Privilege Escalation

<details>
<summary>Click here</summary>


</details>


