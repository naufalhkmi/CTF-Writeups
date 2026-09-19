# Take Over - TryHackMe Write-up

- **Platform:** [TryHackMe](https://tryhackme.com/)
- **Room:** Take Over
- **Difficulty:** Easy 
- **Category:** 
- **Tools Used:** 

## Challenge Overview

This challenge revolves around subdomain enumeration.

> Hello there,

> I am the CEO and one of the co-founders of futurevera.thm. In Futurevera, we believe that the future is in space. We do a lot of space research and write blogs about it. We used to help students with space questions, but we are rebuilding our support.

> Recently blackhat hackers approached us saying they could takeover and are asking us for a big ransom. Please help us to find what they can takeover.

Our website is located at https://futurevera.thm(opens in new tab)

**Hint: Don't forget to add the MACHINE_IP in /etc/hosts for futurevera.thm ; )**

---


## Recon

<details>
<summary>Click here</summary>
<br>

The description state that this challenge revolve subdomain enumaration.

But first , let us start with nmap scanning.

```bash
nmap -Pn -sC -sV -T4 <IP ADDRESS.
```

<img width="946" height="441" alt="image" src="https://github.com/user-attachments/assets/b3cbe04f-0217-44ee-adf7-9bea7baddbbb" />

there is 3 ports open

- port 22/ssh
- port 80/http
- port 443/https : a secure web server

However, there doesn’t seem to be any DNS service running on the machine. Since .thm is not a real tld (top-level domain), there also won’t be any information on public DNS servers. This means that in order to find subdomains of this website, we are probably going to need to use virtual hosts enumeration.



</details>

---



## Subdomain Enumaration
<details>
<summary>Click here</summary>
 <br> 

To trick the server into showing us hidden websites, we used a tool called ```ffuf```

```bash
ffuf -u http://futurevera.thm/ -H "Host: FUZZ.futurevera.thm" -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt
```

but this syntax will show a non stop list because of ```http``` this server blindly accepts almost every random guess, which floods your screen with a non-stop list of fake results.

So instead use that syntax, we should try

```bash
ffuf -u https://futurevera.thm/ -H "Host: FUZZ.futurevera.thm" -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -mc all -fs 4605
```

> - Tells ffuf to show every response code so nothing gets hidden automatically.
> - Hides any result matching the fake error page size

<img width="1287" height="450" alt="image" src="https://github.com/user-attachments/assets/9f57698e-3152-40d8-9dc0-d58cd1d54a3b" />

we found 2 more subdomain

- support
- blog


</details>

---


## Digging Deeper

<details>
<summary>Click here</summary>
<br>
  
Before we dig deeper, add the new subdomain into our etc/hosts first.

after add, let us visit the new subdomain. 

<img width="1911" height="646" alt="image" src="https://github.com/user-attachments/assets/b4c0bcea-e428-46bf-8323-5854713e9ded" />

the main page looked ordinary. However, websites secured with HTTPS use SSL Certificates (like an ID card) that can reveal extra secrets.


By checking the certificate details (specifically the Subject Alternative Names), we found a hidden nested subdomain that wasn't in our wordlist:

<img width="530" height="76" alt="image" src="https://github.com/user-attachments/assets/511d840d-2d3d-48ea-96a5-b373c449c5db" />

Let’s also make this domain point to the IP of the target machine and visit it. 

<img width="1901" height="687" alt="image" src="https://github.com/user-attachments/assets/c71184b4-a575-44a9-ad16-fb8ef5673011" />


found it !




