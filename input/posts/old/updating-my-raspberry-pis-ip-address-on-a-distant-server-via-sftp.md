Title: Updating my Raspberry Pi's IP address on a distant server via SFTP
Published: 05/07/2013
Tags: [Migrated, Rpi] 
---

Now I have my external IP address, I want to collect it on a regular basis and push it to a server that accepts SFTP. Now sadly, that server doesn't allow me to use passkey files, so I have to workaround using sshpass (a great little module allowing to pass passwords to the SSH commands - install as usual with ```apt-get install sshpass``` ).

The EOF at the end of the sftp command is to ensure that I am passing the rest of the commands to the sftp command. I could also use a -b flag and a file, but it would mean one more file to deal with.

```bash
#!/bin/bash
wget -q -O- http://ip4.me |
grep -o '\[0-9\]\\{1,3\\}\\.\[0-9\]\\{1,3\\}\\.\[0-9\]\\{1,3\\}\\.\[0-9\]\\{1,3\\}' > ip.txt
sshpass -p XXXXX sftp -oPort=22 username@xxx.cloudplush.com <<-EOF
cd /location
put ip.txt
bye
```

Then it was a simple matter of putting that in a crontab job, running it every five minutes is enough. Calling crontab crons.txt launched the job. crontab -l lists the jobs.
```cron
\*/5 \* \* \* \* ~/go.sh >/dev/null
```