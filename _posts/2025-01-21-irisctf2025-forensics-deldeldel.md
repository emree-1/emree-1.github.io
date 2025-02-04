---
title: IrisCTF2025 - Forensics - Deldeldel
date: 2025-01-21 18:00:00 +0100
categories: [CTFs, IrisCTF2025]
tags: [ctf, irisctf2025, forensics]
description: xxxx
author: <author_id>

image:
  path: /assets/img/irisctf2025/irisctf_logo.png
---

## Challenge description 

> I managed to log more than just keys... perhaps it was too much data to capture?
>  
> *Find the flag.*
>  
> *By: skat*
{: .prompt-info }

**Flag : `irisctf{this_keylogger_is_too_hard_to_use}`**

## Ressources

For this challenge we were given a .pcapng file named "klogger.pcapng", containing some USB trafic.

## Tools 

We will mainly using Wireshark to analyse this .pcapng file.  
We will also use a custom python script in order to parse some keyboard strokes. 

## Resolution

### Initial Analysis 

First we look at some general information about the trafic inside the capture file. To do so we will use the statistics provided by Wireshark. 

Looking at the protocol hierarchy section, we can see that this pcapng file only containes USB packets. This will make our analysis easier as protocol communication is pretty straight forward.

![xxx](/assets/img/irisctf2025/forensics/deldeldel/protocol_hierarchy.png)

Now that we know this packet contains only USB trafic, we now to know who is exchanging with who. To do this, we can have a look at the Endpoints listing. 

![xxx](/assets/img/irisctf2025/forensics/deldeldel/endpoints.png)

We can see six devices : 
- 1.3.2
- 1.5.1
- 1.5.2
- 1.7.1
- 1.7.2
- host

Looking at the actual communications happening in this packet, we see typicall USB communications where a device is exchanging with the host only. 

![xxx](/assets/img/irisctf2025/forensics/deldeldel/conversations.png)

### 



## Conclusion


*emree1*
