# Hex QR - 200 Points

#### phsst - VoidMercy's writeup

We were given a website with a custom "QR" generator and we were given an encrypted image.

We had to find the text to input into the generator to create the same image as the one we were given.

We noticed that as you enter more characters, the size of the image created increased. We found that to create an image similar to the flag image, we had to use somewhere in between 51-63 (odds only) characters. Then when we changed a character, we observed that a unique section of the image was changed. From this, we can brute force the flag while xoring the generated image with the flag image for the maximum match. Here is my python code for this:

```python
import requests
from PIL import Image
import base64
import os
import sys
from bs4 import BeautifulSoup
import time
from PIL import ImageFile

ImageFile.LOAD_TRUNCATED_IMAGES = True

orig = Image.open("hex.png")
orig_w, orig_h = orig.size
orig_p = orig.convert("RGB")

def xor(im):
    c = 0
    width,height = im.size
    imp = im.convert("RGB")
    for x in range(width):
        for y in range(height):
            pix1 = orig_p.getpixel((x,y))
            if (pix1 == (255, 255, 255)):
                pix1 = 1
            else:
                pix1 = 0
            pix2 = imp.getpixel((x,y))
            if (pix2 == (255, 255, 255)):
                pix2 = 1
            else:
                pix2 = 0
            if(pix1 ^ pix2 == 0):
                c += 1
    return c

def extract(html):
    try:
        soup = BeautifulSoup(html, 'html.parser')
        gu = (soup.find('img')['src'])
        return (gu[gu.find("base64,") + 7:])
    except:
        return "bad!"

str1 = list("a" * 57)
for i in range (57):
    cnt = -1
    ascii = "a"
    for j in "abcdefghijklmnopqrstuvwxyz_{}?!":
        blah = str1
        blah[i] = j
        g = open("test.png", "wb")
        print(''.join(blah))
        r = requests.post("http://hexqr.web.easyctf.com/", data={'text': ''.join(blah)})
        res = extract(r.text)
        if(res == "bad!"):
            print("bad!")
        else:
            g.write(base64.b64decode(extract(r.text)))
            g.close()
            gu = Image.open("test.png")
            anr = xor(gu)
            print (anr)
            if(anr == cnt):
                print("problem")
            if (anr > cnt):
                cnt = anr
                ascii = j
        os.remove("test.png")
        time.sleep(0.75)
    str1[i] = ascii
    print("".join(str1))
```

After getting the majority of the characters of the flag, I added "0123456789" into the charset, and finished computing the rest of the flag.

## Flag

>easyctf{are_triangles_more_secure_than_squares?_ae9856ab}
