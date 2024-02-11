# ROOT_ME CTF

Today I’m going to do a write up on the Root_me ctf on tryhackme. This is labeled as an easy ctf, and should be a good test of what I’ve learned on vulnerability testing so far!

First step, start the room and obtain the machine IP. Check!

![Untitled](ROOT_ME%20CTF%2016da511ae8b64041b5ba323a61457e02/Untitled.png)

![Untitled](ROOT_ME%20CTF%2016da511ae8b64041b5ba323a61457e02/Untitled%201.png)

The first round of questions revolve around reconnaissance, looking for open ports and service versions. So lets run a Nmap scan on the target and see what comes up.

![Untitled](ROOT_ME%20CTF%2016da511ae8b64041b5ba323a61457e02/Untitled%202.png)

Looks like we have SSH port 20 and HTTP port 80 open. I also see Apache version 2.4.29. We can answer the first few questions just off of the nmap scan.

![Untitled](ROOT_ME%20CTF%2016da511ae8b64041b5ba323a61457e02/Untitled%203.png)

Next question is asking me to run Gobuster to find other directories on the web server. So lets run Gobuster with a directory wordlist and see what comes up. While that runs, I’m going to visit [http://10.10.132.92:80](http://10.10.132.92:80) and see what information we can find there.

![Untitled](ROOT_ME%20CTF%2016da511ae8b64041b5ba323a61457e02/Untitled%204.png)

I inspect the page source code to see if any notes were left, but no clues there.

![Untitled](ROOT_ME%20CTF%2016da511ae8b64041b5ba323a61457e02/Untitled%205.png)

Gobuster is finished scanning and I found a few directories available. /uploads and /panel.

![Untitled](ROOT_ME%20CTF%2016da511ae8b64041b5ba323a61457e02/Untitled%206.png)

We can go ahead and finish the recon section by selecting “completed” and filling in the blank directory with /panel/.

![Untitled](ROOT_ME%20CTF%2016da511ae8b64041b5ba323a61457e02/Untitled%207.png)

Moving on to the next section of the CTF, it’s labeled “Getting a shell”. It says to get a reverse shell by leveraging the uploads form. Sounds easy enough!

![Untitled](ROOT_ME%20CTF%2016da511ae8b64041b5ba323a61457e02/Untitled%208.png)

Lets check out the 2 directories I found during the Gobuster search. /panel allows me to upload to the web server, and /uploads shows what has been uploaded. In theory, I can upload a reverse shell, and then execute it from the /uploads page.

![Untitled](ROOT_ME%20CTF%2016da511ae8b64041b5ba323a61457e02/Untitled%209.png)

![Untitled](ROOT_ME%20CTF%2016da511ae8b64041b5ba323a61457e02/Untitled%2010.png)

I loaded up Burpsuite, navigated to 10.10.132.92/panel, and clicked upload to get an error. That way I could analyze the traffic as it’s attempting to upload. There’s some useful information on the POST packet, including a PHP session ID cookie. That tells me the web server is using PHP, and I can use pentestmonkey’s php reverse shell on it!

![Untitled](ROOT_ME%20CTF%2016da511ae8b64041b5ba323a61457e02/Untitled%2011.png)

![Untitled](ROOT_ME%20CTF%2016da511ae8b64041b5ba323a61457e02/Untitled%2012.png)

Going over to Revshells.com, I can have it generate a shell for me by just adding my IP address.

![Untitled](ROOT_ME%20CTF%2016da511ae8b64041b5ba323a61457e02/Untitled%2013.png)

Lets nano a file called “php-reverse.php”.

![Untitled](ROOT_ME%20CTF%2016da511ae8b64041b5ba323a61457e02/Untitled%2014.png)

Now we can paste our reverse shell code directly in that document.

![Untitled](ROOT_ME%20CTF%2016da511ae8b64041b5ba323a61457e02/Untitled%2015.png)

After saving, lets upload it to the server. 

![Untitled](ROOT_ME%20CTF%2016da511ae8b64041b5ba323a61457e02/Untitled%2016.png)

OUCH! I was so sure that this would work. I had to take a break at this point and wonder if I’m going down the right path, but while away, I remembered I could change the extension to .php5 and possibly get past the restriction. 

![Untitled](ROOT_ME%20CTF%2016da511ae8b64041b5ba323a61457e02/Untitled%2017.png)

Okay that worked. Admittedly, I spent way too long on this before walking away and drinking some coffee. Now, lets check our uploads page and see if our shell is in there.

![Untitled](ROOT_ME%20CTF%2016da511ae8b64041b5ba323a61457e02/Untitled%2018.png)

perfect! Lets set up a listener and be ready to catch this thing! I found a simple but great python program called penelope on github by brightio. It listens for a shell, and automatically upgrades the shell for me. It’s a time saver for sure. After configuring penelope, I click on my php file and caught the shell!

![Untitled](ROOT_ME%20CTF%2016da511ae8b64041b5ba323a61457e02/Untitled%2019.png)

We have a shell! One of the best feelings watching it get caught in real time! looks like we are in as www-data. After using “id’ I can see its part of the www-data group. First things first, lets look for the user flag.

![Untitled](ROOT_ME%20CTF%2016da511ae8b64041b5ba323a61457e02/Untitled%2020.png)

Surprisingly on an easy CTF, the user flag isn’t in either of the users home folders. Lets use find / -type f -name user.txt 2>/dev/null to find the user.txt file and weed out all of the errors.

![Untitled](ROOT_ME%20CTF%2016da511ae8b64041b5ba323a61457e02/Untitled%2021.png)

And when we cat the file, we find our user flag.

![Untitled](ROOT_ME%20CTF%2016da511ae8b64041b5ba323a61457e02/Untitled%2022.png)

Great! lets put it in the required field and move on.

![Untitled](ROOT_ME%20CTF%2016da511ae8b64041b5ba323a61457e02/Untitled%2023.png)

The next section is called “Privilege escalation” and the first task is asking us to look for files with SUID permissions. After that, finding a file that looks weird compared to the rest.

![Untitled](ROOT_ME%20CTF%2016da511ae8b64041b5ba323a61457e02/Untitled%2024.png)

Lets use “find / -perm -u=s -type f 2>/dev/null to search for suid files.

![Untitled](ROOT_ME%20CTF%2016da511ae8b64041b5ba323a61457e02/Untitled%2025.png)

python with SUID permissions? That looks promising. After answering the question about the weird file, I know were on the right track.

![Untitled](ROOT_ME%20CTF%2016da511ae8b64041b5ba323a61457e02/Untitled%2026.png)

Insert multiple google searches about exploiting python with SUID permissions, I finally land on GTFObins. 

Lets head to GTFObins! Going to the python section and scrolling down to SUID, I see a python script that escalates permissions. lets copy that and put it in our shell.

![Untitled](ROOT_ME%20CTF%2016da511ae8b64041b5ba323a61457e02/Untitled%2027.png)

After running the script, whoami revealed we are root, and we were able to cd to the root directory for the root flag.

![Untitled](ROOT_ME%20CTF%2016da511ae8b64041b5ba323a61457e02/Untitled%2028.png)

![Untitled](ROOT_ME%20CTF%2016da511ae8b64041b5ba323a61457e02/Untitled%2029.png)

# Conclusion:

This room was a blast, and admittedly I had to walk away a few times from being stumped. When getting an error uploading the .php file, I had some doubt on if I was going down the right path with pentestmonkey’s shell. Luckily enough, the solution popped out at me while drinking coffee, it’s crazy how that happens.

 I also took a while before landing on GTFObins for the python script, which took me numerous google searches to realize I was going overboard and overthinking my solution. All in all, this room was a fun challenge and I look forward to the next one!