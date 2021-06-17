# Zranitelnosti webových aplikací - Útoky proti uživatelům

## Úvod

### HTTP

- Bezstavový protokol, který funguje na principu **dotaz → odpověď**
- Neumožňuje uložení obsahu (nutno použít cookies)
- Data se předávají v **plain textu bez šifrování** (komunikaci lze odposlechnout)
- Port **TCP/80**

### Dotazovací metody

- **GET** - požadavek na uvedený objekt se zasláním případných dat (parametrů)
- **HEAD** - podobná GET, avšak nepředává data, ale pouze metadata o požadovaném cíli (velikost, typ, datum změny, …)
- **POST** - odesílá uživatelská data na server (nová data)
- **PUT** - vkládá data na server (návaznost na již existující data)
- **DELETE** - smaže uvedený objekt ze serveru
- **TRACE** - odešle kopii obdrženého požadavku zpět odesílateli
- **OPTIONS** - dotaz na server, jaké podporuje metody
- **CONNECT** - spojí se s uvedeným objektem přes uvedený port

### HTTPS

- Využívá protokol **HTTP** spolu s protokolem **SSL** nebo **TLS**
- Pomocí **asymetrické kryptografie** je ověřena identita webového serveru, volitelně i klienta
- Následuje dohoda na klíči pro **symetrické šifrování** samotné komunikace
- Zásadní je **infrastruktura veřejného klíče** a **X.509 certifikáty**
- Port **TCP/443**

### Burp Suite

- Penetrační nástroj umožňující průzkum a změny komunikace mezi webovým prohlížečem a serverem (lokální proxy)
- **Proxy**
  - Prostředník mezi klientem a cílovým serverem
  - Překládá klientské požadavky a vůči cíli vystupuje sám jako klient
  - Přijatou odpověď následně odesílá zpět klientovi
- Zachytávání HTTP / HTTPS
- Platforma: Java (multiplatformní)
- Moduly nástroje
  - **Sniper** - jeden útok, označen jeden parametr
  - **Batering ram** - stejná hodnota pro všechny parametry (např. admin/admin)
  - **Pitchfork** - kombinace více listů podle pozice (x1 a y1, x2 a y2, ...)
  - **Cluster bomb** = kombinace slovníků (na každého uživatele z jednoho slovníku se použijí všechny hesla z druhého)

### Web Parameter Tampering (Hidden Fields)

- Úprava předávaných hodnot
- Některé prvky formulářů mohou být skryté
- Výběrové nabídky ve formulářích omezují volbu uživatele
- Skrze lokální proxy lze parametry změnit (Burb Suite)
- Častý problém u (flash) her

## Autentizace

### Útoky na autentizaci

- **Guessing**
  - Brute Force Attack
  - Dictionary Attack
  - Horizontální guessing (stejné heslo postupně na každého uživatele)
  - Vertikální guessing (na jednoho uživatele postupně více hesel)
- **Enumerace uživatelů**
  - Predikovatelné loginy
  - Forced browsing (id)
  - Skrz přihlašovací formulář
  - Skrz registrační formulář
  - Skrz formulář pro reset hesla

#### Obrana před Guessing

- Vynucení silných hesel
- Captcha
- Kontrola IP adresy

## Session Management

- HTTP je **bezstavový protokol**
- Udržování stavu (session) je potřeba řešit odděleně
- Generuje se **session identifikátor SID**
- SID se ukládá do **session storage** na straně serveru společně s identifikací uživatele
- V historii se pro přenos SID používalo URL
- Dnes je standardem ukládání SID do **cookies**

### Útoky na session management

- Session Prediction
- Session Stealing / Hijacking
- Session Fixation
- Session Donation
- Session Puzzling
- Cross-Site Cooking / Cross-Subdomain Cooking
- Same-Site Scripting
- Odposlech SID na síti
- Insufficient Session Expiration
- Insufficient Logout / Logout action availability

#### Session Prediction

- Pokus o uhodnutí SID hrubou silou při krátkých SID, nebo malé množině použitých znaků
- Pro generování SID je použito algoritmů, které nejsou kryptograficky 100% náhodné a je tak možné odhadnout hodnoty cizích SID
- Burp Suite: modul sequencer

#### Session Stealing / Hijacking

- Pokud se SID předává v URL, pak může uniknout skrz **referer**
  - Uživatel klikne na odkaz a přejde na webovou stránku útočníka
  - Webová stránka načte externí obsah (např. obrázek) ze serveru útočníka
- Pokud se SID předává v cookie, je k jeho zcizení potřeba využít například útoku XSS
- Úniku dat skrz referrer lze zabránit HTTP Response hlavičkou **Referrer-policy**
  - Referrer-Policy: no-referrer
  - Referrer-Policy: no-referrer-when-downgrade
  - Referrer-Policy: origin
  - Referrer-Policy: origin-when-cross-origin
  - Referrer-Policy: same-origin
  - Referrer-Policy: strict-origin
  - Referrer-Policy: strict-origin-when-cross-origin
  - Referrer-Policy: unsafe-url

#### Session Fixation

- Útočník může uživateli **podstrčit SID** (ID relace) před přihlášením
- Po přihlášení zůstane uživateli stejné SID
- Útočník zná toto SID a může přistoupit k aplikaci pod **identitou přihlášeného uživatele**
- Nejsnáze proveditelné při předávání ID relace v URL
- Při relacích založených na cookies nutno použít:
  - Cross-Site Cooking
  - Cross-Site Scripting
  - HTTP Response Header Spliting

#### Session Donation

- Princip podobný útoku Session Fixation
- Uživatel obdrží **session od útočníka**
- Pracuje v přesvědčení, že je ve vlastním účtu
- Útočník si následně ve svém účtu může prohlédnout uživatelovu práci
- Útok může posloužit také jako spouštěč skriptu při XSS ve vlastním účtu

#### Session Puzzling

- Pokud se stejně pojmenovaná session proměnná používá na různých místech aplikace k různým účelům

#### Cross-Subdomain Cooking / Cross-Site Cooking

- Pokud je nastavena příliš benevolentní platnost u cookie
- Prohlížeče umožňují nastavit cookie pouze pro domény druhého a vyšších řádů
- Starší prohlížeče považovaly dvouslabičné TLD za domény druhého řádu (co.uk)
- Pozor také na správné omezení cesty platnosti

#### Same-Site Scripting

- Chybný DNS záznam pro localhost
- Chybějící tečka za localhost.
- localhost.example.cz se překládá na 127.0.0.1

#### Odposlech SID na síti

- Wi-Fi sítě
- ARP Poisoning & Routing (APR)
- Nutnost šifrovat komunikaci (HTTPS)
- SSL Strip
- Příznaky u session cookie: HttpOnly, Secure
- HTTP Response Hlavičky
  - Strict-Transport-Security (HSTS)
  - Public-Key-Pins (HPKP)

#### Insufficient Session Expiration

- Znovupoužití starších údajů session nebo SID

#### Insufficient Logout / Logout action availability

- Logout musí být snadno dostupný
- Logout musí zničit session na straně serveru
- Logout by měl také změnit, nebo smazat hodnotu SID uloženou v session cookie

### Obrana Session Management

- Bezpečné session identifikátory
- Nevkládat SID do URL
- Zamezit odesílání refereru
- Po přihlášení vygenerovat nové SID
- Odmítat SID nevygenerované serverem
- Ignorovat SID předané prostřednictvím URL
- Propojit SID s konkrétním uživatelem
- Automatická expirace session při nečinnosti
- Omezení cookies na subdoménu / cestu

# Cross-Site Request Forgery (CSRF)

- Cílem CSRF útoku jsou **koncoví uživatelé**
- Je zneužíváno důvěry serveru v uživatele
- Útok **zneužívá identitu uživatelů** a jejich **uživatelská práva v aplikaci**
- Podle zranitelnosti webové aplikace může, ale také nemusí, být vyžadována spoluúčast oběti
- **Princip CSRF**
  - Útočník nemůže serveru odesílat požadavky jménem jiného uživatel
  - Útočník zneužije uživatele, aby sami nevědomě odeslali potřebné požadavky serveru
  - Použití u HTTP metod (GET, POST)

## Útoky metodou GET

- **Cíle útoků**
  - Hlasování v anketách
  - Provedení akce
- **Možnosti zneužití uživatele**
  - Nalákání uživatele ke kliknutí na připravený odkaz
  - Maskování odkazu přesměrováním
  - Využití odkazů na externí zdroje img, iframe, ...)
  - Použití AJAX

## Útoky metodou POST

- **Cíle útoků**
  - Vkládání příspěvků do diskuzí
  - Změny v nastavení uživatelských účtů
    - Změna přístupového hesla
    - Změna kontaktní e-mailové adresy
    - Přidání adresy pro přesměrování příchozí pošty
- **Příprava útoku**
  - Zjištění informací o formuláři a odesílaných datech
    - Průzkum zdrojového kódu stránky
    - Doplněk pro webový prohlížeč (informace o formulářích)
    - Průzkum síťové komunikace
  - Vytvoření stránky s kopií formuláře (předvyplněná data)
  - Nalákání oběti na připravenou stránku
  - Odeslání dat z formuláře po kliknutí na tlačítko
- **Nedostatky útoku**
  - Viditelnost formuláře => prvky typu hidden
  - Odeslání formuláře kliknutím na tlačítko => automatické odeslání JavaScriptem
  - Zobrazení odpovědi od serveru => vložení formuláře do skrytého rámu
  - Možnost použít k odeslání XMLHttpRequest (AJAX)

## Napadení intranetu

- Intranetové aplikace
- Síťová zařízení ovládaná přes webové rozhraní
- Změny v nastavení hraničních prvků mohou umožnit vstup útočníka do intranetu (hrozba použití výchozích autentizačních údajů)

## Rizika

- Aplikace může trpět i vážnějšími zranitelnostmi, které například po obdržení "správného" požadavku vymažou celou databázi
- Tento požadavek můžeme nevědomě odeslat i vy
- V logu zůstane vaše IP adresa
- Zdroj útoku
  - Návštěva webových stránek
  - Externí zdroje v libovolném dokumentu
  - Odkaz nebo externí zdroj v e-mailové zprávě
- Rizika trvalého přihlášení
  - Identitu je možné zneužít hlavně ve chvíli, kdy je uživatel přihlášen
  - Trvalé přihlášení nabízí útočníkům možnost zaútočit kdykoliv

## Obrana

- **Obrana na straně uživatele**
  - Prakticky neexistuje
  - Zabránit browseru v načítání externích zdrojů a odesílání dat z rámů (IFRAME)
- **Obrana na straně aplikace**
  - Při každé akci vyžadovat heslo
    - Používá se pouze při změně hesla a kritických operacích
  - Kontrola HTTP hlavičky REFERER
    - Možnost odeslání požadavků bez této hlavičky
- **Kontrola hlavičky X-Requested-With nebo ORIGIN u XMLHttpRequestů**
  - Nepřenáší parametry jako Referer, vyskytly se ale exploity, jak hlavičku podstrčit
- **Přidání autorizačního tokenu ke všem požadavkům**
  - Útočník nemůže připravit útočný požadavek bez jeho znalosti
  - Ideální je platnost tokenu časově omezit
- **Nastavení příznaku SameSite u session cookie**
  - Možné hodnoty: None, Lax, Strict
- **Speciální prefixy v názvu cookie**
  - \_\_Secure- musí obsahovat Secure, Set-Cookie musí jít přes HTTPS
  - **Host- jako **Secure, ale navíc musí být nastaveno Path na /

# HTTP Verb Tampering

- HTTP protokol podporuje různé metody
- Metody: GET, POST, PUT, DELETE? HEAD, TRACE, OPTIONS, CONNECT, DEBUG
- PHP interpretuje neznámé metody jako GET
- Aplikace může mylně očekávat requesty pouze konkrétní metodou a obranné mechanismy tak nasazuje pouze u ní

# Cookie Stuffing / Injection

- Nastavení cookie pro cizí doménu
- Během MiTM útoku (HTTP)
- Prostřednictvím rámu
- Ze subdomény
- Zneužitím zranitelnosti aplikace
- Cross-Site Scripting (XSS)
- HTTP Response Splitting

# Clickjacking

- Funkční i při ochraně před útoky CSRF
- Nic netušící uživatel sám klikne na prvek nebo vyplní a odešle formulář bez toho, aby věděl, co vlastně dělá
- Útok založen na možnosti načíst webovou stránku do rámu
  - Průhlednost rámu
  - Překrytí nechtěných prvků
- Princip útoku
  - Útočník zjistí, na která místa je potřeba kliknout pro smazání všech zpráv
  - Vytvoří stránku s prvky, na které donutí kliknout uživatele
  - Útočník překryje obsah své stránky průhledným iframem se zranitelnou stránkou (CSS opacity)
  - Po kliknutí na útočné stránce, vymaže uživatel své zprávy
- Kliknutí na určitý prvek webové stránky je možné zajistit jejím minimalizováním na rám velikosti 1x1 px a tento umístit a dynamicky umisťovat trvale pod kurzor

## Typy Clickjackingu

- Překrytí nevhodného obsahu
- Drag & Drop
- Vykrádání zobrazeného obsahu

### Překrytí nevhodného obsahu

- Nemusí jít jen o klikání
- Útočník může obdobným způsobem přimět svou oběť také k nevědomému vyplnění a odeslání formuláře
- Princip útoku
  - Útočník vytvoření webovou stránku
  - Vloží iframe se zranitelnou stránkou
  - Umístí další rámy pro překrytí obsahu
  - Viditelné zůstanou jen napadené prvky
  - Jakmile uživatel opíše uvedený text a uloží nastavení, provede tím změnu ve svém účtu

### Drag & Drop

- Nevědomé vyplnění formuláře a jeho následné odeslání lze zamaskovat i do nevinně vyhlížející grafické hry využívající HTML Drag&Drop

### Vykrádání zobrazeného obsahu

- Vykrádaná aplikace je načtena v okně
- Uživatel provede označení a kopírování obsahu v tomto rámu s následným vložením do formuláře na stránce útočníka
- V některých případech lze použít také Drag&Drop přetažení obsahu do formuláře
- Formulář je odeslán, obsah získává útočník

## Obrana

- Obrana na straně uživatele
  - Prakticky nereálná
  - Zakázání prvků frame a iframe ve webovém prohlížeči
- Obrana na straně aplikace
  - Možnost načtení zdrojového kódu view-source:
  - XSS filtr
  - Načtení rámu v Sandbox módu HTML5
  - HTTP response hlavička X-Frame Options
    - PHP: header('X-Frame-Options: DENY')¨;
    - Apache: Header set X-Frame-Options Deny
    - Možné hodnoty: DENY, SAMEORIGIN, ALLOW-FROM, ALLOWALL
  - Content Security Policy
    - HTTP hlavičky (X-)Content-Security-Policy
    - Direktiva frame-ancestors

# Path Relative StyleSheet Import (PRSSI)

- Pokud se styly načítají z relativního umístění
- Pokud server ignoruje přidaná lomítka na konci url
- Pokud můžeme do obsahu stránky vložit text obsahující css kód (perzistentně, reflektovaně)
- Možnost injekce vlastních stylů
  - Clickjacking bez použití rámů
  - Krádež dat ze stránky
  - Cross-Site Scripting v IE

# Cross-Domain data Hijacking

- Referrer hijacking
- Javascript hijacking
- Cross-Site Callbacks
- CORS misconfiguration
- Cached data hijacking
- Post & Back Replay Attack
- WWW-authenticate data
- Cross-Site Websockets hijacking

## Javascript hijacking

- AJAX response data je možné vykrádat například pomocí clickjackingu, pokud je načteme do rámu
- Dnes již v prohlížečích ošetřeno
  
## Cross-origin resource sharing (CORS)

- Cross-Site Callbacks (JSONP)
- Vývojáři často pro přenos dat mezi různými doménami využívají callbacků
- Vrácená data se obalují názvem funkce
- Pro načítání dat z externích domén pomocí AJAX mohou vývojáři použít taté CORS politiku definovanou pomocí HTTP response hlaviček
- Lze použít také HTTP Response hlavičku X-Permitted-Cross-Domain-Policies

## Cached data hijacking

- Nutný fyzický přístup útočníka
- Data mohou být uložena v diskové cache, nebo v paměti
- V paměti se data mažou až po zavření prohlížeče
- Cachování a ukládání lze ovlivnit HTTP response hlavičkou Cache-Control

## Post & Back Replay Attack

- Nutný fyzický přístup útočníka
- Cache-Control: no-cache, no-store, must-revalidate
- Při procházení stránkami zpět
- Na POST požadavky vracet přesměrování 302

## WWW-Authenticate Attack

- Zneužití HTTP autentizace
- Nepovolujte uživatelům vkládání externího obsahu (např. obrázky)
- Ve chvíli, kdy návštěvník webu navštíví webovou stránku s externím obsahem, který je chráněn HTTP autentizací, vyvolá prohlížeč přihlašovací formulář

## Cross-Site Websockets hijacking

- Při navazování websocket spojení
- Authentizaci není možné provést jen na základě cookies
- Útočník by mohl vytvořit websocket ze svého webu


# Cross-Site Scripting (XSS)

## Injekce skriptů do obsahu

- Pro injekci skriptů není možné měnit kódy HTML dokumentů na webovém serveru
- Pokud existuje chyba v zabezpečení, je možné injektovat skript do HTML dokumentu při jeho sestavování a zobrazování uživateli
- Rozdělení XSS
  - Perzistentní (trvalé)
  - Non-perzistentní (reflektované)
  - DOM-based XSS
  - Self-XSS

### Perzistentní (trvalé) XSS

- Skript uložen v datovém úložišti na straně serveru
  - V databázi
  - V souboru
- Skript je zobrazen (vykonán) vždy, když je zobrazena webová stránka
- Místa častého výskytu
  - Webové diskuze (komentáře)
  - Profil uživatele
  - Tabulky výsledků (nejlepší hráči)
  - Nejvyhledávanější fráze
  - Zprávy ve webmailu (soukromé zprávy v aplikaci)

### Non-perzistentní XSS

- Dočasné, reflektované
- Skript uložen na straně uživatele
  - Ve formě odkazu
  - Ve formě webové stránky odesílající požadavek
- Skript odesílá uživatel na stranu serveru jako součást HTTP requestu
- Server převezme hodnotu předanou uživatelem a zakomponuje ji do HTML obsahu stránky, kterou vrátí uživateli
- Místa častého výskytu
  - Vyhledávání
  - Registrační formuláře
  - Přihlašovací formuláře
  - Stránky vkládající do HTML obsah URL
  - Chybové stránky (404)
  - Redirekty
