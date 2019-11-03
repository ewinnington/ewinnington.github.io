Title: Commenting and Uncommenting in vi
Published: 23/07/2013
Tags: [Migrated, Vi] 
---

Sometimes I have to edit large config files in vi. Typically I have to comment out a block or even the whole file. Here's the command to insert a # before every line in the file:
```
:%s/^/# /
```
To comment the next 4 lines (. is current line , . + 3 lines more)
```
:.,.+3s/^/# /
```
And to uncomment the first character from every line:

```
:%s/^#  
```

Or uncomment a section:
```
:.,.+3s/^#
```