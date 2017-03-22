#Flag Collection - 80 Points

We are given a zip file containing many countries' flags and a Thumbs.db file. 

Although my first instict was to check the Thumbs.db, I figured I might as well make sure there was nothing in the images. Just to make sure, I found the same exact [zip file](http://flags.fmcdn.net/data/flags-normal.zip) online, and computed the checksums of the images in the given zip and the checksums of the images in the zip file I found, and compared them. The checksums were the exact same, revealing that there was nothing stored in the images.

To read the Thumbs.db file, I used a tool called [Thumbs Viewer](https://thumbsviewer.github.io/). Using the tool and opening the Thumbs.db file, I found that one of the recently deleted files was a QR code.

![](https://raw.githubusercontent.com/VoidMercy/EasyCTF-Writeups-2017/master/forensics/Flag%20Collection/qrcode.PNG)

Scanning it gives us the flag.

##Flag
>easyctf{thumbs.db_c4n_b3_useful}

