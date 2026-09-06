# ohSINT - TryHackMe Write-up

- **Platform:** [TryHackMe](https://tryhackme.com/)
- **Room:** ohSINT
- **Difficulty:** Easy
- **Category:** OSINT
- **Tools Used:** 

## Challenge Overview

What information can you possibly get with just one image file?

---


## Extracting MetaData

<details>
<summary>Click here</summary>
<br>

When you download the task file, it just show an ordinary picture :

<img width="1461" height="818" alt="image" src="https://github.com/user-attachments/assets/92e1e3f5-46e5-4ae8-9d86-e94a68ab9f6e" />


but it had hidden details that we need to extract. we can use 

```bash
exiftool
```



<img width="620" height="455" alt="image" src="https://github.com/user-attachments/assets/8e2ad202-89c4-4390-855b-d7dc579cd7f9" />

<br>


there is one clue that we found, which is "copyright". it tells who does this image belong to.

<br>

<img width="632" height="458" alt="image" src="https://github.com/user-attachments/assets/581326c7-5b60-4f0a-abb2-084c2f15c252" />




</details>

---



## Investigating The Name
<details>
<summary>Click here</summary>
 <br> 

Lets google the name. it show three sites: GitHub, X (formerly Twitter), and a WordPress website.

<img width="1300" height="680" alt="image" src="https://github.com/user-attachments/assets/85889af2-78a6-4da7-a151-9723a940d8ab" />


<br>


Take a look at X platform first.

<br>


<img width="576" height="301" alt="image" src="https://github.com/user-attachments/assets/0e2b9af5-6624-424d-a2a7-c3623f6be17f" />


from this info, we can answer 1 question

<br>

> What is this user's avatar of?

<br>

Next, we look at github page andd we found some information.

<br>

<img width="772" height="513" alt="image" src="https://github.com/user-attachments/assets/0e922ed6-ed4a-4966-807c-12486e53e1ac" />

from these information, we can answer 2nd question 4th question, 5th question.

> What city is this person in?

> What is his personal email address?

> What site did you find his email address on?


Back to the page X, we notice that, the author tweet of a BSSID (B4:5D:50:AA:86:41). This is a unique identifier for a wireless access point, kind of like fingerprint for networks. To find the SSID (the name of the Wi-Fi network), we will use [wigle.net](https://wigle.net/)

> What is the SSID of the WAP he connected to ? 



</details>

---


## Track Holiday & Find Password

<details>
<summary>Click here</summary>
<br>

Next, we look at the wordpress sites, we found another clue that can answer question 6th.

<img width="960" height="170" alt="image" src="https://github.com/user-attachments/assets/9cda9af5-b059-49f9-b112-757605789f57" />

> Where has he gone on holiday?


And for the last question, there is no other clue given in the page. so lets inspect this page source.

And we found some strange strings that might be an answer for the last question.

<img width="1327" height="576" alt="image" src="https://github.com/user-attachments/assets/1e194637-b37e-4bf9-9471-9d985c932ea7" />

> What is the person's password?


</details>

---


