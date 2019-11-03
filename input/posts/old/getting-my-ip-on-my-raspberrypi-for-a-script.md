Title: Getting my ip on my Raspberry pi for a script
Published: 11/02/2012
Tags: [Migrated, Rpi] 
---

One of those wonderful one-liners when you need your external IP address on a linux box:

```
wget -q -O- http://ip4.me | 
grep -o '\[0-9\]\\{1,3\\}\\.\[0-9\]\\{1,3\\}\\.\[0-9\]\\{1,3\\}\\.\[0-9\]\\{1,3\\}'
```

You can then direct it to a file with the '> myip.txt'
