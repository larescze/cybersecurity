# Keberos - zranitelnosti

## Ticket encryption with password hash

### Popis zranitelnosti

Žádost o ticket služby pomocí SPN (Service Principal Name) způsobí, že doménový řadič službu vyhledá a TGS zašifruje ticket pomoci NTLM hashe hesla cíleného uživatelského účtu.

### Příčiny

Tickety služeb jsou šifrované přímo NTLM hashem hesla.

### Dopady

Získání hesla k účtu služby.

### Náprava / doporučení

Doporučeno používat dostatečně silná hesla podle standardu NIST.

### Zranitelnost je možné zneužít těmito útoky

- Kerberoasting 

## User Enumeration

### Popis zranitelnosti

Jména uživatelských účtů v KDC je možné získat opakovanými pokusy o autentizaci.

### Příčiny

Kerberos ukazuje, když  je při autentizaci zadáno správné uživatelské jméno, ale špatné heslo. Útočníkovi toto umožňuje zjistit při útoku hrubou silou, kdy útočí na správné názvy účtů.

### Projevy

Snaha autentizovat se pomocí hrubé síly při znalosti uživatelského jména je logováno jako Kerberos Pre-Auth Failure (Event ID: 4771), ne jako běžné Logon failure (Event ID: 4625) při běžném pokusu o útok hrubou silou.

### Dopady

Snadnější autentizace pomocí útoku hrubou silou.

### Náprava / doporučení

Uzamčení účtu při opakované neúspěšné autentizaci.

### Zranitelnost je možné zneužít těmito útoky

- User Enumeration + útok hrubou silou
- User Enumeration + slovníkový útok

## Unconstrained delegation (TGT Forwarding / Trusted Forwarding)

### Popis zranitelnosti

Unconstrained delegation (neomezená pravomoc) znamená, že poté co se uživatel autentizuje na server s tímto nastavením, uživatelovi TGT tickety jsou zachyceny na serveru. Tyto tickety poté mohou být použity pro Pass-the-hash útok.


### Příčiny

V Active directory nastavení unconstrained delegation umožňuje uživateli/počítači se autentizovat vůči službě, která se poté může autentizovat vůči jiné službě jménem uživatele/počítače. To znamená, že když uživatel/počítač vygeneruje TGS ticket pro první službu, další TGS ticket je generovaný, aby první služba mohla jménem uživatele využít službu druhou.

### Projevy

Počítače a servery s tímto nastavením lze vypsat pomocí modulu ActiveDirectory a funkce Get-ADComputer. Nebo přímo na zařízení v konfiguraci Delegation v Active Directory Users and Computers “Trust this computer for Delegation to any Service (Kerberos Only)”.

### Dopady

Zranitelnost dává prostor pro Pass-the-ticket útoky. Pokud má administrátor domény povolené unconstrained delegation tak má útočník přístup ke všem službám.

### Náprava / doporučení

Zakázat unconstrained delegation pro privilegované a chráněné účty. Místo Unconstrained delegation zvolit Constrained (i to má ale slabiny).

### Zranitelnost je možné zneužít těmito útoky

- Pass-the-Ticket

## Constrained Delegation

### Popis zranitelnosti
 
Constrained delegation má opravovat zranitelnost, která vznikla nastavením unconstrained delegation. Constrained delegation, když povoleno na zařízení, omezuje přesně ke kterým službám zařízení může přistupovat jménem uživatele. Služba S4U2Self využívaná pro constrained delegation má oprávnění žádat o TGS tikety uživatelů bez jejich autentizace.

### Příčiny

Kerberos je rozšířen o služby S4U2Self a S4U2Proxy. Když uživatel přistoupí k službě a ta má povoleno constrained delegation k službě druhé, první služba zažádá o TGS tiket druhé služby pomocí TGS tiketu uživatele. První služba pošle TGS tiket druhé službě a připojí se k ní jako původní uživatel. Tento popis platí pro S4U2Proxy, S4U2Self rozšíření umožňuje tento postup, když se uživatel neautentizuje pomocí Kerberos protokolu. V tomto případě totiž uživatel službě nepošle TGS, který je potřebný pro výše uvedený postup. S4U2Self může zažádat autentizační službu o TGS libovolného uživatele, tento tiket je poté předán S4U2Proxy, která tak může žádat o další tikety.

### Projevy

Povolené Constrained Delegation a užití jiného autentizačního protokolu než Kerberos.

### Dopady

Útočník se může vydávat za libovolného uživatele v doméně vůči službě či účtu služby.

### Náprava / doporučení

Nastavit “Account is Sensitive and Cannot be Delegated” pro privilegované a chráněné účty. Účtům, které musí mít povolené delegation, nastavit silná hesla.

### Zranitelnost je možné zneužít těmito útoky

- Constrained delegation
- Pass-The-Ticket

## Relevantní zranitelnosti

### CVE-2021-37750

- https://github.com/krb5/krb5/commit/d775c95af7606a51bf79547a94fa52ddd1cb7f49
- https://security.netapp.com/advisory/ntap-20210923-0002/

### CVE-2021-3671

- https://github.com/heimdal/heimdal/commit/04171147948d0a3636bc6374181926f0fb2ec83a
- https://github.com/heimdal/heimdal/commit/04171147948d0a3636bc6374181926f0fb2ec83a

### CVE-2021-31962

- https://packetstormsecurity.com/files/163206/Windows-Kerberos-AppContainer-Enterprise-Authentication-Capability-Bypass.html
