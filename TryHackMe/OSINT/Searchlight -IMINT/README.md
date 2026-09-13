# Searchlight - IMINT - TryHackMe Write-up

- **Platform:** [TryHackMe](https://tryhackme.com/)
- **Room:** Searchlight - IMINT
- **Difficulty:** Easy
- **Category:** OSINT
- **Tools Used:** 

## Challenge Overview

In this room we will be exploring the discipline of IMINT/GEOINT, which is short for Image intelligence and geospatial intelligence. This room is suited for those of you who are just beginning your OSINT journey or those brand new to the field of IMINT/GEOINT.

This room will introduce you to several topics within IMINT, among them:

- Getting into the right mindset and how to be analytical
- Visually extracting key data points from an image or video
- Applying different tools to assist you in geolocation and answering context questions
  
When you have completed this room you should be comfortable applying tools and methodologies to geolocate and answer context questions based on visual intelligence alone. This room will prepare you for harder CTF challenges in this category as well as real-world geolocation work.

Any thoughts, feedback or issues can be forwarded to me directly on the THM or OSINT Curious Discord. You'll find me there as zewen.

The flag format is: ```sl{flag}``` . This means that every answer needs to be submitted within the brackets, ```sl{your answer}```. No capitalization is needed.

---


## Welcome to the searchlight - imint room

<details>
<summary>Click here</summary>
<br>

**Answer: sl{ready}**


</details>

---



## Your first challenge !
<details>
<summary>Click here</summary>
 <br> 

  

**What is the name of the street where this image was taken?**

> **Answer: sl{carnaby street}**


</details>

---



## Just Google It!

<details>
<summary>Click here</summary>

this task, you was given a photo of public subway (tube). look for the clues in the picture

<img width="1800" height="1440" alt="image" src="https://github.com/user-attachments/assets/fecfd42f-946d-45e8-96d7-45651ac9841a" />



we found 

- the name of the station : circus street underground.

Go to the google and search the name and Go to wikipedia to read more details


<img width="901" height="798" alt="image" src="https://github.com/user-attachments/assets/b83ac3f5-460c-4c5f-a208-dec1fa3c39f9" />



we found :

**1. Which city is the tube station located in?**

> **Answer: sl{london}**

**2. Which tube station do these stairs lead to?**

>  **Answer: sl{piccadilly circus}**

**3. Which year did this station open?**

> **Answer: sl{1906}**

**4. How many platforms are there in this station?**

> **Answer: sl{4}**

</details>

## Keep at it!

<details>
<summary>Click here</summary>
<br>
  
Look closely at the image for any signs, text, or billboards. In this photo, the most important clue is a billboard that says "YVR Connects".


<img width="478" height="267" alt="image" src="https://github.com/user-attachments/assets/95cae119-7095-48af-85b5-569cfc1bf49b" />


we just one google search we have found so many information about our target.

<img width="1226" height="756" alt="image" src="https://github.com/user-attachments/assets/9d55a1fe-caf6-45c1-89de-b45b942ca40c" />



**1. Which building is this photo taken in?**

> **Answer: sl{vancouver international airport}**

**2. Which country is this building located in?**

>  **Answer: sl{canada}**

**3. Which city is this building located in?**

> **Answer: sl{richmond}**



</details>

---

## Coffee and light lunch

<details>
<summary>Click here</summary>
<br>
  
For this task, we need to locate the coffee shop based on the image given to us.

So, if we look closely on the image, we notice there is a building name "The Edinburgh Woolen Mill"

Let search on google.

<img width="891" height="487" alt="image" src="https://github.com/user-attachments/assets/2339b22d-402c-4992-a98c-aec19fa88bb1" />


I found out that this shop was located at "Blairgowrie". Blairgowrie is a city name located at scotland.


**1. Which city is this coffee shop located in?**

> **sl{blairgowrie}**


Im trying to find the coffee shop name by using google maps. since the coffee shop is located infront of the The Edinburgh Woolen Mill.

<img width="1532" height="650" alt="image" src="https://github.com/user-attachments/assets/d9196d2e-2b47-4d76-b683-1fe8db506c84" />

and then try inspect a little bit more closer.

<img width="700" height="468" alt="image" src="https://github.com/user-attachments/assets/b610ab48-4523-4878-b688-f12ce2f54ba6" />

<br>

nice we found it. if you look closely on the coffee shop board, you can found the street name.



**2. Which street is this coffee shop located in?**

> **Answer: sl{allan street}**


<br>

next, let search the coffee shop name on the google to find out more details.

<img width="922" height="502" alt="image" src="https://github.com/user-attachments/assets/00fa8a42-ca1f-4f30-b77b-363c248cd0d5" />


**3. What is their phone number?**

> **Answer: sl{+447878 839128}**

**4. What is their email address?**

> **Answer: sl{theweecoffeeshop@aol.com}**

To find the owner surname, simply search on the google.

<img width="905" height="332" alt="image" src="https://github.com/user-attachments/assets/6d66519e-8245-4ffa-b93d-daef7d4a0328" />


**5. What is the surname of the owners?**

> **Answer: sl{cochrane}**
</details>

---


## Reverse your thinking

<details>
<summary>Click here</summary>
<br>

Before we start find the flag, let us understand first what is reverse Reverse Image.

> 1. What is Reverse Image Search?
> Instead of typing words into a search engine, you give the search engine a picture. It scans the web to find where else that exact image (or a very similar one) appears.

> 2. Why Google Alone Isn’t Enough
> - **The Problem:** Many people only use Google. While Google is great for popular photos, it often misses older, regional, or niche images.
> - **The Solution:** Use multiple search engines because each one indexes different parts of the internet.

> 3. The Main Tools to Use
> - **Google Lens / Google Images:** Good for general web searches, finding products, and pulling text from images.
> - **Yandex:** A Russian search engine that is often much better at recognizing faces, tracking down the origins of obscure photos, and finding visually similar images globally.
> - **Bing:** Microsoft’s search engine, which excels at finding exact copies and recognizing objects or text within photos.
> - **TinEye:** Great for finding the oldest version of an image to see when and where it first appeared online (helpful for spotting fake news or recycled photos).


> 4. Tips for Investigators
> - **Crop the Image:** If an image is busy, crop it down to just a face, a unique landmark, or a sign to get better results.
> - **Check the Dates:** Look at the publication dates of the search results. If a photo is claimed to be from today, but search results show it online from five years ago, you know the claim is false.
> - **Look for Higher Quality:** Use reverse image search to find the original, unedited, high-resolution version of a picture to spot details that are hidden in blurry screenshots.


No we understand the concept of reverse image now lets go find find the flag.

First, lets find the information about the image by directly searching them into search engine.


<img width="1018" height="685" alt="image" src="https://github.com/user-attachments/assets/b3d913d2-46b7-4623-9283-4b7ad274727f" />



<img width="943" height="885" alt="image" src="https://github.com/user-attachments/assets/c3a6dc66-7500-404d-967a-1ac29410f4d1" />


we found the name of the restaurant !

**1. Which restaurant was this picture taken at?**

> **Answer: sl{katz's deli}**

and then Im trying to fiddling around the website to find the name of the Bon Appétit editor that worked 24 hours at this restaurant 

and then i found 

<img width="1280" height="437" alt="image" src="https://github.com/user-attachments/assets/46989610-a2ed-40fe-a7d7-7d4de1c84d91" />



**2. What is the name of the Bon Appétit editor that worked 24 hours at this restaurant?**

> **Answer: sl{andrew knowlton}**
</details>

---


## Locate this sculpture

<details>
<summary>Click here</summary>
<br>

For this task, i use the same method which is i just search the picture using search engine.

<img width="1325" height="741" alt="image" src="https://github.com/user-attachments/assets/4117ff90-e69e-4e03-83c7-7932994d7136" />

**1. What is the name of this statue?**

> **Answer: Who took this image?**

</details>

---


## ...and justice for all
<details>
<summary>Click here</summary>
<br>

i send the picture directly into search engine and then i found the titled of the statue and the location of the statue

<img width="957" height="733" alt="image" src="https://github.com/user-attachments/assets/96e974e3-0b60-48a2-bf9b-3bdf627eb7b2" />


**2. where is this statue located?**

> **Answer: sl{alexandria, virginia}**


But the titled not the name of the character. So i search "justice Delayed, Justice Denied statue character name".


And then i browsing through the site to find more details, then i found out:

<img width="1435" height="155" alt="image" src="https://github.com/user-attachments/assets/a041362a-80e2-4fdc-8a43-566c86df151e" />

but this is not the name of the character either. so i search "other name for blind justice" and then i found the exact name.

**1. What is the name of the character that the statue depicts?**

> **Answer: sl{lady justice}**


To find the name of the building opposite from this statue , we can simply use google maps.

I search the statue name and where it is located.

<img width="1651" height="771" alt="image" src="https://github.com/user-attachments/assets/7ec6083d-076e-482b-800d-b712ef8414ad" />

if you look closedly the building, we found the building name which is "The Westin Alexandria."

but this is not the full name.

So i just simply search again on the google maps and it show the full name of the building.


**3. What is the name of the building opposite from this statue?**

> **Answer: sl{the westin alexandria old town}**

</details>

---

## The view from my hotel room

<details>
<summary>Click here</summary>
<br>

For last task, we were given a video with some details in it.

in the video we know the place located at singapore because we can see marina bay building.

and if we look carefully in the video. we know that this hotel is located near this building

<img width="1346" height="852" alt="image" src="https://github.com/user-attachments/assets/c0d6cd20-8e1a-41b2-87ba-f4a297c8297a" />


in the video also there is place name "Riverpoint". so let search this place on gooogle maps.


<img width="1172" height="792" alt="image" src="https://github.com/user-attachments/assets/ee0445df-7ffb-4a47-8619-fc6667d169db" />


the red mark is the building that we mark earlier from the video !. now lets go to that building so that we can find the hotel.


<img width="1218" height="860" alt="image" src="https://github.com/user-attachments/assets/2688453c-4881-4c41-916c-8f1698d8105c" />


the hotel name is "Novotel" and located at "Clarke Quay" but this is not the fullname.

So we can just simple search "Novotel Clarke Quay" on google maps to know the full names

**1. What is the name of the hotel that my friend stayed in a few years ago?**

**Answer: sl{novotel singapore clarke quay}**
</details>

---
