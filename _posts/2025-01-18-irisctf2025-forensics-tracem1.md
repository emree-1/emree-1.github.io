---
title: IrisCTF2025 - Forensics - Tracem1
date: 2025-01-18 18:00:00 +0100
categories: [CTFs, IrisCTF2025]
tags: [ctf, irisctf2025, forensics]
description: Who could be involved in illegitimate activities ?
author: <author_id>

image:
  path: /assets/img/irisctf2025/irisctf_logo.png
---

## Challenge description 

> Here at El Corp, ethics are our top priority! That's why our IT team was shocked when we got a knock from our ISP informing us that someone on our computer network was involved in some illegitimate activity. Who would do that? Don't they know that's illegal?
> Our ISP's knocking (and so is HR), and we need someone to hold accountable. Can you find out who committed this violation?
>  
> *Question: Find the user who comitted the violation.*
>  
> *By: skat*
{: .prompt-info }

**Flag : `irisctf{lloyd}`**

## Ressources

The challenge began with a `.tar.gz` file, which, when extracted, revealed a single file named `logs.json`. This file contained what appeared to be system or application logs in JSON format. Here’s a brief excerpt to illustrate the data layout :

```json
{"host": "primary", "source": "udp:514", "sourcetype": "syslog", "_time": "2024-12-04 03:37:00.26", "data": {"_raw": "2024-12-04 03:36:59.354785 net-ctrl2 dot1x-proc:1[4409]: <522275> <4409> <WARN> <net-ctrl2 10.35.0.3> User Authentication failed. username=1021321863 ip=0.0.0.0 usermac=ed:d9:44:6c:af:44 authmethod=802.1x servername=cppm01-RADIUS serverip=10.35.0.3 apname=B2-W2 bssid=F4:2E:7F:A7:9B:CD", "timestamp": "2024-12-04 03:36:59.354785", "username": "1021321863", "ip": "0.0.0.0", "usermac": "ed:d9:44:6c:af:44", "authmethod": "802.1x", "servername": "cppm01-RADIUS", "serverip": "10.35.0.3", "apname": "B2-W2", "bssid": "F4:2E:7F:A7:9B:CD"}}
{"host": "primary", "source": "udp:514", "sourcetype": "syslog", "_time": "2024-12-04 03:37:00.54", "data": {"_raw": "2024-12-04 03:36:59.361281 net-ctrl2 dot1x-proc:1[4407]: <522311> <4407> <INFO> <net-ctrl2 10.33.0.3> User Authentication success. username=1021322040 ip=0.0.0.0 usermac=da:5e:26:6b:de:47 authmethod=802.1x servername=cppm01-RADIUS serverip=10.33.0.3 apname=B2-W0 bssid=F4:2E:7F:51:BE:FE", "timestamp": "2024-12-04 03:36:59.361281", "username": "1021322040", "ip": "0.0.0.0", "usermac": "da:5e:26:6b:de:47", "authmethod": "802.1x", "servername": "cppm01-RADIUS", "serverip": "10.33.0.3", "apname": "B2-W0", "bssid": "F4:2E:7F:51:BE:FE"}}
{"host": "primary", "source": "udp:514", "sourcetype": "syslog", "_time": "2024-12-04 03:36:59.86", "data": {"_raw": "2024-12-04 03:36:59.361860 net-ctrl1 dot1x-proc:1[4407]: <522311> <4407> <INFO> <net-ctrl1 10.19.0.3> User Authentication success. username=1021322028 ip=0.0.0.0 usermac=b3:50:21:9d:39:1c authmethod=802.1x servername=cppm01-RADIUS serverip=10.19.0.3 apname=B1-W2 bssid=F4:2E:7F:DC:2A:DE", "timestamp": "2024-12-04 03:36:59.361860", "username": "1021322028", "ip": "0.0.0.0", "usermac": "b3:50:21:9d:39:1c", "authmethod": "802.1x", "servername": "cppm01-RADIUS", "serverip": "10.19.0.3", "apname": "B1-W2", "bssid": "F4:2E:7F:DC:2A:DE"}}
{"host": "primary", "source": "udp:514", "sourcetype": "syslog", "_time": "2024-12-04 03:37:00.00", "data": {"_raw": "2024-12-04 03:36:59.361862 net-ctrl2 dot1x-proc:1[4407]: <522311> <4407> <INFO> <net-ctrl2 10.35.0.3> User Authentication success. username=1021322129 ip=0.0.0.0 usermac=20:56:89:fe:21:4f authmethod=802.1x servername=cppm01-RADIUS serverip=10.35.0.3 apname=B2-W0 bssid=F4:2E:7F:51:BE:FE", "timestamp": "2024-12-04 03:36:59.361862", "username": "1021322129", "ip": "0.0.0.0", "usermac": "20:56:89:fe:21:4f", "authmethod": "802.1x", "servername": "cppm01-RADIUS", "serverip": "10.35.0.3", "apname": "B2-W0", "bssid": "F4:2E:7F:51:BE:FE"}}
{"host": "primary", "source": "udp:514", "sourcetype": "syslog", "_time": "2024-12-04 03:37:01.03", "data": {"_raw": "2024-12-04 03:36:59.362029 net-ctrl1 dot1x-proc:1[4407]: <522311> <4407> <INFO> <net-ctrl1 10.19.0.3> User Authentication success. username=1021321507 ip=0.0.0.0 usermac=7f:44:3d:a7:49:7c authmethod=802.1x servername=cppm01-RADIUS serverip=10.19.0.3 apname=B1-W2 bssid=F4:2E:7F:DC:2A:DE", "timestamp": "2024-12-04 03:36:59.362029", "username": "1021321507", "ip": "0.0.0.0", "usermac": "7f:44:3d:a7:49:7c", "authmethod": "802.1x", "servername": "cppm01-RADIUS", "serverip": "10.19.0.3", "apname": "B1-W2", "bssid": "F4:2E:7F:DC:2A:DE"}}
{"host": "primary", "source": "udp:514", "sourcetype": "syslog", "_time": "2024-12-04 03:37:00.61", "data": {"_raw": "2024-12-04 03:36:59.365003 net-ctrl4 dot1x-proc:1[4409]: <522275> <4409> <WARN> <net-ctrl4 10.67.0.3> User Authentication failed. username=1021321880 ip=0.0.0.0 usermac=f:81:c3:98:7:9e authmethod=802.1x servername=cppm01-RADIUS serverip=10.67.0.3 apname=B4-W1 bssid=F4:2E:7F:9C:CC:EA", "timestamp": "2024-12-04 03:36:59.365003", "username": "1021321880", "ip": "0.0.0.0", "usermac": "f:81:c3:98:7:9e", "authmethod": "802.1x", "servername": "cppm01-RADIUS", "serverip": "10.67.0.3", "apname": "B4-W1", "bssid": "F4:2E:7F:9C:CC:EA"}}
{"host": "primary", "source": "udp:514", "sourcetype": "syslog", "_time": "2024-12-04 03:37:00.65", "data": {"_raw": "2024-12-04 03:36:59.373456 net-ctrl3 dot1x-proc:1[4409]: <522275> <4409> <WARN> <net-ctrl3 10.49.0.3> User Authentication failed. username=1021321461 ip=0.0.0.0 usermac=59:ff:24:dc:c4:2b authmethod=802.1x servername=cppm01-RADIUS serverip=10.49.0.3 apname=B3-W0 bssid=F4:2E:7F:A3:0B:78", "timestamp": "2024-12-04 03:36:59.373456", "username": "1021321461", "ip": "0.0.0.0", "usermac": "59:ff:24:dc:c4:2b", "authmethod": "802.1x", "servername": "cppm01-RADIUS", "serverip": "10.49.0.3", "apname": "B3-W0", "bssid": "F4:2E:7F:A3:0B:78"}}
{"host": "primary", "source": "udp:514", "sourcetype": "syslog", "_time": "2024-12-04 03:37:00.58", "data": {"_raw": "2024-12-04 03:36:59.381512 net-ctrl3 dot1x-proc:1[4407]: <522311> <4407> <INFO> <net-ctrl3 10.49.0.3> User Authentication success. username=1021321159 ip=0.0.0.0 usermac=8b:cd:c:15:4b:be authmethod=802.1x servername=cppm01-RADIUS serverip=10.49.0.3 apname=B3-W3 bssid=F4:2E:7F:DA:AE:69", "timestamp": "2024-12-04 03:36:59.381512", "username": "1021321159", "ip": "0.0.0.0", "usermac": "8b:cd:c:15:4b:be", "authmethod": "802.1x", "servername": "cppm01-RADIUS", "serverip": "10.49.0.3", "apname": "B3-W3", "bssid": "F4:2E:7F:DA:AE:69"}}
```

## Tools 

To tackle this challenge, I opted for `Splunk’s` free version, as it offers a user-friendly graphical interface that simplifies the visual analysis and interpretation of logs. While I chose Splunk for its powerful features, I noticed that some participants used alternative tools, such as `jq`. These tools are lightweight and excellent for handling JSON data in CLI, but I preferred Splunk for its ability to provide insights at a glance through its visualizations.

## Walkthrough

### Initial Analysis 

So let's look at the logs we got there !

![Image showing Splunk's dashboard after uploading file logs.json of the challenge.](/assets/img/irisctf2025/forensics/tracem1/first_analysis.png)
> During the challenge, I accidentally selected the `json_no_timestamp` format when uploading the data to `Splunk`. This caused some information to appear missing in the parsed data. Thankfully, since the file itself was in JSON format, the analysis still went smoothly, and I was able to extract the necessary insights without significant issues.

From my analysis, I was able to extract the following key insights : 
- **Total number of events** : `523,248`.
- **Sources identified** : `DNS`, `DHCP`, `ActiveDirectory`, and a source labeled  `udp:514`, though no additional details were available for the last one.
- **Event timeline** : from `2024-12-04 03:36:59.63` to `2024-12-04 12:27:44.47`.

### DNS logs analysis

Among the various log sources, the DNS logs stood out due to their detailed information. They reveal the websites the user attempted to access (via DNS resolution) and the IP addresses of the devices initiating these requests.

My initial hypothesis was that an individual might have accessed illegal websites using a work computer. This could align with the illegitimate activities mentioned in the challenge.

To investigate this further in `Splunk`, I explored the filtering parameters available. One parameter, `data.queries{}.name`, caught my attention as potentially useful for identifying the queried domains.

![Image showing values extracted for the data queries name parameter.](/assets/img/irisctf2025/forensics/tracem1/data_queries_name_parameter.png){: w="700"}

This parameter looks promising. It appears to contain all the DNS resolutions requested by users. The challenge, however, is that there are over 100 unique values, and manually reviewing them isn’t the most efficient approach.

Fortunately, we’re investigating illegitimate activities, which, by their nature, are likely rare events. `Splunk` conveniently offers a rare event filter, which I utilized to quickly generate a clear barplot showing the top 5 rarest events :

![Barplot of the rarest values for the data queries name parameter.](/assets/img/irisctf2025/forensics/tracem1/rare_events_barplot.png)

You can also retrieve these values directly using the following search filter :

```bash 
source="logs.json"| rare limit=5 "data.queries{}.name"
```

Here’s what I found : 
1. `copious-amounts-of-illicit-substances-marketplace.com` : definitely  suspicious.
2. `smith-wesson.com` : also suspicious.
3. `bmj.com` : a british medical review not relevant here.
4. `breachforums.st` : highly concerning.
5. `welt.de` : a German news outlet, seemingly unrelated.

SSo, we’ve identified three suspicious searches. The next step is to determine who initiated them by examining the `data.src_ip` associated with each query : 

![Screenshot of the first suspicious search](/assets/img/irisctf2025/forensics/tracem1/suspicious_search_log1.png){: w="500"}
![Screenshot of the second suspicious search](/assets/img/irisctf2025/forensics/tracem1/suspicious_search_log2.png){: w="500"}
![Screenshot of the third suspicious search](/assets/img/irisctf2025/forensics/tracem1/suspicious_search_log3.png){: w="500"}

As suspected, all three searches originate from the same source IP: `10.33.18.209`. Now, the goal is to uncover the user behind this IP address.

### Finding suspicious user

This part of the challenge was slightly more complex for me, as it was my first forensics investigation. The objective was to link the IP address to a username, so I delved deeper into the logs, searching for clues. Eventually, I came across the following log entry : 

```json
{
  "host": "primary",
  "source": "udp:514",
  "sourcetype": "syslog",
  "_time": "2024-12-04 03:37:11.32",
  "data": {
    "_raw": "2024-12-04 03:37:09.789047||https://sso.evil-insurance.corp/idp/profile/SAML2/Redirect/SSO|/idp/profile/SAML2/Redirect/SSO|aa820a8eed6e339af60e3c0a5870f8a9|authn/MFA|10.17.108.72|Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_5) AppleWebKit/537.36 (KHTML, like Gecko) Firefox/53.0.3 Safari/537.3|https://sso.evil-insurance.corp/ns/profiles/saml2/sso/browser|vfrancis||uid|evil-insurance.corp|https://sso.evil-insurance.corp/idp/sso|url:oasis:names:tc:SAML:2.0:protocol|urn:oasis:names:tc:SAML:2.0:bindings:HTTP-Redirect|urn:oasis:names:tc:SAML:2.0:bindings:HTTP-POST|3lEZ5SAqS6v6YNmFl/IYc7xCZGtoExx05i5R8A0cuPADVFtOKCCsSztj1vbKooKu3m9xQHIpfx9pwUKeKQO23g==|urn:oasis:names:tc:SAML:2.0:nameid-format:transient|_91394744004c61a4bcf8a7d46f5372be|2024-12-04 03:37:09.789047|_bed8f180-a7f2-4a2a-a9c7-c8c81f0ecb22||||urn:oasis:names:tc:SAML:2.0:status:Success|||false|false|true",
    "timestamp": "2024-12-04 03:37:09.789047",
    "IYc7xCZGtoExx05i5R8A0cuPADVFtOKCCsSztj1vbKooKu3m9xQHIpfx9pwUKeKQO23g": "=|urn:oasis:names:tc:SAML:2.0:nameid-format:transient|_91394744004c61a4bcf8a7d46f5372be|2024-12-04"
  }
}
```

The source `udp:514`, which initially lacked any useful information, revealed an interesting clue when I closely examined the `data._raw` field:

![Screenshot of SSO login log with IP and username.](/assets/img/irisctf2025/forensics/tracem1/sso_log.png)

Using this type of log, I found a way to link the suspicious IP address to a username. To achieve this, I opened the challenge file `logs.json` in `VSCode` and performed a simple search (`CTRL + F`) for `|10.33.18.209`. Here’s what I discovered:

```json 
{
  "host": "primary",
  "source": "udp:514",
  "sourcetype": "syslog",
  "_time": "2024-12-04 04:58:36.95",
  "data": {
    "_raw": "2024-12-04 04:58:35.622504||https://sso.evil-insurance.corp/idp/profile/SAML2/Redirect/SSO|/idp/profile/SAML2/Redirect/SSO|5b52053ac1ab1f4935a3d7d6c6aa4ff0|authn/MFA|10.33.18.209|Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/58.0.3029.110 Safari/537.3 Edge/16.16299|https://sso.evil-insurance.corp/ns/profiles/saml2/sso/browser|llloyd||uid|service.evil-insurance.corp|https://sso.evil-insurance.corp/idp/sso|url:oasis:names:tc:SAML:2.0:protocol|urn:oasis:names:tc:SAML:2.0:bindings:HTTP-Redirect|urn:oasis:names:tc:SAML:2.0:bindings:HTTP-POST|kzYQV+Jk2w3KkwmRjR+HK4QWVQ3qzLPLgA5klV2b8bQT+NLYLeqCZw5xUGKbx1U1158jlnUYRrILtVTtMkMdbA==|urn:oasis:names:tc:SAML:2.0:nameid-format:transient|_60b0fd4b0ed5bba3474faeb85b3944e|2024-12-04 04:58:35.622504|_c4b56d58-625b-49aa-b859-4a2068422979||||urn:oasis:names:tc:SAML:2.0:status:Success|||false|false|true",
    "timestamp": "2024-12-04 04:58:35.622504",
    "NLYLeqCZw5xUGKbx1U1158jlnUYRrILtVTtMkMdbA": "=|urn:oasis:names:tc:SAML:2.0:nameid-format:transient|_60b0fd4b0ed5bba3474faeb85b3944e|2024-12-04"
  }
}
```

![Snippet of SSO login log for the suspicious IP](/assets/img/irisctf2025/forensics/tracem1/suspicious_user.png)

## Conclusion

And this is how i found that the user involved in illegitimade activities was `lloyd`. This challenge turned out to be incredibly engaging ! As my first forensics challenge, it was quite demanding. I spent a good amount of time figuring out how to link the IP address to a username, but thankfully, I managed to solve it.

I also thoroughly enjoyed using `Splunk` for this task. While it might have been overkill for a challenge of this scale, I wanted to use the opportunity to deepen my understanding of the tool.

Thank you for reading my writeup ! I hope it was clear and helpful. I’m eager to hear your feedback, as this is one of my first writeups, and I’m looking forward to sharing more. See you in the next one !

*emree1*
