# Fools Mate - TryHackMe Write-up

- **Platform:** [TryHackMe](https://tryhackme.com/)
- **Room:** Fools Mate
- **Difficulty:** Easy
- **Category:** Web Exploitation
- **Tools Used:** Kali Linux, Burp Suite Community Edition

---

> **Note on Network Access:**  
> The target IP address belongs to TryHackMe's private internal network. To access it from your local Kali Linux machine, you must connect to the network using the OpenVPN configuration file (`sudo openvpn your-config.ovpn`). Alternatively, you can use TryHackMe's in-browser **AttackBox**, which is already connected to the lab network.

---

## 1. Challenge Overview
The challenge provides a web-based chess game called **Endgame Trainer**. The game displays a "mate-in-one" board position where white can win immediately. The main objective is to deliver the checkmate move and capture the hidden flag.

<img width="1047" height="746" alt="Screenshot (317)" src="https://github.com/user-attachments/assets/1127180a-6014-40ed-840d-c8c2d3545afe" />

---

## 2. Reconnaissance & Initial Attempt

<details>
<summary>Click here</summary>

1. Opened Kali Linux and ensured the OpenVPN connection was active.
2. Navigated to `http://<TARGET_IP>` in the web browser.
   
   <img width="375" height="80" alt="image" src="https://github.com/user-attachments/assets/32c6a133-d25f-44ba-a382-27eae9c7e4f2" />

3. Attempted to play the winning checkmate move directly on the interactive board by dragging the Rook from square **`a1`** to square **`a8`**.

   <img width="689" height="700" alt="image" src="https://github.com/user-attachments/assets/2cf838b7-a497-4f84-ba7b-94856cdd33b0" />
  
4. The browser immediately blocked the move and displayed a warning popup:

    <img width="416" height="194" alt="image" src="https://github.com/user-attachments/assets/d96e118c-54c2-4732-9d84-2d078d07952c" />

</details>

---

## 3. Vulnerability Analysis
<details>
<summary>Click here</summary>

The security check preventing the checkmate move only runs on the **client side** inside the web browser using JavaScript. 

<img width="546" height="219" alt="image" src="https://github.com/user-attachments/assets/df09c690-273e-4a5b-9227-fe5163c06a45" />

Because the backend server does not verify or enforce these game restrictions independently, the rule can be bypassed. There are two primary ways to exploit this flaw:
- Sending a direct HTTP `POST` request with the winning coordinates via terminal using `curl`.
- Intercepting and modifying the live browser traffic using **Burp Suite**.

This write-up demonstrates the **Burp Suite** method to inspect and manipulate the HTTP request.

</details>

---

## 4. Exploitation via Burp Suite

<details>
<summary>Click here</summary>

### Step 1: Launch Burp Suite
Opened Burp Suite in Kali Linux, kept the default settings by clicking **Next** followed by **Start Burp**, then navigated to the **Proxy** > **Intercept** tab and clicked **Open browser**.

   <img width="1096" height="685" alt="image" src="https://github.com/user-attachments/assets/e0c2708a-a0d9-405a-a1bf-fdaca0b78c3f" />

### Step 2: Intercept a Valid Move
1. Navigated to `http://<TARGET_IP>` inside the Burp browser.

  <img width="455" height="114" alt="image" src="https://github.com/user-attachments/assets/d71de5c1-2781-4903-8f43-3e4ffb73c8c2" />
 
2. Go back to BurpSuite and toggled **Intercept is on** inside Burp Suite.

   <img width="526" height="163" alt="image" src="https://github.com/user-attachments/assets/967184bf-0178-409c-868e-305303eda9b4" />

3. Made a normal, permitted move on the chessboard (such as moving a pawn) to trigger an outgoing HTTP request inside BurpSuite.

### Step 3: Modify the Payload
In Burp Suite, inspected the paused request and edited the JSON parameters to the checkmate coordinates:

   <img width="831" height="305" alt="image" src="https://github.com/user-attachments/assets/d0128acd-73b9-4333-9f03-9ba5a9f3a1cc" />

Change to this:

```json
{
  "from": "a1",
  "to": "a8"
}
```
### Step 4: Forward the Request & Retrieve Flag
Clicked Forward in Burp Suite to send the modified data to the server.

Switched Intercept is off.

The backend server accepted the move without client restrictions, completing the checkmate and rendering the flag on the screen.

  <img width="1044" height="764" alt="image" src="https://github.com/user-attachments/assets/9d7bf317-fbb4-47be-9579-28703842582e" />

</details>



