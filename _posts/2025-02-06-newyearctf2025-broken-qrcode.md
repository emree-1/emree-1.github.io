---
title: New Year CTF 2025 - Forensics - Broken QR
date: 2025-02-06 18:00:00 +0100
categories: [CTFs, New Year CTF 2025]
tags: [ctfs, newyearctf2025, forensics]
description: A QR code lost… and found again.
author: <author_id>

image:
  path: /assets/img/newyearctf2025/new_year_ctf_2025.png
---

## Challenge description 

> I took a picture of this QR code on a cybersecurity forum. I just didn't notice two problems: vandals scratched it and someone "imprinted" a palm print on it. Whatever it is, the flag needs to be read...
>  
> *Find the flag.*
>  
{: .prompt-info }

**Flag : grodno{It’s_h4rd_t0_l1ve_without_R33d-S0l0m0n_c0d3s!}**

## Ressources

![Broken QR code given for the challenge](/assets/img/newyearctf2025/broken_qr/qrcode_broken.png){: .right w="150"}

For this challenge, we are given an image of a QR code.

At first glance, the QR code appears to be significantly damaged, as mentioned in the challenge description. We can clearly distinguish a hand-shaped figure overlapping part of the QR code, and it is evident that some sections are missing or altered, potentially preventing it from being scanned properly.

## Tools 

Ce don’t need to install any additional software. However, we will be using [`QRazyBox`](https://merri.cx/qrazybox/), an online tool designed for analyzing and repairing QR codes, made by [Merricx](https://github.com/Merricx).

Additionally, we will need an image editor to manually modify the QR code. In my case, I used `Microsoft Paint`, as I was working on a Windows machine while solving this challenge.

## Resolution

### Initial Analysis 

The first step is to attempt scanning the QR code using `QRazyBox`. However, upon uploading the image, we encounter the following error message:

![Error message about alignment marks missing displayed by QRazyBox](/assets/img/newyearctf2025/broken_qr/qrazybox_alignment_error.png){: w="500"}

This confirms our suspicion—some critical elements of the QR code are missing or corrupted. Before attempting to repair it, we need to understand the structure of a QR code and the function of its various components.

A standard QR code consists of several key elements, each serving a specific purpose, here is a breakdown:
1. `Positioning Markers`: These are large square patterns in three corners of the QR code, allowing scanners to detect its orientation.
2. `Alignment Markers`: Found in larger QR codes, these help maintain readability when the code is distorted. (This seems to be the missing component in our QR code, as suggested by the `QRazyBox` error message.)
3. `Timing Patterns`: Alternating black and white lines that define the grid structure and help scanners determine the QR code's data size.
4. `Version Information`: Specifies the QR code version (ranging from 1 to 40). Most commonly, versions 1 to 7 are used
5. Format Information: Contains details about the error correction level and data masking pattern, aiding in accurate decoding.
6. `Data and Error Correction Codewords`: This section stores the encoded information and redundant data for error recovery.
7. `Quiet Zone`: A blank margin around the QR code, essential for distinguishing it from its surroundings.

By comparing these structural elements with our corrupted QR code, we can immediately see that the `Positioning Markers` are damaged and that the `Alignment Marker` is completely missing—which explains why `QRazyBox` fails to read it.

<div style="display: flex; justify-content: space-between; align-items: center;">
    <img src="/assets/img/newyearctf2025/broken_qr/normal_qrcode_structure.png" alt="Image gauche" style="height: 400px; padding:4px;">
    <img src="/assets/img/newyearctf2025/broken_qr/qrcode_broken2.png" alt="Image droite" style="height: 400px;padding:4px;">
</div>

### Repairing alignment marks 

![Broken QR code after we repari aligment patterns](/assets/img/newyearctf2025/broken_qr/qrcode_alignment_patterns.png){: w="150" .right}

The first step is to restore the `Alignment Pattern`. At first, I wasn't sure if there was a tool that could do this automatically. However, since the alignment pattern follows a fixed structure and location (depending on the QR code size), and this QR code appeared to be a small version (Version 3, we can see it counting the `Timing patterns`), I decided to reconstruct it manually.

Using `Microsoft Paint`, I added black rectangles to replicate the missing alignment pattern. At the same time, I also repaired the damaged `Positioning Marker` to ensure the QR code scanner could properly detect its orientation. This manual process was quite fun, but in the future, I would like to explore ways to automate these steps, possibly through image processing techniques with `python`.

### Reading data

After repairing the QR code, I uploaded it again to `QRazyBox` to test if it could now be decoded.

![Result when we first try to decode the QR code](/assets/img/newyearctf2025/broken_qr/qrazybox_decode1.png){: w="600"}

This time, the tool successfully recognized the QR code structure. However, we can see that some areas are incorrectly interpreted as black squares (data bits), even though no actual data should be present there. This issue is caused by the handprint visible on the original QR code, which altered some sections.

Due to these errors, `QRazyBox` fails to correctly extract the encoded data. The output partially resembles a flag format but the full flag remains unreadable.

### Complete clean

![QRcode after we have completely cleaned it](/assets/img/newyearctf2025/broken_qr/qrcode_completely_clean.png){: w="150" .right}

To recover the original data, we need to manually clean the QR code by removing the erroneous black squares introduced by the handprint. Fortunately, `QRazyBox` makes our job easy thanks to its editor.  
> When solving the challenge, I didn’t realize that `QRazyBox` had an editing feature, so I ended up cleaning the image manually using `Paint`...

With the cleaned QR code, I uploaded it again to `QRazyBox`.

![Result when we decode the QR code after all the cleaning is done](/assets/img/newyearctf2025/broken_qr/qrazybox_decode2.png){: w="600"}

The result is much better, but some data still appears to be missing. This suggests that we might need a method to correct residual errors in the QR code.

### Error correction codes

As mentioned earlier in the QR code structure, QR codes include error correction codewords alongside the actual data. This is achieved using `Reed-Solomon codes`, an error correction algorithm that allows damaged QR codes to be partially or fully recovered.

> **Read-Solomon codes**
> These error correction codes function by adding redundant data blocks to the QR code. These extra blocks don’t store new information but are instead derived from the original data. This allows the scanner to reconstruct missing or corrupted parts, depending on the error correction level of the QR code.
> The higher the level, the more damage a QR code can sustain while still being readable.

`QRazyBox` includes a built-in tool that leverages `Reed-Solomon codes` to correct errors in a QR code. By enabling this feature, we attempt to decode the cleaned image once again.

This time, the tool successfully reconstructs the missing data, revealing the complete flag:
`grodno{It’s_h4rd_t0_l1ve_without_R33d-S0l0m0n_c0d3s!}`.

![Final output from QRazyBox when we read the QR code](/assets/img/newyearctf2025/broken_qr/qrazybox_reed_solomon_codes.png){: w="600"}

## Conclusion

This challenge was a great opportunity to explore QR code structures, error correction mechanisms, and manual recovery techniques. I learned a lot about the way QR codes store and organize data, the different components of a QR code, and how error correction codes enables recovery even when part of the QR code is missing.

Thank you for reading my write-up! See you in the next one! 

*emree1*
