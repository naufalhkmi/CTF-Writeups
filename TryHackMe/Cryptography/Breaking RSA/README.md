# Breaking RSA - TryHackMe Write-up

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

to get n (modulus), natural approach is to use the pycryptodome library. 

> make sure you have download pycryptodome library in you terminal so that you can use code below
> ```bash
> sudo apt install python3-pycryptodome
> ```

<br>


```bash
from Cryptodome.PublicKey import RSA

key = RSA.importKey(open("id_rsa.pub").read())
print("e =", key.e)
print("n =", key.n)

</details>
```

<img width="1896" height="283" alt="image" src="https://github.com/user-attachments/assets/c555f5e6-3e30-4574-91d8-5c53843dd467" />

nicee. we got the n.

okay this part is a little bit complex but its funn.

> Fermat's factorization method relies on writing n as a difference of two squares:
> 
> ```
> n = a² − b² = (a + b)(a − b)
> ```
> 
> By setting:
> 
> ```
> a = (p + q) / 2
> b = (q − p) / 2
> ```
> 
> we get:
> 
> ```
> p = a + b
> q = a − b
> ```
> 
> <br>
> 
> If p and q are very close, then b is small and a ≈ √n. So we start at a = ⌈√n⌉ and increment until a² − n becomes a perfect square. Once it does, the two factors are a + b and a − b.

Since we already installed pycryptodome via apt, we can implement this python to find p and q.

> PyCryptodome's RSA.import_key() expects the key in PEM form. Since the challenge supplied an OpenSSH public key (ssh-rsa AAAAB3...), we first convert it:
> ```bash
> ssh-keygen -f id_rsa.pub -e -m PKCS8 > id_rsa_pub.pem
> ```

<br>

```bash
from Cryptodome.PublicKey import RSA
from Cryptodome.Util.number import isPrime, long_to_bytes, bytes_to_long
from math import isqrt, lcm

# ---------- Step 1: Load the SSH public key ----------
with open("id_rsa_pub.pem", "r") as f:
    pub_key = RSA.import_key(f.read())

n = pub_key.n
e = pub_key.e
print("[+] e =", e)
print("[+] n =", n)
print("[+] bit length of n =", n.bit_length())

# ---------- Step 2: Fermat factorization ----------
def fermat_factor(n):
    a = isqrt(n)
    if a * a < n:
        a += 1

    iterations = 0
    while True:
        b2 = a * a - n
        b = isqrt(b2)
        if b * b == b2:
            return (a + b, a - b, iterations)
        a += 1
        iterations += 1

p, q, iters = fermat_factor(n)
print("[+] p =", p)
print("[+] q =", q)
print("[+] iterations =", iters)
print("[+] p * q == n ?", p * q == n)
print("[+] |p - q| =", abs(p - q))
```

<img width="1901" height="308" alt="image" src="https://github.com/user-attachments/assets/3d8de6e2-3643-4616-8143-57b1309e0082" />

we got the prime p and q. Once p and q are known, the RSA private key is no longer private.


---


## Find the private key (d)

<details>
<summary>Click here</summary>


pycryptodome provides inverse() in Cryptodome.Util.number, so we don't need to write our own:

```bash
from Cryptodome.Util.number import inverse
from math import lcm

lam = lcm(p - 1, q - 1)
d = inverse(e, lam)

print("[+] lambda =", lam)
print("[+] d =", d)
print("[+] (e * d) mod lambda =", (e * d) % lam)   # must be 1
```

>The last line must print 1. That confirms d is the correct


after we got all five components ( n, e, d, p, q ), we can rebuilding the private key with PyCryptodome :

```bash
key = RSA.construct((n, e, d, p, q))

with open("id_rsa", "wb") as f:
    f.write(key.export_key("PEM"))

print("[+] id_rsa saved.")
```

<img width="410" height="152" alt="image" src="https://github.com/user-attachments/assets/80f604a9-30b2-4ded-856b-dc6a71e5ba2f" />

if you remmeber, there is 2 open port which is 80 and 22(ssh). 

we have to add the correct permissions to the key to log on the machine as root. In the home directory of root, we find the flag.

<img width="987" height="617" alt="image" src="https://github.com/user-attachments/assets/16e6805f-1675-4704-a976-60de3c7c5f65" />


</details>


