---
title: IrisCTF2025 - OSINT - Checking Out of Winter
date: 2025-01-17 18:00:00 +0100
categories: [CTFs, IrisCTF2025]
tags: [ctf, irisctf2025, osint]
description: Let's find out where Adam stayed !
author: <author_id>

image:
  path: /assets/img/irisctf2025/irisctf_logo.png
---

## Challenge description 

> We took our annual road trip to Baja California Sur to visit the beach and play some golf. I like how this location is farther from the city compared to other resorts. I really enjoyed the sweet and savory sauce on the pizza with shredded chicken. After eating, I fell asleep, and half of my legs ended up getting tanned.  
>  
> *Question: Which hotel did Adam stay at?*
>  
> *By: Lychi*
{: .prompt-info }

**Flag : `irisctf{hilton_los_cabos_beach_and_golf_resort}`**

## Ressources

For this challenge, we were given access to Adam Holmes' food blog, which features various articles with photos, reviews of places he visited, and food he tried. One of the articles shared the same name as the challenge, so we quickly guessed it is the one we should focus on.  
→ The website could be accessed with the [following URL](https://osint-food-blog-web.chal.irisc.tf/).

The article's description matched the challenge description. Additionally, we were provided with the following image : 

![Image of the challenge.](/assets/img/irisctf2025/osint/checking_out_of_winter/chall_image.png){: w="400"}

## Tools 

We don’t need any sophisticated tools to solve this challenge; a web browser will suffice. In my case, I’ll be using **`Google Chrome`**.  

## Resolution

### Initial Analysis 

Our goal is to identify the name of the hotel mentioned in the description. Fortunately, the description provides us with some useful information about the location :
- It is located in **`Baja California Sur`**, a state in Mexico.
- The hotel is not in the downtown area.
- It is near the beach and has a **`golf course`**, or is close to one.  

From the picture, we observe :
- The pizza Adam had in the foreground.
- Some lounge chairs and a building with a very **`distinctive  architecture`**, which might be unique enough to identify through a web search.  

It’s quite clear that the building in the background is the hotel we need to find...

### Image lookup

To make use of the picture, I immediately utilized G **`Google Image's reverse lookup`** feature. Since the foreground is not relevant to our search, we cropped the image to focus on the background.   

![Output from google image lookup based on the challenge image.](/assets/img/irisctf2025/osint/checking_out_of_winter/google_image_lookup.png){: w="400"}

<br>

We quickly found a match by reviewing the associated searches. Without even specifying the city, the architecture appeared unique enough to identify the hotel. Interestingly, the name of the hotel includes *Beach* and *Golf*, which makes it a very promising lead.  

Before submitting the hotel’s name, let's confirm our hypothesis.

### Confirming hypothesis

To ensure accuracy, we’ll search for additional pictures of the hotel and compare them with the one we have, paying particular attention to the window shapes.  

We’ll search for images of the hotel we suspect is the answer—Hilton Los Cabos Beach & Golf Resort. By examining these images, we can verify our assumption. It didn’t take long to find several photos that confirmed our hypothesis : 

![First image of Hilton Loc Cabos.](/assets/img/irisctf2025/osint/checking_out_of_winter/hilton_loscabos1.jpg){: w="500"}
![Second image of Hilton Loc Cabos.](/assets/img/irisctf2025/osint/checking_out_of_winter/hilton_loscabos2.jpg){: w="500"}

### Going further

One interesting detail about hotels, particularly large chains like Hilton, is that they often have Street View images showcasing the hotel’s interior. These images might help us pinpoint the lounge chair Adam likely used in the photo. Based on the background, I suspect he was sitting in one of the following chairs :  

![Image of the lounge chair I assume the picture was taken.](/assets/img/irisctf2025/osint/checking_out_of_winter/adams_chair.png){: w="500"}

Unfortunately, identifying the exact lounge chair wasn’t part of the challenge, so I’ll never know if I was correct—unless Adam himself confirms it !

## Conclusion 

And that’s how I discovered that the hotel Adam stayed at was *Hilton Los Cabos* ! This challenge was relatively easy but very engaging. With a bit of luck, it could even have been solved in just one Google Image lookup. It highlights how much information a single picture can reveal and serves as a valuable reminder to be cautious about what we share online.

Thank you for reading my writeup ! I hope it was clear and helpful. Feel free to share your feedback—this is my first writeup, and I’m eager to improve my writing and explanations. See you in the next one ! 

*emree1*
