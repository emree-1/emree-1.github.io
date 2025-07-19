---
title: L3akCTF - Web - Flag L3ak
date: 2025-07-14 10:00:00 +0100
categories: [CTFs, L3akCTF]
tags: [ctfs, l3akCTF, web]
description: Who knew a search bar could snitch the flag?
author: <author_id>

image:
  path: /assets/img/l3akctf/L3akCTF.png
---

## Challenge description 

> What's the name of this CTF? Yk what to do 😉
>
> *Find the flag.*
> 
> *By: p._.k*
{: .prompt-info }

**Flag : L3AK{L3ak1ng_th3_F14g??}**

## Ressources

For this challenge, we were provided with access to a website named `L3ak's Blog` and the complete `HTML` source code of the page.

## Tools 

I used `Burp Suite` to analyze `HTTP` **requests** and **responses** exchanged between the browser and the server. However, it’s very likely that the entire challenge could be completed using only the **browser’s developer tools**.

## Resolution

### Initial Analysis 

Upon visiting the website, it appeared to be a basic **blog** platform where users can browse and read messages posted by others. A **search box** was also available, suggesting the ability to **filter** the messages by **keywords**.

![Screenshot of the website, displaying some user messages and a searchbox.](/assets/img/l3akctf/web/flag_leak/challenge_web_page.png)

While scrolling through the posts, one specific message stood out — it was supposedly posted by the *admin*, and seemed like it could contain the flag:

```bash
Not the flag?
Well luckily the content of the flag is hidden so here it is: ********************

By admin on 2025-05-13
```

> The post titled *"Real flag fr"* is a decoy. It contains a **fake flag** and is clearly meant to mislead — and yes, I initially fell for it...
>  
{: .prompt-tip }

### Source Code analysis

My first instinct was to examine the **source code**. I suspected that the flag might be retrieved in cleartext by the browser and then hidden via some `JS` or `CSS`.

Reviewing the `HTML` and associated JavaScript, I identified two API endpoints:
- `/api/posts`: Returns **all blog posts**, likely used to populate the page on load.
- `/api/search`: Accepts a **query** and returns only the posts that contain the given search term.

Using `Burp Suite`, I observed the JSON response from `/api/posts`. It confirmed that the *admin*’s message content was already redacted: the flag was replaced with asterisks in the data sent by the server.

This indicated that the content was **filtered** or **obfuscated** server-side, which meant there was no hope of retrieving the flag directly from this endpoint — the browser never received the original message.

At this point, I turned my attention to the `/api/search` endpoint...

### Search API 

I started experimenting with the `search` API by entering various inputs and observing the network traffic using `Burp Suite`. Here's how the endpoint behaves:
1. The client sends a `query` parameter containing a string. 
2. The server responds with all messages containing that string.
However, the query is **limited to 3 characters max**.

At first, I didn't spot anything unusual — I don’t usually specialize in Web CTFs, so it took me a while to make the connection. But then I realized: *"If the flag is hidden in the message and starts with something like `L3AK{`, could it be possible to brute-force the actual flag content character by character?"*

Then I tried with the query `L3A` as a POC to see if it would work:

![Screenshot of a search for "L3A" to see what the API responds.](/assets/img/l3akctf/web/flag_leak/search_correct.png)

This response was extremely promising.

It clearly showed that the server performs the search before applying the content redaction. In other words:
- The message still contains the real flag on the server.
- The search API matches the query against the original content.
- The flag is only masked after the match, before the content is returned to the client.

This opens up a clear attack path: by carefully choosing 3-character queries, it becomes possible to **leak the flag** one character at a time (btw the challenge's name was actually flag leak).

### Brute forcing the flag

The brute-force approach is actually quite straightforward. Since the flag starts with a known prefix (e.g., L`3AK{`), it’s possible to infer each subsequent character by testing all possibilities and observing which ones return a matching message.

The method consists of:
1. Defining an **alphabet** of likely characters.
2. Iteratively building **prefixes** of the flag.
3. For each new character position, **trying all candidates** from the alphabet.
4. **Detecting the correct character** based on whether the API returns a hit.

For the character set, I used a `leet-speak-friendly` alphabet that included:
- Lowercase letters: `a-z`
- Uppercase letters: `A-Z`
- Digits: `0-9`
- Special characters typically used in flags: `{ } _ - ! ?`

A **visual demonstration** is often more effective than a long explanation, so here’s a short presentation of the process in action:

![Video of me trying to feed some inputs to the chall binary to test it.](/assets/img/l3akctf/web/flag_leak/brute_forcing_flag.gif)

To automate the attack, I put together a small `Python script` (well, with some help from `ChatGPT`) to handle the brute-force for me.

```py
import requests

url = "xxx"
alphabet = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789{}_!?.-"

headers = {
    "Host": "xxx",
    "Accept-Language": "fr-FR,fr;q=0.9",
    "User-Agent": "xxx",
    "Content-Type": "application/json",
    "Accept": "*/*",
    "Origin": "xxx",
    "Referer": "xxx",
    "Accept-Encoding": "gzip, deflate, br",
    "Connection": "keep-alive"
}

def test_characters(query):
    for character in alphabet:
        data = {"query": query + character}
        response = requests.post(url, json=data, headers=headers)
        if response.status_code == 200:
            results = response.json().get("results", [])
            if results and "***" in results[0].get("content", ""):
                return character
        else : 
            return None

flag = "L3AK{"
tested_flag = "K{"

while tested_flag != "}" :
    res = test_characters(tested_flag)
    if res : 
        flag += res
        tested_flag = tested_flag[1:]  + res
        print(f"[+] Flag : {flag}")
    else:
        print("No valid character found, stopping search.")
        break
```

> You can retrieve the necessary headers by inspecting requests captured using `Burp Suite`.
>  
{: .prompt-tip }

And that’s it! Just let the script run — it will iterate over each character and, with a bit of patience, eventually reconstruct the entire flag.

![The script is performing the bruteforcing, collecting the flag character by character.](/assets/img/l3akctf/web/flag_leak/brute_forcing_flag.png){: w="450"}

## Conclusion

I don’t usually focus on web challenges, as they often require deep knowledge of specific frameworks or more advanced web exploitation techniques. But this one stood out — it relied purely on analytical thinking and a bit of **creativity**.

It was also the first time I encountered a challenge involving a kind of **side-channel inference**, where the behavior of a search function could be leveraged to gradually extract hidden information. It was genuinely fun and insightful.

Thanks for reading — hope you enjoyed the writeup. See you in the next one!

emree1
