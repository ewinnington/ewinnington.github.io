Title: Python - Neat little file http download routine
Published: 24/04/2012
Tags: [Migrated, Python] 
---

File downloading in Python is not hard, but this neat little routine also picks up the name of the file to be downloaded from the server if you don't specify one. Here's the magic sauce for python 2.x. For Python 3, not much to change, use urllib instead of urlinb2 and urlparse has been merged into urllib.
```
\### Download Section
import urllib2
import urlparse
import shutil
import os
import sys

def download(url, fileName=None):
    def getFileName(url,openUrl):
        if 'Content-Disposition' in openUrl.info():
            # If the response has Content-Disposition, try to get filename from it
            cd = dict(map(lambda x: x.strip().split('=') if '=' in x else (x.strip(),''), openUrl.info()\['Content-Disposition'\].split(';')))
            if 'filename' in cd:
                filename = cd\['filename'\].strip("\\"'")
                if filename: return filename
        # if no filename was found above, parse it out of the final URL.
        return os.path.basename(urlparse.parse.urlsplit(openUrl.url)\[2\])
    r = urllib2.urlopen(urllib2.Request(url))
    try:
        fileName = fileName or getFileName(url,r)
        with open(fileName, 'wb') as f:
           shutil.copyfileobj(r,f)
    finally:
        r.close()
```
I found this on [StackOverflow](http://stackoverflow.com/questions/862173/how-to-download-a-file-using-python-in-a-smarter-way) and have been using it happily.
