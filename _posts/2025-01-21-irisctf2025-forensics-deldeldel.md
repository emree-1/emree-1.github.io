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

For this challenge we were given a .pcapng file named "klogger.pcapng". Looking at it, it was only containing USB protocol trafic. 

## Tools 

We will mainly be using Wireshark to analyse this .pcapng file.  
We will also use custom python scripts in order to parse some keyboard strokes. 

## Resolution

### Initial Analysis 

First we will look at some general information about the trafic inside the capture file. To do so we will use the Statistics provided by Wireshark. 

Looking at the protocol hierarchy section, we can see that this pcapng file only contains USB packets. This will make our analysis easier as USB communications are pretty straight forward.

> Indeed, USB is using a master-slave approach where the host is requesting connected USB devices for data, and then the connected devices can reply. If a USB device tries to send data without the host asking for it, the packet is not handled by the OS. 
{: .prompt-info}  

![xxx](/assets/img/irisctf2025/forensics/deldeldel/protocol_hierarchy.png)

Now that we know this packet contains only USB trafic, now we need to know who is exchanging with who. To do this, we can have a look at the Endpoints listing. 

![xxx](/assets/img/irisctf2025/forensics/deldeldel/endpoints.png)

We can see several USB devices 1.3.2, 1.5.1, 1.5.2, 1.7.1, 1.7.2 and the host listed as endpoints. 

### Device types

To have more information for each device we can look at the packets they transmit. Based on there length and there structure, we can try to deduce the device type sending the data. 

Looking through the transmistted USB packets, we can see two types of transfer : 
- URB_BULK : mainly used to transfer heavy packets, especially useful for external USBstoreage devices (usb keys, hard drives, SSDs).
- URB_INTERRUPT : mainly used with HID devices, such as a keyboard or a mouse. 

Using the filter usb.transfer_type == "URB_BULK" and looking at the listed Endpoints, we see ther eis only one endpoint appering 1.7.2 (and the host). So this is the only device using "URB_BULK" type of transfers. 

Using the filter usb.transfer_type == "URB_INTERRUPT" we can see that all the other devies are listed. 

### Keylogger?

The filename is klogger.pcapng, as a keylogger is intended to capture every keypress a user doing, we assume at least one of the USB devices must be a USB keyboard.

Based on the HID specifications it is actually pretty easy to spot a USB keyboards as it applying the following criterias : 
- Transmitted data length is equal to 8 bytes.
- Second byte is always equal to 0x00 as it is a Reserverd Byte.
- If a keyboard is used by a human, the device should sometimes send some empty reports.
- Each keypress is represented by a scancode, the association between scancode and value is defined by the USB HID standards.

It is pretty easy to spot a USB keyboards manually, we need to apply the filter usb.data_len == 8 to filter based on length of the actual data carried by the packet. 

Based on our filtering we can see that only two devices are concerned by this length of packet sent.

> Image

Looking at the data sent by the device 1.3.2, it doesn't seem to valid keybord as keyboard scancodes ends at E7 and scancodes from E8 to FFFF are actually reserved. 

> Image

So we can assume the device 1.3.2 is probably not a keyboard. Now looking at the device 1.5.1, we see some more interesting data captures.

> Image

This fits way more the structure of data sent by a keyboard. To be sure we can check for all the properties we mentionned previouvly : 

1. All the data packets transmitted by device 1.5.1 have a length of 8 bytes.
2. All the data packets transmitted by device 1.5.1 have the second byte equal to 0x00.
3. And this devices sends sometimes some empty keywoards, representing that there is no key pressed by the user. 

Based on all the information I gathered about USB protocol and USB keyboard, I decided to create a python script in order to automate the USB keyboard reconnaissance phase. You can look at the guide to see how you can use it, or find the source code on my Github.  

### Extracting information from the keylogger

Assuming device 1.5.1 is indeed the USB keyboard, we would like to know which keys have been pressed by the user, maybe it is hiding some secrects ?

There is a lot of scripts online allowing to do that, you just need to search for "usb keyboard ctf on:github", I opted to choose the one from TeamRocketIst.

Following the instructions on there README, first i extracted the required packets using tshark 

```
tshark -r ./klogger.pcapng -Y 'usb.capdata && usb.data_len == 8 && usb.src == 1.5.1' -T fields -e usb.capdata > usbPcapData
```
- -r specifies the pcap file we want to work on.
- -Y defines a display filter, in this case we are filtering for all the USB packets containing capdata where the data length in the header section is equal to 8, where the source is the device with ID 1.5.1.
- -T fields specifies the actual fields we would like to extract from this packets, in this case the captured data only. 

Then I executed there script as they tellt to do.

```
python3 usbkeyboard.py usbPcapData
```

Obtaining the following output : 

> Image

Just because I am a lazy (or maybe efficient) programmer I decided to code a script automating the extraction and resolution part. You can find a look at the guide, or look the source code on my Github. 

## Conclusion


*emree1*
