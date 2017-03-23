# Decomphose - 300 Points

#### phsst - VoidMercy's writeup

We were given 84 images in 4 folders.

After looking into each of the images, we can see that there are black squares surrounding pixels randomly in each of the images. We surmised that the solution requires you to combine all of the pixels with black pixels surrounding them. Here is my script to do this:

```python
import os
from PIL import Image


flag = Image.new("RGB", (1280, 720))
path = "E:\easyctf\decomphose\decomp1"
for filename in os.listdir(path):
    if (".py" not in filename):
        f = Image.open(path + "\\" + filename)
        width, height = f.size
        for i in range(width):
            for a in range(height):
                add = True
                if (i > 0 and f.getpixel((i - 1, a)) != (0, 0, 0)):
                    add = False
                if (i < width - 1 and f.getpixel((i + 1, a)) != (0, 0, 0)):
                    add = False
                if (a > 0 and f.getpixel((i, a - 1)) != (0, 0, 0)):
                    add = False
                if (a < height - 1 and f.getpixel((i, a + 1)) != (0, 0, 0)):
                    add = False
                if (add):
                    flag.putpixel((i, a), f.getpixel((i, a)))
flag.save("flag.png")
```

![](https://github.com/VoidMercy/EasyCTF-Writeups-2017/blob/master/forensics/Decomphose/flag.png)

## Flag

>easyctf{wh4t_a_5weet_fFLag_2b04e1}
