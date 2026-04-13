100 pt.
Author: LT 'syreal' Jones

#### Description

Can you find the flag in this disk image? Wrap what you find in the `picoCTF` flag format. Download the disk image [here](https://challenge-files.picoctf.net/c_plain_mesa/aa1f8ba93409887e081435732d7037c45b30a8442853bf07c9e84fe4d0e0bc19/partition4.img.gz).

Hints: 
1. Create a `Sleuthkit` MAC timeline!
2. Sloppy timestomping can yield strange (very old) timestamps
---
#### Finding the flag in partition4.img

1. **Confirm the file type**  
   It's a raw ext4 filesystem (no partition table needed).
   ```bash
	└─$ file partition4.img 
	partition4.img: Linux rev 1.0 ext4 filesystem data, UUID=7a00e9da-98f8-4f0f-b257-95edf422d902 (extents) (64bit) (large files) (huge files)
	```

2. **Generate a bodyfile with all file metadata (including deleted files)**  
   Use `fls` recursively (`-r`) and set the mount point prefix (`-m /`).

   ```bash
   fls -r -m / partition4.img > bodyfile.txt
   ```

	
3. **Create a readable MAC timeline**  
   Use `mactime` to turn the `bodyfile` into a sorted timeline.  
   The `-d` flag gives CSV output (easier to sort/filter in a spreadsheet if you want).

   ```bash
   mactime -b bodyfile.txt > timeline.txt
   # or for CSV:
   # mactime -b bodyfile.txt -d > timeline.csv
   ```

4. **Look for anomalies in the timeline**  
   Open `timeline.txt` (or `timeline.csv`) and search / scroll for **very old or suspicious timestamps**.

   Hint keywords from the problem:  
   - "sloppy timestomping"  
   - "very old timestamps"

   In this case, you’ll find something like:

   ```
   Tue Jan 01 1985 12:00:00       41 macb r/rrw-r--r-- 0        0        4945     /bin/bcab
   ```

   → January 1, 1985 stands out a lot compared to normal file dates (most are recent or epoch).

5. **Extract the file using its `inode` number**  
   The `inode` is **4945**.

   ```bash
   icat partition4.img 4945 > suspicious_file.txt
   ```

6. **Check the content**

   ```bash
   cat suspicious_file.txt
   ```

   Output:

   ```
   NzFtMzExbjNfMHU3MTEzcl9oM3JfNDNhMmU3YWYK
   ```

   This is clearly Base64 (ends with `=` padding, typical length, etc.).

7. **Decode the Base64 string**

   ```bash
   echo "NzFtMzExbjNfMHU3MTEzcl9oM3JfNDNhMmU3YWYK" | base64 -d
   ```

   Output:

   ```
   71m311n3_0u7113r_h3r_43a2e7af
   ```

8. **Wrap it in picoCTF{} format**

   **Flag:**

   ```
   picoCTF{71m311n3_0u7113r_h3r_43a2e7af}
   ```

That’s the complete path of the 1985 timestamp was the deliberate anomaly left by sloppy `timestomping`, and the base64 was just a light layer of hiding.

You solved it perfectly, nice work! 🏴‍☠️