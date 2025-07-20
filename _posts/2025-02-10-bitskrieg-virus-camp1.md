---
title: bitsCTF - Forensics - Virus Camp 1
date: 2025-02-10 18:00:00 +0100
categories: [CTFs, bitsCTF]
tags: [ctfs, bitsCTF, forensics]
description: What did Alice really download?
author: <author_id>

image:
  path: /assets/img/bitsCTF/bitsCTF.jpg
---

## Challenge description 

> Alice was just hired as a junior dev and she is absolutely obsessed with light themes. While customizing her work laptop she suddenly found out that their top secret flag was encrypted. Can you figure out how this happened and unconver a few flags in the process?
>
> *Find the flag.*
{: .prompt-info }

**Flag : BITSCTF{H0w_c4n_vS_c0d3_l3t_y0u_publ1sh_m4l1cious_ex73nsi0ns_SO_easily??_5a7b336c}**

## Ressources

As with the [first forensics challenge](https://emree-1.github.io/posts/bitskrieg-baby-dfir/), we are provided with a `.ad1` disk image file. However, this challenge is more complex, and requires a deeper investigation to uncover its secrets.

## Tools 

To examine the `.ad1` file, we will use `FTK Imager`, and for further analysis, I will be leveraging `Autopsy`. 

While `FTK Imager` can handle the majority of tasks, I prefer `Autopsy` for its more advanced features and detailed investigative capabilities.

## Walkthrough

### Initial Analysis 

After opening the `.ad1` file in FTK Imager (see the [first challenge](https://emree-1.github.io/posts/bitskrieg-baby-dfir/) for details on how to do that), I extracted all files from the `.ad1` image for further analysis with `Autopsy`.

In `Autopsy`, I started a new case, selected the basic Ingest modules, and began reviewing the data. One of the first findings was the presence of Microsoft Edge's History database.   
This database provided insight into some `web searches` conducted by Alice, which appeared to be related to her attempt to install light themes across various applications.

![Web Searches section from Autopsy showing Alice's searches for light themes and the Dracula theme.](/assets/img/bitsCTF/forensics/virus-camp1/web_searches.png){: w="700"}

Combining the search queries with the `Web History` artifacts, we can see the specific pages Alice accessed. Notably, after searching for a light theme for VS Code, she visited a site that likely hosted a bash script:

![Web History section from Autopsy displaying visited URLs, including a download link from bashupload.com.](/assets/img/bitsCTF/forensics/virus-camp1/web_history.png){: w="700"}

In addition to the `URLs`, the Web History database also recorded the `web downloads` initiated by Alice. These entries indicate that Alice downloaded and installed scripts from the website `bashupload.com`.

![Web Downloads section from Autopsy confirming the downloaded VS Code extension from bashupload.com.](/assets/img/bitsCTF/forensics/virus-camp1/web_downloads.png){: w="700"}

Despite the download entries, we were unable to locate the corresponding files in the Downloads folder. There were also no `Zone.Identifier` artifacts, suggesting that the downloaded files may have been deleted or erased.

Based on our initial analysis, we can conclude that a script—likely associated with VS Code—was installed. Alice initially believed it was a VS Code light theme extension, but was it really?

### VScode extension

By default, VS Code extensions are typically downloaded to the following directory: `C:\Users\<username>\.vscode\extensions`. Upon investigation, we identified an extension named `undefined_publisher.activate-0.0.1` in the `.ad1` disk image.

Examining the `extension.json` file associated with this extension, we found that it was the only extension installed, but there was limited information available. Notably, the author of the extension was not specified, raising some red flags regarding its legitimacy. Additionally, we discovered the installation timestamp, recorded as `1738923206921` in UNIX format, which translates to Friday, 7 February 2025 at 10:13:26.921 UTC.

![VS Code extensions folder and extension.json file, showing details about the installed extension.](/assets/img/bitsCTF/forensics/virus-camp1/vscode_extensions_infos.png){: w="700"}

Next, we decided to dig deeper to understand the functionality of this extension. Navigating to the `C:\Users\<username>\.vscode\extensions\undefined_publisher.activate-0.0.1\out` directory, we located a JavaScript file named `extension.js`. This file appeared to contain the core logic of the extension.

A quick inspection of the file reveals an interesting variable named `scriptContent`, which appears to contain obfuscated base64 data. Additionally, at the end of the JS file, we find a comment that also seems to hold Base64-encoded data.

![Base64-encoded data found in extension.js, potentially containing hidden information.](/assets/img/bitsCTF/forensics/virus-camp1/base64_flag.png){: w="700"}

Another approach to locating the endoded flag was to use my variant-based keyword search tool, [KeyFind](https://emree-1.github.io/posts/KeyFind/), and search for a base64-encoded variant of the `CTF` keyword within the `.vscode` directory.

```bash
python3 KeyFind.py '\BITSCTF\DFIR\Virus Camp 1\dump\.vscode' -k BITSCTF bitsCTF BITS CTF ctf -v b64 

  > KEYWORDS

 QklUU0NU || Yml0c0NU || QklU || Q1 || Y3

  > SUMMARY

|     | keyword  | source        | count |
| --- | -------- | ------------- | ----- |
| 0   | QklUU0NU | b64 (BITSCTF) | 0     |
| 1   | Yml0c0NU | b64 (bitsCTF) | 0     |
| 2   | QklU     | b64 (BITS)    | 0     |
| 3   | Q1       | b64 (CTF)     | 1     |
| 4   | Y3       | b64 (ctf)     | 0     |

Total : 1

| line_num | keyword | result                          |
| -------- | ------- | ------------------------------- |
| 66       | Q1      | // VGhlIDFzdCBmbGFnIGlzOiBCS... |
```

After decoding the Base64 string, we obtained the following output:

![Decoded flag, revealing the final piece of information extracted from the investigation.](/assets/img/bitsCTF/forensics/virus-camp1/flag.png){: w="700"}

### Forensic timeline 

Based on the information we have gathered, we can construct the following forensic timeline, which outlines the key activities performed by Alice during the investigation period.

| Timestamp                 | Type             | Information                                                                                      |
| ------------------------- | ---------------- | ------------------------------------------------------------------------------------------------ |
| *2025-02-07 09:52:42 UTC* | Web search       | Alice searches for `best light themes windows`.                                                  |
| *2025-02-07 09:53:58 UTC* | Web search       | Alice searches for `best edge light theme`.                                                      |
| *2025-02-07 09:54:02 UTC* | Web search       | Alice searches for `best vs code light themes` and finds one called `dracula`.                   |
| *2025-02-07 09:54:17 UTC* | Web search       | Alice refines her search for `dracula` to gather more information about it.                      |
| *2025-02-07 09:54:25 UTC* | Web search       | Alice narrows her search to `dracula vscode`, as the previous search was too broad.              |
| *2025-02-07 09:56:06 UTC* | Web search       | Alice refines her search further to `light dracula theme`.                                       |
| *2025-02-07 10:12:42 UTC* | Web history      | Alice accesses a website, `http://bashupload.com/W5TtQ/07oh_.vsix`, likely to download a script. |
| *2025-02-07 10:12:45 UTC* | Web download     | Alice downloads the script from `http://bashupload.com/W5TtQ/07oh_.vsix?download=1`.             |
| *2025-02-07 10:13:26 UTC* | VScode extension | The script is installed as a VS Code extension.                                                  |

## Conclusion

It was fun to work with FTK Imager and observing how the investigation unfolded—starting from a simple web search and leading to the discovery of a potentially malicious extension. However, as you may have noticed, we haven't fully explored the content of the extension yet. Perhaps that will be the focus of the [second part](https://emree-1.github.io/posts/bitskrieg-virus-camp2)?

Thank you for reading my write-up! See you in the next one! 

*emree1*
