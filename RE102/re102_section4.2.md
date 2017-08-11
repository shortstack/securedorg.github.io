---
layout: default
permalink: /RE102/section4.2/
title: Setup
---
[Go Back to Reverse Engineering Malware 102](https://securedorg.github.io/RE102/)

# Section 4.2: Writing a Decryptor #

## The Return Address ##

Before we begin to decrypt the Junk2 data, you need to know where our decrypted junk2 will be located at. Let’s go back to the subroutine that calls the decryption function in `sub_45B794`. Remember that DWORD that you saved earlier in the road map? The value 0x4B27 was added to the address of the newly allocated memory (i.e., the return value of VirtualAlloc).. This value Offset+0x4B27 is being saved in register `esi` and then **pushed** onto the stack before the function returns. Typically functions will **pop** the `ebp` on the stack to restore  the previous stack frame of the calling function. Here the eip will return to Offset+42B7 which is where our decrypted junk2 data will be. 

You should recognize that the malware plans to execute the encrypted Junk2 data here. Now you know the purpose of the Junk2 data which is Position Independent Code (PIC) more typically known as Shellcode. 

![alt text](https://securedorg.github.io/RE102/images/Section4.2_ReturnAddress.png "Section4.2_ReturnAddress")

---

## Export the Key and Shellcode ##

In order to extract the shellcode and the key from the malware you will use the HxD hex editor. 

In IDA, select the shellcode (labeled as `unk_45CCD4`) with starting offset of 0x5BED4. From previous sections, we know that the size of this data is 0x65E4. Open the mbam.exe with HxD and choose **Edit->Select Block**. Plug in the offset and length.

![alt text](https://securedorg.github.io/RE102/images/Section4.2_HxDextract.png "Section4.2_HxDextract")

Copy and save these bytes into a new binary file in HxD hex editor and name it **shellcode.bin**.

Again, in addition to extracting the shellcode, you need to extract the key as well. So, do the same for the key offset and name it as **key.bin**.

---

## RC4 Decrypt Script ##

Let’s code the RC4 key scheduling  and pseudo-random generation algorithm algorithm in Python based on the pseudo code given below:

### Key Schedule Pseudo Code [1](https://en.wikipedia.org/wiki/RC4#Key-scheduling_algorithm_.28KSA.29) ###

```
for i from 0 to 255
    S[i] := i
endfor
j := 0
for i from 0 to 255
    j := (j + S[i] + key[i mod keylength]) mod 256
    swap values of S[i] and S[j]
endfor
```

### Pseudo-random generation algorithm (PRGA) [2](https://en.wikipedia.org/wiki/RC4#Pseudo-random_generation_algorithm_.28PRGA.29) ###

```
i := 0
j := 0
while GeneratingOutput:
    i := (i + 1) mod 256
    j := (j + S[i]) mod 256
    swap values of S[i] and S[j]
    K := S[(S[i] + S[j]) mod 256]
    output K
endwhile
```

### Python Code ###

Here is the python code that mirrors the pseudo code above.

```
import os
import sys


def key_schedule(key):
    keylength = len(key)
    S = range(256)
    j = 0
    for i in range(256):
        k = ord(key[i % keylength])
        j = (j + S[i] + k) % 256
        S[i], S[j] = S[j], S[i]  # swap
    return S


with open(sys.argv[1], 'rb') as key_file, open(sys.argv[2], 'rb') as encrypted, open("decrypted_shellcode.bin", 'wb') as out:
    key_size = os.path.getsize(sys.argv[1])  # 0x20
    key = key_file.read(key_size)
    S = key_schedule(key)

    j = 0
    i = 0

    shellcode_size = os.path.getsize(sys.argv[2])  # 0x65E4

    while (shellcode_size > 0):
        char = encrypted.read(1)
        i = (i + 1) % 256
        j = (j + S[i]) % 256

        # swap
        S[i], S[j] = S[j], S[i]
        k = S[(S[i] + S[j]) % 256]
        shellcode_size -= 1

        out.write(chr(ord(char) ^ k))
    out.close()
    key_file.close()
    encrypted.close()
```

## Error in the Malware’s Decryption Algorithm! ##

![alt text](https://securedorg.github.io/RE102/images/Section4.2_error.gif "Section4.2_error")

If you run the script above against the extracted data, the decrypted data will not make a lot of sense.This is mainly because there is an error in the RC4 algorithm implemented by the malware author. If you pay attention to disassembled code in IDA, you will see between Loop 3 and Loop 4 the register that stores the j variable was never reseted after the key schedule is made.

## Run the Correct Decrypt Algorithm ##

This python script has the correct decryption algorithm.
[decrypt_shellcode.py](https:\\securedorg.github.io/RE102/decrypt_shellcode.py)


In the Victim VM, open up the command prompt and run the following line. Replace location to the folder you stored the bin files and script.
```
 c:\Python27\python.exe <location>\decrypt_shellcode.py  <location>\key.bin  <location>\shellcode.bin
```

Now that you have the decrypted shellcode let’s turn it into an exe so you can analyze it in IDA. The next subsection will provide these instructions.


[Section 4.1 <- Back](https://securedorg.github.io/RE102/section4.1) | [Next -> Section 4.3](https://securedorg.github.io/RE102/section4.3)