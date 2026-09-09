> Beginner friendly OSINT Challenge

## Task 1: The Leaked Photo

```
An ACME Jet Solutions employee uploaded a photo of a residential property believed to be linked to ACME Jet's early operations. Can you figure out where the picture was taken to confirm or debunk the rumour? 

Flag format: THM{City}
```

`Given image:`
![[edited-house-1763031553617.jpg]]

When I used the `exiftool` on the given image got the following information

```bash
❯ exiftool edited-house-1763031553617.jpg
ExifTool Version Number         : 13.55
File Name                       : edited-house-1763031553617.jpg
Directory                       : .
File Size                       : 793 kB
File Modification Date/Time     : 2026:09:09 20:25:37+05:30
File Access Date/Time           : 2026:09:09 20:27:11+05:30
File Inode Change Date/Time     : 2026:09:09 20:27:06+05:30
File Permissions                : -rwxrwxrwx
File Type                       : JPEG
File Type Extension             : jpg
MIME Type                       : image/jpeg
Exif Byte Order                 : Big-endian (Motorola, MM)
GPS Latitude                    : 26 deg 12' 14.76"
GPS Longitude                   : 28 deg 2' 50.28"
JFIF Version                    : 1.01
Resolution Unit                 : None
X Resolution                    : 1
Y Resolution                    : 1
Image Width                     : 1306
Image Height                    : 837
Encoding Process                : Baseline DCT, Huffman coding
Bits Per Sample                 : 8
Color Components                : 3
Y Cb Cr Sub Sampling            : YCbCr4:4:4 (1 1)
Image Size                      : 1306x837
Megapixels                      : 1.1
GPS Position                    : 26 deg 12' 14.76", 28 deg 2' 50.28"
```

And we got the exact GPS position where the image was taken

> `GPS Position                    : 26 deg 12' 14.76", 28 deg 2' 50.28"`

And we can see the city in which the photo was taken is `Johannesburg`.

![[Pasted image 20260909203735.png]]

![[Pasted image 20260909203824.png]]

## Task 2: Archived Company Website

```
ACME Jet Solutions (warc-acme.com/jef/), is all over social meda claiming they were founded in 2025 and that they're the fastest-growing data company in Africa.
But something doesn't add up, one of their ex-employees ensures you that the company existed long before that.

Your job as an OSINT investigator is to verify their founding date using only public information.

Flag Format: THM{YYYYMMDDHHMMSS}
```

For this I searched the URL `warc-acme.com/jef/` onto the Internet Archive.

> [!NOTE]
> Because the website name includes `warc` (`Web ARChive`), so we should look at the global **Internet Archive Items metadata** rather than the standard `Wayback Machine` timeline calendar.

![[Pasted image 20260909221723.png]]

And this is the datetime we need for the answer

![[Pasted image 20260909222100.png]]

## Task 3: Mysterious Landmark

```
Further Investigation uncovers another image believed to be connected to the company's international expansion.

Research reveals that to the right of the iconic landmark is a building that played a big role in the fight for independence of a particular country. Signs on the external wall provides the name of the building. 

Submit the name of building translated into English as the flag.

The flag format is THM{Landmark}
```

`Given Image:`

![[landmark-1763035881792.jpg]]

On the left side we can see a banner with which has `DUBLINONE` on it when search on google we get to know that it is a hotel name and one google search about `famous landmark` gives us this result

![[Pasted image 20260909222828.png]]
