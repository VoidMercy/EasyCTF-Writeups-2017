# 67k - 400 Points

We were given approximately 67000 binaries, named in order, and we had to reverse each one and append the results together.

Because we were given 67000 binaries, we can safely assume the structure of each binary is very similar to the others. Running one of the executables, we see that we had to input a number, and if the number is correct, the program will give us a character, which we would concatenate with the rest. However, the program does not exit after a correct input. This leads me to believe that we have to reverse the encryption system for outputting the character. Let's take a look at the code in IDA.

Here are the interesting parts:

```
.Gu5D:00402012                 push    offset dword_40306C
.Gu5D:00402017                 push    offset Format   ; "%d"
.Gu5D:0040201C                 call    scanf
.Gu5D:00402022                 add     esp, 8
.Gu5D:00402025                 mov     eax, ds:dword_403000
.Gu5D:00402025                 mov     eax, ds:dword_403000
.Gu5D:0040202A                 mov     ecx, 0A1A8A7EDh
.Gu5D:0040202F                 call    sub_402003
.Gu5D:00402034                 cmp     eax, ds:dword_40306C
.Gu5D:0040203A                 jnz     short loc_40205A
.Gu5D:0040203C                 mov     cl, ds:byte_403007
.Gu5D:00402042                 sar     eax, cl
.Gu5D:00402044                 and     eax, 0FFh
.Gu5D:00402049                 push    eax
.Gu5D:0040204A                 push    offset aWowYouGotIt_He ; "Wow you got it. Here is the result: (%c"...
.Gu5D:0040204F                 call    printf
```

We can see that two things are being moved into registers eax and ecx. Then another function is called.

```
.Gu5D:00402003 sub_402003      proc near               ; CODE XREF: start+29
.Gu5D:00402003                 sub     eax, ecx
.Gu5D:00402005                 retn
.Gu5D:00402005 sub_402003      endp
```

We can see that subtraction is performed on these two values, and stored into eax. Eax is then compared to the user input, presumably, and if it is the same, the character is printed.
We can see that the character is just the correct user input value right shifted by a byte, stored at ds:byte_403007, then binary anded with 0xFF.

Now we know how to obtain the character in each binary. Take the two values moved into eax and ecx, perform an operation (xor, add, or sub depending on the binary), right shift by a byte value, and binary and the result with 0xFF. We now just need to write a script to automate this process. I took advantage of objdump -d and objdump -s to obtain the two operands, the operation, and the right shift value. Here is my script:

```python
import os, subprocess

def reverse(shit):
    ans = ""
    a = 3
    while a >= 0:
        ans += shit[a * 2] + shit[a * 2 + 1]
        a -= 1
    return ans

decoded = open("decoded.txt", "w")

for i in range(67140):
    temp = str(hex(i))[2:]
    filename = "0" * (5 - len(temp)) + temp

    #print (filename)
    
    p=subprocess.Popen(["objdump -d " + filename + ".exe"],stdout=subprocess.PIPE,shell=True)
    (out,err)=p.communicate()
    lines = out.split("\n")
    n1 = -1
    n2 = -1
    for line in lines:
        if ("mov" in line and "eax" in line):
            n1 = line.split("mov")[1].split(",")[0].replace("0x", "").strip()
        if ("mov" in line and "ecx" in line):
            n2 = line.split("mov")[1].split(",")[0].replace("0x", "").strip()
    add1 = 0
    add2 = 0
    if ("$" in str(n1)):
        n1 = str(n1).replace("$", "")
    else:
        add1 = int(n1, 16)
    if ("$" in str(n2)):
        n2 = str(n2).replace("$", "")
    else:
        add2 = int(n2, 16)
        
    p=subprocess.Popen(["objdump -s " + filename + ".exe"],stdout=subprocess.PIPE,shell=True)
    (out2,err) = p.communicate()
    lines = out2.split("\n")
    for a in range(len(lines)):
        line = lines[a]
        startAdd = line.strip().split(" ")[0].strip()
        try:
            if (int(startAdd, 16) <= add1 and int(startAdd, 16) + 16 > add1):
                stuff = line.strip().split(" ")[1] + line.strip().split(" ")[2] + line.strip().split(" ")[3] + line.strip().split(" ")[4]
                #print (stuff)
                try:
                    line2 = lines[a + 1]
                    shit = line2.strip().split(" ")
                    del shit[0]
                    stuff += str("".join(shit))
                except:
                    a=2
                c = add1 - int(startAdd, 16)
                n1 = stuff[c * 2:][:8]
                n1 = reverse(n1)
                #print (n1)
        except:
            a=1
        try:
            if (int(startAdd, 16) <= add2 and int(startAdd, 16) + 16 > add2):
                stuff = line.strip().split(" ")[1] + line.strip().split(" ")[2] + line.strip().split(" ")[3] + line.strip().split(" ")[4]
                c = add2 - int(startAdd, 16) + 1
                n2 = stuff[c * 8 - 8:][:8]
                n2 = reverse(n2)
        except:
            a=1
    ans1 = 0
    ans2 = 0
    bin1 = bin(int(n1, 16))[2:]
    bin2 = bin(int(n2, 16))[2:]
    
    ans1 = int(bin1, 2)
    ans2 = int(bin2, 2)


    lines = out.split("\n")
    op = -1
    for line in lines:
        if ("mov" in line and "cl" in line):
            add = line.split("mov")[1].split(",")[0].replace("0x", "").strip()
            break
    for line in lines:
        if ("call" in line and "*" not in line):
            op = line.split("call")[1].replace("0x", "").strip()
            break
    for line in lines:
        if (":" in line and op in line.split(":")[0] and ">" not in line):
            op = line.split("\t")[2].strip().split(" ")[0].strip()
            break

    if ("xor" not in op and "add" not in op and "sub" not in op):
        errFile.write(filename)
    p=subprocess.Popen(["objdump -s " + filename + ".exe"],stdout=subprocess.PIPE,shell=True)
    (out2,err)=p.communicate()
    lines = out2.split("\n")
    add = int(add, 16)
    byte = 0
    for line in lines:
        startAdd = line.strip().split(" ")[0].strip()
        try:
            if (int(startAdd, 16) <= add and int(startAdd, 16) + 16 > add):
                stuff = line.strip().split(" ")[1] + line.strip().split(" ")[2] + line.strip().split(" ")[3] + line.strip().split(" ")[4]
                c = add - int(startAdd, 16) + 1
                byte = stuff[c * 2 - 2:][:2]
        except:
            a=1
    #print ("Number 1: " + n1)
    #print ("Number 2: " + n2)

    shift = byte

    n1 = ans1
    n2 = ans2
    
    ans = 0
    if (op == "sub"):
        ans = int(n1) - int(n2)
    elif (op == "xor"):
        ans = int(n1) ^ int(n2)
    elif (op == "add"):
        ans = int(n1) + int(n2)
    ch = chr((ans >> int(shift, 16)) & 0xff)
    decoded.write(ch)
print ("done")
decoded.close()
```

Note: This problem could also have been done directly from the hex of the binary.

After extracting every character into [decoded.txt](https://github.com/VoidMercy/EasyCTF-Writeups-2017/blob/master/reversing/67k/decoded.txt) we can see that this is obfuscated javascript. We can simply paste the contents of decoded.txt (excluding "Javascript: ") into console, and obtain the flag.

## Flag

>easyctf{double_you_tee_eff?so_mAny_b1ns}
