**InCTF Secure Extractor (Network Pentest): http://172.30.0.4:5000**

This is the first time i write a write-up so forgive me for any mistake!!!
Now let's start the challenge xD

------------------------------------------------

This is very similiar to Hackthebox, and a very good one too but I didn't solve this one during the contest time :'(

You need to use they openvpn config file to access the machine. The web-app originally run on port 5000 but in the screenshot is 5001, reason later lol.

At first, they give us a hint about a web app is exploit-able and "forget to clear history". I don't know what "history" did the admin forget to clear at first, so I just connect to the machine to see what is happening.

![Web App](https://i.imgur.com/WKCDCXp.png)

So web upload, only accept zip, rar, tar. The uploaded file will be extracted on the web server.

I tried to upload a shell, but when open it will download it, only text file are showed. So I tried upload symbolic link to the /etc/passwd file (https://www.cyberciti.biz/faq/creating-soft-link-or-symbolic-link/):
```
ln -s "../../../../../../../../../../etc/passwd" link
zip -y test2.zip link
```
And it work: (skip those lmao file, i forgot to clear the folder xD )
![/etc/passwd](https://i.imgur.com/L5iKCq3.png)

So i upload another file link to current folder, we got Local File Read: 
![LFI](https://i.imgur.com/n09vxYM.png)

After that i check all the file in the folder, and in .bash_history (oh that history lol) we have a credential:
![cred](https://i.imgur.com/uolyJFi.png)

SSH into the server with obtained cred, we got user **joyhopkins**

The machine is isolateed with the internet, so i hosted LinEnum.sh on my pc using python3 http.server and run it. No world readable file, no sudo, nothing special.

The user have no privilege, so i thought the server is running with root. I change the port in the app.py file to 5001, run it to get the pin code and login to the /console on the web-app. But it only have user privilege.

After that i check if any process running with root, but nothing. 

Something is wrong, so i ran the script again, and found a crontab job running a file called **updater**

![crontab](https://i.imgur.com/GcwoxBs.png)

Let's see what inside:

![updater](https://i.imgur.com/rjT3N4F.png)

So basically it download all file in /uploads/packages/ on the server updates.safextractor.lan, run it with **dpgk -i** So we can exploit it by relace updates.safextractor.lan ip with our own ip, or change our. After that place a deb file with **--before-install** tag in that folder. It will execute any script placed in that tag.

Let's check /etc/hosts:

![hosts](https://i.imgur.com/z8YgmDR.png)

We don't have root so we cannot change the /etc/hosts file, but we can change the ip on our machine with **sudo ifconfig tap0 172.30.0.6/28**  ( your network interface may different)

I created a deb file with **sudo fpm -s dir -t deb -n sploit --before-install lmao.sh ./** and place it in **/uploads/packages/**

The **lmao.sh** is a python reverse shell script, i get it on http://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet

And we got reverse shell, with **root**

![root](https://media.discordapp.net/attachments/738757474785427516/739735873347911680/unknown.png?width=845&height=475)

flag is in /root folder

The reason why i didnt solve the challenge during contest time is because I skipped the **updater** file. You learn something new everyday.

Thanks for reading.
